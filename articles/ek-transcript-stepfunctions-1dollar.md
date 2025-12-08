---
title: '$1で最大8時間の動画を話者分離・文字起こし・LLM分析するAWSパイプラインを作った'
type: 'tech'
topics: ['aws', 'stepfunctions', 'lambda', 'whisper', 'pyannote']
published: true
---

## TL;DR

- **月額固定費 約$2**（Secrets Manager + ECR + ログ）で話者分離付き文字起こしパイプラインを構築
- **AWS Step Functions + Lambda** のフルサーバーレス構成
- **pyannote.audio 3.1** で話者分離、**faster-whisper** で文字起こし、**gpt-5-mini** でLLM分析
- 8時間の動画処理が **約$0.76** で完了（AWS Transcribe比で約15倍コスト効率）
- **States.DataLimitExceeded** などの落とし穴と解決策を詳解

リポジトリ: https://github.com/ekusiadadus/ek-transcript

## はじめに

ユーザーインタビューの録画を分析する機会が増えてきました。既存サービスを検討しましたが：

- **AWS Transcribe**: 8時間で約$11.52（$0.024/分）、話者分離精度がイマイチ
- **商用SaaS**: 月額 $50〜$200 の固定費、使わない月も課金される
- **GPUサーバー常時起動**: EC2 g4dn.xlarge で月額 $380+、個人には厳しい

**最大の問題は「固定費」でした。** 月に数回しか使わないのに毎月課金されるのは避けたい。使った分だけ払う従量課金で、**月額固定費を限りなくゼロに抑えたい**—これが最優先要件でした。

そこで、AWSサーバーレスサービスを組み合わせて自作することにしました。

### 要件

1. **月額固定費ゼロ**（使った分だけ従量課金）
2. 最大8時間の長時間動画に対応
3. 話者分離（誰が何を言ったか識別）
4. 日本語の高精度文字起こし
5. LLMによる要約・分析
6. 低コスト（$1/動画程度）
7. フルサーバーレス

