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

Wield Nodeを運用することで、D.A.G.G.E.R.のフェーズ1テストネットにトラストレスに参加することができます。このテストネットは、[こちら](https://dagger-hammer.shdwdrive.com/)にあるD.A.G.G.E.R. Hammerインターフェースを動かすために使用されます。Wield Nodeのオペレーターには、Wield Nodeの役割とD.A.G.G.E.R. Hammerの目的について完全に理解するために、私たちのブログ記事を確認することをお勧めします。

Wield Nodeのオペレーターは、数千ものライブユーザーテストトランザクションを扱い、D.A.G.G.E.R.内で要求されるすべてのモジュールをトラストレスに実行します。これには、Hammerテストインターフェースにアップロードされたファイルを消去コード化し、保存する作業が含まれます。

## 1. ノードの必要条件:
**動作環境はUbuntu 22.04 LTS kernel 5.15.0です。他のLinux x86ディストリビューションも動作する可能性がありますが、現時点ではサポートされていません。**

**Testnet Phase 1のD.A.G.E.R. Wield Nodeを動作させるための最小ハードウェア要件は以下のとおりです。**

* **16 CPUコア**
* **32 GB RAM**
* **データ・ストレージと高速I/O操作用の250GB SSD**
* **上下100mbpsのネットワーク接続が最低限必要です**

## 1.1. ガイド付きインストール ＋ スタートアップ

よりガイド付きの体験をお望みなら、特別なインストーラー・スクリプトを作成しました：

* システムのハードウェアをチェックし、指定された最低ハードウェアを満たしていることを確認する。
* tcp でのトラフィックフローを改善するためのカーネルチューンを適用する。
* `wield`と`shdw-keygen`のバイナリをダウンロードする。
* 鍵ペアファイルを生成する。
* マシンスペックに基づいた設定ファイルとスタートアップスクリプトを生成する。
* システムサービスの生成と設定
* wield サービスを起動する。

そのためには、ターミナルから以下のコマンドを実行してください：

```sh
wget -O wield-installer.sh https://shdw-drive.genesysgo.net/4xdLyZZJzL883AbiZvgyWKf2q55gcZiMgMkDNQMnyFJC/wield-installer.sh && chmod +x wield-installer.sh && ./wield-installer.sh
```

上記のスクリプトに問題がある場合は、以下の手動インストールを続けてください。

## 2. Operating system configuration:

`etc/security/limits.conf`を編集し、コンフィギュレーションファイルの一番下に以下の行を追加することで、開いているファイルディスクリプタの最大値（`ulimit`）をハードリミットの最大値である`1048576`に設定することを推奨する（変更を有効にするには、一度ログアウトして再度ログインする）：

```sh
*               soft    nofile          2097152
*               hard    nofile          2097152
```

以下のカーネルチューニングパラメーターは、`/etc/sysctl.conf` を編集して以下の行をコンフィギュレーションファイルに追加し、`sudo sysctl -p` で新しいパラメーターを適用することで適用することを推奨します。注意: これらのパラメーターがあなたの特定のハードウェア構成に合っていることを確認してください：

```sh
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

# open files limit
fs.nr_open = 2097152
```

## 3. ノードの構成:

まだ実行していない場合は、アプリケーションを実行する専用のユーザを作成することをお勧めします。この場合、 `sudo adduser dagger` で `dagger` ユーザを作成し（任意のパスワードを作成）、 `sudo usermod -aG sudo dagger` で `dagger` ユーザを `sudo` ユーザグループに追加します。sudo su - dagger` で `dagger` ユーザに切り替えます。残りのタスクはすべて `dagger` ユーザーとして実行します。

Wieldのバイナリを`dagger`ユーザディレクトリにダウンロード：

```sh
wget -O wield https://shdw-drive.genesysgo.net/4xdLyZZJzL883AbiZvgyWKf2q55gcZiMgMkDNQMnyFJC/wield-latest
```

Wieldのバイナリを実行可能にします:

```sh
sudo chmod +x wield
```

Shdw-Keygenユーティリティを`dagger`ユーザディレクトリにダウンロードします：

```sh
wget -O shdw-keygen https://shdw-drive.genesysgo.net/4xdLyZZJzL883AbiZvgyWKf2q55gcZiMgMkDNQMnyFJC/shdw-keygen-latest
```

Shdw-Keygen ユーティリティを実行可能にします:

```sh
sudo chmod +x shdw-keygen
```

Shdw-Keygenユーティリティを使用して、新しい一意のキーペアIDを作成します：

```sh
./shdw-keygen new -o ~/id.json
```
`nano config.toml`で設定ファイルを作成し、以下の内容を貼り付けます:

```toml
trusted_nodes = ["24.199.104.119:2030", "24.144.92.19:2030", "134.209.162.83:2030"]
dagger = "JoinAndRetrieve"

[node_config]
socket = 2030
keypair_file = "id.json"

[storage]
peers_db = "dbs/peers.db"
```

`nano start_wield.sh`でWieldスタートアップ・スクリプトを作成し、以下の内容をファイルに貼り付けます。注意：この起動スクリプトは16スレッドCPU用に最適化されています。異なるハードウェアでのパラメータ調整については、`wield --help`の出力を参照してください：

```bash
#!/bin/bash
PATH=/home/dagger
exec wield \
--processor-threads 8 \
--global-threads 8 \
--comms-threads 4 \
--log-level info \
--history-db-path /mnt/dag/historydb \
--config-toml config.toml \
```

スクリプトを `sudo chmod +x start_wield.sh` で実行可能にします。

少なくとも 200GB の空き容量のあるディスクに `historydb` を格納する場所を作成します（ディスクの準備とマウントはこのドキュメントの範囲外です）。この場所は `start_wield.sh` 起動スクリプトの `--history-db-path` フラグで指定した場所と一致しなければいけません。この例では、予備ディスクを `/mnt/dag` にマウントし、そこに `historydb` ディレクトリを作成します：

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

`tail -f config.log`でログのtailingを行い、適切な動作を確認しましょう。

\*免責事項：GenesysGoのD.A.G.G.E.R. Testnet Phase 1でWield Nodeを操作することにより、利用者は自発的かつ自己責任でこれを行うことを認めます。GenesysGoはTestnetソフトウェアを「現状のまま」無保証で提供し、お客様が被る可能性のある直接的、間接的、偶発的、結果的損害について一切の責任を負いません。お客様は、ご自身のシステムおよびデータのセキュリティに責任を負うものとします。GenesysGoは、お客様がTestnetに参加した結果生じたいかなる損失や損害についても責任を負いません。本ソフトウェアを使用することにより、お客様は、お客様のノード操作に関連するいかなるクレームや紛争からもGenesysGoを免責することに同意するものとします。本契約は、Wieldノードをダウンロードし、操作した時点で拘束力を持ちます。GenesysGoは予告なくTestnetを変更または中止することがあります。これにはTestnetフェーズ1のD.A.G.G.E.R. Wield Nodeを操作するためのハードウェア要件が含まれますが、これに限定されるものではありません。これらの条件に同意できない場合は、Testnetに参加しないでください。

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

A: 推奨されるシステム要件は、AWS EC2 t2.2xlargeインスタンスまたは同等のもので、8 vCPU、32GB RAM、Ubuntu 22.04 LTS、カーネルバージョン5.15.0以降です。あるいは、古いゲームマシンのパーツを使用することもできますし、これらの最低要件を満たすセットアップを行うこともできます。

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

## **Q: ノードのバージョンを確認し、必要に応じてアップグレードするにはどうすればよいですか?

A: ノードのバージョンは `./wield --version` で確認できます。アップグレードするには、対話型インストーラースクリプトを使います。

## **Q: どうすれば最新版にアップデートできますか？

A: ノードのサービスを停止し、最新のバイナリをダウンロードした後、 サービスを再起動することでアップデートできます。詳しいコマンドはユーザがチャットで教えてくれます。

## **Q: Wieldノードのエラーや問題をどのようにトラブルシューティングできますか？**

A: トラブルシューティングについては、システムログをチェックし、ノードの設定ファイルにエラーがないか確認し、すべての依存関係が正しくインストールされていることを確認してください。また、特定のエラーメッセージやログを共有することで、Discord コミュニティのサポートチャンネルに助けを求めることもできます。

## **Q:サーバーのシステム使用量を確認するにはどうすればよいですか?**

A: Ubuntuサーバーでは、Ubuntuシステムモニターを開くか、`htop`、`top`、`vmstat`などのコマンドラインツールを使ってシステムの使用状況を確認できます。これらのツールは、CPU、メモリ、ディスクの使用状況に関する情報を提供します。

## **Q: Wieldノードのログファイルはどのように確認できますか？**

A: ログファイルは `cat`、`less`、`tail`、`grep` などのコマンドラインツールで確認できます。具体的なログファイルの場所はノードの設定によって異なります。systemd のサービスログをチェックするには `journalctl` を使ってください、例えば `sudo journalctl -u wield.service` とします。

## **Q: 1エポックの長さはどれくらいですか？**

A: エポックは現在200バンドルです。

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

## Q: サーバーのログファイルを共有するにはどうすればいいですか？

A: `scp` コマンドなどを使ってサーバーからログファイルをダウンロードし、共有サービスにアップロードすることができます。
