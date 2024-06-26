# ウォレット

{% hint style="warning" %}
**重要 - あなたのノードが報酬の対象となる前に、あなたのDiscordユーザーにノードIDを追加して身元を確認する必要があります。それが完了すると、報酬があなたのノードに蓄積され始めます。**

**検証されたステータスを維持し、ネットワーク内での地位を維持するためには、Discordの検証システムで検証されたすべてのウォレットで最低でも100 SHDWを保有する必要があります。**
{% endhint %}

shdwDriveのIDセキュリティは、台帳に透過的に発行されるCLIベースのローカルshdwWalletによって信頼できる形で処理されます。shdwDrive v2の開発が進むにつれ、重要なマイルストーンは、ユーザーのネットワーク体験を大幅に向上させるフル機能のshdwWalletのリリースとなります。

この開発段階では、testnetsのすべての経済機能は、高速で安価なトランザクション実行のためにSolanaブロックチェーンを利用しています。このSolanaの統合により、私たちはSolanaにネイティブなサードパーティのウォレットを利用することができ、Solanaネイティブトークン$SHDWとして、Solanaの構築者とユーザーの両方とshdwDrive v2をシームレスに統合するのに役立ちます。

各shdwNodeは、公開鍵と秘密鍵の2つからなるアカウントを使用します。2つの鍵を持つこのアプローチは公開鍵暗号として知られており、広く使われている暗号システムです。これらはweb2の世界におけるユーザー名とパスワードのようなものです。shdwNodes上で動作するD.A.G.G.E.R.プロトコルは、ユーザー名とパスワードをこれらの公開鍵と秘密鍵に置き換えます。

## サードパーティウォレット

D.A.G.G.E.R.はSolanaブロックチェーンと並行して運営されており、Solanaと互換性のあるウォレットであればアカウントで使用することができます。

### Solana エコシステムウォレット

いくつかのブラウザとモバイルアプリベースのウォレットがSolanaをサポートしています。[Solana Ecosystem](https://solana.com/ecosystem/explore?categories=wallet)のページで、自分に合ったものを見つけることができます。

上級ユーザーや開発者には、コマンドライン（CLI）ウォレットがより適切かもしれません:

* [Solana CLI](https://docs.solana.com/cli)

### Solanaウォレットへの$SHDWアカウントのインポート

shdwWallet から別の Solana ウォレットに shdwAccount をインポートするには、`private Key`（秘密鍵）メソッドを使います。

秘密鍵メソッドは、サードパーティのウォレットにインポートするために必要な鍵を抽出するために推奨される方法です。技術的には、秘密鍵は shdwNode ウォレットで使用される特定の派生パスを埋め込みます。

### 秘密鍵を使う方法

#### アカウントの秘密鍵をshdwNodeから取得しましょう

1. [インストールガイド](install.md#id-3.-initial-node-configuration)で指定されているように、`shdw-keygen`バイナリをダウンロードしたことを確認してください。
2. `shdw-keygen` バイナリと同じディレクトリで、以下のコマンドを実行します。

```
/shdw-keygen privkey <path-to-keypair.json>
```

ここで、`<path-to-keypair.json>` はノードのIDキーペア・ファイルへのパスです。例えば、デフォルトでは `~/id.json` にあります。

3. ランダムな文字列ができるはずです。これがあなたのノードIDの秘密鍵です。_<mark style="color:red;">**誰にも教えないでください！**</mark>_ この出力をコピーしてください。次のステップで必要になります。

#### 秘密鍵をSolanaウォレットにインポートしましょう

そのために、[Phantom](https://phantom.app/)　walletでデモンストレーションを行います。

1. ファントムウォレットを開きます。
2. 以下のようなハンバーガーメニューアイコンをクリックします。
   ![](<../.gitbook/assets/image (2).png>)
3. サイドメニューが開いたら、「Add / Connect Wallet」の「+」アイコンをクリックします。
   ![](<../.gitbook/assets/image (3).png>)
4. 以下のような画面が表示されます。
   ![](<../.gitbook/assets/image (4).png>)
5. 秘密鍵のインポート をクリックします。
6. ネットワークとしてSolanaが選択されていることを確認し、好きな名前を付けて、[#アカウントの秘密鍵をshdwNodeから取得しましょう](wallet.md#get-your-accounts-private-key-from-your-shdwnode "mention")の出力から秘密鍵を貼り付けます。
7. あなたのshdwNodeのIDがSolana Walletにあるはずです！[https://holders.genesysgo.com/](https://holders.genesysgo.com/) にアクセスし、Discordアカウントでサインインしてください。あなたのノードがネットワークで識別されると、Discordで自動的に`shdwOperator`ロールを受け取ることができます。

