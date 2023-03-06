---
title: "言語が違う複数のリポジトリをmonorepoへ移行した話"
emoji: "🚚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["monorepo","polyrepo"]
published: true
publication_name: "moneyforward"
---
# 導入

最近私はマネーフォワードで開発しているプロジェクトをMonorepoに移行する機会があったので、それに至った経緯やどのように移行したか、そして、移行してどんな問題が解決されたかについてまとめたいと思います。

ちなみに、私のMonorepoでの開発経験は皆無ですので、この記事はMonorepoの正解を提示するものではなく、あくまで私たちが抱えていた問題を解消するために行った一つのケースであることはご了承ください。

# Monorepoとは

Monorepoは、複数のプロジェクトやコンポーネントを単一のリポジトリで管理する開発手法です。一方で、プロジェクトやコンポーネント毎にリポジトリを分割して、個々のプロジェクトを独立して管理する方法をPolyrepoと呼びます。

# チーム構成

私たちの現在のチーム構成です。

| 役割             | 人数         |
|----------------|------------|
| PdM            | 1人         |
| デザイナー          | 1人         |
| Backend エンジニア  | 3人（うち1人EM） |
| Frontend エンジニア | 3人         |

# Monorepo移行前

monorepoに移行する前は、以下のようなpolyrepo構成でした。

| リポジトリ         | プログラム言語    | 役割                                                                     |
|---------------|------------|------------------------------------------------------------------------|
| project       | なし         | PdMが管理するEpic issueや仕様書、他チーム向けのガイドラインなどを管理します。                          |
| project_api   | kotlin     | バックエンドのコードやバックエンドエンジニアのタスクを管理するissue、バックエンド領域における開発者向けのドキュメントなどを管理します。 |
| project_front | typescript | project_apiと同様                                                         |

:::message
上記リポジトリ名の*project*には、私たちのプロジェクト名が入ります。
:::

![our team and polyrepo](/images/articles/migrating-to-a-monorepo/our-team-and-polyrepo.png)

割とよくある構成かなと思いますが、私たちは主に **"ドキュメント管理の難しさ"**と**"共有リソースの管理コスト"**、**"分業制によるチーム内の分断"** に課題感を持っていました。それぞれの問題とそれらが解決されどのようになったかを詳細に見ていきましょう。

# 抱えていた問題とMonorepo移行後

私たちが抱えていた問題がmonorepo移行によってどのように解決されたかをまとめます。

## ドキュメントが分散して探しにくい問題

### Before

プロジェクトに関するドキュメントが複数のリポジトリに分散するためにドキュメントの検索性や一覧性が低下しており、私たちのプロジェクトについて知りたい人が情報を探すのが難しい問題がありました。他方で、私たち自身も、どこに情報を残すかで迷うことも少なくありませんでした。

### After

プロジェクトに関する情報が1つのリポジトリに集約されたため、ドキュメントの探しやすさが向上しました。また、私たち自身、どのリポジトリに情報を置くかという迷いはなくなりました。そして、プロジェクトに関する様々なドキュメントを一箇所に集約できるようになったので、より良いドキュメント構成を考えられるようになりました。

## 開発サイクル外のドキュメントが陳腐化しがち

### Before

基本的に開発者は、開発サイクルの中でコードと一緒にドキュメントをメンテナンスする意識がありますが、リポジトリ外にあるドキュメントへの意識は薄れがちです。このため、コードと別の場所に置かれたドキュメントの更新が気づいた時になってしまったり、そのドキュメントを更新するのが作成者だけになってしまうことがありました。

### After

1コミットで全ての情報を編集可能になり、あらゆる情報が開発サイクルの中で更新されるようになりました。

## GitHubの設定を同期するコスト

### Before

リポジトリのラベルや設定値はリポジトリの数だけ手作業で同期していました。

### After

GitHubのラベルや設定値の変更箇所が一箇所になり管理コストを削減できました。

## 共通のビルドやデプロイプロセスの管理コスト

### Before

マネーフォワードではサービスのリポジトリでいくつか決められたワークフローを実行することが求められているので、私たちが管理する全てのリポジトリでも全く同じCIの設定を行う必要があります。このため、リポジトリを増やすたびに、CIの設定をコピーしたり、変更を反映したりする必要がありました。

### After

共通のビルドジョブやデプロイプロセスの変更箇所が一箇所になり管理コストが削減されました。

## 分業制によるチーム内の分断

### Before