## システム構成図

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                    AWS Cloud                                     │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  ┌──────────────┐     ┌─────────────────┐                                       │
│  │   Amazon S3  │     │   EventBridge   │                                       │
│  │ (Input)      │────▶│   Rule          │                                       │
│  │ uploads/     │     │ (Object Created)│                                       │
│  └──────────────┘     └────────┬────────┘                                       │
│                                │                                                 │
│                                ▼                                                 │
│                       ┌────────────────┐      ┌─────────────────┐               │
│                       │     Lambda     │      │    DynamoDB     │               │
│                       │ StartPipeline  │─────▶│ InterviewsTable │               │
│                       └────────┬───────┘      └─────────────────┘               │
│                                │                       ▲                         │
│                                ▼                       │                         │
│  ┌─────────────────────────────────────────────────────┼───────────────────────┐│
│  │                     AWS Step Functions              │                       ││
│  │                                                     │                       ││
│  │  ┌─────────────┐   ┌─────────────┐   ┌────────────┐│                       ││
│  │  │   Lambda    │   │   Lambda    │   │  Lambda    ││                       ││
│  │  │ExtractAudio │──▶│ ChunkAudio  │──▶│(Map State) ││                       ││
│  │  │  (ffmpeg)   │   │ (8min+30s)  │   │DiarizeChunk││                       ││
│  │  └─────────────┘   └─────────────┘   │ x5 並列    ││                       ││
│  │        │                              │ pyannote   ││                       ││
│  │        │                              └─────┬──────┘│                       ││
│  │        ▼                                    │       │                       ││
│  │  ┌──────────┐                               ▼       │                       ││
│  │  │Amazon S3 │◀───────────────────┬─────────────────┤                       ││
│  │  │(Output)  │                    │                 │                       ││
│  │  │processed/│   ┌────────────────┴──────┐          │                       ││
│  │  │analysis/ │   │       Lambda          │          │                       ││
│  │  └──────────┘   │    MergeSpeakers      │          │                       ││
│  │       ▲         │ (埋め込みクラスタリング)│          │                       ││
│  │       │         └───────────┬───────────┘          │                       ││
│  │       │                     ▼                      │                       ││
│  │       │         ┌───────────────────────┐          │                       ││
│  │       │         │       Lambda          │          │                       ││
│  │       │         │   SplitBySpeaker      │          │                       ││
│  │       │         │     (ffmpeg)          │          │                       ││
│  │       │         └───────────┬───────────┘          │                       ││
│  │       │                     ▼                      │                       ││
│  │       │         ┌───────────────────────┐          │                       ││
│  │       │         │       Lambda          │          │                       ││
│  │       │         │     (Map State)       │          │                       ││
│  │       │         │   Transcribe x10      │          │                       ││
│  │       │         │   faster-whisper      │          │                       ││
│  │       │         └───────────┬───────────┘          │                       ││
│  │       │                     ▼                      │                       ││
│  │       │         ┌───────────────────────┐          │                       ││
│  │       │         │       Lambda          │          │                       ││
│  │       ├─────────│  AggregateResults     │          │                       ││
│  │       │         └───────────┬───────────┘          │                       ││
│  │       │                     ▼                      │                       ││
│  │       │         ┌───────────────────────┐   ┌─────────────────┐            ││
│  │       │         │       Lambda          │   │Secrets Manager  │            ││
│  │       └─────────│     LLMAnalysis       │◀──│ OpenAI API Key  │            ││
│  │                 │    gpt-5-mini         │   └─────────────────┘            ││
│  │                 └───────────┬───────────┘                                  ││
│  │                             │                                              ││
│  └─────────────────────────────┼──────────────────────────────────────────────┘│
│                                │                                                │
│                                ▼                                                │
│                       ┌────────────────┐      ┌─────────────────┐              │
│                       │  EventBridge   │      │    DynamoDB     │              │
│                       │  (Completion)  │─────▶│ (status更新)    │              │
│                       └────────────────┘      └─────────────────┘              │
│                                                        │                        │
├────────────────────────────────────────────────────────┼────────────────────────┤
│                                                        ▼                        │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                           AWS AppSync (GraphQL)                          │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                      │                                          │
└──────────────────────────────────────┼──────────────────────────────────────────┘
                                       │
                                       ▼
                              ┌─────────────────┐
                              │   Frontend      │
                              │   (Next.js)     │
                              └─────────────────┘
```

### データフロー詳細

```
[動画アップロード]
       │
       ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│ S3: ek-transcript-input-{env}                                                │
│ Key: uploads/{interview_id}/video.mp4                                        │
│ Metadata: x-amz-meta-interview-id, x-amz-meta-original-filename              │
└──────────────────────────────────────────────────────────────────────────────┘
       │
       │ EventBridge (Object Created)
       ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│ Lambda: StartPipeline                                                         │
