+++
date = '2025-08-18T13:05:19+09:00'
lastmod = '2025-08-18T13:05:19+09:00'
draft = false
tags = ["Dify", "Firecrawl"]
category = "Howto"
subcategory = ""
title = 'クラウドのDifyでローカルのFirecrawlを使用する'
+++

## 概要

Difyは、AIアプリケーションを作成するためのノーコードツールです。

Firecrawlは、Webサイトをクロールしてデータを抽出するツールです。

そしてDifyは、Firecrawlを使用してWebサイトのデータを取得し、RAGを構築することができます。今回はここの設定についての解説です。

![Dify](/images/16/dify.png)

DifyもFirecrawlも、OSS版とSaaS版が提供されています。

その中でも今回は、**SaaS版のDify**で**OSS版のFirecrawl**を使用する場合の設定の仕方を記載します。

## 設定方法

### Firecrawlの設定

こちらのFirecrawlのリポジトリをクローンして使用します。

https://github.com/mendableai/firecrawl

```bash
git clone https://github.com/mendableai/firecrawl.git
```

ディレクトリを移動し、`.env.example`をコピーして`.env`を作成します。

```bash
cd firecrawl
cp ./apps/api/.env.example ./.env
```

.envファイルは以下の2行のみを編集します。

```bash
USE_DB_AUTHENTICATION=false
TEST_API_KEY=fc-test
```

- DBの認証は不要なので、`USE_DB_AUTHENTICATION`を`false`にします。
- APIキーは`fc-`から始まる文字であれば何でも可です。

次に、Dockerコンテナの作成を行います。

Docker Desktopを使用している場合は、アプリの起動を忘れずに行ってください。

```bash
docker compose up -d
```

アプリが動いているかを確認します。

`Hello, world!`と表示されれば、Firecrawlの設定は完了です。

```bash
curl -X GET http://localhost:3002/test
```



### Difyの設定

![config](/images/16/conf.png)

データソースの設定からFirecrawlの設定を行います。

APIキーには先ほどの`fc-test`と入力します。

Base URLには、もしDifyもローカルで動かしているのなら`http://host.docker.internal:3002`と設定すれば良いですが、SaaS版の場合は追加で設定が必要なので詳しくみていきましょう。

#### Base URLの設定

ローカルのFirecrawlを、Cloudflare Tunnelを使用して一時的に外部からHTTPSでアクセスできるURLにします。

まずはCloudflareをインストールします。

```bash
# Macの場合
brew install cloudflared
```

一時トンネルを張ります。

```bash
cloudflared tunnel --url http://localhost:3002
```

すると、`https://<ランダム>.trycloudflare.com`というURLが作成されます。

このURLをDifyのBase URLに設定すればOKです。

![config](/images/16/conf2.png)


これで接続することができます！

## 蛇足

もし常時同じURLを使用したい場合は、独自ドメインを使用して永続トンネルを張ることで実現できると思います。