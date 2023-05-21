---
title: "将棋 UI ライブラリを Vite.js で作ってみた！ - with ChatGPT -" # 記事のタイトル
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["npm", "TypeScript", "ChatGPT", "Vite", "将棋"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# 将棋 UI ライブラリを Vite.js で作ってみた！ - with ChatGPT -

![react-shogi1.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/ca2151c9-5e3d-86a8-914e-f6e09ee2114b.gif)

## はじめに

こんにちは！ 私は[@ekusiadadus](https://twitter.com/ekusiadadus)と申します。今回は、以前から作りたいと思っていた将棋の UI ライブラリを Vite.js で作ってみました！ そして、そのかかった時間はなんと... 約 10 時間！

「えっ、そんなに短時間で将棋 UI ライブラリが作れちゃったの？」と思ったそこのあなた！ それはね、実はこの開発では ChatGPT を使って質問しながらアドバイスをもらいコーディングを行うという、全く新しい開発スタイルが確立できたからなんです！ おかげで、スムーズに作業が進み、あっという間にライブラリを開発・公開できました。

そして、最も印象的だったのが KIF 形式の変換で悩んだ際、実は ChatGPT からもらったアドバイスが非常に役立ったんです！ おかげで、手間が省け、楽しく開発が進められました。

将棋 UI を作る際に、最も苦労した点は KIF 形式の変換でした。最終的には、自分で正規表現を用いた KIF 形式のパーサを実装することにしました。実は、この部分でかなりのアドバイスが ChatGPT からもらえました。これにより、手間が省け、スムーズに作業が進められました。

## 作ったもの

デモページはこちらです!
[https://ekusiadadus.github.io/react-shogi](https://ekusiadadus.github.io/react-shogi/)

リポジトリも公開しています！
[https://github.com/ekusiadadus/react-shogi](https://github.com/ekusiadadus/react-shogi)

## 将棋 UI の実装

```
.
├── Board
│   └── Board.tsx
├── EmbeddedShogi
│   ├── EmbeddedShogi.tsx
│   └── index.tsx
├── Game
│   └── Game.tsx
├── Komadai
│   └── Komadai.tsx
├── Piece
│   └── Piece.tsx
└── Square
    └── Square.tsx
```

### 駒

将棋の**駒**を表すために、enum で独自に駒定義を作っていきました。

`src/mode/pieceType.ts` ファイルに将棋の駒の定義を書き、実際には方眼にこの `src/Piece/Piece.tsx` で表示するような実装にしています。

```typescript
// TypeScriptのEnumを使用して駒の種類を定義します
export enum PieceType {
  Gyoku = "Gyoku",
  Hisha = "Hisha",
   ...
}

// 駒の種類に対応する表示を表すオブジェクト
export const PieceDisplay = {
  [PieceType.Gyoku]: "玉",
  [PieceType.Hisha]: "飛",
   ...
}

export interface PieceState {
  type: PieceType
  direction: "up" | "down"
}

export interface PieceStateWithCount extends PieceState {
  count: number
}
```

駒自体は、こんな感じで UI 上は表すようにしています
PieceDisplay で駒の表示を変換しています。

```typescript
import { PieceDisplay, type PieceType } from "../../model/pieceType"

export const Piece = ({
  type,
  direction,
  position
}: {
  type: PieceType | null
  direction: "up" | "down"
  position: {
    row: number
    column: number
  }
}) => {
  const transform = direction === "up" ? "rotate(0deg)" : "rotate(180deg)"
  const displayType = type === null ? "" : PieceDisplay[type]

  return (
    <button
      style={{
        display: "flex",
           ...
      }}
      onClick={() => {
        alert(
          `row: ${position.row}, column: ${position.column}, type: ${displayType}`
        )
      }}
    >
      {type === null ? "" : PieceDisplay[type]}
    </button>
  )
}

export default Piece
```

### 方眼

**方眼** は、駒や、向き(先手、後手)を考慮するような将棋盤の一コンポーネントとして実装しています。
割とどこまでの情報を持つか？等を悩んでいたりしました。

```typescript
import type { PieceType } from "../../model/pieceType";
import Piece from "../Piece/Piece";

export const Square = ({
  piece,
  position,
}: {
  piece: {
    type: PieceType;
    direction: "up" | "down";
  } | null;
  position: {
    row: number;
    column: number;
  };
}) => {
  return (
    <div className="square">
      {piece === null && (
        <Piece type={null} direction="up" position={position} />
      )}
      {piece !== null && (
        <Piece
          type={piece.type}
          direction={piece.direction}
          position={position}
        />
      )}
    </div>
  );
};

export default Square;
```

### 将棋盤

**将棋盤** は、実際の将棋盤を表すような UI コンポーネントにしています。
ラベルや、駒台等を表示するようにしています。

```typescript
import type { BoardType } from "../../model/boardType"
import { Komadai } from "../Komadai/Komadai"
import { Square } from "../Square/Square"

const Board = ({ board }: { board: BoardType }) => {
  const rowLabels = ["一", "二", "三", "四", "五", "六", "七", "八", "九"]
  const columnLabels = ["1", "2", "3", "4", "5", "6", "7", "8", "9"].reverse()

  return (
    <div
      style={{
        display: "flex",
   ...
      }}
    >
      <Komadai pieces={board.upKomadai} turn="up" />
      <div
        style={{
          display: "flex",
   ...
        }}
      >
        <div style={{ display: "flex" }}>
          <div style={{ width: "40px" }}></div>
          {columnLabels.map((label, j) => (
            <div
              key={j}
   ...
              }}
            >
              {label}
            </div>
          ))}
          <div style={{ width: "40px" }}></div>
        </div>
        {board.board.map((row, i) => (
          <div key={i} style={{ display: "flex" }}>
            <div style={{ width: "40px" }}></div>
            {row.map((piece, j) => (
              <Square key={j} position={{ row: i, column: j }} piece={piece} />
            ))}

            <div
              style={{
                width: "40px",
   ...
              }}
            >
              {rowLabels[i]}
            </div>
          </div>
        ))}
        <div style={{ display: "flex" }}>
          <div style={{ width: "40px" }}></div>
          {columnLabels.map((_, j) => (
            <div
              key={j}
              style={{
                width: "40px",
   ...
              }}
            ></div>
          ))}
          <div style={{ width: "40px" }}></div>
        </div>
      </div>
      <Komadai pieces={board.downKomadai} turn="down" />
    </div>
  )
}

export default Board
```

### 駒台

**駒台**は、結構悩んで実装しました。
実装段階で、先手、後手の駒台の表現方法とかで 1 時間は溶かしていたと思います。

```typescript
import type { PieceType, PieceStateWithCount } from "../../model/pieceType"
import { PieceDisplay } from "../../model/pieceType"

export const Komadai = ({
  pieces,
  turn
}: {
  pieces: Record<PieceType, number>
  turn: "up" | "down"
}) => {
  const defaultPieceCount = 9 // Define the total number of slots in the grid

  // Prepare a list of pieces to display
  const displayPieces = Object.entries(pieces).reduce<
    Array<PieceStateWithCount | null>
  >((list, [type, count]) => {
    if (count > 0) {
      return [
        ...list,
        {
          type: type as PieceType, // Ensure the correct PieceType is assigned
          direction: "up" as const,
          count
        }
      ]
    } else {
      return list
    }
  }, [])

  // Add null pieces to fill the empty spaces
  while (displayPieces.length < defaultPieceCount) {
    displayPieces.push(null)
  }

  return (
    <div
      style={{
        display: "grid",
        gridTemplateColumns: "repeat(3, 40px)",
        gridTemplateRows: "repeat(3, 40px)", // Added to enforce a 3x3 grid
   ...
      }}
    >
      {displayPieces.map((piece, index) => (
        <button
          key={index}
          style={{
            display: "flex",
   ...
          }}
          onClick={() => {
            if (piece) {
              alert(
                `type: ${PieceDisplay[piece.type]}, direction: ${
                  piece.direction
                }`
              )
            }
          }}
          disabled={piece === null}
        >
          {piece === null ? "" : PieceDisplay[piece.type]}
          {piece && piece.count > 1 ? (
            <span
              style={{
                position: "absolute",
   ...
              }}
            >
              {piece.count}
            </span>
          ) : null}
        </button>
      ))}
    </div>
  )
}
```

### ゲーム情報

**ゲーム情報** も、どうやって

```typescript
import { useState, useEffect, useMemo } from "react"
import Board from "../Board/Board"
import { parseKIF } from "../../helpers/kifParser"

export const Game = ({ KIF }: { KIF: string }) => {
  const [stepNumber, setStepNumber] = useState(0)
  const [data, setData] = useState(KIF)
  const kifData = useMemo(() => parseKIF(data), [data])

  useEffect(() => {
    const handleWheel = (event: WheelEvent) => {
      if (event.deltaY > 0) {
        // Scrolling down
        nextStep()
      } else if (event.deltaY < 0) {
        // Scrolling up
        prevStep()
      }
    }

    window.addEventListener("wheel", handleWheel)
    return () => {
      window.removeEventListener("wheel", handleWheel)
    }
  }, [stepNumber, kifData])

  function nextStep() {
    if (stepNumber < kifData.length - 1) {
      setStepNumber(stepNumber + 1)
    }
  }

  function prevStep() {
    if (stepNumber > 0) {
      setStepNumber(stepNumber - 1)
    }
  }

  const [reverseBoardStyleString, setReverseBoardStyleString] =
    useState<string>("")
  return (
    <div
      className="game"
      style={{
        display: "flex",
   ...
      }}
    >
      <div
        className="board"
        style={{
          transform: reverseBoardStyleString
        }}
      >
        <Board board={kifData[stepNumber]} />
      </div>
      <div className="game-info">
        <button onClick={prevStep}>前へ戻る</button>
        <button onClick={nextStep}>次へ進む</button>
        {/* 現状のboardを表示するボタン */}
        <button
          onClick={() => {
            alert(JSON.stringify(kifData[stepNumber]))
          }}
        >
          現状のboardを表示する
        </button>
        {/* 反転する */}
        <button
          onClick={() => {
            setReverseBoardStyleString(
              reverseBoardStyleString === "rotate(180deg)"
                ? ""
                : "rotate(180deg)"
            )
          }}
        >
          反転する
        </button>
        {/* KIF形式の入力formとボタン */}
        <form
          onSubmit={e => {
            e.preventDefault()
            const inputValue = (
              document.getElementById("kif") as HTMLInputElement
            ).value
            setData(inputValue)
            setStepNumber(0)
          }}
        >
          <textarea
            name="kif"
            id="kif"
            cols={30}
            rows={10}
            defaultValue={data}
          ></textarea>
          <button type="submit">KIFを入力する</button>
        </form>
      </div>
    </div>
  )
}

export default Game
```

## ChatGPT と正規表現！

KIF 形式の変換が最も悩んだ実装ポイントでした。
結局、自分で正規表現で KIF 形式 parser を実装することにしました。

```typescript
const matches = line.match(
  /^((\d+) (同 |同　|同|\d{2})(玉|飛|龍|竜|角|馬|金|銀|成銀|全|桂|成桂|圭|香|成香|杏|歩|と)(打|成)?(\((\d{2})\))?)|(中断|投了|持将棋|千日手|切れ負け|反則勝ち|反則負け|入玉勝ち|不戦勝|不戦敗|詰み|不詰)|^(\*.*$)/
);
```

上は、ChatGPT にめちゃくちゃアドバイスをもらいました。

## Vite と コンポーネント

Vite を選んだのは、Vite のライブラリを作る機能を使用したかったからです。
実際に使ってみて、使いやすいと感じました。
しかし、チャンク戦略はあんまりよくないといわれているらしい＜要出典という感じでした

### Vite で 組み込みスクリプト

組み込みスクリプトを作るのは、Vite の機能で簡単にできます！
https://ekusiadadus.github.io/react-shogi/

```html
<!DOCTYPE html>

<head>
  <meta charset="utf-8" />
  <title>example</title>
</head>

<script
  id="shogi"
  type="text/javascript"
  async
  src="../dist/embedded-shogi.umd.cjs"
></script>

<script type="text/javascript">
  const script = document.getElementById("shogi");
  script.addEventListener("load", () => {
    globalThis.run();
  });
</script>
```

```ts
import { defineConfig } from "vite";
import { resolve } from "path";

export default defineConfig({
  build: {
    cssCodeSplit: true,
    lib: {
      entry: resolve(__dirname, "src/components/EmbeddedShogi/index.tsx"),
      formats: ["umd"],
      name: "embeddedShogi",
      fileName: "embedded-shogi",
    },
  },
  define: {
    "process.env": {},
  },
});
```

### Vite で ライブラリ

実は、ライブラリが壊れていて未完成です。

```
$ pnpm install react-shogi
```

等でインストールはできるのですが、型定義などがうまく生成できていないみたいで困っています。

```json tsconfig.library.json
{
  "compilerOptions": {
    "target": "ESNext",
    "lib": ["DOM", "DOM.Iterable", "ESNext"],
    "module": "ESNext",
    "skipLibCheck": true,

    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "emitDeclarationOnly": true,
    "declaration": true,
    "outDir": "dist",
    "declarationDir": "dist/types",
    "jsx": "react-jsx",

    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

```typescript vite.Library.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react-swc";
import { resolve } from "path";
import terser from "@rollup/plugin-terser";

export default defineConfig({
  plugins: [react()],
  build: {
    minify: false,
    lib: {
      entry: resolve(__dirname, "src/lib/react-shogi/ShogiGame.tsx"),
      name: "ShogiGame",
      fileName: "ShogiGame",
    },
    rollupOptions: {
      // 必要な外部依存関係を指定します。
      external: ["react", "react-dom"],
      output: {
        // ライブラリのグローバル変数名を指定します。
        globals: {
          react: "React",
          "react-dom": "ReactDOM",
        },
        plugins: [terser()],
      },
    },
  },
});
```

## 最後に

将棋 UI は前々から作ってみたいという気持ちがあったのですが、KIF 形式の Parser とかがかなり面倒だったので苦心していました。

ChatGPT に聞いてもらいながら作るという新しい開発スタイルが確立されたことによって、かなり簡単に(10 時間くらい)でライブラリを公開できました！
