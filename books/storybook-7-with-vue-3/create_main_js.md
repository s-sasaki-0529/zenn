---
title: 設定ファイルの作成
free: true
---

`Storybook` では、 `.storybook` ディレクトリにベースとなる設定ファイルを配置します。

設定ファイルは `main.js` `config.js` (あるいは `.ts`) のいずれかであれば良いので、本書では `.storybook/main.ts` を作成します。

```ts:.storybook/main.ts
import { StorybookConfig } from "@storybook/vue3-vite";

const config: StorybookConfig = {
  framework: "@storybook/vue3-vite",
  stories: ["../src/**/*.stories.@(js|ts)"],
};

export default config;
```

`framework` では使用するフレームワークに対応するパッケージ名を指定するので、前章でインストールした `@storybook/vue3-vite` を指定します。

`stories` には、 `Storybook` で描画するストーリーファイル(`.stories.ts` など) のパスを指定します。今回は `src/stories` ディレクトリを作成するため、このような設定にします。

これで最低限の設定ファイルは完成です。 CLI を使って、 `Storybook` を起動してみましょう。

```bash
$ yarn storybook dev --port 6006

@storybook/cli v7.0.0-beta.20

info => Starting manager..
╭───────────────────────────────────────────────────╮
│                                                   │
│   Storybook 7.0.2 for vue3-vite started           │
│   22 ms for manager and 692 ms for preview        │
│                                                   │
│    Local:            http://localhost:6006/       │
│    On your network:  http://192.168.0.28:6006/    │
│                                                   │
╰───────────────────────────────────────────────────╯
```

http://localhost:6006 にアクセスし、以下のような画面が表示されていれば成功です。
まだストーリーを定義していないのでエラーは発生していますが、 `Storybook` の起動まで 22ms という高速さは既に実感できます。

:::message
当然、起動時間はお使いの環境などで上下します。
:::

![](https://storage.googleapis.com/zenn-user-upload/23ae47827b98-20221224.png)