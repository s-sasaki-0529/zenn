---
title: "Vite は使ってないけど Jest を Vitest に移行する"
emoji: "🌟"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vite", "vitest", "jest"]
published: true
---

# 概要

本記事は、[SmartHR Advent Calendar 2023 シリーズ2](https://qiita.com/advent-calendar/2023/smarthr) の11日目です。

今回は、[React](https://ja.react.dev/) と [Webpack](https://webpack.js.org/) で開発している Web アプリケーションのフロントエンドテストフレームワークを [Jest](https://jestjs.io/ja/) から [Vitest](https://vitest.dev/) に移行した理由と、その具体的な作業内容についてまとめました。

# バージョン情報

- `Vite` v5.0.6
- `Vitest` v1.0.2
- `jest` v29.7.0
- `typescript` v5.3.3
- `webpack` v5.89.0
- `Node.js` v20.10.0
- `yarn` v1.22.19
- `macOS` Ventura 13.5.2

# 移行結果

手っ取り早く最初に移行結果をまとめました。
(正確には移行以外の改善による変化も含むので参考程度にしてください)

|項目|Jest|Vitest|備考|
|----|----|----|-----|
|ローカル起動時間|2秒程度|0.6秒程度|非常に簡素なテストコード一種の実行時間|
|ローカル実行時間|35秒程度|20秒程度|すべてのテスト実行時間|
|CI実行時間|90秒程度|55秒程度|すべてのテストの実行時間|
|依存パッケージ数|6|3|プラグインパッケージ含む|

上記以外で特筆すべき点として、**他の開発者(≒チームメンバー)にとっては、変更の影響をほとんど受けずに、ノーコストで上記恩恵を受けられる**点があります。

これは `Vitest` の `Jest` に対する高い互換性のおかげでテストコードの書き方に大きな変更がなかったことと、テスト実行コマンドを `npm-scripts` によって隠蔽していたことによるもので、移行したことに気づきさえしない可能性もあります。

# `Vite` を使ってないのに `Vitest` 使ってええんか？

今回 `Jest` から `Vitest` への移行を行ったプロジェクトは、開発サーバーやプロダクションビルドには `Webpack` を使用しており、`Vite` は一切使用していませんでした。

そういったプロジェクトにおいても、`Vite` をベースとしたテストフレームワークである `Vitest` は使用して良いものでしょうか？

これについては `Vitest` のドキュメント内にて、以下のように言及されています。

https://vitest.dev/guide/comparisons.html#jest

> Even if your library is not using Vite (for example, if it is built with esbuild or Rollup), Vitest is an interesting option as it gives you a faster run for your unit tests and a jump in DX thanks to the default watch mode using Vite instant Hot Module Reload (HMR). Vitest offers compatibility with most of the Jest API and ecosystem libraries, so in most projects, it should be a drop-in replacement for Jest.

`Vitest` は `Vite` を用いた開発・プロダクションを行っている場合に、共通の設定・エコシステムをテストフレームにも適用できることが強みなのは事実です。

しかし、そうでなく**テストフレームワークのためだけに `Vite` を導入することになっても、上記ドキュメントで言及されている通りの大きなメリットを得られる**と思います。

また、`Vitest` は `dependencies` で `Vite` に依存していることから、 `Vite` を直接インストールする必要もありませんし、設定ファイルを `vite.config.ts` でなく `vitest.config.ts` に記述することもできることから、**`Vitest` 自体も `Vite` の存在をあまり意識せずに使えるように作られている**と言えます。

https://github.com/vitest-dev/vitest/blob/7006bb367494536e2ecf762a5636e509734e43e5/packages/vitest/package.json#L158

そういった背景から、今回は特に悩むこと無く `Vitest` を導入することを決めました。

# なぜ `Jest` だと辛かったのか

`Jest` は既に10年以上開発が続けられている、JavaScript テストフレームワーク界の重鎮にして、デファクトスタンダートにも近い存在です。
https://jestjs.io/ja/

テストランナー、アサーション、モック、カバレッジレポート、スナップショットといった、テストに関わる有用な機能を一通り網羅し、オールインワンかつゼロコンフィグであることを売りとしています。

私自身も過去に、`Jest` 以前の時代の様々なテストツールに辟易し、`Jest` に一本化することで全てを解決できました。

一方で、`Node.js` や `Web`, `TypeScript`, `React` といった、フロントエンド開発の基礎となる技術要素は日々進化を重ねています。

それらの変化に対して、`Jest` 側もサポートの範囲を広げたり、豊富なエコシステムにより解決されたりを繰り返しています。

例えば `TypeScript` を使用するためには [ts-jest](https://github.com/kulshekhar/ts-jest) を別途導入し、コード変換のルールを設定ファイルに記述する必要がありますし、[ESM](https://developer.mozilla.org/ja/docs/Web/JavaScript/Guide/Modules) を `Jest` で動かすのはまだまだ課題がありそうです。

そのような経緯から、**`Jest` を長らく使い続けていると、職人技や歴史的経緯がプロジェクト上に現れてしまい**、何か触れてはいけないもののようになってしまいます。

:::message
もちろん、適切に `Jest` を理解し、最新情報のキャッチアップと追従が出来ている場合には、`Jest` でも大抵のユースケースを満たすことは可能です。

ここではフロントエンドにあまり詳しくない人の視点での複雑さを課題としています。
:::

# なぜ `Vitest` を選んだのか

`Vitest` は以下のような特性を持ちます。(多くは `Vite` の特性でもあります)

- [esbuild](https://esbuild.github.io/) を用いた高速なビルド
- `HMR` のように、コード変更に影響するテストのみを再実行する機能
- `ESM`, `TypeScript`, `JSX` のビルトインサポート
- `Jest` との高い互換性による移行容易性

前述の、`なぜ Jest だと辛かったのか` のポイントを解決するのにも十分であることがわかります。

**そして何より、つい先日、満を持して `Vitest` は v1.0.0 がリリースされました。**

https://github.com/vitest-dev/vitest/releases/tag/v1.0.0

それまでは `Vitest` 自身もマイナーバージョンで頻繁に破壊的変更が入りましたが、v1.0.0 となってしまえば安定するはずなので、このビッグウェーブに乗るしかないでしょう。

# 移行作業

ここからは、`Jest` から `Vitest` への具体的な移行作業と、各所でのハマったポイント、トラブルシューティングを時系列で記載します。

:::message
移行作業はプロジェクトの構成や特性によって大きく変わることがあります。

すべてのプロジェクトでこの手順で解決できたり、同様の問題が発生するわけではありませんのでご注意ください。
:::

## Vitest のインストール

`Vitest` を依存に追加すれば自動で `Vite` もインストールされるので、個別インストールは不要です。

```bash
$ yarn add -D vitest
```

## 設定ファイルを作成

`Vitest` の設定は `Vite` の設定ファイルである `vite.config.ts` に記述することも出来ますが、`Vitest` 単体の場合は `vitest.config.ts` も使用できます。

今回は、`vitest.config.ts` を使用することで、**ファイル名から「`Vitest` は使うが `Vite` を直接使っているわけではない」ということを認知しやすくします。**

```ts:vitest.config.ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    // ここに Vitest 用の設定を書き込んでいく
  },
})
```

ファイル名は違えど、実態は `Vite` の設定ファイルと同じであるため、`Vitest` 用の設定は `test` フィールドに記述します。

## グローバルAPI を有効化する

[globals](https://vitest.dev/config/#globals) は、 `describe` や `test`, `beforeEach` といったテスト用の API を、テストコード内で `import` することなく使えるようにする設定です。

`Jest` ではこれがデフォルトで有効化されていましたが、`Vitest` の場合は有効化が必要です。

```ts:vitest.config.ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    globals: true
  },
})
```

グローバルAPIの多くは `Jest` と互換性を持っているため、これだけで既存のテストコードの多くは動くようになります。

## グローバルAPI の型解決をする

前述の `globals: true` によって、テストコード上で `describe` などのグローバルAPIが利用できるようになりました。

しかし、これだけだとランタイムでグローバルAPIが自動で読み込まれるだけなので、`TypeScript` を使用している場合は型レベルでも自動で読み込む必要があります。

基本的には以下のように `compilerOptions.types` フィールドにグローバルAPIの型ファイルを指定することで、ソースコード上での `import` が無い型でもここから読み込めるようになります。

```json:tsconfig.json
// 一部抜粋
{
  "compilerOptions": {
    "types": ["vitest/globals"]
  }
}
```

が、ここで以下のようなエラーが発生しました。

```bash
node_modules/vite/dist/node/index.d.ts:6:41 - error TS2307: Cannot find module 'rollup/parseAst' or its corresponding type declarations.

6 export { parseAst, parseAstAsync } from 'rollup/parseAst';
```

結論だけ書くと、`tsconfig.json` の `compilerOptions.moduleResolution` が `Node` または `Node10` になっている場合、 `Node16` や `NodeNext` あるいは `Bundler` に設定する必要がありました。

今回は既存コードの都合、`Node16` では不十分であったため、`Bundler` に設定しました。

```json:tsconfig.json
// 一部抜粋
{
  "compilerOptions": {
    "moduleResolution": "bundler",
    "types": ["vitest/globals"]
  }
}
```

結論だけ書きましたが、小難しい調査メモは以下スクラップに記載し、ここでは割愛します。
https://zenn.dev/sa2knight/scraps/636bedb1f9b019


## パスエイリアスを反映させる

ここでいうパスエイリアスは、 `tsconfig.json` における [compilerOptions.paths](https://www.typescriptlang.org/tsconfig#paths) フィールドや、 `webpack.config.js` における　[resolve.alias](https://webpack.js.org/configuration/resolve/#resolvealias) にあたる設定で、 `import` 時のパスに対して読み書きしやすいショートカット用途のエイリアスを付与する機能です。

プロジェクトで使用しているエイリアスを `Vitest` にも読み込ませるためには、`resolve.alias` を設定します。エイリアスはモジュール解決のための `Vite` 側の設定のため、 `test` フィールド内ではないことにご注意ください。

```ts:vitest.config.ts
export default defineConfig({
  resolve: {
    alias: {
      '@/': '/src/client',
    }
  }
  test: {
    // こっちじゃないので注意
  }
})
```

これによって、 `@/hogehoge` というパスが、 `/src/client/hogehoge` と読み替えられます。

しかし、既に `tsconfig.json` 側でも同様の設定をしている場合は DRY にしたいでしょう。そこで、`vite-tsconfig-paths` というプラグインを導入しました。
https://github.com/aleclarson/vite-tsconfig-paths

これをインストールし、`Vite` に読み込ませるだけで、 `tsconfig.json` 内の `compilerOptions.paths` オプションを `vitest.config.ts` の `resolve.alias` に読み込めるようになりました。

```bash
$ yarn add -D vite-tsconfig-paths
```

```ts:vitest.config.ts
export default defineConfig({
  plugins: [tsconfigPaths()],
  test: {
    // こっちじゃないので注意
  }
})
```

## ブラウザAPI を利用できるようにする

あるテストコードを実行する際に以下のエラーが出ました。

```bash
ReferenceError: document is not defined
```

`document` は `window` や `navigator` などに読み替えても構いません。要は `Node.js` 上には定義されていないブラウザAPIに依存したコードを実行した際に生じるエラーです。

Web アプリケーションを開発する上では、ブラウザAPIを利用したコードを書き、そのテストコードも書くのは一般的です。

しかし、テストコードは `Node.js` (などブラウザ以外の実行環境)上で実行するため、これらの API が呼び出せません。

そこで、[jsdom](https://github.com/jsdom/jsdom) のようなブラウザAPI互換を持った仮想環境を用意し、その中でテストコードを実行させることでブラウザAPIもシミュレートできるようにします。

`Jest` 時代は定番の `jsdom` を使用していましたが、今回はさらにパフォーマンスが優れているとされる [happy-dom](https://github.com/capricorn86/happy-dom) を使用します。

```bash
$ yarn add -D happy-dom
```

`Vitest` で `happy-dom` を使用する場合は、`environment` オプションを設定します。

```ts:vitest.config.ts
export default defineConfig({
  test: {
    environment: 'happy-dom',
  },
})
```

これでブラウザAPIを使用したテストコードも実行できるようになりました。

## モック系のAPIを差し替える

`jest.fn()` や、`jest.spyOn` など、`Jest` ネームスペース以下のメソッドを使用している箇所を、機械的に `vi.fn()`, `vi.spyOn` と置換していきます。

多くの API は `Jest` 互換があるため単純な置換だけで解決しますが、一部は以下のように異なる呼び出し方が必要になる場合もあるようです。

```diff
- jest.setTimeout(5_000)
+ vi.setConfig({ testTimeout: 5_000 })
```

## テストのスキップ方法を差し替える

`Jest` では以下のように `xtest` `xdescribe` `xit` を用いてテストのスキップが出来ていました。

```ts
xdescribe('スキップするテストスイート', () => {
  xit('スキップするテスト', () => {
  })
  xtest('スキップするテスト', () => {
  })
})
```

これらは `describe.skip` `it.skip` `xtest.skip` のエイリアスでしたが、`Vitest` ではエイリアスが無くなったようなので、以下のように差し替えます。

```ts
describe.skip('スキップするテストスイート', () => {
  it.skip('スキップするテスト', () => {
  })
  test.skip('スキップするテスト', () => {
  })
})
```

また、以下のように、テストスイートはあるがテスト自体が含まれていない空のブロックがある場合、`Jest` では何も実行されずスルーされていました。

```ts
describe('hogehoge のテスト', () => {
  // TODO テストを実装する
})
```

`Vitest` ではこのようなテストスイートはエラーとなるため、こちらも `describe.skip` などに差し替える必要があります。

```bash
Error: No test found in suite hogehoge のテスト
```

## E2E テストのコードは除外する

今回移行を行ったプロジェクトでは、[Playwright](https://playwright.dev/) で作成している E2E テストのコードも `.spec.ts` の形式で配置されています。

`Vitest` はデフォルトで `['**/*.{test,spec}.?(c|m)[jt]s?(x)']` に合致するファイルを実行してしまうため、E2Eテスト用のコードまで実行対象となってしまいました。

そのため、明示的に E2E テストのディレクトリを除外するように設定します。

```ts:vitest.config.json
export default defineConfig({
  test: {
    exclude: ['client/test/e2e/**/*'],
  },
})
```

しかし、このような設定にすると今度は `node_modules/` 以下にある有象無象のテストコードらしきファイルが実行対象となってしまいました。

どうやら `exclude` オプションは省略した場合に `['**/node_modules/**', '**/dist/**', '**/cypress/**', '**/.{idea,git,cache,output,temp}/**', '**/{karma,rollup,webpack,vite,vitest,jest,ava,babel,nyc,cypress,tsup,build}.config.*']` がデフォルトで設定されるようです。

上記デフォルト設定のおかげで、これまでは `node_modules/` 以下が実行対象から外れていたのに、`exclude` オプションを指定したことで自分で面倒を見る必要が出たようです。

`exclude` オプションに `node_modules` を追記するのも良いですが、他のディレクトリも怪しいため、ここではテスト対象を明示するよう `include` オプションを追加するようにしました。

```ts:vitest.config.json
export default defineConfig({
  test: {
    include: ['client/**/*.test.ts', 'client/**/*.test.tsx'],
    exclude: ['client/test/e2e/**/*'],
  }
})
```

これによって、`client` ディレクトリ内のテストコードを対象とするが、E2E テストのディレクトリは除外にするというシンプルな設定にできました。

## 型チェックを `tsc` にまかせる

`Jest` + `ts-jest` を使用した場合と、`Vitest` を使用した場合の大きな違いとして、前者はテスト内で(デフォルトで)型チェックが行われますが、後者では行われません。

そのため、テストコードを通じて型の整合性を担保していた場合は、`Vitest` とは別に `tsc` などを用いて型チェックを行うプロセスを CI に含める必要があります。

今回のプロジェクトでは、元々 CI 内で `tsc` を実行していましたが、テストコード内の型チェックについては `Jest` にまかせるという構成だったため、`tsc` でテストコードも対象となるように、`tsconfig.json` を修正しました。

```json:tsconfig.json
{
  "compilerOptions": {
    // 以前はテストコードは含まれないように調整されていた
    "include": ["client/**/*"],
  }
}
```

## スナップショットの差分の扱いを考える

`Jest` には[スナップショットテスト](https://jestjs.io/ja/docs/snapshot-testing) という、テスト対象データをシリアライズしてファイルに保存し、次回のテスト実行時に差分が発生していないかを検証する機能があります。

これは、`React` などの UI コンポーネントの描画結果をシリアライズし、HTML/CSS レベルでの細かい差分の発生を検知してデグレを防止できる強力な機能です。

```ts
  test('<TheComponent> の描画結果が変わっていないこと', () => {
    const component = renderer.create(<TheComponent />)
    const tree = component.toJSON()
    expect(tree).toMatchSnapshot()
  })
```

この機能についても、`Vitest` は `Jest` に対する互換性を持っており、上記コードは `Jest` `Vitest` いずれでも動作します。

しかし、コンポーネントを描画するプロセスと、それをシリアライズするプロセスに細かな差異があるため、どうしてもスナップショットの差分が発生することは避けられません。

[Difference from Jest](https://vitest.dev/guide/snapshot.html#difference-from-jest) でも触れられているように、シリアライザのオプションを `Jest` と揃えることで、差分を抑えることができます。

```ts:vitest.config.ts
export default defineConfig({
  test: {
    snapshotFormat: {
      printBasicPrototype: true,
    },
  },
})
```

多少は緩和しましたが、それでも差分をゼロにすることは困難で、その後試行錯誤を重ねた結果、今回は移行時のスナップショット差分については全て受け入れることにしました。

**スナップショットテストにおいて重要なのは、差分を出さないことでなく、アプリケーションの変更を検知することです。テストフレームワークの移行で発生する差分を苦労して抑える必要はないでしょう。**

ただし、React コンポーネントのスナップショットが膨大であることから、スナップショットを更新するプルリクエストは、`Vitest` 移行のプルリクエストと分けることにしました。

![](https://storage.googleapis.com/zenn-user-upload/de68ef3943ff-20231209.png)
*PR1: Vitest 移行を行うだけで、スナップショットの変更は含まない*

![](https://storage.googleapis.com/zenn-user-upload/30ed456751e6-20231209.png)
*PR2: スナップショットの変更のみを行う*

一つ目のプルリクエストでは、スナップショットテストの差分が発生していても CI が落ちないように、以下のコマンドでテストを実行しています。

```bash
yarn vitest --update # vitest 移行直後だけ、PRをシンプルにするために一時的にスナップショット差分を無視する
```

この場合、差分があってもテストは成功扱いとなるので、二つ目のプルリクエストにてスナップショットを更新しつつ、CI で実行するコマンドも以下に戻します。

```bash
yarn vitest
```

これによって、スナップショットの差分を一瞬受け入れつつも、移行のプルリクエストをチームにレビューしてもらいやすくしました。

# まとめ

本記事では、`Vite` を使用していないプロジェクトにおける、`Jest` から `Vitest` への移行に関する意思決定理由と、その具体的な移行作業内容をまとめました。

移行手順は一見すると非常に多くの作業をしているように見えますが、実際は数時間内の作業で終わり、パフォーマンス計測や細かい設定の調整、それにプルリクエストの作成も含めても1日程度で完了しています。

これもひとえに、`Vitest` が `Jest` に対して高い互換性を持っており、`Jest` からの移行を強くサポートしているおかげと思います。

比較的低コストで移行が完了したにも関わらず、得られた恩恵は大きいなと個人的にも感じております。本記事がどこかのプロジェクトの意思決定を後押しするきっかけになれば幸いです。