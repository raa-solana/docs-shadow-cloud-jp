# JavaScript

## **Contents**

* [**はじめに**](sdk-javascript.md#getting-started-javascript-sdk)
* [**ストレージアカウントを作成する**](sdk-javascript.md#create-a-storage-account)
* [**ファイルをアップロードする**](sdk-javascript.md#upload-a-file)
* [**ファイルを編集する**](sdk-javascript.md#edit-a-file-aka-replace-a-file)
* [**ファイルを削除する**](sdk-javascript.md#delete-a-file)
* [**メソッド**](sdk-javascript.md#methods)
  * [**constructor**](sdk-javascript.md#constructor)
  * [**addStorage**](sdk-javascript.md#addstorage)
  * [**cancelDeleteStorageAccount**](sdk-javascript.md#canceldeletestorageaccount)
  * [**claimStake**](sdk-javascript.md#claimstake)
  * [**createStorageAccount**](sdk-javascript.md#createstorageaccount)
  * [**deleteFile**](sdk-javascript.md#deletefile)
  * [**deleteStorageAccount**](sdk-javascript.md#deletestorageaccount)
  * [**editFile**](sdk-javascript.md#editfile)
  * [**getStorageAccount**](sdk-javascript.md#getstorageaccount)
  * [**getStorageAccounts**](sdk-javascript.md#getstorageaccount)
  * [**listObjects**](sdk-javascript.md#listobjects)
  * [**makeStorageImmutable**](sdk-javascript.md#makestorageimmutable) **(updated)**
  * [**migrate**](sdk-javascript.md#migrate)
  * [**redeemRent**](sdk-javascript.md#redeemrent)
  * [**reduceStorage**](sdk-javascript.md#reducestorage) **(updated)**
  * [**storageConfigPDA**](sdk-javascript.md#storageconfigpda)
  * [**refreshStake**](sdk-javascript.md#refreshstake) **(new)**
  * [**topUp**](sdk-javascript.md#topup) **(new)**
  * [**uploadFile**](sdk-javascript.md#uploadfile)
  * [**uploadMultipleFiles**](sdk-javascript.md#uploadmultiplefiles)
  * [**userInfo**](sdk-javascript.md#userinfo)
* [**例 - SDKを使用してPOSTリクエストを行う**](sdk-javascript.md#example---post-request-via-sdk-make-immutable)
### **Getting Started: Javascript SDK**

Reactアプリをscaffoldして、依存関係を追加してみましょう。

```bash
npx create-react-app shdwapp
cd shdwapp/
yarn add @shadow-drive/sdk @project-serum/anchor \
    @solana/wallet-adapter-base \
    @solana/wallet-adapter-react \
    @solana/wallet-adapter-react-ui \
    @solana/wallet-adapter-wallets \
    @solana/web3.js \
    @solana-mobile/wallet-adapter-mobile
```

[Solana Web3.js](https://solana-labs.github.io/solana-web3.js/) SDK と [Solana API](https://docs.solana.com/developing/clients/javascript-api) のリソースを確認しましょう.

#### **Instantiate the Wallet and Connection**

助けが必要な場合は、[Solana docs と 例 ](https://github.com/solana-labs/wallet-adapter)を参照してください。本ドキュメントではShadow Drive SDKに焦点を当てますので、SolanaでReactサイトを構築する方法について入門が必要な場合は、他のリソースを紹介します。

Solana example code:

<details>

<summary>Solanaが提供するサンプルコード</summary>

```javascript
import { WalletNotConnectedError } from '@solana/wallet-adapter-base';
import { useConnection, useWallet } from '@solana/wallet-adapter-react';
 import { Keypair, SystemProgram, Transaction } from '@solana/web3.js';
 import React, { FC, useCallback } from 'react';
export const SendSOLToRandomAddress: FC = () => {
const { connection } = useConnection();
const { publicKey, sendTransaction } = useWallet();
const onClick = useCallback(async () => {
    if (!publicKey) throw new WalletNotConnectedError();

    // 890880 lamports as of 2022-09-01
    const lamports = await connection.getMinimumBalanceForRentExemption(0);

    const transaction = new Transaction().add(
        SystemProgram.transfer({
            fromPubkey: publicKey,
            toPubkey: Keypair.generate().publicKey,
            lamports,
        })
    );

    const {
        context: { slot: minContextSlot },
        value: { blockhash, lastValidBlockHeight }
    } = await connection.getLatestBlockhashAndContext();

    const signature = await sendTransaction(transaction, connection, { minContextSlot });

    await connection.confirmTransaction({ blockhash, lastValidBlockHeight, signature });
}, [publicKey, sendTransaction, connection]);

return (
    <button onClick={onClick} disabled={!publicKey}>
        Send SOL to a random address!
    </button>
);
```

};

</details>

#### **Building components for various Shadow Drive operations**

まず、Shadow Drive 接続クラスオブジェクトをインスタンス化することから始めましょう。これはすべてのShadow Driveメソッドを持ち、すべてのトランザクションのためにクラス内の署名ウォレットを実装します。

最も単純なレベルでは、Reactアプリがウォレット接続時にユーザーのShadow Driveへの接続を直ちにロードしようとすることが推奨されます。これは `useEffect` という React フックを使って行うことができます。

```javascript
import React, { useEffect } from "react";
import * as anchor from "@project-serum/anchor";
import {ShdwDrive} from "@shadow-drive/sdk";
import { useWallet, useConnection } from "@solana/wallet-adapter-react";

export default function Drive() {
    const { connection } = useConnection();
    const wallet = useWallet();
    useEffect(() => {
        (async () => {
            if (wallet?.publicKey) {
                const drive = await new ShdwDrive(connection, wallet).init();
            }
        })();
    }, [wallet?.publicKey])
    return (
        <div></div>
```

This can be done with a NodeJS + TypeScript program as well.

```javascript
const anchor = require("@project-serum/anchor");
const { Connection, clusterApiUrl, Keypair } = require("@solana/web3.js");
const { ShdwDrive } = require("@shadow-drive/sdk");
const key = require("./shdwkey.json");

async function main() {
    let secretKey = Uint8Array.from(key);
    let keypair = Keypair.fromSecretKey(secretKey);
    const connection = new Connection(
        clusterApiUrl("mainnet-beta"),
        "confirmed"
    );
    const wallet = new anchor.Wallet(keypair);
    const drive = await new ShdwDrive(connection, wallet).init();
}

main();
```

#### **Create a Storage Account**

この実装は、WebとNodeの両方の実装で実質的に同じです。ストレージアカウントを作成するために必要なパラメータは3つあります：

* `name`: ストレージアカウントのフレンドリーな名前です。
* `size`： サイズ`: ストレージアカウントのサイズです。
* `version`: `v1` または `v2` のいずれかを指定します。注意 - `v1` は完全に非推奨であるため、今後は `v2` のみを使用する必要があります。

```javascript
//create account
const newAcct = await drive.createStorageAccount("myDemoBucket", "10MB", "v2");
console.log(newAcct);
```

#### **Get a list of Owned Storage Accounts**

この実装は、WebとNodeの両方の実装で実質的に同じです。必要なパラメータは、前のステップで作成したストレージアカウントのバージョンを表す `v1` または `v2` のいずれかだけです。

```javascript
const accts = await drive.getStorageAccounts("v2");
// handle printing pubKey of first storage acct
let acctPubKey = new anchor.web3.PublicKey(accts[0].publicKey);
console.log(acctPubKey.toBase58());
```

Full Response:

<details>

<summary>Output</summary>

```javascript
[
  {
    publicKey: PublicKey {
      _bn: <BN: c9217d175c7257f52a64cb9faa203e65487343720441cef2d9abcb3f8c706053>
    },
    account: {
      immutable: false,
      toBeDeleted: false,
      deleteRequestEpoch: 0,
      storage: <BN: a00000>,
      owner1: [PublicKey],
      accountCounterSeed: 0,
      creationTime: 1665690481,
      creationEpoch: 359,
      lastFeeEpoch: 359,
      identifier: 'myDemoBucket'
    }
  },
  {
    publicKey: PublicKey {
      _bn: <BN: 502aa038c1b95a8bc36c301f3ae06beda929ec3a044b0543e70d4378d4c574ea>
    },
    account: {
      immutable: false,
      toBeDeleted: false,
      deleteRequestEpoch: 0,
      storage: <BN: a00000>,
      owner1: [PublicKey],
      accountCounterSeed: 1,
      creationTime: 1665690563,
      creationEpoch: 359,
      lastFeeEpoch: 359,
      identifier: 'myDemoBucket'
    }
  }
]
```

</details>

#### **Get a Specific Storage Account**

この実装は、WebとNodeの両方の実装で実質的に同じです。必要なパラメータは、PublicKeyオブジェクトまたは公開鍵のbase-58文字列のみです。

```javascript
const acct = await drive.getStorageAccount(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);
console.log(acct);
```

Full Response:

<details>

<summary>Output</summary>

```javascript
{
  storage_account: 'EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN',
  reserved_bytes: 10485760,
  current_usage: 0,
  immutable: false,
  to_be_deleted: false,
  delete_request_epoch: 0,
  owner1: 'GXE2RHkdivb7pdBykiAr9mpKy1kHF1Qe7oZqnX4NwkoD',
  account_counter_seed: 0,
  creation_time: 1665690481,
  creation_epoch: 359,
  last_fee_epoch: 359,
  identifier: 'myDemoBucket',
  version: 'V2'
}
```

</details>

#### **Upload a File**

`uploadFile`メソッドは、2つのパラメータを必要とします：

* `key`: シャドウストレージアカウントの公開鍵を表す PublicKey オブジェクト。
* `data`: `File` オブジェクトタイプまたは `ShadowFile` オブジェクトタイプのファイル。

メソッドにカーソルを合わせると、以下のようなインテリセンスのポップアップが表示されるので確認してください。

<figure><img src="../../.gitbook/assets/Screen Shot 2022-10-13 at 7.40.37 PM.png" alt=""><figcaption></figcaption></figure>

`File`オブジェクトはWebブラウザで実装されており、`ShadowFile`はTypeScriptで実装したカスタムタイプです。つまり、Webで`File`を使うか、TSでスクリプトを書くかのどちらかです。

以下はReact Componentを使った例です:

<details>

<summary>React アップロードコンポーネント</summary>

```javascript
import React, { useState } from "react";
import { ShdwDrive } from "@shadow-drive/sdk";
import { useWallet, useConnection } from "@solana/wallet-adapter-react";
import FormData from "form-data";

export default function Upload() {
    const [file, setFile] = useState < File > undefined;
    const [uploadUrl, setUploadUrl] = useState < String > undefined;
    const [txnSig, setTxnSig] = useState < String > undefined;
    const { connection } = useConnection();
    const wallet = useWallet();
    return (
        <div>
            <form
                onSubmit={async (event) => {
                    event.preventDefault();
                    const drive = await new ShdwDrive(
                        connection,
                        wallet
                    ).init();
                    const accounts = await drive.getStorageAccounts();
                    const acc = accounts[0].publicKey;
                    const getStorageAccount = await drive.getStorageAccount(
                        acc
                    );

                    const upload = await drive.uploadFile(acc, file);
                    console.log(upload);
                    setUploadUrl(upload.finalized_location);
                    setTxnSig(upload.transaction_signature);
                }}
            >
                <h1>Shadow Drive File Upload</h1>
                <input
                    type="file"
                    onChange={(e) => setFile(e.target.files[0])}
                />
                <br />
                <button type="submit">Upload</button>
            </form>
            <span>You may have to wait 60-120s for the URL to appear</span>
            <div>
                {uploadUrl ? (
                    <div>
                        <h3>Success!</h3>
                        <h4>URL: {uploadUrl}</h4>
                        <h4>Sig: {txnSig}</h4>
                    </div>
                ) : (
                    <div></div>
                )}
            </div>
        </div>
    );
}
```

</details>

そして、NodeJS + TypeScriptの実装は、以下のようになります

<details>

<summary>TS UploadFile</summary>

```javascript
const { ShdwDrive, ShadowFile } = require("@shadow-drive/sdk");
/**
 * ...
 * ここでストレージアカウントを初期化します...
 * ...
 */
const fileBuff = fs.readFileSync("./mytext.txt");
const acctPubKey = new anchor.web3.PublicKey(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);
const fileToUpload: ShadowFile = {
    name: "mytext.txt",
    file: fileBuff,
};
const uploadFile = await drive.uploadFile(acctPubKey, fileToUpload);
console.log(uploadFile);
```

</details>

#### **Upload Multiple Files**

これは uploadFile とほぼ同じ実装ですが、`FileList` または `ShadowFiles` の配列とオプションの同時実行パラメータを必要とします。

<figure><img src="../../.gitbook/assets/Screen Shot 2022-10-13 at 8.12.08 PM.png" alt=""><figcaption></figcaption></figure>

デフォルトでは、3つのファイルを同時にアップロードしようとする設定になっていることを思い出してください。ここでは、これを上書きして、インフラのコアと帯域幅に基づいてアップロードを試みるファイル数を指定することができます。

#### **Delete a File**

`deleteFile`の実装は、WebとNodeで同じです。ファイルを削除するために必要なパラメータは3つあります：

* `key`: ストレージアカウントの公開鍵
* `url`: 削除するファイルの現在の URL。
* `version`: `v1` または `v2` のいずれかを指定します。

```javascript
const url =
    "https://shdw-drive.genesysgo.net/4HUkENqjnTAZaUR4QLwff1BvQPCiYkNmu5PPSKGoKf9G/fape.png";
const acctPubKey = new anchor.web3.PublicKey(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);
const delFile = await drive.deleteFile(acctPubKey, url, "v2");
console.log(delFile);
```

#### **Edit a File (aka Replace a file)**

<figure><img src="../../.gitbook/assets/Screen Shot 2022-10-13 at 8.28.30 PM.png" alt=""><figcaption></figcaption></figure>

editFileメソッドは、`uploadFile`と`deleteFile`のコンボです。パラメータを見てみましょう：

* `key`: ストレージアカウントの公開鍵
* `url`: 置換されるファイルの URL です。
* `data`: 現在のファイルを置き換えるためのファイルです。**ファイル名と拡張子が全く同じで、`File` または `ShadowFile` オブジェクトである必要があります**
* `version`: `v1` または `v2` のいずれかを指定します。

```typescript
const fileToUpload: ShadowFile = {
    name: "mytext.txt",
    file: fileBuff,
};
const url =
    "https://shdw-drive.genesysgo.net/4HUkENqjnTAZaUR4QLwff1BvQPCiYkNmu5PPSKGoKf9G/fape.png";
const acctPubKey = new anchor.web3.PublicKey(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);
const editFile = await drive.editFile(acctPubKey, url, "v2", fileToUpload);
```

#### **List Storage Account Files (aka List Objects)**

ストレージアカウントのファイル名を取得するために公開鍵が必要なだけのシンプルな実装です。

```javascript
const acctPubKey = new anchor.web3.PublicKey(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);
const listItems = await drive.listObjects(acctPubKey);
console.log(listItems);
```

And the response payload:

```
{ keys: [ 'index.html' ] }
```

#### **Increase Storage Account Size**

これは、ストレージアカウントのストレージの上限を単純に増加させるメソッドです。3つのパラメータを必要とします

* `key`: ストレージアカウント公開鍵
* `size`: 増加する量。末尾に `KB`、`MB`、`GB` が必要です。
* `version`: ストレージアカウントのバージョンで、`v1` または `v2` のいずれかを指定します。

```javascript
const accts = await drive.getStorageAccounts("v2");
let acctPubKey = new anchor.web3.PublicKey(accts[1].publicKey);
const addStgResp = await drive.addStorage(acctPubKey, "10MB", "v2");
```

#### **Reduce Storage Account Size**

これは、ストレージアカウントのストレージの上限を減少させるメソッドです。この実装では、ストレージアカウントのキー、減らす量、バージョンという3つのパラメタのみを必要とします。

```javascript
const acctPubKey = new anchor.web3.PublicKey(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);
const shrinkAcct = await drive.reduceStorage(acctPubKey, "10MB", "v2");
```

#### **Next you'll want to claim your unused SHDW**

この方法では、使用されなくなったSHDWをリクレームすることができます。この方法では、ストレージアカウントの公開鍵とバージョンだけが必要です。

```javascript
const acctPubKey = new anchor.web3.PublicKey(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);
const claimStake = await drive.claimStake(acctPubKey, "v2");
```

#### **Delete a Storage Account**

その名の通り、ストレージアカウントとそのファイルをすべて削除することができます。ストレージアカウントは、現在のエポックが終了するまではまだ復元可能ですが、それ以降は削除されます。この実装では、ストレージアカウントのキーとバージョンという2つのパラメータしか必要としません。

```javascript
const acctPubKey = new anchor.web3.PublicKey(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);
const delAcct = await drive.deleteStorageAccount(acctPubKey, "v2");
```

#### **Undelete the Deleted Storage Account**

現在のエポックが経過していない場合でも、ストレージアカウントを取り戻すことができます。この実装では、アカウントの公開鍵とバージョンという2つのパラメータだけを必要とします。

```javascript
const acctPubKey = new anchor.web3.PublicKey(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);
const cancelDelStg = await drive.cancelDeleteStorageAccount(acctPubKey, "v2");
```

## **Methods**

### **`constructor`**

#### **Definition**

このメソッドは、ShadowDrive クラスの新しいインスタンスを作成するために使用されます。web3 接続オブジェクトと web3 ウォレットを受け取ります。これは、ShadowDrive クラスのインスタンスを返します。

#### **Parameters**

* `connection`: `Connection` - initialized web3 connection object
* `wallet`: `any` - Web3 wallet

#### **Returns**

ShadowDriveクラスのインスタンスを返します。

{% tabs %}
{% tab title="Example" %}
```javascript
const shadowDrive = new ShadowDrive(connection, wallet).init();
```
{% endtab %}

{% tab title="Explanations" %}
{% code overflow="wrap" %}
```javascript
// Javascript SDK の例では、コンストラクタ・メソッドを使用して
// ShadowDrive クラスの新しいインスタンスを作成し、与えられた接続とウォレットのパラメータでそれを初期化します。
const shadowDrive = new ShadowDrive(connection, wallet).init();
```
{% endcode %}
{% endtab %}
{% endtabs %}

### **`addStorage`**

#### **Definition**

`addStorage` は `index.ts` の 121 行目に定義された `ShadowDrive` クラスのメソッドです。このメソッドは3つのパラメータを受け取ります: `key`, `size`, `version` の 3 つのパラメータを受け取り、確認されたトランザクション ID を含む `Promise<ShadowDriveResponse>` を返します。

#### **Parameters**

* `key`: `PublicKey` - サイズを増加させる既存のストレージの公開鍵。
* `size`: `string` - ストレージアカウントへの追加を要求するストレージの量。1KB'、'1MB'、'1GB'のような文字列である必要があります。現在、KB、MB、GB のストレージ区切りのみがサポートされています。
* `version`: `ShadowDriveVersion` - ShadowDrive のバージョン（v1 または v2）。

#### **Returns**

確認されたトランザクションID

```json
{
  message: string;
  transaction_signature?: string
}
```

{% tabs %}
{% tab title="Example" %}
```javascript
const accts = await drive.getStorageAccounts("v2");
let acctPubKey = new anchor.web3.PublicKey(accts[1].publicKey);
const addStgResp = await drive.addStorage(acctPubKey, "10MB", "v2");
```
{% endtab %}

{% tab title="Explanations" %}
{% code overflow="wrap" %}
```javascript
// addStorage メソッドを使用した Javascript SDK の例
// この行は、`drive` オブジェクトの `getStorageAccounts` メソッドを使用してバージョン "v2" のストレージアカウントを取得し、`accts` 変数に格納します。
const accts = await drive.getStorageAccounts("v2")

// この行では、前の行で取得した2番目のストレージアカウントの公開鍵を使用して新しい `PublicKey` オブジェクトを作成し、変数 `acctPubKey` に格納します。
let acctPubKey = new anchor.web3.PublicKey(accts[1].publicKey)

// この行は、サイズ "10MB"、バージョン "v2" の新しいストレージ割り当てを `acctPubKey` の公開鍵によって識別されるストレージアカウントに追加します。レスポンスは `addStgResp` 変数に格納されます。
const addStgResp = await drive.addStorage(acctPubKey,"10MB","v2"ca
```
{% endcode %}
{% endtab %}
{% endtabs %}

### **`cancelDeleteStorageAccount`**

#### **Definition**

index.ts:135 で定義されている ShadowDrive.cancelDeleteStorageAccount の実装。このメソッドは、Shadow Drive 上のストレージアカウントの削除要求をキャンセルするために使用されます。ストレージアカウントの公開鍵とシャドウドライブのバージョン (v1 または v2) を受け取ります。このメソッドは、削除解除要求の確認済みトランザクション ID を含む Promise<{ txid: string }> を返します。

#### **Parameters**

* `key`: `PublicKey` - Publickey

#### **Returns**

確認されたトランザクションID

{% tabs %}
{% tab title="Example" %}
```javascript
const acctPubKey = new anchor.web3.PublicKey(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);
const cancelDelStg = await drive.cancelDeleteStorageAccount(acctPubKey, "v2");
```
{% endtab %}

{% tab title="Explanations" %}
{% code overflow="wrap" %}
```javascript
// cancelDeleteStorageAccount メソッドを使用した Javascript SDK の例。
// Solanaアカウントの公開鍵の文字列表現から、新しい公開鍵オブジェクトを作成します。
const acctPubKey = new anchor.web3.PublicKey(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);

// アカウント公開鍵オブジェクトと、削除を取り消すストレージアカウントのバージョンを示す文字列を渡して、Shadow Drive API の "cancelDeleteStorageAccount" 関数を呼び出します。
const cancelDelStg = await drive.cancelDeleteStorageAccount(acctPubKey, "v2");
```
{% endcode %}
{% endtab %}
{% endtabs %}

### **`claimStake`**

#### **Definition**

このメソッドは、ShadowDrive 上の Stake を要求するために使用されます。ストレージアカウントの PublicKey と ShadowDrive のバージョン (v1 または v2) を受け取ります。このメソッドは、claimStake 要求の確認済みトランザクション ID を含む Promise<{ txid: string }> を返します。

#### **Parameters**

* `key`: `PublicKey` - ストレージアカウントの公開鍵
* `バージョン`: \`Shadow Drive

#### **Returns**

確認されたトランザクションID

{% tabs %}
{% tab title="Example" %}
```javascript
const acctPubKey = new anchor.web3.PublicKey(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);
const claimStake = await drive.claimStake(acctPubKey, "v2");
```
{% endtab %}

{% tab title="Explanations" %}
{% code overflow="wrap" %}
```javascript
// claimStakeメソッドを使用したJavascript SDKの例
// 指定された値を持つ新しい公開鍵オブジェクトを作成します。
const acctPubKey = new anchor.web3.PublicKey(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);

// `drive`オブジェクトに対して、アカウント公開鍵と`v2`をパラメータとして`claimStake`関数を呼び出し、その完了を待って処理を進めます。
const claimStake = await drive.claimStake(acctPubKey, "v2");
```
{% endcode %}
{% endtab %}
{% endtabs %}

### **`createStorageAccount`**

#### **Definition**

index.ts:120 で定義されている ShadowDrive.createStorageAccount の実装 このメソッドは、Shadow Drive で新しいストレージアカウントを作成するために使用されます。このメソッドは、ストレージアカウントの名前、要求されたストレージアカウントのサイズ、および ShadowDrive のバージョン (v1 または v2) を受け付けます。また、ストレージアカウントのオプションのセカンダリーオーナーを受け付けます。作成されたストレージアカウントとトランザクション署名を含む Promise を返します。

#### **Parameters**

* `name`: `string` - ストレージアカウントに付けたい名前です。(一意である必要はありません)
* `size`: `string` - 作成を要求しているストレージの量。1KB', '1MB', '1GB'のような文字列であるべきです。現在、KB、MB、GB のストレージ区切りのみがサポートされています。
* `version`: `ShadowDriveVersion` - ShadowDrive のバージョン（v1 または v2）。
* `owner2` (オプション): `PublicKey` - オプションで、ストレージアカウントのセカンダリーオーナーを指定します。

#### **Returns**

```json
{
    "shdw_bucket": String,
    "transaction_signature": String
}
```

{% tabs %}
{% tab title="Example" %}
```javascript
// アカウント作成
const newAcct = await drive.createStorageAccount("myDemoBucket", "10MB", "v2");
console.log(newAcct);
```
{% endtab %}

{% tab title="Explanations" %}
{% code overflow="wrap" %}
```javascript
// createStorageAccount メソッドを使用したJavascript SDKの例
// `drive`オブジェクトに対して "myDemoBucket","10MB","v2"をパラメータとして`createStorageAccount`関数を呼び出し、その完了を待って処理を進めます。関数呼び出しの結果は、'newAcct' に入ります。
const newAcct = await drive.createStorageAccount("myDemoBucket", "10MB", "v2");

// 変数 newAcct の値をコンソールにログ出力します
console.log(newAcct);
```
{% endcode %}
{% endtab %}
{% endtabs %}

### **`deleteFile`**

#### **Definition**

このメソッドは、Shadow Drive 上のファイルを削除するために使用されます。ストレージアカウントの公開鍵、削除を要求するファイルの Shadow Drive URL、および Shadow Drive のバージョン (v1 または v2) を受け付けます。削除要求の確認済みトランザクション ID を含む Promise を返します。

#### **Parameters**

* `key`: `PublicKey` - ストレージアカウントの公開鍵
* `url`: `string` - 削除を要求しているファイルのシャドウドライブの URL
* `version`： \`Shadow Drive
* Version\` - Shadow Drive のバージョン（v1 または v2）．

#### **Returns**

```json
{
    "message": String,
    "error": String or not passed if no error
}
```

{% tabs %}
{% tab title="Example" %}
```javascript
const url =
    "https://shdw-drive.genesysgo.net/4HUkENqjnTAZaUR4QLwff1BvQPCiYkNmu5PPSKGoKf9G/fape.png";
const acctPubKey = new anchor.web3.PublicKey(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);
const delFile = await drive.deleteFile(acctPubKey, url, "v2");
console.log(delFile);
```
{% endtab %}

{% tab title="Explanations" %}
{% code overflow="wrap" %}
```javascript
// deleteFile メソッドを使用したJavascript SDKの例
// 変数`url`に削除するファイルのURLを含む文字列値を入れます
const url =
    "https://shdw-drive.genesysgo.net/4HUkENqjnTAZaUR4QLwff1BvQPCiYkNmu5PPSKGoKf9G/fape.png";

// 特定の値を持つ新しい公開鍵オブジェクトを作成し、変数 'acctPubKey' に入れます
const acctPubKey = new anchor.web3.PublicKey(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);

// `drive`オブジェクトに対して、アカウント公開鍵、URL、"v2"をパラメータとして`deleteFile`関数を呼び出し、完了を待って処理を進めます。関数呼び出しの結果は、変数 `delFile`に入ります。
const delFile = await drive.deleteFile(acctPubKey, url, "v2");

// 変数 delFile の値をコンソールにログ出力します
console.log(delFile);
```
{% endcode %}
{% endtab %}
{% endtabs %}

### **`deleteStorageAccount`**

#### **Definition**

index.ts:124 で定義されている ShadowDrive.deleteStorageAccount の実装 このメソッドは、Shadow Drive 上のストレージアカウントを削除するために使用されます。ストレージアカウントの公開鍵とShadow Driveのバージョン（v1 または v2）を受け取ります。このメソッドは、削除要求の確認済みトランザクション ID を含む Promise<{ txid: string }> を返します。

#### **Parameters**

* `key`: `PublicKey` - ストレージアカウントの公開鍵。
* `version`: `ShadowDriveVersion` - Shadow Drive のバージョン（v1 または v2）.

#### **Returns**

確認されたトランザクションID

{% tabs %}
{% tab title="Example" %}
```javascript
const acctPubKey = new anchor.web3.PublicKey(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);
const delAcct = await drive.deleteStorageAccount(acctPubKey, "v2");
```
{% endtab %}

{% tab title="Explanations" %}
{% code overflow="wrap" %}
```javascript
// deleteStorageAccount メソッドを使用したJavascript SDKの例
// 特定の値を持つ新しい公開鍵オブジェクトを作成し、変数 'acctPubKey' に入れます。
const acctPubKey = new anchor.web3.PublicKey(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);

// `drive`オブジェクトに対して、アカウント公開鍵と"v2"をパラメータとして`deleteStorageAccount`関数を呼び出し、完了を待って処理を進めます。関数呼び出しの結果は、変数`delAcct`に入ります。
const delAcct = await drive.deleteStorageAccount(acctPubKey, "v2");
```
{% endcode %}
{% endtab %}
{% endtabs %}

### **`editFile`**

#### **Definition**

このメソッドは、Shadow Drive 上のファイルを編集するために使用されます。ストレージアカウントの公開鍵、既存のファイルの URL、File または ShadowFile オブジェクト、および ShadowDrive のバージョン (v1 または v2) を受け入れます。ファイルの場所とトランザクション署名を含む Promise を返します。

#### **Parameters**

* `key`: `PublicKey` - ストレージアカウントの公開鍵
* `url`: `string` - 既存のファイルの URL
* `data`: `File | ShadowFile` - ファイルまたは ShadowFile オブジェクト、ファイル拡張子は ShadowFiles の名前プロパティに含める必要があります。
* `version`: `ShadowDriveVersion` - ShadowDrive のバージョン（v1 または v2）。

#### **Returns**

```json
{
  finalized_location: string;
}
```

{% tabs %}
{% tab title="Example" %}
```javascript
const fileToUpload: ShadowFile = {
    name: "mytext.txt",
    file: fileBuff,
};
const url =
    "https://shdw-drive.genesysgo.net/4HUkENqjnTAZaUR4QLwff1BvQPCiYkNmu5PPSKGoKf9G/fape.png";
const acctPubKey = new anchor.web3.PublicKey(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);
const editFile = await drive.editFile(acctPubKey, url, "v2", fileToUpload);
```
{% endtab %}

{% tab title="Explanations" %}
{% code overflow="wrap" %}
```javascript
// editFile メソッドを使用したJavascript SDKの例
// アップロードするファイルの名前と内容を含むオブジェクトを作成し、変数`fileToUpload`に入れます。
const fileToUpload: ShadowFile = {
    name: "mytext.txt",
    file: fileBuff,
};

// 編集するファイルの URL を含む文字列値を変数`url`に入れます。
const url =
    "https://shdw-drive.genesysgo.net/4HUkENqjnTAZaUR4QLwff1BvQPCiYkNmu5PPSKGoKf9G/fape.png";

// 特定の値を持つ新しい公開鍵オブジェクトを作成し、変数 `acctPubKey`に入れます。
const acctPubKey = new anchor.web3.PublicKey(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);

// アカウント公開鍵，URL，"v2"，ファイルオブジェクト をパラメータとして`drive`オブジェクトの`editFile`関数を呼び出し，その完了を待って処理を進めます。関数呼び出しの結果は変数`editFile`に入ります。
const editFile = await drive.editFile(acctPubKey, url, "v2", fileToUpload);
```
{% endcode %}
{% endtab %}
{% endtabs %}

### **`getStorageAccount`**

#### **Definition**

このメソッドは、Shadow Drive 上のストレージアカウントの詳細を取得するために使用されます。このメソッドはストレージアカウントの公開鍵を受け取り、ストレージアカウントの詳細を含む Promise を返します。

#### **Parameters**

* `key`: `PublicKey` - ストレージアカウントの公開鍵

#### **Returns**

```json
{
  storage_account: PublicKey;
  reserved_bytes: number;
  current_usage: number;
  immutable: boolean;
  to_be_deleted: boolean;
  delete_request_epoch: number;
  owner1: PublicKey;
  account_counter_seed: number;
  creation_time: number;
  creation_epoch: number;
  last_fee_epoch: number;
  identifier: string;
  version: `${Uppercase<ShadowDriveVersion>}`;
}
```

{% tabs %}
{% tab title="Example" %}
```javascript
const acct = await drive.getStorageAccount(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);
console.log(acct);
```
{% endtab %}

{% tab title="Explanations" %}
{% code overflow="wrap" %}
```javascript
// getStorageAccount メソッドを使用したJavascript SDKの例
// `drive`オブジェクトに対して、アカウントの公開鍵をパラメータとして`getStorageAccount`関数を呼び出し、その完了を待って処理を進めます。関数呼び出しの結果は、変数`acct`に入ります。
const acct = await drive.getStorageAccount(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);

// 結果のオブジェクトをコンソールにログ出力します。
console.log(acct);
```
{% endcode %}
{% endtab %}
{% endtabs %}

### **`getStorageAccounts`**

#### **Definition**

このメソッドは、現在のユーザーに関連するすべてのストレージアカウントのリストを取得するために使用されます。Shadow Drive のバージョン (v1 または v2) を受け付けます。ストレージアカウントのリストを含む Promise\<StorageAccountResponse\[]> を返します。

#### **Parameters**

* `version`: `ShadowDriveVersion` - ShadowDrive のバージョン（v1 または v2）

#### **Returns**

```json
{
  publicKey: anchor.web3.PublicKey;
  account: StorageAccount;
}
```

{% tabs %}
{% tab title="Example" %}
```javascript
ShadowDrive.getStorageAccounts(shadowDriveVersion)
    .then((storageAccounts) =>
        console.log(`List of storage accounts: ${storageAccounts}`)
    )
    .catch((err) => console.log(`Error getting storage accounts: ${err}`));
```
{% endtab %}

{% tab title="Explanations" %}
{% code overflow="wrap" %}
```javascript
// getStorageAccounts メソッドを使用したJavascript SDKの例
// `drive`オブジェクトの`getStorageAccounts`関数をバージョンパラメーター "v2" で呼び出し、その完了を待って処理を進めます。関数呼び出しの結果は、変数`accts`に入ります。
const accts = await drive.getStorageAccounts("v2");

// `let`キーワードで変数`acctPubKey`を宣言し`accts`配列の最初のオブジェクトのpublicKeyの値を代入しています。この値は、Base58形式の文字列に変換されます。
let acctPubKey = new anchor.web3.PublicKey(accts[0].publicKey);
console.log(acctPubKey.toBase58());
```
{% endcode %}
{% endtab %}
{% endtabs %}

### **`listObjects`**

#### **Definition**

このメソッドは、Shadow Drive 上のストレージアカウントにあるオブジェクトをリストアップするために使用されます。ストレージアカウントの公開鍵を受け取り、ストレージアカウント内のオブジェクトのリストを含む Promise を返します。

#### **Parameters**

* `storageAccount`: `PublicKey`

#### **Returns**

```json
{
  keys: string[];
}
```

{% tabs %}
{% tab title="Example" %}
```javascript
const acctPubKey = new anchor.web3.PublicKey(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);
const listItems = await drive.listObjects(acctPubKey);
console.log(listItems);
```
{% endtab %}

{% tab title="Explanations" %}
{% code overflow="wrap" %}
```javascript
// listObjects メソッドを使用したJavascript SDKの例
// 特定の公開鍵文字列を使用して新しい`PublicKey`オブジェクトを作成し、それを`acctPubKey`変数に代入します。
const acctPubKey = new anchor.web3.PublicKey(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);

// 変数`acctPubKey`をパラメータとして`drive`オブジェクトの `listObjects`関数を呼び出し、その完了を待って処理を進めます。関数呼び出しの結果は`listItems`変数に入ります。
const listItems = await drive.listObjects(acctPubKey);

// 結果のオブジェクトをコンソールにログ出力します。
console.log(listItems);
```
{% endcode %}
{% endtab %}
{% endtabs %}

### **`makeStorageImmutable`**

#### **Definition**

このメソッドは、ShadowDrive 上でストレージアカウントを不変にするために使用されます。ストレージアカウントの公開鍵とシャドウドライブのバージョン（v1 または v2）を受け取ります。makeStorageImmutable 要求の確認済みトランザクション ID を含む Promise を返します。

#### **Parameters**

* `key`: `PublicKey` - ストレージアカウントの公開鍵
* `version`: `ShadowDriveVersion` - Shadow Drive のバージョン（v1 または v2）

#### **Returns**

```json
{
  message: string;
  transaction_signature?: string;
}
```

{% tabs %}
{% tab title="Example" %}
```javascript
const key = new anchor.web3.PublicKey(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);
const result = await drive.makeStorageImmutable(key, "v2");
console.log(result);
```
{% endtab %}

{% tab title="Explanations" %}
{% code overflow="wrap" %}
```javascript
// makeStorageImmutable メソッドを使用したJavascript SDKの例
// 公開鍵文字列を使用して、新しいPublicKeyオブジェクトを作成します。
const key = new anchor.web3.PublicKey(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);

// PublicKeyオブジェクトとバージョン文字列を指定して`makeStorageImmutable`関数を呼び出し、完了を待ちます。
const result = await drive.makeStorageImmutable(key, "v2");

// 結果のオブジェクトをコンソールにログ出力します。
console.log(result);
```
{% endcode %}
{% endtab %}
{% endtabs %}

### **`migrate`**

#### **Definition**

このメソッドは、Shadow Drive 上の Storage Account を移行するために使用します。ストレージアカウントの PublicKey を受け取ります。移行要求の確認済みトランザクション ID を含む Promise<{ txid: string }> を返します。

#### **Parameters**

* `key`: `PublicKey` - ストレージアカウントの公開鍵

#### **Returns**

確認されたトランザクションID

{% tabs %}
{% tab title="Example" %}
```javascript
const result = await drive.migrate(key);
```
{% endtab %}

{% tab title="Explanations" %}
{% code overflow="wrap" %}
```javascript
// migrate メソッドを使用したJavascript SDKの例
// `drive`オブジェクトの`migrate`関数を呼び出し、PublicKeyオブジェクトをパラメータとして渡します。
const result = await drive.migrate(key);
```
{% endcode %}
{% endtab %}
{% endtabs %}

### **`redeemRent`**

#### **Definition**

このメソッドは、Shadow Drive 上の Rent を償還するために使用されます。ストレージアカウントの公開鍵と、クローズするファイルアカウントの公開鍵を受け取ります。redeemRent リクエストの確認済みトランザクション ID を含む Promise<{ txid: string }> を返します。

#### **Parameters**

* `key`: `PublicKey` - ストレージアカウントの公開鍵
* `fileAccount`: `PublicKey` - 閉じようとするファイルアカウントの公開鍵

#### **Returns**

確認されたトランザクションID

{% tabs %}
{% tab title="Example" %}
```javascript
const fileAccount = new anchor.web3.PublicKey(
    "3p6U9s1sGLpnpkMMwW8o4hr4RhQaQFV7MkyLuW8ycvG9"
);
const result = await drive.redeemRent(key, fileAccount);
```
{% endtab %}

{% tab title="Explanations" %}
{% code overflow="wrap" %}
```javascript
// redeemRent メソッドを使用したJavascript SDKの例
// ファイルアカウントの公開鍵文字列を使用して、新しいPublicKeyオブジェクトを作成します。
const fileAccount = new anchor.web3.PublicKey(
    "3p6U9s1sGLpnpkMMwW8o4hr4RhQaQFV7MkyLuW8ycvG9"
);

// `drive`オブジェクトに対して`redeemRent`関数を呼び出し、パラメータとして両方のPublicKeyオブジェクトを渡します。
const result = await drive.redeemRent(key, fileAccount);
```
{% endcode %}
{% endtab %}
{% endtabs %}

### **`reduceStorage`**

#### **Definition**

このメソッドは、Shadow Drive 上のストレージアカウントのストレージを削減するために使用されます。ストレージアカウントの公開鍵、ストレージアカウントから削減を要求するストレージの量、およびシャドウドライブのバージョン（v1 または v2）を受け付けます。ストレージ削減要求の確認済みトランザクション ID を含む Promise を返します。

#### **Parameters**

* `key`: `PublicKey` - ストレージアカウントの公開鍵
* `size`: `string` - ストレージアカウントから削減することを要求するストレージの量。1KB'、'1MB'、'1GB'のような文字列である必要があります。現在、KB、MB、GB のストレージ区切りのみがサポートされています。
* `version`: `ShadowDriveVersion` - ShadowDrive のバージョン（v1 または v2）

#### **Returns**

```json
{
  message: string;
  transaction_signature?: string;
}
```

{% tabs %}
{% tab title="Example" %}
```javascript
const acctPubKey = new anchor.web3.PublicKey(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);
const shrinkAcct = await drive.reduceStorage(acctPubKey, "10MB", "v2");
```
{% endtab %}

{% tab title="Explanations" %}
{% code overflow="wrap" %}
```javascript
// reduceStorage メソッドを使用したJavascript SDKの例
// 与えられた文字列で、新しい公開鍵オブジェクトを作成します。
const acctPubKey = new anchor.web3.PublicKey(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);

// 指定された公開鍵のストレージアカウントのストレージサイズを
// 指定されたバージョンで10MBに縮小する。
const shrinkAcct = await drive.reduceStorage(acctPubKey, "10MB", "v2");
```
{% endcode %}
{% endtab %}
{% endtabs %}

### **`storageConfigPDA`**

#### **Definition**

これは、開発者がアカウントに保存されているデータを表示/使用する必要がある場合に、PDAアカウントを公開するものです。

#### **Parameters**

* `key`: `PublicKey` - ストレージアカウントの公開鍵
* `data`: `File | ShadowFile` - ファイルまたは ShadowFile オブジェクト、ファイル拡張子は ShadowFiles の name プロパティに含める必要があります。

#### **Returns**

Public Key

{% tabs %}
{% tab title="Example" %}
```javascript
storageConfigPDA: PublicKey;
```
{% endtab %}

{% tab title="Explanations" %}
{% code overflow="wrap" %}
```javascript
// storageConfigPDA は、Shadow SDK のメソッドで、Shadow ストレージプログラムのコンフィグのプログラム派生アカウント (PDA) の公開鍵を返します。プログラム派生アカウントとは、プログラムの公開鍵と特定のシードから派生するsolanaブロックチェーン上の特別なアカウントです。この方法の目的は、ShadowストレージプログラムのconfigのPDAを取得するための便利な方法を提供することです。この config には、現在のストレージrent免除の閾値やストレージアカウントのデータサイズ制限など、重要な情報が含まれています。この公開鍵は、シャドウストレージプログラムのconfigアカウントと対話するために使用され、ユーザーはプログラムのグローバル構成設定を取得および変更することができます。
storageConfigPDA: PublicKey;
```
{% endcode %}
{% endtab %}
{% endtabs %}

### `refreshStake`

#### Definition

このメソッドは、ストレージアカウントのステーキング金額を更新するために使用されます。ステージアカウントを正しく更新するためには、[\`topUp\`](sdk-javascript.md#topup)メソッドを呼び出した後に、このメソッドを呼び出すことが必要です。

#### Parameters

* `key`: `PublicKey` - ストレージアカウントの公開鍵
* `version`: `v1` か `v2` のどちらか. 注意 - `v1` は完全に非推奨であるため、今後は `v2` のみを使用する必要があります。

#### Returns

```json
{
    txid: string
}
```

### `topUp`

#### Definition

This method is used to top up a storage account's $SHDW balance to cover any necessary fees, like mutable storage fees which are collected every epoch. It is necessary to call the \`[refreshStake](sdk-javascript.md#refreshstake)\` method after this.
このメソッドは、ストレージアカウントの $SHDW 残高を、エポックごとに徴収される可変ストレージ料金などの必要な料金に充当するために使用されます。この後 \`[refreshStake](sdk-javascript.md#refreshstake)\`メソッドを呼び出す必要があります。

#### Parameters

* `key`: `PublicKey` - ストレージアカウントの公開鍵
* `amount`: `Number` - ステークアカウントに送金する$SHDWの金額

#### Returns

```json
{
    txid: string;
}
```

### **`uploadFile`**

#### **Definition**

このメソッドは、Shadow Drive にファイルをアップロードするために使用されます。ストレージアカウントの公開鍵、およびファイルまたは ShadowFile オブジェクトを受け取ります。ファイル拡張子は、ShadowFiles の name プロパティに含める必要があります。このメソッドは、ファイルの場所とトランザクション署名を含む Promise を返します。

#### **Parameters**

* `key`: `PublicKey` - ストレージアカウントの公開鍵
* `data`: `File | ShadowFile` - ファイルまたは ShadowFile オブジェクト。ファイル拡張子は ShadowFiles の name プロパティに含まれる必要があります。

#### **Returns**

```json
{
  finalized_locations: Array<string>;
  message: string;
  upload_errors: Array<UploadError>;
}
```

{% tabs %}
{% tab title="Example" %}
```javascript
const uploadFile = await drive.uploadFile(acctPubKey, fileToUpload);
```
{% endtab %}

{% tab title="Explanations" %}
{% code overflow="wrap" %}
```javascript
// uploadFile メソッドを使用したJavascript SDKの例
// この行では`drive`オブジェクトの`uploadFile`メソッドを呼び出し、2つのパラメータを渡しています：
// 1. acctPubKey： ファイルがアップロードされるストレージアカウントの公開鍵を表す PublicKey オブジェクト。
// 2. fileToUpload： アップロードされるファイル名とファイルバッファを含む ShadowFile オブジェクトです。
const uploadFile = await drive.uploadFile(acctPubKey, fileToUpload);
```
{% endcode %}
{% endtab %}
{% endtabs %}

### **`uploadMultipleFiles`**

#### **Definition**

このメソッドは、Shadow Drive 上のストレージアカウントに複数のファイルをアップロードするために使用されます。ストレージアカウントの PublicKey、アップロードするファイルの FileList または ShadowFile 配列を含むデータオブジェクト、同時にアップロードするファイル数を表すオプションの concurrent number、アップロードされたファイルのバッチごとに指定するオプションのコールバック関数を受け取ります。アップロードされたファイルのファイル名、場所、トランザクション署名を含む Promise\<ShadowBatchUploadResponse\[]> を返します。

#### **Parameters**

* `key`: `PublicKey` - ファイルをアップロードするストレージアカウントの PublicKey です。
* `data`: `FileList | ShadowFile[]` - 
* `concurrent` (オプション)： `number` - 同時にアップロードするファイルの数。デフォルト：3
* `callback` (オプション)： `Function` - アップロードされるファイルのバッチごとにコールバック関数が実行されます。callback(num) のように、特定のバッチで確認されたファイル数を示す数値がコールバックに渡されます。

#### **Returns**

```json
{
  fileName: string;
  status: string;
  location: string;
}
```

{% tabs %}
{% tab title="Example" %}
```javascript
const drive = new ShadowDrive();
const acctPubKey = new anchor.web3.PublicKey(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);
const files = [
    {
        name: "file1.txt",
        file: new File(["hello"], "file1.txt"),
    },
    {
        name: "file2.txt",
        file: new File(["world"], "file2.txt"),
    },
    {
        name: "file3.txt",
        file: new File(["!"], "file3.txt"),
    },
];
const concurrentUploads = 2;
const callback = (response) => {
    console.log(`Uploaded file ${response.fileIndex}: ${response.fileName}`);
};
const responses = await drive.uploadMultipleFiles(
    acctPubKey,
    files,
    concurrentUploads,
    callback
);
console.log(responses);
```
{% endtab %}

{% tab title="Explanations" %}
{% code overflow="wrap" %}
```javascript
// uploadMultipleFiles メソッドを使用したJavascript SDKの例
// Shadow Drive クライアントのインスタンスを作成します。
const drive = new ShadowDrive();

// ファイルをアップロードするストレージアカウントの公開鍵を定義します。
const acctPubKey = new anchor.web3.PublicKey(
    "EY8ZktbRmecPLfopBxJfNBGUPT1LMqZmDFVcWeMTGPcN"
);

// アップロードするファイルの配列を定義します
const files = [
    {
        name: "file1.txt",
        file: new File(["hello"], "file1.txt"),
    },
    {
        name: "file2.txt",
        file: new File(["world"], "file2.txt"),
    },
    {
        name: "file3.txt",
        file: new File(["!"], "file3.txt"),
    },
];

// 最大同時アップロード数を定義します（オプション）
const concurrentUploads = 2;

// 各ファイルがアップロードされた後に呼び出されるコールバック関数を定義します（オプション）
const callback = (response) => {
    console.log(`Uploaded file ${response.fileIndex}: ${response.fileName}`);
};

// `uploadMultipleFiles`メソッドを呼び出して、すべてのファイルをアップロードします。
const responses = await drive.uploadMultipleFiles(
    acctPubKey,
    files,
    concurrentUploads,
    callback
);

// アップロードされた各ファイルに対して、サーバーから返されたレスポンスを表示します
console.log(responses);
```
{% endcode %}
{% endtab %}
{% endtabs %}

### **`userInfo`**

#### **Definition**

userInfo: PublicKey

### **Example - Using POST API requests via the Javascript SDK to make an account immutable with solana transaction signing**

```javascript
// 必要なモジュールや定数をインポートします
import * as anchor from "@project-serum/anchor";
import { getStakeAccount, findAssociatedTokenAddress } from "../utils/helpers";
import {
  emissions,
  isBrowser,
  SHDW_DRIVE_ENDPOINT,
  tokenMint,
  uploader,
} from "../utils/common";
import {
  ASSOCIATED_TOKEN_PROGRAM_ID,
  TOKEN_PROGRAM_ID,
} from "@solana/spl-token";
import { ShadowDriveVersion, ShadowDriveResponse } from "../types";
import fetch from "node-fetch";

/**
 *
 * @param {anchor.web3.PublicKey} key - ストレージアカウントの公開鍵
 * @param {ShadowDriveVersion} version - ShadowDrive バージョン (v1 or v2)
 * @returns {ShadowDriveResponse} - 確定したトランザクションID
 */
export default async function makeStorageImmutable(
  key: anchor.web3.PublicKey,
  version: ShadowDriveVersion
): Promise<ShadowDriveResponse> {
  let selectedAccount;
  
  // バージョンに基づき、選択したアカウントを取得します
  try {
    switch (version.toLocaleLowerCase()) {
      case "v1":
        selectedAccount = await this.program.account.storageAccount.fetch(key);
        break;
      case "v2":
        selectedAccount = await this.program.account.storageAccountV2.fetch(
          key
        );
        break;
    }
    
    // 関連するトークンアドレスを検索します
    const ownerAta = await findAssociatedTokenAddress(
      selectedAccount.owner1,
      tokenMint
    );
    const emissionsAta = await findAssociatedTokenAddress(emissions, tokenMint);
    
    // ステイクアカウントを取得します
    let stakeAccount = (await getStakeAccount(this.program, key))[0];
    let txn;
    
    // バージョンに応じたトランザクションを作成します
    switch (version.toLocaleLowerCase()) {
      case "v1":
        txn = await this.program.methods
          .makeAccountImmutable()
          .accounts({
            storageConfig: this.storageConfigPDA,
            storageAccount: key,
            stakeAccount,
            emissionsWallet: emissionsAta,
            owner: selectedAccount.owner1,
            uploader: uploader,
            ownerAta,
            tokenMint: tokenMint,
            systemProgram: anchor.web3.SystemProgram.programId,
            tokenProgram: TOKEN_PROGRAM_ID,
            associatedTokenProgram: ASSOCIATED_TOKEN_PROGRAM_ID,
            rent: anchor.web3.SYSVAR_RENT_PUBKEY,
          })
          .transaction();
      case "v2":
        txn = await this.program.methods
          .makeAccountImmutable2()
          .accounts({
            storageConfig: this.storageConfigPDA,
            storageAccount: key,
            owner: selectedAccount.owner1,
            ownerAta,
            stakeAccount,
            uploader: uploader,
            emissionsWallet: emissionsAta,
            tokenMint: tokenMint,
            systemProgram: anchor.web3.SystemProgram.programId,
            tokenProgram: TOKEN_PROGRAM_ID,
            associatedTokenProgram: ASSOCIATED_TOKEN_PROGRAM_ID,
            rent: anchor.web3.SYSVAR_RENT_PUBKEY,
          })
          .transaction();
        break;
    }
    
    // 最近のブロックハッシュと支払う人を設定します
    txn.recentBlockhash = (
      await this.connection.getLatestBlockhash()
    ).blockhash;
    txn.feePayer = this.wallet.publicKey;
    let signedTx;
    let serializedTxn;
    
    // トランザクションに署名し、シリアライズします
    if (!isBrowser) {
      await txn.partialSign(this.wallet.payer);
      serializedTxn = txn.serialize({ requireAllSignatures: false });
    } else {
      signedTx = await this.wallet.signTransaction(txn);
      serializedTxn = signedTx.serialize({ requireAllSignatures: false });
    }
    
    // トランザクションをサーバーに送信します
    const makeImmutableResponse = await fetch(
      `${SHDW_DRIVE_ENDPOINT}/make-immutable`,
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          transaction: Buffer.from(serializedTxn.toJSON().data).toString(
            "base64"
          ),
        }),
      }
    );
    
    // サーバーの応答を処理します
    if (!makeImmutableResponse.ok) {
      return Promise.reject(
        new Error(`Server response status code: ${
          makeImmutableResponse.status
        } \n
			Server response status message: ${(await makeImmutableResponse.json()).error}`)
      );
    }
    
    // レスポンスJSONを返します
    const responseJson = await makeImmutableResponse.json();
    return Promise.resolve(responseJson);
  } catch (e) {
    return Promise.reject(new Error(e));
  }
}
```


#### **この**[**リソース**](sdk-javascript.md)**の改善に協力する、または**[**フィードバック**](discord/)**を提供する**
