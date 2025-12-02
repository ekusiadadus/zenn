---
title: 'GTMのES5地獄から解放！Claude Code Skillを作ってマーケットプレイスに公開した話'
type: 'tech'
topics: ['claudecode', 'googletagmanager', 'javascript', 'analytics', 'ga4']
published: false
---

# GTMのES5地獄から解放！Claude Code Skillを作ってマーケットプレイスに公開した話

## TL;DR

- Google Tag Manager (GTM) のカスタムHTMLタグは**ES5しか対応していない**
- Claude Codeでコード生成すると`const`や`=>`がバンバン出てきて毎回修正が必要だった
- **Claude Code Skill**を作成して問題を根本解決
- さらに**マーケットプレイスに公開**して誰でもインストール可能に

リポジトリ: https://github.com/ekusiadadus/claude-skill-gtm-javascript

## 発端: GTMカスタムHTMLタグの闘い

### GTMの厳しい現実

Google Tag Managerでカスタムトラッキングを実装する際、**カスタムHTMLタグ**を使うことがあります。ここで問題になるのが、GTMのJavaScriptコンパイラは**ES5モードでしか動作しない**ということ。

つまり、以下のようなモダンなJavaScriptは**全てエラー**になります：

```javascript
// これは全部NG
const items = [];           // const禁止
let count = 0;              // let禁止
const fn = () => {};        // アロー関数禁止
const str = `Hello ${name}`; // テンプレートリテラル禁止
const {a, b} = obj;         // 分割代入禁止
```

### Claude Codeとの格闘

Claude Codeは非常に優秀で、GTMのコードも書いてくれます。しかし...

```
私: 「GTMでフォーム送信をトラッキングするカスタムHTMLタグを書いて」

Claude: 「はい、こちらです」
```

```javascript
// Claudeが生成したコード
(() => {
  const form = document.querySelector('form');
  form?.addEventListener('submit', (e) => {
    const formData = new FormData(e.target);
    window.dataLayer.push({
      event: 'form_submit',
      formId: form.id
    });
  });
})();
```

**これをGTMに貼り付けると...**

```
エラー: Unexpected token 'const'
エラー: Arrow function is not supported
```

毎回手動で`const`→`var`、`=>`→`function`に書き換える作業が発生。これが本当に辛かった。

## 解決策: Claude Code Skillを作る

### Skillとは？

Claude Code Skillは、**Claudeが自動的に参照する専門知識パッケージ**です。

