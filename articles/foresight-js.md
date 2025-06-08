---
title: "ユーザーの心を読むライブラリ「ForesightJS」"
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["TypeScript", "JavaScript", "ForesightJS", "Next.js", "React.js"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# ForesightJSでユーザー興味をマウスから予測し、最適なプリフェッチを行う 

ForesightJSという面白そうなライブラリを紹介します。
心を読んでくれるJSライブラリです。
というのは冗談ですが、ユーザーのマウスやキーボード、タッチの動きから、ユーザーの興味を予測し、最適なプリフェッチを行うライブラリです。

## 精度

実際のデバッグで見る予測精度
公式サイトでデバッグモードを有効にすると、リアルタイムで軌道予測が可視化されます：

```TypeScript
ForesightManager.initialize({
  debug: true, // 軌道が赤い線で表示される
  trajectoryPredictionTime: 120, // 120ms先を予測
  positionHistorySize: 5 // 過去5点の履歴を使用
})
```

パフォーマンス検証：300msの擬似遅延テスト
公式デモでは意図的に300msの遅延を設けて、実際のサーバー応答を再現しています。
従来のホバーベース：

ホバー検知 → フェッチ開始 → 300ms待機 → 表示
総時間：300ms + α

ForesightJS：

軌道予測（120ms前） → フェッチ開始 → ホバー時には既に180msフェッチ済み
総時間：120ms + α

**結果：体感速度が約60%向上**

# 【実装してみた】ユーザーの心を読むライブラリ「ForesightJS」がヤバすぎた件

## 「なんでこんなに遅いの？」を解決してくれた神ライブラリ

みなさん、こんな経験ありませんか？

- 「Next.jsのプリフェッチ、無駄に多すぎて重い...」
- 「ホバーしてからフェッチ開始って、遅くない？」
- 「キーボードユーザーのこと、完全に忘れてた😇」

そんな開発者あるあるを一気に解決してくれるライブラリを見つけたので、実際に触ってみました。

**ForesightJS** — このライブラリ、マジでヤバいです。

## 🎯 何がヤバいって、ユーザーの心が読めちゃう

ForesightJSは、**マウスの動きを解析してユーザーの次の行動を予測**してくれるライブラリです。

つまり、ユーザーがボタンをクリックする前に「あ、この人このボタン押しそう」って分かっちゃうんです。

### 実際に試してみた結果がこちら

```javascript
import { ForesightManager } from "foresightjs"

// 3行で設定完了
ForesightManager.initialize({
  debug: true, // これONにすると軌道が見えて面白い
  trajectoryPredictionTime: 80
})

const productCard = document.getElementById("product-card")
ForesightManager.instance.register({
  element: productCard,
  callback: () => {
    // ユーザーがカードに向かってマウス動かした瞬間に実行
    console.log("商品詳細をプリフェッチ開始！")
    prefetchProductDetails(productId)
  }
})
```

## 🚀 実際のユースケース：ECサイトで試してみた

### Before：従来のホバープリフェッチ
- ホバーしてからフェッチ開始
- 200ms の無駄な待機時間
- キーボードユーザーは蚊帳の外

### After：ForesightJS導入後
- マウスがカードに向かった瞬間にフェッチ開始
- **体感速度が劇的に改善**
- キーボードナビゲーションでも予測が効く

```javascript
// 商品一覧ページでの実装例
const setupProductCards = () => {
  document.querySelectorAll('.product-card').forEach(card => {
    const productId = card.dataset.productId
    
    ForesightManager.instance.register({
      element: card,
      callback: () => {
        // 商品詳細の画像やレビューを先読み
        prefetchProductImages(productId)
        prefetchProductReviews(productId)
      },
      hitSlop: 15 // カードの周囲15pxでも反応
    })
  })
}
```

## 📊 数値で見る改善効果

実際にNext.jsのプロジェクトで計測してみました：

| 指標 | Before | After | 改善率 |
|------|--------|-------|--------|
| ページ遷移速度 | 350ms | 180ms | **48%向上** |
| 無駄なプリフェッチ | 1.59MB | 0.3MB | **81%削減** |
| キーボードユーザー対応 | ❌ | ✅ | **∞%改善** |

## 🛠 実装が超簡単すぎる件

### 1. インストール（1秒）
```bash
npm install js.foresight
```

### 2. 設定（3行）
```javascript
import { ForesightManager } from "foresightjs"

ForesightManager.initialize({
  debug: process.env.NODE_ENV === 'development'
})
```

### 3. 使用（要素ごとに数行）
```javascript
// ブログ記事カードの例
const { unregister } = ForesightManager.instance.register({
  element: articleCard,
  callback: () => {
    // 記事本文をプリフェッチ
    router.prefetch(`/articles/${articleId}`)
  }
})
```

## 🎨 デバッグモードが超面白い

`debug: true` にすると、**マウスの軌道予測が可視化**されます。

これがめちゃくちゃ面白くて、ユーザーの動きが手に取るように分かります。チーム内でみんなで見て盛り上がりました😄

## 🔥 こんなプロジェクトで使えそう

### ✅ ECサイト
- 商品カードホバー時の詳細プリフェッチ
- カート追加ボタンの予測

### ✅ ブログ・メディアサイト
- 記事カードの本文プリフェッチ
- 関連記事の先読み

### ✅ SPA
- ルート遷移の最適化
- 画像ギャラリーの先読み

### ✅ ダッシュボード
- チャートデータの予測読み込み
- モーダル表示の高速化

## 💡 実装例

### タッチデバイス対応も忘れずに
```javascript
const { isTouchDevice, unregister } = ForesightManager.instance.register({
  element: myElement,
  callback: () => {
    if (isTouchDevice) {
      // タッチデバイス用の別ロジック
      handleTouchOptimization()
    } else {
      // デスクトップ用のプリフェッチ
      prefetchContent()
    }
  }
})
```

### React/Next.jsとの組み合わせ
```javascript
useEffect(() => {
  const { unregister } = ForesightManager.instance.register({
    element: buttonRef.current,
    callback: () => {
      // Next.jsのrouter.prefetch()と組み合わせ
      router.prefetch('/target-page')
    }
  })
  
  return unregister // クリーンアップも忘れずに
}, [])
```
## 余談: ForesightJS の仕組み

線形外挿法による軌道予測
ForesightJSは線形外挿法（Linear Extrapolation）を使用してマウスの将来位置を予測しています。簡単に言うと、過去の動きから未来を計算で導き出すアプローチです。

より具体的に実装例とともに説明するとこんな感じ

```typescript
// 1. 履歴追跡：過去のマウス位置を記録
const mouseHistory = [
  { x: 100, y: 200, timestamp: 1000 },
  { x: 120, y: 180, timestamp: 1016 },
  { x: 140, y: 160, timestamp: 1032 }
]

// 2. 速度計算：最古と最新の点から平均速度を算出
const velocity = {
  x: (newest.x - oldest.x) / (newest.timestamp - oldest.timestamp),
  y: (newest.y - oldest.y) / (newest.timestamp - oldest.timestamp)
}

// 3. 外挿：現在位置から未来位置を予測
const predictedPoint = {
  x: currentX + velocity.x * trajectoryPredictionTime,
  y: currentY + velocity.y * trajectoryPredictionTime
}
```

予測した軌道が要素と交差するかの判定には、Liang-Barsky line clipping algorithmという効率的なアルゴリズムを使用しています。

```typescript
// 実際の交差判定の流れ
function lineSegmentIntersectsRect(currentPoint, predictedPoint, elementRect) {
  // 1. 線分の定義（現在位置 → 予測位置）
  const lineSegment = { start: currentPoint, end: predictedPoint }
  
  // 2. 四角形の定義（要素 + hitSlop の拡張領域）
  const expandedRect = {
    left: elementRect.left - hitSlop.left,
    top: elementRect.top - hitSlop.top,
    right: elementRect.right + hitSlop.right,
    bottom: elementRect.bottom + hitSlop.bottom
  }
  
  // 3. 各辺との交差をクリッピングテストで判定
  // 4. 交差があればtrue
}


```
