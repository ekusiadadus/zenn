---
title: '$1で最大8時間の動画を話者分離・文字起こし・LLM分析するAWSパイプラインを作った'
type: 'tech'
topics: ['aws', 'stepfunctions', 'lambda', 'whisper', 'pyannote']
published: false
---

# $1で最大8時間の動画を話者分離・文字起こし・LLM分析するAWSパイプラインを作った

## TL;DR

- **AWS Step Functions + Lambda** で話者分離付き文字起こしパイプラインを構築
- **pyannote.audio 3.1** で話者分離、**faster-whisper** で文字起こし、**GPT-5-mini** でLLM分析
- 8時間の動画処理が **約$1** で完了
- 今後 **Google Meet 自動録画連携** も実装予定

リポジトリ: https://github.com/ekusiadadus/ek-transcript

## はじめに

ユーザーインタビューの録画を分析する機会が増えてきました。しかし、既存のサービスはコストが高く、話者分離の精度もイマイチ。そこで、AWSのサーバーレスサービスを組み合わせて、低コストで高品質な文字起こしパイプラインを自作することにしました。

### 要件

1. 最大8時間の長時間動画に対応
2. 話者分離（誰が何を言ったか識別）
3. 日本語の高精度文字起こし
4. LLMによる要約・分析
5. 低コスト（$1/動画程度）
6. フルサーバーレス

## アーキテクチャ概要

```
┌──────────────────────────────────────────────────────────────────────────┐
│                              INPUT                                        │
│                         S3: input-bucket                                  │
│                      ┌─────────────────┐                                  │
│                      │ video.mp4       │                                  │
│                      │ (最大8時間)      │                                  │
│                      └────────┬────────┘                                  │
│                               ↓                                           │
├──────────────────────────────────────────────────────────────────────────┤
│                          STEP FUNCTIONS                                   │
│  ┌────────────────────────────────────────────────────────────────────┐  │
│  │  ┌─────────────┐                                                   │  │
│  │  │ExtractAudio │ → ffmpeg で音声抽出（16kHz mono WAV）              │  │
│  │  └─────┬───────┘                                                   │  │
│  │        ↓                                                           │  │
│  │  ┌─────────────┐                                                   │  │
│  │  │ ChunkAudio  │ → 8分チャンク + 30秒オーバーラップに分割           │  │
│  │  └─────┬───────┘                                                   │  │
│  │        ↓                                                           │  │
│  │  ┌─────────────────────────────────────────────────────────────┐  │  │
│  │  │            MAP STATE (並列実行 MaxConcurrency=5)             │  │  │
│  │  │  ┌────────────┐ ┌────────────┐ ┌────────────┐               │  │  │
│  │  │  │DiarizeChunk│ │DiarizeChunk│ │DiarizeChunk│ ...           │  │  │
│  │  │  │ pyannote   │ │ pyannote   │ │ pyannote   │               │  │  │
│  │  │  └────────────┘ └────────────┘ └────────────┘               │  │  │
│  │  └───────────────────────┬─────────────────────────────────────┘  │  │
│  │                          ↓                                         │  │
│  │  ┌─────────────┐                                                   │  │
│  │  │MergeSpeakers│ → 埋め込みベクトルで話者統一 + オーバーラップ解決  │  │
│  │  └─────┬───────┘                                                   │  │
│  │        ↓                                                           │  │
│  │  ┌─────────────┐                                                   │  │
│  │  │SplitBySpeaker│ → 話者ごとに音声分割                             │  │
│  │  └─────┬───────┘                                                   │  │
│  │        ↓                                                           │  │
│  │  ┌─────────────────────────────────────────────────────────────┐  │  │
│  │  │            MAP STATE (並列実行 MaxConcurrency=10)            │  │  │
│  │  │  ┌───────────┐ ┌───────────┐ ┌───────────┐                  │  │  │
│  │  │  │Transcribe │ │Transcribe │ │Transcribe │ ...              │  │  │
│  │  │  │faster-    │ │faster-    │ │faster-    │                  │  │  │
│  │  │  │whisper    │ │whisper    │ │whisper    │                  │  │  │
│  │  │  └───────────┘ └───────────┘ └───────────┘                  │  │  │
│  │  └───────────────────────┬─────────────────────────────────────┘  │  │
│  │                          ↓                                         │  │
│  │  ┌─────────────┐                                                   │  │
│  │  │  Aggregate  │ → 文字起こし結果統合                              │  │
│  │  └─────┬───────┘                                                   │  │
│  │        ↓                                                           │  │
│  │  ┌─────────────┐                                                   │  │
│  │  │ LLMAnalysis │ → GPT-5-mini で要約・分析                         │  │
│  │  └─────────────┘                                                   │  │
│  └────────────────────────────────────────────────────────────────────┘  │
│                               ↓                                           │
├──────────────────────────────────────────────────────────────────────────┤
│                              OUTPUT                                       │
│   ┌────────────┐ ┌────────────┐ ┌────────────┐                          │
│   │transcript. │ │segments.   │ │analysis.   │                          │
│   │json        │ │json        │ │json        │                          │
│   └────────────┘ └────────────┘ └────────────┘                          │
└──────────────────────────────────────────────────────────────────────────┘
```

