---
title: "hardtime.nvimでVimキーバインドを矯正する" 
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["vim", "neovim"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# hardtime.nvimでVimキーバインドを矯正する

なんちゃってVim使いのみなさん、こんにちは。
Vimを使っているのに、「j」連打や「l」連打でカーソルを動かしている人はVSCodeを使いましょう。
hardtime.nvimは、なんとしてでもVimのキーバインドを覚えたい人向けのプラグインです。

### こんな感じで最適なキーを提案してくれます

![デモ動画](/images/hardtime-vim/hardtime-vim.gif)

![ポップアップで怒ってもらうデモ動画](/images/hardtime-vim/hardtime-vim-popup.mp4)

## hardtime.nvimとは

[hardtime.nvim](https://github.com/m4xshen/hardtime.nvim)は、Vimのキーバインドを矯正するためのNeovimプラグインです。
このプラグインは、Vimのキーバインドを使っているときに、特定のキーを連打することを防ぎます。

例えば、「j」を連打すると画面下にスクロールできますが、長いスクロールの場合非効率です。
その場合は、「j」を連打するのではなく、「Ctrl」+「d」や「Ctrl」+「u」を使うことを促します。
このプラグインを使うと、連打よりも効率的な方法でカーソルを移動することを促します。

## 導入方法

```lua
{
   "m4xshen/hardtime.nvim",
   lazy = false,
   dependencies = { "MunifTanjim/nui.nvim" },
   opts = {},
},
```

## ポップアップで怒ってもらう方法

```lua
  {
    "rcarriga/nvim-notify",
    event = "VeryLazy",
    config = function()
      require("notify").setup({
        stages = "fade_in_slide_out",
        timeout = 3000,
        render = "default",
        background_colour = "#000000",
      })
      vim.notify = require("notify")
    end,
  },
```
