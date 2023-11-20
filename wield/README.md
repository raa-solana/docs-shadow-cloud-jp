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
**動作環境はUbuntu 22.04 LTS kernel 5.15.0です。その他の Linux x86 ディストリビューションも動作する可能性がありますが、現時点では **_**公式には**_** サポートされていません。**

**D.A.G.G.E.R. Wield Node for Testnet Phase 1 を動作させるために最低限必要なハードウェアは以下の通りです。**

* 8 CPU コア
* 32 GB RAM
* データストレージおよび高速I/O操作用250GB SSD***
* ネットワーク接続は最低限必要です。

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


\*免責事項： GenesysGoのD.A.G.G.E.R. Testnet Phase 1上でWield Nodeを操作することで、利用者は自発的に、かつ自己の責任においてこれを行うことを認めます。GenesysGoはTestnetソフトウェアを「現状のまま」無保証で提供し、あなたが被る可能性のある直接的、間接的、偶発的、結果的損害について一切の責任を負いません。あなたは、ご自身のシステムおよびデータのセキュリティに責任を負うものとします。GenesysGoは、あなたがTestnetに参加した結果生じたいかなる損失や損害についても責任を負いません。本ソフトウェアを使用することにより、あなたは、あなたのノード操作に関連するいかなるクレームや紛争からもGenesysGoを免責することに同意するものとします。本契約は、Wieldノードをダウンロードし、操作した時点で拘束力を持ちます。GenesysGoは予告なくTestnetを変更または中止することがあります。これにはTestnet Phase 1でD.A.G.G.E.R.Wieldノードを操作するためのハードウェア要件が含まれますが、これに限定されるものではありません。これらの条件に同意できない場合は、Testnetに参加しないでください。


### D.A.G.G.E.R.テストネットとWieldノードに関するよくある質問(FAQ)

## **Q: Linuxを使ったことがなくてもWield Nodeを設定できますか？**
A: 学ぶ意欲があれば、YouTubeのチュートリアルを参考にしたり、ChatGPTのようなLinuxガイダンスを利用したりすることで、ノードの設定が可能です。コミュニティやエンジニアリングチームも質問に答えるために利用できます。

## **Q: Wield Nodeを設定するために必要なハードウェアは何ですか？**
A: 推奨されるシステム要件は、AWS EC2 t2.2xlargeインスタンスまたは同等のもので、8 vCPU、32GB RAMを備え、Ubuntu 22.04 LTSを実行し、カーネルバージョンは5.15.0以上が必要です。あるいは、古いゲーム機の部品やこれらの最低要件を満たす任意の設定を使用することもできます。

## **Q: Wield Nodeを実行するために仮想マシンを使用できますか？**
A: はい、Wield NodeはVirtualBoxやVMwareのような仮想マシンや、さまざまなクラウドプラットフォームで成功裏にテストされています。VMが最適なパフォーマンスを発揮するための推奨システム要件を満たしていることを確認してください。

## **Q: ノードの設定中やキーペア生成時にGLIBCが見つからないというエラーに遭遇した場合、どうすればよいですか？**
A: このエラーは通常、UbuntuのバージョンまたはLinuxカーネルが正しくないことを示しています。Ubuntu 22.04を使用し、カーネルが5.15.0以上であることを確認してください。システムの更新（`sudo apt update` および `sudo apt upgrade`）を行い、必要に応じてマシンを再起動する必要があるかもしれません。

## **Q: Ubuntu 20.04から22.04へのアップグレードは安全ですか？**
A: はい、多くのユーザーがUbuntu 20.04から22.04へのインプレースアップグレードを問題なく行っています。

## **Q: Wieldサービスや設定プロセス中に問題に遭遇した場合、どうすればよいですか？**
A: まず、`config.toml`が正しく設定されていること、そして`start_wield.sh`に実行権限が与えられていること（`chmod +x`）を確認してください。さらに詳細を知るために`sudo journalctl -u wield`でサービスの状態をチェックしてください。`wield.service`が失敗と表示された場合は、`start_wield.sh`の設定を調整するか、`--help`コマンドを使用して指導を受けてください。サポートチャンネルでエラーや問題を共有してさらなる支援を求めてください。

## **Q: すでに別のノードをホスティングしているサーバー上でWield Nodeをデプロイできますか？**
A: 可能ですが、サーバーに両方のノードをサポートするのに十分なリソースがあることを確認し、パフォーマンスに影響を与えないようにリソース使用状況を密接に監視してください。

## **Q: テストネットの段階は何回あり、メインネットの開始予定はいつですか？**
A: テストネットの段階の数とメインネットの開始日はまだ決まっていません。チームは安定性とパフォーマンスを確保することに焦点を当てており、いかなる期限も設定する前にこれらを確実にします。

## **Q: D.A.G.G.E.R.のロードマップはどこで見ることができますか？**
A: ロードマップは公式ブログで見ることができます。これには、今後のテストネットフェーズの計画が概説されています。

## **Q: 推奨される仕様よりも低いスペックのシステムでWield Nodeを実行できますか？テスト目的で。**
A: 低スペックのシステムでノードを実行してみることは可能ですが、最適に機能しなかったり、特にRAMとCPUのパワーが最小要件を満たしていない場合はクラッシュする可能性があります。チームはあなたの経験から学ぶかもしれませんが、スムーズに動作することは保証されません。

## **Q: D.A.G.G.E.R.ネットワーク上のエポックとは何ですか？**
A: D.A.G.G.E.R.ネットワーク上のエポックは、ネットワークの活動に応じて、現在は5-10分程度の短い時間期間です。

## **Q: Wield Nodeを実行するためのサーバーをレンタルするにはどこがよいですか？**
A: ほとんどのクラウドプロバイダーは、Wield Nodeを実行するのに適したインスタンスを提供しています。Digital Oceanは比較のための堅実な基準ですが、プロバイダーがノードを実行するためのシステム要件を満たしていることを常に確認してください。

## **Q: 詳しく知らない用語に遭遇したり、さらなる支援が必要な場合はどうすればよいですか？**
A: サポートチャンネルで説明や支援を求めることをためらわないでください。コミュニティやコアエンジニアリングチームは、プロセスを通じてあなたを支援するために存在します。