│ - DynamoDB に interview レコード作成                                          │
│ - Step Functions 実行開始                                                     │
└──────────────────────────────────────────────────────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│ Step Functions: ek-transcript-pipeline-{env}                                  │
│                                                                               │
│ 1. ExtractAudio: video.mp4 → audio.wav (16kHz mono)                          │
│ 2. ChunkAudio: audio.wav → chunk_0.wav, chunk_1.wav, ... (8分+30秒overlap)   │
│ 3. DiarizeChunks (Map x5): 各チャンクで pyannote 話者分離                     │
│ 4. MergeSpeakers: 埋め込みベクトルでグローバル話者統一                        │
│ 5. SplitBySpeaker: 話者セグメントごとに音声分割                               │
│ 6. TranscribeSegments (Map x10): faster-whisper で文字起こし                  │
│ 7. AggregateResults: 結果統合 → transcript.json                              │
│ 8. LLMAnalysis: gpt-5-mini で構造化分析 → analysis.json                      │
└──────────────────────────────────────────────────────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│ S3: ek-transcript-output-{env}                                               │
│ - processed/{interview_id}/audio.wav                                         │
│ - processed/{interview_id}/segments.json (話者分離結果)                       │
│ - processed/{interview_id}/transcript.json (文字起こし)                       │
│ - analysis/{interview_id}_structured.json (LLM分析結果)                       │
└──────────────────────────────────────────────────────────────────────────────┘
```

## なぜサーバーレスか: 月額固定費を最小化

このパイプラインの最大の特徴は **月額固定費を極限まで抑えた** という点です。

| 構成要素 | 課金形態 | 月額固定費 |
|----------|----------|-----------|
| Lambda | 実行時間課金 | $0 |
| Step Functions | 状態遷移課金 | $0 |
| S3 | ストレージ + リクエスト課金 | $0〜 |
| DynamoDB | オンデマンド課金 | $0 |
| EventBridge | イベント課金 | $0 |
| Cognito | 50,000 MAU まで無料 | $0 |
| AppSync | リクエスト課金 | $0〜 |
| **Secrets Manager** | シークレット数課金 | **$0.80/月**（2つ） |
| **ECR** | コンテナイメージ保存 | **$0.50〜$1.00/月** |
| **CloudWatch Logs** | ログ保存 | **$0.05/月** |

### 実際の月額固定費

```
Secrets Manager:    $0.80/月（OpenAI + HuggingFace の2シークレット）
ECR:               $0.50〜$1.00/月（MLモデル含むDockerイメージ 8個）
CloudWatch Logs:    $0.05/月程度（Step Functions + Lambda ログ）
───────────────────────────────────────────
合計:               約 $1.50〜$2.00/月
```

:::message
**使わない月でも約 $2。** 商用SaaSの月額 $50〜$200 と比較すると、年間で $576〜$2,376 の節約になります。
:::

## 設計変遷: 当初案から現在の設計へ

### 当初案: 単純な直列処理

```
[動画] → ExtractAudio → Diarize → SplitBySpeaker → Transcribe → LLMAnalysis
                           │
                    (単一Lambda で全音声処理)
```

**問題点:**
- Lambda 15分タイムアウトで8時間音声の話者分離が完了しない
- pyannote.audio のメモリ使用量が膨大（10GB超）
- 直列処理で全体の処理時間が長い

### 改善案1: ECS Fargate での処理

```
[動画] → ECS Fargate (GPU) → ...
```

**検討結果:**
- GPU インスタンス（g4dn.xlarge）のコストが高い（$0.526/時間）
- 8時間動画で約$4以上のコスト
- スポットインスタンスでも安定性に不安

### 現在の設計: チャンク並列処理

```
                    ┌─ DiarizeChunk_0 ─┐
[動画] → Chunk →   ├─ DiarizeChunk_1 ─┤ → Merge → Split → Transcribe(並列) → LLM
                    ├─ DiarizeChunk_2 ─┤
                    └─      ...       ─┘