## 各コンポーネントの説明

### 1. ExtractAudio Lambda

動画から音声を抽出します。ffmpegを使用して16kHzモノラルWAVに変換。

```python
def extract_audio(input_path: str, output_path: str) -> None:
    """動画から16kHzモノラルWAVを抽出"""
    cmd = [
        "ffmpeg", "-i", input_path,
        "-vn",  # 映像なし
        "-acodec", "pcm_s16le",
        "-ar", "16000",  # 16kHz
        "-ac", "1",  # モノラル
        "-y", output_path,
    ]
    subprocess.run(cmd, check=True)
```

### 2. ChunkAudio Lambda

長時間音声を8分チャンク（+ 30秒オーバーラップ）に分割。Lambda の15分タイムアウトを回避しつつ、話者分離の精度を維持します。

```python
# チャンク設計
CHUNK_DURATION = 480  # 8分
OVERLAP_DURATION = 30  # 30秒オーバーラップ

# chunk_0: 0〜510秒 (effective: 0〜480)
# chunk_1: 450〜960秒 (effective: 480〜960)
# chunk_2: 900〜1410秒 (effective: 960〜1440)
```

### 3. DiarizeChunk Lambda (並列実行)

pyannote.audio 3.1 で話者分離を実行。各話者の埋め込みベクトルも抽出してS3に保存。

```python
from pyannote.audio import Pipeline

pipeline = Pipeline.from_pretrained(
    "pyannote/speaker-diarization-3.1",
    token=hf_token,
)

# 話者分離実行
diarization = pipeline({"waveform": audio_tensor, "sample_rate": sample_rate})

# 埋め込みベクトルを抽出（後の話者統一で使用）
speaker_embeddings = extract_speaker_embeddings(audio_path, segments)
```

### 4. MergeSpeakers Lambda

複数チャンクの話者を埋め込みベクトルのコサイン類似度でクラスタリングし、グローバルに統一。

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

### 5. SplitBySpeaker Lambda

統一された話者情報に基づいて音声をセグメントに分割。

### 6. Transcribe Lambda (並列実行)

faster-whisper (medium モデル) で高速文字起こし。

```python
from faster_whisper import WhisperModel

model = WhisperModel("medium", device="cpu", compute_type="int8")
segments, info = model.transcribe(audio_path, language="ja", beam_size=5)
text = "".join([seg.text for seg in segments])
```

### 7. LLMAnalysis Lambda

GPT-5-mini の Structured Outputs で構造化分析。

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

## Step Functions の定義

CDK (TypeScript) で定義しています。

