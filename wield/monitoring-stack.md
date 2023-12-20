---
description: >-
  以下の説明は、Docker環境で実行するためのものです。
  付属のdocker-compose.yamlファイルには、ノードを監視するための監視スタックが含まれています。
---

# Monitoring Stack

### 公式ダガーモニターコードと手順は[https://github.com/GenesysGo/dagger-monitoring](https://github.com/GenesysGo/dagger-monitoring)へ。

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

**DigitalOceanから入手した22.04用のDockerインストール -** [**https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04**](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04)

**以下の説明は22.04用です。他のシステムにインストールする場合は、特定のOSを確認してください。**

### Step 1: Dockerのインストール

Ubuntuの公式リポジトリで入手可能なDockerインストールパッケージは、最新バージョンではない可能性があります。最新版を確実に入手するために、公式DockerリポジトリからDockerをインストールしましょう。そのためには、新しいパッケージソースを追加し、DockerからのGPGキーを追加してダウンロードが有効であることを確認し、パッケージをインストールします。

まず、既存のパッケージのリストを更新します:

```sh
sudo apt update
```

次に、aptがHTTPS経由でパッケージを使えるようにするための前提パッケージをいくつかインストールします:

```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```

次に、公式DockerリポジトリのGPGキーをシステムに追加します:

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

APTソースにDockerリポジトリを追加します:

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null"
```

これでパッケージ・データベースも、新しく追加されたリポジトリのDockerパッケージで更新されます。

デフォルトのUbuntu repoではなく、Docker repoからインストールしようとしていることを確認してください:

```basic
apt-cache policy docker-ce
```

最後にDockerをインストールします:

```bash
sudo apt install docker-ce
```

これでDockerがインストールされ、デーモンが起動し、起動時にプロセスが開始できるようになったはずです。起動していることを確認します:

```bash
sudo systemctl status docker
```

出力は以下のようになり、サービスがアクティブに動作していることを示しています:

```
Output
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2020-05-19 17:00:41 UTC; 17s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 24321 (dockerd)
      Tasks: 8
     Memory: 46.4M
     CGroup: /system.slice/docker.service
             └─24321 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
Installing Docker now gives you not just the Docker service (daemon) but also the docker command line utility, or the Docker client. We’ll explore how to use the docker command later in this tutorial.
```

### Step 2: Sudoを使わずにDockerコマンドを実行する（オプションですが推奨です）

デフォルトでは、dockerコマンドはrootユーザーか、Dockerのインストール時に自動的に作成されるdockerグループ内のユーザーのみが実行できます。sudoのプレフィックスを付けずに、あるいはdockerグループに属さずにdockerコマンドを実行しようとすると、次のように出力されます：

Output docker: Cannot connect to the Docker daemon. Is the docker daemon running on this host?. See 'docker run --help'. If you want to avoid typing sudo whenever you run the docker command, add your username to the docker group:

```bash
sudo usermod -aG docker ${USER}
```

新しいグループ・メンバーシップを適用するには、サーバーからログアウトして再度ログインするか、次のように入力します。

```bash
su - ${USER}
```

続行するには、ユーザーのパスワードを入力するよう求められます。

以下を入力して、あなたのユーザが docker グループに追加されたことを確認してください：

```bash
groups
```

出力

```
dagger sudo docker
```

ログインしていないユーザーをdockerグループに追加する必要がある場合は、以下のようにユーザー名を明示的に宣言してください。

```bash
sudo usermod -aG docker <username>
```

**注意：この記事の残りの部分では、dockerコマンドをdockerグループのユーザーとして実行していることを前提としています。そうしない場合は、コマンドの先頭にsudoを付けてください。**

**
**Docker Composeのインストール - DigitalOceanから引用。-** [**https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-22-04**](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-22-04)

### Step 3: Docker Composeのインストールと起動

Docker Composeの最新安定版を入手するために、公式GitHubリポジトリからダウンロードします。

まず、リリースページで最新バージョンを確認する。この記事を書いている時点では、最新の安定バージョンは2.23.3です。

以下のコマンドで2.23.3のリリースをダウンロードし、実行ファイルを保存することで、このソフトウェアがdocker-composeとしてグローバルにアクセスできるようになります：

```bash
mkdir -p ~/.docker/cli-plugins/
```

```sh
curl -SL https://github.com/docker/compose/releases/download/v2.3.3/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose
```

次に、docker-composeコマンドが実行可能になるように、正しい権限を設定します：

```sh
chmod +x ~/.docker/cli-plugins/docker-compose// Some codechmod +x ~/.docker/cli-plugins/docker-compose
```

インストールが成功したことを確認するには、次のコマンドを実行します:

```sh
docker compose version
```

次のように出力されます:

```
Output
docker-compose version v2.23.3
```

Grafana用のdockerストレージボリュームを作成し、リブート中も持続するようにします:

```sh
docker volume create grafana-storage
```

_**Githubからリポジトリをクローン**_

以下を実行します:

```sh
git clone https://github.com/GenesysGo/dagger-monitoring && cd dagger-monitoring
```

_**Docker Composeを実行し、モニタリングを有効にします**_

適切なディレクトリにいることを確認します

```
/home/dagger/dagger-monitoring
```

docker-compose.yamlファイルが表示されます。

```sh
docker compose up -d
```

ブラウザでサーバーのIPにポート3000でアクセスし、Grafanaに接続します:

例:

```
http://1.2.3.4:3000
```

デフォルトの認証情報でログイン

```
admin/admin
```

**PrometheusをデータセットとしてGrafanaにインポートします**

カラムの一番下にある左パネルの歯車ボタンにカーソルを合わせ、**データソース**をクリックします。右上の**データソースの追加**をクリックします。リストから**Prometheus**を選択します。[http://prometheus:9090](http://prometheus:9090/) (shadow docker-compose built docker networkのため、prometheusというホスト名で動作します)。**ENSURE PORT IS 9090, NOT 9000** 一番下に追加して保存します。

左側の「ダッシュボード」ボタンに移動します。これは4つの正方形を積み重ねたように見えます。この画面の右上にあるインポートをクリックします。import via grafana.comの下に、次のように入力します： `1860`

これが読み込まれたら、一番下のimportをクリックします。

これでwieldノードのダッシュボードができ、メトリクスがダッシュボードに表示されるようになります。

**注：Grafanaの最新バージョンには不具合があります。チャート/ゲージのどれかが切り取られているように見える場合、クリック+ドラッグして再び展開する必要があります。これはGrafanaの視覚的なバグです。**
