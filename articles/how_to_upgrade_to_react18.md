# React 18 にアップグレードする

仕事で、React 16 or 17 -> React 18 にアップグレードするお祭りがあったので、備忘録です。
公式で、["How to Upgrade to React 18"](https://reactjs.org/blog/2022/03/08/react-18-upgrade-guide.html) というドキュメントが公開されているので参考にしてください。

## 1. React 18 にアップグレード

方法は、三つくらいあります。(たぶんもっとあると思います。)

1. `npm install react react-dom`
1. `yarn add react react-dom`
1. `yarn upgrade react react-dom @types/react @types/react-dom --latest`

[公式の記事](https://reactjs.org/blog/2022/03/08/react-18-upgrade-guide.html)は、1,2 が紹介されています。

## 2. index.tsx の書き換え

React 18 では、Client Rendering APIs のアップデートが必要です。
具体的には、`ReactDom.reder` は React 18 ではサポートされていません。

```typescript
// React 17以前
import ReactDom from "react-dom";
import App from "./App";

ReactDom.render(<App />, document.getElementById("app"));
```

```typescript
// React 18以降
import ReactDOM from "react-dom/client";
import App from "./App";

const container = document.getElementById("app");
if (container) {
  root.render(<App />);
}
```

もしくは、

```typescript
import { createRoot } from "react-dom/client";
const container = document.getElementById("app");
const root = createRoot(container); // createRoot(container!) if you use TypeScript
root.render(<App tab="home" />);
```

のように書き直す必要があります。
二つ目のは、公式のものです。
TypeScript の non-null warning が出力されるので、`container!` と書く必要があるみたいです。

## 3. React.FC の書き換え

> If your project uses TypeScript, you will need to update your @types/react and @types/react-dom dependencies to the latest versions. The new types are safer and catch issues that used to be ignored by the type checker. The most notable change is that the children prop now needs to be listed explicitly when defining props, for example:

[公式の記事](https://reactjs.org/blog/2022/03/08/react-18-upgrade-guide.html)

上のように、children props は明示的に渡す必要があります。

```typescript
// React 18以前
const HogeComponent: React.FC = () => {};
return (
  <HogeComponent>
    <div> hoge </div>
  </HogeComponent>
);
```

```typescript
// React 18以降
interface HogeProps {
  children: React.ReactNode;
}
const HogeComponent = (props: HogeProps) => {};
return (
  <HogeComponent>
    <div> hoge </div>
  </HogeComponent>
);
```

## 4. TypeScript の型エラー修正

TypeScript のバージョンを上げたので、Class を使用している場合は型エラーが起きる場合があります。
