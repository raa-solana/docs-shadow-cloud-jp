---
description: >-
  The current testnet phase now allows for trustless shdwOperator to join, help
  test, and earn rewards!
---

# Operate

<table data-view="cards"><thead><tr><th></th><th align="center"></th><th align="center"></th><th data-hidden data-card-target data-type="content-ref"></th><th data-hidden data-card-cover data-type="files"></th></tr></thead><tbody><tr><td></td><td align="center"><strong>インストール</strong></td><td align="center"></td><td><a href="install.md">install.md</a></td><td><a href="../.gitbook/assets/SHDW-15.png">SHDW-15.png</a></td></tr><tr><td></td><td align="center"><strong>ウォレット</strong></td><td align="center"></td><td><a href="wallet.md">wallet.md</a></td><td><a href="../.gitbook/assets/SHDW-16.png">SHDW-16.png</a></td></tr><tr><td></td><td align="center"><strong>モニター</strong></td><td align="center"></td><td><a href="monitoring-stack.md">monitoring-stack.md</a></td><td><a href="../.gitbook/assets/SHDW-17.png">SHDW-17.png</a></td></tr><tr><td></td><td align="center">FAQ</td><td align="center"></td><td><a href="node-faq.md">node-faq.md</a></td><td><a href="../.gitbook/assets/SHDW-19.png">SHDW-19.png</a></td></tr></tbody></table>

## shdwNodeを操作するには、次のことが必要です：

1. GenesysGoのDirected Acyclic Gossiping Graph Enabling Replication (D.A.G.G.E.R.)の最新バージョンを[インストール](https://docs.shdwdrive.com/operate/install.md) 
2. ウォレットの[作成](https://docs.shdwdrive.com/operate/wallet.md)
3. あなたのDiscordユーザーが接続されたshdwNodeウォレットで100 SHDWを取得します。[ノートを読む](https://docs.shdwdrive.com/operate/node-faq.md#q-how-are-earnings-paid-out-and-how-do-i-claim-my-shdwoperator-earnings)
4. ネットワークで[認証](https://docs.shdwdrive.com/operate#discord-verification)を行う（リワード獲得に必要）
5. あなたのshdwNodeを[監視](https://docs.shdwdrive.com/operate/monitoring-stack.md)し、維持し、アナウンスの最新情報を入手しましょう。[リーダーボードを見る](https://testnet.shdwdrive.com/uptime-leaderboard)
6. 現在のテストネットについては[FAQ](node-faq.md#testnet-2)をお読みください
7. リワードを[獲得](https://testnet.shdwdrive.com/operator-rewards)

{% hint style="warning" %}
**重要 - あなたのノードが報酬の対象となる前に、あなたのDiscordユーザーにノードIDを追加する必要があります。それが完了すると、報酬があなたのノードに蓄積され始めます。**

**Discordの認証システムで認証されたすべてのウォレットで最低100 SHDWを保有する必要があります。**
{% endhint %}

## テストネットのご案内:

[Testnet 2: リリース概要](https://www.shdwdrive.com/blog/shdwdrive-v2-incentivized-testnet)

[Testnet 1: リリース概要](https://www.shdwdrive.com/blog/dagger-testnet-release)

[D.A.G.G.E.R. Hammer](https://dagger-hammer.shdwdrive.com/)

shdwNodeを運営することで、[ここ](https://dagger-hammer.shdwdrive.com/)にあるshdwDriveのHammerインターフェイスを動かすテストネットにトラストレスに参加することができます。shdwOperatorsの皆さんには、私たちのブログ記事やDiscordチャンネルで、その役割の全容を確認することをお勧めします。

ShdwNodesは、何千ものライブ・ユーザー・テスト・トランザクションを処理し、shdwDrive v2のコンセンサスを形成するD.A.G.G.E.Rプロトコル内のすべてのモジュールをトラストレスに実行します。shdwNodesによって実行される分散型コンセンサスの詳細については、[litepaper](https://github.com/GenesysGo/dagger-litepaper/blob/main/DAGGER-Litepaper.pdf)と[blogs](https://www.shdwdrive.com/blog)をご覧ください。

（訳注：[ブログの日本語訳はこちら](https://note.com/raa_solana/)）

## ハードウェア要件:
**動作環境はUbuntu 22.04 LTS kernel 5.15.0です。その他のLinux x86ディストリビューションも動作する可能性がありますが、現時点では公式にはサポートされていません。**

**動作環境はUbuntu 22.04 LTS kernel 5.15.0です。他のLinux x86ディストリビューションも動作するかもしれませんが、現時点では \_公式には\_ サポートされていません。**

**テストネット用shdwNodeを動作させるための最小ハードウェア要件は以下の通りです:**

* **16 CPU threads (8 cores)**
* **32 GB RAM**
* **データ・ストレージと高速I/O操作用に2 TB SSD（またはNVMe）を搭載**
* **上下100mbpsのネットワーク接続が最低限必要**

{% hint style="info" %}
\*\*注：2 TB未満のSSDやNVMeを使用してもshdwNodeの実行を妨げることはありませんが、testnet 2中にファイル・トランザクション・ボリュームを増やすと、より小さなストレージ・ドライブがより早くいっぱいになり、より頻繁な再起動が必要になります。現在では2 TB以上のストレージ・ドライブを推奨しています。
{% endhint %}

## Discord認証 

ノードIDをDiscordユーザーに接続したい場合は、このガイドに従ってください。これにより、GenesysGo DiscordサーバーのshdwOperatorロールとwield operatorsチャンネルのロックが解除されます。

* [このガイド](https://docs.shdwdrive.com/operate/wallet#importing-usdshdw-accounts-into-solana-wallets)に従って、shdwNodeのIDキーペアをSolana UIウォレットにインポートしてください。
* [https://holders.genesysgo.com/](https://holders.genesysgo.com/) にアクセスしてください。
* あなたのshdwNode IDを接続したいDiscordユーザーでログインしてください。
* ウォレットを接続し、"Add wallet"をクリックし、署名してメッセージを送信します。
* 数秒後、リクエストが送信されたという通知が表示され、ウォレットのリストにあなたのshdwNodeウォレットが表示されるはずです。ロールの更新がDiscordに反映されるまで数分かかる場合があります。
