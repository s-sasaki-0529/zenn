---
title: "はじめに"
free: true
---

本書は [Storybook](https://storybook.js.org/) を用いて、[Vue.js](https://jp.vuejs.org/index.html), [TypeScript](https://www.typescriptlang.org/), [Vite](https://ja.vitejs.dev/) の技術スタックにおける Web アプリケーションのコンポーネントカタログ化を行うためのハンズオンとなっています。

**ハンズオンでは、公式が提供するスキャフォルドツールを使用せず、必要なパッケージの導入や設定ファイルの作成を段階的に行うことで、曖昧な理解で済ませないよう心がけています。**

# バージョン情報

|パッケージ|バージョン|備考|
|----|----|----|
|`Storybook`|7.0.2|モノレポ内の各パッケージ共通|
|`vue`|3.2.47||
|`vite`|4.2.1||
|`@vitejs/plugin-vue`|4.1.0||
|`typescript`|4.9.5||

また、 [`Node.js`](https://nodejs.org/ja/) は v16.16.0 を、 [`yarn`](https://yarnpkg.com/) は 1.22.19 を使用します。

上記含め、Mac(Monterey) 上での動作を確認していますが、この辺りは環境に多少の差異があっても問題はないと思われます。

なお、旧バージョンである `Storybook 6` と `Vue 2` をご利用の方向けに、別途以下の本を公開しています。使用するバージョンに合わせて本書か以下いずれかを参照ください。

https://zenn.dev/sa2knight/books/aca5d5e021dd10262bb9

# 取扱範囲

基本的には公式ドキュメントをベースに、広く浅く取り扱います。

なお、**Storybook 6 からのマイグレーションについては取り扱っていません。** 必要に応じて公式の[マイグレーションガイド](https://storybook.js.org/docs/react/migration-guide#page-top)などをご利用ください。

また、`React` など他の技術スタックを使っている方でも、`Storybook 7` 自体に関しては参考になるようには努めています。

# 想定読者

## Storybook 自体に不慣れな方

`Storybook` 自体の紹介を含みますので、頭から順にハンズオンすることをオススメします。

## Storybook には慣れているが、雰囲気で使っている方

`Storybook` v7 ではアーキテクチャ変更が行われていることから、セットアップ手順やベストプラクティスに影響が出ており、これまでの `Storybook` からは異なっている点があります。

また、スキャフォルドツールなしでセットアップ、拡張してく構成のため、改めて `Storybook` というものを理解するためにもハンズオンすることをオススメします。

## Storybook を適切に活用されている方

本書は基礎的な内容に閉じている都合、目新しい情報を得られる可能性は低いです。気になる章のみ拾って読むことをオススメします。

# 前提知識

アプリケーションを構成する技術スタック(`Vue 3`, `TypeScript`, `Vite`) に関する基礎的な説明は省略しています。

とはいえ本書は `Storybook` が主題であるため、それぞれのコード、設定については読み流しても大丈夫です。