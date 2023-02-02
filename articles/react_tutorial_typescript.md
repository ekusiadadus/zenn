---
title: React の 旧チュートリアル にキレています
description: "React の 旧チュートリアル にキレています"
tags: ["React", "TypeScript"]
slide: false
---

# React の チュートリアルを TypeScript + 関数コンポーネント で書き直す

どうも、[@ekusiadadus](https://twitter.com/ekusiadadus) です。

今回は、React の公式チュートリアルを TypeScript + 関数コンポーネントで書き直してみました。
React の旧チュートリアルは、本当の React 初心者にとってはかなり厳しいと思っています。

今頃、クラスコンポーネントで実装しているフロントエンド開発現場なんて限られていると思います。
新チュートリアルは、 関数コンポーネントで書き直している + もっと優しくなっています。

(フロントエンドガチ勢でもなければ、React ガチ勢でもないです 🙇)

**[旧 React のチュートリアル](https://ja.reactjs.org/tutorial/tutorial.html)**

https://ja.reactjs.org/tutorial/tutorial.html

**[新 React のチュートリアル(おすすめ)](https://beta.reactjs.org/learn/tutorial-tic-tac-toe)**

https://beta.reactjs.org/learn/tutorial-tic-tac-toe

一応、自分の作業リポジトリも載せておきます 🙇

https://github.com/ekusiadadus/react-old-tutorial

## 旧チュートリアル

旧チュートリアルは、[スターターコード](https://codepen.io/gaearon/pen/oWWQNa?editors=0010) が用意されています。

### 旧チュートリアルの『スターターコードの中身を確認する』の改修

そもそも、初期状態で Class コンポーネントで書かれています。
旧チュートリアルでいうと、[ここ](https://ja.reactjs.org/tutorial/tutorial.html#inspecting-the-starter-code) からです。

https://ja.reactjs.org/tutorial/tutorial.html#inspecting-the-starter-code

```tsx
export const Square = () => {
  return <button className="square">{/* TODO */}</button>;
};
```

```tsx
export const Board = () => {
  // デフォで rederSquare 刺さっているけどまぁいっかの気持ち
  return (
    <div>
      <div className="board-row">
        <Square />
        <Square />
        <Square />
      </div>
      <div className="board-row">
        <Square />
        <Square />
        <Square />
      </div>
      <div className="board-row">
        <Square />
        <Square />
        <Square />
      </div>
    </div>
  );
};
```

```tsx
export const Game = () => {
  return (
    <div className="game">
      <div className="game-board">
        <Board />
      </div>
      <div className="game-info">
        <div>{/* status */}</div>
        <ol>{/* TODO */}</ol>
      </div>
    </div>
  );
};
```

まずこの時点で、大分コードの見え方が全然違います。

### 旧チュートリアルの『データを props 経由で渡す』の改修

次に、[ここ](https://ja.reactjs.org/tutorial/tutorial.html#passing-data-through-props) からです。

https://ja.reactjs.org/tutorial/tutorial.html#passing-data-through-props

ここでは、`Board` コンポーネントから `Square` コンポーネントにデータを渡す方法が記載されています。
React 初心者で props という表現でちゃんと理解できる人はどのくらいなのだろうか...🤔

```tsx
export const Board = () => {
  // デフォで rederSquare 刺さっているけどまぁいっかの気持ち <- いらないと判断したので消します。
  // 原理主義者なら、ここで renderSquare を定義して、Board の中で使うというのもありかもしれません。
  return (
    <div>
      <div className="board-row">
        <Square value={"0"} />
        <Square value={"1"} />
        <Square value={"2"} />
      </div>
      <div className="board-row">
        <Square value={"3"} />
        <Square value={"4"} />
        <Square value={"5"} />
      </div>
      <div className="board-row">
        <Square value={"6"} />
        <Square value={"7"} />
        <Square value={"8"} />
      </div>
    </div>
  );
};
```

```tsx
// ここの value は string か number かという議論はあります。
export const Square = ({ value }: { value: string }) => {
  return <button className="square">{value}</button>;
};
```

### 旧チュートリアルの『インタラクティブなコンポーネントを作る』の改修

次に、[ここ](https://ja.reactjs.org/tutorial/tutorial.html#making-an-interactive-component) からです。

https://ja.reactjs.org/tutorial/tutorial.html#making-an-interactive-component

ここでは、Square コンポーネントがクリックされたときに文字列"X" を表示するようにしています。
このことから、上の value props は string だと判断しています。(諸説あります)

#### クリックしたときの動作チェック

```tsx
export const Square = ({ value }: { value: string }) => {
  return (
    <button
      className="square"
      onClick={() => {
        alert("click");
      }}
    >
      {value}
    </button>
  );
};
```

#### クリックしたときに文字列"X" を表示するようにする (state)

```tsx
export const Square = ({ value }: { value: string }) => {
  // ここでは、本家コードだと state, setState になっています
  // 予約語の state と setState は使えないので、value2, setValue2 にしています
  const [value2, setValue2] = useState<string>(value);

  return (
    <button
      className="square"
      onClick={() => {
        setValue2("X");
      }}
    >
      {value2}
    </button>
  );
};
```

### 旧チュートリアルの『state のリフトアップ』の改修

次に、[ここ](https://ja.reactjs.org/tutorial/tutorial.html#lifting-state-up) からです。

https://ja.reactjs.org/tutorial/tutorial.html#lifting-state-up

おそらく、React の初心者にとっては、ここが一番の鬼門になっていると思います。
というのも、ここで、state を親コンポーネントに持たせるということをやっているからです。
これ、本当の React 初心者には、なかなか難しいと思います。(React の概念が理解できていないと難しい)

上の方で定義した value2 という Square コンポーネントの state を 親の Board コンポーネントにリフトアップします。 <- リフトアップという言葉も非英語圏の人にはちょっとわかりづらい

```tsx
export const Board = () => {
  // 本家はここで null が入っていますがstringだけの型にしたいので、0-8 までの数字を入れています。
  const [squares, setSquares] = useState<string[]>(
    [...Array(9)].map((_, i) => i.toString())
  );

  // ここで、setSquares を関数の中で呼び出すようにしている
  // setSquares を直接子コンポーネントに渡すとき、TypeScript だと Dispatch 型みたいなので渡すことになる
  const handlerClick = (i: number) => {
    const squares2 = squares.slice();
    squares2[i] = "X";
    setSquares(squares2);
  };

  return (
    <div>
      <div className="board-row">
        <Square value={squares[0]} onClick={() => handlerClick(0)} />
        <Square value={squares[1]} onClick={() => handlerClick(1)} />
        <Square value={squares[2]} onClick={() => handlerClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onClick={() => handlerClick(3)} />
        <Square value={squares[4]} onClick={() => handlerClick(4)} />
        <Square value={squares[5]} onClick={() => handlerClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onClick={() => handlerClick(6)} />
        <Square value={squares[7]} onClick={() => handlerClick(7)} />
        <Square value={squares[8]} onClick={() => handlerClick(8)} />
      </div>
  );
};
```

```tsx
// ここのonClick の型は、() => void となっています
// setSquares を渡すと、Dispatch 型になってしまうので注意
const Square = ({ value, onClick }: { value: string; onClick: () => void }) => {
  return (
    <button className="square" onClick={onClick}>
      {value}
    </button>
  );
};
```

### 旧チュートリアルの『手番の処理』の改修

次に、[ここ](https://ja.reactjs.org/tutorial/tutorial.html#taking-turns) からです。
この章では、手番の処理を実装しています。

https://ja.reactjs.org/tutorial/tutorial.html#taking-turns

```tsx
export const Board = () => {
  const [squares, setSquares] = useState<string[]>(
    [...Array(9)].map((_, i) => i.toString())
  );
  const [xIsNext, setXIsNext] = useState<boolean>(true);

  const handlerClick = (i: number) => {
    const squares2 = squares.slice();
    squares2[i] = xIsNext ? "X" : "O";
    setSquares(squares2);
    setXIsNext(!xIsNext);
  };

  return (
    <div>
      <div className="status">Next player: {xIsNext ? "X" : "O"}</div>
      <div className="board-row">
        <Square value={squares[0]} onClick={() => handlerClick(0)} />
        <Square value={squares[1]} onClick={() => handlerClick(1)} />
        <Square value={squares[2]} onClick={() => handlerClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onClick={() => handlerClick(3)} />
        <Square value={squares[4]} onClick={() => handlerClick(4)} />
        <Square value={squares[5]} onClick={() => handlerClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onClick={() => handlerClick(6)} />
        <Square value={squares[7]} onClick={() => handlerClick(7)} />
        <Square value={squares[8]} onClick={() => handlerClick(8)} />
      </div>
  );
};
```

### 旧チュートリアルの『ゲーム勝者の判定』の改修

次に、[ここ](https://ja.reactjs.org/tutorial/tutorial.html#declaring-a-winner) からです。

https://ja.reactjs.org/tutorial/tutorial.html#declaring-a-winner

ここで、勝者の判定を行う関数を実装しています。
React 初心者にとっては、この関数と状態の管理ちゃんと分けて考えられているのかな？というのが気になります...🤔

```tsx
export const calculateWinner = (squares: string[]) => {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
};
```

```tsx
export const Board = () => {
  const [squares, setSquares] = useState<string[]>(
    [...Array(9)].map((_, i) => i.toString())
  );
  const [xIsNext, setXIsNext] = useState<boolean>(true);

  const handlerClick = (i: number) => {
    const squares2 = squares.slice();
    squares2[i] = xIsNext ? "X" : "O";
    setSquares(squares2);
    setXIsNext(!xIsNext);
  };

  const winner = calculateWinner(squares);
  const status = winner
    ? `Winner: ${winner}`
    : `Next player: ${xIsNext ? "X" : "O"}`;

  return (
    <div>
      <div className="status">{status}</div>
      <div className="board-row">
        <Square value={squares[0]} onClick={() => handlerClick(0)} />
        <Square value={squares[1]} onClick={() => handlerClick(1)} />
        <Square value={squares[2]} onClick={() => handlerClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onClick={() => handlerClick(3)} />
        <Square value={squares[4]} onClick={() => handlerClick(4)} />
        <Square value={squares[5]} onClick={() => handlerClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onClick={() => handlerClick(6)} />
        <Square value={squares[7]} onClick={() => handlerClick(7)} />
        <Square value={squares[8]} onClick={() => handlerClick(8)} />
      </div>
    </div>
  );
};
```

### 旧チュートリアルの『着手の履歴の保存』の改修

次に、[ここ](https://ja.reactjs.org/tutorial/tutorial.html#storing-a-history) からです。

https://ja.reactjs.org/tutorial/tutorial.html#storing-a-history

ここではさらに鬼畜なことに、Board で定義した state を Game に再びリフトアップするということをしています。
おそらく、React ガチ初心者には鬼畜な所業でしょう。(自分が初心者のときは何やっているのかさっぱりだった)

```tsx
export const Game = () => {
  const [history, setHistory] = useState<{ squares: string[] }[]>([
    { squares: [...Array(9)].map((_, i) => i.toString()) },
  ]);
  const [xIsNext, setXIsNext] = useState<boolean>(true);

  return(
          <div className="game">
        <div className="game-board">
          <Board />
        </div>
        <div className="game-info">
          <div>{/* status */}</div>
          <ol>{/* TODO */}</ol>
        </div>
      </div>
  );
);
```

1. Board の constructor を削除する。
2. Board の renderSquare にある this.state.squares[i] を this.props.squares[i] に置き換える。
3. Board の renderSquare にある this.handleClick(i) を this.props.onClick(i) に置き換える。

↑ を 関数コンポーネントで置き換えると以下になります。

1. Board の state を削除する。
2. Board の props にある squares[i] を 親コンポーネントから渡される props.squares[i] に置き換える。
3. Board の props にある onClick(i) を 親コンポーネントから渡される props.onClick(i) に置き換える。

```tsx
export const Board = ({squares, onClick}:{ squares: string[]; onClick: (i: number) => void }) => {

  return (
    <div>
      <div className="board-row">
        <Square value={squares[0]} onClick={() => onClick(0)} />
        <Square value={squares[1]} onClick={() => onClick(1)} />
        <Square value={squares[2]} onClick={() => onClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onClick={() => onClick(3)} />
        <Square value={squares[4]} onClick={() => onClick(4)} />
        <Square value={squares[5]} onClick={() => onClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onClick={() => onClick(6)} />
        <Square value={squares[7]} onClick={() => onClick(7)} />
        <Square value={squares[8]} onClick={() => onClick(8)} />
      </div>
  );
};
```

```tsx
export const Game = () => {
  const [history, setHistory] = useState<{ squares: string[] }[]>([
    { squares: [...Array(9)].map((_, i) => i.toString()) },
  ]);
  const [xIsNext, setXIsNext] = useState<boolean>(true);

  const handleClick = (i: number) => {
    const history2 = history.slice(0, history.length);
    const current = history2[history2.length - 1];
    const squares = current.squares.slice();
    if (calculateWinner(squares) === "X" || calculateWinner(squares) === "O") {
      return;
    }
    if (squares[i] === "X" || squares[i] === "O") {
      alert("This square is already taken");
      return;
    }
    squares[i] = xIsNext ? "X" : "O";
    setHistory(history2.concat([{ squares: squares }]));
    setXIsNext(!xIsNext);
  };

  const current = history[history.length - 1];
  const winner = calculateWinner(current.squares);
  const status = winner
    ? `Winner: ${winner}`
    : `Next player: ${xIsNext ? "X" : "O"}`;

  return (
    <div className="game">
      <div className="game-board">
        <Board squares={current.squares} onClick={handleClick} />
      </div>
      <div className="game-info">
        <div>{status}</div>
        <ol>{/* TODO */}</ol>
      </div>
    </div>
  );
};
```

### 旧チュートリアルの『過去の着手の表示』の改修

次に、[ここ](https://ja.reactjs.org/tutorial/tutorial.html#showing-the-past-moves) からです。

https://ja.reactjs.org/tutorial/tutorial.html#showing-the-past-moves

ここではさらにさらに鬼畜なことに、Game の state history を使って 着手履歴を表示するということをしています。
とくに jumpTo() が実装されていないのが、このチュートリアルの鬼畜さです。

```tsx
export const Game = () => {
  const [history, setHistory] = useState<{ squares: string[] }[]>([
    { squares: [...Array(9)].map((_, i) => i.toString()) },
  ]);
  const [xIsNext, setXIsNext] = useState<boolean>(true);

  const handleClick = (i: number) => {
    const history2 = history.slice(0, history.length);
    const current = history2[history2.length - 1];
    const squares = current.squares.slice();
    if (calculateWinner(squares) === "X" || calculateWinner(squares) === "O") {
      return;
    }
    if (squares[i] === "X" || squares[i] === "O") {
      alert("This square is already taken");
      return;
    }
    squares[i] = xIsNext ? "X" : "O";
    setHistory(history2.concat([{ squares: squares }]));
  };

  const jumpTo = (step: number) => {};

  const current = history[history.length - 1];
  const winner = calculateWinner(current.squares);
  const status = winner
    ? `Winner: ${winner}`
    : `Next player: ${xIsNext ? "X" : "O"}`;
  const moves = history.map((step, move) => {
    const desc = move ? `Go to move #${move}` : "Go to game start";
    return (
      <li key={move}>
        <button onClick={() => jumpTo(move)}>{desc}</button>
      </li>
    );
  });

  return (
    <div className="game">
      <div className="game-board">
        <Board squares={current.squares} onClick={handleClick} />
      </div>
      <div className="game-info">
        <div>{status}</div>
        <ol>{moves}</ol>
      </div>
    </div>
  );
};
```

### 旧チュートリアルの『タイムトラベルの実装』の改修

一応公式チュートリアルの最後で、[ここ](https://ja.reactjs.org/tutorial/tutorial.html#implementing-time-travel) からです。

https://ja.reactjs.org/tutorial/tutorial.html#implementing-time-travel

ここで、jumpTo() を実装して stepNumber も入れます。
ここまでたどり着ける React 初心者は一体何人いるのでしょうか..🤔

```tsx
export const Game = () => {
  const [history, setHistory] = useState<{ squares: string[] }[]>([
    { squares: [...Array(9)].map((_, i) => i.toString()) },
  ]);
  const [xIsNext, setXIsNext] = useState<boolean>(true);
  const [stepNumber, setStepNumber] = useState<number>(0);

  const handleClick = (i: number) => {
    const history2 = history.slice(0, stepNumber + 1);
    const current = history2[history2.length - 1];
    const squares = current.squares.slice();

    if (calculateWinner(squares) === "X" || calculateWinner(squares) === "O") {
      return;
    }
    if (squares[i] === "X" || squares[i] === "O") {
      alert("This square is already taken");
      return;
    }

    squares[i] = xIsNext ? "X" : "O";
    setHistory(history2.concat([{ squares: squares }]));
    setStepNumber(history2.length);
    setXIsNext(!xIsNext);
  };

  const jumpTo = (step: number) => {
    setStepNumber(step);
    setXIsNext(step % 2 === 0);
  };

  const current = history[stepNumber];
  const winner = calculateWinner(current.squares);
  const status = winner
    ? `Winner: ${winner}`
    : `Next player: ${xIsNext ? "X" : "O"}`;
  const moves = history.map((step, move) => {
    const desc = move ? `Go to move #${move}` : "Go to game start";
    return (
      <li key={move}>
        <button onClick={() => jumpTo(move)}>{desc}</button>
      </li>
    );
  });

  return (
    <div className="game">
      <div className="game-board">
        <Board squares={current.squares} onClick={handleClick} />
      </div>
      <div className="game-info">
        <div>{status}</div>
        <ol>{moves}</ol>
      </div>
    </div>
  );
};
```

## 最後に

今回は、React の公式チュートリアルを TypeScript + 関数コンポーネントで書き直してみました。
React の旧チュートリアルは、本当の React 初心者にとってはかなり厳しいと思っています。

さらに、今頃 class コンポーネントで実装しているフロントエンド開発現場なんて限られていると思います。
React の新チュートリアルは、React の公式チュートリアルを 関数コンポーネントで書き直している + もっと優しくなっています。
[こちら](https://beta.reactjs.org/learn/tutorial-tic-tac-toe) から見れます。

https://beta.reactjs.org/learn/tutorial-tic-tac-toe
