# FAQ

質問があれば、私たちがお答えします。

## 技術的質問

<details>

<summary>技術的な支援を受けるには、どこに相談すればよいのでしょうか？</summary>

私たちの[Discordサーバー](https://discord.gg/genesysgo)は、私たちと連絡を取るのに最適な場所です。
私たちは、専用のサポートセクションを持っています。

このFAQの他に、より深い技術的な問題が議論されているので、[Github Q\&A](https://github.com/GenesysGo/Shdw-drive/issues?q=is%3Aissue+is%3Aclosed) が役に立つかもしれません。

Discord Server: https://discord.gg/genesysgo

GitHub FAQ: https://github.com/GenesysGo/Shdw-drive/issues?q=is%3Aissue+is%3Aclosed

（訳注：英語情報です）
</details>

<details>

<summary>CLIを使おうとすると「410 Gone」エラーが出ます。どうすればいいでしょうか？</summary>

このエラーは、CLI で使用している Solana RPC プロバイダが、CLI が機能するために必要な特定の RPC メソッドをサポートしていないことを意味します。これは、\`getProgramAccounts\` または他のメソッドである可能性があります。

[Helius](https://www.helius.dev/)、Hellomoon.io、またはすべてのSolana RPCメソッドを使用できるその他のプレミアムSolana RPCプロバイダーのような、よりプレミアムなRPCプロバイダーを試してみることをお勧めします。

</details>

<details>

<summary>ストレージアカウントの作成が失敗する場合はどうすればよいですか？</summary>

ストレージアカウントの作成に失敗した場合は、ウォレットに適切な量のSOLとSHDWの両方があることを確認してください。ストレージアカウントの作成には、取引手数料を賄うための少量のSOLと、最初のストレージ割り当てを賄うための若干のSHDWが必要です。ウォレットにこれらの要件を満たす十分な資金があることを確認してください。こちらのドキュメントをご覧ください： https://docs.shadow.cloud/build/the-cli#create-a-storage-account

ウォレットに適切な量のSOLとSHDWがあるにもかかわらず、ストレージアカウントの作成が失敗する場合、問題を引き起こしている他の要因が存在する可能性があります。考えられる原因としては、ネットワーク接続の問題、ShdwDrive、ノードの問題、SDKのバグや問題などが考えられます。

問題のトラブルシューティングを行うには、以下のことを試してみてください：

- [ShdwDrive のネットワーク](https://status.genesysgo.net/) が稼働していることを確認します。https://status.genesysgo.net/
- ShdwDrive [Change Log](../reference/change-logs.md) を確認し、問題の原因となる既知の問題やバグがないかを確認してください。 https://docs.shadow.cloud/reference/change-logs
- ShdwDrive [support](https://discord.gg/genesysgo)にお問い合わせください。https://discord.gg/genesysgo

</details>

<details>

<summary>予約できるストレージ容量はどのくらいですか？</summary>

ユーザーが予約できる容量は最低4kバイト。

現在、この上限を大幅に増やす開発が進められています。

</details>

<details>

<summary>アップロードできる最小または最大のファイルは？</summary>

現在、以下の制限があります：

* 最低： 最小：4kb。100バイトのファイルをアップロードしても、4kbの容量を占めます。これはレプリケーションのオーバーヘッドが必要なためです。
* 最大： 最大：1GB。

[s3-compatible-client-access](s3-compatible-client-access.md)では、1TiBまでのファイルをアップロードすることができます。

最大ファイルサイズを増やすための開発が進行中です。

</details>

<details>

<summary>ウォレットの実装方法に問題があると思われる場合、または取引がうまくいかない場合はどうすればよいですか？</summary>

ウォレットの実装方法に問題があると思われる場合、またはトランザクションが機能しない場合は、ウォレットアダプターのアップグレードを試してみてください。アダプタをインポートするプロセスが変更されている可能性があるため、Solanaウォレットアダプタのリポジトリでその例を確認してください。

さらに、ウォレットを適切に実装してトランザクションを実行する方法の詳細については、ShdwDrive のドキュメントと SDK を参照することができます。ここで例を確認することができます： https://docs.shadow.cloud/build/the-sdk/sdk-javascript#example-post-request-via-sdk-make-immutable

もしあなたがreactを使って `const drive = await new ShdwDrive(connection, wallet).init();` を使ってウォレットを構築していて、「Cannot read properties of undefined (reading 'toBytes') 」というエラーが発生したら、ウォレット全体を必ず渡してdeconstruct されないようにすることを忘れないで下さい。

まだ問題がある場合は、ShdwDriveのサポートにお問い合わせください。

</details>

<details>

<summary>CLIからphantomウォレットを使用してストレージ アカウントを作成できますが、SDKからアプリで試したところ、残高不足で取引に失敗しました。これはなぜでしょうか？</summary>

ShdwDriveの活用のためには、経験上、〜0.1SOLで残高不足のエラーを回避できます。また、CLIを使用した場合とSDKの方法を使用した場合の消費額に違いがあるかどうか、TXを調べてみてください。

</details>

<details>

<summary>ShdwDriveはLedgerウォレットの署名に対応していますか？</summary>

いいえ、ShdwDrive は現在、Ledger のウォレット署名をサポートしていません。Ledgerのサポートを提供できない理由は、Ledger用のSolanaアプリにメッセージ署名機能がないためで、私たちのシステムはこの機能に依存しているためです。

Ledgerサポートの実装を早めるため、このGitHubの課題にコメントを残していただくことをご検討ください： https://github.com/solana-labs/wallet-adapter/pull/712

</details>

<details>

<summary>`getStorageAccounts`メソッドを呼び出すと、アカウントは特定の順序で返されますか？</summary>

はい、GenesysGo ShdwDrive の `getStorageAccounts` メソッドを呼び出すと、アカウントは作成された順番で返されます。これは、アカウントが作成された順に返されるようにシステムが設計され、構築されているためです。 https://docs.shadow.cloud/build/the-sdk/sdk-javascript#getstorageaccounts

</details>

<details>

<summary>複数のファイルを同時に削除する方法はありますか？</summary>

現在のところ、複数のファイルを一度に削除することはできません。しかし、この機能をロードマップに追加しましたので、近い将来、この機能に取り組む予定です。ご指摘ありがとうございました！

</details>

<details>

<summary>アップロードハッシュをサーバーサイドで生成する際に、`edit-file`と`upload-file`の動作が異なるのは何か理由があるのでしょうか？</summary>

`edit-file`の機能は`upload-file`とは異なる動作をします。これは、ShdwDriveの最初のイテレーションで、すべてのファイルが追跡のために重要なメタデータを持つ関連アカウントをオンチェーンしていた名残りです。
 しかし、私たちはまだ文書化されておらず、SDKにも実装されていないいくつかの変更を行っています。SDKを介さず手動で行うアップロードリクエストのリクエストボディに `overwrite: true` を追加すると、ファイルを編集するのと同じことが行われます。

</details>

<details>

<summary>フロントエンドで、Shdw取引と別の取引を同時に署名するようユーザーに求めることは可能ですか？</summary>

現在、フロントエンドでShdw取引と別の取引に同時に署名するようユーザーに求めることはできません。Shdwネットワークでは、ShdwDrive固有のトランザクションは、チェーンプログラム上のShdwDriveに関連する指示を持つことのみを許可します。それ以外の指示は、トランザクションを失敗させる原因となります。このセキュリティ機能は、悪意のあるトランザクションを防ぐために設置されています。

</details>

<details>

<summary>ReactアプリでFileオブジェクトを作成してShdwDriveにアップロードしようとしているのですが、エラーが出続けています。</summary>

このエラーは、ウォレットプロバイダーが準備される前にShdwDriveインスタンスが作成されたことが原因かもしれません。メインブランチの最新の例では、ドライブインスタンスを作成するuseEffectに若干の変更があり、この問題が解決される可能性があります。さらに、`new Blob([Buffer.from("data")])` を使用して、ファイルデータバッファが Blob に変換されることを確認してください。

</details>

<details>

<summary>nodeスクリプトで`createStorageAccount`を使っていて、問題なく動作しているのですが、Reactアプリで使おうとすると、403エラーが出ます。</summary>

デフォルトでは、使用されるRPCはSolana mainnet rpc api.mainnet-beta.solana.com です。Solana mainnet rpcのエンドポイントがどのように制限されているかは制御できないため、それでブロックされている場合は、有料のRPCにサインアップする必要があります。セキュリティ上の理由から、エンドポイントがブラウザからのリクエストをブロックしている可能性があります。

追加のヘルプについては、私たちの[Discord](https://discord.gg/genesysgo)に参加し、サポートチャネルで尋ねることを検討してください。

</details>

<details>

<summary>"Blockhash not found" エラーでメソッドが失敗するのはなぜですか？</summary>

これはSolana RPC側の問題で、残念ながらできることはメソッドを再試行することだけです。アプリケーションにリトライやエラー処理を実装することを検討してください。

</details>

<details>

<summary>SDKを使って、アカウントからファイルの内容を取得するにはどうすればよいですか？</summary>

https://shdw-drive.genesysgo.net に通常の GET リクエストを送信することで、アカウントからファイル内容を取得することができます。APIメソッドの詳細はこちらでご覧いただけます： https://docs.shadow.cloud/build/the-api

</details>

<details>

<summary>ファイルタイプに関する情報を取得し、異なるタイプのファイルを扱えるようにする方法はありますか？</summary>

ファイルタイプに関する情報を取得するには、HEADリクエストまたはGETリクエストを行うことができます。GETリクエストの場合、レスポンスヘッダにはコンテンツタイプが含まれているはずです。APIメソッドについては、こちらをご覧ください： https://docs.shadow.cloud/build/the-api

</details>

<details>

<summary>ファイルのメタデータを取得するにはどうすればよいですか？</summary>

https://shdw-drive.genesysgo.net に POST リクエストを行うことで、ファイルのメタデータを取得することができます。レスポンスには、そのファイルのメタデータが含まれます。APIメソッドについては、こちら（https://docs.shadow.cloud/build/the-api）をご参照ください。

</details>

<details>

<summary>ストレージアカウントの所有権を他のウォレットに移すことは可能ですか？</summary>

現在、この機能はCLIやSDKでアクティブな機能ではありません。しかし、将来のリリースのために計画された機能です。

</details>

<details>

<summary>ストレージの所有者だけがファイルを編集できるんですか？</summary>

はい、現在はストレージアカウントの所有者のみがファイルを編集することができます。

</details>

<details>

<summary>アップロードに失敗した場合、中断したところから再開することは可能でしょうか？</summary>

いいえ、残念ながらアップロードに失敗した場合、中断したところから再開することはできません。ただし、CLIはアップロード前にファイルをチェックし、すでに存在する場合はスキップします。また、各ファイルのアップロードに対して、ファイルがすでに存在するかどうかを示す出力JSONファイルを受け取ることができます。

</details>

<details>

<summary>アンカープログラムでrust SDKを使用することはできますか？</summary>

いいえ、SDKはhttpリクエストを送信するためにインターネットアクセスが必要です。任意のhttp応答は決定論的ではなく、異なるSolana元帳の状態遷移を生成する可能性があるため、これはSolanaランタイム内で許可されていません。

</details>

<details>

<summary>Shdw取引と無関係な別の取引に同時に署名することは可能ですか？</summary>

現在、Shdwネットワークでは、ShdwDrive専用トランザクションにのみ、ShdwDriveオンチェーンプログラムに関連する指示を含めることができます。それ以外の指示は、セキュリティ対策としてトランザクションを失敗させることになります。つまり、ユーザーがShdw取引と別の無関係な取引に同時に署名することは不可能です。

</details>

<details>

<summary>400エラーが出ます</summary>

エラーが発生したコマンドに --log-level debug を設定してみてください。最新のバージョンと依存関係をインストールしたことを確認し、キーペア・ファイルが正しくアクセスされていることを確認してください。SOLとSHDWの両方でウォレットが適切に資金供給されていること、Solana接続オブジェクトを適切に処理していること、Solana RPC関連のエラーが発生していないことを確認します。さらなるヘルプのために、私たちの[Discord](https://discord.gg/genesysgo)のテクニカルサポートチャンネルでログをキャプチャし、関連コードを共有することができます。

</details>

<details>

<summary>ShdwDrive CLIを使用する際にENOTFOUNDエラーが発生した場合、どうすればよいですか？</summary>

ShdwDrive CLI を使用する際に ENOTFOUND エラーが発生した場合、お客様の側のローカル DNS の問題である可能性があります。ENOTFOUND は DNS リゾルバの問題であるため、インターネットサービスプロバイダー (ISP) に問題を解決するよう確認する必要があります。また、仮想プライベートネットワーク（VPN）を使用して問題が解決するかどうか試してみることもできます。

</details>

<details>

<summary>エラーが出るときに確認すべきことは？</summary>

エラーが出ているコマンドに --log-level debug を設定してみてください。 \~/.config/solana/id.json が存在することを確認してください。

</details>

<details>

<summary>ShdwDrive APIを呼び出す際の「Internal Server Error」とはどのような意味ですか？</summary>

このエラーの原因はいくつかありますが、最も一般的なのは、元のバージョン1形式のストレージアカウントから新しいバージョン2形式に移行されていないファイルです。レガシースタイルのShdwDriveアカウントを作成したユーザーについては、移行手順を終了してください。

その他のヘルプについては、Discord (https://discord.gg/genesysgo) でお問い合わせください。

</details>

<details>

<summary>バグやセキュリティ問題を提出するにはどうすればよいですか？</summary>

**https://github.com/GenesysGo/shdw-drive-bug-reports**

 私たちは、セキュリティ関連の問題に対して、責任ある開示プロセスを遵守しています。セキュリティの脆弱性を責任を持って開示し、対処するために、以下のプロセスに従うことをお願いします。

**Bug Reporting Process**

1. このリポジトリに[new issue](https://github.com/GenesysGo/shdw-drive-bug-reports/issues/new/choose)を作成し、新しいバグレポートを提出する。https://github.com/GenesysGo/shdw-drive-bug-reports/issues/new/choose。
2. 問題の明確で簡潔な説明、それを再現する手順、関連するスクリーンショットやログを提供してください。
3. あなたの問題を「バグ」または「セキュリティ」として適宜ラベル付けしてください。

**重要**： セキュリティ関連の問題については、問題の説明文に機密情報を含めないでください。代わりに、必要な詳細を含むプルリクエストを私たちのリポジトリに提出し、問題が解決されるまで情報が隠されたままになるようにします。

**セキュリティ関連の問題は、このリポジトリを通じてのみ報告されるべきです**

</details>

<details>

<summary>ネットワークを監視して、問題やダウンタイムがあるかどうかを知る方法はないでしょうか？</summary>

はい、Shdwネットワークの状況はこちらからご購読いただけます： https://status.genesysgo.net/

また、Twitter https://twitter.com/GenesysGo でフォローしたり、技術サポートのDiscord: https://discord.gg/genesysgo に参加することもできます。

</details>

<details>

<summary>私のストレージアカウントに<a href="https://docs.shadow.cloud/reference/shdw-token">$SHDW</a>がなくなった場合、変更可能な料金をカバーするための猶予期間はありますか？</summary>

はい、あなたのストレージアカウントは6ヶ月間保管されます。その後、ストレージノードは、あなたのストレージアカウントとその中のすべてのデータを削除されます。

</details>

<details>

<summary>変更可能な料金のストレージ・アカウントに、さらに<a href="https://docs.shadow.cloud/reference/shdw-token">$SHDW</a> を追加するにはどうすればよいですか？</summary>

1. [SDKs](the-sdk/)にある \`topUp\` メソッドを使用するか、ストレージアカウントのトークンアドレスに$SHDWを直接送信します。
2. SDKのいずれかの\`refreshStake\`メソッドを使用して、ストレージアカウントのステータスをリフレッシュします。これは、あなたのために行われません、あなたは手動でこのステップを行う必要があります。

</details>

<details>

<summary>可変ストレージのの保存料はいくらですか？</summary>

可変ストレージの料金は、特定のUSD価格を目標としています。現在、それは1年あたり1gibyteあたり0.05米ドルです。これは、solana・エポック（可変ストレージ料金が徴収される間隔）あたり1gibあたり0.0002739726米ドルになります。この目標価格は、料金徴収時に$SHDW/$USDCに換算されます。

可変ストレージ使用料は、保存されたバイト数に応じて徴収されます。

</details>

<details>

<summary> 良い事例がある場合、PRはどこに送ればよいのでしょうか？技術文書についてフィードバックをする方法はありますか？</summary>

私たちは、あなたが私たちのドキュメントに提供できるフィードバックや例を歓迎します。私たちの技術文書リポジトリ - https://github.com/GenesysGo/docs-Shdw-cloud/tree/main - にPRを提出してください。

</details>

### 一般的な質問

<details>

<summary>ShdwDriveの特徴は？</summary>

ShdwDriveは、複数のサービスオプションを提供するコモディティクラウドネットワークで、分散型台帳技術を活用し、垂直統合されたL1専用のストレージとコンピュートを提供しています。パフォーマンスを犠牲にすることなく、従来のクラウドプラットフォームの収益を民主化するために設計された唯一のクラウドネットワークです。S3互換であるShdwDriveは、オープンソースのSDKと相互運用性基準を維持し、一般的なビルダーツールやSDKから簡単にアクセスできるようになっています。その目的は、構築するアプリケーションに関係なく、構築を容易にする一般的なツールをサポートすることです。

</details>

<details>

<summary>ShdwDriveは、どのようにデータのプライバシーとセキュリティを確保しているのですか？</summary>

ShdwDriveは、データを暗号化および消去符号化し、その断片を分散ネットワークにアルゴリズムで分散させることで、データのプライバシーとセキュリティを確保します。これはスマートコントラクトを介してトラストレスに行われ、署名されたSolanaトランザクションを必要とし、公に検証可能なオンチェーンログが作成されます。さらに、ShdwDriveは、開発者がGDPRを遵守するために必要なツールを提供し、ユーザーの個人データを削除したことを証明する記録を示すことができます。 

</details>

<details>

<summary>GDPRはどのように扱われますか？？</summary>

ShdwDriveは、GDPRに準拠するためのツールを開発者に提供し、ユーザーの個人データの削除を証明する記録を提供することができます。GDPR準拠のための記録はすべてオンチェーンで保存され、Solana Validatorネットワークによって検証されています。その後、データは暗号化され、アルゴリズムによって3重にネットワークに分散されます。すべてのトランザクションは署名され、オンチェーンで公に検証可能です。

</details>

<details>

<summary>ShdwDriveはモバイルに対応していますか？</summary>

はい、ShdwDrive は、モバイルでの開発を積極的に行っているエコシステム・パートナーを通じてモバイルでサポートされています。詳しくはShdwエコシステムページをご覧ください。https://docs.shadow.cloud/build/community-mainted-uis

さらに将来的には、当社の分散型台帳技術「_D.A.G.G.E.R._」により、低コストの分散型モバイルクラウドを求める方々のために、Solana Sagaを搭載したストレージソリューションが実現する予定です。詳細については、「Learn」セクションをご覧ください。詳しくはこちらでご覧いただけます： https://docs.shadow.cloud/learn#compute

</details>

<details>

<summary>ShdwDriveはS3対応ですか？</summary>

はい、ShdwDriveはS3互換です。S3互換はクラウドストレージ業界で広く採用されている標準であり、多くのプロバイダがS3互換のAPIとプロトコルを提供しています。これは、開発者が互換性の問題を心配することなく、異なるサービス間でデータを簡単に移動できることを意味します。さらに、S3互換性は、仮想マウント機能とともに、高速で信頼性の高いクエリを可能にする堅牢なAPIを提供し、Web2、Web3、分散台帳技術やAIの最前線にとって重要なものとなります。ShdwDriveは、開発者が自分のビルドに直接統合できるようにし、ShdwDriveのための革新的なプラットフォームを創造する才能あるデザイナーのコミュニティをサポートすることを目指しています。詳しくはこちら：https://docs.shadow.cloud/learn/design#s3-compatibility

</details>

<details>

<summary>ShdwDriveの動力源となる物理インフラは？</summary>

ShdwDriveは、ベアメタルインフラのグローバルネットワーク上で動作し、すべてのコンピュートとストレージはベアメタル上に存在します。ShdwDriveの運用において、クラウドプロバイダーへの依存はありません。ShdwDriveの設計の詳細については、「Learn」カテゴリの「Design」セクションをご覧ください： https://docs.shadow.cloud/learn/design

</details>

<details>

<summary>GenesysGoとは？</summary>

GenesysGo（GG）は、2021年4月にSolanaのバリデーターとして設立された会社です。その後、GGはSolanaのためのツールやインフラの大規模なエコシステムに焦点を当て、提供範囲を拡大してきました。提供範囲の詳細については、「Learn」カテゴリーでご覧いただけます。GGには、Solanaコミュニティのために革新的なソリューションを構築することに専念する、才能ある開発者とコーダーのチームがあります。詳細については、当社のウェブサイト（http://shadow.cloud/）をご覧ください。

</details>

<details>

<summary>D.A.G.G.E.R./ShdwDriveを利用した場合、プロジェクトの宣伝は可能ですか？</summary>

はい、ShdwDriveチームは、Driveの上に構築している場合や _D.A.G.G.E.R._ を使用している場合、あなたのプロジェクトについてぜひ聞きたいと思っています。可視性を得るための最良の方法は、docs-shadow-cloudレポに直接PRを提出し、あなたのプロジェクト/ビジネス、詳細、画像をShdwエコシステムリストに追加することです： https://github.com/GenesysGo/docs-Shdw-cloud

ここにあるファイルを編集するためにPRを提出する: https://github.com/GenesysGo/docs-Shdw-cloud/blob/main/build/Shdw-drive/community-mainted-uis.md

また、[ShdwDrive Discord](https://discord.com/invite/genesysgo)で共有することができます。Shdwエコシステムページに追加される自動化プロセスを近日中に公開する予定です。

</details>
