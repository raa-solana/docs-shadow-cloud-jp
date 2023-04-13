# FAQ

質問があれば、私たちがお答えします。

## 技術的質問

<details>

<summary>技術的な支援を受けるには、どこに相談すればよいのでしょうか？</summary>

私たちの[Discordサーバー](https://discord.gg/genesysgo)は、私たちと連絡を取るのに最適な場所です。
私たちは、専用のサポートセクションを持っています。

このFAQの他に、より深い技術的な問題が議論されているので、[Github Q&A](https://github.com/GenesysGo/shadow-drive/issues?q=is%3Aissue+is%3Aclosed) が役に立つかもしれません。

Discord Server: https://discord.gg/genesysgo

GitHub FAQ: https://github.com/GenesysGo/shadow-drive/issues?q=is%3Aissue+is%3Aclosed

（訳注：英語情報です）
</details>

<details>

<summary>ストレージアカウントの作成が失敗する場合はどうすればよいですか？</summary>

ストレージアカウントの作成に失敗した場合は、ウォレットに適切な量のSOLとSHDWの両方があることを確認してください。ストレージアカウントの作成には、取引手数料を賄うための少量のSOLと、最初のストレージ割り当てを賄うための若干のSHDWが必要です。ウォレットにこれらの要件を満たす十分な資金があることを確認してください。こちらのドキュメントをご覧ください： https://docs.shadow.cloud/build/the-cli#create-a-storage-account

ウォレットに適切な量のSOLとSHDWがあるにもかかわらず、ストレージアカウントの作成が失敗する場合、問題を引き起こしている他の要因が存在する可能性があります。考えられる原因としては、ネットワーク接続の問題、Shadow Drive、ノードの問題、SDKのバグや問題などが考えられます。

問題のトラブルシューティングを行うには、以下のことを試してみてください：

- [Shadow Drive ネットワーク](https://status.genesysgo.net/) が稼働していることを確認します。https://status.genesysgo.net/
- Shadow Drive [Change Log](../../reference/change-logs.md) を確認し、問題の原因となる既知の問題やバグがないかを確認してください。 https://docs.shadow.cloud/reference/change-logs
- Shadow Drive [support](https://discord.gg/genesysgo)にお問い合わせください。https://discord.gg/genesysgo

</details>

<details>

<summary>予約できるストレージ容量はどのくらいですか？</summary>

CLIに記載されているように、1バケットあたり1GBの上限があります: https://docs.shadow.cloud/build/the-cli#create-a-storage-account

現在、この上限を大幅に増やす開発が進められています。

</details>

<details>

<summary>ウォレットの導入方法に問題があると思われる場合、または取引がうまくいかない場合はどうすればよいですか？</summary>

ウォレットの実装方法に問題があると思われる場合、またはトランザクションが機能しない場合は、ウォレットアダプターのアップグレードを試してみてください。アダプタをインポートするプロセスが変更されている可能性があるため、Solanaウォレットアダプタのリポジトリでその例を確認してください。さらに、ウォレットを適切に実装してトランザクションを実行する方法の詳細については、Shadow DriveのドキュメントとSDKを参照できます。それでも問題がある場合は、Shadow Drive のサポートにお問い合わせください。

</details>

<details>

<summary>CLIからphantomウォレットを使用してストレージ アカウントを作成できますが、SDKからアプリで試したところ、残高不足で取引に失敗しました。これはなぜでしょうか？</summary>

Shadow Driveの活用を目的とした場合、経験上、～0.1SOLで残高不足のエラーを回避できます。また、CLIを使用した場合とSDKの方法を使用した場合の消費額に違いがあるかどうか、TXを調べてみてください。

</details>

<details>

<summary>Shadow DriveはLedgerウォレットの署名に対応していますか？</summary>

いいえ、Shadow Driveは現在、Ledgerウォレットの署名をサポートしていません。

</details>

<details>

<summary>`getStorageAccounts`メソッドを呼び出すと、アカウントは特定の順序で返されますか？</summary>

はい、GenesysGo Shadow Drive の `getStorageAccounts` メソッドを呼び出すと、アカウントは作成された順番で返されます。これは、アカウントが作成された順に返されるようにシステムが設計され、構築されているためです。 https://docs.shadow.cloud/build/the-sdk/sdk-javascript#getstorageaccounts

</details>

<details>

<summary>複数のファイルを同時に削除する方法はありますか？</summary>

現在のところ、複数のファイルを一度に削除することはできません。しかし、この機能をロードマップに追加しましたので、近い将来、この機能に取り組む予定です。ご指摘ありがとうございました！

</details>

<details>

<summary>アップロードハッシュをサーバーサイドで生成する際に、`edit-file`と`upload-file`の動作が異なるのは何か理由があるのでしょうか？</summary>

`edit-file`の機能は`upload-file`とは異なる動作をします。これは、シャドウドライブの最初のイテレーションで、すべてのファイルが追跡のために重要なメタデータを持つ関連アカウントをオンチェーンしていた名残りです。
 しかし、私たちはまだ文書化されておらず、SDKにも実装されていないいくつかの変更を行っています。SDKを介さず手動で行うアップロードリクエストのリクエストボディに `overwrite: true` を追加すると、ファイルを編集するのと同じことが行われます。

</details>

<details>

<summary>フロントエンドで、シャドウ取引と別の取引を同時に署名するようユーザーに求めることは可能ですか？</summary>

現在、フロントエンドでシャドウ取引と別の取引に同時に署名するようユーザーに求めることはできません。シャドウネットワークでは、Shadow Drive固有のトランザクションは、チェーンプログラム上のShadow Driveに関連する指示を持つことのみを許可します。それ以外の指示は、トランザクションを失敗させる原因となります。このセキュリティ機能は、悪意のあるトランザクションを防ぐために設置されています。

</details>

<details>

<summary>ReactアプリでFileオブジェクトを作成してShadow Driveにアップロードしようとしているのですが、エラーが出続けています。</summary>

このエラーは、ウォレットプロバイダーが準備される前にShadow Driveインスタンスが作成されたことが原因かもしれません。メインブランチの最新の例では、ドライブインスタンスを作成するuseEffectに若干の変更があり、この問題が解決される可能性があります。さらに、`new Blob([Buffer.from("data")])` を使用して、ファイルデータバッファが Blob に変換されることを確認してください。

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

<summary>Shadowトランザクションと無関係な別のトランザクションに同時に署名することは可能ですか？</summary>

現在、Shadowネットワークでは、Shadow Drive専用トランザクションにのみ、Shadow Driveオンチェーンプログラムに関連する指示を含めることができます。それ以外の指示は、セキュリティ対策としてトランザクションを失敗させることになります。つまり、ユーザーがShadowトランザクションと別の無関係なトランザクションに同時に署名することは不可能です。

</details>

<details>

<summary>400エラーが出ます</summary>

トランザクションの送信に400のタイムアウトが発生する場合、Solanaネットワークの混雑が原因である可能性が高いです。Solanaの混雑時にタイムアウトや再試行が発生するのは正常ですが、現在多くの方が優先料金を使用しているため、混雑に関連する問題の解決に役立つ可能性があります。RPCプロバイダーにお問い合わせください。

400エラーが「Invalid transaction supplied」と表示されている場合、[Discord](https://discord.gg/genesysgo)のサポートチャンネルに参加し、特定の方法について詳細を説明する必要があるかもしれません。このエラーの典型的な原因を解決するには、次のことを行ってください：

1. Discordのアナウンス(https://discord.gg/genesysgo)やネットワーク状況(https://status.genesysgo.net/)を確認し、プラットフォーム全体に問題がないことを確認します。
2. すべてのバージョンと依存関係をチェックします。Solanaウォレットアダプターの依存関係やJavaScript SDKのバージョンが最新である必要があります。
3. 選択したウォレットに問題がないことを再確認します。直接連絡を取る必要がある場合もあります。

</details>

<details>

<summary>Shadow Drive CLIを使用する際にENOTFOUNDエラーが発生した場合、どうすればよいですか？</summary>

Shadow Drive CLI を使用する際に ENOTFOUND エラーが発生した場合、お客様の側のローカル DNS の問題である可能性があります。ENOTFOUND は DNS リゾルバの問題であるため、インターネットサービスプロバイダー (ISP) に問題を解決するよう確認する必要があります。また、仮想プライベートネットワーク（VPN）を使用して問題が解決するかどうか試してみることもできます。

</details>

<details>
<summary>バグやセキュリティ問題を提出するにはどうすればよいですか？</summary>

**https://github.com/GenesysGo/shdw-drive-bug-reports**

 私たちは、セキュリティ関連の問題に対して、責任ある開示プロセスを遵守しています。セキュリティの脆弱性を責任を持って開示し、対処するために、以下のプロセスに従うことをお願いします。

#### **Bug Reporting Process**

1. このリポジトリに[new issue](https://github.com/GenesysGo/shdw-drive-bug-reports/issues/new/choose)を作成し、新しいバグレポートを提出する。https://github.com/GenesysGo/shdw-drive-bug-reports/issues/new/choose。
2. 問題の明確で簡潔な説明、それを再現する手順、関連するスクリーンショットやログを提供してください。
3. あなたの問題を「バグ」または「セキュリティ」として適宜ラベル付けしてください。

**重要**： セキュリティ関連の問題については、問題の説明文に機密情報を含めないでください。代わりに、必要な詳細を含むプルリクエストを私たちのリポジトリに提出し、問題が解決されるまで情報が隠されたままになるようにします。

**セキュリティ関連の問題は、このリポジトリを通じてのみ報告されるべきです**

バグレポートやセキュリティ問題については、このリポジトリの使用を強く推奨しますが、私たちの[**Discord**](https://discord.gg/genesysgo)サーバーを経由して連絡を取ることもできます。#shdw-drive-technical-support チャンネルに参加し、支援を求めてください。ただし、適切な処理と追跡のために、このGitHubリポジトリを通じてバグレポートを提出するようにリダイレクトすることに注意してください。

</details>

<details>

<summary>ネットワークを監視して、問題やダウンタイムがあるかどうかを知る方法はないでしょうか？</summary>

はい、シャドーネットワークの状況はこちらからご購読いただけます： https://status.genesysgo.net/

また、Twitter https://twitter.com/GenesysGo でフォローしたり、技術サポートのDiscord: https://discord.gg/genesysgo に参加することもできます。

</details>

## 一般的な質問

<details>

<summary>Shadow Driveの特徴は？</summary>

Shadow Driveは、複数のサービスオプションを提供するコモディティクラウドネットワークで、分散型台帳技術を活用し、垂直統合されたL1専用のストレージとコンピュートを提供しています。パフォーマンスを犠牲にすることなく、従来のクラウドプラットフォームの収益を民主化するために設計された唯一のクラウドネットワークです。S3互換であるShadow Driveは、オープンソースのSDKと相互運用性基準を維持し、一般的なビルダーツールやSDKから簡単にアクセスできるようになっています。その目的は、構築するアプリケーションに関係なく、構築を容易にする一般的なツールをサポートすることです。

</details>

<details>

<summary>GDPRはどのように扱われるのですか？</summary>

Shadow Driveは、GDPRに準拠するためのツールを開発者に提供し、ユーザーの個人データの削除を証明するための記録を提供することができます。GDPR準拠のための記録はすべてオンチェーンで保存され、Solanaバリデーターネットワークによって検証されています。その後、データは暗号化され、アルゴリズムによってネットワーク上に三重に分散されます。すべての取引は署名され、オンチェーンで公に検証可能です。

</details>

<details>

<summary>シャドウドライブはモバイルに対応していますか？</summary>

はい、Shadow Drive は、モバイルでの開発を積極的に行っているエコシステム・パートナーを通じてモバイルでサポートされています。詳しくはシャドウエコシステムページをご覧ください。https://docs.shadow.cloud/build/community-mainted-uis

さらに将来的には、当社の分散型台帳技術「DAGGER」により、低コストの分散型モバイルクラウドを求める方々のために、Solana Sagaを搭載したストレージソリューションが実現する予定です。詳細については、「Learn」セクションをご覧ください。詳しくはこちらでご覧いただけます： https://docs.shadow.cloud/learn#compute

</details>

<details>

<summary>Shadow Driveにデータを保存する場合、費用はどのくらいかかるのでしょうか？</summary>

Shadow Driveのストレージコストは、1GBあたり0.25SHDWの固定レートである卸売ネットワークコストによって駆動され、モーメントインタイムの見積もりを取得する様々なフロントエンドUIを介して推定することができます。その一例が、エコシステムパートナーによって設計されたフロントエンドで、ネットワークに関する詳細な情報も提供されます。フロントエンドへのリンクはこちらです： https://sdrive.app/stats

</details>

<details>

<summary>Shadow driveはS3対応ですか？</summary>

はい、Shadow Drive は S3 互換です。S3互換はクラウドストレージ業界で広く採用されている標準であり、多くのプロバイダがS3互換のAPIとプロトコルを提供しているため、ビルダーはクラウドストレージプロバイダをより柔軟に選択できるようになりました。つまり、開発者は互換性の問題を心配することなく、異なるサービス間でデータを容易に移動させることができます。さらに、S3互換は、高速で信頼性の高いクエリを可能にする堅牢なAPIを提供し、仮想マウント機能とともに、Web2、Web3、分散型台帳技術やAIの最前線にとって重要です。Shadow Driveは、開発者が自分のビルドに直接統合できるようにすること、そしてShadow Driveのための革新的なプラットフォームを創造するデザイナーの才能あるコミュニティをサポートすることを目的としています。詳しくはこちらでご覧いただけます： https://docs.shadow.cloud/learn/design#s3-compatibility

</details>

<details>

<summary>Shadow Driveの動力源となる物理インフラは？</summary>

Shadow Driveは、ベアメタルインフラのグローバルネットワーク上で動作し、すべてのコンピュートとストレージはベアメタル上に存在します。Shadow Driveの運用において、クラウドプロバイダーへの依存はありません。Shadow Driveの設計の詳細については、「Learn」カテゴリの「Design」セクションをご覧ください： https://docs.shadow.cloud/learn/design

</details>

<details>

<summary>ストレージコストはどのように決定されるのですか？</summary>

価格はフロントエンドによって異なり、単位SHDWあたりのストレージコストの市場価値も異なります。ネットワーク全体でストレージ1GBあたり0.25SHDWの固定があります。https://sdrive.app/stats などのフロントエンドのUIをご覧いただき、コストを判断してください。

</details>

<details>

<summary>ノードオペレーターになるにはどうしたらいいですか？</summary>

Shadow Operatorsは現在、クローズドプライベートアルファテスト中です。メインネットのローンチに向け、今後のアップデートも提供される予定です。

</details>

<details>

<summary>GenesysGoとは？</summary>

GenesysGo（GG）は、2021年4月にSolanaのバリデーターとして設立された会社です。それ以来、GGはRPCを提供し、Solanaのためのツールとインフラストラクチャの大規模なエコシステムを構築するために、サービスを拡大してきました。GGは、Solanaコミュニティのための革新的なソリューションの構築に専念する、才能ある開発者とコーダーのチームを擁しています。詳細については、同社のウェブサイト（http://shadow.cloud/）をご覧ください。

</details>

<details>

<summary>DAGGER/Shadow Driveを利用した場合、プロジェクトの宣伝は可能ですか？</summary>

はい、Shadow Driveチームは、Driveの上に構築している場合やDAGGERを使用している場合、あなたのプロジェクトについてぜひ聞きたいと思っています。可視性を得るための最良の方法は、docs-shadow-cloudレポに直接PRを提出し、あなたのプロジェクト/ビジネス、詳細、画像をシャドウエコシステムリストに追加することです： https://github.com/GenesysGo/docs-shadow-cloud

ここにあるファイルを編集するためにPRを提出する: https://github.com/GenesysGo/docs-shadow-cloud/blob/main/build/shadow-drive/community-mainted-uis.md

また、[Shadow Drive Discord](https://discord.com/invite/genesysgo)で共有することができます。シャドウエコシステムページに追加される自動化プロセスを近日中に公開する予定です。

</details>
