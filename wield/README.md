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

Wieldノードを運営することで、D.A.G.G.E.R. Hammerのインターフェイス[ここ](https://dagger-hammer.shdwdrive.com/)にあるD.A.G.G.E.R. フェーズ1テストネットにトラストレスに参加することができます。Wieldノードの役割とD.A.G.G.E.R.Hammerの目的については、ノード・オペレーターの皆様には私たちのブログ記事をご覧いただくことをお勧めします。

Wield Nodeのオペレーターは、何千ものライブ・ユーザー・テスト・トランザクションを処理し、D.A.G.G.E.R.内のすべてのモジュールのうち、消去コードに必要なものを信頼して実行し、Hammerテスト・インターフェースにアップロードされたファイルを保存します。

## 1. ノードの必要条件:
**具体的なハードウェア要件は未定です。オペレーティングシステムの要件はUbuntu 22.04 LTS kernel 5.15.0です。その他のLinux x86ディストリビューションは動作する可能性がありますが、現時点では**_**公式に**_**サポートされていません。**

## 2. オペレーティング・システムの構成:

`/etc/security/limits.conf`を編集し、コンフィギュレーションファイルの一番下に以下の行を追加することで、開いているファイルディスクリプタの最大値（`ulimit`）をハードリミットの最大値である`1048576`に設定することを推奨します（変更を有効にするには、一度ログアウトして再度ログインします）：

```
*               soft    nofile          1048576
*               hard    nofile          1048576
```

以下のカーネルチューニングパラメーターは、`/etc/sysctl.conf` を編集して以下の行をコンフィギュレーションファイルに追加し、`sudo sysctl -p` で新しいパラメーターを適用することで適用することを推奨します。注意: これらのパラメーターがあなたの特定のハードウェア構成に合っていることを確認してください：

```
# set default and maximum socket buffer sizes to 12MB
net.core.rmem_default=12582912
net.core.wmem_default=12582912
net.core.rmem_max=12582912
net.core.wmem_max=12582912

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
kernel.pid_max=65536

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

## 3. ノードの構成:

まだ実行していない場合は、アプリケーションを実行する専用のユーザを作成することをお勧めします。この場合、 `sudo adduser dagger` で `dagger` ユーザを作成し（任意のパスワードを作成）、 `sudo usermod -aG sudo dagger` で `dagger` ユーザを `sudo` ユーザグループに追加します。sudo su - dagger` で `dagger` ユーザに切り替えます。残りのタスクはすべて `dagger` ユーザーとして実行します。

Wieldのバイナリを`dagger`ユーザディレクトリにダウンロード：

```
wget -O wield https://shdw-drive.genesysgo.net/4xdLyZZJzL883AbiZvgyWKf2q55gcZiMgMkDNQMnyFJC/wield-0.1.0-x84_64-linux-gnu
```

Wieldのバイナリを実行可能にします:

```
sudo chmod +x wield
```

Shdw-Keygenユーティリティを`dagger`ユーザディレクトリにダウンロードします：

```
wget -O shdw-keygen https://shdw-drive.genesysgo.net/4xdLyZZJzL883AbiZvgyWKf2q55gcZiMgMkDNQMnyFJC/shdw-keygen-0.1.0-x84_64-linux-gnu
```

Shdw-Keygen ユーティリティを実行可能にします:

```
sudo chmod +x shdw-keygen
```

Shdw-Keygenユーティリティを使用して、新しい一意のキーペアIDを作成します：

```
./shdw-keygen new -o ~/id.json
```
`nano config.toml`で設定ファイルを作成し、以下の内容を貼り付けます:

```
trusted_node = "139.178.81.111:2030"
dagger = "JoinAndRetrieve"

[node_config]
socket = 2030
idle_timeout = 3600
keypair_file = "id.json"

[storage]
peers_db = "dbs/peers.db"
```

`nano start_wield.sh`でWield起動スクリプトを作成し、以下の内容をファイルに貼り付けます。注意：これらのパラメータは32c/64tプロセッサに基づいています。異なるハードウェアでのパラメータ調整については、`wield --help`の出力を参照してください：

```
#!/bin/bash
PATH=/home/dagger
exec wield \
--processor-threads 36 \
--global-threads 12 \
--comms-threads 8 \
--log-level info \
--history-db-path /mnt/dag/historydb \
--config-toml config.toml \
```

スクリプトを `sudo chmod +x start_wield.sh` で実行可能にします。

少なくとも 200GB の空き容量のあるディスクに `historydb` を格納する場所を作成します（ディスクの準備とマウントはこのドキュメントの範囲外です）。この場所は `start_wield.sh` 起動スクリプトの `--history-db-path` フラグで指定した場所と一致しなければいけません。この例では、予備ディスクを `/mnt/dag` にマウントし、そこに `historydb` ディレクトリを作成します：

```
sudo mkdir -p /mnt/dag/historydb
```

`historydb` ロケーションの所有者を `dagger` ユーザに変更します：

```
sudo chown -R dagger:dagger /mnt/dag/*
```

また、`wield`バイナリと同じ場所に`snapshots`ディレクトリが必要です。ここまでたどってきたのであれば、これはあなたのホームディレクトリであるはずです。以下のようにして作ることができます：

```
mkdir ~/snapshots
```

`sudo nano /etc/systemd/system/wield.service`で`wield`用のシステムサービスを作成し、以下の内容をファイルに貼り付けます：

```
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

```
sudo systemctl enable --now wield.service
```

`tail -f config.log`でログのtailingを行い、適切な動作を確認しましょう。


免責事項：GenesysGoのD.A.G.G.E.R. Testnet Phase 1でWield Nodeを操作することにより、利用者は自発的かつ自己責任でこれを行うことを認めます。GenesysGoは、いかなる保証もなく「現状のまま」テストネットソフトウェアを提供し、お客様が被る可能性のある直接的、間接的、偶発的、結果的損害について一切の責任を負いません。お客様は、ご自身のシステムおよびデータのセキュリティに責任を負うものとします。GenesysGoは、テストネットへの参加に起因するいかなる損失や損害に対しても責任を負いません。本ソフトウェアを使用することにより、お客様は、お客様のノード操作に関連するいかなるクレームや紛争からもGenesysGoを免責することに同意するものとします。本契約はWieldノードをダウンロードし、操作した時点で拘束力を持ちます。GenesysGoは予告なくテストネットを変更または中止することがあります。これらの条件に同意できない場合は、テストネットに参加しないでください。
