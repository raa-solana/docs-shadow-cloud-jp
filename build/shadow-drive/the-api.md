---
description: >-
    Shadow Driveは、APIを公開しており、そのAPIを使用することなく、直接対話することができます。
    CLIまたはSDKを使用します。これらのメソッドの上に構築することもできます。
---

# API

## **Contents**

* [**Example - サインとアップロード**](the-api.md#example-sign-and-upload-a-file)
* [**Example - ファイルの更新**](the-api.md#example-edit-a-file)
* [**Example - ファイルの削除**](the-api.md#example-delete-a-file)

{% swagger method="post" path="" baseUrl="https://shadow-storage.genesysgo.net" summary="storage-account" %}
{% swagger-description %}
storage account の作成

Request content type: application/json
{% endswagger-description %}

{% swagger-parameter in="body" required="true" name="transaction" type="" %}
ストレージアカウント所有者によって部分的に署名された、シリアライズされたストレージアカウント作成トランザクション
{% endswagger-parameter %}

{% swagger-response status="200: OK" description="" %}
```json
{
    "shdw_bucket": String,
    "transaction_signature": String
}
```
{% endswagger-response %}
{% endswagger %}

{% swagger method="post" path="" baseUrl="https://shadow-storage.genesysgo.net" summary="storage-account-info" %}
{% swagger-description %}
ストレージアカウントに関するオンチェーンデータとShadow Driveネットワークデータを取得します。

Request content type: application/json
{% endswagger-description %}

{% swagger-parameter in="body" name="storage_account" required="true" %}
情報を取得したいストレージアカウントのPublickey
{% endswagger-parameter %}

{% swagger-response status="200: OK" description="Response for V1 Storage Account" %}
```json
{
  storage_account: PublicKey,
  reserved_bytes: Number,
  current_usage: Number,
  immutable: Boolean,
  to_be_deleted: Boolean,
  delete_request_epoch: Number,
  owner1: PublicKey,
  owner2: PublicKey,
  accountCoutnerSeed: Number,
  creation_time: Number,
  creation_epoch: Number,
  last_fee_epoch: Number,
  identifier: String
  version: "V1"
}
```
{% endswagger-response %}

{% swagger-response status="200: OK" description="Response for V2 Storage Account" %}
```json
{
  storage_account: PublicKey,
  reserved_bytes: Number,
  current_usage: Number,
  immutable: Boolean,
  to_be_deleted: Boolean,
  delete_request_epoch: Number,
  owner1: PublicKey,
  accountCoutnerSeed: Number,
  creation_time: Number,
  creation_epoch: Number,
  last_fee_epoch: Number,
  identifier: String,
  version: "V2"
}json
```
{% endswagger-response %}
{% endswagger %}

{% swagger method="post" path="" baseUrl="https://shadow-storage.genesysgo.net" summary="upload" %}
{% swagger-description %}
1つのファイル、または複数のファイルを一度にアップロードすることができます。\
\
Request content type: multipart/form-data\
\
[**実装例**](the-api.md#example-sign-and-upload-a-file)

\
Parameters (FormData fields)
{% endswagger-description %}

{% swagger-parameter in="body" name="file" type="" required="true" %}
アップロードしたいファイルです。最大5つのファイルを追加することができ、それぞれのフィールド名は

`file`

.
{% endswagger-parameter %}

{% swagger-parameter in="body" name="message" required="true" %}
Base58 メッセージの署名
{% endswagger-parameter %}

{% swagger-parameter in="body" name="signer" required="true" %}
メッセージ署名の署名者であり、ストレージアカウントの所有者の公開鍵
{% endswagger-parameter %}

{% swagger-parameter in="body" name="storage_account" required="true" %}
アップロード先となるストレージアカウントのキー
{% endswagger-parameter %}

{% swagger-response status="200: OK" description="" %}
```json
{
    "finalized_locations": [String],
    "message": String
    "upload_errors": [{file: String, storage_account: String, error: String}] or [] if no errors
}
```
{% endswagger-response %}
{% endswagger %}

{% swagger method="post" path="" baseUrl="https://shadow-storage.genesysgo.net" summary="edit" %}
{% swagger-description %}
既存のファイルを編集

Request content type: multipart/form-data

[**実装例**](the-api.md#example-edit-a-file)

Parameters (FormData fields)
{% endswagger-description %}

{% swagger-parameter in="body" name="file" required="true" %}
アップロードしたいファイルです。最大5つのファイルを追加することができ、それぞれのフィールド名は

`file`

.
{% endswagger-parameter %}

{% swagger-parameter in="body" name="message" required="true" %}
Base58 メッセージの署名.
{% endswagger-parameter %}

{% swagger-parameter in="body" name="signer" required="true" %}
メッセージ署名の署名者であり、ストレージアカウントの所有者の公開鍵
{% endswagger-parameter %}

{% swagger-parameter in="body" name="storage_account" required="true" %}
アップロード先となるストレージアカウントのキー
{% endswagger-parameter %}

{% swagger-parameter in="body" name="url" required="true" %}
編集したい元ファイルのURL。例:

`https://shdw-drive.genesysgo.net/<storage-account>/<file-name>`
{% endswagger-parameter %}

{% swagger-response status="200: OK" description="" %}
```json
{
    "finalized_location": String,
    "error": String or not provided if no error
}
```
{% endswagger-response %}
{% endswagger %}

{% swagger method="post" path="" baseUrl="https://shadow-storage.genesysgo.net" summary="list-objects" %}
{% swagger-description %}
ストレージアカウントに関連する全ファイルの一覧を取得

Request content type: application/json
{% endswagger-description %}

{% swagger-parameter in="body" name="storageAccount" required="false" %}
ファイルのリストを取得したいストレージアカウントPublicKeyの文字列バージョン
{% endswagger-parameter %}

{% swagger-response status="200: OK" description="" %}
```json
{
    "keys": [String]
}
```
{% endswagger-response %}
{% endswagger %}

{% swagger method="post" path="" baseUrl="https://shadow-storage.genesysgo.net" summary="list-objects-and-sizes" %}
{% swagger-description %}
ストレージアカウントに関連するすべてのファイルとそのサイズの一覧を取得する

Request content type: application/json
{% endswagger-description %}

{% swagger-parameter in="body" name="storageAccount" required="true" %}
ファイルのリストを取得したいストレージアカウントPublicKeyの文字列バージョン
{% endswagger-parameter %}

{% swagger-response status="200: OK" description="" %}
```json
{
    "files": [{"file_name": String, size: Number}]
}
```
{% endswagger-response %}
{% endswagger %}

{% swagger method="post" path="" baseUrl="https://shadow-storage.genesysgo.net" summary="get-object-data" %}
{% swagger-description %}
オブジェクトの情報を取得

Request content type: application/json
{% endswagger-description %}

{% swagger-parameter in="body" name="location" required="true" %}
情報を取得したいファイルのURL
{% endswagger-parameter %}

{% swagger-response status="200: OK" description="" %}
```
Shadow Driveネットワーク内のファイルのメタデータのJSONオブジェクト、またはエラー
```
{% endswagger-response %}
{% endswagger %}

{% swagger method="post" path="" baseUrl="https://shadow-storage.genesysgo.net" summary="delete-file" %}
{% swagger-description %}
指定されたストレージアカウントからファイルを削除

Request content type: application/json

[**Example Implementation**](the-api.md#example-delete-a-file)
{% endswagger-description %}

{% swagger-parameter in="body" name="message" required="false" %}
Base58 メッセージの署名.
{% endswagger-parameter %}

{% swagger-parameter in="body" name="signer" required="false" %}
メッセージ署名の署名者であり、ストレージアカウントの所有者の公開鍵
{% endswagger-parameter %}

{% swagger-parameter in="body" name="location" required="false" %}
削除したいファイルのURL
{% endswagger-parameter %}

{% swagger-response status="200: OK" description="" %}
```json
{
    "message": String,
    "error": String or not passed if no error
}
```
{% endswagger-response %}
{% endswagger %}

{% swagger method="post" path="" baseUrl="https://shadow-storage.genesysgo.net" summary="add-storage" %}
{% swagger-description %}
ストレージを追加

Request content type: application/json
{% endswagger-description %}

{% swagger-parameter in="body" name="transaction " required="true" %}
Shadow Driveネットワークによって部分的に署名されたシリアル化されたストレージ追加トランザクション
{% endswagger-parameter %}

{% swagger-response status="200: OK" description="" %}
```json
{
    message: String,
    transaction_signature: String,
    error: String or not provided if no error
}
```
{% endswagger-response %}
{% endswagger %}

{% swagger method="post" path="" baseUrl="https://shadow-storage.genesysgo.net" summary="reduce-storage (updated)" %}
{% swagger-description %}
ストレージの削減

Request content type: application/json
{% endswagger-description %}

{% swagger-parameter in="body" name="transaction " required="true" %}
Serialized reduce storage transaction that is partially signed by the shadow drive network
{% endswagger-parameter %}

{% swagger-response status="200: OK" description="" %}
```json
{
    message: String,
    transaction_signature: String,
    error: String or not provided if no error
}
```
{% endswagger-response %}
{% endswagger %}

{% swagger method="post" path="" baseUrl="https://shadow-storage.genesysgo.net" summary="make-immutable (updated)" %}
{% swagger-description %}
ファイルを不変にする

Request content type: application/json
{% endswagger-description %}

{% swagger-parameter in="body" name="transaction" required="false" %}
Shadow Driveネットワークによって部分的に署名されたシリアル化された不変化トランザクション
{% endswagger-parameter %}

{% swagger-response status="200: OK" description="" %}
```json
{
    message: String,
    transaction_signature: String,
    error: String or not provided if no error
}
```
{% endswagger-response %}
{% endswagger %}

### **Example -** セキュアサインとアップロードファイルをAPIでシャドウドライブに変換します。

この例では、提供された API を使用して、Shadow Drive にファイルを安全にアップロードする方法を示します。ファイル名のハッシュ化、署名付きメッセージの作成、必要な情報とともにファイルを Shadow Drive のエンドポイントに送信するプロセスが含まれています。

```javascript
import bs58 from 'bs58'
import nacl from 'tweetnacl'
import crypto from 'crypto'

// `files`は、渡された各ファイルの配列です。
const allFileNames = files.map(file => file.fileName)
const hashSum = crypto.createHash("sha256")
// `allFileNames.toString()` は、すべてのファイル名をカンマで区切ったリストを作成します。
const hashedFileNames = hashSum.update(allFileNames.toString())
const fileNamesHashed = hashSum.digest("hex")
// `storageAccount` はストレージアカウントのpubkeyの文字列表現です。
let msg = `Shadow Drive Signed Message:\nStorage Account: ${storageAccount}\nUpload files with hash: ${fileNamesHashed}`;
const fd = new FormData();
// `files`は、渡された各ファイルの配列です。
for (let j = 0; j < files.length; j++) {
    fd.append("file", files[j].data, {
        contentType: files[j].contentType as string,
        filename: files[j].fileName,
    });
}
// 最終的なメッセージ文字列を出力すると、次のようになることが期待されます。
// Shadow Drive Signed Message:
// Storage Acount: ABC123
// Upload files with hash: hash1

// メッセージが上記のように正確にフォーマットされていない場合
// Shadow Drive ネットワーク側でメッセージの署名検証に失敗することになります。
const encodedMessage = new TextEncoder().encode(message);
// メッセージの署名に https://github.com/dchest/tweetnacl-js を使用します。
// 同じ方法で署名されていない場合、メッセージは Shadow Network 側で署名検証に失敗します。
// 署名の base58 バイト配列が返されます。
const signedMessage = nacl.sign.detached(encodedMessage, keypair.secretKey);
// バイト配列をbs58エンコードされた文字列に変換します。
const signature = bs58.encode(signedMessage)
fd.append("message", signature);
fd.append("signer", keypair.publicKey.toString());
fd.append("storage_account", storageAccount.toString());
fd.append("fileNames", allFileNames.toString());
const request = await fetch(`${SHDW_DRIVE_ENDPOINT}/upload`, {
    method: "POST",
    body: fd,
});
```

### **Example -** APIとメッセージ署名検証を利用したシャドウドライブ内のファイルの編集について

この例では、API とメッセージ署名検証を使用して、Shadow Drive 上のファイルを編集する方法を示します。このコードでは、必要なライブラリをインポートし、署名するメッセージを構築し、メッセージをエンコードして署名し、Shadow Drive 上のファイルを編集するための API リクエストを送信しています。

```javascript
import bs58 from 'bs58'
import nacl from 'tweetnacl'

// `storageAccount` はストレージアカウントのpubkeyの文字列表現です
// `fileName` は編集するファイルの名前です。
// `sha256Hash` は、新しいファイルの内容の sha256 ハッシュです。
const message = `Shadow Drive Signed Message:\n StorageAccount: ${storageAccount}\nFile to edit: ${fileName}\nNew file hash: ${sha256Hash}`
// 最終的なメッセージ文字列を出力すると、以下のようになることを期待します。
// Shadow Drive Signed Message:
// Storage Acount: ABC123
// File to delete: https://shadow-drive.genesysgo.net/ABC123/file.png

// メッセージが上記のように正確にフォーマットされていない場合
// Shadow drive ネットワーク側でメッセージの署名検証に失敗することになります。
const encodedMessage = new TextEncoder().encode(message);
// メッセージの署名に https://github.com/dchest/tweetnacl-js を使用します。
// 同じ方法で署名されていない場合、メッセージは Shadow Network 側で署名検証に失敗します。
// 署名の base58 バイト配列が返されます。
const signedMessage = nacl.sign.detached(encodedMessage, keypair.secretKey);
// バイト配列をbs58エンコードされた文字列に変換します。
const signature = bs58.encode(signedMessage)


const fd = new FormData();
fd.append("file", fileData, {
    contentType: fileContentType as string,
    filename: fileName,
});
fd.append("signer", keypair.publicKey.toString())
fd.append("message", signature)
fd.append("storage_account", storageAccount)

const uploadResponse = await fetch(`${SHDW_DRIVE_ENDPOINT}/edit`, {
    method: "POST",
    body: fd,
});
```

### **Example -** 署名付きメッセージとAPIを使用してシャドウドライブからファイルを削除します。

この例では、署名付きメッセージと Shadow Drive API を使用して、Shadow Drive からファイルを削除する方法を示します。このコードでは、まず、ストレージアカウントと削除するファイルの URL を含むメッセージを作成します。次に、tweetnaclライブラリを使用してメッセージをエンコードし、署名します。署名されたメッセージは、bs58エンコードされた文字列に変換されます。最後に、Shadow Drive API エンドポイントに POST リクエストを送信してファイルを削除します。

```javascript
import bs58 from 'bs58'
import nacl from 'tweetnacl'

// `storageAccount` はストレージアカウントのpubkeyの文字列表現です。
// `url` はシャドウドライブファイルへのリンクで、以前の実装ではurl入力が必要だったのと同じです。
const message = `Shadow Drive Signed Message:\nStorageAccount: ${storageAccount}\nFile to delete: ${url}`
// 最終的なメッセージ文字列を出力すると、以下のようになることを期待します。
// Shadow Drive Signed Message:
// Storage Acount: ABC123
// File to delete: https://shadow-drive.genesysgo.net/ABC123/file.png

// メッセージが上記のように正確にフォーマットされていない場合
// Shadow drive ネットワーク側でメッセージの署名検証に失敗することになります。
const encodedMessage = new TextEncoder().encode(message);
// メッセージの署名に https://github.com/dchest/tweetnacl-js を使用します。
// 同じ方法で署名されていない場合、メッセージは Shadow Network 側で署名検証に失敗します。
// 署名の base58 バイト配列が返されます。
const signedMessage = nacl.sign.detached(encodedMessage, keypair.secretKey);
// バイト配列をbs58エンコードされた文字列に変換します。
const signature = bs58.encode(signedMessage)
const deleteRequestBody = {
    signer: keypair.publicKey.toString(),
    message: signature,
    location: options.url
}
const deleteRequest = await fetch(`${SHDW_DRIVE_ENDPOINT}/delete-file`, {
    method: "POST",
    headers: {
        "Content-Type": "application/json"
    },
    body: JSON.stringify(deleteRequestBody)
})
```
