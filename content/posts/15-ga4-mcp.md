+++
date = '2025-08-11T21:53:03+09:00'
draft = false
tags = ['n8n', 'mcp', 'google analytics']
category = "Howto"
subcategory = ""
title = 'Google Analytics4の公式MCPサーバーをn8nに導入する【GCP】'
+++

最近、Google公式が[Google Analytics4のMCPサーバー](https://github.com/googleanalytics/google-analytics-mcp?tab=readme-ov-file)を提供し始めたので、n8n上で使えるようにしてみました。

## 前書き

### 注意

注意！今回、npxやnpmコマンドは使用しません。<br>
GA4のMCPサーバの作成方法の記事を見ると、以下のリポジトリを使用している記事が出てきます。<br>

https://github.com/ruchernchong/mcp-server-google-analytics <br>

しかし、こちらはruchernchong氏が作成した非公式のMCPサーバーです。
公式はPythonベースなのでnpxやnpmコマンドは使用しません。混同しないよう注意です。

### 環境
n8nをセルフホストし、GCPのVMインスタンス上で動かしています。

前回そのセットアップ方法はこちらに書いているので、ご参考までに：<br>
https://kamikigit.github.io/posts/11-n8n/

## APIの有効化

GCP上の設定です。VMインスタンスのあるプロジェクトで次のAPIを有効化します。

- Google Analytics Data API
- Google Analytics Admin API



## 認証の発行

ローカルのターミナルからgcloudを使用してGCPにログインし、認証情報を作成します。

```bash
gcloud auth login
gcloud config set project [PROJECT_ID]
```

```bash
gcloud iam service-accounts create [NAME] 
```
`NAME`には任意のサービスアカウントの名前を入力します。なんでもいいですが短すぎるとエラーになるので、n8n-connectとかでいいと思います。

```bash
gcloud compute instances stop [INSTANCE_NAME] --zone [ZONE]
```

インスタンスを止めないとエラーになるので一旦ストップします。

```bash
gcloud compute instances set-service-account [INSTANCE] \
  --zone [ZONE] \
  --service-account [SA_EMAIL] \
  --scopes="https://www.googleapis.com/auth/analytics.readonly"
```

VMインスタンスにサービスアカウントを付与し、Google Analytics読み取り専用のスコープを設定します。

ちなみにSA_EMAILには先ほど作成したサービスアカウントのメールアドレスを入力します。（`[サービスアカウント名]@[プロジェクトID].iam.gserviceaccount.com`のやつ）

```bash
gcloud compute instances start [INSTANCE_NAME] --zone [ZONE]
```

インスタンスを再起動して完了です。

## GA4上の設定
先ほど作成したサービスアカウントのメールアドレスをGA4に登録します。

GA4を開いて、<u>管理 > アカウントのアクセス管理</u> と辿るとユーザを追加できるので、先ほどのメールアドレスを入力し、権限を **閲覧者** に設定して追加します。

![GA4](/images/15/ga4-add-user.png)

## Dockerfileの作成

```Dockerfile
# Dockerfile.n8n
FROM docker.n8n.io/n8nio/n8n:latest

USER root

# Python環境とビルドツールをインストール
RUN apk add --no-cache --virtual .build-deps \
      build-base python3-dev libffi-dev && \
    apk add --no-cache python3 pipx git


# pipx環境変数とGoogle Analytics MCPインストール
ENV PIPX_HOME=/opt/pipx PIPX_BIN_DIR=/usr/local/bin
RUN pipx ensurepath && \
    pipx install "git+https://github.com/googleanalytics/google-analytics-mcp.git" && \
    apk del .build-deps && \
    chown -R node:node /opt/pipx


# n8n 実行ユーザーに戻す
USER node

```

n8nの公式が提供しているイメージはPythonが入っていないので、そこにpythonとpipxをインストールするようにします。
てっきりCodeのノード上でPythonが使えるので入ってると思ったのですが、あれはPyodideを使用してサンドボックス上で動いていました。

ついでにGA4のMCPも入れておくことで、MCPノードの実行時に毎回環境を作る必要がなくなるので便利です。

## docker composeファイルの修正

先ほどのDokcerfileを使用するように、compose.yamlを修正します。

このcompose.yamlは[公式のもの](https://docs.n8n.io/hosting/installation/server-setups/docker-compose/#6-create-docker-compose-file)を使用していることを前提に考え、変更部分だけ抜粋して紹介します。

```yaml
  n8n:
    build:
      context: .
      dockerfile: Dockerfile
    # image: docker.n8n.io/n8nio/n8n # 元のイメージ
```
これまで直接imageで公式のn8nイメージを使用していたところを、Dockerfileを使ってbuildするように修正します。

```bash
sudo docker compose up -d --build n8n
```

コンテナを再起動します。少し時間がかかるので休憩。


## MCPノードの設定

ではn8nのページを開いて、MCPノードの設定をしていきます。

### コミュニティーノードのインストール

既存のMCPノードはSSEしか使えないので、コミュニティーノードという拡張機能を利用します。

[n8n-nodes-mcp](https://github.com/nerding-io/n8n-nodes-mcp)を使います。

導入方法は、[こちらの記事](https://zenn.dev/lecto/articles/d61ff851f6efd5)がわかりやすいので、公式のREADMEと併せて見るのがおすすめです。

ただし、セルフホストしている場合は、`.env`ファイルに以下を追加するのを忘れないようにしましょう。

```bash
N8N_COMMUNITY_PACKAGES_ALLOW_TOOL_USAGE=true
```

### ノードの設定

- Tool Description: `Set Automaticaly`
- Operation: `List Tools`

![n8n-node](/images/15/node-setting.png)

### クレデンシャルの設定

**Command Line(Studio)** を選択できるようになるので、これを使います。

- Command: `google-analytics-mcp`

![command](/images/15/command.png)


Execute stepを押して実行すると…

![result](/images/15/result.png)


実行できました！長い道のりだった…

### 蛇足

ちなみに、ベーシックにpipxコマンドを使用して実行できないか試してみたのですが、実行時間超過でエラーになってしまいました。毎回pipxを実行するのは時間も負荷もかかるので、Dockerfileを使って先にインストールしておくのがやっぱり良さそうですね。

- Command: `pipx`
- Arguments: `run --spec git+https://github.com/googleanalytics/google-analytics-mcp.git google-analytics-mcp`



## 実行してみる

![workflow](/images/15/workflow.png)

とりあえずこういう感じで配置してみました。Date & Timeのノードは、相対的な日付を指定したいときに使えるので入れました。ChatGPTのAPIは現在時刻とかはわからないので。

### プロンプト
```
3日前のビュー数を教えてください。

MCPツールを使うときは必ずプロパティIDを指定しなさい。
property_id="properties/12345678"

## 追加情報：
{{ $json.currentDate }}
```


多分プロンプトにプロパティIDを書かなくても動くと思いますが、分析するプロパティが固定で決まっているなら直接プロパティIDを指定した方が早いかなと思いました。

ちなみにプロパティIDは、GA4のページの 管理 > プロパティの詳細 から、右上にあります。私は間違えてストリームIDをプロパティIDと勘違いして入力していて、403エラーでしばらく悩んでいました…😇


### 出力結果

```
1 item
3日前（2025-08-08, JST）のビュー数は2,052です。


プロパティID: properties/12345678

指標: screenPageViews
```

問題なく実行できました！GA4コンソールと同じ結果が出ていることも確認できました。

## （おまけ）GCP以外の場合の認証

今回はGCPで動かしているので認証周りが少し簡単になったと思います。

GCP以外でサーバーを使用している場合は、おそらく次のようにやれば動くと思います。試していないので保証はできませんが…

1. GCPコンソール上で：作成したサービスアカウントの鍵を発行（jsonファイルがローカルに保存される）
2. サーバー上で：jsonファイルを、local-filesフォルダ以下にアップロード
3. n8n上で：MCPサーバの認証情報に次のように指定。
    - Environments: `GOOGLE_APPLICATION_CREDENTIALS=/files/hoge.json`

多分セキュリティ上、local-filesじゃなくてもっと別の場所においた方がいいと思うけど。一度試してみるにはこれが最も簡単だと思います。