```

**設計ポイント:**
1. **8分チャンク + 30秒オーバーラップ**: Lambda 15分制限内で処理可能、境界の話者変化も捕捉
2. **埋め込みベクトルによる話者統一**: チャンク間で SPEAKER_00 が別人でも、コサイン類似度でクラスタリング
3. **Map State 並列実行**: 話者分離 x5、文字起こし x10 の並列処理で高速化

## 技術選定と理由

| 技術 | 選定理由 | 代替案との比較 |
|------|---------|---------------|
| **pyannote.audio 3.1** | 最新の話者分離精度、Hugging Face 統合 | AWS Transcribe の話者分離より高精度 |
| **faster-whisper** | Whisper の4-8倍高速、int8量子化対応 | OpenAI Whisper API はコスト高 |
| **gpt-5-mini** | Structured Outputs 対応、低コスト | Claude は Structured Outputs 未対応（当時） |
| **Lambda + コンテナ** | 最大10GB イメージ、コールドスタート許容 | ECS Fargate は常時起動コストが課題 |
| **Step Functions** | 複雑なワークフロー管理、エラーハンドリング | SQS + Lambda は状態管理が複雑 |

## 各コンポーネントの詳細実装

### 1. ExtractAudio Lambda

動画から16kHzモノラルWAVを抽出。Whisperの推奨サンプリングレート。

```python
def extract_audio(input_path: str, output_path: str) -> None:
    """動画から16kHzモノラルWAVを抽出"""
    cmd = [
        "ffmpeg", "-i", input_path,
        "-vn",                    # 映像なし
        "-acodec", "pcm_s16le",   # 16bit PCM
        "-ar", "16000",           # 16kHz
        "-ac", "1",               # モノラル
        "-y", output_path,
    ]
    subprocess.run(cmd, check=True)
```

### 2. ChunkAudio Lambda

8分チャンク + 30秒オーバーラップで分割。オーバーラップにより境界での話者変化を正確に捕捉。

```python
CHUNK_DURATION = 480      # 8分
OVERLAP_DURATION = 30     # 30秒オーバーラップ

# chunk_0: 0〜510秒 (effective: 0〜480)
# chunk_1: 450〜960秒 (effective: 480〜960)
# chunk_2: 900〜1410秒 (effective: 960〜1440)
```

### 3. DiarizeChunk Lambda (並列実行)

pyannote.audio 3.1 で話者分離。各話者の埋め込みベクトルも抽出してS3に保存。

```python
from pyannote.audio import Pipeline

pipeline = Pipeline.from_pretrained(
    "pyannote/speaker-diarization-3.1",
    token=hf_token,
)

# GPU があれば使用
if torch.cuda.is_available():
    pipeline.to(torch.device("cuda"))

# 話者分離実行
diarization = pipeline({"waveform": audio_tensor, "sample_rate": sample_rate})

# 埋め込みベクトルを抽出（後の話者統一で使用）
speaker_embeddings = extract_speaker_embeddings(audio_path, segments)
```

### 4. MergeSpeakers Lambda

複数チャンクの話者を埋め込みベクトルのコサイン類似度でクラスタリング。

```python
from sklearn.cluster import AgglomerativeClustering
from sklearn.metrics.pairwise import cosine_similarity

# コサイン類似度でクラスタリング
similarity_matrix = cosine_similarity(all_embeddings)
distance_matrix = 1 - similarity_matrix

clustering = AgglomerativeClustering(
    n_clusters=None,
    distance_threshold=1 - 0.75,  # 類似度75%以上を同一話者
    metric="precomputed",
    linkage="average",
)
labels = clustering.fit_predict(distance_matrix)
```

### 5. Transcribe Lambda (並列実行)

faster-whisper (medium モデル) で高速文字起こし。

```python
from faster_whisper import WhisperModel

model = WhisperModel("medium", device="cpu", compute_type="int8")
segments, info = model.transcribe(audio_path, language="ja", beam_size=5)
text = "".join([seg.text for seg in segments])
```

### 6. LLMAnalysis Lambda

gpt-5-mini の Structured Outputs で構造化分析。

```python
from openai import OpenAI

