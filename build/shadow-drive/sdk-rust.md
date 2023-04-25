# **Contents**
* **[インストール](#install)**
* **[例](#example)**
* **[メソッド](#methods)**
    * **[add_immutable_storage](#add_immutable_storage)**
    * **[add_storage](#add_storage)**
    * **[cancel_delete_storage_account](#cancel_delete_storage_account)**
    * **[claim_stake](#claim_stake)**
    * **[create_storage_account](#create_storage_account)**
    * **[delete_file](#delete_file)**
    * **[delete_storage_account](#delete_storage_account)** 
    * **[edit_file](#edit_file)**
    * **[get_object_data](#get_object_data)**
    * **[get_storage_account](#get_storage_account)**
    * **[get_storage_accounts](#get_storage_accounts)**
    * **[list_objects](#list_objects)**
    * **[make_storage_immutable](#make_storage_immutable)**
    * **[migrate](#migrate)**
    * **[migrate_step_1](#migrate_step_1)**
    * **[migrate_step_2](#migrate_step_2)**
    * **[new](#new)**
    * **[new_with_rpc](#new_with_rpc)**
    * **[redeem_rent](#redeem_rent)**
    * **[reduce_storage](#reduce_storage)**
    * **[store_files](#store_files)**
* **[その他の例](#add_immutable_storage)**

## **Install**

Rust SDKは[crates.io](https://crates.io/crates/shadow-drive-sdk)、Rust SDK [Github](https://github.com/GenesysGo/shadow-drive-rust)で公開されています。

プロジェクトディレクトリで以下のCargoコマンドを実行します：  
`cargo add shadow-drive-sdk`

または、Cargo.tomlに以下の行を追加します。 
`shadow-drive-sdk = "0.6.1"`

#### **私たちの[Github](https://github.com/GenesysGo/shadow-drive-rust/blob/main/sdk/examples/end_to_end.rs)には、より多くの例が掲載されています。**

## **Example**

```rust
    //init tracing.rs subscriber
    tracing_subscriber::fmt()
        .with_env_filter("off,shadow_drive_rust=debug")
        .init();

    // ファイルからキーペアを読み込む
    let keypair = read_keypair_file(KEYPAIR_PATH).expect("failed to load keypair at path");

    // Shadow Drive クライアント作成
    let shdw_drive_client = ShadowDriveClient::new(keypair, "https://ssc-dao.genesysgo.net");

    // ストレージアカウントpubkeyを取得する
    let pubkey = keypair.pubkey();
    let (storage_account_key, _) =
        shadow_drive_rust::derived_addresses::storage_account(&pubkey, 0);

    // ディレクトリ内のファイルを読み込む
    let dir = tokio::fs::read_dir("multiple_uploads")
    .await
    .expect("failed to read multiple uploads dir");

    // アップロード用のShadowFile構造体Vectorを作成する。
    let mut files = tokio_stream::wrappers::ReadDirStream::new(dir)
        .filter(Result::is_ok)
        .and_then(|entry| async move {
            Ok(ShadowFile::file(
                entry
                    .file_name()
                    .into_string()
                    .expect("failed to convert os string to regular string"),
                entry.path(),
            ))
        })
        .collect::<Result<Vec<_>, _>>()
        .await
        .expect("failed to create shdw files for dir");

    // Bytes もサポートしています
    files.push(ShadowFile::bytes(
        String::from("buf.txt"),
        &b"this is a buf test"[..],
    ));

    // アップロード開始
    let upload_results = shdw_drive_client
        .upload_multiple_files(&storage_account_key, files)
        .await
        .expect("failed to upload files");

    //profit
    println!("upload results: {:#?}", upload_results);
```

# **Methods**

## **`add_immutable_storage`**

### **Definition**
指定された不変の `StorageAccount` にストレージ容量を追加します。StorageAccount` が immutable でない場合は、失敗します。

### **Parameters**
* `storage_account_key` - 不変な `StorageAccount` の公開鍵です。
* `size` - 追加したいストレージの量。例えば、既存の StorageAccount に 1MB のストレージがあり、合計 2MB が必要な場合、size は 1MB となります。サイズを指定する場合、現在サポートされているストレージ単位はKB、MB、GBのみです。

### **Example**
```rust
let add_immutable_storage_response = shdw_drive_client
    .add_immutable_storage(storage_account_key, Byte::from_str("1MB").expect("invalid byte string"))
    .await?;
```

### **Response**
```json
{
    message: String,
    transaction_signature: String,
    error: Option<String>,
}
```

## **`add_storage`**

### **Definition**
指定された StorageAccount にストレージ容量を追加します。

### **Parameters**

* `storage_account_key` - StorageAccount の公開鍵です。
* `size` - 追加したいストレージの量です。例：既存の StorageAccount に 1MB のストレージがあり、合計 2MB が必要な場合、size は 1MB となります。サイズを指定する場合、現在サポートされているストレージ単位はKB、MB、GBのみです。

### **Example**
```rust
let add_immutable_storage_response = shdw_drive_client
    .add_immutable_storage(storage_account_key, Byte::from_str("1MB").expect("invalid byte string"))
    .await?;
```

### **Response**
```json
{
    message: String,
    transaction_signature: String,
    error: Option<String>,
}
```

## **`cancel_delete_storage_account`**

### **Definition**

Shadow Driveから削除する StorageAccount のマークを解除します。削除を防ぐために、このメソッドは `delete_storage_account` が呼び出された Solana エポックの終了前に呼び出す必要があります。

### **Parameters**
* `storage_account_key` - 削除のマークを解除したい `StorageAccount` の公開鍵です。

### **Example**

```rust
let cancel_delete_storage_account_response = shdw_drive_client
    .cancel_delete_storage_account(&storage_account_key)
    .await?;
```
### **Response**

```json
{
    txid: String,
}
```

## **`claim_stake`**

### **Definition**
`reduce_storage`コマンド後、利用可能なステークを請求することができます。ストレージを削減した後、ユーザはエポックの終わりまで待たなければ、ステークを正常にクレームすることが出来ません。

### **Parameters**
* `storage_account_key` - 超過分のステークをクレームしたい StorageAccount の公開鍵です。

### **Example**
```rust
let claim_stake_response = shdw_drive_client
    .claim_stake(&storage_account_key)
    .await?;
```

### **Response**

```json
{
    txid: String,
}
```

## **`create_storage_account`**

### **Definition**

Shadow Drive上に `StorageAccount` を作成します。`StorageAccount`は複数のファイルを保持することができ、SHDWトークンを使用して料金を支払います。

### **Parameters**
* `name` - `StorageAccount` の名前。一意である必要はありません。
* `size` - `StorageAccount` が初期化されるべきストレージの量。サイズを指定する場合、現在は KB、MB、GB のストレージ単位のみがサポートされています。

### **Example**

この方法の使用例は、同じ[githubリポジトリ](https://github.com/phantom-labs/shadow_sdk/blob/master/examples/end_to_end.rs)にも掲載されています。

```rust
async fn main() {
    // キーペア取得
    let keypair_file: String = std::env::args()
        .skip(1)
        .next()
        .expect("no keypair file provided");
    let keypair: Keypair = read_keypair_file(keypair_file).expect("failed to read keypair file");
    println!("loaded keypair");

    // クライアント初期化
    let client = ShadowDriveClient::new(keypair, SOLANA_MAINNET_RPC);
    println!("initialized client");

    // アカウント作成
    let response = client
        .create_storage_account(
            "test",
            Byte::from_bytes(2_u128.pow(20)),
            shadow_drive_sdk::StorageAccountVersion::V2,
        ).await?;
    Ok(())
}
```

### **Response**

```json
{
    message: String,
    transaction_signature: String,
    storage_account_address: String,
    error: Option<String>,
}
```

## **`delete_file`**

### **Definition**

Shadow Driveから削除するためにファイルをマークします。削除のためにマークされたファイルは、Solanaのエポックの終了時に削除されます。削除のためにマークされたファイルは、cancel_delete_fileで取り消すことができますが、これはSolanaエポックが終了する前に行う必要があります。

### **Parameters**

* `storage_account_key` - ファイルが格納されている `StorageAccount` の公開キーです。
* `url` - 削除マークを付けたいファイルの Shadow Drive の URL です。

### **Example**

```rust
let delete_file_response = shdw_drive_client
    .delete_file(&storage_account_key, url)
    .await?;
```

この方法の使用例は、[githubリポジトリ](https://github.com/phantom-labs/shadow_sdk/blob/master/examples/end_to_end.rs)にも掲載されています。

```rust
async fn main() {
    // キーペア取得
    let keypair_file: String = std::env::args()
        .skip(1)
        .next()
        .expect("no keypair file provided");
    let keypair: Keypair = read_keypair_file(keypair_file).expect("failed to read keypair file");
    println!("loaded keypair");

    // クライアント初期化
    let client = ShadowDriveClient::new(keypair, SOLANA_MAINNET_RPC);
    println!("initialized client");

    // アカウント作成
    let response = client
        .create_storage_account(
            "test",
            Byte::from_bytes(2_u128.pow(20)),
            shadow_drive_sdk::StorageAccountVersion::V2,
        )
        .await
        .expect("failed to create storage account");
    let account = Pubkey::from_str(&response.shdw_bucket.unwrap()).unwrap();
    println!("created storage account");

    // アップロードするファイルを定義
    let files: Vec<ShadowFile> = vec![
        ShadowFile::file("alpha.txt".to_string(), "./examples/files/alpha.txt"),
        ShadowFile::file(
            "not_alpha.txt".to_string(),
            "./examples/files/not_alpha.txt",
        ),
    ];
    let response = client
        .store_files(&account, files.clone())
        .await
        .expect("failed to upload files");
    println!("uploaded files");
    for url in &response.finalized_locations {
        println!("    {url}")
    }

    // 編集してみてください
    for file in files {
        let response = client
            .edit_file(&account, file)
            .await
            .expect("failed to upload files");
        assert!(!response.finalized_location.is_empty(), "failed edit");
        println!("edited file: {}", response.finalized_location);
    }

    // ファイル削除
    for url in response.finalized_locations {
        client
            .delete_file(&account, url)
            .await
            .expect("failed to delete files");
    }
}
```

## **`delete_storage_account`**

### **Definition**

この関数は、シャドウドライブから削除するために StorageAccount をマークします。アカウントに削除マークが付けられると、アカウント内のすべてのファイルも削除されます。StorageAccountに残っているステークは、作成者に返金されます。削除のマークが付けられたアカウントは、solanaエポックの終了時に削除されます。

### **Parameters**
* `storage_account_key` - 削除したいStorageAccountの公開鍵です。

### **Response**

* このメソッドは、現在のsolanaエポックが終了する前に、アカウントを削除するためのマークしたアカウントに残っているステークを払い戻すことに成功した場合、成功を返します。

### Example

```rust
let delete_storage_account_response = shdw_drive_client
    .delete_storage_account(&storage_account_key)
    .await?;
```
この方法の使用例は、[githubリポジトリ](https://github.com/phantom-labs/shadow_sdk/blob/master/examples/end_to_end.rs)の71行目にも掲載されています。


## **`edit_file`**

### **Definition**

シャドウドライブ上の既存のファイルを、指定された更新されたファイルに置き換えます。

### **Parameters**

* `storage_account_key` - ファイルが格納されている `StorageAccount` の公開鍵です。
* `url` - 置換したいファイルのあるShadow Driveの URL。
* `data` - 更新された `ShadowFile` です。

### **Example**

```rust
let edit_file_response = shdw_drive_client
    .edit_file(&storage_account_key, url, file)
    .await?;
```

### **Response**

```json
{
    finalized_location: String,
    error: String,
}
```

[リポジトリ](https://github.com/GenesysGo/shadow-drive-rust/blob/main/sdk/examples/end_to_end.rs)にある例。

ファイル：examples/end_to_end.rs, Line 53

```rust
async fn main() {
    // Get keypair
    let keypair_file: String = std::env::args()
        .skip(1)
        .next()
        .expect("no keypair file provided");
    let keypair: Keypair = read_keypair_file(keypair_file).expect("failed to read keypair file");
    println!("loaded keypair");

    // Initialize client
    let client = ShadowDriveClient::new(keypair, SOLANA_MAINNET_RPC);
    println!("initialized client");

    // Create account
    let response = client
        .create_storage_account(
            "test",
            Byte::from_bytes(2_u128.pow(20)),
            shadow_drive_sdk::StorageAccountVersion::V2,
        )
        .await
        .expect("failed to create storage account");
    let account = Pubkey::from_str(&response.shdw_bucket.unwrap()).unwrap();
    println!("created storage account");

    // Upload files
    let files: Vec<ShadowFile> = vec![
        ShadowFile::file("alpha.txt".to_string(), "./examples/files/alpha.txt"),
        ShadowFile::file(
            "not_alpha.txt".to_string(),
            "./examples/files/not_alpha.txt",
        ),
    ];
    let response = client
        .store_files(&account, files.clone())
        .await
        .expect("failed to upload files");
    println!("uploaded files");
    for url in &response.finalized_locations {
        println!("    {url}")
    }
    // Try editing
    for file in files {
        let response = client
            .edit_file(&account, file)
            .await
            .expect("failed to upload files");
    }    
}
```
## **`get_object_data`**

オブジェクトのデータを取得します

## **`get_storage_account`**

### **Definition**

ユーザーが提供したpubkeyに関連付けられた `StorageAccount` を返します。

### **Parameters**

* `key` - `StorageAccount` の公開鍵です。

### **Example**

```rust
let storage_account = shdw_drive_client
    .get_storage_account(&storage_account_key)
    .await
    .expect("failed to get storage account");
```

### **Response for V1 StorageAccount**

```json
{
    storage_account: Pubkey,
    reserved_bytes: u64,
    current_usage: u64,
    immutable: bool,
    to_be_deleted: bool,
    delete_request_epoch: u32,
    owner_1: Pubkey,
    owner_2: Pubkey,
    account_counter_seed: u32,
    creation_time: u32,
    creation_epoch: u32,
    last_fee_epoch: u32,
    identifier: String,
}
```
### **Response for V2 StorageAccount**
```json
{
    storage_account: Pubkey,
    reserved_bytes: u64,
    current_usage: u64,
    immutable: bool,
    to_be_deleted: bool,
    delete_request_epoch: u32,
    owner_1: Pubkey,
    account_counter_seed: u32,
    creation_time: u32,
    creation_epoch: u32,
    last_fee_epoch: u32,
    identifier: String,
}
```

## **`get_storage_accounts`**

### **Definition**

ユーザーが提供した公開鍵に関連する全ての `StorageAccounts` を返します。

### **Parameters**

* `owner` - 返されたすべての `StorageAccounts` の所有者である公開鍵です。

### **Example**

```rust
let storage_accounts = shdw_drive_client
    .get_storage_accounts(&user_pubkey)
    .await
    .expect("failed to get storage account");
```

### **Response for V1 StorageAccount**

```json
{

    storage_account: Pubkey,
    reserved_bytes: u64,
    current_usage: u64,
    immutable: bool,
    to_be_deleted: bool,
    delete_request_epoch: u32,
    owner_1: Pubkey,
    owner_2: Pubkey,
    account_counter_seed: u32,
    creation_time: u32,
    creation_epoch: u32,
    last_fee_epoch: u32,
    identifier: String,
}
```
### **Response for V2 StorageAccount**
```json
{
    storage_account: Pubkey,
    reserved_bytes: u64,
    current_usage: u64,
    immutable: bool,
    to_be_deleted: bool,
    delete_request_epoch: u32,
    owner_1: Pubkey,
    account_counter_seed: u32,
    creation_time: u32,
    creation_epoch: u32,
    last_fee_epoch: u32,
    identifier: String,
}
```

## **`list_objects`**

### **Definition**
`StorageAccount`に関連付けられたすべてのファイルのリストを取得します。出力には、すべてのファイル名が文字列として含まれます。

### **Parameters**

* `storage_account_key` - ファイルを所有する `StorageAccount` の公開鍵です。

### **Example**

```rust
let files = shdw_drive_client
    .list_objects(&storage_account_key)
    .await?;
```

### **Response**
注：レスポンスは、すべてのファイル名を文字列として含むvectorです。

## **`make_storage_immutable`**

### **Definition**

`StorageAccount`とその中に含まれるすべてのファイルを永久にロックします。ロックされた `StorageAccount` は、ファイルの削除・編集、ストレージ容量の追加・削減、`StorageAccount` の削除ができなくなります。

### **Parameters**

* `storage_account_key` - 不変にする `StorageAccount` の公開鍵です。

### **Example**

```rust
let make_immutable_response = shdw_drive_client  
    .make_storage_immutable(&storage_account_key)  
    .await?;  
```

### **Response**

```json
{
    message: String,
    transaction_signature: String,
    error: Option<String>,
}
```

## **`migrate`**

### **Definition**

これは、元のパブキーを再利用するために2つの別々のトランザクションを必要とします。失敗の可能性を最小限にするために、コミットメント・レベルを Finalized にしてこのメソッドを呼び出すことが推奨されます。

### **Parameters**

* `storage_account_key` - 移行する StorageAccount の公開鍵です。

### **Example**

```rust
let migrate_response = shdw_drive_client
    .migrate(&storage_account_key)
    .await?;
```

###  **Result**
```json
{
    txid: String,
}
```

## **`migrate_step_1`**

### **Definition**

v1 の `StorageAccount` を v2 に移行する最初のトランザクションステップ。既存のアカウントのデータを中間アカウントにコピーし、v1 の `StorageAccount` を削除することで構成されます。

## **`migrate_step_2`**

### **Definition**

v1 の `StorageAccount` を v2 に移行する 2 番目のトランザクションステップ。元の pubkey を使用して `StorageAccount` を再作成し、中間アカウントを削除することから構成されます。


## **`new`**

### **Definition**

与えられた `Signer` と URL から新しい ShadowDriveClient を作成します。

### **Parameters**

* `wallet` - クライアントが生成するすべてのトランザクションに署名するための `Signer` です。一般的に、これはユーザーのキーペアです。
* `rpc_url` - Solana RPCプロバイダのHTTP URLです。

RpcClientの設定をカスタマイズするには、`new_with_rpc`を参照してください。

### **Example**

```rust
use solana_sdk::signer::keypair::Keypair;    

let wallet = Keypair::generate();
let shdw_drive = ShadowDriveClient::new(wallet, "https://ssc-dao.genesysgo.net");
```

[リポジトリ](https://github.com/GenesysGo/shadow-drive-rust/blob/main/sdk/examples/end_to_end.rs)にある例。

`examples/end_to_end.rs` Line 19

```rust
async fn main() {
    // Get keypair
    let keypair_file: String = std::env::args()
        .skip(1)
        .next()
        .expect("no keypair file provided");
    let keypair: Keypair = read_keypair_file(keypair_file).expect("failed to read keypair file");
    println!("loaded keypair");

    // Initialize client
    let client = ShadowDriveClient::new(keypair, SOLANA_MAINNET_RPC);
    println!("initialized client");
}
```

## **`new_with_rpc`**

### **Definition**

与えられた `Signer` と `RpcClient` から新しい ShadowDriveClient を作成します。

### **Parameters**

* `wallet` - クライアントが生成するすべてのトランザクションに署名するための `Signer` です。一般的に、これはユーザーのキーペアです。
* トランザクションの送信とブロックチェーンからのアカウントの読み取りを処理する Solana `RpcClient` です。 
    `RpcClient`を提供することで、タイムアウトとコミットメントレベルのカスタマイズが可能になります。

### **Example**

```rust
use solana_client::rpc_client::RpcClient;
use solana_sdk::signer::keypair::Keypair;    
use solana_sdk::commitment_config::CommitmentConfig;

let wallet = Keypair::generate();
let solana_rpc = RpcClient::new_with_commitment("https://ssc-dao.genesysgo.net", CommitmentConfig::confirmed());
let shdw_drive = ShadowDriveClient::new_with_rpc(wallet, solana_rpc);
```

## **`redeem_rent`**

### **Definition**

オンチェーンファイルのアカウントからsolanaレントをリクレームします。古いバージョンのShadow Driveは、アップロードされたファイルのアカウントを作成するために使用されました。

### **Parameters**

* `storage_account_key` - 削除されたファイルが格納されていた StorageAccount の公開鍵です。
* `file_account_key` - 閉鎖されるファイルアカウントの公開鍵です。

### **Example**

```rust
let redeem_rent_response = shdw_drive_client
    .redeem_rent(&storage_account_key, &file_account_key)
    .await?;
```

### **Response**
```json
{
  message: String,
  transaction_signature: String,
  error: Option<String>,
}
```

## **`reduce_storage`**

### **Definition**

与えられた `StorageAccount` で利用可能な総ストレージ量を削減します。

### **Parameters**

* `storage_account_key` - ストレージを削減する `StorageAccount` の公開鍵です。
* `size` - 削除したいストレージの量。例えば、既存の `StorageAccount` に 3MB のストレージがあり、合計 2MB にしたい場合、size は 1MB となります。サイズを指定する場合、現在サポートされているストレージ単位はKB、MB、GBのみです。

### **Example**

```rust
let reduce_storage_response = shdw_drive_client
    .reduce_storage(&storage_account_key, reduced_bytes)
    .await?;
```

### **Response**

```json
{
  message: String,
  transaction_signature: String,
  error: Option<String>,
}
```

## **`store_files`**

### **Definition**

指定された `StorageAccount` にファイルを保存します。

### **Parameters**

* `storage_account_key` - `StorageAccount` の公開鍵です。
* `data` - 格納されるファイルを表す `ShadowFile` オブジェクトのベクトルです。

### **Example**

```rust
let files: Vec<ShadowFile> = vec![
    ShadowFile::file("alpha.txt".to_string(), "./examples/files/alpha.txt"),
    ShadowFile::file(
        "not_alpha.txt".to_string(),
        "./examples/files/not_alpha.txt",
    ),
];
let store_files_response = shdw_drive_client
    .store_files(&storage_account_key, files)
    .await?;
```

### **Response**

```json
{
  finalized_locations: Array<string>;
  message: string;
  upload_errors: Array<UploadError>;
}
```

## **Example - Add Immutable Storage**

```rust
use byte_unit::Byte;
use shadow_drive_rust::{
    models::storage_acct::StorageAcct, ShadowDriveClient, StorageAccountVersion,
};
use solana_sdk::{
    pubkey::Pubkey,
    signer::{keypair::read_keypair_file, Signer},
};
use std::str::FromStr;

const KEYPAIR_PATH: &str = "/Users/dboures/.config/solana/id.json";

#[tokio::main]
async fn main() {
    //ファイルからキーペアを読み込む
    let keypair = read_keypair_file(KEYPAIR_PATH).expect("failed to load keypair at path");

    //shdw drive クライアントを作成する
    let shdw_drive_client = ShadowDriveClient::new(keypair, "https://ssc-dao.genesysgo.net");

    // // 1.
    // create_storage_accounts(shdw_drive_client).await;

    // // 2.
    // let v1_pubkey = Pubkey::from_str("J4RJYandDDKxyF6V1HAdShDSbMXk78izZ2yEksqyvGmo").unwrap();
    let v2_pubkey = Pubkey::from_str("9dXUV4BEKWohSRDn4cy5G7JkhUDWoSUGGwJngrSg453r").unwrap();

    // make_storage_immutable(&shdw_drive_client, &v1_pubkey).await;
    // make_storage_immutable(&shdw_drive_client, &v2_pubkey).await;

    // // 3.
    // add_immutable_storage_test(&shdw_drive_client, &v1_pubkey).await;
    add_immutable_storage_test(&shdw_drive_client, &v2_pubkey).await;
}

async fn create_storage_accounts<T: Signer>(shdw_drive_client: ShadowDriveClient<T>) {
    let result_v1 = shdw_drive_client
        .create_storage_account(
            "shdw-drive-1.5-test-v1",
            Byte::from_str("1MB").expect("invalid byte string"),
            StorageAccountVersion::v1(),
        )
        .await
        .expect("error creating storage account");

    let result_v2 = shdw_drive_client
        .create_storage_account(
            "shdw-drive-1.5-test-v2",
            Byte::from_str("1MB").expect("invalid byte string"),
            StorageAccountVersion::v2(),
        )
        .await
        .expect("error creating storage account");

    println!("v1: {:?}", result_v1);
    println!("v2: {:?}", result_v2);
}

async fn make_storage_immutable<T: Signer>(
    shdw_drive_client: &ShadowDriveClient<T>,
    storage_account_key: &Pubkey,
) {
    let storage_account = shdw_drive_client
        .get_storage_account(storage_account_key)
        .await
        .expect("failed to get storage account");
    match storage_account {
        StorageAcct::V1(storage_account) => println!("account: {:?}", storage_account),
        StorageAcct::V2(storage_account) => println!("account: {:?}", storage_account),
    }

    let make_immutable_response = shdw_drive_client
        .make_storage_immutable(&storage_account_key)
        .await
        .expect("failed to make storage immutable");

    println!("response: {:?}", make_immutable_response);

    let storage_account = shdw_drive_client
        .get_storage_account(&storage_account_key)
        .await
        .expect("failed to get storage account");
    match storage_account {
        StorageAcct::V1(storage_account) => println!("account: {:?}", storage_account),
        StorageAcct::V2(storage_account) => println!("account: {:?}", storage_account),
    }
}

async fn add_immutable_storage_test<T: Signer>(
    shdw_drive_client: &ShadowDriveClient<T>,
    storage_account_key: &Pubkey,
) {
    let storage_account = shdw_drive_client
        .get_storage_account(&storage_account_key)
        .await
        .expect("failed to get storage account");

    match storage_account {
        StorageAcct::V1(storage_account) => {
            println!("old size: {:?}", storage_account.reserved_bytes)
        }
        StorageAcct::V2(storage_account) => {
            println!("old size: {:?}", storage_account.reserved_bytes)
        }
    }

    let add_immutable_storage_response = shdw_drive_client
        .add_immutable_storage(
            storage_account_key,
            Byte::from_str("1MB").expect("invalid byte string"),
        )
        .await
        .expect("error adding storage");

    println!("response: {:?}", add_immutable_storage_response);

    let storage_account = shdw_drive_client
        .get_storage_account(&storage_account_key)
        .await
        .expect("failed to get storage account");

    match storage_account {
        StorageAcct::V1(storage_account) => {
            println!("new size: {:?}", storage_account.reserved_bytes)
        }
        StorageAcct::V2(storage_account) => {
            println!("new size: {:?}", storage_account.reserved_bytes)
        }
    }
}
```

## **Example - Cancel Delete Storage Accounts**

```rust
use shadow_drive_rust::ShadowDriveClient;
use solana_sdk::{pubkey::Pubkey, signer::keypair::read_keypair_file};
use std::str::FromStr;

const KEYPAIR_PATH: &str = "keypair.json";

#[tokio::main]
async fn main() {
    //ファイルからキーペアを読み込む
    let keypair = read_keypair_file(KEYPAIR_PATH).expect("failed to load keypair at path");
    let storage_account_key =
        Pubkey::from_str("GHSNTDyMmay7xDjBNd9dqoHTGD3neioLk5VJg2q3fJqr").unwrap();

    //shdw drive クライアントを作成する
    let shdw_drive_client = ShadowDriveClient::new(keypair, "https://ssc-dao.genesysgo.net");

    let response = shdw_drive_client
        .cancel_delete_storage_account(&storage_account_key)
        .await
        .expect("failed to cancel storage account deletion");

    println!("Unmark delete storage account complete {:?}", response);
}
```

## **Example - Claim Stake**

```rust
use shadow_drive_rust::ShadowDriveClient;
use solana_sdk::{pubkey::Pubkey, signer::keypair::read_keypair_file};
use std::str::FromStr;

const KEYPAIR_PATH: &str = "keypair.json";

/// この例では、なかなかうまくいきません。
/// claim_stakeは、アカウントのストレージ量を減らした後にSHDWと交換するために使用されます
/// claim_stakeを成功させるためには、ストレージを減らしてから1エポック待つ必要があります。
/// 還元と同じエポックにclaim_stakeをしようとすると
/// "custom program error: 0x1775"
/// "Error Code: ClaimingStakeTooSoon"

#[tokio::main]
async fn main() {
    //ファイルからキーペアを読み込む
    let keypair = read_keypair_file(KEYPAIR_PATH).expect("failed to load keypair at path");
    let storage_account_key =
        Pubkey::from_str("GHSNTDyMmay7xDjBNd9dqoHTGD3neioLk5VJg2q3fJqr").unwrap();

    //create shdw drive client
    let shdw_drive_client = ShadowDriveClient::new(keypair, "https://ssc-dao.genesysgo.net");

    let url = String::from(
        "https://shdw-drive.genesysgo.net/B7Qk2omAvchkePhzHubCVQuVpZHcieqPQCwFxeeBZGuT/hey.txt",
    );

    // ストレージを削る

    // let reduce_storage_response = shdw_drive_client
    //     .reduce_storage(
    //         storage_account_key,
    //         Byte::from_str("100KB").expect("invalid byte string"),
    //     )
    //     .await
    //     .expect("error adding storage");

    // println!("txn id: {:?}", reduce_storage_response.txid);

    // 1エポック待って

    // クレームする
    // let claim_stake_response = shdw_drive_client
    //     .claim_stake(storage_account_key)
    //     .await
    //     .expect("failed to claim stake");

    // println!(
    //     "Claim stake complete {:?}",
    //     claim_stake_response
    // );
}
```

## **Example - Delete File**

```rust
use shadow_drive_rust::{models::ShadowFile, ShadowDriveClient};
use solana_sdk::{pubkey::Pubkey, signer::keypair::read_keypair_file};
use std::str::FromStr;

const KEYPAIR_PATH: &str = "keypair.json";

#[tokio::main]
async fn main() {
    //ファイルからキーペアを読み込む
    let keypair = read_keypair_file(KEYPAIR_PATH).expect("failed to load keypair at path");

    let v1_pubkey = Pubkey::from_str("GSvvRguVTtSayF5zLQPLVTJQHQ6Fqu1Z3HSRMP8AHY9K").unwrap();
    let v2_pubkey = Pubkey::from_str("2cvgcqfmMg9ioFtNf57ZqCNbuWDfB8ZSzromLS8Kkb7q").unwrap();

    //shdw drive クライアントを作成する
    let shdw_drive_client = ShadowDriveClient::new(keypair, "https://ssc-dao.genesysgo.net");

    //ファイル追加
    let v1_upload_reponse = shdw_drive_client
        .store_files(
            &v1_pubkey,
            vec![ShadowFile::file(
                String::from("example.png"),
                "multiple_uploads/0.txt",
            )],
        )
        .await
        .expect("failed to upload v1 file");
    println!("Upload complete {:?}", v1_upload_reponse);

    let v2_upload_reponse = shdw_drive_client
        .store_files(
            &v2_pubkey,
            vec![ShadowFile::file(
                String::from("example.png"),
                "multiple_uploads/0.txt",
            )],
        )
        .await
        .expect("failed to upload v2 file");

    println!("Upload complete {:?}", v2_upload_reponse);

    let v1_url = String::from(
        "https://shdw-drive.genesysgo.net/GSvvRguVTtSayF5zLQPLVTJQHQ6Fqu1Z3HSRMP8AHY9K/example.png",
    );
    let v2_url = String::from(
        "https://shdw-drive.genesysgo.net/2cvgcqfmMg9ioFtNf57ZqCNbuWDfB8ZSzromLS8Kkb7q/example.png",
    );

    //ファイル削除
    let v1_delete_file_response = shdw_drive_client
        .delete_file(&v1_pubkey, v1_url)
        .await
        .expect("failed to delete file");
    println!("Delete file complete {:?}", v1_delete_file_response);

    let v2_delete_file_response = shdw_drive_client
        .delete_file(&v2_pubkey, v2_url)
        .await
        .expect("failed to delete file");
    println!("Delete file complete {:?}", v2_delete_file_response);
}
```

## **Example - Delete Storage Account**

```rust
use shadow_drive_rust::ShadowDriveClient;
use solana_sdk::{pubkey::Pubkey, signer::keypair::read_keypair_file};
use std::str::FromStr;

const KEYPAIR_PATH: &str = "keypair.json";

#[tokio::main]
async fn main() {
    //ファイルからキーペアを読み込む
    let keypair = read_keypair_file(KEYPAIR_PATH).expect("failed to load keypair at path");
    let storage_account_key =
        Pubkey::from_str("9VndNFwL7cVTshY2x5GAjKQusRCAsDU4zynYg76xTguo").unwrap();

    //shdw drive クライアントを作成する
    let shdw_drive_client = ShadowDriveClient::new(keypair, "https://ssc-dao.genesysgo.net");

    let response = shdw_drive_client
        .delete_storage_account(&storage_account_key)
        .await
        .expect("failed to request storage account deletion");

    println!("Delete Storage Account complete {:?}", response);
}
```

## **Example - Tests**

```rust
use byte_unit::Byte;
use shadow_drive_rust::{models::ShadowFile, ShadowDriveClient, StorageAccountVersion};
use solana_sdk::{
    pubkey,
    pubkey::Pubkey,
    signer::{keypair::read_keypair_file, Signer},
};

const KEYPAIR_PATH: &str = "keypair.json";

#[tokio::main]
async fn main() {
    //ファイルからキーペアを読み込む
    let keypair = read_keypair_file(KEYPAIR_PATH).expect("failed to load keypair at path");
    // let pubkey = keypair.pubkey();
    // let (storage_account_key, _) =
    //     shadow_drive_rust::derived_addresses::storage_account(&pubkey, 0);

    let storage_account_key = pubkey!("G6nE9EbNgSDcvUvs67enP2Jba3exgLyStgsg8S7n9StS");

    //shdw drive クライアントを作成する
    let shdw_drive_client = ShadowDriveClient::new(keypair, "https://ssc-dao.genesysgo.net");

    // get_storage_accounts_test(shdw_drive_client, &pubkey).await

    // create_storage_account_v2_test(shdw_drive_client).await
    upload_file_test(shdw_drive_client, &storage_account_key).await
}

async fn get_storage_accounts_test<T: Signer>(
    shdw_drive_client: ShadowDriveClient<T>,
    pubkey: &Pubkey,
) {
    let storage_accounts = shdw_drive_client
        .get_storage_accounts(pubkey)
        .await
        .expect("failed to get storage account");
    println!("{:?}", storage_accounts);
}

async fn create_storage_account_v2_test<T: Signer>(shdw_drive_client: ShadowDriveClient<T>) {
    let result = shdw_drive_client
        .create_storage_account(
            "shdw-drive-1.5-test",
            Byte::from_str("10MB").expect("invalid byte string"),
            StorageAccountVersion::v2(),
        )
        .await
        .expect("error creating storage account");
    println!("{:?}", result);
}

// async fn get_object_data_test<T: Signer>(
//     shdw_drive_client: ShadowDriveClient<T>,
//     location: &str,
// ) {
//     let result = shdw_drive_client
//         .get_object_data(location)
//         .await
//         .expect("error getting object data");
//     println!("{:?}", result);
// }

// async fn list_objects_test<T: Signer>(
//     shdw_drive_client: ShadowDriveClient<T>,
//     storage_account_key: &Pubkey,
// ) {
//     let objects = shdw_drive_client
//         .list_objects(storage_account_key)
//         .await
//         .expect("failed to list objects");

//     println!("objects {:?}", objects);
// }

// async fn make_storage_immutable_test<T: Signer>(
//     shdw_drive_client: ShadowDriveClient<T>,
//     storage_account_key: &Pubkey,
// ) {
//     let storage_account = shdw_drive_client
//         .get_storage_account(storage_account_key)
//         .await
//         .expect("failed to get storage account");
//     println!(
//         "identifier: {:?}; immutable: {:?}",
//         storage_account.identifier, storage_account.immutable
//     );

//     let make_immutable_response = shdw_drive_client
//         .make_storage_immutable(&storage_account_key)
//         .await
//         .expect("failed to make storage immutable");

//     println!("txn id: {:?}", make_immutable_response.txid);

//     let storage_account = shdw_drive_client
//         .get_storage_account(&storage_account_key)
//         .await
//         .expect("failed to get storage account");
//     println!(
//         "identifier: {:?}; immutable: {:?}",
//         storage_account.identifier, storage_account.immutable
//     );
// }

// async fn add_storage_test<T: Signer>(
//     shdw_drive_client: &ShadowDriveClient<T>,
//     storage_account_key: &Pubkey,
// ) {
//     let storage_account = shdw_drive_client
//         .get_storage_account(&storage_account_key)
//         .await
//         .expect("failed to get storage account");

//     let add_storage_response = shdw_drive_client
//         .add_storage(
//             storage_account_key,
//             Byte::from_str("10MB").expect("invalid byte string"),
//         )
//         .await
//         .expect("error adding storage");

//     println!("txn id: {:?}", add_storage_response.txid);

//     let storage_account = shdw_drive_client
//         .get_storage_account(&storage_account_key)
//         .await
//         .expect("failed to get storage account");

//     println!("new size: {:?}", storage_account.storage);
// }

// async fn reduce_storage_test<T: Signer>(
//     shdw_drive_client: ShadowDriveClient<T>,
//     storage_account_key: &Pubkey,
// ) {
//     let storage_account = shdw_drive_client
//         .get_storage_account(storage_account_key)
//         .await
//         .expect("failed to get storage account");

//     println!("previous size: {:?}", storage_account.storage);

//     let add_storage_response = shdw_drive_client
//         .reduce_storage(
//             storage_account_key,
//             Byte::from_str("10MB").expect("invalid byte string"),
//         )
//         .await
//         .expect("error adding storage");

//     println!("txn id: {:?}", add_storage_response.txid);

//     let storage_account = shdw_drive_client
//         .get_storage_account(storage_account_key)
//         .await
//         .expect("failed to get storage account");

//     println!("new size: {:?}", storage_account.storage);
// }

async fn upload_file_test<T: Signer>(
    shdw_drive_client: ShadowDriveClient<T>,
    storage_account_key: &Pubkey,
) {
    let upload_reponse = shdw_drive_client
        .store_files(
            storage_account_key,
            vec![ShadowFile::file(String::from("example.png"), "example.png")],
        )
        .await
        .expect("failed to upload file");

    println!("Upload complete {:?}", upload_reponse);
}
```

## **Example - Migrate**

```rust
use byte_unit::Byte;
use shadow_drive_rust::{ShadowDriveClient, StorageAccountVersion};
use solana_sdk::{pubkey::Pubkey, signer::keypair::read_keypair_file};
use std::str::FromStr;

const KEYPAIR_PATH: &str = "keypair.json";

#[tokio::main]
async fn main() {
    //ファイルからキーペアを読み込む
    let keypair = read_keypair_file(KEYPAIR_PATH).expect("failed to load keypair at path");

    //shdw drive クライアントを作成する
    let shdw_drive_client = ShadowDriveClient::new(keypair, "https://ssc-dao.genesysgo.net");

    // V1 ストレージアカウントを作成
    let v1_response = shdw_drive_client
        .create_storage_account(
            "1.5-test",
            Byte::from_str("1MB").expect("invalid byte string"),
            StorageAccountVersion::v1(),
        )
        .await
        .expect("error creating storage account");

    println!("v1: {:?} \n", v1_response);

    let key_string: String = v1_response.shdw_bucket.unwrap();
    let v1_pubkey: Pubkey = Pubkey::from_str(&key_string).unwrap();

    // 一気にマイグレーションが可能
    let migrate = shdw_drive_client
        .migrate(&v1_pubkey)
        .await
        .expect("failed to migrate");
    println!("Migrated {:?} \n", migrate);

    // もしくは、マイグレーションを2つのステップに分けることもできます（どちらのステップも公開されます）

    // // step 1
    // let migrate_step_1 = shdw_drive_client
    //     .migrate_step_1(&v1_pubkey)
    //     .await
    //     .expect("failed to migrate v1 step 1");
    // println!("Step 1 complete {:?} \n", migrate_step_1);

    // // step 2
    // let migrate_step_2 = shdw_drive_client
    //     .migrate_step_2(&v1_pubkey)
    //     .await
    //     .expect("failed to migrate v1 step 2");
    // println!("Step 2 complete {:?} \n", migrate_step_2);
}
```

## **Example - Redeem Rent**

```rust
use shadow_drive_rust::ShadowDriveClient;
use solana_sdk::{pubkey::Pubkey, signer::keypair::read_keypair_file};
use std::str::FromStr;

const KEYPAIR_PATH: &str = "keypair.json";

#[tokio::main]
async fn main() {
    //ファイルからキーペアを読み込む
    let keypair = read_keypair_file(KEYPAIR_PATH).expect("failed to load keypair at path");

    //shdw drive クライアントを作成する
    let shdw_drive_client = ShadowDriveClient::new(keypair, "https://ssc-dao.genesysgo.net");

    let storage_account_key =
        Pubkey::from_str("D7Qk2omAvchkePhzHubCVQuVpZHcieqPQCwFxeeBZGuT").unwrap();

    let file_account_key =
        Pubkey::from_str("B41kFXqFkDhY7kHbMhEk17bP2w7QLUYU9X5tRhDLttnJ").unwrap();

    let redeem_rent_response = shdw_drive_client
        .redeem_rent(&storage_account_key, &file_account_key)
        .await
        .expect("failed to redeem_storage");
    println!("Redeemed {:?} \n", redeem_rent_response);
}
```

## **Example - Upload Multiple Files**

```rust
use byte_unit::Byte;
use futures::TryStreamExt;
use shadow_drive_rust::{models::ShadowFile, ShadowDriveClient, StorageAccountVersion};
use solana_sdk::signer::{keypair::read_keypair_file, Signer};
use tokio_stream::StreamExt;

const KEYPAIR_PATH: &str = "keypair.json";

#[tokio::main]
async fn main() {
    tracing_subscriber::fmt()
        .with_env_filter("off,shadow_drive_rust=debug")
        .init();

    //ファイルからキーペアを読み込む
    let keypair = read_keypair_file(KEYPAIR_PATH).expect("failed to load keypair at path");
    let pubkey = keypair.pubkey();
    let (storage_account_key, _) =
        shadow_drive_rust::derived_addresses::storage_account(&pubkey, 21);

    //shdw drive クライアントを作成する
    let shdw_drive_client = ShadowDriveClient::new(keypair, "https://ssc-dao.genesysgo.net");

    //ストレージアカウントを確保する
    if let Err(_) = shdw_drive_client
        .get_storage_account(&storage_account_key)
        .await
    {
        println!("Error finding storage account, assuming it's not created yet");
        shdw_drive_client
            .create_storage_account(
                "shadow-drive-rust-test-2",
                Byte::from_str("1MB").expect("failed to parse byte string"),
                StorageAccountVersion::v2(),
            )
            .await
            .expect("failed to create storage account");
    }

    let dir = tokio::fs::read_dir("multiple_uploads")
        .await
        .expect("failed to read multiple uploads dir");

    let mut files = tokio_stream::wrappers::ReadDirStream::new(dir)
        .filter(Result::is_ok)
        .and_then(|entry| async move {
            Ok(ShadowFile::file(
                entry
                    .file_name()
                    .into_string()
                    .expect("failed to convert os string to regular string"),
                entry.path(),
            ))
        })
        .collect::<Result<Vec<_>, _>>()
        .await
        .expect("failed to create shdw files for dir");

    files.push(ShadowFile::bytes(
        String::from("buf.txt"),
        &b"this is a buf test"[..],
    ));

    let upload_results = shdw_drive_client
        .store_files(&storage_account_key, files)
        .await
        .expect("failed to upload files");

    println!("upload results: {:#?}", upload_results);
}
```
