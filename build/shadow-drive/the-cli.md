# CLI

## **Contents**

* [**Shadow Drive CLIのインストール**](the-cli.md#install-the-shadow-drive-cli)
  * [**ビデオガイドとチュートリアル**](the-cli.md#video-guide-and-walkthrough)
  * [**Solana CLIのインストール**](the-cli.md#install-the-solana-cli)
  * [**ストレージアカウントの作成**](the-cli.md#create-a-storage-account)
  * [**ファイルのアップロード**](the-cli.md#upload-file-to-shadow-drive)
  * [**複数のファイルをアップロードする**](the-cli.md#upload-multiple-files-to-shadow-drive)
  * [**ファイルの編集**](the-cli.md#edit-a-file-aka-overwrite-a-file-aka-replace-a-file)
  * [**ファイルの削除**](the-cli.md#delete-a-file)
  * [**ストレージの追加**](the-cli.md#add-storage-to-storage-account)
  * [**ストレージアカウントのサイズを縮小する**](the-cli.md#reduce-storage-account-size)
  * [**ファイルを不変にする**](the-cli.md#make-storage-account-immutable)
* [**Rust CLIのインストール - 実験的!**](the-cli.md#the-rust-cli)

## **Introduction**

CLIは、Shadow Driveと対話する最も簡単な方法です。好きなシェルスクリプト言語を使用したり、単一のコマンドをタイプしたりすることができます。Shadow Driveをテストドライブするには、これが最適な方法です。

## **Install the Shadow Drive CLI**

前提条件：任意のOSに[NodeJS LTS 16.17.1](https://nodejs.org/en/download/)をインストールしてください。

次に、以下のコマンドを実行します。

```bash
npm install -g @shadow-drive/cli
```

#### [**Video Guide and Walkthrough**](https://www.youtube.com/watch?v=MfSuzFDDQ30)

### **Install the Solana CLI**

Shadow Driveと対話するためには、SolanaウォレットとSolanaブロックチェーンと対話するためのCLIが必要です。

_注：Shadow Drive CLI は、独自の RPC 設定を使用します。Solana 環境の構成は使用しません。_

[最新バージョンを確認してください](https://docs.solana.com/cli/install-solana-cli-tools).

```bash
sh -c "$(curl -sSfL https://release.solana.com/v1.14.3/install)"
```

インストールしたら、すぐに次のようにします:

```bash
export PATH="/home/sol/.local/share/solana/install/active_release/bin:$PATH"
```

### **Create a Keypair file**

Shadow Drive CLIを使用するには、.json形式のキーペアを用意する必要があります。これは、ストレージアカウントを所有するウォレットになる予定です。必要であれば、秘密鍵をエクスポートすることで、ブラウザのウォレットを .json ファイルに変換できます。Solflareはデフォルトで.json形式でエクスポートします（標準的な整数の配列のように見えます[1,2,3,4...]）。しかし、Phantomは、いくつかの助けを必要とし、[私たちは、ちょうどツールを持っています](https://gist.github.com/tracy-codes/f17e7ed8acfdd1be442f632f5b80763c)。

新しいウォレットを作成する場合は、そのまま

```
solana-keygen new -o ~/shdw-keypair.json
```

新しいキーペアファイルが作成され、あなたのウォレットアドレスである `pubkey` が表示されます。

**該当ウォレットアドレスに少量のSOLとSHDWを送信して手続きをする必要があります！SOLは取引手数料の支払いに、SHDWはストレージアカウントの作成（および拡張）に使用されます！**

#### **Context-Sensitive Help**

Shadow Drive CLI には、統合されたヘルプが付属しています。すべてのシャドウドライブコマンドは `shdw-drive` で始まります。

```
shdw-drive help
```

上記のコマンドを実行すると、次のように出力されます。

<figure><img src="../../.gitbook/assets/2022-10-11_13-17-13.png" alt=""><figcaption></figcaption></figure>

これらのコマンドの詳細なヘルプは、コマンドの後に `--help` オプションを入力することで出力されます。

```
shdw-drive create-storage-account --help
```

### **Create a Storage Account**

これは[SHDW](https://docs.shadow.cloud/reference/shdw-token)が必要になる数少ないコマンドの一つです。コマンドを実行する前に、ストレージアカウントを予約するのに必要な[SHDW](https://docs.shadow.cloud/reference/shdw-token)の量を尋ねるプロンプトが表示されます。必要なオプションは3つあります：

`-kp, --keypair`

* ストレージアカウントを作成するウォレットへのパス

`-n, --name`

* ストレージアカウントに付けたい名前。(ユニークである必要はありません)

`-s, --size`

* 作成を依頼するストレージの量。これは「1KB」「1MB」「1GB」のような文字列であるべきです。ストレージの区切りは、KB、MB、GB のみサポートされます。

**例:**

```
shdw-drive create-storage-account -kp ~/shdw-keypair.json -n "pony storage drive" -s 1GB
```

### **Upload File to Shadow Drive**

このコマンドに必要なオプションは2つだけです:

`-kp, --keypair`

* ファイルをアップロードするウォレットへのパス。

`-f, --file`

* ファイルパスです。現在のファイルサイズの上限はCLIで1GBです。

複数のストレージアカウントをお持ちの場合は、所有するストレージアカウントのリストが表示され、そこから選択することができます。オプションで、ストレージアカウントのアドレスを指定することができます:

`-s, --storage-account`

* ファイルをアップロードするストレージアカウント。

**例:**

```
shdw-drive upload-file -kp ~/shdw-keypair.json -f ~/AccountHolders.csv
```

### **Upload Multiple Files to Shadow Drive**

より現実的なユースケースは、例えばNFTの画像とメタデータのディレクトリ全体をアップロードすることです。コマンドにディレクトリを指定する以外は、基本的に同じことです。

オプション:

`-kp, --keypair`

* ファイルをアップロードするウォレットへのパス。

`-d, --directory`

* アップロードしたいファイルのあるフォルダへのパス。

`-s, --storage-account`

* ファイルをアップロードするストレージアカウント。

`-c, --concurrent`

* バッチアップロードの同時実行数。(デフォルト:"3")

**例:**

```
shdw-drive upload-multiple-files -kp ~/shdw-keypair.json -d ~/ponyNFT/assets/
```

### **Edit a File (aka Overwrite a File aka Replace a File)**

このコマンドは、_全く同じ名前を持つ_ 既存のファイルを置き換えるために使用されます。 `edit-file`を使ってこのファイルをアップロードしようとしたときに、同じ名前を持つ既存のファイルがまだ存在しない場合、リクエストは失敗します。

このコマンドには3つの要件があります:

`-kp, --keypair`

* ファイルをアップロードするウォレットへのパス。

`-f, --file`

* ファイルパスです。現在のファイルサイズ制限は、CLIを通じて1GBです。ファイル名は、最初にアップロードしたものと同じでなければなりません。

`-u, --url`

* 削除を依頼するファイルのシャドウドライブURL

**例:**

```
shdw-drive edit-file --keypair ~/shdw-keypair.json --file ~/ponyNFT/01.json --url https://shdw-drive.genesysgo.net/abc123def456ghi789/0.json
```

### **Delete a File**

これは簡単なことで、一度削除されると永久に消えてしまうことに注意する必要があります。

条件は2つで、標準的なもの以外の選択肢はありません:

`-kp, --keypair`

* ストレージアカウントとファイルを所有するウォレットのキーペアファイルへのパス。

`-u, --url`\
削除を依頼するファイルのシャドウドライブURL

**例:**

```
shdw-drive delete-file --keypair ~/shdw-keypair.json --url https://shdw-drive.genesysgo.net/abc123def456ghi789/0.json
```

### **Add Storage to Storage Account**

ストレージアカウントのストレージサイズを拡張することができます。このコマンドは、SHDW トークンを消費します。

この呼び出しには、次の2つの要件があるだけです。

`-kp, --keypair`

* ファイルをアップロードするウォレットへのパス。

`-s, --size`

* ストレージアカウントへの追加を要求するストレージの量。1KB'、'1MB'、'1GB'のような文字列であるべきです。現在、KB、MB、GBのストレージの区切りのみがサポートされています。

複数のアカウントをお持ちの場合は、どのストレージアカウントにストレージを追加するかを選択することになります。

**例:**

```
shdw-drive add-storage -kp ~/shdw-keypair.json -s 100MB
```

### **Reduce Storage Account Size**

ストレージアカウントを減らし、未使用のSHDWトークンを再利用することができます。_**これは、まずアカウントサイズを減らし、次にSHDWトークンを要求するという2つの操作**_ です。まず、ストレージアカウントサイズを縮小しましょう。

要件は次の2つです。

`-kp, --keypair`

* ファイルをアップロードするウォレットへのパス。

`-s, --size`

* ストレージアカウントへの追加を要求するストレージの量。1KB'、'1MB'、'1GB'のような文字列であるべきです。現在、KB、MB、GBのストレージの区切りのみがサポートされています。

**例:**

```
shdw-drive reduce-storage -kp ~/shdw-keypair.json -s 500MB
```

### **Claim Stake (aka Claim Unused** [**SHDW**](https://docs.shadow.cloud/reference/shdw-token) **Tokens after Reduction)**

前のステップでストレージの使用量を減らしたので、未使用の[SHDW](https://docs.shadow.cloud/reference/shdw-token)トークンを自由に請求できます。ここで必要なのはキーペアだけです。

**例:**

```
shdw-drive claim-stake -kp ~/shdw-keypair.json
```

### **Delete a Storage Account**

シャドードライブからストレージアカウントを完全に削除することができます。完了すると、[SHDW](https://docs.shadow.cloud/reference/shdw-token) トークンがウォレットに戻されます。

_注：削除の際には、現在のソラナエポックの終わりまで続く猶予期間があります。_ [_こちらをご覧ください_](https://explorer.solana.com/) _現在のソラナエポックの残り時間を知るには、どれくらいの猶予期間があるのか知ることができます。_

ここで必要なのはキーペアだけで、削除する特定のストレージアカウントの入力を促されます。

**例:**

```
shdw-drive delete-storage-account ~/shdw-keypair.json
```

### **Undelete a Storage Account**

エポックがまだ有効であると仮定すると、ストレージアカウントの削除を解除することができます。必要なのはキーペアだけです。コマンドを実行する際に、ストレージアカウントを選択するよう促されます。これにより、削除要求が削除されます。

```
shdw-drive undelete-storage-account -kp ~/shdw-keypair.json
```

### **Make Storage Account Immutable**

Shadow Driveの最もユニークで便利な機能の1つは、ストレージを真に永久的なものにすることができることです。不変のストレージでは、アカウントにアップロードされたファイルが削除されたり編集されたりすることは決してありません。それらは、ストレージアカウント自体もそうであるように、強固で永久的なものです。不変アカウントにファイルをアップロードし続けることも、不変アカウントにストレージを追加することも可能です。

必要なのはキーペアだけです。コマンドを実行する際にストレージアカウントを選択するよう促されます。

**例:**

```
shdw-drive make-storage-account-immutable -kp ~/shdw-keypair.json
```

## **The Rust CLI**

### **(This section is under development)**

### **CreateStorageAccount**

データを保存するためのアカウントを作成します。ストレージアカウントは、1回限りの料金で、グローバルに、不可逆的な不変マークを付けることができます。それ以外の場合は、ファイルの追加や削除が可能で、容量は無期限でレンタルできます。

**Parameters:**

`--name`

* String

`--size`

* Byte

**例:**

```
shadow-drive-cli create-storage-account --name example_account --size 10MB
```

### **DeleteStorageAccount**

削除のためにストレージアカウントをキューに入れます。要求がまだキューに入っていて実行されていない間は、キャンセルすることができます（cancel-delete-storage-accountサブコマンドを参照）。

**Parameters:**

`--storage-account`

* Pubkey

**例:**

```
shadow-drive-cli delete-storage-account --storage-account FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB
```

**例:**

```
shadow-drive-cli delete-storage-account FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB
```

### **CancelDeleteStorageAccount**

削除のためにエンキューされたストレージアカウントの削除をキャンセルします。

**Parameters:**

`--storage-account`

* Pubkey

**例:**

```
shadow-drive-cli cancel-delete-storage-account --storage-account FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB
```

**例:**

```
shadow-drive-cli cancel-delete-storage-account FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB
```

### **ClaimStake**

ストレージの容量を削減した後、ストレージアカウントに付与されたトークンを戻します。

**Parameters:**

`--storage-account`

* Pubkey

**例:**

```
shadow-drive-cli claim-stake --storage-account FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB
```

**例:**

```
shadow-drive-cli claim-stake FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB
```

### **AddStorage**

ストレージアカウントの容量を増やします。

**Parameters:**

`--storage-account`

* Pubkey

`--size`

* Byte (accepts KB, MB, GB)

**例:**

```
shadow-drive-cli add-storage --storage-account FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB --size 10MB
```

**例:**

```
shadow-drive-cli add-storage FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB 10MB
```

### **AddImmutableStorage**

ストレージアカウントの不変ストレージ容量を増やします。

**Parameters:**

`--storage-account`

* Pubkey

`--size`

* Byte (accepts KB, MB, GB)

**例:**

```
shadow-drive-cli add-immutable-storage --storage-account FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB --size 10MB
```

**例:**

```
shadow-drive-cli add-immutable-storage FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB 10MB
```

### **ReduceStorage**

ストレージアカウントの容量を削減します。

**Parameters:**

`--storage-account`

* Pubkey

`--size`

* Byte (accepts KB, MB, GB)

**例:**

```
shadow-drive-cli reduce-storage --storage-account FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB --size 10MB
```

**例:**

```
shadow-drive-cli reduce-storage FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB 10MB
```

### **MakeStorageImmutable**

ストレージアカウントを不変にします。これは不可逆です。

**Parameters:**

`--storage-account`

* Pubkey

**例:**

```
shadow-drive-cli make-storage-immutable --storage-account FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB
```

**例:**

```
shadow-drive-cli make-storage-immutable FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB
```

### **GetStorageAccount**

ストレージアカウントに関連するメタデータを取得します。

**Parameters:**

`--storage-account`

* Pubkey

**例:**

```
shadow-drive-cli get-storage-account --storage-account FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB
```

**例:**

```
shadow-drive-cli get-storage-account FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB
```

### **GetStorageAccounts**

特定のpubkeyが所有するストレージアカウントのリストを取得します。所有者が指定されていない場合は、設定されている署名者が使用されます。

**Parameters:**

`--owner`

* Option\<Pubkey>

**例:**

```
shadow-drive-cli get-storage-accounts --owner FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB
```

**例:**

```
shadow-drive-cli get-storage-accounts
```

### **ListFiles**

List all the files in a storage account.

**Parameters:**

`--storage-account`

* Pubkey

**例:**

```
shadow-drive-cli list-files --storage-account FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB
```

**例:**

```
shadow-drive-cli list-files FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB
```

### **GetText**

ファイルを取得し、それをテキストと仮定して出力します。

**Parameters:**

`--storage-account`

* Pubkey

`--filename`

**例:**

```
shadow-drive-cli get-text --storage-account FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB --filename example.txt
```

**例:**

```
shadow-drive-cli get-text FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB example.txt
```

### **GetObjectData**

ストレージアカウントファイルから基本ファイルオブジェクトデータを取得します。

**Parameters:**

`--storage-account`

* Pubkey

`--file`

* String

**例:**

```
shadow-drive-cli get-object-data --storage-account FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB --file example.txt
```

**例:**

```
shadow-drive-cli get-object-data FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB example.txt
```

### **DeleteFile**

ストレージアカウントからファイルを削除します。

**Parameters:**

`--storage-account`

* Pubkey

`--filename`

* String

**例:**

```
shadow-drive-cli delete-file --storage-account FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB --filename example.txt
```

**例:**

```
shadow-drive-cli delete-file FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB example.txt
```

### **EditFile**

ストレージアカウント内のファイルを編集します。

**Parameters:**

`--storage-account`

* Pubkey

`--path`

* PathBuf

**例:**

```
shadow-drive-cli edit-file --storage-account FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB --path /path/to/new/file.txt
```

**例:**

```
shadow-drive-cli edit-file FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB /path/to/new/file.txt
```

### **StoreFiles**

ストレージアカウントに1つまたは複数のファイルをアップロードします。

**Parameters:**

`--batch-size`

* usize (default: value of FILE\_UPLOAD\_BATCH\_SIZE)

`--storage-account`

* Pubkey

`--files`

* Vec\<PathBuf>

**例:**

```
shadow-drive-cli store-files --batch-size 100 --storage-account FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB file1.txt file2.txt
```

**例:**

```
shadow-drive-cli store-files FKDU64ffTrQq3E1sZsNknefrvY8WkKzCpRyRfptTnyvB file1.txt file2.txt
```