特定のタスクに対して、Claudeが自動的にスキルを検出し、適切な知識を使ってコードを生成してくれます。詳しくは[公式ドキュメント](https://docs.anthropic.com/ja/docs/claude-code/skills)を参照してください。

### ディレクトリ構造

```
claude-skill-gtm-javascript/
├── .claude-plugin/
│   ├── plugin.json          # プラグインメタデータ
│   └── marketplace.json     # マーケットプレイス設定
├── skills/
│   └── gtm-javascript/
│       ├── SKILL.md         # メインスキル定義（必須）
│       ├── reference.md     # ES6→ES5変換リファレンス
│       ├── examples.md      # 実践コード例
│       └── checklist.md     # テスト・検証チェックリスト
├── README.md
└── LICENSE
```

### SKILL.mdの作成

スキルの核となる`SKILL.md`を作成します。YAMLフロントマターで`name`と`description`を定義するのが必須です。

```markdown
---
name: gtm-javascript
description: Generate ES5-compliant JavaScript for Google Tag Manager Custom HTML tags. Use when writing GTM tags, dataLayer code, or analytics implementations.
---

# GTM JavaScript Coding Standards

This skill ensures all JavaScript code generated for Google Tag Manager (GTM) Custom HTML tags is **ES5-compliant** and follows current best practices (2024-2025).

## Critical Constraint: ES5 Only

GTM's JavaScript compiler operates in **ES5 (ECMAScript 5) mode by default**. ES6+ syntax causes compilation errors and prevents tag publishing.

### Prohibited ES6+ Features

**NEVER use these in GTM Custom HTML tags:**

| Feature | ES6+ (Prohibited) | ES5 (Required) |
|---------|-------------------|----------------|
| Variables | `const`, `let` | `var` |
| Functions | `() => {}` | `function() {}` |
| Strings | `` `${var}` `` | `'str' + var` |
| Destructuring | `{a, b} = obj` | `var a = obj.a` |
| Spread | `[...arr]` | `arr.concat()` |
| for-of | `for (x of arr)` | `for (var i...)` |

## Code Patterns

### IIFE Pattern (Recommended)

(function() {
  'use strict';

  window.dataLayer = window.dataLayer || [];
  window.dataLayer.push({
    event: 'my_event',
    my_parameter: 'value'
  });
})();
```

### descriptionの書き方のコツ

`description`はClaudeがスキルを自動検出するための重要な要素です。以下を含めると効果的：

1. **何ができるか**: "Generate ES5-compliant JavaScript for GTM"
2. **いつ使うか**: "Use when writing GTM tags, dataLayer code"

## マーケットプレイスへの公開

### Step 1: plugin.jsonの作成

`.claude-plugin/plugin.json`を作成します：

```json
{
  "name": "gtm-javascript",
  "description": "Generate ES5-compliant JavaScript for GTM Custom HTML tags",
  "version": "1.0.0",
  "author": {
    "name": "ekusiadadus"
  },
  "homepage": "https://github.com/ekusiadadus/claude-skill-gtm-javascript",
  "repository": "https://github.com/ekusiadadus/claude-skill-gtm-javascript",
  "license": "MIT",
  "keywords": ["gtm", "google-tag-manager", "javascript", "es5", "analytics"]
}
```

### Step 2: marketplace.jsonの作成

複数のプラグインをカタログ化するためのマーケットプレイス設定です：

```json
{
  "name": "claude-skill-gtm-javascript",
  "owner": {
    "name": "ekusiadadus"
  },
  "plugins": [
    {
      "name": "gtm-javascript",
      "source": "./",
      "description": "Generate ES5-compliant JavaScript for GTM Custom HTML tags",
      "version": "1.0.0"
    }
  ]
}
```

### Step 3: GitHubにプッシュ

```bash
git add .
git commit -m "feat: add plugin manifest for marketplace distribution"
git push origin main
```

## インストール方法

公開後、以下のコマンドでインストールできます：

```bash
# Step 1: マーケットプレイスとして追加
/plugin marketplace add ekusiadadus/claude-skill-gtm-javascript

# Step 2: プラグインをインストール
/plugin install gtm-javascript@claude-skill-gtm-javascript
```

インストール後、Claude Codeを再起動すれば完了です。

## 実際に使ってみる

### Before（スキルなし）

```
私: 「GTMでフォーム送信をトラッキングして」
```

```javascript
// ES6が出力されてエラー
const form = document.querySelector('form');
form?.addEventListener('submit', () => {
  // ...
});
```

### After（スキルあり）

```
私: 「GTMでフォーム送信をトラッキングして」
```

```javascript
// ES5準拠のコードが自動生成
(function() {
  'use strict';

  var form = document.querySelector('form');
  if (form) {
    form.addEventListener('submit', function(e) {
      window.dataLayer = window.dataLayer || [];
      window.dataLayer.push({
        event: 'form_submit',
        form_id: form.id || 'unknown'
      });
    });
  }
})();
```

## 2024-2025年の重要な変更点

スキルには最新のGTM変更点も含めています：

| 変更 | 日付 | 影響 |
|------|------|------|
| IE11サポート終了 | 2024年7月15日 | IE11互換性不要に |
| Consent Mode v2必須 | 2024年3月 | `ad_user_data`, `ad_personalization`パラメータ追加 |
| Google Ads自動タグ | 2025年4月10日 | コンテナがGoogleタグを自動読み込み |

### Consent Mode v2の例

```javascript
window.dataLayer = window.dataLayer || [];
function gtag() { dataLayer.push(arguments); }

// デフォルト状態（同意前）
gtag('consent', 'default', {
  ad_storage: 'denied',
  ad_user_data: 'denied',
  ad_personalization: 'denied',
  analytics_storage: 'denied'
});
```

## まとめ

Claude Code Skillを作成することで：

1. **毎回の手動修正が不要に** - ES5準拠のコードが自動生成
2. **最新のベストプラクティスを反映** - Consent Mode v2、GA4対応
3. **チームで共有可能** - マーケットプレイス経由で誰でもインストール
4. **メンテナンスが容易** - GitHubで一元管理

GTMのカスタムHTMLタグで消耗している方は、ぜひ試してみてください。

## 参考リンク

### 公式ドキュメント

- [Claude Code Skills](https://docs.anthropic.com/ja/docs/claude-code/skills) - スキルの公式ガイド
- [Claude Code Plugins](https://docs.anthropic.com/ja/docs/claude-code/plugins) - プラグインの作成方法
- [Plugin Marketplaces](https://docs.anthropic.com/ja/docs/claude-code/plugin-marketplaces) - マーケットプレイスの公開方法

### 参考記事

- [Claude Code Skills について理解する](https://zenn.dev/ino_h/articles/2025-10-23-claude-code-skills-guide) - Skills の詳細な解説
- [Claude Code の Skills/Subagent/Plugin を理解してチームの生産性を爆上げする](https://dev.classmethod.jp/articles/claude-code-skills-subagent-plugin-guide/) - 実践的なプラグイン作成ガイド

### GTM関連

- [GTM Developer Guide](https://developers.google.com/tag-manager)
- [GA4 Ecommerce](https://developers.google.com/analytics/devguides/collection/ga4/ecommerce)
- [Consent Mode](https://developers.google.com/tag-platform/security/guides/consent)

### リポジトリ

- [claude-skill-gtm-javascript](https://github.com/ekusiadadus/claude-skill-gtm-javascript) - 本記事で紹介したスキル