# Structured Outputs でスコアリング
completion = client.beta.chat.completions.parse(
    model="gpt-5-mini",
    messages=[
        {"role": "system", "content": ANALYSIS_PROMPT},
        {"role": "user", "content": f"分析してください:\n{transcript}"},
    ],
    response_format=AnalysisResult,
)
```

## コスト計算（$1 で 8時間動画）

実際に8時間（約900セグメント）の動画を処理した際のコスト内訳：

| サービス | 使用量 | コスト |
|----------|--------|--------|
| Lambda (Diarize) | 10240MB × 10分 × 6チャンク | $0.10 |
| Lambda (Transcribe) | 3008MB × 30秒 × 900回 | $0.34 |
| Lambda (その他) | 各種処理 | $0.06 |
| Step Functions | 約6,000遷移 | $0.15 |
| S3 | 読み書き | $0.01 |
| OpenAI API | gpt-5-mini (入力300K + 出力8K tokens) | $0.10 |
| **合計** | | **約 $0.76** |

:::message alert
**月額固定費: $0**
Lambda、Step Functions、S3 はすべて従量課金。使わなければ課金されません。
:::

:::message
pyannote.audio と faster-whisper をフル活用することで、AWS Transcribe ($0.024/分 = 8時間で $11.52) と比較して **約15倍コスト効率** が良くなっています。
:::

## 実装でハマったポイント集

### 1. States.DataLimitExceeded（256KB制限）

**現象:**
900+セグメントを処理すると、Step Functions の Map state で以下のエラーが発生。

```
States.DataLimitExceeded - The state/task returned a result with a size
exceeding the maximum number of bytes service limit.
```

**原因:**
Step Functions は **256KB のペイロード制限**があり、Map state の結果を全て蓄積すると超過。

**解決策:**
```typescript
// CDK: Map state の結果を破棄
const transcribeSegments = new sfn.Map(this, "TranscribeSegments", {
  itemsPath: "$.segment_files",
  maxConcurrency: 10,
  resultPath: sfn.JsonPath.DISCARD,  // ← これが重要
});
```

```python
# Lambda 側: 結果は S3 に保存
s3.put_object(
    Bucket=bucket,
    Key=f"transcribe_results/{segment_name}.json",
    Body=json.dumps(result_data, ensure_ascii=False),
)
# Step Functions にはメタデータのみ返す
return {"bucket": bucket, "result_key": result_key}
```

### 2. PyTorch 2.6+ の torch.load 問題

**現象:**
pyannote.audio のモデル読み込みで以下のエラー:

```
FutureWarning: You are using `torch.load` with `weights_only=False`
```

PyTorch 2.6 からデフォルトが `weights_only=True` に変更され、一部モデルが読み込めなくなった。

**解決策: torch.load をモンキーパッチで上書き**
```python
import torch

# PyTorch 2.6+ の weights_only=True デフォルトを無効化
# pyannote の HuggingFace チェックポイントは信頼できるソースなので問題なし
_orig_torch_load = torch.load

def _torch_load_legacy(*args, **kwargs):
    """torch.load を常に weights_only=False で呼び出す"""
    kwargs["weights_only"] = False
    return _orig_torch_load(*args, **kwargs)

torch.load = _torch_load_legacy  # pyannote import 前に適用
```

**重要:** このパッチは `from pyannote.audio import Pipeline` より**前**に適用する必要がある。

### 3. Lambda コンテナのモデルダウンロード戦略

**問題:**
- Hugging Face モデル（pyannote, whisper）は数GBのサイズ
- Lambda の `/tmp` は 512MB〜10GB（設定による）
- コールドスタート時のダウンロードで Lambda タイムアウト

**解決策: ビルド時にモデルを含める**
```dockerfile
# Dockerfile
FROM public.ecr.aws/lambda/python:3.11

# ビルド時に Hugging Face モデルをダウンロード
ENV HF_HOME=/var/task/models
RUN pip install huggingface_hub
RUN python -c "from huggingface_hub import snapshot_download; \
    snapshot_download('pyannote/speaker-diarization-3.1', token='${HF_TOKEN}')"
```

**重要:** HF_TOKEN は **ビルド引数**で渡し、最終イメージには含めない
```dockerfile
ARG HF_TOKEN
RUN --mount=type=secret,id=hf_token \
    HF_TOKEN=$(cat /run/secrets/hf_token) python download_models.py