私たちの開発チームは元より、バックエンドとフロントエンドを分業制で開発し始めたため、バックエンドとフロントエンドのリポジトリは自然と分かれてスタートしました。その状況が続き、自身がメインで開発するリポジトリに集中するようになった結果、大きく複雑になったどちらのリポジトリにも精通したメンバーがいなくなり、互いのリポジトリにコミットすることの心理的ハードルが高まってしまいました。そうして、チーム内の分断を招いていました。

### After

リポジトリという隔たりは予想以上に心理的に影響を及ぼしていたようで、monorepoになり1コミットで複数のアプリケーションを変更できるようになったことで、自分の変更によって他方のアプリケーションにも変更が必要な場面で、今まで触ってこなかった他方のコードも一緒に変更してしまおうと思うメンバーが出てきました。このように、monorepoにすることでメンバーのメンタルモデルが変化し、他のアプリケーションを開発する方法を学ぼうとしたり、プルリクエストを見ようとするメンバーが増えてきたことは嬉しい変化です。monorepoにしたことでチーム内の心理的な壁が解消されたと感じています。これからさらにコラボレーションが生まれていくこと期待しています。

# 僕たちのMonorepo 1.0

Monorepoといってもいろいろなやり方がありますが、ここでは僕たちのMonorepo 1.0がどのように構成されたかを紹介します。

## Monorepo移行のゴール

今回のMonorepo移行作業は、**project_api、project_frontリポジトリをprojectリポジトリに統合しprojectリポジトリでこれまで通りの開発サイクルを回せる状態**を目指しました。

## ビルドツール

