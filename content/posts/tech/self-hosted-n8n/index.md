+++
date = '2025-07-26T14:45:18+09:00'
draft = false
tags = ["n8n"]
category = "Howto"
subcategory = ""
title = 'n8nをほぼ0円で使う。GCEを使ったセルフホストのやり方'
+++

n8nのセルフホストをGCPで行う場合、Cloud RunやKubernetesを使う方法もありますが、低コストかつ常時稼働させたいのでGoogle Compute Engineを使用していきます。

セルフホストを低コストで運営するポイント：
- GCEの無料枠で運営
- ドメインはサブドメを取得する
- SSL認証はLet's Encryptを使用する

ではやり方の説明に入ります。

## GCEでVMを作成する

無料枠で運営したいので、e2-microのインスタンスを作成します。今のところe2-microでも問題なく動作しています。

詳しい設定は主に <a href="https://qiita.com/muhi111/items/24b881975f98e0409ef7" target="_blank" rel="noopener noreferrer">こちらの記事</a> を参考にさせていただきました。

設定のポイント：
- インスタンス名は分かりやすく「n8n」とします。
- 外部IPアドレスをエフェメラル（可変）から静的IP（固定）に変更します。これをしないと、外部IPアドレスがインスタンスの停止・再起動時に変わってしまい面倒です。

アドレスだけ取得してVMインスタンスを割り当てないとそこそこの額の請求が来てしまうので、インスタンスを削除したらIPアドレスを解放することを忘れずにすること。

## SSH接続
ローカルのターミナルからGCEへSSH接続します。
SSH接続のやり方はいくつかあります。（参考：<a href="https://qiita.com/hollyhock-s/items/791b662bbf816df2f784" target="_blank" rel="noopener noreferrer">こちらの記事</a>）

私はgcloudを使用しているため、以下の方法で接続します。(gcloudのインストールは省略)

1. gcloudへのログイン
```
gcloud auth login
```

2. SSH接続

```
gcloud compute ssh --zone "{zone}" "n8n" --project "{project_name}"
```

このコマンドは、VMインスタンスの「接続」>「gcloudコマンドを表示」からコピペできます。

![gcloud](/images/11/gcloud.png)



## ドメインとDNS設定

### ドメイン取得
HTTP接続を行いたいのでドメインを取得します。これはお名前.comやCloudflareなどのドメイン取得サービスを使用します。

私は普段使用しているドメインのn8nというサブドメを取得しました。例として、n8n.example.comとします。

### DNS設定
ドメイン取得サービス内で、AレコードのIPアドレスの値を、GCEの外部IPアドレスに変更します。

![external-ip](/images/11/externalIP.png)

### SSL認証
後述するDocker composeファイルにTraefikが使用されておりSSL認証が可能なので、ここで特に設定することはありません。


## n8nのインストール


基本的に<a href="https://docs.n8n.io/hosting/installation/server-setups/docker-compose/" target="_blank" rel="noopener noreferrer">公式ドキュメント</a>
に従って、Dockerを用いてセットアップを行います。

まず1のDockerとDocker Composeのインストールを行います。
コマンドは公式ドキュメントと同じなので省略します。


次に、.envファイルを作成します。
```
# DOMAIN_NAME and SUBDOMAIN together determine where n8n will be reachable from
# The top level domain to serve from
DOMAIN_NAME=example.com
SUBDOMAIN=n8n

GENERIC_TIMEZONE=Asia/Tokyo

# The email address to use for the TLS/SSL certificate creation
SSL_EMAIL=hoge@example.com
```

この`SSL_EMAIL`は、普段使ってるやつを入力すればOKです。

あとは公式ドキュメント通りにDokcer composeファイルを作成して、`sudo docker compose up -d`するだけです。
簡単！

ちなみにインスタンス内の階層構造はこうなっています。
```bash
~
└── n8n-compose/
    ├── .env
    ├── docker-compose.yaml
    └── local-files/
```

## n8nの設定

https://n8n.example.com/にアクセスすると、最初にn8nの登録画面が出るので、アカウントの登録を行います。

ログインすると“get paid features for free (forever)”というポップアップが出てくるので、ライセンスキーを受け取ります。
（スクショ忘れました…）

これは、セルフホスト向けに一部の有料機能が一生無料で提供されているようです。（すごい！）

以下のような機能が開放されます：
- エディター内のデバッグ機能
- バージョンの確認・復元
- フォルダ管理
- 実行履歴の詳細な検索


![license](/images/11/license.png)

ライセンスキーはメールで配布されるので、Active License Keyのボタンを押してライセンスを登録すれば完了です。

## 余談

ちなみにn8nの公式ドキュメントだと、GCPでは<a href="https://docs.n8n.io/hosting/installation/server-setups/google-cloud/" target="_blank" rel="noopener noreferrer">Kubernetesを使用したセルフホスト</a>のやり方が紹介されています。





