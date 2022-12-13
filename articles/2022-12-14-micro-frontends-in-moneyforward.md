---
title: "マイクロサービスのその先へ。マネーフォワードのビジネスを加速するマイクロフロントエンドという選択"
emoji: "🥑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["マイクロフロントエンド", "マイクロサービス", "フロントエンド", "アーキテクチャ", "アドベントカレンダー"]
published: true
published_at: 2022-12-14 09:00
---

この記事は、[マネフォアドベントカレンダー2022](https://adventar.org/calendars/7397) 14日目の投稿です。
13日目は [廣瀬](廣瀬) さんで「[チーム運営の仮説検証サイクルを高速化させるために、新卒スクラムマスターが取り組んだこと](https://moneyforward.com/engineers_blog/2022/12/13/shinsotsu-scrum-master/)」でした。
本日は 岐阜在住で名古屋開発拠点の[sainu](https://github.com/sainu) が「マネーフォワードで実践したマイクロフロントエンド」について書きたいと思います。

# はじめに

マイクロフロントエンドは、2022年10月に[Building Micro-Frontends](https://www.buildingmicrofrontends.com/)（Luca Mezzalira 著）の[翻訳版](https://www.oreilly.co.jp/books/9784814400027/)がオライリーから出版されて少し話題になってました。[ThroughWorks Technology Radar](https://www.thoughtworks.com/radar/techniques/micro-frontends)には2016年から登場して、2019年にAdoptになっています。

私は今年の年初にマイクロフロントエンドという言葉を初めて知りました。名古屋拠点で開発している[マネーフォワードクラウド](https://biz.moneyforward.com/)の業務基盤システムでマイクロフロントエンドを実践し、リリースできる状態まで形にすることができました。この記事では、マイクロフロントエンドに至った背景と実践する過程で考えたことの中から「アーキテクチャ全体図」「マイクロフロントエンドの統合」「配信インフラとバージョニング」「認証」について書こうと思います。

また、この業務基盤システムはこの執筆タイミングではまだリリースしてません。なので、「運用してみてどうだったか？」という話は書いていません。

:::message
リリース前のため、本システムの詳細については伏せて書いてます。そのため、多少読み難くなっているかもしれません。
:::

# マイクロフロントエンドとは

マイクロフロントエンドは、マイクロサービスの考え方をフロントエンドまで拡張した設計パターンです。ビジネスの拡大に伴って複雑で肥大化したアプリケーションを、バックエンド領域ではマイクロサービスで解決してきました。しかし、フロントエンドは一つの大きなアプリケーションのままで、フロントエンドのリリースがボトルネックになっていました。

![](https://storage.googleapis.com/zenn-user-upload/ea95db72c707-20221206.png)

> 出典: [micro-frontends.org](https://micro-frontends.org/)

マイクロフロントエンドは、これまでのバックエンド領域を分割したマイクロサービスチームの関心事をユーザーインターフェースまで拡張します。このチームは、あるドメイン領域に関して高い専門性を有したクロスファンクショナルチームになり、エンドツーエンドでユーザーへ価値提供します。これによって、開発サイクルを早め、肥大化したフロントエンドがボトルネックになるのを解消することができると言われています。

![](https://storage.googleapis.com/zenn-user-upload/c1d4f8f2847b-20221206.png)

> 出典: [micro-frontends.org](https://micro-frontends.org/)

本記事では、実践的な内容にフォーカスしたいので、マイクロフロントエンドの解説は、「[フロントエンドエンジニアは Micro Frontends の夢を見るか](https://engineering.mercari.com/blog/entry/2018-12-06-162827/)」や「[Micro Frontends の理論と実践 -価値提供を高速化する真のマイクロサービスのあり方- / The Theory and Practice of Micro Frontends](https://speakerdeck.com/nobuhikosawai/the-theory-and-practice-of-micro-frontends)」が分かりやすくまとめられているので、こちらを参照していただけると助かります🙏

# マネーフォワードの事業

まず、私が今回関わったシステムの特性を知ってもらうために、軽くマネーフォワードの事業に触れます。

私が開発している業務基盤システムは、[マネーフォワードクラウド](https://biz.moneyforward.com/)が課題解決するバックオフィス業務全般に広く関わる機能を提供するマイクロサービスで、ユーザーが各サービスで業務を行う際に業務領域を問わずに使われる機能を提供します。

ちなみに、マネーフォワードクラウドを含む法人向け事業（Businessドメイン）は、マネーフォワードの売上の過半数を占め、最も高い成長率で拡大している事業です。

![事業ドメイン毎の売上推移](https://storage.googleapis.com/zenn-user-upload/b461c11de592-20221206.png)
> 出典: [2022年11月期 第3四半期決算説明資料](https://ssl4.eir-parts.net/doc/3994/ir_material_for_fiscal_ym/123523/00.pdf)

このBusinessドメインの中核であるマネーフォワードクラウドは、毎年、数サービスが新しくリリースされ続けています。

![](https://storage.googleapis.com/zenn-user-upload/41ae4b19101d-20221207.png)

中堅企業向けのサービスラインナップだけでもこれだけあり、個人事業主や中小企業、士業向けのサービスも含めるとさらに増えます。

> 出典: [2022年11月期 第3四半期決算説明資料](https://ssl4.eir-parts.net/doc/3994/ir_material_for_fiscal_ym/123523/00.pdf)

また、これらのサービスは、2022年度上半期には167件もの新機能・サービス向上がリリースされています。

![](https://storage.googleapis.com/zenn-user-upload/73fa6355a14e-20221207.png)

> 出典: [2022年11月期 第3四半期決算説明資料](https://ssl4.eir-parts.net/doc/3994/ir_material_for_fiscal_ym/123523/00.pdf)

# 開発体制

この規模でもこれだけ高速にリリースし続けることができているのは、マネーフォワードの開発体制やアーキテクチャに理由があります。

開発チームは数人〜十数人で形成されています。また、それぞれ独立したアプリケーションコードやコード管理されたインフラ、デプロイパイプラインを持っているので、開発からリリースまでが高速回転します。

典型的なサービスの開発体制はこのようになっています。

![](https://storage.googleapis.com/zenn-user-upload/09440bfd1b4d-20221213.jpg)


# 課題

このようにサービスが独立して機能開発を進めていくと、同じ機能を別々のサービスが開発することが発生します。これまでは、各サービスを利用するユーザーのニーズを満たす最小機能をチームに閉じて素早くリリースすることで、個々のサービスが最短で成長してきました。また、マネーフォワードクラウドの1サービスのみを利用するユーザーが多かったので、サービス毎に独自で同じ機能を開発しても大きな問題にはなっていませんでした。

![](https://storage.googleapis.com/zenn-user-upload/28b5aa4743f9-20221213.jpg)

しかし、ビジネスがさらに成長していき、あるユーザーが複数のサービスを併用するようになると、サービス毎に管理されたデータを二重でメンテナンスする問題が大きくなってきました。また、同じ機能でもサービス毎に作りやユーザーインターフェースが違うことで併用時のユーザー体験もよくありませんでした。

![](https://storage.googleapis.com/zenn-user-upload/bff8291659db-20221213.jpg)

この構成だと開発の観点でも問題があります。新たに同様の機能をイチから作る工数が新規サービスや既存サービスの数だけ発生したり、既にこの機能を備えているサービスでもユーザーの要求に応じるための機能拡張が必要になります。このような業務基盤に対してのユーザーの要求は、どのサービスへのものも似通ったものになることが想定されていたので、マネーフォワードクラウドのサービス全体でユーザーへ価値を届けるスピードを加速させるためにも、更なる車輪の再発明を止める必要がありました。

# マイクロフロントエンドに至るまで

私たちのチームはこのような課題に立ち向かうためにマイクロフロントエンドの構成を選んだのですが、どのようにしてその結論に至ったのかを整理します。

まず、ユーザーの二重メンテナンスの課題を解決するために、各サービスで管理されるデータベースを一箇所で管理します。サービスは抽象化されたAPIによる同期あるいはメッセージキューを使った非同期コミュニケーションでそれらのデータを読み書きできるようにし、バックエンド領域でマイクロサービスに分割する方法を取ります。

![](https://storage.googleapis.com/zenn-user-upload/ee66c7b988ce-20221213.jpg)

バックエンドのロジックがマイクロサービスとしてサービスから分離されることで、サービスがこの業務に関するビジネスロジック等を実装するコストを削減することができます。ただ、バックエンドを切り出しただけなので、ユーザーインターフェースの実装はまだサービスに残ります。

次に、ユーザー体験とユーザーインターフェースの実装コストの問題を考えます。

ユーザー体験がサービス毎に揃っていないことについては、見た目の問題だけでなく、根本的なデータ構造の違いから影響されるデータの見せ方や扱い方が全く異なるという点が本質的な課題になっていました。データ構造はバックエンドのマイクロサービス化によって統一されるので、ユーザー体験を統一するために、ユーザーインターフェースの仕様をマイクロサービスチームが策定して、サービスチームに実装してもらうというアプローチもできることはできると思います。

![](https://storage.googleapis.com/zenn-user-upload/b6ca1a457f9e-20221213.jpg)

しかし、この方針では以下の問題があります。

- 今回扱うドメインは、米国ではこの領域だけで上場するくらいそれ単体を切り取っても複雑で難しい領域なので、UIもそれ相応の複雑度を持ちます。そのため、全てのサービスがそのUIのドメインを理解し、実装するコストは決して低くありません。
- この機能はかなりの頻度で機能アップデートを予定しているため、一度実装した後も、機能追加の度にサービス開発チームとのコミュニケーションやサービス開発チームの実装コストが発生し続けます。
- サービス開発チームにもスケジュールがあるので、サービスによってリリースのバラツキが発生します。

なので、マイクロサービスとしてバックエンドのAPIだけ提供して、UIをサービスチームに任せるのは運用まで見据えると提供価値よりも運用コストの方が大きくなることが想定されました。なので、マイクロサービスチームがUIの実装およびデプロイまでエンドツーエンドで提供できることが必要だろうと考えました。

マイクロサービスチームがUIまで提供するアプローチとして、新しくドメインを切って、別のページとして切り出す方法があります。例えば、`domain-A.moneyforward.com`にマイクロサービスチームが機能をホストし、サービスはユーザーをこの画面にSSOさせます。この方法なら、サービスはUIの実装を手放すことができ、マイクロサービスチームはリアルタイムにエンドユーザーに価値提供することができます。幸い、マネーフォワードクラウドには既にSSO基盤があるので、新しく大きな実装を必要とせずに実現できそうでした。

![](https://storage.googleapis.com/zenn-user-upload/2775c5803875-20221213.jpg)

しかし、この方法では、ユーザーにシームレスな体験を提供することができません。

元々の1サービスで完結していた時は、当然サービス内でシームレスな体験を提供することができていました。

![シームレスな体験](https://storage.googleapis.com/zenn-user-upload/2ea43f5ef514-20221211.jpg)

しかし、この方法は、画面単位での分割になるので、ユーザーはあるストーリーを完遂するために、少なくとも2度の画面遷移が必要になります。しかもSSOを使った遷移なので少し時間がかかります。

![](https://storage.googleapis.com/zenn-user-upload/29f38c1353f9-20221213.jpg)


これは良いユーザー体験ではありません。特に、今回切り出した機能には、あるユーザーストーリーの一部を切り出したものもあるので、画面遷移によって体験を分断させたくありませんでした。

なので、この方法は見送りました。

しかし、マイクロサービスチームがUIまで提供する方針は、サービスの開発コストの多くをマイクロサービスチームに移譲できるので悪くないように思えました。なのであとは、ユーザーがシームレスに体験できるように各サービスのUIにマイクロサービスのUIを上手く統合することさえできれば求めていた体験を実現できそうです。

![](https://storage.googleapis.com/zenn-user-upload/6e904754dab3-20221211.jpg)

これがちょうどマイクロフロントエンドの考え方と重なったので、結果的にマイクロフロントエンドを実践することとなりました。

# ユーザーからの見た目

サービス内で統合されるマイクロサービスは、ユーザーからはこのように見えます。

例えば、サービス内のボタンをクリックして開くサイドモーダルにマイクロサービスのUIを表示します。

![](https://storage.googleapis.com/zenn-user-upload/f32fb3ca50d5-20221213.jpg)

あるいは、マイクロサービスの機能を実装したボタンをサービスに配置することで、ユーザーへマイクロサービスの機能を直接提供することができます。

![](https://storage.googleapis.com/zenn-user-upload/124450ddeead-20221213.jpg)

:::message
画像は筆者が独自に作った実在しない画面です
:::

[micro-frontends.org](https://micro-frontends.org/)で紹介されているパターンがイメージに近いです。この画像でいう、「Team Product」がサービスチームで、「Team Checkout」がマイクロサービスチームといった感じです。

![](https://storage.googleapis.com/zenn-user-upload/67d79e9f7b96-20221211.png)

> 出典: [micro-frontends.org](https://micro-frontends.org/)

# アーキテクチャ全体図

私たちが実践しているマイクロフロントエンドのアーキテクチャはこのようになっています。

- 🔵 - マイクロサービスチームのコンポーネント
- 🟡 - サービスチームのコンポーネント

![](https://storage.googleapis.com/zenn-user-upload/347c12fc9f16-20221213.jpg)

# マイクロフロントエンドの統合

マイクロフロントエンドの統合パターンは、マイクロフロントエンドのバリューが一番発揮されるランタイム統合を選択しました。（統合パターンの比較は、[Micro Frontend Guide: Technical Integrations](https://www.trendmicro.com/ru_ru/devops/21/h/micro-frontend-guide-technical-integrations.html)がわかりやすいです。)

私たちの場合は、専任の開発チームが組成され一定の開発コストをかけて継続的に機能開発しアップデートし続けることが前提になっています。一定コストを払う対価として、サービスによってUXのバラツキが生じる期間もなるべく最小にし、エンドユーザーへいち早く価値を届けることが私たちにとっては重要でした。なのでサービスのビルドにブロックされずにエンドユーザーへ届けられるランタイム統合を選びました。ただし、ランタイムで統合するということはサービスのリリースとは関係なくユーザーの環境でアップデートが行われることを意味します。この場合、サービスはマイクロフロントエンドを前のバージョンに戻すようなオペレーションが基本的にはできないので、変更によって生じるバグを修正対応する責任は基本的にはマイクロサービスチームに生じます。そのため、リアルタイムでエンドユーザーに価値を届けられることと、バグ等の修正対応の責任を負うことのトレードオフになると思います。私たちは、そのトレードオフを受け入れました。

逆に、運用メンテにあまり力を入れずに枯れることを前提としたマイクロフロントエンドにするなら、バージョン毎にパッケージとして配布して、サービスのビルドタイムで使うバージョンを指定する方法の方が良いかもしれません。他のライブラリと同様にサービスが細かくバージョンを選べるので、もしマイクロフロントエンドにバグがあってもバージョンを戻すなどの対応をサービスがしやすくなり、マイクロフロントエンドのメンテチームの負荷は下がります。

マイクロフロントエンドは、私のチームではReactで実装していて、Reactコンポーネントを[カスタム要素](https://developer.mozilla.org/ja/docs/Web/Web_Components/Using_custom_elements)にカプセル化します。そして、`.js`にコンパイルして、webサーバーから配信します。

`registerWebComponent`という、Reactコンポーネントをカスタム要素として登録する関数を作り、

```ts
export function registerWebComponent<TProps extends Record<string, unknown>>({
  propTypes,
  tag,
  render,
}: {
  propTypes: React.WeakValidationMap<TProps> | undefined;
  tag: string;
  render: (args: {
    config: WebComponentConfig | undefined;
    root: HTMLElement;
    styleRoot: HTMLElement;
    appRoot: HTMLElement;
    props: TProps;
  }) => JSX.Element;
}): CustomElementConstructor {
  const wc = customElements.get(tag);
  if (wc) {
    return wc;
  }
  const attributeNames = propTypes
    ? Object.keys(propTypes).map(camelToKebab)
    : [];

  class WC extends HTMLElement {
    private attachedShadowRoot: ShadowRoot;
    private styleRoot: HTMLElement;
    private appRoot: HTMLElement;
    private reactRoot: Root;

    _config?: WebComponentConfig;

    static get observedAttributes() {
      return attributeNames;
    }

    constructor() {
      super();

      this.attachedShadowRoot = this.attachShadow({ mode: 'open' });

      this.styleRoot = document.createElement('div');
      this.attachedShadowRoot.appendChild(this.styleRoot);

      this.appRoot = document.createElement('div');
      this.attachedShadowRoot.appendChild(this.appRoot);

      this.reactRoot = createRoot(this.appRoot);
    }

    get config() {
      return this._config;
    }

    set config(value: WebComponentConfig | undefined) {
      this._config = value;
    }

    getAllAttributes() {
      return attributeNames.reduce((acc, attr) => {
        acc[kebabToCamel(attr)] = this.getAttribute(attr);
        return acc;
      }, {} as Record<string, unknown>);
    }

    connectedCallback() {
      this.render();
    }

    disconnectedCallback() {
      this.reactRoot.unmount();
    }

    attributeChangedCallback() {
      this.render();
    }

    private render() {
      this.reactRoot.render(
        render({
          config: this.config,
          root: this,
          styleRoot: this.styleRoot,
          appRoot: this.appRoot,
          props: this.getAllAttributes() as TProps,
        })
      );
    }
  }

  customElements.define(tag, WC);

  return WC;
}
```

Reactコンポーネントをカスタム要素として登録しています。

```tsx
registerWebComponent({
  propTypes: FeatureA.propTypes,
  render: ({ config, root, appRoot, styleRoot, props }) => {
    return (
      <WebComponentProvider
        config={config}
        root={root}
        appRoot={appRoot}
        styleRoot={styleRoot}
      >
        <FeatureA {...props} />
      </WebComponentProvider>
    );
  },
  tag: 'team-a-feature-a',
});
```

サービスは、あらかじめマイクロフロントエンドのタグ名をHTMLに記述しておき、マイクロフロントエンドのjsファイルを読み込みます。

例えば、nuxt2/vue2にカスタム要素を統合する場合、まず`nuxt.config.js`にカスタム要素をvueコンパイラーが無視する設定を追記します。

```js
export default {
  vue: {
    config: {
      ignoredElements: ['team-a-feature-a']
    }
  }
}
```

そして、vue templateにカスタム要素を記述し、マイクロフロントエンドのjsを読み込みます。

```vue
<template>
  <team-a-feature-a></team-a-feature-a>
</template>
<script>
export default {
  async mount() {
    const fetchManifestResponse = await fetch('http://team-a-domain/v1/manifest.json')
    const manifest = await fetchManifestResponse.json()
    const scriptUrl =  URL(manifest.files['team-a-feature-a.js'].replace(/^\//, ''), manifestUrl).toString()
    const injectScript = (url: string): Promise<void> => {
      return new Promise((resolve) => {
        const script = document.createElement('script')
	script.src = url
	script.onload = () => resolve()
	document.head.appendChild(script)
      })
    }
    await injectScript(scriptUrl)
  }
}
</script>
```

:::message
私たちは、サービスチームがマイクロフロントエンドのローディング処理を書かなくて良いように専用のマイクロフロントエンド統合用パッケージを配布しているので、もっと完結にロードできるようになってます。
:::

私たちは、[single-spa](https://single-spa.js.org/)のようなマイクロフロントエンドをオーケストレーションするフレームワークを使用しませんでした。一般的に、マイクロフロントエンドで構築することが前提となっている場合は、そのようなフレームワークを導入することでコンポーネントサイクルやルーティングなどの管理から解放されます。しかし、私たちのマイクロフロントエンドが統合されるアプリケーションは、もともとRailsなどで動作している単一のアプリケーションであり、これからもそのアプリケーションは自身が主として関心毎とすることをそのアプリケーションに実装していきます。そのため、あくまで共通機能を低コストに組み込みたいというニーズでしかないので、私たちはsingle-spaやmodule federationなどの特定の技術の導入を前提とせずに、ブラウザの標準APIで簡単に部分的にサービスがマイクロフロントエンドを利用できることを重要視していました。そのため、web componentsでマイクロフロントエンドをカプセル化し、jsを読み込むだけという方法をとりました。もし、初めからマイクロフロントエンドの組み合わせを前提としたアプリケーションを構築するのであれば、フレームワークを初めから導入するのは良い考えだと思います。

# 配信インフラとバージョニング

ランタイムでマイクロフロントエンドを統合する場合でもバージョンコントロールは必要です。マイクロフロントエンドは、それを利用するサービスとDOMのインターフェースを介して、サービスから情報を受け取ったり、マイクロフロントエンドから通知したりします。仮に、バージョンコントロールをしないと、マイクロフロントエンドのインターフェースに破壊的変更が加わったリリースをしたときに、アプリケーションが壊れてしまいます。

なので、マイクロフロントエンドの内部に閉じた変更と閉じない変更の2パターンでリリースの方法を考える必要があります。以下に私たちのプロジェクトで実際に運用しようとしている設計を紹介します。

私たちは、マイクロフロントエンドのリリースを2つに分類しました。

1. マイクロフロントエンドを利用するアプリの動作に影響する : ここでは便宜上`メジャーアップデート`と呼びます。
2. マイクロフロントエンドを利用するアプリの動作に影響しない : ここでは便宜上`マイナーアップデート`と呼びます。

まず私たちは、2種類のファイルをビルド結果として出力します。

- `[name]-[hash].js` - マイクロフロントエンドとして動作するweb componentを初期化するためのエントリーポイントとなるJavaScriptファイルです。デプロイ後にキャッシュを破棄するために、デプロイのたびに変化するハッシュ値をファイル名に含みます。
- `manifest.json` - クライアントがアセットファイルのURLを見つけるために、固定名とそれに対応する実体ファイルへのURLのマッピング情報をもつJSON形式のファイルです。

```json:manifest.json
{
    "files": {
        "team-a-feature-a.js": "https://example.moneyforward.com/assets/v[version]/team-a-feature-a-[hash].js",
        "team-a-feature-b.js": "https://example.moneyforward.com/assets/v[version]/team-a-feature-b-[hash].js",
    }
}
```

## 1. メジャーアップデート

マイクロフロントエンドのスクリプトは、URLにバージョンを含みます。（例: `https://host/v[major-version]/team-a-feature-a.js` ）

URLに含むバージョンのことを便宜上`メジャーバージョン`と呼びます。

`メジャーアップデート`の場合は、このURLのバージョンを変更することで既存のアプリの動作に影響を与えないようにします。

### デプロイ前

`v1` を利用できます。

![](https://storage.googleapis.com/zenn-user-upload/813de69150e2-20221212.png)

`v1/manifest.json`は、`v1`のアセットへの参照を持ちます。

```json:v1/manifest.json
{
    "files": {
        "team-a-feature-a.js": "https://host/v1/team-a-feature-a-[hash].js",
        "team-a-feature-b.js": "https://host/v1/team-a-feature-b-[hash].js"
    }
}
```

### メジャーアップデートをデプロイ後

`v1`と`v2`の両方を利用できる状態になります。

![](https://storage.googleapis.com/zenn-user-upload/621d6cf013ca-20221212.png)

`v2/manifest.json`は、`v2`のアセットへの参照を持ちます。

```json:v2/manifest.json
{
    "files": {
        "team-a-feature-a.js": "https://host/v2/team-a-feature-a-[hash].js",
        "team-a-feature-b.js": "https://host/v2/team-a-feature-b-[hash].js"
    }
}
```

:::message
これ以降のバグ修正や変更は基本的には`v2`に対して行います。例外として、`v1`で致命的なバグが見つかった場合のみ`v1`へ修正版のリリースを行います。
:::

### サービスチームがやること

Vueプロジェクトがマイクロフロントエンドを利用しているケースを考えます。

サービスチームは、マイクロフロントエンドを利用するためのパッケージをインストールしています。このパッケージでは`manifest.json`から利用したいweb componentのエントリーファイルをダウンロードする機能を提供します。サービスチームは、新しく利用するマイクロフロントエンドのメジャーバージョンのURLに変更し、実装を修正し、アプリをリリースします。

```diff vue
 <template>
    <team-a-feature-a
        :v1-attr="v1Attr"
+       :v2-attr="v2Attr"
    ></team-a-feature-a>
 </template>

 <script>
 export default {
   async mounted() {
-    const fetchManifestResponse = await fetch('http://team-a-domain/v1/manifest.json')
+    const fetchManifestResponse = await fetch('http://team-a-domain/v2/manifest.json')
     const manifest = await fetchManifestResponse.json()
     const scriptUrl =  URL(manifest.files['team-a-feature-a.js'].replace(/^\//, ''), manifestUrl).toString()
     const injectScript = (url: string): Promise<void> => {
       return new Promise((resolve) => {
         const script = document.createElement('script')
         script.src = url
	 script.onload = () => resolve()
	 document.head.appendChild(script)
       })
     }
     await injectScript(scriptUrl)
   }
 }
 </script>
```

### v1を削除するオペレーション後

全てのサービスが`v2`を利用するようになり安定したら、マイクロフロントエンドの開発チームは`v1`を削除して`v2`のみが利用できる状態にします。

![](https://storage.googleapis.com/zenn-user-upload/8f4ae81de6f3-20221212.png)

:::message
削除オペレーションは、開発チームの任意のタイミングで行えるようにします。
:::

## 2. マイナーアップデート

マイクロフロントエンドを利用するアプリの動作に影響しないようなリリースの場合は、プロダクトの開発チームがリリースしなくともブラウザ上で最新のスクリプトをダウンロードできるように、最新のメジャーバージョンのファイルを置換する形で新しいハッシュ値を持ったファイルをデプロイします。

### デプロイ前

![](https://storage.googleapis.com/zenn-user-upload/b1b2f2e4573b-20221212.png)

`v1/manifest.json`は、ハッシュ値`xxxxxx`のファイルへの参照を持ちます。

```json:v1/manifest.json
{
    "files": {
        "team-a-feature-a.js": "https://host/v1/team-a-feature-a-xxxxxx.js",
        "team-a-feature-b.js": "https://host/v1/team-a-feature-b-xxxxxx.js"
    }
}
```

### デプロイ後

`v1`の中で、ハッシュ値`xxxxxx`のファイルが新しいハッシュ値`yyyyyy`のファイルで置き換わります。

![](https://storage.googleapis.com/zenn-user-upload/ea833955874d-20221212.png)

`v1/manifest.json`は、新しいハッシュ値`yyyyyy`を参照するように書き変わります。

```json:v1/manifest.json
{
    "files": {
        "team-a-feature-a.js": "https://host/v1/team-a-feature-a-yyyyyy.js",
        "team-a-feature-b.js": "https://host/v1/team-a-feature-b-yyyyyy.js"
    }
}
```

:::message
リリース後に古いmanifest.jsonのキャッシュを参照して、JSファイルが404にならないように、ダウンロードするmanifest.jsonと実体の整合性を保たれるように注意します。
:::

### サービスチームがやること

マイナーアップデートの場合は、サービスチームがやることはありません。

以上がデプロイ等の設計です。

# 認証

最後に認証をまとめて終わります。

マネーフォワードクラウドは、ユーザー認証した上で自身が所属する事業者を選択すると機能が利用できるようになります。そのため、マイクロフロントエンドで認証済みリクエストをどのように実現するかも考える必要がありました。ここでは私たちが実際にどのように設計したかを紹介します。

考えるべき要件は以下の通りです。インシデントにならないためにも、確実に以下の要件が満たされている必要があります。

- セキュリティ観点
  - どんな時もサービスにログインしているユーザーIDと利用中の事業者IDを使ってコンテンツを表示していること。
  - サービスでのセッション情報が変化（ログアウトや利用事業者の変更）してもマイクロフロントエンドの認証情報も同期できていること。
- UX観点
  - ユーザーは、マイクロフロントエンドに対して認証していることを意識しないこと。

私たちは、初めに社内のOpenIDプロバイダーを使った認証を考えました。マネーフォワードには既に認証基盤であるOpenIDプロバイダーが実装されており、「ユーザー認証する＝これを使うこと」だったからです。

![](https://storage.googleapis.com/zenn-user-upload/2d85364d80ff-20221213.jpg)

ユーザーがサービスにログイン後画面を開くところまでは通常のサービスの認証フローです。マイクロフロントエンドの画面を開くところ（4枚目の画像）からのポイントを説明します。

1. マイクロフロントエンドの内部で、見えないiframeを挿入します。
2. そのiframeは、認証基盤へAuthZリクエストをします。この時、`prompt=none`と`login_hint`パラメーターで送ることで、ユーザーへの再認証をスキップできます。
3. そして、コールバックに付与される認可コードをバックエンドサーバーに送り、バックエンド間でトークンの交換や認証等の処理をします。

:::message
auth0に実装されている[Silent Authentication](https://auth0.com/docs/authenticate/login/configure-silent-authentication)を参考にしました。
:::

しかし、UX観点の要件を満たすことはできても、セキュリティ観点の要件を満たすことが非常に困難でした。

>- どんな時もサービスにログインしているユーザーIDと利用中の事業者IDを使ってコンテンツを表示していること。
>- サービスでのセッション情報が変化（ログアウトや利用事業者の変更）してもマイクロフロントエンドの認証情報も同期できていること。


認証情報を同期するためにさまざまな方法を検討しましたが、どちらもかなり複雑になってしまい、実装コストがあまりに高くいくつものサービスに横展開できないと思ったので見送りました。

この設計は、本質的にサービスとマイクロフロントエンドとで別々のセッションを作るというものでしたが、これだと、それぞれのセッションの状態を同期することが非常に困難だということがわかりました。

なので、私たちは、マイクロフロントエンド自身が認証を持つことをやめることにし、サービスのバックエンドサーバーをBFFとし、そこで認証を行うことにしました。

![](https://storage.googleapis.com/zenn-user-upload/f66e462ccbba-20221213.jpg)

サービスのバックエンドサーバーは間違いなくそのセッションの最新の認証情報を持っているので、セキュリティ観点の要件を満たすことができます。

BFFは、クライアントからのリクエスト情報をそのままマイクロフロントエンドのAPIサーバーにリクエストするだけです。この時、ユーザーIDや事業者IDなどの認証情報、権限情報などは、サービスのバックエンドサーバーからマイクロサービスのバックエンドサーバーに送るリクエストのヘッダーに含めることにしました。こうすることで、マイクロフロントエンドのAPIサーバーを外部に公開することなく、BFFによって保証されたリクエストのみを受け付けることができます。そして、それは常に新鮮な認証情報を持つので、マイクロサービスチームが認証の扱いに頭を悩ませる必要がなくなります。

また、この方法をとるとBFFを実装するサービス開発チームに実装コストが発生するので、私たちはサービスチームがBFFの実装を考えるのにコストを払わなくて良いように、BFFのサンプルコードや詳細の仕様を綿密にまとめまています。これはストリームアラインドチームに負荷がかからないために重要です。

# 最後にポエム

長い文章をここまで読んでくださって、ありがとうございます。

この1年間マイクロフロントエンドを実装するためにさまざまなことを考え実践してきたことの一部を今回アドベントカレンダーというお祭りに乗じて世に出すことができました。マイクロフロントエンドについては、賛否両論さまざまな意見があるかとは思いますが、私は国内でもまだまだ事例が少ないマイクロフロントエンドに挑戦することで私たちの可能性を広げビジネスをさらに加速させることと信じて取り組んできました。

このシステムがリリースを迎え、今後マネーフォワードクラウドのサービスでこのシステムを利用していく中で、想定していないさまざまな困難に直面することと思います。しかし、この挑戦は私たちのビジネスを加速させ、よりエキサイティングなものになると信じています。

良いお年を！

# 求人

マネーフォワードの名古屋拠点ではエンジニアを募集してます！マイクロフロントエンドはもちろん、他にも挑戦的な取り組みをしてます。
まずは気軽にカジュアル面談からでも歓迎ですのでご応募お待ちしてます👉[求人ページ](https://hrmos.co/pages/moneyforward/jobs/1705418982829588494)
