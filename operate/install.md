# Install

### Option 1 - ガイド付きインストール + startup script

{% hint style="info" %}
どのように進めるかを決める前に、少なくとも以下の手動インストールの手順を確認することを強くお勧めします。基本的なLinuxコマンドに慣れており、ノードの設定に関与したい場合は、以下の「オプション2 - 手動インストール」で説明されている手順に従ってください。
{% endhint %}

よりガイド付きの体験をお望みなら、特別なインストーラー・スクリプトを作成しました。

* システムのハードウェアをチェックし、指定された最小ハードウェアを満たしていることを確認します
* tcp/udp でのトラフィックフローを改善するためのカーネルチューンを適用します
* `wield` と `shdw-keygen` のバイナリをダウンロードします
* 鍵ペアファイルを生成します
* マシンスペックに基づいた設定ファイルとスタートアップスクリプトを生成します
* システムサービスの生成と設定します
* wield サービスを起動します

そのために、ターミナルから以下のコマンドを実行します：

```sh
wget -O wield-installer.sh https://shdw-drive.genesysgo.net/4xdLyZZJzL883AbiZvgyWKf2q55gcZiMgMkDNQMnyFJC/wield-installer.sh && chmod +x wield-installer.sh && ./wield-installer.sh
```

上記のスクリプトに問題がある場合は、以下のオプション2をお試しください。

### Option 2 - 手動インストール

#### 1. ノード要件 - 16 CPUスレッド（8コア）、32 GBのRAM、2 TBのSSDストレージ、上下100 Mbpsのネットワーク。オペレーティング・システムの要件はUbuntu 22.04 LTS kernel 5.15.0です。その他のLinux x86ディストリビューションも動作する可能性がありますが、現時点では正式にサポートされていません。

#### 2. オペレーティングシステム構成.

まず、オペレーティング・システムが最新であることを確認することから始めましょう：

```sh
sudo apt update && sudo apt upgrade -y
```

必要に応じてシステムを再起動し、カーネルが最新であることを確認してください。

以下のカーネルチューニングパラメーターは、`/etc/sysctl.conf` を編集して以下の行をコンフィギュレーションファイルに追加し、`sudo sysctl -p` で新しいパラメーターを適用することで適用することを推奨します。注意: これらのパラメーターがあなたの特定のハードウェア構成に合っていることを確認してください：

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

`etc/security/limits.conf`を編集し、コンフィギュレーションファイルの一番下に以下の行を追加することで、開いているファイルディスクリプタの最大値（`ulimit`）をハードリミットの最大値である`5000000`よりも増やすことが推奨されます（変更を有効にするには、ログアウトして再度ログインします）：

```sh
*               soft    nofile          5000000
*               hard    nofile          5000000
```

#### 3. ノードの初期設定:

まだ実行していない場合は、アプリケーションを実行する専用のユーザを作成することをお勧めします。この場合、 `sudo adduser dagger` で `dagger` ユーザを作成し（任意のパスワードを作成）、 `sudo usermod -aG sudo dagger` で `dagger` ユーザを `sudo` ユーザグループに追加します。`sudo su - dagger` で `dagger` ユーザに切り替えます。残りのタスクはすべて `dagger` ユーザーとして実行します。

Wieldのバイナリを`dagger`ユーザディレクトリにダウンロード：

```sh
wget -O ~/wield https://shdw-drive.genesysgo.net/4xdLyZZJzL883AbiZvgyWKf2q55gcZiMgMkDNQMnyFJC/wield-latest
```

Wieldのバイナリを実行可能にする

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

{% hint style="danger" %}
**重要：シードフレーズとid.jsonキーペアファイルを自分のノード以外の安全な場所にバックアップする必要があります！ノードのパフォーマンスと報酬はこのIDに関連しています。ノードを再インストールする必要がある場合は、このバックアップから復元することで、ノードIDの継続性を維持することができます。キーペアを紛失した場合、復旧のお手伝いはできません。**

**ノードを再インストールまたは更新する必要がある場合は、新しいキーペアを作成せず、元の`id.json`ファイルを保持またはコピーしてください。**
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

`nano start_wield.sh`でWield起動スクリプトを作成し、以下の内容をファイルに貼り付けます。注意：これらのパラメータは16スレッドプロセッサーに基づいています。`--processor-threads`と`--global-threads`はマシンの総スレッド数と同じに設定し、`--comms-threads`は`2`に設定することを推奨します。異なるハードウェアでのパラメータの調整については、`wield --help` の出力を参照してください：

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

#### **4. ノードメンテナンス**


通常のネットワーク運用中にノードのメンテナンスを行う必要がある場合は、D.A.G.G.E.R.エポックを5回待ってからネットワークに再参加する必要があります。D.A.G.G.E.R.の進行状況はこちらで確認できます： [https://dagger-hammer.shdwdrive.com/explorer](https://dagger-hammer.shdwdrive.com/explorer)または公式公開の[ダッシュボード](https://dashboard.shdwdrive.com/)をご覧ください。

ノードを停止するには、`sudo systemctl stop wield`を使用します。この時点で、現在のバイナリをダウンロードするだけで `wield` を最新バージョンにアップグレードするなど、必要なメンテナンスを行うことができます：

```
wget -O ~/wield https://shdw-drive.genesysgo.net/4xdLyZZJzL883AbiZvgyWKf2q55gcZiMgMkDNQMnyFJC/wield-latest
```

厳密には必要ではありませんが、最新の `shdw-keygen` ユーティリティにもアップグレードすることをお勧めします。

```
wget -O ~/shdw-keygen https://shdw-drive.genesysgo.net/4xdLyZZJzL883AbiZvgyWKf2q55gcZiMgMkDNQMnyFJC/shdw-keygen-latest
```

`wield` のアップグレードが必要なだけであれば、5エポック後に `sudo systemctl start wield` でノードを再起動できます。

クラスタの完全な再起動が必要な場合は、適切な Discord チャンネルの指示に従うことをお勧めします。

#### shdwNodeをセットアップした後はどうすればいいですか？

さて、ノードを立ち上げて稼動させたら、[wallet verification](./#discord-verification)を確認し、[monitoring guide](monitoring-stack.md)でノードのパフォーマンスを表示・監視するための監視スタックの設定方法を確認してください。

\*免責事項：GenesysGoのD.A.G.G.E.R. Testnet Phase 1でWield Nodeを操作することにより、利用者は自発的かつ自己責任でこれを行うことを認めます。GenesysGoはTestnetソフトウェアを「現状のまま」無保証で提供し、お客様が被る可能性のある直接的、間接的、偶発的、結果的損害について一切の責任を負いません。お客様は、ご自身のシステムおよびデータのセキュリティに責任を負うものとします。GenesysGoは、お客様がTestnetに参加した結果生じたいかなる損失や損害についても責任を負いません。本ソフトウェアを使用することにより、お客様は、お客様のノード操作に関連するいかなるクレームや紛争からもGenesysGoを免責することに同意するものとします。本契約は、Wieldノードをダウンロードし、操作した時点で拘束力を持ちます。GenesysGoは予告なくTestnetを変更または中止することがあります。これにはTestnetフェーズ1のD.A.G.G.E.R. Wield Nodeを操作するためのハードウェア要件が含まれますが、これに限定されるものではありません。これらの条件に同意できない場合は、Testnetに参加しないでください。
