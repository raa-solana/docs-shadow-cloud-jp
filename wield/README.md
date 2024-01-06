---
description: >-
  GenesysGo の Directed Acyclic Gossiping Graph Enabling Replication (D.A.G.G.E.R.)は、現在テストネット・フェーズ1を実施中で、オペレーターもテストに参加できるようになっています！
---

# Run a D.A.G.G.E.R. Wield Node

## Testnet Information:

[Testnet 1: Release Overview](https://www.shdwdrive.com/blog/dagger-testnet-release)

[D.A.G.G.E.R. Hammer: The magic behind a keypress](https://www.shdwdrive.com/blog/dagger-hammer-the-magic-behind-a-keypress)

[Testnet Phase 1 Learnings](https://www.shdwdrive.com/blog/dagger-testnet-phase-1-learnings)

[D.A.G.G.E.R. Hammer](https://dagger-hammer.shdwdrive.com/)

Wield Nodeを実行することで、[ここ](https://dagger-hammer.shdwdrive.com/)にあるshdwDrive用のHammerインターフェイスを動かすテストネットに信頼して参加することができます。Wieldノードの役割とD.A.G.G.E.R. Hammerの目的については、ノードオペレータの皆様には、当社のブログ記事をご覧いただくことをお勧めします。

Wield Nodeのオペレーターは、何千ものライブ・ユーザー・テスト・トランザクションを処理し、shdwDriveテスト・インターフェースにアップロードされたファイルの消去符号化と保存に必要なD.A.G.G.E.R.内の全モジュールをトラストレスに実行します。


## ノードの必要条件:

オペレーティングシステムの要件は、Ubuntu 22.04 LTSカーネル5.15.0です。その他のLinux x86ディストリビューションも動作する可能性がありますが、現時点では公式にはサポートされていません。

**Testnet Phase 1のD.A.G.E.R. Wield Nodeを動作させるための最小ハードウェア要件は以下のとおりです。**

* **16 CPU threads**
* **32 GB RAM**
* **データ・ストレージと高速I/O操作用の250GB SSD**
* **上下100mbpsのネットワーク接続が最低限必要です**

## Option 1 - ガイド付きインストール + スタートアップスクリプト


{% hint style="info" %}
どのように進めるかを決める前に、少なくとも以下の手動インストールの手順を確認することを強くお勧めします。基本的なLinuxコマンドに慣れており、ノードの設定に関与したい場合は、以下の「オプション2 - 手動インストール」で説明されている手順に従ってください。
{% endhint %}

よりガイド付きの体験をお望みなら、特別なインストーラー・スクリプトを作成しました：

* システムのハードウェアをチェックし、指定された最小ハードウェアを満たしていることを確認する。
* tcp/udp でのトラフィックフローを改善するためにカーネルをチューンを適用する。
* `wield`と`shdw-keygen`のバイナリをダウンロードする。
* 鍵ペアファイルを生成する。
* マシンスペックに基づいた設定ファイルとスタートアップスクリプトを生成する。
* システムサービスの生成と設定
* wield サービスを起動する。

そのためには、ターミナルから以下のコマンドを実行してください：

```sh
wget -O wield-installer.sh https://shdw-drive.genesysgo.net/4xdLyZZJzL883AbiZvgyWKf2q55gcZiMgMkDNQMnyFJC/wield-installer.sh && chmod +x wield-installer.sh && ./wield-installer.sh
```

上記のスクリプトに問題がある場合は、以下のオプション2をお試しください。

## Option 2 - マニュアルインストール

### 1. ノードの要件 - 16CPUスレッド、32GBのRAM、250GBのSSDストレージ、上下100Mbpsのネットワーク。オペレーティング・システムの要件は Ubuntu 22.04 LTS カーネル 5.15.0 です。その他のLinux x86ディストリビューションも動作する可能性がありますが、現時点では公式にはサポートされていません。

### 2. オペレーションシステム構成

オペレーティングシステムが最新であることを確認することから始めましょう：

```sh
sudo apt update && sudo apt upgrade -y
```

必要に応じてシステムを再起動し、カーネルが最新であることを確認します。

以下のカーネルチューニングパラメーターは、`/etc/sysctl.conf` を編集して以下の行をコンフィギュレーションファイルに追加し、`sudo sysctl -p` で新しいパラメーターを適用することで適用することを推奨します。注意: これらのパラメーターがあなたの特定のハードウェア構成に合っているか確認してください。

```sh
# set default and maximum socket buffer sizes to 12MB
net.core.rmem_default=12582912
net.core.wmem_default=12582912
net.core.rmem_max=12582912
net.core.wmem_max=12582912

# make changes for ulimit
fs.nr_open = 5000000
# set minimum, default, and maximum tcp buffer sizes (10k, 87.38k (linux default), 12M resp)
net.ipv4.tcp_rmem=10240 87380 12582912
net.ipv4.tcp_wmem=10240 87380 12582912

# Enable TCP westwood for kernels greater than or equal to 2.6.13
net.ipv4.tcp_congestion_control=westwood
net.ipv4.tcp_fastopen=3
net.ipv4.tcp_timestamps=0
net.ipv4.tcp_sack=1
net.ipv4.tcp_low_latency=1
# don't cache ssthresh from previous connection
net.ipv4.tcp_no_metrics_save = 1
net.ipv4.tcp_moderate_rcvbuf = 1

# kernel Tunes
kernel.timer_migration=0
kernel.hung_task_timeout_secs=30
# A suggested value for pid_max is 1024 * <# of cpu cores/threads in system>
kernel.pid_max=16384

# vm.tuning
vm.swappiness=30
vm.max_map_count=1000000
vm.stat_interval=10
vm.dirty_ratio=40
vm.dirty_background_ratio=10
vm.min_free_kbytes = 3000000
vm.dirty_expire_centisecs=36000
vm.dirty_writeback_centisecs=3000
vm.dirtytime_expire_seconds=43200
```

`etc/security/limits.conf`を編集し、コンフィギュレーションファイルの一番下に以下の行を追加することで、開いているファイルディスクリプタの最大値（`ulimit`）をハードリミットの最大値である`5000000`よりも増やすことが推奨します（変更を有効にするには、ログアウトして再度ログインする必要があります）。

```sh
*               soft    nofile          5000000
*               hard    nofile          5000000
```

### 3. ノードの初期設定:

まだ実行していない場合は、アプリケーションを実行する専用のユーザを作成することをお勧めします。この場合、 `sudo adduser dagger` で `dagger` ユーザを作成し（任意のパスワードを作成）、 `sudo usermod -aG sudo dagger` で `dagger` ユーザを `sudo` ユーザグループに追加します。sudo su - dagger` で `dagger` ユーザに切り替えます。残りのタスクはすべて `dagger` ユーザーとして実行します。

Wieldのバイナリを`dagger`ユーザディレクトリにダウンロード：

```sh
wget -O ~/wield https://shdw-drive.genesysgo.net/4xdLyZZJzL883AbiZvgyWKf2q55gcZiMgMkDNQMnyFJC/wield-latest
```

Wieldのバイナリを実行可能にします:

```sh
sudo chmod +x ~/wield
```

Shdw-Keygenユーティリティを`dagger`ユーザディレクトリにダウンロードします：

```sh
wget -O ~/shdw-keygen https://shdw-drive.genesysgo.net/4xdLyZZJzL883AbiZvgyWKf2q55gcZiMgMkDNQMnyFJC/shdw-keygen-latest
```

Shdw-Keygen ユーティリティを実行可能にします:

```sh
sudo chmod +x shdw-keygen
```

Shdw-Keygenユーティリティを使用して新しい一意のキーペアIDを作成し、一意のシードフレーズを書き留め、id.jsonファイルを別の場所にバックアップします：

```sh
./shdw-keygen new -o ~/id.json
```
**重要：シードフレーズとid.jsonキーペアファイルを自分のノード以外の安全な場所にバックアップする必要があります！ノードのパフォーマンスと報酬はこのIDに関連しています。ノードを再インストールする必要がある場合は、このバックアップから復元することで、ノードIDの継続性を維持することができます。キーペアを紛失した場合、復旧のお手伝いはできません。**

{% hint style="danger" %}
**重要：シードフレーズとid.jsonキーペアファイルを自分のノード以外の安全な場所にバックアップする必要があります！ノードのパフォーマンスと報酬はこのIDに関連しています。ノードを再インストールする必要がある場合は、このバックアップから復元することで、ノードIDの継続性を維持することができます。キーペアを紛失した場合、復旧のお手伝いはできません。**

**ノードを再インストールまたは更新する必要がある場合は、新しいキーペアを作成せず、元の `id.json` ファイルを保持またはコピーしてください。**
{% endhint %}

pubkey を表示するには、以下のコマンドを実行します：

```sh
./shdw-keygen pubkey ~/id.json
```

`nano config.toml` で設定ファイルを作成し、以下の内容を貼り付けます。

```toml
trusted_nodes = ["184.154.98.116:2030", "184.154.98.117:2030", "184.154.98.118:2030", "184.154.98.119:2030", "184.154.98.120:2030"]
dagger = "JoinAndRetrieve"

[node_config]
socket = 2030
keypair_file = "id.json"

[storage]
peers_db = "dbs/peers.db"
```

`nano start_wield.sh`でWield起動スクリプトを作成し、以下の内容をファイルに貼り付けます。注意：これらのパラメータは16スレッドプロセッサーに基づいています。`processor-threads`と`--global-threads`はマシンの総スレッド数と同じに設定し、`--comms-threads`は`2`に設定することを推奨します。また、`--comms-threads` は `2` に設定しておくことを推奨します。異なるハードウェアでのパラメータの調整については、`wield --help` の出力を参照してください：

```bash
#!/bin/bash
PATH=/home/dagger
exec wield \
--processor-threads 16 \
--global-threads 16 \
--comms-threads 2 \
--log-level info \
--history-db-path /mnt/dag/historydb \
--config-toml config.toml \
```

スクリプトを `sudo chmod +x start_wield.sh` で実行可能にします。

少なくとも 200GB の空き容量があるディスクに `historydb` を保存する場所を作成します（ディスクの準備とマウントは、ハードウェアの構成によって多くの変数があるため、このドキュメントの範囲を超えていますが、Google や ChatGPT でソートできるはずです）。この場所は `start_wield.sh` 起動スクリプトの `--history-db-path` フラグで指定された場所と一致しなければなりません。我々の場合、予備ディスクが `/mnt/dag` にマウントされ、そこに `historydb` ディレクトリが作成されます：

```sh
sudo mkdir -p /mnt/dag/historydb
```

`historydb` ロケーションの所有者を `dagger` ユーザに変更します：

```sh
sudo chown -R dagger:dagger /mnt/dag/*
```

また、`wield`バイナリと同じ場所に`snapshots`ディレクトリが必要です。ここまでたどってきたのであれば、これはあなたのホームディレクトリであるはずです。以下のようにして作ることができます：

```sh
mkdir ~/snapshots
```

`sudo nano /etc/systemd/system/wield.service`で`wield`用のシステムサービスを作成し、以下の内容をファイルに貼り付けます：

```sh
[Unit]
Description=DAGGER Wield Service
After=network.target

[Service]
User=dagger
WorkingDirectory=/home/dagger
ExecStart=/home/dagger/start_wield.sh
Restart=always

[Install]
WantedBy=multi-user.target
```

サービスを登録し、 `wield` プロセスを開始します：

```sh
sudo systemctl enable --now wield.service
```

`tail -f config.log`でログをtailingして、適切な動作を確認してください。ソフトウェアが初期化される前に様々なスタートアップタスクがバックグラウンドで実行されるため、Wieldがログファイルに書き込みを開始するまでに時間がかかる場合があります。`tail -f config.log | grep "finalized"`でバンドルがファイナライズされているかログファイルをチェックすることで適切なノードの動作を確認できます。

ログファイルが大きくなりすぎないように、`sudo nano /etc/logrotate.d/wield.conf`で`wield`用の`logrotate`エントリを追加し、以下の内容をファイルに貼り付けることを推奨します。

```/home/dagger/config.log
/home/dagger/config.log {
    su dagger dagger
    daily
    rotate 5
    size 10M
    missingok
    copytruncate
    delaycompress
    compress
}
```

`sudo systemctl restart logrotate` で `logrotate` サービスを再起動し、`sudo logrotate -d /etc/logrotate.d/wield.conf` で `wield.conf` 設定が正しく機能していることを確認し、エラーがないかチェックします。

`sudo logrotate -d /etc/logrotate.d/rsyslog`を実行し、エラーがないかチェックして修正することで、`syslog`ログファイルが適切にローテーションするように設定されていることを確認することもできます。例として、`/etc/logrotate.d/rsyslog`に使用した設定は以下の通りです。

```
/var/log/syslog
/var/log/mail.info
/var/log/mail.warn
/var/log/mail.err
/var/log/mail.log
/var/log/daemon.log
/var/log/kern.log
/var/log/auth.log
/var/log/user.log
/var/log/lpr.log
/var/log/cron.log
/var/log/debug
/var/log/messages
{
        su syslog syslog
	rotate 4
	weekly
	missingok
	notifempty
	compress
	delaycompress
	sharedscripts
	postrotate
		/usr/lib/rsyslog/rsyslog-rotate
	endscript
}
```

### **4. ノード メンテナンス**

通常のネットワーク運用中にノードのメンテナンスを行う必要がある場合は、D.A.G.G.E.R.エポックを5回分待ってからネットワークに再参加してください。D.A.G.G.E.R.の進行状況はこちらで確認できます：[https://dagger-hammer.shdwdrive.com/explorer](https://dagger-hammer.shdwdrive.com/explorer)

ノードを停止するには、`sudo systemctl stop wield`を使用すします。この時点で、現在のバイナリをダウンロードするだけで `wield` を最新バージョンにアップグレードするなど、必要なメンテナンスを行うことができます：

```
wget -O ~/wield https://shdw-drive.genesysgo.net/4xdLyZZJzL883AbiZvgyWKf2q55gcZiMgMkDNQMnyFJC/wield-latest
```

厳密には必要ではありませんが、最新の `shdw-keygen` ユーティリティにもアップグレードすることをお勧めします。

```
wget -O ~/shdw-keygen https://shdw-drive.genesysgo.net/4xdLyZZJzL883AbiZvgyWKf2q55gcZiMgMkDNQMnyFJC/shdw-keygen-latest
```

`wield` のアップグレードが必要なだけであれば、5エポック後に `sudo systemctl start wield` でノードを再起動できます。

クラスタの完全な再起動が必要な場合は、適切な Discord チャンネルの指示に従うことをお勧めします。

\*免責事項：GenesysGoのD.A.G.G.E.R. Testnet Phase 1でWield Nodeを操作することにより、利用者は自発的かつ自己責任でこれを行うことを認めます。GenesysGoはTestnetソフトウェアを「現状のまま」無保証で提供し、お客様が被る可能性のある直接的、間接的、偶発的、結果的損害について一切の責任を負いません。お客様は、ご自身のシステムおよびデータのセキュリティに責任を負うものとします。GenesysGoは、お客様がTestnetに参加した結果生じたいかなる損失や損害についても責任を負いません。本ソフトウェアを使用することにより、お客様は、お客様のノード操作に関連するいかなるクレームや紛争からもGenesysGoを免責することに同意するものとします。本契約は、Wieldノードをダウンロードし、操作した時点で拘束力を持ちます。GenesysGoは予告なくTestnetを変更または中止することがあります。これにはTestnetフェーズ1のD.A.G.G.E.R. Wield Nodeを操作するためのハードウェア要件が含まれますが、これに限定されるものではありません。これらの条件に同意できない場合は、Testnetに参加しないでください。

### ここからどこへ行くのか？

さて、ノードを立ち上げて稼動させたところで、ノードのパフォーマンスを表示・監視するための監視スタックの設定方法について、[monitoring-stack.md](monitoring-stack.md "mention")をチェックしましょう。

### D.A.G.G.E.R. テストネットと Wield ノードに関するよくある質問（FAQ） 

## **Q: D.A.G.G.E.R.のロードマップはどこで見ることができますか？**

A: ロードマップは公式ブログに掲載されており、今後のテストネット・フェーズの計画がまとめられています。

## **Q: テストネットのフェーズ数はいくつで、メインネットの開始予定日はいつですか？**

A: テストネットのフェーズ数とメインネットの開始日は未定です。チームは期限を決める前に、安定性とパフォーマンスを確保することに注力しています。

## **Q: Wield Nodeを実行するサーバーはどこで借りられますか？**

A: ほとんどのクラウドプロバイダーがWield Nodeの実行に適したインスタンスを提供しています。Digital Oceanは比較のための確かな基準ですが、プロバイダーがノードを実行するためのシステム要件を満たしていることを常に確認してください。

## **Q: Linuxを使ったことがないのですが、Wield Nodeをセットアップできますか？**

A: はい、YouTubeのチュートリアルを見たり、ChatGPTのようなLinuxガイダンスのリソースを使うなど、学ぶ意欲とリサーチがあれば、ノードをセットアップすることができます。また、コミュニティやエンジニアリング・チームも質問を受け付けています。

## **Q: Wieldノードをセットアップするために必要なハードウェアは何ですか？**

A: 推奨されるシステム要件は、AWS EC2 c5.4xlargeインスタンスまたは同等のもので、16 vCPU、32GB RAM、Ubuntu 22.04 LTS、カーネルバージョン5.15.0以降です。あるいは、古いゲームマシンのパーツを使用することもできますし、これらの最低要件を満たすセットアップを行うこともできます。

## **Q: 仮想マシンでWield Nodeを実行できますか？**

A: はい、Wield NodeはVirtualBoxやVMwareのような仮想マシンや様々なクラウドプラットフォームでテストされています。最適なパフォーマンスを得るために、仮想マシンが推奨されるシステム要件を満たしていることをご確認ください。

## **Q: Ubuntu 20.04から22.04へのアップグレードは安全ですか？**

A: はい、多くのユーザーが Ubuntu 20.04 から 22.04 へのインプレースアップグレードに成功しています。

## **Q: Wieldノードのエラーや問題をどのようにトラブルシューティングできますか？**

A: トラブルシューティングについては、システムログをチェックし、ノードの設定ファイルにエラーがないか確認し、すべての依存関係が正しくインストールされていることを確認してください。また、特定のエラーメッセージやログを共有することで、Discord コミュニティのサポートチャンネルに助けを求めることもできます。

## **Q: Wieldノードのログファイルはどのように確認できますか？**

A: ログファイルは `cat`、`less`、`tail`、`grep` などのコマンドラインツールで確認できます。具体的なログファイルの場所はノードの設定によって異なります。systemd のサービスログをチェックするには `journalctl` を使ってください、例えば `sudo journalctl -u wield.service` とします。

## **Q: Wield Nodeのステータスとログファイルはどのように確認できますか？**

A: `cat`、`less`、`tail`、`grep`などのコマンドラインツールを使ってログファイルをチェックできます。具体的なログファイルの場所はノードの設定によって異なります。systemd のサービスログをチェックするには `journalctl` を使ってください。例えば、`sudo journalctl -u wield.service`。コマンド `sudo systemctl status wield` を実行してノードのステータスを確認することができます。これでノードの現在の稼働状況の概要が分かります。コマンド `tail -f config.log | grep "finalized"` を実行すると、最終化プロセスを監視できます。このコマンドはログエントリーをフィルタリングして、"finalized" に言及している行だけを表示するので、最終化プロセスをリアルタイムで追跡することができます。

## **Q: サーバーのログファイルはどこにありますか?**

A: ログファイルは通常`/home/dagger/config.log`にあります。

## **Q: ノードIDはどうやって見つけるのですか?**

A: `shdw-keygen`プログラムをダウンロードしたと仮定して、`home/dagger`ディレクトリで`./shdw-keygen pubkey id.json`を実行してください。

## **Q: ノードのバージョンを確認し、必要に応じてアップグレードするにはどうすればよいですか?**

A: ノードのバージョンは `./wield --version` で確認できます。アップグレードするには、対話型インストーラースクリプトを使います。

## **Q: どうすれば最新版にアップデートできますか？**

A: ノードのサービスを停止し、最新のバイナリをダウンロードした後、 サービスを再起動することでアップデートできます。詳しいコマンドはユーザがチャットで教えてくれます。

## **Q: Wieldノードのエラーや問題をどのようにトラブルシューティングできますか？**

A: トラブルシューティングについては、システムログをチェックし、ノードの設定ファイルにエラーがないか確認し、すべての依存関係が正しくインストールされていることを確認してください。また、特定のエラーメッセージやログを共有することで、Discord コミュニティのサポートチャンネルに助けを求めることもできます。

## **Q:サーバーのシステム使用量を確認するにはどうすればよいですか?**

A: Ubuntuサーバーでは、Ubuntuシステムモニターを開くか、`htop`、`top`、`vmstat`などのコマンドラインツールを使ってシステムの使用状況を確認できます。これらのツールは、CPU、メモリ、ディスクの使用状況に関する情報を提供します。

## **Q: Wieldノードのログファイルはどのように確認できますか？**

A: ログファイルは `cat`、`less`、`tail`、`grep` などのコマンドラインツールで確認できます。具体的なログファイルの場所はノードの設定によって異なります。systemd のサービスログをチェックするには `journalctl` を使ってください、例えば `sudo journalctl -u wield.service` とします。

## **Q: 1エポックの長さはどれくらいですか？**

A: エポックは現在64バンドルです。

## **Q: VPS上でノードを起動した後、ターミナルウィンドウを閉じることはできますか？**

A: はい、`systemd`のようなサービスマネージャーを使ってノードを実行している場合は、ターミナルウィンドウを閉じることができます。サービスはバックグラウンドで実行され続けます。ノードの状態を確認するには、`systemctl status <サービス名>`を使ってください。

## **Q: ノードの起動時に 'Resource temporarily unavailable' エラーに遭遇した場合はどうすればよいですか？**

A: このエラーは通常、データベースファイルなどの必要なリソースが他のプロセスによって使用されているためにロックされていることを意味します。ノードの他のインスタンスが実行されていないことを確認してください。手動で起動する前に、実行中のサービスを停止する必要があるかもしれません。

## **Q: より多くのメモリやCPUリソースを使用するようにノードを設定する方法はありますか？**

A: ノードソフトウェアはシステムで利用可能な限界まで必要なリソースを自動的に使用します。リソースの使用量が少ないと感じられる場合は、ネットワークのアイドル時間やノードソフトウェアの非効率性が原因である可能性があり、将来のアップデートで対処される可能性があります。

## **Q: ログに無効なエポックに関するエラーが表示された場合、どうすればよいですか？**

A: これはピアまたはあなたのノードが非常に遅れていることを示している可能性があります。ノードが最新で、正しく再起動されていることを確認してください。ノードを再度起動する前に、数エポック待つ必要があるかもしれません。

## **Q: ハンドシェイクエラーや最大ハンドシェイク時間超過のメッセージが表示された場合はどうすればよいですか?**

A: `config.toml` に正しい信頼できるピアの値を設定し、ノードを再起動する前に、更新後5エポックを完全に待ったことを確認してください。

## **Q: 誤って自分のノードの2つ目のインスタンスを起動しようとするとどうなりますか？**

A: 1つのインスタンスがすでに起動している状態で2つ目のインスタンスを起動しようとすると、リソースのロックやポートの競合に関連するエラーが発生する可能性があります。常に1つのインスタンスのみが実行されているようにしてください。

## **Q: 既に他のノードをホストしているサーバーにWield Nodeをデプロイできますか？**

A: 可能ですが、パフォーマンスに影響を与えることなく両方のノードを処理するのに十分なリソースがサーバーにあることを確認してください。リソースの使用状況を注意深くモニターしてください。

## **Q: ネットワーク上のエポックの進行状況をモニターするにはどうすればいいですか？**

A: ノードのログやD.A.G.G.E.R.チームが提供するネットワーク監視ツールを使って、エポックの進行状況を監視することができます。

## **Q: ネットワークが遅くなったり、進行していないことに気づいたら、どうすればよいですか？**

A: ネットワークの速度低下は様々な理由で起こり得ます。D.A.G.G.E.R.チームからの公式アナウンスで、ネットワークの状態や取るべき対処法についての情報を常に入手するようにしてください。

## **Q: ノードのセットアップやキーペアの生成時にGLIBC not foundというエラーが発生した場合はどうすればよいですか？**

A: このエラーは通常、UbuntuまたはLinuxカーネルのバージョンが正しくないことを示しています。カーネル5.15.0以降のUbuntu 22.04を使用していることを確認してください。システムのアップデート（`sudo apt update`と`sudo apt upgrade`）を行い、場合によってはマシンを再起動する必要があるかもしれません。

## **Q: ターミナルを閉じた後もノードが生きていることを確認するにはどうしたらいいですか？**

A: ターミナルを閉じた後もノードがアクティブであることを確認するには、`systemd` や `screen` を使ってバックグラウンドサービスとして実行してください。こうすることで、プロセスがターミナルセッションに縛られることがなくなります。

## **Q: ノードのリソース使用量の制限を増やすにはどうしたらいいですか？**

A: リソース使用量の限界は一般的にシステムの能力とノードのソフトウェアによって定義されます。チューニングを可能にする設定オプションがあるかもしれませんが、不安定にならないように注意して使用してください。

## **Q: テストネットのアナウンスやアップデートはどこで見ることができますか？**

A: テストネットのアナウンスやアップデートは通常、D.A.G.G.E.R.チームが提供する公式チャンネルに投稿されます。最新情報はこれらのチャンネルでご確認ください。

## **Q: ノード運営の収益性はどのように判断すればよいですか？**

A: 採算性は報酬体系、運用コスト、ネットワーク・パフォーマンスなど様々な要因によって異なります。ダウンタイムに対する手当やペナルティを含むオペレーターの報酬に関する詳細は、テストネットの進行に伴いD.A.G.G.E.R.チームから提供されます。

## **Q: 聞き慣れない用語に遭遇したり、さらなる支援が必要な場合はどうすればよいですか？**

A: Discordのサポート・チャンネルで説明や支援を求めることをためらわないでください。コミュニティとコア・エンジニアリング・チームがあなたをサポートします。

## **Q: サーバーのログファイルを共有するにはどうすればいいですか？**

A: `scp` コマンドなどを使ってサーバーからログファイルをダウンロードし、共有サービスにアップロードすることができます。