今回[Turbo](https://turbo.build/)や[nx](https://nx.dev/)などのモノレポを効率的に管理するためのツールは導入しませんでした。

理由は、既存のpolyrepoは既にそれぞれ違う言語で違うビルドツールで既に開発、運用されてきたことがあります。このようなpolyrepoを一つの管理ツールに統合するには、CI/CDの再構築や新たなツールの学習コストが必要になったり、運用体制を大きく変える必要があることが想定されました。最初からmonorepo環境でプロジェクトを始めるならまだしも、私たちが解消したい課題に対してはtoo muchだと考えたので、このタイミングで導入することは見送りました。

## フォルダ構造

私たちは、このようなフォルダ構造にしました。`apps`フォルダをルートに作成し、`apps`フォルダ内にpolyrepoだった各アプリケーションを配置しました。docsは、すぐに目につくようにルートに配置しました。

```shell:フォルダ構造
.circleci
.github
apps
├ backend # 元project_apiリポジトリ(kotlin)
└ web     # 元project_frontリポジトリ(typescript)
docs
```

ルートにアプリケーションをそのまま配置するパターンも検討しましたが、ルートが肥大化し見通しが悪くなることが予想されたため、`apps`フォルダを作成しました。

## 移行時にやったこと/諦めたこと

### やったこと

* **gitコミットの移行** - ファイルのみを移行すると移行後にファイルの変更履歴を遡っても移行時点の巨大なコミットにしか辿れなくなってしまうので将来誰かが困ってしまいます。そのため、[tomono](https://github.com/hraban/tomono)を使ってpolyrepoのファイルとgitコミットを移行しました。
* **issueの移行** - polyrepoのopenなissueはmonorepoに移行しました。
* **コミットメッセージ中のissue/PR番号の一括変換** - GitHubはコミットメッセージ中の「#issue番号」の形式で書かれた文字列を同リポジトリのissue/PR番号として解釈して画面にリンクで表示します。コミットは別のリポジトリへコピーするので、何もしなければリンクが新しいリポジトリに向き、リンクが壊れるか全く違うissueへリンクしてしまいます。これは未来の開発者が困るので対処しました。マージ済みPRを新しいリポジトリへ移行することが難しいので、コミットメッセージ中の「#issue番号」を「org/polyrepo#issue番号」のようなフォーマットに変換することで元のリポジトリへのリンクを保持するようにしました。
* **issueにappラベルを付与** - project_apiリポジトリとproject_frontリポジトリのissueをprojectリポジトリに移行すると、どちらの関心事のissueなのかを判別できなくなることが想定されました。これを防ぐために、移行する前に全てのissueに「app:backend」「app:web」のようなラベルを付与しました。移行後はGitHub Projectsのカスタム項目に付け替えを行いました。
* **CI/CDの修正** - それぞれのリポジトリで実行していたCI/CDをmonorepoに統合した後も動作するようにしました。この時、変更されたファイルパスに基づいて、特定のワークフローだけをトリガーするようにしました。そうしなければ、ちょっとした変更を加えるだけでも、全てのアプリケーションでビルド、テスト、デプロイの一連のプロセスを毎回行うことになり非効率だからです。

### 諦めたこと

* **PRとブランチの移行** - 既にproject_apiリポジトリとproject_frontリポジトリで作成されたPRをprojectリポジトリに移行すると手間が増えるのでコストカットの面で諦めました。なので、移行作業はなるべく開発がキリよく終わっているスプリント最終日の金曜の夜に行いました。その時点でも取り込まれていないPRはありましたが、担当者に土下座して（気持ち）多少不安定でも取り込んでもらいました。
* **「#issue番号」ではない形式のissue番号の変換** - Githubは「#issue番号」の形式ではないものをissueとして解釈しないのと、GitHubがマージコミットメッセージに含める「#PR番号」から関連issueへ辿れるので、変換しなくても大きな支障はないと判断し対応しませんでした。
* **discussionsの移行** - GitHubのdiscussions機能を一時期使ってましたがあまり私たちに馴染まなかったので形骸化しました。そのため、そこにある情報量は少なく重要でないものばかりだったので、移行の対象外としました。既存のpolyrepoは消さずに残すので、discussionsは必要になったタイミングでマニュアルで移行することにしました。

# 移行作業

当日の移行作業は、以下の手順で行いました。

1. ファイルとgit履歴の移行
2. issueの移行
3. CircleCIの設定変更
4. GitHub Actionsの設定変更
5. GitHub Projectsの修正

## ファイルとgit履歴の移行

当日私が実行したコマンドの履歴です。

```shell
# 作業用ディレクトリを作成
mkdir monorepo-migration-workspace && cd monorepo-migration-workspace
# tomonoのための環境変数を設定
export MONOREPO_NAME={monorepo名} # 移行先のリポジトリ名
# git filter-branchの警告を抑制
export FILTER_BRANCH_SQUELCH_WARNING=1
# 最新のリポジトリをクローン
git clone git@github.com:moneyforward/$MONOREPO_NAME.git
git clone git@github.com:moneyforward/project_front.git
git clone git@github.com:moneyforward/project_api.git
# tomonoコマンドをインストール
curl https://raw.githubusercontent.com/hraban/tomono/master/tomono | sed "s/shopt -s inherit_errexit/shopt -s inherit_errexit 2>\/dev\/null || true/" > tomono && chmod 755 tomono
# tomonoを使ってproject_frontをwebという名前でmonorepoフォルダに移行
echo "$PWD/project_front web" | ./tomono --continue
# monorepoに移動
cd $MONOREPO_NAME
# appsディレクトリを作成
mkdir apps
# webをappsに移動し、tomonoの移行コミットに追加
mv web  apps
git add .
git commit --amend
# git filter-branchでコミットメッセージ中の#issue番号を旧polyrepoのURLに置換
git filter-branch --msg-filter 'sed -e "s/.*\/\(#\d*\)/\1/g" | sed -e "s/\(#\d*\)/moneyforward\/project_front\1/g"' origin/main..main
git push origin main
# 作業用ディレクトリに戻る
cd ..
# tomonoを使ってproject_apiをbackendという名前でmonorepoフォルダに移行
echo "$PWD/project_api backend" | ./tomono --continue
# monorepoに移動
cd $MONOREPO_NAME
# backendをappsに移動し、tomonoの移行コミットに追加
mv backend apps
git add .
git commit --amend
# git filter-branchでコミットメッセージ中の#issue番号を旧polyrepoのURLに置換
git filter-branch --msg-filter 'sed -e "s/.*\/\(#\d*\)/\1/g" | sed -e "s/\(#\d*\)/moneyforward\/project_api\1/g"' origin/main..main
git push origin main
```

## issueの移行

旧polyrepoをクローンしたフォルダで以下のコマンドを実行し、issueを移行しました。

```shell
cd project_front
gh issue list -s open -L 100 --json number | jq -r '.[] | .number' | xargs -I% gh issue edit % --add-label "app:web"
gh issue list -s open -L 100 --json number | jq -r '.[] | .number' | xargs -I% gh issue transfer % https://github.com/moneyforward/monorepo

cd project_api
gh issue list -s open -L 100 --json number | jq -r '.[] | .number' | xargs -I% gh issue edit % --add-label "app:backend"
gh issue list -s open -L 100 --json number | jq -r '.[] | .number' | xargs -I% gh issue transfer % https://github.com/moneyforward/monorepo
```

## CircleCIの設定変更

CircleCIは現在設定ファイルを分割できないので、1つのconfig.ymlしか読み取れません。しかし、そうすると複数のアプリケーションのための設定が混在するので、ファイルサイズが大きくなり見通しが悪く管理コストが高くなってしまいそうでした。なので、monorepo構成でも変わらずにアプリケーション単位でCircleCIの設定ファイルを管理する方法を考えました。

```text
.circleci
├ workflows // 各アプリケーションのワークフロー設定を格納するフォルダ
│ ├ backend-workflow.yml
│ └ web-workflow.yml
├ base.yml // マージ時にベースになるYAMLファイル
└ config.yml // CircleCIが実行するYAMLファイル
```

CircleCIでは、[ダイナミックコンフィグ](https://circleci.com/docs/ja/dynamic-config/)というconfig.ymlを動的生成できる機能と[Path Filtering Orb](https://circleci.com/developer/orbs/orb/circleci/path-filtering)という実行するジョブを変更ファイルパスでフィルターするOrbを組み合わせることで、変更があったファイルによって実行するジョブを制御することができます。しかし、その制御用のYAMLファイルは1つのYAMLファイルで構成する必要があり、プロジェクト単位で設定ファイルを作れない問題があります。この問題は、事前に分割したYAMLファイルをCIのプロセス中に動的に結合する処理を行うことで解決することができます。（参考 [CircleCIで設定ファイルを分割してpath-filteringも適用する方法 - Qiita](https://qiita.com/algas/items/d0e4d460b938d924ae8f)）

YAMLファイルの結合には、yqを利用します。circleci config packコマンドでもYAMLの結合ができますが、断片のYAMLファイルがあらかじめ決まったフォルダ構成になっている必要があるらしいので、今回のように任意のプロジェクトフォルダ名にYAMLファイルを格納したいケースにはマッチしないので見送ります。（参考 [Dynamic Configurationを使って.circleci/config.ymlを分割する - Zenn](https://zenn.dev/kesin11/articles/96f0947583f34d#circleci-config-pack:~:text=%E3%81%82%E3%82%89%E3%81%8B%E3%81%98%E3%82%81%E6%B1%BA%E3%81%BE%E3%81%A3%E3%81%9F%E3%83%87%E3%82%A3%E3%83%AC%E3%82%AF%E3%83%88%E3%83%AA%E6%A7%8B%E6%88%90%E3%81%AB%E5%BE%93%E3%81%A3%E3%81%A6%E5%88%86%E5%89%B2%E3%81%95%E3%82%8C%E3%81%9Fyaml%E3%81%8C%E9%85%8D%E7%BD%AE%E3%81%95%E3%82%8C%E3%81%A6%E3%81%84%E3%82%8B%E5%89%8D%E6%8F%90%E3%81%A8%E3%81%AA%E3%81%A3%E3%81%A6%E3%81%8A%E3%82%8A%E3%80%81%E3%81%9D%E3%81%AE%E5%88%86%E5%89%B2%E3%81%AE%E5%8D%98%E4%BD%8D%E3%81%8Cexecutor%2C%20job%E3%81%AA%E3%81%A9%E3%81%A8%E6%B1%BA%E3%81%BE%E3%81%A3%E3%81%A6%E3%81%84%E3%82%8B%E3%81%93%E3%81%A8%E3%81%8C%E5%80%8B%E4%BA%BA%E7%9A%84%E3%81%AB%E3%81%AF%E4%BD%BF%E3%81%84%E3%81%AB%E3%81%8F%E3%81%84)）

```yaml:.circleci/config.yml
# CircleCIが実行するYAMLファイル
# 1. プロジェクトのYAMLファイルを結合する
# 2. path-filteringで変更ファイルに対応したワークフローを実行する
version: 2.1
setup: true # CircleCIのダイナミックコンフィグ機能を利用することを宣言する
orbs:
  path-filtering: circleci/path-filtering@0.1.3
jobs:
  merge-configs:
    docker:
      - image: cimg/go:1.18.3 # yqを利用するためにgoのイメージを使う
    steps:
      - checkout
      - run:
          name: Install yq
          command: |
            go install github.com/mikefarah/yq/v4@latest
            yq --version
      - run:
          name: Merge config files
          command: |
            mkdir -p /tmp/workspace
            yq eval-all '. as $item ireduce ({}; . * $item )' ./.circleci/base.yml ./.circleci/workflows/*.yml > /tmp/workspace/merged.yml
            cat /tmp/workspace/merged.yml
      - persist_to_workspace:
          root: /tmp/workspace
          paths:
            - merged.yml
workflows:
  generate-config:
    jobs:
      - merge-configs
      - path-filtering/filter:
          requires:
            - merge-configs
          pre-steps:
            - attach_workspace:
                at: /tmp/workspace
          base-revision: main # 差分を計算するために比較する先のrevision
          config-path: /tmp/workspace/merged.yml # mappingのパイプラインパラメーターを渡すYAMLファイル
          mapping: | # Syntax: <regex path> <pipeline parameter name> <value>
            apps/backend/.* build-backend true
            apps/web/.* build-web true
```

```yaml:.circleci/base.yml
# 共通で利用するCircleCIの設定値を含める
version: 2.1
docker_hub_auth: &docker_hub_auth
  auth:
    username: $DOCKERHUB_USERNAME
    password: $DOCKERHUB_PASSWORD
workflows:
  version: 2
```

プロジェクト用のcircleciの設定ファイルは、以下のparametersとwhenを新しく記述し、base.ymlに書いてある項目を削除しました。これで、変更ファイルパスに応じてトリガーするワークフローを制御できます。

```yaml:.circleci/workflows/*.yml
parameters:
  run-(APP NAME)-workflow:
    type: boolean
    default: false
workflows:
  (APP NAME)-workflow:
    when: << pipeline.parameters.run-(APP NAME)-workflow >>
```

最後に、このままだとセットアップワークフローによって生成されたパイプラインがトリガーするワークフローがない時に、CircleCIはエラーにしてしまうので、必ず実行するワークフローを追加しておきます。

```yaml:.circleci/base.yml
jobs:
  path-filtering-fallback:
    docker:
      - image: cimg/base:2023.02
    steps:
      - run:
          name: PASS CI
          command: echo "Dynamic config is executed"
workflows:
  version: 2
  path-filtering-fallback:
    jobs:
      - path-filtering-fallback
```

:::message
⚠️ yqコマンドでYAMLファイルを結合するにあたって、同一キー名の値は上書きしてしまうので、プロジェクト用circleci設定ファイルのworkflowやjobなどの設定値にはアプリ名をprefixに加えることを規約としました。
:::

## GitHub Actionsの設定変更

GitHubは簡単です。`on.pull_request.paths`で変更検知するファイルパスを指定することで、そのワークフローを実行できます。`working-directory`の設定も追加しておくと、step実行時のカレントディレクトリを変更できます。

```yaml
on:
  pull_request:
    paths:
      - 'apps/web/**'
defaults:
  run:
    shell: bash
    working-directory: ./apps/web
```

## GitHub Projectsの修正

私たちはGitHub Projectsでプロジェクトを管理しているので、その中で`repo`でフィルターしていた箇所を`app`カスタム項目でフィルターするように修正しました。

# 最後に

私が開発環境の細かい設定まで気が回らず、モノレポ移行後に所々壊れてしまった部分がありましたが、メンバーが協力して半日程度で修正してくれました。ありがとうございます。また、モノレポに移行したことで多くの課題が解消され、それ以上のメリットも享受できました。この環境で今後さらに開発が加速することを期待しています。

# 参考

- [monorepo.tools](https://monorepo.tools/)
- [Monorepo開発のメリット vs デメリット](https://circleci.com/ja/blog/monorepo-dev-practices/)
- [ダイナミックコンフィグ - CircleCI](https://circleci.com/docs/ja/dynamic-config/)
- [変更されたファイルに基づいて特定の workflows または steps を実行する - CircleCI](https://circleci.com/docs/ja/using-dynamic-configuration/#execute-specific-workflows-or-steps-based-on-which-files-are-modified)
- [Path Filtering Orb](https://circleci.com/developer/orbs/orb/circleci/path-filtering)
- [CircleCIで設定ファイルを分割してpath-filteringも適用する方法 - Qiita](https://qiita.com/algas/items/d0e4d460b938d924ae8f)
- [GitHub Actions のワークフロー構文](https://docs.github.com/ja/actions/using-workflows/workflow-syntax-for-github-actions#onpushpull_requestpull_request_targetpathspaths-ignore)