```typescript
// Map state for parallel diarization
const diarizeChunks = new sfn.Map(this, "DiarizeChunks", {
  itemsPath: "$.chunks",
  maxConcurrency: 5,  // GPU リソース考慮
  parameters: {
    "bucket.$": "$.bucket",
    "chunk.$": "$$.Map.Item.Value",
  },
  resultPath: "$.chunk_results",
});

// Map state for parallel transcription
const transcribeSegments = new sfn.Map(this, "TranscribeSegments", {
  itemsPath: "$.segment_files",
  maxConcurrency: 10,
  parameters: {
    "bucket.$": "$.bucket",
    "segment_file.$": "$$.Map.Item.Value",
  },
  resultPath: sfn.JsonPath.DISCARD,  // 結果はS3経由
});
```

## コスト計算（$1 で 8時間動画）

実際に8時間（約900セグメント）の動画を処理した際のコスト内訳です。

| サービス | 使用量 | コスト |
|----------|--------|--------|
| Lambda (Diarize) | 10240MB × 10分 × 6チャンク | $0.10 |
| Lambda (Transcribe) | 3008MB × 30秒 × 900回 | $0.34 |
| Lambda (その他) | 各種処理 | $0.06 |
| Step Functions | 約6,000遷移 | $0.15 |
| S3 | 読み書き | $0.01 |
| OpenAI API | GPT-5-mini | $0.20 |
| **合計** | | **約 $0.86** |

:::message
pyannote.audio と faster-whisper をフル活用することで、AWS Transcribe ($0.024/分 = 8時間で $11.52) と比較して **約13倍コスト効率** が良くなっています。
:::

## ハマりポイント: States.DataLimitExceeded

### 問題

900+セグメントを処理すると、Step Functions の Map state で `States.DataLimitExceeded` エラーが発生。

```
States.DataLimitExceeded - The state/task returned a result with a size exceeding the maximum number of bytes service limit.
```

Step Functions は **256KB のペイロード制限** があり、Map state の結果を全て蓄積すると超過してしまいます。

### 解決策

1. **各 Lambda で結果を S3 に保存** し、メタデータのみ返す
2. **Map state の `resultPath` を `DISCARD`** に設定
3. **後続の Lambda で S3 から読み込み**

```typescript
// Before (NG)
resultPath: "$.transcription_results",  // 蓄積される

// After (OK)
resultPath: sfn.JsonPath.DISCARD,  // 結果を破棄
```

```python
# Lambda 側で S3 に保存
s3.put_object(
    Bucket=bucket,
    Key=f"transcribe_results/{segment_name}.json",
    Body=json.dumps(result_data, ensure_ascii=False),
)

# メタデータのみ返す
return {"bucket": bucket, "result_key": result_key}
```

## 今後の展望: Google Meet 自動連携

Google Meet REST API の **Auto-Recording 機能** (2025年4月追加) を使い、Google Calendar の会議を自動録画・自動分析する機能を計画中。

```
Google Calendar → Google Meet (Auto-Recording)
       ↓
  Google Drive (録画保存)
       ↓
  Pub/Sub → EventBridge → Lambda
       ↓
  S3 → Step Functions (既存パイプライン)
       ↓
  DynamoDB + AppSync → Dashboard
```

設計ドキュメントは [docs/google-meet-integration/](https://github.com/ekusiadadus/ek-transcript/tree/main/docs/google-meet-integration) に公開しています。

## まとめ

- **AWS Step Functions + Lambda** で低コスト話者分離文字起こしパイプラインを構築
- **pyannote.audio** + **faster-whisper** + **GPT-5-mini** の組み合わせで高品質・低コスト
- **8時間動画を約$1** で処理可能
- **Map state + 並列実行** で処理時間を大幅短縮
- **256KB 制限** は S3 経由で回避

全コードは [GitHub](https://github.com/ekusiadadus/ek-transcript) で公開しています。

## 参考資料

- [pyannote.audio 3.1](https://huggingface.co/pyannote/speaker-diarization-3.1)
- [faster-whisper](https://github.com/SYSTRAN/faster-whisper)
- [AWS Step Functions ドキュメント](https://docs.aws.amazon.com/step-functions/)
- [Google Meet REST API](https://developers.google.com/workspace/meet/api/guides/overview)
