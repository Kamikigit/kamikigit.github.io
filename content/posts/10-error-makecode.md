+++
date = '2025-08-13T19:26:20+09:00'
draft = false
tags = ["マインクラフト"]
category = "Debug"
subcategory = ""
title = '教育版Minecraftでmakecodeを開こうとした時のエラーの対処法'
+++

![エラー](/images/10/makecode-error.png)

マインクラフトEducationを使用してMakecodeを開こうとした際に、「インターネットアクセスを確認し、チュートリアルが有効であることを確認してください。」というエラーが出て開けないことがあります。

この場合、**エディタをリセット**すると解決する可能性があります。

## エラー解決手順

![設定](/images/10/settings.png)

makecode右上の歯車マークをクリックして、「リセット」を押します。

![ダイアログ](/images/10/dialog.png)

本当に削除しますか？という確認が出ます。

ここに記載されている通り、エディタをリセットすると**これまで作成されたコードが全て消えてしまいます。**

それでも良ければ「リセット」を押します。

ブラウザが再起動され、開くことができます。


## なぜエディタリセットで直るのか

Makecodeエディタのキャッシュの問題なのかと思います。

同じチュートリアルを複数回開くと起こりうるエラーです。


```typescript
export async function resetAsync() {
    allScripts = []

    await impl.resetAsync();
    await cloudsync.resetAsync();
    await db.destroyAsync();
    await pxt.BrowserUtils.clearTranslationDbAsync();
    await pxt.BrowserUtils.clearTutorialInfoDbAsync();
    await compiler.clearApiInfoDbAsync();
    pxt.storage.clearLocal();
    data.clearCache();

    // keep local token (localhost and electron) on reset
    if (Cloud.localToken) {
        pxt.storage.setLocal("local_token", Cloud.localToken);
    }

    await syncAsync(); // sync again to notify other tabs
}
```
引用元： https://github.com/microsoft/pxt/blob/master/webapp/src/workspace.ts

このやり方だとこれまでに作成したプロジェクトが全て消えてしまうという問題があるのでなるべく実行したくないのですが、他の解決策は今のところ見つかっていません。

PXTのリポジトリをフォークしてキャッシュのみクリアできるようにコードを改変すれば確実ですが、手間がかかってしまうのであまり理想的ではないです。

すでに開いたチュートリアルファイルを個別で削除するのも効果的だと思いますが、ファイルが膨大だと探すのが手間になります。

あとは可能性として、チュートリアルのファイルの設定からキャッシュを無効にするよう変更できれば解決するかもしれません。


## そのほかの解決方法

https://edusupport.minecraft.net/hc/en-us/community/posts/360077766912-Internet-Connection-Invalid-Tutorial?utm_source=chatgpt.com

こちらに記載されている通り、ネットワークやウイルスソフトによる問題の可能性もあります。

https://makecode.com/writing-docs/tutorials/troubleshooting?utm_source=chatgpt.com

また、チュートリアルファイルが大きすぎる、コードが間違っている場合も「インターネットアクセスを確認し、チュートリアルが有効であることを確認してください。」のエラーが出る可能性があります。

私の場合は、ネットワーク接続やチュートリアルのファイルについて、いずれも問題ありませんでした。その場合はやはりキャッシュが原因であり、エディタリセットが有効なのかなと思います。


