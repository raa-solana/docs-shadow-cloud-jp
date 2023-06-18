# Python

* **[Python SDK をインストールする](sdk-python.md#installing-shadow-drive)**
* **[例](sdk-python.md#example-end-to-end-script)**
* **[Github](sdk-python.md#about-the-github-repo)**
* **[メソッド](sdk-python.md#methods)**

## **Installing Shadow Drive**

`pip install shadow-drive`

また、examples 用にも:

`pip install solders`

機能のデモンストレーションは、[`examples`](https://github.com/GenesysGo/shadow-drive-rust/tree/main/py)ディレクトリで確認してください。

https://shdw-drive.genesysgo.net/\[STORAGE\_ACCOUNT\_ADDRESS]

### **Example -** Pythonを用いたアカウント作成、ファイルアップロード、ストレージの追加・削減、アカウント削除について

```python
from shadow_drive import ShadowDriveClient
from solders.keypair import Keypair
import argparse

parser = argparse.ArgumentParser()
parser.add_argument('--keypair', metavar='keypair', type=str, required=True,
                    help='The keypair file to use (e.g. keypair.json, dev.json)')
args = parser.parse_args()

# クライアント初期化
client = ShadowDriveClient(args.keypair)
print("Initialized client")

# アカウント作成
size = 2 ** 20
account, tx = client.create_account("full_test", size, use_account=True)
print(f"Created storage account {account}")

# アップロードするファイルを定義
files = ["./files/alpha.txt", "./files/not_alpha.txt"]
urls = client.upload_files(files)
print("Uploaded files")

# ストレージの追加と削減
client.add_storage(2**20)
client.reduce_storage(2**20)

# ファイル取得
current_files = client.list_files()
file = client.get_file(current_files[0])
print(f"got file {file}")

# ファイル削除
client.delete_files(urls)
print("Deleted files")

# アカウント削除
client.delete_account(account)
print("Closed account")
```

#### **[**Github Repo**](https://github.com/GenesysGo/shadow-drive-rust/tree/main/py)について**
本パッケージは、PyO3を使って公式のShadow Drive Rust SDKのラッパーを構築します。詳しくは[Rust SDK documentation](https://github.com/GenesysGo/shadow-drive-rust/tree/main/sdk)を参照してください。

#### **Methods**

Section under development.
