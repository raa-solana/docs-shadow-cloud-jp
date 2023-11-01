---
description: >-
  ShdwDriveは現在、ShdwDriveネットワークにアクセスするS3互換の方法を提供しています。
---

# S3互換 クライアントアクセス

ShdwDrive ネットワークのS3互換ゲートウェイを使用するには、[https://portal.shadow.cloud](https://portal.shadow.cloud)にアクセスし、ウォレットに接続してサインインし、アクセスキー/秘密鍵ペアを生成したいストレージアカウントに移動します。"Addons" タブを選択し、s3互換キー/シークレットペアを有効にし、追加することができます。

## クレデンシャルの取得

指定したストレージアカウントで S3 Access アドオンを有効にすると、[https://portal.shadow.cloud](https://portal.shadow.cloud)で選択したストレージアカウントに移動して、s3互換のキー/シークレットペアを表示および追加できます。

認証情報を取得するには、"View Credentials "をクリックしてください。

## アクセス許可

一度有効にすると、指定したストレージアカウント用に生成した各キー/シークレットペアの個別のパーミッションを管理することができます。パーミッションのリストは、既存のs3クライアントと互換性があるように設定されています。互換性のあるs3クライアントの例としては、rclone、s3cmd、aws s3 cliなどがあります。s3互換ゲートウェイのためのツールは数多く存在し、ShdwDrive ネットワークにこれを組み込むことにしたのはそのためです。

一般的に、キーの一つでアップロードを有効にしたい場合、以下のパーミッションを有効にする必要があります：

```
Get Object, Put Object, List Multipart Upload Parts, Abort Multipart upload
```

また、キーが読み取り可能かどうかを制御することもできるため、さまざまな用途に合わせて読み取り、書き込み、読み取り＋書き込みのキーを持つことができます。

## ゲートウェイ　エンドポイント

ShdwDrive ネットワーク上のs3互換ゲートウェイにアクセスするには、s3クライアントが以下のゲートウェイのいずれかを使用するように設定する必要があります：

1. https://us.shadow.cloud
2. https://eu.shadow.cloud

これらのエンドポイントは、ShdwDrive ネットワークへのリクエストをプロキシし、地理的な好みに応じて可能な限り最良のネットワーク接続を可能にします。すべてのアップロードは同期され、グローバルに利用できるため、どちらのエンドポイントも使用できます。

## クライアント設定の例

### RCLONE

以下は、rclone設定ファイルに追加できる例です。一般的に、これは、\`\~/.config/rclone/rclone.conf\` に配置されます。

```
[shdw-cloud]
type = s3
provider = Other
access_key_id = [redacted]
secret_access_key = [redacted]
endpoint = https://us.shadow.cloud
acl = public-read
bucket_acl = public-read
```

## スピード

ShdwDriveネットワークの安定性を確保するため、速度は当初20MiB/秒に制限されています。生成する個々のキー/シークレットペアごとに、帯域幅レートの制限をアップグレードすることができます。アップグレードされた速度制限は 40 MiB/s です。

## セキュリティ

s3 キーが漏洩した場合、簡単にキーをローテーションすることができます。[https://portal.shadow.cloud](https://portal.shadow.cloud)に移動し、ウォレットを接続し、ストレージアカウントを選択し、"Addons" タブをクリックするだけです。そこから "Rotate" ボタンをクリックし、指定されたキー/シークレットペアをローテーションし、新しいペアを生成することができます。
