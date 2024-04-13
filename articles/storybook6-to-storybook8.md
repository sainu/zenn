---
title: "Storybook 6をStorybook 8ヘ移行した時にやったこと"
emoji: "🚢"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["storybook"]
published: true
---
以前、マネーフォワード クラウドの製品にマイクロフロントエンドを実装した記事を書きました。

https://zenn.dev/moneyforward/articles/2022-12-14-micro-frontends-in-moneyforward

現在このマイクロフロントエンドは実際に本番で稼働しており、開発も継続的に行ってます。ただ、Storybookのバージョンは当初の v6 のままだったので、Storybook 7の便利な機能が使えないストレスを抱えていました。また、開発が進むにつれコンポーネントやストーリーの数も増えてきて、ビルド時間も次第に長くなってきていました。

ちょうど[先月Storybook 8がリリース](https://storybook.js.org/blog/storybook-8/)されたので、この機会にStorybook 6からStorybook 8への移行を行いました。この記事は、私が実際に行ったStorybook 6からStorybook 8へ移行作業をした時に、社内向けに記録し共有した文書を公開用に少しアレンジしたものです。

# 環境

* Storybook "6.5.16"
* TypeScript "5.4.3"
* Webpack "5.1.4"
* Babel "7.21.3"

# 参考

## Storybook 8の移行ガイド

まずはこのページを読むと良いと思います。Storybook 8の目玉の変更や破壊的変更を要約してくれてます。また、自動で移行する方法と注意点についても丁寧に記載されています。

https://storybook.js.org/docs/migration-guide/from-older-version

## リリースノート

ざっくりと各バージョンでのリリース内容を確認したい場合は、以下のページを参照すると良いと思います。前述のStorybook 8の移行ガイドには記載れていない粒度の変更点の情報が得られます。

https://storybook.js.org/releases

## 詳細な移行ガイド

各メジャーバージョン間のコードレベルでの詳細な変更点とその移行方法が記載されています。基本的に移行作業中時に困ったらこのページで検索をかけると解決策が見つかると思います。

今回該当するのが、以下の見出しです。

* [From version 6.5.x to 7.0.0](https://github.com/storybookjs/storybook/blob/next/MIGRATION.md#from-version-65x-to-700)
* [From version 7.x to 8.0.0](https://github.com/storybookjs/storybook/blob/next/MIGRATION.md#from-version-7x-to-800)

# Storybook 6からStorybook 8への移行

大きな流れとして、以下の手順で作業を行いました。

1. 自動移行ツールと手動移行
2. Storybookを起動してログのエラーと警告を解消
3. TypeScriptのコンパイルエラーを解消
4. Jestのテストを通す
5. VRTを通す

## 自動移行と手動移行

まずは、Storybook 8の移行ガイドに記載されている自動移行ツールを使ってみました。このツールは、依存パッケージの更新やStorybookの設定ファイルや各種設定ファイルを自動で変換してくれるツールです。このツールを使っても全ての変更が自動で行われるわけではないので、手動での修正も必要です。

:::message
実際の環境によって、自動移行ツールに指示される内容が異なる場合があります。
:::

まず、以下のコマンドで自動移行ツールを実行します。変更内容毎にインタラクティブに確認されるので、その単位でコミットしていくと良いと思います。

```
npx storybook@latest upgrade
```

### 依存パッケージの更新

自動移行ツールを実行すると、依存パッケージのバージョンが更新されます。私の環境では以下のパッケージのバージョンが更新されました。

```diff:package.json
-    "@storybook/addon-a11y": "^6.5.16",
-    "@storybook/addon-actions": "^6.5.14",
+    "@storybook/addon-a11y": "^8.0.6",
+    "@storybook/addon-actions": "^8.0.6",
-    "@storybook/addon-docs": "^6.5.16",
-    "@storybook/addon-essentials": "^6.5.16",
-    "@storybook/addon-interactions": "^7.0.20",
-    "@storybook/addon-links": "^7.0.2",
+    "@storybook/addon-docs": "^8.0.6",
+    "@storybook/addon-essentials": "^8.0.6",
+    "@storybook/addon-interactions": "^8.0.6",
+    "@storybook/addon-links": "^8.0.6",
-    "@storybook/builder-webpack5": "^6.5.16",
+    "@storybook/builder-webpack5": "^8.0.6",
-    "@storybook/react": "^6.5.16",
+    "@storybook/react": "^8.0.6",
```

### frameworksの移行

以下のメッセージが表示されたら、`frameworks`の移行が必要です。詳細は、[New Framework API](https://github.com/storybookjs/storybook/blob/next/MIGRATION.md#new-framework-api)を参照してください。

```
🔎 found a 'new-frameworks' migration:
```

以下のように、移行したいか質問されるので、`yes`と回答します。

```
✔ Do you want to run the 'new-frameworks' migration on your project? … yes
```

`main.ts`と`package.json`に以下のような変更が加えられました。

```diff:main.ts
-  framework: '@storybook/react',
-  core: {
-    builder: '@storybook/builder-webpack5',
+  framework: {
+    name: '@storybook/react-webpack5',
+    options: {}
```

```diff:package.json
-    "@storybook/builder-webpack5": "^8.0.6",
-    "@storybook/manager-webpack5": "^6.5.10",
+    "@storybook/react-webpack5": "^8.0.6",
```

### Storybook実行コマンドの変更

以下のメッセージが表示されたら、`storybook`バイナリの移行が必要です。詳細は、[start-storybook / build-storybook binaries removed](https://github.com/storybookjs/storybook/blob/next/MIGRATION.md#start-storybook--build-storybook-binaries-removed)を参照してください。

```
🔎 found a 'storybook-binary' migration:
```

以下のように、移行したいか質問されるので、`yes`と回答します。

```
✔ Do you want to run the 'storybook-binary' migration on your project? … yes
```

`package.json`に以下のような変更が加えられました。

```diff:package.json
+    "storybook": "^8.0.6",
```

続いて、以下のメッセージが表示されました。詳細は、[start-storybook / build-storybook binaries removed](https://github.com/storybookjs/storybook/blob/next/MIGRATION.md#start-storybook--build-storybook-binaries-removed)を参照してください。

```
🔎 found a 'sb-scripts' migration:
```

以下のように、移行したいか質問されるので、`yes`と回答します。

```
✔ Do you want to run the 'sb-scripts' migration on your project? … yes
```

scriptsの設定が変更されました。

```diff:package.json
-    "storybook": "start-storybook -p 6006",
-    "build-storybook": "build-storybook",
+    "storybook": "storybook dev -p 6006",
+    "build-storybook": "storybook build",
```

### JestとTesting Libraryの間接パッケージが廃止

以下のメッセージが表示されたら、Storybook向けのJestパッケージの移行が必要です。

```
🔎 found a 'remove-jest-testing-library' migration:
```

以下のように、移行したいか質問されるので、`yes`と回答します。また、`stories`のファイルパスを入力します。

```
✔ Do you want to run the 'remove-jest-testing-library' migration on your project? … yes
✔ Please enter the glob for your stories to migrate to @storybook/test … ./src/**/*.stories.(ts|tsx)
```

`package.json`に以下のような変更が加えられました。`@storybook/jest`と`@storybook/testing-library`が削除され、`@storybook/test`が追加されました。

```diff:package.json
+    "@storybook/test": "^8.0.6",
-    "@storybook/jest": "^0.0.10",
-    "@storybook/testing-library": "^0.0.13",
```

また、指定したstoriesのファイルのパッケージ参照先が自動で置換されました。

```diff:/path/to/*.stories.tsx
-import { within, userEvent, waitFor } from '@storybook/testing-library';
+import { within, userEvent, waitFor } from '@storybook/test';
```

### argTypesRegexが廃止

以下のメッセージが表示されたら、`argTypesRegex`の削除が必要です。

```
🔎 found a 'remove-argtypes-regex' migration:
```

`argTypesRegex`が廃止され、`onClick`のようなPropをStorybookが自動でモックすることがなくなりました。その代わりに、明示的に`fn()`を使ってモックとスパイを行うようになりました。詳細は、[Via @storybook/test fn spy function](https://storybook.js.org/docs/essentials/actions#via-storybooktest-fn-spy-function)を参照してください。

以下のコマンドを実行して、`play`関数内でmockを使用している箇所をチェックできます。私の環境では、`play`関数内でモックを使用していなかったので、何も検出されませんでした。

```
npx storybook migrate find-implicit-spies --glob="**/*.stories.@(js|jsx|ts|tsx)"
```

また、`preview.js`の`argTypesRegex`を手動で削除する必要があります。

```diff:.storybook/preview.js
 export const parameters = {
-  actions: { argTypesRegex: '^on[A-Z].*' },
   controls: {
```

### `.stories.mdx`の廃止

以下のメッセージが表示されたら、`.stories.mdx`のファイルを`.stories.tsx`と`mdx`に分割する必要があります。

```
🔎 found a 'mdx-to-csf' migration:
```

これまでは、`.stories.mdx`拡張子のファイルでストーリーの定義とドキュメントの作成をハイブリッドに出来ましたが、Storybook 8では廃止されました。代わりに、`.stories.tsx`と`.mdx`に分割し、ストーリーの定義とドキュメントの作成を分離するようになりました。詳細は、[Dropping support for *.stories.mdx (CSF in MDX) format and MDX1 support](https://github.com/storybookjs/storybook/blob/next/MIGRATION.md#dropping-support-for-storiesmdx-csf-in-mdx-format-and-mdx1-support)を参照してください。

私のプロジェクトでは、`.stories.mdx`ファイルが存在していたものの、メンテナンスされていなかったので、これを機に削除しました。

以下のように、移行したいか質問されるので、`yes`と回答します。

```
✔ Do you want to run the 'mdx-to-csf' migration on your project? … yes
```

私の場合は、`.storybook/main.js`の設定が書き換えられました。

```diff:.storybook/main.js
-  stories: ['../src/**/*.stories.mdx', '../src/**/*.stories.@(js|jsx|ts|tsx)'],
+  stories: ['../src/**/*.mdx', '../src/**/*.stories.@(js|jsx|ts|tsx)'],
```

もし、`.stories.mdx`ファイルを利用している場合は、手動による移行が必要になるかもしれません。

### Autodocsの自動生成が廃止へ

Storybook 7以降は、Storybookをドキュメント表示モードで表示するのではなく、サイドバーの項目に表示されるようになりました。また、Storybook 6までは、`autodocs`が`true`に設定されていましたが、Storybook 7以降では`"tag"`が設定されるようになりました。

もし、Docsを表示したい場合は、metaオブジェクトに`tags: ['autodocs']`を追加する必要があります。

```ts
export default {
  component: MyComponent
  // Tags are a new feature coming in 7.1, that we are using to drive this behaviour.
  tags: ['autodocs']
}
```

私のプロジェクトでは、Docsページを運用していなかったので、新しいデフォルト値である`"tag"`をそのまま利用し、Docsを必要に応じて追加することにしました。

### "@storybook/addon-queryparams"のアップデート

Storybook 8以降は、`@storybook/addon-queryparams@6.2.9`が互換性がないため、`@storybook/addon-queryparams@^7`移行にアップデートする必要があります。

以下のコマンドを実行して、最新化すれば良いです。

```
npm install @storybook/addon-queryparams@latest
```

```diff:package.json
-    "@storybook/addon-queryparams": "^6.2.9",
+    "@storybook/addon-queryparams": "^7.0.1",
```

以上で、自動移行ツールによる移行作業は完了です。次に、Storybookを起動してログのエラーと警告を解消します。

## Storybookを起動してログのエラーと警告を解消

`npm run storybook`でStorybookを起動して、ログに表示されるエラーや警告を解消します。以下のコマンドでStorybookを起動します。

### @storybook/client-api と @storybook/store が見つからないエラー

以下のようなエラーが表示された場合は、モジュールが削除されたためです。それらのモジュールは、`@storybook/preview-api`に統合されました。詳細は、[Removed deprecated shim packages](https://github.com/storybookjs/storybook/blob/next/MIGRATION.md#removed-deprecated-shim-packages)を参照してください。

```
Module not found: Error: Can't resolve '@storybook/client-api'
```

```
Module not found: Error: Can't resolve '@storybook/store'
```

以下のように、モジュールのインポート元を一括変換すれば解消できます。

```diff:package.json
-import { useArgs } from '@storybook/client-api';
+import { useArgs } from '@storybook/preview-api';
```

```diff:package.json
-import { useArgs } from '@storybook/store';
+import { useArgs } from '@storybook/preview-api';
```

### withQuery が見つからない警告

以下のような警告が表示された場合は、`@storybook/addon-queryparams`のバージョンアップデートによって、`withQuery`がエクスポートされなくなったためです。詳細は、[withQuery decorator removed](https://github.com/storybookjs/addon-queryparams/blob/main/MIGRATION.md#withquery-decorator-removed)を参照してください。

```
WARN export 'withQuery' (imported as 'withQuery') was not found in '@storybook/addon-queryparams' (possible exports: default)
```

ドキュメントに書かれている通り、以下のように`withQuery`を削除すれば解消できます。

```diff
  import React from "react";
  import { Button } from "../Button";
- import { withQuery } from "@storybook/addon-queryparams";

  export default {
    title: "Button",
    component: Button,
-   decorators: [withQuery],
    parameters: {
      query: {
        greeting: "Hello world!",
      },
    },
  };
```

以上で、すべてのエラーや警告を解消しました。次に、TypeScriptのコンパイルエラーを解消します。

## TypeScriptのコンパイルエラーを解消

tscコマンドでコンパイルチェックをしたところ、大量のエラーが表示されました。

```
npx tsc

(...)

Found 1674 errors in 256 files.

Errors  Files
(...)
```

地道に一つずつエラーを倒していきます...（遠い目）

### 型名変更

[Storybook 7でいくつか非推奨になっていたTypeScriptの型名](https://github.com/storybookjs/storybook/blob/next/MIGRATION.md#componentstory-componentstoryobj-componentstoryfn-and-componentmeta-types-are-deprecated)が[Storybook 8で完全に廃止](https://github.com/storybookjs/storybook/blob/next/MIGRATION.md#decoratorfn-story-componentstory-componentstoryobj-componentstoryfn-and-componentmeta-typescript-types)されました。

変更点は以下の通りです。

```
ComponentStory -> StoryFn (CSF2) or StoryObj (CSF3)
ComponentStoryObj -> StoryObj
ComponentStoryFn -> StoryFn
ComponentMeta -> Meta
```

一括置換します。

### @storybook/testing-react が @storybook/react へ昇進

`@storybook/testing-react`がファーストクラスのStorybook機能に昇格したので、`@storybook/testing-react`ではなく`@storybook/react`から`composeStories`をインポートする必要があります。詳しくは[@storybook/testing-reactのREADME](https://github.com/storybookjs/testing-react?tab=readme-ov-file#%EF%B8%8F-attention)を参照してください。

以下のように、インポート元を変更するだけでした。

```diff
-import { composeStories } from '@storybook/testing-react';
+import { composeStories } from '@storybook/react';
```

### `composeStories`が返すコンポーネントの`play`関数の型が変更

`composeStories`が返すコンポーネントの`play`関数の型が変更され、オプショナルになりました。詳細は、[Type change in composeStories API
](https://github.com/storybookjs/storybook/blob/next/MIGRATION.md#type-change-in-composestories-api)を参照してください。

以下のように、2つの方法で対応できます。私は、まず移行を完了することを優先して、オプショナルチェイニングを使いました。

```
const { Primary } = composeStories(stories)

// before
await Primary.play(...)

// after
await Primary.play?.(...) // if you don't care whether the play function exists
await Primary.play!(...) // if you want a runtime error when the play function does not exist
```

他にも、細かなタイプエラーがありましたが一つずつ地道に修正しすべての型エラーを解消しました。

## Jestのテストを通す

Jestのテストを実行して、テストが通るか確認します。

### setGlobalConfig 廃止

`setGlobalConfig`関数は非推奨となり、Storybook 7.0の命名法に沿った`setProjectAnnotations`に変更されました。詳細は[v2.0.0](https://github.com/storybookjs/testing-react/releases/tag/v2.0.0)を参照してください。

以下のように、`setGlobalConfig`を`setProjectAnnotations`に変更すれば解消できます。

```diff:.jest/setupTests.js
-import { setGlobalConfig } from '@storybook/react';
+import { setProjectAnnotations } from '@storybook/react';

-setGlobalConfig(globalStorybookConfig);
+setProjectAnnotations(globalStorybookConfig);
```

以上で、Jestのテストが通るようになりました。次に、VRTを通します。

## VRTを通す

最後は、VRTでUIの変更がないか確認します。

私の場合、`argTypes`でデフォルト値を利用していたストーリーのUIが壊れてしまっていました。`argTypes.defaultValue`ではなく`args`を利用するように修正します。詳細は、[No longer inferring default values of args](https://github.com/storybookjs/storybook/blob/next/MIGRATION.md#no-longer-inferring-default-values-of-args)を参照してください。

```diff
   argTypes: {
     color: {
-      defaultValue: 'error',
     },
     children: {
-      defaultValue: 'メッセージ',
     },
   },
+  args: {
+    color: 'error',
+    children: 'メッセージ',
+  },
```

### userEventの関数呼び出し時にawaitを利用する

`@storybook/test`の`userEvent`の各ユーザーインタラクション関数は`Promise`を返すので`await`で待ち構える必要がありますが、一部`await`が抜けている箇所がありました。`@storybook/testing-library`から`@storybook/test`への移行によって変わったものかもしれませんが、詳しくは特定できませんでした。

`await`と必要に応じて、`play`関数を`async`で呼び出すように修正します。

```diff
import { userEvent } from '@storybook/test';

  play: async () => {
-    userEvent.type(canvas.getByRole('textbox'), '山田');
-    userEvent.keyboard('{enter}');
+    await userEvent.type(canvas.getByRole('textbox'), '山田');
+    await userEvent.keyboard('{enter}');
   },
 };
 ```

これで、VRTも通るようになりました。

# まとめ

以上で、Storybook 6からStorybook 8への移行作業が完了しました。自動移行ツールを使っても全ての変更が自動で行われるわけではないので、手動での修正も必要です。また、移行作業中には、リリースノートや移行ガイドを参照して変更点を理解しながらエラーや警告を解消していくことが重要です。

もしチーム開発をしている場合は、移行作業の概要と変更点をまとめた文書を作成し、1時間程度の時間を確保しそれらを対面で話すと多くの背景理解をレビュワーに強いる必要がなくなるので、PRのレビューもスムーズに進んでおすすめです。