```

### 4. HF_TOKEN のセキュアな管理

**問題:**
- Hugging Face の pyannote モデルは認証が必要
- Lambda 環境変数に直接書くとセキュリティリスク

**解決策: AWS Secrets Manager + ビルド時ダウンロード**
```python
# Lambda 実行時には Secrets Manager から取得
# （ただし実際にはビルド時に含めるので実行時は不要）
secrets_client = boto3.client("secretsmanager")
secret = secrets_client.get_secret_value(SecretId=HF_SECRET_ARN)
hf_token = json.loads(secret["SecretString"])["token"]
```

### 5. 8分チャンク長の選定

**試行錯誤:**
| チャンク長 | 結果 |
|-----------|------|
| 5分 | 話者分離精度が低下（文脈が短すぎる） |
| 10分 | Lambda メモリ不足（10GB でもギリギリ） |
| 15分 | Lambda 15分タイムアウト超過 |
| **8分** | 精度・メモリ・時間のバランス最適 |

**オーバーラップ30秒の理由:**
- 話者交代は通常2-3秒の間隔
- 30秒あれば境界での話者変化を確実に捕捉
- それ以上長くすると重複処理が増えてコスト増

### 6. Lambda vs ECS の判断基準

**Lambda を選んだ理由:**
```
処理時間 < 15分 かつ メモリ < 10GB → Lambda
処理時間 > 15分 または GPU必須 → ECS Fargate
```

pyannote.audio は CPU でも動作し、8分チャンクなら Lambda 制限内に収まる。

## 今後の展望: Google Meet 自動連携

Google Meet REST API の **Auto-Recording 機能** (2025年4月追加) を使い、自動録画・自動分析する機能を計画中。

```
Google Calendar (会議予定)
       │
       ▼ Cloud Functions (Calendar Webhook)
Google Meet Space (Auto-Recording 設定)
       │
       ▼ 録画完了
Google Drive (録画保存)
       │
       ▼ Workspace Events API + Pub/Sub
EventBridge (Cross-Cloud)
       │
       ▼
Lambda (DownloadRecording)
       │
       ▼
S3 → Step Functions (既存パイプライン)
       │
       ▼
DynamoDB + AppSync → Dashboard
```

設計ドキュメント: [docs/google-meet-integration/](https://github.com/ekusiadadus/ek-transcript/tree/main/docs/google-meet-integration)

## まとめ

- **月額固定費 約$2**（Secrets Manager + ECR + ログ）で話者分離文字起こしパイプラインを実現
- **AWS Step Functions + Lambda** のフルサーバーレス構成で使った分だけ課金
- **pyannote.audio** + **faster-whisper** + **gpt-5-mini** で高品質・低コスト
- **8時間動画を約$0.76** で処理（AWS Transcribe 比 15倍コスト効率）
- **チャンク並列処理** + **埋め込みクラスタリング** で長時間音声に対応
- **256KB 制限** は `resultPath: DISCARD` + S3 経由で回避

全コードは [GitHub](https://github.com/ekusiadadus/ek-transcript) で公開しています。

:::message alert
**ところで Secrets Manager、1シークレット $0.40/月って高くない？** API キー2つ保存するだけで年間 $9.60。SSM Parameter Store の SecureString なら無料なのに...。でもまあ、自動ローテーションとか監査ログとか、エンタープライズ機能のお布施だと思って諦めてます。
:::

## 参考資料

- [pyannote.audio 3.1](https://huggingface.co/pyannote/speaker-diarization-3.1)
- [faster-whisper](https://github.com/SYSTRAN/faster-whisper)
- [AWS Step Functions ドキュメント](https://docs.aws.amazon.com/step-functions/)
- [AWS Step Functions Quotas (256KB制限)](https://docs.aws.amazon.com/step-functions/latest/dg/limits-overview.html)
- [Google Meet REST API](https://developers.google.com/workspace/meet/api/guides/overview)
