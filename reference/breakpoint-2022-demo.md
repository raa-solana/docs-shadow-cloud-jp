---
description: D.A.G.G.E.R.を使用して、Shdw Cloudポータルを通じてトラストレスなVMをデプロイする方法をご覧ください。
---

# Solana Breakpoint 2022 Demo

<figure><img src="https://media.tenor.com/r-mBXs5HH2EAAAAM/thumbs-up-keanu-reeves.gif" alt=""><figcaption></figcaption></figure>

### Get Setup on Devnet

このデモに挑戦し始める前に、完全に明確にしておきたいことがあります - ウォレットをDevnetに設定する必要があります。デフォルトでは、PhantomやSolflareのようなウォレットで作業する場合、それはメインネットに設定されています。ウォレットの設定で「ネットワーク」を選択する必要があります...

<figure><img src="../.gitbook/assets/Screenshot 2022-11-01 at 1.07.38 PM.png" alt=""><figcaption></figcaption></figure>

そして、Devnetになるように反転させます（ほら、デフォルトではMainnetに設定されています。 Devnetをチェックさせたいでしょ）

<figure><img src="../.gitbook/assets/Screenshot 2022-11-01 at 1.07.52 PM.png" alt=""><figcaption></figcaption></figure>

DevnetのアカウントにSOLやUSDCがなくても、心配しないでください。私たちがカバーします。このサイトに移動してデモを開始します👇。

{% hint style="danger" %}
BreakPoint でデプロイ中のDemo
{% endhint %}

ウォレットを接続すると、Shdw Cloudのダッシュボードページに着地します。右上にある2つのボタンが見えますか？これをクリックすると、DevnetウォレットにSOLとUSDCがエアドロップされます。今はあまり夢中にならず、私たちのために少し取っておいてください。必要な分だけを取ってください。

<figure><img src="../.gitbook/assets/Screenshot 2022-11-01 at 1.13.12 PM.png" alt=""><figcaption></figcaption></figure>

### Create a VM

さて、Devnetを正しく理解したところで、VMのデプロイがいかに簡単か見てみましょう。あまりに簡単なので、自動車保険会社の前時代的な擬人化された広報担当者なら、著作権や商標の規則に違反することなく実行できるかもしれませんね。

画面の中央に、大きなボタンがあります。これをタップしてください。

<figure><img src="../.gitbook/assets/Screenshot 2022-11-01 at 1.15.23 PM.png" alt=""><figcaption></figcaption></figure>

次の画面に移動し、いくつかの質問に答えますが、まずは...。

#### Where?

<figure><img src="../.gitbook/assets/Screenshot 2022-11-01 at 1.23.09 PM.png" alt=""><figcaption></figcaption></figure>

このデモでは、クリックできる選択肢はアムステルダムだけですが、拡張計画がどこにあるのかがわかります。アムステルダムを選択すると、緑色にハイライトされ、次の決定までスクロールダウンします。しかし、その前に、いくつかの重要なテキストに気づくことを確認してください。同じウォレットからそのデータセンターにデプロイされたすべてのVMは、プライベートIPスペースを通じて互いに直接通信します。ですから、複数のVMをデプロイした場合、それらはすでに同じプライベートVLAN内に配置されることになります。

では、次の質問です...

#### Which?

どのOSを使いたいですか？現在、Linuxディストロがサポートされています。注：ディストロの中にドロップダウンがあり、そこで複数のバージョンが利用可能になります。

<figure><img src="../.gitbook/assets/Screenshot 2022-11-01 at 1.30.39 PM.png" alt=""><figcaption></figcaption></figure>

#### How much?

社長、馬力はどれくらい必要ですか？そして、データのバックアップを取りますか？注：デフォルトで表示されるオプションは、あらかじめ設定されたクッキーカッターのようなVMです。カスタム構成をクリックすると、デプロイされるVMのスペックを自分で作成することができます。

<figure><img src="../.gitbook/assets/Screenshot 2022-11-01 at 1.33.17 PM.png" alt=""><figcaption></figcaption></figure>

#### Who?

そして最後に、VMでどのように認証を行うかです。パスワードを生成したり、VM内で使用するSSHキーを配布したりすることができます。

<figure><img src="../.gitbook/assets/Screenshot 2022-11-01 at 1.40.43 PM.png" alt=""><figcaption></figcaption></figure>

時間です。Solana Payで支払うボタンをクリックすると、プロビジョニングが始まります。デプロイには数分かかりますが、その後、分散型インフラ上で本格的なVMが稼働するようになります。数分後、"ready fren? "というダイアログボックスが表示され、準備が整ったことがわかります。ランディング・ページに戻り、ダッシュボードにVMの詳細が表示されます。

<figure><img src="../.gitbook/assets/vm.png" alt=""><figcaption></figcaption></figure>
