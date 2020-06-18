**Managing Cloud Infrastructure with Terraform**

このクエストでは、Google Cloud の上級ユーザーを対象に、Terraform を使用してクラウド リソースを記述および起動する方法について学習します。  
Terraform は、API を宣言的な構成ファイルに体系化するオープンソース ツールであり、チームメンバー間での共有、コード処理、編集、レビュー、バージョン管理を行うことができます。

# Terraform の基礎知識
## はじめに
Terraform を使用すると、クラウド インフラストラクチャを安全かつ想定どおりに作成、変更、改善できます。Terraform は、API を宣言的な構成ファイルにコード化するオープンソースのツールです。ファイルは、チームメンバー間で共有したり、コードとして扱ったりできます。また、編集、レビュー、バージョン管理を行うこともできます。

## 目標
- Google Cloud で Terraform を使ってみる。
- Terraform をインストール バイナリからインストールする。
- Terraform を使用して VM インスタンスのインフラストラクチャを作成する。

## Terraform とは
Terraform とは、**インフラストラクチャを安全かつ効率的に構築、変更、バージョン管理**するためのツールです。既存の一般的なサービス プロバイダだけでなく、社内独自のカスタム ソリューションも管理できます。

構成ファイルには、Terraform が単一のアプリケーションまたはデータセンター全体を実行するために必要なコンポーネントが記述されています。Terraform は、インフラストラクチャを希望の状態にするまでの操作を記載した _実行プランを生成_ します。  
次にそのプランを実行して、構成ファイルに記述されたインフラストラクチャを構築します。構成に変更があった場合、Terraform では変更箇所を特定し、変更を含む実行プランを作成して適用することができます。

Terraform で管理できるインフラストラクチャは、コンピューティング インスタンス、ストレージ、ネットワークなどの低レベルのコンポーネントから、DNS エントリ、SaaS 機能などの高レベルのコンポーネントまで、多岐にわたります。

## 主な機能
### Infrastructure as Code
インフラストラクチャの記述には高レベルの構成構文を使用します。これにより、データセンターのブループリントをバージョン管理したり、他のコードと同じように扱ったりできます。インフラストラクチャを共有したり再利用したりすることも可能です。

### 実行プラン
Terraform には、実行プランを生成する「プランニング」のステップがあります。このプランでは、apply コマンドを呼び出した時の実行内容を確認できるため、インフラストラクチャ構築時に、予想外の操作が実行されるのを回避できます。

### リソースグラフ
Terraform では、すべてのリソースのグラフが作成され、依存しないリソースの作成と変更が同時に実行されます。こうしてインフラストラクチャは効率的に構築され、インフラストラクチャ内の依存関係も把握できます。

### 変更の自動化
インフラストラクチャには、最小限の手作業で、複雑な変更を適用できます。上述の実行プランとリソースグラフを使うと、何がどの順番で変更されるかがわかるため、多くの人的ミスを回避できます。

## Terraform のインストールの確認
Terraform は、Cloud Shell にプリインストールされています。

Terraform が利用可能であること確認するには、次のコードを実行します。

```bash
terraform
```

次のようなヘルプ出力が表示されます。

```
Usage: terraform [--version] [--help] <command> [args]

The available commands for execution are listed below.
The most common, useful commands are shown first, followed by
less common or more advanced commands. If you're just getting
started with Terraform, stick with the common commands. For the
other commands, please read the help and docs before usage.

Common commands:
    apply              Builds or changes infrastructure
    console            Interactive console for Terraform interpolations
    destroy            Destroy Terraform-managed infrastructure
    env                Workspace management
    fmt                Rewrites config files to canonical format
    get                Download and install modules for the configuration
    graph              Create a visual graph of Terraform resources
    import             Import existing infrastructure into Terraform
    init               Initialize a Terraform working directory
    output             Read an output from a state file
    plan               Generate and show an execution plan
    providers          Prints a tree of the providers used in the configuration
    push               Upload this Terraform module to Atlas to run
    refresh            Update local state file against real resources
    show               Inspect Terraform state or plan
    taint              Manually mark a resource for recreation
    untaint            Manually unmark a resource as tainted
    validate           Validates the Terraform files
    version            Prints the Terraform version
    workspace          Workspace management

All other commands:
    debug              Debug output management (experimental)
    force-unlock       Manually unlock the terraform state
    state              Advanced state management
```

<h2 id="step7">インフラストラクチャを構築する</h2>
<p>Terraform がインストールされたら、インフラストラクチャを作成できます。</p>
<h3>構成</h3>
<p>インフラストラクチャを説明するために Terraform で使用される一連のファイルは、<code>Terraform 構成</code> と呼ばれています。さっそく最初の構成を記述して、VM インスタンスを 1 つ起動してみましょう。</p>
<p>構成ファイルの形式については<a href="https://www.terraform.io/docs/configuration/index.html">こちらのドキュメント</a>をご覧ください。構成ファイルの作成には JSON を使用することをおすすめします。</p>
<p><code>vim</code>、<code>nano</code> などのエディタを使用して、<code>instance.tf</code> という構成ファイルを作成します。</p>
<pre><code>nano instance.tf&#x000A;</code></pre>
<p>ファイルに次の内容を追加します。<code>&lt;PROJECT_ID&gt;</code> は GCP プロジェクト ID に置き換えてください。</p>
<pre><code>resource "google_compute_instance" "default" {&#x000A;  project      = "&lt;PROJECT_ID&gt;"&#x000A;  name         = "terraform"&#x000A;  machine_type = "n1-standard-1"&#x000A;  zone         = "us-central1-a"&#x000A;&#x000A;  boot_disk {&#x000A;    initialize_params {&#x000A;      image = "debian-cloud/debian-9"&#x000A;    }&#x000A;  }&#x000A;&#x000A;  network_interface {&#x000A;    network = "default"&#x000A;    access_config {&#x000A;    }&#x000A;  }&#x000A;}&#x000A;</code></pre>
<p><strong>Ctrl+X</strong> キーを押してファイルを保存します。</p>
<p>これは完全な構成なので、このまま Terraform で適用できます。全体的な構造は直感的で単純です。</p>
<p><code>instance.tf</code> ファイルの「resource」ブロックは、インフラストラクチャ内に存在するリソースを定義します。リソースは、VM インスタンスなどの物理コンポーネントである場合もあります。</p>
<p>resource ブロックの前に、<strong>リソースタイプ</strong>と<strong>リソース名</strong>の 2 つの文字列があります。このラボでは、リソースタイプは <code>google_compute_instance</code> で、リソース名は <code>default</code> です。タイプの接頭辞はプロバイダに対応しているため、<code>google_compute_instance</code> と入力すると、<code>Google</code> プロバイダが管理するリソースであることを Terraform が自動的に認識します。</p>
<p>resource ブロック自体は、リソースの記載に必要な構成になっています。</p>
<p>次のコマンドを実行して、この新しいファイルが追加されたことと、ディレクトリにほかの <code>*.tf</code> ファイルが含まれないことを確認します。Terraform はすべての *.tf ファイルを読み込むからです。</p>
<pre><code>ls&#x000A;</code></pre>
<h3>初期化</h3>
<p>新しい構成に対して（または既存の構成をバージョン管理からチェックアウトした後に）最初に実行するコマンドは、<code>terraform init</code> です。このコマンドを実行すると、それ以降のコマンドで使用されるさまざまなローカル設定やローカル データが初期化されます。</p>
<p>Terraform にはプラグイン ベースのアーキテクチャが使用されているので、さまざまなインフラストラクチャ プロバイダやサービス プロバイダに対応できます。「プロバイダ」はそれぞれ独自のバイナリにカプセル化されて、Terraform とは別に配布されます。<code>terraform init</code> コマンドを実行すると、プロバイダ（この場合は Google プロバイダのみ）のすべてのプロバイダ バイナリが自動的にダウンロード、インストールされ、構成で使用できるようになります。</p>
<pre><code>terraform init&#x000A;</code></pre>
<p>Google プロバイダのプラグインが、他のさまざまな簿記関連のファイルとともにダウンロードされて、現在の作業ディレクトリのサブディレクトリにインストールされます。「Initializing provider plugins」というメッセージが表示されます。Google プロジェクトから実行していることが認識されて、Google のリソースが取得されます。</p>
<pre><code>Downloading plugin for provider "google" (2.12.0)...&#x000A;</code></pre>
<p>インストールされるプラグインのバージョンが出力に示されます。次回も同じバージョンの構成ファイルを指定すると、<code>terraform init</code> に互換性のあるバージョンがインストールされるという内容のメッセージも表示されます。</p>
<p><code>terraform plan</code> コマンドを使用して実行プランを作成します。このコマンドを実行すると、明示的に無効にしない限り、実行プランが更新されます。その後、構成ファイルで指定した、インフラストラクチャを希望の状態にするのに必要な操作が確定します。</p>
<pre><code>terraform plan&#x000A;</code></pre>
<p>このコマンドは、一連の変更を加えた実行プランで予想どおりの状態を実現できるかを、実際のリソースや状態に変更を加えずに確認するのに便利です。たとえば、変更をバージョン管理に commit する前に terraform plan を実行すると、予想どおりに実行プランが機能することを事前に確認できます。</p>
<aside>
<b>注:</b> オプションの <code>-out</code> 引数を使用すると、生成されたプランをファイルに保存できます。保存したファイルは、後で <code>terraform apply</code> で実行できます。

</aside>
<h3>変更を適用</h3>
<p>作成した <code>instance.tf</code> ファイルと同じディレクトリで、<code>terraform apply</code> を実行します。</p>
<pre><code>terraform apply&#x000A;</code></pre>
<p>出力に実行プランが表示されます。実行プランには、実際のインフラストラクチャをこの構成に合わせて変更するために実行される操作が記述されています。出力の形式は、Git などのツールで生成される差分形式に似ています。</p>
<p><code>google_compute_instance.terraform</code> の横に <code>google_compute_instance.default</code> があります。これは、Terraform でこのリソースが作成されることを意味しています。その下には、設定される属性が並んでいます。値が <code>&lt;computed&gt;</code> になっている場合、その属性の値はリソースが作成されるまでわかりません。</p>
<p><strong>出力例:</strong></p>
<pre><code>An execution plan has been generated and is shown below.&#x000A;Resource actions are indicated with the following symbols:&#x000A;  + create&#x000A;&#x000A;Terraform will perform the following actions:&#x000A;&#x000A;  # google_compute_instance.default will be created&#x000A;  + resource "google_compute_instance" "default" {&#x000A;      + can_ip_forward       = false&#x000A;      + cpu_platform         = (known after apply)&#x000A;      + deletion_protection  = false&#x000A;      + guest_accelerator    = (known after apply)&#x000A;      + id                   = (known after apply)&#x000A;      + instance_id          = (known after apply)&#x000A;      + label_fingerprint    = (known after apply)&#x000A;      + machine_type         = "n1-standard-1"&#x000A;      + metadata_fingerprint = (known after apply)&#x000A;      + name                 = "terraform"&#x000A;      + project              = "qwiklabs-gcp-42390cc9da8a4c4b"&#x000A;      + self_link            = (known after apply)&#x000A;      + tags_fingerprint     = (known after apply)&#x000A;      + zone                 = "us-central1-a"&#x000A;&#x000A;      + boot_disk {&#x000A;          + auto_delete                = true&#x000A;          + device_name                = (known after apply)&#x000A;          + disk_encryption_key_sha256 = (known after apply)&#x000A;          + kms_key_self_link          = (known after apply)&#x000A;          + source                     = (known after apply)&#x000A;&#x000A;          + initialize_params {&#x000A;              + image  = "debian-cloud/debian-9"&#x000A;              + labels = (known after apply)&#x000A;              + size   = (known after apply)&#x000A;              + type   = (known after apply)&#x000A;            }&#x000A;        }&#x000A;&#x000A;      + network_interface {&#x000A;          + address            = (known after apply)&#x000A;          + name               = (known after apply)&#x000A;          + network            = "default"&#x000A;          + network_ip         = (known after apply)&#x000A;          + subnetwork         = (known after apply)&#x000A;          + subnetwork_project = (known after apply)&#x000A;&#x000A;          + access_config {&#x000A;              + assigned_nat_ip = (known after apply)&#x000A;              + nat_ip          = (known after apply)&#x000A;              + network_tier    = (known after apply)&#x000A;            }&#x000A;        }&#x000A;&#x000A;      + scheduling {&#x000A;          + automatic_restart   = (known after apply)&#x000A;          + on_host_maintenance = (known after apply)&#x000A;          + preemptible         = (known after apply)&#x000A;&#x000A;          + node_affinities {&#x000A;              + key      = (known after apply)&#x000A;              + operator = (known after apply)&#x000A;              + values   = (known after apply)&#x000A;            }&#x000A;        }&#x000A;    }&#x000A;&#x000A;Plan: 1 to add, 0 to change, 0 to destroy.&#x000A;&#x000A;Do you want to perform these actions?&#x000A;  Terraform will perform the actions described above.&#x000A;  Only 'yes' will be accepted to approve.&#x000A;&#x000A;  Enter a value:&#x000A;</code></pre>
<p>プランが正常に作成された場合、Terraform はここで一時停止して、続行する前に承認を求めます。本番環境で、実行プランに正しくない内容や危険な内容が含まれていると思われる場合は、安全のためにここで中止してください。この時点で、インフラストラクチャはまだ変更されていません。</p>
<p>この例ではプランに問題はないようなので、確認プロンプトで「<code>yes</code>」と入力して続行します。</p>
<p>プランの実行には数分かかります。VM インスタンスが利用可能になるまで Terraform が待機するためです。</p>
<p>以上で Terraform の作業は終了です。</p>

<p>Cloud Platform Console で [<strong>Compute Engine</strong>] &gt; [<strong>VM インスタンス</strong>] に移動して、作成された VM インスタンスを確認します。</p>
<p><img alt="cf68a77143674661.png" src="https://cdn.qwiklabs.com/vbSlqad%2BVrl178fWsGAaH%2BmPyCI%2Fdjo0hGtwvn793mg%3D"></p>
<p>Terraform は、<code>terraform.tfstate</code> ファイルにデータを書き込みます。この状態ファイルは非常に重要です。作成されたリソースの ID がトラッキングされるため、Terraform で管理しているリソースを把握できるからです。</p>
<p>現在の状態を確認するには <code>terraform show</code> を使用します。</p>
<pre><code>terraform show&#x000A;</code></pre>
<p><strong>出力例:</strong></p>
<pre><code class="language-bash prettyprint"># google_compute_instance.default:&#x000A;resource "google_compute_instance" "default" {&#x000A;    can_ip_forward       = false&#x000A;    cpu_platform         = "Intel Haswell"&#x000A;    deletion_protection  = false&#x000A;    guest_accelerator    = []&#x000A;    id                   = "terraform"&#x000A;    instance_id          = "3408292216444307052"&#x000A;    label_fingerprint    = "42WmSpB8rSM="&#x000A;    machine_type         = "n1-standard-1"&#x000A;    metadata_fingerprint = "s6I5s2tjfKw="&#x000A;    name                 = "terraform"&#x000A;    project              = "qwiklabs-gcp-42390cc9da8a4c4b"&#x000A;    self_link            = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-42390cc9da8a4c4b/zones/us-central1-a/instances/terraform"&#x000A;    tags_fingerprint     = "42WmSpB8rSM="&#x000A;    zone                 = "us-central1-a"&#x000A;&#x000A;    boot_disk {&#x000A;        auto_delete = true&#x000A;        device_name = "persistent-disk-0"&#x000A;        source      = "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-42390cc9da8a4c4b/zones/us-central1-a/disks/terraform"&#x000A;....&#x000A;</code></pre>
<p>このリソースを作成すると、多くの関連情報が収集されたことがわかります。これらの値を参照すると、追加のリソースや出力を構成できます。</p>
<p>適用された実行プランを確認するには <code>terraform plan</code> コマンドを使用します。</p>
<pre><code>terraform plan&#x000A;</code></pre>
<p>これで完了です。ここでは、Terraform を使用して最初のインフラストラクチャを構築しました。さらに、構成構文、基本的な実行プランの例、状態ファイルを確認しました。</p>

# Terraform を使用した Kubernetes ロードバランサ Service のデプロイ
<h2 id="step2">概要</h2>
<p>Terraform のプロバイダは、上流の API を論理的に抽象化したものです。このラボでは、Kubernetes クラスタを設定して LoadBalancer タイプの NGINX Service をデプロイする方法について説明します。</p>
<h2 id="step3">要件</h2>
<ul>
<li>Kubernetes Service の基本知識</li>
<li>
<code>kubectl</code> CLI の基本知識</li>
</ul>
<h2 id="step4">ラボの内容</h2>
<ul>
<li>
<p>Terraform を使用して Kubernetes クラスタと Service をデプロイする。</p>
</li>
</ul>
<h2 id="step5">設定</h2>
<h4>[ラボを開始] ボタンをクリックする前に</h4>
<p>こちらの手順をお読みください。ラボの時間は制限されており、一時停止することはできません。[ラボを開始] ボタンをクリックするとタイマーが開始され、Cloud リソースを利用できる時間が表示されます。</p>
<p>この Qwiklabs ハンズオンラボでは、シミュレーションやデモ環境ではなく、実際のクラウド環境を使ってご自身でラボのアクティビティを行うことができます。一時的な認証情報が新しく提供されるため、ラボ受講中の Google Cloud Platform へのログインおよびアクセスにはその認証情報を使用してください。</p>
<h4>前提条件</h4>
<p>このラボを完了するには、次のものが必要です。</p>
<ul>
<li>標準的なインターネット ブラウザ（Chrome を推奨）。</li>
<li>ラボを完了するために必要な時間。</li>
</ul>
<p><strong><em>注:</em></strong> すでに個人の GCP アカウントやプロジェクトをお持ちの場合でも、そのアカウントやプロジェクトはラボでは使用しないでください。</p>

<h4>ラボを開始して Console にログインする方法</h4>
<ol>
<li>
<p>[<strong>ラボを開始</strong>] ボタンをクリックします。ラボの料金をお支払いいただく必要がある場合は、表示されるポップアップでお支払い方法を選択してください。
左側のパネルには、このラボで使用する必要がある一時的な認証情報が表示されます。</p>
<p><img alt="Google Console を開く" src="https://cdn.qwiklabs.com/%2FtHp4GI5VSDyTtdqi3qDFtevuY014F88%2BFow%2FadnRgE%3D"></p>
</li>
<li>
<p>ユーザー名をコピーし、[<strong>Google Console を開く</strong>] をクリックします。
ラボでリソースが起動し、別のタブで [<strong>アカウントの選択</strong>] ページが表示されます。</p>
<p><strong>ヒント:</strong> タブをそれぞれ別のウィンドウで開き、並べて表示しておきましょう。<em></em></p>
</li>
<li>
<p>[アカウントの選択] ページで [<strong>別のアカウントを使用</strong>] をクリックします。</p>
<p><img alt="アカウントを選択" src="https://cdn.qwiklabs.com/eQ6xPnPn13GjiJP3RWlHWwiMjhooHxTNvzfg1AL2WPw%3D"></p>
</li>
<li>
<p>[ログイン] ページが開きます。[接続の詳細] パネルでコピーしたユーザー名を貼り付けます。パスワードもコピーして貼り付けます。</p>
<p><strong>重要:</strong> 認証情報は [接続の詳細] パネルに表示されたものを使用してください。<em></em>ご自身の Qwiklabs 認証情報は使用しないでください。請求が発生する事態を避けるため、GCP アカウントをお持ちの場合でもそのアカウントはラボで使用しないでください。</p>
</li>
<li>
<p>以降のページでは次の点にご注意ください。</p>
<ul>
<li>利用規約に同意してください。</li>
<li>復元オプションや 2 要素認証プロセスは設定しないでください（一時的なアカウントであるため）。</li>
<li>無料トライアルには登録しないでください。</li>
</ul>
</li>
</ol>
<p>しばらくすると、このタブで GCP Console が開きます。</p>
<aside>
<b>注:</b> 左上の [Google Cloud Platform] の横にある<b>ナビゲーション メニュー</b>をクリックすると、GCP のプロダクトやサービスのリストが含まれるメニューが表示されます。
<img alt="Cloud Console メニュー" src="https://cdn.qwiklabs.com/9vT7xPlxoNP%2FPsK0J8j0ZPFB4HnnpaIJVCDByaBrSHg%3D">
</aside>

<h3>Google Cloud Shell の有効化</h3>
<p>Google Cloud Shell は、デベロッパー ツールと一緒に読み込まれる仮想マシンです。5 GB の永続ホーム ディレクトリが用意されており、Google Cloud で稼働します。Google Cloud Shell では、コマンドラインで GCP リソースにアクセスできます。</p>
<ol>
<li>
<p>GCP Console の右上のツールバーにある [Cloud Shell をアクティブにする] ボタンをクリックします。</p>
<p><img alt="Cloud Shell アイコン" src="https://cdn.qwiklabs.com/vdY5e%2Fan9ZGXw5a%2FZMb1agpXhRGozsOadHURcR8thAQ%3D"></p>
</li>
<li>
<p>[<strong>続行</strong>] をクリックします。</p>
<p><img alt="cloudshell_continue" src="https://cdn.qwiklabs.com/lr3PBRjWIrJ%2BMQnE8kCkOnRQQVgJnWSg4UWk16f0s%2FA%3D"></p>
</li>
</ol>
<p>環境のプロビジョニングと接続には少し時間がかかります。接続すると、すでに認証されており、プロジェクトは<em> PROJECT_ID </em>に設定されています。例えば:</p>
<p><img alt="Cloud Shell 端末" src="https://cdn.qwiklabs.com/hmMK0W41Txk%2B20bQyuDP9g60vCdBajIS%2B52iI2f4bYk%3D"></p>
<p><strong>gcloud</strong> は Google Cloud Platform のコマンドライン ツールです。このツールは、Cloud Shell にプリインストールされており、タブ補完がサポートされています。</p>
<p>次のコマンドを使用すると、有効なアカウント名を一覧表示できます。</p>
<pre><code>gcloud auth list&#x000A;</code></pre>
<p>出力:</p>
<pre><code class="language-output prettyprint">Credentialed accounts:&#x000A;- &lt;myaccount&gt;@&lt;mydomain&gt;.com (active)&#x000A;	</code></pre>
<p>出力例:</p>
<pre><code class="language-Output prettyprint">Credentialed accounts:&#x000A;- google1623327_student@qwiklabs.net&#x000A;	</code></pre>
<p>次のコマンドを使用すると、プロジェクト ID を一覧表示できます。</p>
<pre><code>gcloud config list project&#x000A;	</code></pre>
<p>出力:</p>
<pre><code class="language-output prettyprint">[core]&#x000A;project = &lt;project_ID&gt;&#x000A;	</code></pre>
<p>出力例:</p>
<pre><code class="language-Output prettyprint">[core]&#x000A;project = qwiklabs-gcp-44776a13dea667a6&#x000A;	</code></pre>
<aside>
<strong>gcloud</strong> について詳しくは、<a href="https://cloud.google.com/sdk/gcloud">gcloud の概要</a>をご確認ください。

	</aside>

<h2 id="step6">Kubernetes Service</h2>
<p>Service とは、クラスタで実行されるポッドのグループです。Service は「安価」であるため、クラスタ内で多くの Service を使用できます。Kubernetes Service を使用すると、マイクロサービス アーキテクチャを効率的に実現できます。</p>
<p>Service は、負荷分散、アプリケーション間のサービス ディスカバリ、アプリケーションをダウンタイムなしでデプロイするための機能など、クラスタ全体で標準化される重要な機能を提供します。</p>
<p>各 Service には、その Service のデータを処理するポッドを定義するラベルクエリが含まれています。このラベルクエリは、1 つ以上のレプリケーション コントローラによって作成されるポッドに頻繁に一致します。デプロイ ソフトウェアで Kubernetes API を使用して Service のラベルクエリを更新することにより、強力なルーティング シナリオを実現できます。</p>
<h2 id="step7">Terraform を使用する理由</h2>
<p>YAML ファイルに記述される Kubernetes リソースはすべて、API 呼び出しにマッピングされた <code>kubectl</code> などの CLI ベースのツールを使用して管理できますが、Terraform によるオーケストレーションにはいくつかのメリットがあります。</p>
<ul>
<li>Kubernetes インフラストラクチャのプロビジョニングとアプリケーションのデプロイに同じ<a href="https://www.terraform.io/docs/configuration/syntax.html" target="_blank">構成言語</a>を使用できます。</li>
<li>
<strong>ずれの検出</strong> - 「terraform plan」を使用すると、適用しようとしている構成と現状との相違をいつでも確認できます。</li>
<li>
<strong>ライフサイクル全体の管理</strong> - Terraform では、最初にリソースを作成するだけでなく、追跡しているリソースを 1 つのコマンドで作成、更新、削除できます。それらのリソースを特定する API を調べる必要はありません。</li>
<li>
<strong>同期フィードバック</strong> - 非同期動作は便利ですが、オペレーション結果（エラー、作成されたリソースの詳細など）の確認がユーザーに委ねられるため、非生産的な場合もあります。たとえば、ロードバランサの IP やホスト名はプロビジョニングが完了するまでわからないため、ロードバランサを参照する DNS レコードを作成できません。</li>
<li>
<strong><a href="https://www.terraform.io/docs/internals/graph.html" target="_blank">関係のグラフ</a></strong> - Terraform ではリソース間の関係が認識されるため、スケジューリングに役立ちます。たとえば、Kubernetes クラスタの Service の作成は、そのクラスタが作成されるまで行われません。</li>
</ul>
<h2 id="step8">サンプルコードのクローンを作成する</h2>
<pre><code class="language-bash prettyprint">gsutil -m cp -r gs://spls/gsp233/* .&#x000A;</code></pre>
<p><code>tf-gke-k8s-service-lb</code> ディレクトリに移動します。</p>
<pre><code class="language-bash prettyprint">cd tf-gke-k8s-service-lb&#x000A;</code></pre>
<h2 id="step9">コードを理解する</h2>
<ol>
<li>
<p><code>main.tf</code> ファイルの内容を確認します。</p>
</li>
</ol>

```tf
...

  variable "region" {
    default = "us-west1"
  }

  variable "location" {
    default = "us-west1-b"
  }

  variable "network_name" {
    default = "tf-gke-k8s"
  }

  provider "google" {
    region = var.region
  }

  resource "google_compute_network" "default" {
    name                    = var.network_name
    auto_create_subnetworks = false
  }

  resource "google_compute_subnetwork" "default" {
    name                     = var.network_name
    ip_cidr_range            = "10.127.0.0/20"
    network                  = google_compute_network.default.self_link
    region                   = var.region
    private_ip_google_access = true
  }

...
```

<ul>
<li>
<code>region</code>、<code>zone</code>、<code>network_name</code> の変数が定義されています。これらは Kubernetes クラスタの作成に使用されます。</li>
<li>Google Cloud プロバイダにより、このプロジェクトのリソースを作成できます。</li>
<li>適切なネットワークとクラスタを作成するためのリソースがいくつか定義されています。</li>
<li>最後に、<code>terraform apply</code> の実行後に表示される出力があります。</li>
</ul>
<ol start="2">
<li>
<p><code>k8s.tf</code> ファイルの内容を確認します。</p>
</li>
</ol>
<pre><code class="language-bash prettyprint">cat k8s.tf&#x000A;</code></pre>
<p>出力例（コピー禁止）:<em></em></p>
<pre><code class="language-output prettyprint">provider "kubernetes" {&#x000A;  version = "~&gt; 1.10.0"&#x000A;  host    = google_container_cluster.default.endpoint&#x000A;  token   = data.google_client_config.current.access_token&#x000A;  client_certificate = base64decode(&#x000A;    google_container_cluster.default.master_auth[0].client_certificate,&#x000A;  )&#x000A;  client_key = base64decode(google_container_cluster.default.master_auth[0].client_key)&#x000A;  cluster_ca_certificate = base64decode(&#x000A;    google_container_cluster.default.master_auth[0].cluster_ca_certificate,&#x000A;  )&#x000A;}&#x000A;&#x000A;resource "kubernetes_namespace" "staging" {&#x000A;  metadata {&#x000A;    name = "staging"&#x000A;  }&#x000A;}&#x000A;&#x000A;resource "google_compute_address" "default" {&#x000A;  name   = var.network_name&#x000A;  region = var.region&#x000A;}&#x000A;&#x000A;resource "kubernetes_service" "nginx" {&#x000A;  metadata {&#x000A;    namespace = kubernetes_namespace.staging.metadata[0].name&#x000A;    name      = "nginx"&#x000A;  }&#x000A;&#x000A;  spec {&#x000A;    selector = {&#x000A;      run = "nginx"&#x000A;    }&#x000A;&#x000A;    session_affinity = "ClientIP"&#x000A;&#x000A;    port {&#x000A;      protocol    = "TCP"&#x000A;      port        = 80&#x000A;      target_port = 80&#x000A;    }&#x000A;&#x000A;    type             = "LoadBalancer"&#x000A;    load_balancer_ip = google_compute_address.default.address&#x000A;  }&#x000A;}&#x000A;&#x000A;resource "kubernetes_replication_controller" "nginx" {&#x000A;  metadata {&#x000A;    name      = "nginx"&#x000A;    namespace = kubernetes_namespace.staging.metadata[0].name&#x000A;&#x000A;    labels = {&#x000A;      run = "nginx"&#x000A;    }&#x000A;  }&#x000A;&#x000A;  spec {&#x000A;    selector = {&#x000A;      run = "nginx"&#x000A;    }&#x000A;&#x000A;    template {&#x000A;      container {&#x000A;        image = "nginx:latest"&#x000A;        name  = "nginx"&#x000A;&#x000A;        resources {&#x000A;          limits {&#x000A;            cpu    = "0.5"&#x000A;            memory = "512Mi"&#x000A;          }&#x000A;&#x000A;          requests {&#x000A;            cpu    = "250m"&#x000A;            memory = "50Mi"&#x000A;          }&#x000A;        }&#x000A;      }&#x000A;    }&#x000A;  }&#x000A;}&#x000A;&#x000A;output "load-balancer-ip" {&#x000A;  value = google_compute_address.default.address&#x000A;}&#x000A;&#x000A;</code></pre>
<ul>
<li>このスクリプトは、Terraform を使用して Kubernetes プロバイダを構成し、Service、名前空間、replication_controller リソースを作成します。</li>
<li>このスクリプトは、<code>nginx</code> Service の IP を出力として返します。</li>
</ul>
<h2 id="step10">terraform.tfvars ファイルを作成する</h2>
<p>サンプル ディレクトリに <code>terraform.tfvars</code> ファイルを作成します。</p>
<pre><code class="language-bash prettyprint">cat &gt; terraform.tfvars &lt;&lt;EOF&#x000A;gke_username = "admin"&#x000A;gke_password = "$(openssl rand -base64 16)"&#x000A;EOF&#x000A;</code></pre>
<h2 id="step11">依存関係の初期化とインストール</h2>
<p><code>terraform init</code> コマンドを使用して、Terraform 構成ファイルを含む作業ディレクトリを初期化します。</p>
<p>このコマンドは、いくつかの初期化ステップを実行して、作業ディレクトリを使用できるように準備します。このコマンドは、いつでも何度でも安全に実行できます。実行すると、作業ディレクトリに構成の変更が反映されます。</p>
<pre><code class="language-bash prettyprint">terraform init&#x000A;</code></pre>
<p>出力例（コピー禁止）:<em></em></p>
<pre><code class="language-output prettyprint">...&#x000A;* provider.google: version = "~&gt; 3.8.0"&#x000A;* provider.kubernetes: version = "~&gt; 1.10.0"&#x000A;&#x000A;Terraform has been successfully initialized!&#x000A;&#x000A;You may now begin working with Terraform. Try running "terraform plan" to see&#x000A;any changes that are required for your infrastructure. All Terraform commands&#x000A;should now work.&#x000A;&#x000A;If you ever set or change modules or backend configuration for Terraform,&#x000A;rerun this command to reinitialize your working directory. If you forget, other&#x000A;commands will detect it and remind you to do so if necessary.&#x000A;</code></pre>
<p><code>terraform apply</code> コマンドは、設定の望ましい状態に到達するために必要な変更を適用するために使用されます。</p>
<pre><code class="language-bash prettyprint">terraform apply&#x000A;</code></pre>
<p>出力例（コピー禁止）:<em></em></p>
<pre><code>Apply complete! Resources: 7 added, 0 changed, 0 destroyed.&#x000A;&#x000A;Outputs:&#x000A;&#x000A;cluster_name = tf-gke-k8s&#x000A;cluster_region = us-west1&#x000A;cluster_zone = us-west1-b&#x000A;load-balancer-ip = 35.233.177.223&#x000A;network = https://www.googleapis.com/compute/beta/projects/qwiklabs-gcp-5438ad3a5e852e4a/global/networks/tf-gke-k8s&#x000A;subnetwork_name = tf-gke-k8s&#x000A;</code></pre>
<h3>Terraform によって作成されたリソースを確認する</h3>
<ul>
<li>Cloud Platform Console で、<strong>ナビゲーション メニュー</strong> &gt; [<strong>Kubernetes Engine</strong>] に移動します。</li>
<li>
<code>tf-gke-k8s</code> クラスタをクリックして構成を確認します。</li>
<li>左側のパネルで [<strong>Services と Ingress</strong>] をクリックして、<code>nginx</code> Service のステータスを確認します。</li>
<li>[<strong>エンドポイント</strong>] の IP アドレスをクリックして、新しいブラウザタブで「Welcome to nginx!」ページを開きます。</li>
</ul>
<p><img alt="nginx_response.png" src="https://cdn.qwiklabs.com/Buc5aDKVT5gyxV9dEn3ginoluom7aCsqZ0WlcVYh7GQ%3D"></p>

# Terraform を使用した HTTPS コンテンツ ベース ロードバランサ
<h2 id="step2">概要</h2>
<p>このラボでは、HTTPS ロードバランサを作成してトラフィックをカスタム URL マップに転送します。この URL マップは、Cloud Storage バケットから提供される静的アセットがある最も近いリージョンにトラフィックを送信します。TLS の鍵と証明書は、<a href="https://www.terraform.io/docs/providers/tls/index.html" target="_blank">TLS プロバイダ</a>を使用して Terraform によって生成されます。</p>
<h2 id="step3">目標</h2>
<ul>
<li>Terraform の負荷分散モジュールについて学習する。</li>
<li>GCP 環境で Terraform を構成する。</li>
<li>グローバル HTTPS コンテンツ ベース ロードバランサを作成する。</li>
</ul>

<h2 id="step5">コンテンツ ベースのロードバランサ</h2>
<p><img alt="app_view.png" src="https://cdn.qwiklabs.com/8BAGcQptix4GVZGtF1UCJxH7rjKvaLjx5y5lgXBCwSk%3D"></p>
<h2 id="step6">Terraform のインストールの確認</h2>
<p>Terraform は、Cloud Shell にあらかじめインストールされています。</p>
<p>その Terraform のバージョンを確認します。</p>
<pre><code class="language-bash prettyprint">terraform version&#x000A;</code></pre>
<p>次のような出力が表示されます（コピーしないでください）。<em></em></p>
<pre><code class="language-output prettyprint">Terraform v0.12.x&#x000A;</code></pre>
<h2 id="step7">サンプル リポジトリのクローンを作成する</h2>
<p>Cloud Shell で、<code>terraform-google-lb-http</code> リポジトリのクローンを作成します。</p>
<pre><code class="language-bash prettyprint">git clone https://github.com/GoogleCloudPlatform/terraform-google-lb-http.git&#x000A;</code></pre>
<p><code>multi-backend-multi-mig-bucket-https-lb</code> ディレクトリに移動します。</p>
<pre><code class="language-bash prettyprint">cd ~/terraform-google-lb-http/examples/multi-backend-multi-mig-bucket-https-lb&#x000A;</code></pre>
<h2 id="step8">Terraform を実行する</h2>
<h3>作業ディレクトリを初期化する</h3>
<p><code>terraform init</code> コマンドを使用して、Terraform 構成ファイルを含む作業ディレクトリを初期化します。このコマンドは、いくつかの初期化ステップを実行して、作業ディレクトリを使用できるように準備します。このコマンドは、いつでも何度でも安全に実行できます。実行すると、作業ディレクトリに構成の変更が反映されます。</p>
<p>次のコマンドを実行します。</p>
<pre><code class="language-bash prettyprint">terraform init&#x000A;</code></pre>
<p>出力例（コピーしないでください）:<em></em></p>
<pre><code class="language-output prettyprint">...&#x000A;Terraform has been successfully initialized!&#x000A;</code></pre>
<h3>実行プランを作成する</h3>
<p><code>terraform plan</code> コマンドを使用して実行プランを作成します。このコマンドを実行すると、明示的に無効にしない限り更新が行われ、構成ファイルに指定されている目的の状態を実現するために必要なアクションが特定されます。一連の変更に対する実行プランが予想どおりになっているかどうかを、実際のリソースや状態に変更を加えることなく確認するのに便利です。たとえば、変更をバージョン管理に commit する前に <code>terraform plan</code> を実行すると、予想どおりに機能するかどうかを事前に確認できます。</p>
<p>次のコマンドを実行します。<code>&lt;PROJECT_ID&gt;</code> は Qwiklabs のプロジェクト ID に置き換えてください。</p>
<pre><code class="language-bash prettyprint">terraform plan -out=tfplan -var 'project=&lt;PROJECT_ID&gt;'&#x000A;</code></pre>
<p>出力例（コピーしないでください）:<em></em></p>
<pre><code class="language-output prettyprint">...&#x000A;Plan: 42 to add, 0 to change, 0 to destroy.&#x000A;</code></pre>
<p>オプションの <code>-out</code> 引数を使用すると、生成されたプランをファイルに保存できます。保存したファイルは、後で <code>terraform apply</code> を使用して実行できます。</p>
<p>現在のディレクトリの内容を表示します。保存した Terraform プラン（<code>tfplan</code>）が表示されます。</p>
<pre><code class="language-bash prettyprint">ls&#x000A;</code></pre>
<p>出力例（コピーしないでください）:<em></em></p>
<pre><code class="language-output prettyprint">diagram.png  gceme.sh.tpl  gcp-logo.svg  main.tf  mig.tf  outputs.tf  README.md  test.sh  tfplan  tls.tf  variables.tf&#x000A;</code></pre>
<h3>変更を適用する</h3>
<p><code>terraform apply</code> コマンドを使用して、構成を目的の状態にするために必要な変更を適用します。terraform plan を実行した場合は、実行プランによって生成された一連の定義済みアクションを適用できます。</p>
<p>以下のコマンドを実行します。</p>
<pre><code class="language-bash prettyprint">terraform apply tfplan&#x000A;</code></pre>
<p>出力例（実際のものとは異なります）:<em></em></p>
<pre><code class="language-output prettyprint">...&#x000A;Apply complete! Resources: 42 added, 0 changed, 0 destroyed.&#x000A;...&#x000A;Outputs:&#x000A;&#x000A;asset-url = https://34.96.112.153/assets/gcp-logo.svg&#x000A;group1_region = us-west1&#x000A;group2_region = us-central1&#x000A;group3_region = us-east1&#x000A;load-balancer-ip = 34.96.112.153&#x000A;</code></pre>
<p>Terraform によって作成されたリソースを確認します。</p>
<ul>
<li>左側のメニューで、[<strong>ネットワーク サービス</strong>] &gt; [<strong>負荷分散</strong>] に移動します。</li>
<li>[バックエンド] 列に緑色のチェックマークが表示されるまで待ちます。</li>
<li>
<strong>ml-bk-ml-mig-bkt-s-lb</strong> ロードバランサをクリックして詳細を確認します。</li>
</ul>
<p><img alt="front-host-conf.png" src="https://cdn.qwiklabs.com/q7xg0NxKUly8mBZvlf4Wj12Q45kiHiN81vO0GDHH9JY%3D"></p>
<p><img alt="backend-conf.png" src="https://cdn.qwiklabs.com/19TRSMp0h5VhqrEI1CWV8nTRejQ5gdetjaYr2XnvP%2Fw%3D"></p>
<p>次のコマンドを実行して外部 URL を取得します。</p>
<pre><code>EXTERNAL_IP=$(terraform output | grep load-balancer-ip | cut -d = -f2 | xargs echo -n)&#x000A;</code></pre>
<pre><code>echo https://${EXTERNAL_IP}&#x000A;</code></pre>
<p>返された <code>EXTERNAL_IP</code> のリンクをクリックして、新しいブラウザタブでロードバランサの URL を開きます。</p>
<ql-infobox>
<strong>注:</strong> ブラウザに想定どおりの出力が表示されない場合は、ロードバランサの詳細パネルが上のスクリーンショットと同じになっていることを確認して、数分待ちます。
</ql-infobox>
<p>GCP のロゴと、現在のリージョンに最も近いグループのインスタンスの詳細が表示されます。</p>
<p><img alt="default-response.png" src="https://cdn.qwiklabs.com/PTIvWseLPEfR9rGTyauAvDQLdwdvvkfUXaUkALsJCw8%3D"></p>
<p>次に、URL の末尾に <code>group1</code>、<code>group2</code>、<code>group3</code> をそれぞれ追加します。</p>
<p>最終的な URL は以下のようになります（<code>EXTERNAL_IP</code> は作成したロードバランサの IP に置き換えてください）。</p>
<p><code>https://EXTERNAL_IP/group1</code></p>
<ul>
<li>
<code>group1</code> の場合: GCP のロゴと、<code>us-west1</code> にあるグループのインスタンスの詳細が表示されます。</li>
</ul>
<p>出力例:<em></em></p>
<p><img alt="group1-response.png" src="https://cdn.qwiklabs.com/Z%2FjoGtYpU7gfYME1OelQk%2BuFif1gl93fzxcqsUBi%2Fyg%3D"></p>
<p><code>https://EXTERNAL_IP/group2</code></p>
<ul>
<li>
<code>group2</code> の場合: GCP のロゴと、<code>us-central1</code> にあるグループのインスタンスの詳細が表示されます。</li>
</ul>
<p>出力例:<em></em></p>
<p><img alt="group2-response.png" src="https://cdn.qwiklabs.com/9ZuYR16XnKCgyT1ZwslgqtUPqgMpGlnexG6tIZHVJ2s%3D"></p>
<p><code>https://EXTERNAL_IP/group3</code></p>
<ul>
<li>
<code>group3</code> の場合: GCP のロゴと、<code>us-east1</code> にあるグループのインスタンスの詳細が表示されます。</li>
</ul>
<p>出力例:<em></em></p>
<p><img alt="group3-response.png" src="https://cdn.qwiklabs.com/c%2Bc1%2Fga2u6U5SMFHZ%2BoR20axw8XuVFSpdyoBpXBanDA%3D"></p>

# Terraform を使用したモジュール式負荷分散 - リージョン ロードバランサ
<h2 id="step2">概要</h2>
<p>Google Cloud Platform（GCP）の負荷分散では、他のクラウド プロバイダとは違って、ルーティング インスタンスではなく転送ルールを使用します。これらの転送ルールがバックエンド サービス、ターゲット プール、URL マップ、ターゲット プロキシと組み合わされて、複数のリージョンおよびインスタンス グループにまたがって機能するロードバランサが構成されます。</p>
<p><a href="https://www.terraform.io/">Terraform</a> はオープンソースのインフラストラクチャ管理ツールで、モジュールを使用して簡単にロードバランサを GCP にプロビジョニングできます。</p>
<h2 id="step3">目標</h2>
<ul>
<li>Terraform の負荷分散モジュールについて学習する。</li>
<li>リージョン TCP ロードバランサを作成する。</li>
<li>リージョン内部 TCP ロードバランサを作成する。</li>
<li>Kubernetes Engine を使用してグローバル HTTP ロードバランサを作成する。</li>
<li>グローバル HTTPS コンテンツ ベース ロードバランサを作成する。</li>
</ul>

<h2 id="step5">Terraform モジュールの概要</h2>
<p>このラボで使用するリポジトリには、いくつかのロードバランサ モジュールが含まれています。まずそれらのモジュールについて確認してから、リポジトリのクローンを作成してそれらを使用します。</p>
<h3>terraform-google-lb（リージョン転送ルール）</h3>
<p>このモジュールは、マネージド インスタンス グループのリージョン負荷分散用の <a href="https://cloud.google.com/compute/docs/load-balancing/network/example">TCP ネットワーク ロードバランサ</a>を作成します。マネージド インスタンス グループへの参照を指定すると、そのマネージド インスタンス グループがターゲット プールに追加され、ターゲット プールの正常なインスタンスにトラフィックを転送するためのリージョン転送ルールが作成されます。</p>
<p><img alt="terraform-google-lb-diagram.png" src="https://cdn.qwiklabs.com/vosb%2FYcbcF%2BIB%2FbUJoMKlsN87f%2BtZ%2FGU7fR60tdzHVM%3D"></p>
<p><strong>スニペットの例:</strong></p>
<pre><code>module "gce-lb-fr" {&#x000A;  source       = "github.com/GoogleCloudPlatform/terraform-google-lb"&#x000A;  region       = "${var.region}"&#x000A;  name         = "group1-lb"&#x000A;  service_port = "${module.mig1.service_port}"&#x000A;  target_tags  = ["${module.mig1.target_tags}"]&#x000A;}&#x000A;</code></pre>
<h3>terraform-google-lb-internal（リージョン内部転送ルール）</h3>
<p>このモジュールは、内部リソースのリージョン負荷分散用の<a href="https://cloud.google.com/compute/docs/load-balancing/internal/">内部ロードバランサ</a>を作成します。マネージド インスタンス グループへの参照を指定すると、そのマネージド インスタンス グループがリージョン <a href="https://cloud.google.com/compute/docs/load-balancing/internal/#backend-service">バックエンド サービス</a>に追加され、正常なインスタンスにトラフィックを転送するための内部転送ルールが作成されます。</p>
<p><img alt="terraform-google-lb-internal-diagram.png" src="https://cdn.qwiklabs.com/%2BHUnHV2BgVhK0JbEFbqywqXaW2BBDG2KjVOrJCkqYX4%3D"></p>
<p><strong>スニペットの例:</strong></p>
<pre><code>module "gce-ilb" {&#x000A;  source         = "github.com/GoogleCloudPlatform/terraform-google-lb-internal"&#x000A;  region         = "${var.region}"&#x000A;  name           = "group2-ilb"&#x000A;  ports          = ["${module.mig2.service_port}"]&#x000A;  health_port    = "${module.mig2.service_port}"&#x000A;  source_tags    = ["${module.mig1.target_tags}"]&#x000A;  target_tags    = ["${module.mig2.target_tags}","${module.mig3.target_tags}"]&#x000A;  backends       = [&#x000A;    { group = "${module.mig2.instance_group}" },&#x000A;    { group = "${module.mig3.instance_group}" },&#x000A;  ]&#x000A;}&#x000A;</code></pre>
<h3>terraform-google-lb-http（グローバル HTTP(S) 転送ルール）</h3>
<p>このモジュールは、マルチリージョンのコンテンツ ベース負荷分散用の<a href="https://cloud.google.com/compute/docs/load-balancing/http/">グローバル HTTP ロードバランサ</a>を作成します。マネージド インスタンス グループへの参照と SSL 終端の証明書（オプション）を指定すると、HTTP パスに基づいて正常なインスタンスにトラフィックを転送するための <a href="https://cloud.google.com/compute/docs/load-balancing/http/backend-service">http バックエンド サービス</a>、<a href="https://cloud.google.com/compute/docs/load-balancing/http/url-map">URL マップ</a>、<a href="https://cloud.google.com/compute/docs/load-balancing/http/target-proxies">HTTP(S) ターゲット プロキシ</a>、<a href="https://cloud.google.com/compute/docs/load-balancing/http/global-forwarding-rules">グローバル http 転送ルール</a>が作成されます。</p>
<p><img alt="terraform-google-lb-http-diagram.png" src="https://cdn.qwiklabs.com/SVuyORdqRZ1dPr1nApd8xit7EeMvLp3%2Fza0FBKG4ycw%3D"></p>
<p><strong>スニペットの例:</strong></p>
<pre><code>module "gce-lb-http" {&#x000A;  source            = "github.com/GoogleCloudPlatform/terraform-google-lb-http"&#x000A;  name              = "group-http-lb"&#x000A;  target_tags       = ["${module.mig1.target_tags}", "${module.mig2.target_tags}"]&#x000A;  backends          = {&#x000A;    "0" = [&#x000A;      { group = "${module.mig1.instance_group}" },&#x000A;      { group = "${module.mig2.instance_group}" }&#x000A;    ],&#x000A;  }&#x000A;  backend_params    = [&#x000A;    # ヘルスチェック パス, ポート名, ポート番号, タイムアウト（秒）&#x000A;    "/,http,80,10"&#x000A;  ]&#x000A;}&#x000A;</code></pre>
<p>それでは作業を始めましょう。</p>
<h2 id="step6">Terraform をインストールする</h2>
<p>このラボでは、Terraform バージョン <code>v0.11.x</code> が必要です。Cloud Shell には Terraform <code>v0.12.x</code> がプリインストールされているので、次の手順で適切なバージョンに切り替えます。</p>
<p>次のコマンドで Terraform のバージョンを確認します。</p>
<pre><code>terraform version&#x000A;</code></pre>
<p>出力は次のようになります。</p>
<pre><code>Terraform v0.12.x&#x000A;</code></pre>
<p>次に、Terraform のバージョン管理ツールを使用して目的のバージョンに切り替えます。</p>
<p><code>tfswitch</code> をインストールして設定します。</p>
<pre><code>wget https://github.com/warrensbox/terraform-switcher/releases/download/0.7.737/terraform-switcher_0.7.737_linux_amd64.tar.gz&#x000A;mkdir -p ${HOME}/bin&#x000A;tar -xvf terraform-switcher_0.7.737_linux_amd64.tar.gz -C ${HOME}/bin&#x000A;export PATH=$PATH:${HOME}/bin&#x000A;tfswitch -b ${HOME}/bin/terraform 0.11.14&#x000A;echo "0.11.14" &gt;&gt; .tfswitchrc&#x000A;exit&#x000A;</code></pre>
<p>上記のコマンドを実行すると Cloud Shell が終了します。もう一度 Cloud Shell を開いて、次のコマンドで Terraform のバージョンを確認します。</p>
<pre><code>terraform version&#x000A;</code></pre>
<p>出力は次のようになります。</p>
<pre><code>Terraform v0.11.14&#x000A;</code></pre>

<h2 id="step7">サンプル リポジトリのクローンを作成する</h2>
<p><a href="https://github.com/GoogleCloudPlatform/terraform-google-lb">terraform-google-examples</a> リポジトリのクローンを作成します。</p>
<pre><code>git clone https://github.com/GoogleCloudPlatform/terraform-google-lb&#x000A;cd ~/terraform-google-lb/&#x000A;git checkout 028dd3dc09ae1bf47e107ab435310c0a57b1674c&#x000A;cd ~/terraform-google-lb/examples/basic&#x000A;</code></pre>
<p>Terraform Google プロバイダのバージョン文字列を更新します。<code>main.tf</code> ファイルにバージョン文字列を追加します（Cloud Shell にインストールされている <code>nano</code>、<code>vim</code> などの shell エディタを使用するか、統合コードエディタを使用します。コードエディタを起動するには鉛筆アイコンをクリックします）。</p>
<pre><code>version = "1.18.0"&#x000A;</code></pre>
<p>更新後の構成は次のようになります。</p>
<pre><code>provider google {&#x000A;  region = "${var.region}"&#x000A;  version = "1.18.0"&#x000A;}&#x000A;</code></pre>
<h2 id="step8">リージョン転送ルールを含む TCP ロードバランサ</h2>
<p>このラボでは、同じリージョンの 2 つのインスタンスを含むマネージド インスタンス グループと、ネットワーク TCP ロードバランサを作成します。</p>
<p><img alt="example-lb-diagram.png" src="https://cdn.qwiklabs.com/PcsgtHDeWwbgBzYPLoxdchDhvkoJzuV45gEgKhOCKLg%3D"></p>
<p>Terraform を実行してアーキテクチャをデプロイします。</p>
<pre><code>export GOOGLE_PROJECT=$(gcloud config get-value project)&#x000A;</code></pre>
<p><code>terraform init</code> コマンドを使用して、Terraform 構成ファイルを含む作業ディレクトリを初期化します。このコマンドは、いくつかの初期化ステップを実行して、作業ディレクトリを使用できるように準備します。このコマンドは、いつでも何度でも安全に実行できます。実行すると、作業ディレクトリに構成の変更が反映されます。</p>
<p>次のコマンドを実行します。</p>
<pre><code>terraform init&#x000A;</code></pre>
<p><strong>出力例:</strong></p>
<pre><code>Initializing modules...&#x000A;- module.mig1&#x000A;- module.gce-lb-fr&#x000A;&#x000A;Initializing provider plugins...&#x000A;&#x000A;The following providers do not have any version constraints in configuration,&#x000A;so the latest version was installed.&#x000A;&#x000A;To prevent automatic upgrades to new major versions that may contain breaking&#x000A;changes, it is recommended to add version = "..." constraints to the&#x000A;corresponding provider blocks in configuration, with the constraint strings&#x000A;suggested below.&#x000A;&#x000A;* provider.google: version = "~&gt; 1.13"&#x000A;* provider.null: version = "~&gt; 1.0"&#x000A;* provider.template: version = "~&gt; 1.0"&#x000A;&#x000A;Terraform has been successfully initialized!&#x000A;...&#x000A;</code></pre>
<p><code>terraform plan</code> コマンドを使用して実行プランを作成します。このコマンドを実行すると、明示的に無効にしない限り更新が行われ、構成ファイルに指定されている目的の状態を実現するために必要なアクションが特定されます。
一連の変更に対する実行プランが予想どおりになっているかどうかを、実際のリソースや状態に変更を加えることなく確認するのに便利です。たとえば、変更をバージョン管理に commit する前に <code>terraform plan</code> を実行すると、予想どおりに機能するかどうかを事前に確認できます。</p>
<p>次のコマンドを実行します。</p>
<pre><code>terraform plan&#x000A;</code></pre>
<p><code>terraform apply</code> コマンドを使用して、構成を目的の状態にするために必要な変更を適用します。terraform plan を実行した場合は、実行プランによって生成された一連の定義済みアクションを適用できます。</p>
<pre><code>terraform apply&#x000A;</code></pre>
<p>「<strong>yes</strong>」と入力して続行します。</p>
<p><strong>出力例:</strong></p>
<pre><code>...&#x000A;Apply complete! Resources: 10 added, 0 changed, 0 destroyed.&#x000A;</code></pre>
<p>数分後にインスタンスとロードバランサが使用可能になります。</p>
<p>ロードバランサのステータスは、GCP Console の<strong>ナビゲーション メニュー</strong> &gt; [<strong>ネットワーク サービス</strong>] &gt; [<strong>負荷分散</strong>] で確認できます。</p>
<p>ロードバランサの URL をブラウザで開けるようにするには、次のコマンドを実行します。</p>
<pre><code>EXTERNAL_IP=$(terraform output | grep load-balancer-ip | cut -d = -f2 | xargs echo -n)&#x000A;</code></pre>
<pre><code>echo "http://${EXTERNAL_IP}"&#x000A;</code></pre>
<p>出力の <code>http://${EXTERNAL_IP}</code> というアドレスをクリックすると、ロードバランサへのリンクが開きます。</p>
<p>画面を何度か更新して、us-central1 リージョンの 2 つのインスタンスの間でトラフィックが分散されていることを確認してください。</p>

# Terraform のカスタム プロバイダ
<h2 id="step2">概要</h2>
<p>Terraform のプロバイダは、上流の API を論理的に抽象化したものです。このラボでは、Terraform のカスタム プロバイダを作成する方法について説明します。Terraform はプラグイン モデルをサポートしており、プロバイダはすべてプラグインです。プラグインは Go バイナリとして配布されます。技術的には他の言語で記述することも可能ですが、ほぼすべての Terraform プラグインが <a href="https://golang.org/">Go</a> で記述されています。</p>
<h2 id="step3">前提条件</h2>
<ul>
<li>
<p><code>nano</code>、<code>vi</code> などの Linux エディタの基本知識。</p>
</li>
<li>
<p><code>Go</code> プログラミング言語の基本知識。</p>
</li>
</ul>
<h2 id="step4">カスタム プロバイダの要件</h2>
<p>Terraform のカスタム プロバイダを作成するのは、たとえば次のような場合です。</p>
<ul>
<li>
<p>社内プライベート クラウドで使用する機能が独自仕様の場合、またはオープンソース コミュニティには役立たない場合。</p>
</li>
<li>
<p>「開発中」のプロバイダで公開前にローカルテストを行う場合。</p>
</li>
<li>
<p>既存のプロバイダを拡張する場合。</p>
</li>
</ul>

<h2 id="step6">Terraform をインストールする</h2>
<p>このラボでは、Terraform バージョン <code>v0.11.x</code> が必要です。Cloud Shell には Terraform <code>v0.12.x</code> がプリインストールされているので、次の手順で適切なバージョンに切り替えます。</p>
<p>次のコマンドで Terraform のバージョンを確認します。</p>
<pre><code>terraform version&#x000A;</code></pre>
<p>出力は次のようになります。</p>
<pre><code>Terraform v0.12.x&#x000A;</code></pre>
<p>次に、Terraform のバージョン管理ツールを使用して目的のバージョンに切り替えます。</p>
<p><code>tfswitch</code> をインストールして設定します。</p>
<pre><code>wget https://github.com/warrensbox/terraform-switcher/releases/download/0.7.737/terraform-switcher_0.7.737_linux_amd64.tar.gz&#x000A;mkdir -p ${HOME}/bin&#x000A;tar -xvf terraform-switcher_0.7.737_linux_amd64.tar.gz -C ${HOME}/bin&#x000A;export PATH=$PATH:${HOME}/bin&#x000A;tfswitch -b ${HOME}/bin/terraform 0.11.14&#x000A;echo "0.11.14" &gt;&gt; .tfswitchrc&#x000A;exit&#x000A;</code></pre>
<p>上記のコマンドを実行すると Cloud Shell が終了します。もう一度 Cloud Shell を開いて、次のコマンドで Terraform のバージョンを確認します。</p>
<pre><code>terraform version&#x000A;</code></pre>
<p>出力は次のようになります。</p>
<pre><code>Terraform v0.11.14&#x000A;</code></pre>

<h2 id="step7">プロバイダ スキーマ</h2>
<p><code>provider.go</code> という名前のファイルを作成します。これはプロバイダのルートです。</p>
<p>次に、そのファイルに以下のコードを追加します。</p>
<pre><code>package main&#x000A;&#x000A;import (&#x000A;        "github.com/hashicorp/terraform/helper/schema"&#x000A;)&#x000A;&#x000A;func Provider() *schema.Provider {&#x000A;        return &amp;schema.Provider{&#x000A;                ResourcesMap: map[string]*schema.Resource{},&#x000A;        }&#x000A;}&#x000A;</code></pre>
<p><a href="https://godoc.org/github.com/hashicorp/terraform/helper/schema">helper/schema</a> ライブラリは <a href="https://www.terraform.io/docs/extend/how-terraform-works.html#terraform-core">Terraform Core</a> の一部です。複雑な処理の多くを抽象化して、プロバイダ間の一貫性を確保します。上の例では空のプロバイダを定義しています（リソースがありません）。</p>
<p><code>*schema.Provider</code> 型はプロバイダのプロパティを記述します。以下に例を示します。</p>
<ul>
<li>
<p>プロバイダが受け取る構成キー</p>
</li>
<li>
<p>プロバイダがサポートするリソース</p>
</li>
<li>
<p>構成するコールバック</p>
</li>
</ul>
<h2 id="step8">プラグインのビルド</h2>
<p>Go では <code>main.go</code> ファイルが必要です。このファイルは、バイナリがビルドされるとデフォルトの実行可能ファイルになります。</p>
<p>Terraform プラグインは Go バイナリとして配布されるため、このエントリ ポイントを定義することが重要です。そのためには次のコードをファイルに追加します。</p>
<pre><code>package main&#x000A;&#x000A;import (&#x000A;        "github.com/hashicorp/terraform/plugin"&#x000A;        "github.com/hashicorp/terraform/terraform"&#x000A;)&#x000A;&#x000A;func main() {&#x000A;        plugin.Serve(&amp;plugin.ServeOpts{&#x000A;                ProviderFunc: func() terraform.ResourceProvider {&#x000A;                        return Provider()&#x000A;                },&#x000A;        })&#x000A;}&#x000A;</code></pre>
<p>これにより、Go の有効な実行可能バイナリを生成する main 関数が設定されます。この main 関数の内部では、Terraform の plugin ライブラリを使用しています。このライブラリは、Terraform Core とプラグインのすべての通信を処理します。</p>
<p>Terraform のパッケージと依存関係をダウンロードしてインストールします。</p>
<pre><code>go get github.com/hashicorp/terraform/helper/schema&#x000A;go get github.com/hashicorp/terraform/plugin&#x000A;go get github.com/hashicorp/terraform/terraform&#x000A;</code></pre>
<p>次に、Go ツールチェーンを使用してプラグインをビルドします。</p>
<pre><code>go build -o terraform-provider-example&#x000A;</code></pre>
<p>出力名（-o）は<strong>非常に重要</strong>です。Terraform は次の形式でプラグインを検索します。</p>
<pre><code>terraform-&lt;タイプ&gt;-&lt;名前&gt;&#x000A;</code></pre>
<p>上の例では、プラグインのタイプは <strong>provider</strong> で、名前は <strong>example</strong> です。</p>
<p>ディレクトリのコンテンツを一覧表示します。</p>
<pre><code>ls&#x000A;</code></pre>
<p>正しく機能することを確認するために、作成したバイナリを実行します。</p>
<pre><code>./terraform-provider-example&#x000A;</code></pre>
<p><strong>出力例:</strong></p>
<pre><code>This binary is a plugin. These are not meant to be executed directly.&#x000A;Please execute the program that consumes these plugins, which will&#x000A;load any plugins automatically&#x000A;</code></pre>
<p>ファイルツリーは次のようになります。</p>
<p><img alt="basic_tree.png" src="https://cdn.qwiklabs.com/JgmIcz%2BSSsCOAbxN7u1VQjxNwE113HHfvZFA%2FGujBsw%3D"></p>
<h2 id="step9">リソースの定義</h2>
<p>Terraform プロバイダはリソースを管理します。「プロバイダ」<em><em></em></em>は上流の API を抽象化したもので、「リソース」<em><em></em></em>はそのプロバイダのコンポーネントです。たとえば、Google プロバイダは <code>google_compute_instance</code> と <code>google_compute_address</code> をサポートしています。</p>
<p>Terraform プロバイダでは、リソースをそれぞれ固有のファイルに配置して、リソース名の先頭に <code>resource_</code> を追加した名前を付けるのが慣例になっています。</p>
<p>たとえば、example_server というリソースを作成する場合、慣例では <code>resource_server.go</code> というファイル名になります。このファイルを作成して、次の内容を追加します。</p>
<pre><code>package main&#x000A;&#x000A;import (&#x000A;        "github.com/hashicorp/terraform/helper/schema"&#x000A;)&#x000A;&#x000A;func resourceServer() *schema.Resource {&#x000A;        return &amp;schema.Resource{&#x000A;                Create: resourceServerCreate,&#x000A;                Read:   resourceServerRead,&#x000A;                Update: resourceServerUpdate,&#x000A;                Delete: resourceServerDelete,&#x000A;&#x000A;                Schema: map[string]*schema.Schema{&#x000A;                        "address": &amp;schema.Schema{&#x000A;                                Type:     schema.TypeString,&#x000A;                                Required: true,&#x000A;                        },&#x000A;                },&#x000A;        }&#x000A;}&#x000A;</code></pre>
<p>ここでは <a href="https://godoc.org/github.com/hashicorp/terraform/helper/schema#Resource"><code>schema.Resource</code> 型</a>を使用しています。この構造体は、リソースのデータスキーマと CRUD オペレーションを定義します。リソースを作成するために必要な作業は、これらのプロパティを定義することだけです。</p>
<p>上のスキーマでは <strong>address</strong> という要素を定義しています。これは必須の文字列です。Terraform のスキーマでは、検証と型キャストが自動的に適用されます。</p>
<p>さらに 4 つの<strong>フィールド</strong>が定義されています。<code>Create</code>、<code>Read</code>、<code>Update</code>、<code>Delete</code> の 4 つです。Create、Read、Delete の 3 つの関数は、リソースが機能するために必要です。関数は他にもありますが、必須の関数はこれだけです。どの関数をどのデータで呼び出すかは Terraform によって処理されます。リソースのスキーマと現在の状態に基づいて、新しいリソースを作成するか、既存のリソースを更新または破棄するかが特定されます。</p>
<p>この 4 つの構造体フィールドは、それぞれ 1 つの関数を参照しています。すべての関数をリソース スキーマにインラインで記述することも技術的には可能ですが、それぞれ独自のメソッドに配置することをおすすめします。その方が、テストの面でも読みやすさの面でも有利です。次に、これらのスタブに必要なコードを追加します。メソッド シグネチャに注意してください。</p>
<p><code>resource_server.go</code> ファイルを更新して、ファイルの末尾に次の内容を追加します。</p>
<pre><code>func resourceServerCreate(d *schema.ResourceData, m interface{}) error {&#x000A;        return nil&#x000A;}&#x000A;&#x000A;func resourceServerRead(d *schema.ResourceData, m interface{}) error {&#x000A;        return nil&#x000A;}&#x000A;&#x000A;func resourceServerUpdate(d *schema.ResourceData, m interface{}) error {&#x000A;        return nil&#x000A;}&#x000A;&#x000A;func resourceServerDelete(d *schema.ResourceData, m interface{}) error {&#x000A;        return nil&#x000A;}&#x000A;</code></pre>
<p>最後に、<code>provider.go</code> のプロバイダ スキーマを更新して、新しい example_server リソースを登録します。</p>
<pre><code>func Provider() *schema.Provider {&#x000A;        return &amp;schema.Provider{&#x000A;                ResourcesMap: map[string]*schema.Resource{&#x000A;                        "example_server": resourceServer(),&#x000A;                },&#x000A;        }&#x000A;}&#x000A;</code></pre>
<p>プラグインをビルドしてテストします。オペレーションはすべて NoOps ですが、そのままの状態でコンパイルされるはずです。</p>
<pre><code>go build -o terraform-provider-example&#x000A;</code></pre>
<pre><code>./terraform-provider-example&#x000A;</code></pre>
<p><strong>出力例:</strong></p>
<pre><code>This binary is a plugin. These are not meant to be executed directly.&#x000A;Please execute the program that consumes these plugins, which will&#x000A;load any plugins automatically&#x000A;</code></pre>
<p>レイアウトは次のようになります。</p>
<p><img alt="updated_layout.png" src="https://cdn.qwiklabs.com/0pmZPDWBZyO9kCBSjNUNUTL5LNLk%2BFNhpK40Gta9mSw%3D"></p>
<h2 id="step10">プロバイダの呼び出し</h2>
<p>これまでは、シェルから直接プロバイダを実行していました。この場合、次のような警告メッセージが出力されます。</p>
<pre><code>This binary is a plugin. These are not meant to be executed directly.&#x000A;Please execute the program that consumes these plugins, which will&#x000A;load any plugins automatically&#x000A;</code></pre>
<p>Terraform プラグインは、直接 Terraform で実行する必要があります。この方法をテストするには、次の内容を含む <code>main.tf</code> を作業ディレクトリ（プラグインと同じ場所）に作成します。</p>
<pre><code>resource "example_server" "my-server" {}&#x000A;</code></pre>
<p>Terraform は、構成ファイルの解析時にプロバイダを自動的に検出します。構成ファイルが解析されるのは、init コマンドが実行されたときだけです。一致するプロバイダの検索には <a href="https://www.terraform.io/docs/extend/how-terraform-works.html#discovery">Discovery</a> プロセスが使用されます。これには現在のローカル ディレクトリも含まれます。</p>
<p><code>terraform init</code> を実行して、新たにコンパイルされたプロバイダを検出します。</p>
<pre><code>terraform init&#x000A;</code></pre>
<p><strong>出力例:</strong></p>
<pre><code>Initializing provider plugins...&#x000A;&#x000A;Terraform has been successfully initialized!&#x000A;&#x000A;You may now begin working with Terraform. Try running "terraform plan" to see&#x000A;any changes that are required for your infrastructure. All Terraform commands&#x000A;should now work.&#x000A;&#x000A;If you ever set or change modules or backend configuration for Terraform,&#x000A;rerun this command to reinitialize your working directory. If you forget, other&#x000A;commands will detect it and remind you to do so if necessary.&#x000A;</code></pre>
<p>次に、<code>terraform plan</code> を実行します。</p>
<pre><code>terraform plan&#x000A;</code></pre>
<p><strong>出力例:</strong></p>
<pre><code>1 error(s) occurred:&#x000A;&#x000A;* example_server.my-server: "address": required field is not set&#x000A;</code></pre>
<p>これにより、Terraform がプラグインに処理を正常に委任していることが検証されます。検証は想定どおりに行われています。</p>
<p>この検証エラーを修正するには、<code>main.tf</code> リソースに <code>address</code> フィールドを追加します。</p>
<pre><code>resource "example_server" "my-server" {&#x000A;  address = "1.2.3.4"&#x000A;}&#x000A;</code></pre>
<p><code>terraform plan</code> を実行して、検証に合格することを確認します。</p>
<pre><code>terraform plan&#x000A;</code></pre>
<p><strong>出力例:</strong></p>
<pre><code>Refreshing Terraform state in-memory prior to plan...&#x000A;The refreshed state will be used to calculate this plan, but will not be&#x000A;persisted to local or remote state storage.&#x000A;&#x000A;&#x000A;------------------------------------------------------------------------&#x000A;&#x000A;An execution plan has been generated and is shown below.&#x000A;Resource actions are indicated with the following symbols:&#x000A;  + create&#x000A;&#x000A;Terraform will perform the following actions:&#x000A;&#x000A;  + example_server.my-server&#x000A;      id:      &lt;computed&gt;&#x000A;      address: "1.2.3.4"&#x000A;&#x000A;&#x000A;Plan: 1 to add, 0 to change, 0 to destroy.&#x000A;&#x000A;------------------------------------------------------------------------&#x000A;</code></pre>
<p>ここで <code>terraform apply</code> を実行することもできますが、現時点では、すべてのリソース オプションで何も行われないため NoOps になります。</p>
<h2 id="step11">Create の実装</h2>
<p><code>resource_server.go</code> に戻り、次の更新を加えて Create の機能を実装します。</p>
<pre><code>func resourceServerCreate(d *schema.ResourceData, m interface{}) error {&#x000A;        address := d.Get("address").(string)&#x000A;        d.SetId(address)&#x000A;        return nil&#x000A;}&#x000A;</code></pre>
<p>ここでは、<a href="https://godoc.org/github.com/hashicorp/terraform/helper/schema#ResourceData">schema.ResourceData</a> API を使用して、Terraform 構成に指定された <strong>address</strong> の値を取得しています。Go の仕組みに合わせて型を文字列にキャストする必要がありますが、スキーマによって型が文字列であることが保証されているため、安全のためのオペレーションになります。</p>
<p>次に、組み込み関数の <strong>SetId</strong> を使用して、リソースの ID をそのアドレスに設定しています。空白でない ID が存在することで、リソースが作成されたことが Terraform に伝わります。この ID には任意の文字列値を使用できますが、その値を使用してリソースを再読み取りできる必要があります。</p>
<p>最後にバイナリを再コンパイルし、<code>terraform init</code> を再実行して再初期化する必要があります。再初期化が必要になるのは、コードを変更してバイナリを再コンパイルすると、各オペレーションに同じバイナリが使用されていることを確認するための内部ハッシュにバイナリが一致しなくなるためです。</p>
<p>次のコマンドを実行します。</p>
<pre><code>go build -o terraform-provider-example&#x000A;</code></pre>
<pre><code>terraform init&#x000A;</code></pre>
<pre><code>terraform plan&#x000A;</code></pre>
<p><strong>出力例:</strong></p>
<pre><code>+ example_server.my-server&#x000A;    address: "1.2.3.4"&#x000A;&#x000A;&#x000A;Plan: 1 to add, 0 to change, 0 to destroy.&#x000A;</code></pre>
<p><code>terraform apply</code> を実行すると確認を求められます。そこで「<strong>yes</strong>」と入力すると、example_server が作成されて、次の状態に commit されます。</p>
<pre><code>terraform apply&#x000A;</code></pre>
<p><strong>出力例:</strong></p>
<pre><code>An execution plan has been generated and is shown below.&#x000A;Resource actions are indicated with the following symbols:&#x000A;  + create&#x000A;&#x000A;Terraform will perform the following actions:&#x000A;&#x000A;  + example_server.my-server&#x000A;      id:      &lt;computed&gt;&#x000A;      address: "1.2.3.4"&#x000A;&#x000A;&#x000A;Plan: 1 to add, 0 to change, 0 to destroy.&#x000A;&#x000A;Do you want to perform these actions?&#x000A;  Terraform will perform the actions described above.&#x000A;  Only 'yes' will be accepted to approve.&#x000A;&#x000A;  Enter a value: yes&#x000A;&#x000A;example_server.my-server: Creating...&#x000A;  address: "" =&gt; "1.2.3.4"&#x000A;example_server.my-server: Creation complete after 0s (ID: 1.2.3.4)&#x000A;</code></pre>
<p><code>Create</code> オペレーションで <code>SetId</code> が使用されているため、リソースが正常に作成されたと見なされます。<code>terraform plan</code> を実行してこのことを確認します。</p>
<pre><code>terraform plan&#x000A;</code></pre>
<p><strong>出力例:</strong></p>
<pre><code>Refreshing Terraform state in-memory prior to plan...&#x000A;The refreshed state will be used to calculate this plan, but will not be&#x000A;persisted to local or remote state storage.&#x000A;&#x000A;example_server.my-server: Refreshing state... (ID: 1.2.3.4)&#x000A;&#x000A;------------------------------------------------------------------------&#x000A;&#x000A;No changes. Infrastructure is up-to-date.&#x000A;&#x000A;This means that Terraform did not detect any differences between your&#x000A;configuration and real physical resources that exist. As a result, no&#x000A;actions need to be performed.&#x000A;</code></pre>
<p>再び、<code>SetId</code> が呼び出されているため、リソースが作成されたと見なされます。<code>plan</code> を実行すると、適用する変更がないことが正しく認識されます。</p>
<p>この動作を確認するために、address フィールドの値を変更して terraform plan を再実行します。</p>
<pre><code>terraform plan&#x000A;</code></pre>
<p>次のような出力が表示されます。</p>
<p><strong>出力例:</strong></p>
<pre><code>Refreshing Terraform state in-memory prior to plan...&#x000A;The refreshed state will be used to calculate this plan, but will not be&#x000A;persisted to local or remote state storage.&#x000A;&#x000A;example_server.my-server: Refreshing state... (ID: 1.2.3.4)&#x000A;&#x000A;------------------------------------------------------------------------&#x000A;&#x000A;An execution plan has been generated and is shown below.&#x000A;Resource actions are indicated with the following symbols:&#x000A;  ~ update in-place&#x000A;&#x000A;Terraform will perform the following actions:&#x000A;&#x000A;  ~ example_server.my-server&#x000A;      address: "1.2.3.4" =&gt; "5.6.7.8"&#x000A;&#x000A;&#x000A;Plan: 0 to add, 1 to change, 0 to destroy.&#x000A;&#x000A;------------------------------------------------------------------------&#x000A;&#x000A;Note: You didn't specify an "-out" parameter to save this plan, so Terraform&#x000A;can't guarantee that exactly these actions will be performed if&#x000A;"terraform apply" is subsequently run.&#x000A;</code></pre>
<p>変更が検出されて、差分が表示されます。先頭に ~ という接頭辞が付いています。これは、リソースが新たに作成されるのではなくその場で変更されることを示します。</p>
<p>terraform apply を実行して変更を適用します。</p>
<pre><code>terraform apply&#x000A;</code></pre>
<p>再び確認を求められます。</p>
<p><strong>出力例:</strong></p>
<pre><code>example_server.my-server: Refreshing state... (ID: 1.2.3.4)&#x000A;&#x000A;An execution plan has been generated and is shown below.&#x000A;Resource actions are indicated with the following symbols:&#x000A;  ~ update in-place&#x000A;&#x000A;Terraform will perform the following actions:&#x000A;&#x000A;  ~ example_server.my-server&#x000A;      address: "1.2.3.4" =&gt; "5.6.7.8"&#x000A;&#x000A;&#x000A;Plan: 0 to add, 1 to change, 0 to destroy.&#x000A;&#x000A;Do you want to perform these actions?&#x000A;  Terraform will perform the actions described above.&#x000A;  Only 'yes' will be accepted to approve.&#x000A;&#x000A;  Enter a value: yes&#x000A;&#x000A;example_server.my-server: Modifying... (ID: 1.2.3.4)&#x000A;  address: "1.2.3.4" =&gt; "5.6.7.8"&#x000A;example_server.my-server: Modifications complete after 0s (ID: 1.2.3.4)&#x000A;&#x000A;Apply complete! Resources: 0 added, 1 changed, 0 destroyed.&#x000A;</code></pre>
<p>まだ Update 関数を実装していないため、terraform plan を実行すると変更が報告されるように思われますが、変更は報告されません。Update が実装されていないのに変更が保持されるのはなぜでしょうか。</p>
<h2 id="step12">エラー処理と Partial 状態</h2>
<p>前のセクションでは、Update オペレーションが成功し、関数定義が空のまま新しい状態が保持されました。現在 Update 関数は次のようになっています。</p>
<pre><code>func resourceServerUpdate(d *schema.ResourceData, m interface{}) error {&#x000A;        return nil&#x000A;}&#x000A;</code></pre>
<p><code>return nil</code> は、Update オペレーションがエラーなしで成功したことを Terraform に伝えます。この場合、リクエストされた変更がエラーなしで適用されたと見なされます。そのため、状態が更新され、さらなる変更はないと判断されます。</p>
<p>言い換えれば、コールバックがエラーを返さない限り、差分全体が正常に適用されたと見なされ、差分が最終状態にマージされて保持されます。</p>
<p>関数で意図的に <code>panic</code> や <code>os.Exit</code> を呼び出すのは避けて、常にエラーを返すようにしてください。</p>
<p>実際はもう少し複雑です。たとえば、Update 関数で 2 つのフィールドを更新するために 2 つの API 呼び出しを行う必要がある際に、最初の API 呼び出しは成功したが 2 つ目の呼び出しは失敗した場合、どうなるのでしょうか。差分を半分だけ保持するように Terraform に正しく伝えるにはどうすればよいでしょうか。これは Partial 状態と呼ばれるシナリオで、プロバイダが適切に動作するためにはこのシナリオを正しく実装することが重要です。</p>
<p>Terraform における状態更新のルールを以下に示します。</p>
<ul>
<li>
<code>SetId</code> を使用して ID が設定されていない場合は、Create コールバックが返す値にエラーが含まれているかに関係なく、リソースは作成されていないと見なされて、状態は保存されません。</li>
<li>ID が設定されていた場合は、Create コールバックが返す値にエラーが含まれているかに関係なく、リソースが作成されたと見なされて、すべての状態がリソースとともに保存されます。重要なので繰り返しますが、エラーがあっても ID が設定されていれば状態は完全に保存されます。</li>
<li>Update コールバックが返す値にエラーが含まれているかに関係なく、完全な状態が保存されます。ID が空白になる場合は、リソースが破棄されます（この場合は Update でもリソースが破棄されますが、エラーシナリオ以外では起こらないはずです）。</li>
<li>Destroy コールバックが返す値にエラーが含まれていない場合は、リソースが破棄されたと見なされて、すべての状態が削除されます。</li>
<li>Destroy コールバックが返す値にエラーが含まれている場合は、リソースがまだ存在すると見なされて、以前の状態がすべて保持されます。</li>
<li>Create または Update が値を返す際に Partial モード（この後で説明します）が有効になっていた場合は、明示的に有効にされた構成キーのみが保持されて、Partial 状態になります。</li>
</ul>
<p>Update 関数の Partial モードの例を以下に示します。</p>
<pre><code>func resourceServerUpdate(d *schema.ResourceData, m interface{}) error {&#x000A;        // Partial モードを有効にします。&#x000A;        d.Partial(true)&#x000A;&#x000A;        if d.HasChange("address") {&#x000A;                // アドレスを更新してみます。&#x000A;                if err := updateAddress(d, m); err != nil {&#x000A;                        return err&#x000A;                }&#x000A;&#x000A;                d.SetPartial("address")&#x000A;        }&#x000A;&#x000A;        // 以下で Partial モードを無効にする前にここで戻った場合は、&#x000A;        // 「address」フィールドのみが保存されます。&#x000A;&#x000A;        // 成功したので Partial モードを無効にします。これにより、再びすべてのフィールドが&#x000A;        // 保存されるようになります。&#x000A;        d.Partial(false)&#x000A;&#x000A;        return nil&#x000A;}&#x000A;</code></pre>
<p>このコードは、<code>updateAddress</code> 関数が存在しないためコンパイルされません。この関数のダミー バージョンを実装して Partial 状態を試してみることもできますが、この例の Partial 状態にはあまり意味はありません。<code>updateAddress</code> が失敗した場合、address フィールドは更新されません。</p>
<h2 id="step13">Destroy の実装</h2>
<p>Destroy コールバックは、その名のとおり、リソースを破棄するために呼び出されます。このオペレーションでリソースの状態を更新しないでください。戻り値がエラーでない限り、リソースが正常に削除されたと見なされるため、<code>d.SetId("")</code> を呼び出す必要はありません。</p>
<pre><code>func resourceServerDelete(d *schema.ResourceData, m interface{}) error {&#x000A;  // d.SetId("") は、Delete でエラーが返されなければ自動的に呼び出されますが、&#x000A;  // ここではわかりやすくするために追加しています。&#x000A;        d.SetId("")&#x000A;        return nil&#x000A;}&#x000A;</code></pre>
<p>Destroy 関数では、リソースがすでに破棄されているケース（手動で破棄された場合など）に常に対処する必要があります。リソースがすでに破棄されていてもエラーは返しません。これにより、Terraform ユーザーが手動でリソースを削除しても Terraform が正常に動作するようになります。</p>
<p>プロバイダを再コンパイルして再初期化します。</p>
<pre><code>go build -o terraform-provider-example&#x000A;</code></pre>
<pre><code>terraform init&#x000A;</code></pre>
<p><code>terraform destroy</code> を実行してリソースを破棄します。</p>
<pre><code>terraform destroy&#x000A;</code></pre>
<p><strong>出力例:</strong></p>
<pre><code>example_server.my-server: Refreshing state... (ID: 5.6.7.8)&#x000A;&#x000A;An execution plan has been generated and is shown below.&#x000A;Resource actions are indicated with the following symbols:&#x000A;  - destroy&#x000A;&#x000A;Terraform will perform the following actions:&#x000A;&#x000A;  - example_server.my-server&#x000A;&#x000A;&#x000A;Plan: 0 to add, 0 to change, 1 to destroy.&#x000A;&#x000A;Do you really want to destroy?&#x000A;  Terraform will destroy all your managed infrastructure, as shown above.&#x000A;  There is no undo. Only 'yes' will be accepted to confirm.&#x000A;&#x000A;  Enter a value: yes&#x000A;&#x000A;example_server.my-server: Destroying... (ID: 5.6.7.8)&#x000A;example_server.my-server: Destruction complete after 0s&#x000A;&#x000A;Destroy complete! Resources: 1 destroyed.&#x000A;</code></pre>
<h2 id="step14">Read の実装</h2>
<p>Read コールバックは、ローカルの状態を実際の状態（上流）と同期させるために使用され、Terraform によってさまざまな時点で呼び出されます。読み取り専用のオペレーションにする必要があります。このコールバックで実際のリソースを変更しないでください。</p>
<p>ID が更新されて空白になった場合は、リソースがすでに存在しないことが Terraform に伝えられます（帯域外で破棄された場合など）。Destroy コールバックと同様に、このケースに適切に対処する必要があります。</p>
<pre><code>func resourceServerRead(d *schema.ResourceData, m interface{}) error {&#x000A;  client := m.(*MyClient)&#x000A;&#x000A;  // 上流の API からの読み取りを試行します。&#x000A;  obj, ok := client.Get(d.Id())&#x000A;&#x000A;  // リソースが存在しない場合は Terraform に知らせて、処理が続行されないように&#x000A;  // ここで直ちに戻ります。&#x000A;  if !ok {&#x000A;    d.SetId("")&#x000A;    return nil&#x000A;  }&#x000A;&#x000A;  d.Set("address", obj.Address)&#x000A;  return nil&#x000A;}&#x000A;</code></pre>

# Cloud SQL と Terraform
<h2 id="step2">概要</h2>
<p>このハンズオンラボでは、Terraform で Cloud SQL インスタンスを作成し、Cloud SQL Proxy を設定して、MySQL クライアントとの接続をテストする方法について学びます。</p>
<h2 id="step3">目標</h2>
<ul>
<li>Cloud SQL インスタンスを作成する</li>
<li>Cloud SQL Proxy をインストールする</li>
<li>Cloud Shell を使用して MySQL クライアントとの接続をテストする</li>
</ul>

<h2 id="step5">Cloud SQL</h2>
<p>Cloud SQL は、Google Cloud Platform 上のリレーショナル データベースの設定、維持、運用、管理を簡単にできるようにするフルマネージド データベース サービスです。<a href="https://cloud.google.com/sql/docs/mysql/" target="_blank">MySQL</a> または <a href="https://cloud.google.com/sql/docs/postgres/" target="_blank">PostgreSQL</a> で利用できます。</p>
<h3>Terraform のバージョンの確認</h3>
<p>Cloud Shell にあらかじめインストールされている Terraform のバージョンを確認します。</p>
<pre><code class="language-bash prettyprint">terraform version&#x000A;</code></pre>
<p><strong>出力例:</strong></p>
<pre><code class="language-output prettyprint">Terraform v0.12.x&#x000A;</code></pre>
<h2 id="step6">必要なファイルをダウンロードする</h2>
<p>以下のコマンドを使ってディレクトリを作成し、必要な Terraform スクリプトを Cloud Storage バケットから取得します。</p>
<pre><code class="language-bash prettyprint">mkdir sql-with-terraform&#x000A;cd sql-with-terraform&#x000A;gsutil cp -r gs://spls/gsp234/gsp234.zip .&#x000A;</code></pre>
<p>ダウンロードしたコンテンツを展開します。</p>
<pre><code>unzip gsp234.zip&#x000A;</code></pre>
<h2 id="step7">コードを理解する</h2>
<p><code>main.tf</code> ファイルの内容を確認します。</p>
<pre><code class="language-bash prettyprint">cat main.tf&#x000A;</code></pre>
<p><strong>出力例:</strong></p>

```tf
 provider "google" {
  version = "~> 2.13"
}

provider "google-beta" {
  version = "~> 2.13"
}

provider "random" {
  version = "~> 2.2"
}

resource "random_id" "name" {
  byte_length = 2
}

resource "google_sql_database_instance" "master" {
  name                 = "example-mysql-${random_id.name.hex}"
  project              = var.project
  region               = var.region
  database_version     = var.database_version
  master_instance_name = var.master_instance_name

  settings {
    tier                        = var.tier
    activation_policy           = var.activation_policy
    authorized_gae_applications = var.authorized_gae_applications
    disk_autoresize             = var.disk_autoresize
    dynamic "backup_configuration" {
      for_each = [var.backup_configuration]
      content {

        binary_log_enabled = lookup(backup_configuration.value, "binary_log_enabled", null)
        enabled            = lookup(backup_configuration.value, "enabled", null)
        start_time         = lookup(backup_configuration.value, "start_time", null)
      }
    }
    dynamic "ip_configuration" {
      for_each = [var.ip_configuration]
      content {

        ipv4_enabled    = lookup(ip_configuration.value, "ipv4_enabled", true)
        private_network = lookup(ip_configuration.value, "private_network", null)
        require_ssl     = lookup(ip_configuration.value, "require_ssl", null)

        dynamic "authorized_networks" {
          for_each = lookup(ip_configuration.value, "authorized_networks", [])
          content {
            expiration_time = lookup(authorized_networks.value, "expiration_time", null)
            name            = lookup(authorized_networks.value, "name", null)
            value           = lookup(authorized_networks.value, "value", null)
          }
        }
      }
    }
    dynamic "location_preference" {
      for_each = [var.location_preference]
      content {

        follow_gae_application = lookup(location_preference.value, "follow_gae_application", null)
        zone                   = lookup(location_preference.value, "zone", null)
      }
    }
    dynamic "maintenance_window" {
      for_each = [var.maintenance_window]
      content {

        day          = lookup(maintenance_window.value, "day", null)
        hour         = lookup(maintenance_window.value, "hour", null)
        update_track = lookup(maintenance_window.value, "update_track", null)
      }
    }
    disk_size        = var.disk_size
    disk_type        = var.disk_type
    pricing_plan     = var.pricing_plan
    replication_type = var.replication_type
    availability_type = var.availability_type
  }

  dynamic "replica_configuration" {
    for_each = [var.replica_configuration]
    content {

      ca_certificate            = lookup(replica_configuration.value, "ca_certificate", null)
      client_certificate        = lookup(replica_configuration.value, "client_certificate", null)
      client_key                = lookup(replica_configuration.value, "client_key", null)
      connect_retry_interval    = lookup(replica_configuration.value, "connect_retry_interval", null)
      dump_file_path            = lookup(replica_configuration.value, "dump_file_path", null)
      failover_target           = lookup(replica_configuration.value, "failover_target", null)
      master_heartbeat_period   = lookup(replica_configuration.value, "master_heartbeat_period", null)
      password                  = lookup(replica_configuration.value, "password", null)
      ssl_cipher                = lookup(replica_configuration.value, "ssl_cipher", null)
      username                  = lookup(replica_configuration.value, "username", null)
      verify_server_certificate = lookup(replica_configuration.value, "verify_server_certificate", null)
    }
  }

  timeouts {
    create = "60m"
    delete = "2h"
  }
}

resource "google_sql_database" "default" {
  count     = var.master_instance_name == "" ? 1 : 0
  name      = var.db_name
  project   = var.project
  instance  = google_sql_database_instance.master.name
  charset   = var.db_charset
  collation = var.db_collation
}

resource "random_id" "user-password" {
  byte_length = 8
}

resource "google_sql_user" "default" {
  count    = var.master_instance_name == "" ? 1 : 0
  name     = var.user_name
  project  = var.project
  instance = google_sql_database_instance.master.name
  host     = var.user_host
  password = var.user_password == "" ? random_id.user-password.hex : var.user_password
}
```

<h2 id="step8">Terraform を実行する</h2>
<p><code>terraform init</code> コマンドを使用して、Terraform 構成ファイルを含む作業ディレクトリを初期化します。</p>
<p>このコマンドは、いくつかの初期化ステップを実行して、作業ディレクトリを使用できるように準備します。このコマンドは、いつでも何度でも安全に実行できます。実行すると、作業ディレクトリに構成の変更が反映されます。</p>
<pre><code class="language-bash prettyprint">terraform init&#x000A;</code></pre>
<p><code>terraform plan</code> コマンドを使用して実行プランを作成します。このコマンドは省略可能ですが、実行することをおすすめします。このコマンドを実行すると、明示的に無効にしない限り更新が行われ、構成ファイルに指定されている目的の状態を実現するために必要なアクションが特定されます。</p>
<p>これは、一連の変更に対する実行プランが想定どおりに行われるかどうかを、実際のリソースや状態に変更を加えることなく確認するのに便利です。たとえば、変更をバージョン管理に commit する前に terraform plan を実行すると、想定どおりに機能するかどうかを事前に確認できます。</p>
<pre><code class="language-bash prettyprint">terraform plan -out=tfplan&#x000A;</code></pre>
<p>オプションの <code>-out</code> 引数を使用すると、生成されたプランをファイルに保存できます。保存したファイルは、後で <code>terraform apply</code> を使用して実行できます。</p>
<p><code>terraform apply</code> コマンドを使用して、構成を目的の状態にするために必要な変更を適用します。<code>terraform plan</code> を実行した場合は、実行プランによって生成された一連の定義済みアクションを適用できます。</p>
<pre><code class="language-bash prettyprint">terraform apply tfplan&#x000A;</code></pre>
<p>処理が完了するまで、少し時間がかかります。完了すると、次のような出力が表示されます。</p>
<pre><code class="language-output prettyprint">Apply complete! Resources: 5 added, 0 changed, 0 destroyed.&#x000A;&#x000A;The state of your infrastructure has been saved to the path&#x000A;below. This state is required to modify and destroy your&#x000A;infrastructure, so keep it safe. To inspect the complete state&#x000A;use the `terraform show` command.&#x000A;&#x000A;State path: terraform.tfstate&#x000A;&#x000A;Outputs:&#x000A;&#x000A;generated_user_password = &lt;sensitive&gt;&#x000A;instance_address = 35.232.204.44&#x000A;instance_address_time_to_retire =&#x000A;instance_name = example-mysql-6808&#x000A;self_link = https://www.googleapis.com/sql/v1beta4/projects/[PROJECT_ID]/instances/example-mysql-6808&#x000A;</code></pre>

<h2 id="step9">Cloud SQL Proxy</h2>
<h3>プロキシの機能</h3>
<p>Cloud SQL Proxy を使用すると、<a href="https://cloud.google.com/sql/docs/mysql/configure-ip" target="_blank">IP アドレスのホワイトリストへの登録</a>や、<a href="https://cloud.google.com/sql/docs/mysql/configure-ssl-instance" target="_blank">SSL の構成</a>を行わなくても、Cloud SQL の第 2 世代インスタンスに安全にアクセスできます。</p>
<p>Cloud SQL Proxy を使用して Cloud SQL インスタンスにアクセスする利点は以下のとおりです。</p>
<ul>
<li>
<strong>安全な接続:</strong> プロキシは、TLS 1.2 と 128 ビット AES 暗号を使用して、データベースとの間で送受信されるトラフィックを自動的に暗号化します。クライアントとサーバーの ID の確認には、SSL 証明書が使用されます。</li>
<li>
<strong>簡単な接続管理:</strong> プロキシが Cloud SQL との認証を処理するので、静的な IP アドレスを提供する必要がなくなります。</li>
</ul>
<ql-infobox>
<strong>注:</strong> Cloud SQL に App Engine のスタンダード環境またはフレキシブル環境から接続する場合は、プロキシを使用したり SSL を構成したりする必要はありません。これらの接続では、「組み込み」のプロキシ実装が自動的に使用されます。
</ql-infobox>
<h3>Cloud SQL Proxy の仕組み</h3>
<p>Cloud SQL Proxy は、プロキシと呼ばれるローカル クライアントをローカル環境で実行することによって機能します。アプリケーションは、データベースで使用されている標準のプロトコルを介してプロキシと通信します。プロキシは、セキュアなトンネルを使用して、サーバー上で実行されているコンパニオン プロセスと通信します。</p>
<p>次の図では、プロキシが Cloud SQL に接続する方法を示します。</p>
<p><img alt="proxyconnection.png" src="https://cdn.qwiklabs.com/BEufWW4GnrS6Y29aGV5IH6w3%2F5QxbYk62b6hY5FLAiU%3D"></p>
<h2 id="step10">Cloud SQL Proxy のインストール</h2>
<ol>
<li>
<p>プロキシをダウンロードします。</p>
</li>
</ol>
<pre><code class="language-bash prettyprint">wget https://dl.google.com/cloudsql/cloud_sql_proxy.linux.amd64 -O cloud_sql_proxy&#x000A;</code></pre>
<ol start="2">
<li>
<p>プロキシを実行可能にします。</p>
</li>
</ol>
<pre><code class="language-bash prettyprint">chmod +x cloud_sql_proxy&#x000A;</code></pre>
<p>プロキシは、環境の任意の場所にインストールできます。プロキシ バイナリの場所が、アプリケーションから受信するデータをリッスンする場所に影響することはありません。</p>
<h3>プロキシの起動オプション</h3>
<p>プロキシを起動するときに、次の情報をプロキシに提供します。</p>
<ul>
<li>接続先の Cloud SQL インスタンス</li>
<li>Cloud SQL へ送信するためにアプリケーションから受信するデータをリッスンする場所</li>
<li>Cloud SQL に対するアプリケーションの認証に使用する認証情報を取得できる場所</li>
</ul>
<p>指定したプロキシ スタートアップ オプションによって、TCP ポートと Unix ソケットのどちらでリッスンするのかが決まります。Unix ソケットでリッスンする場合は、選択した場所（通常は <code>/cloudsql/</code> ディレクトリ）にソケットが作成されます。TCP の場合、プロキシはデフォルトで <code>localhost</code> でリッスンします。</p>
<h2 id="step11">データベースへの接続をテストする</h2>
<ol>
<li>
<p>まず、Cloud SQL インスタンスに対して Cloud SQL Proxy を実行します。</p>
</li>
</ol>
<pre><code class="language-bash prettyprint">export GOOGLE_PROJECT=$(gcloud config get-value project)&#x000A;</code></pre>
<pre><code class="language-bash prettyprint">MYSQL_DB_NAME=$(terraform output -json | jq -r '.instance_name.value')&#x000A;MYSQL_CONN_NAME="${GOOGLE_PROJECT}:us-central1:${MYSQL_DB_NAME}"&#x000A;</code></pre>
<ol start="2">
<li>
<p>次のコマンドを実行します。</p>
</li>
</ol>
<pre><code class="language-bash prettyprint">./cloud_sql_proxy -instances=${MYSQL_CONN_NAME}=tcp:3306&#x000A;</code></pre>
<p>次に、プラス記号（<strong>+</strong>）のアイコンをクリックして新しい Cloud Shell タブを開きます。このシェルを使用して Cloud SQL Proxy に接続します。</p>
<ol start="3">
<li>
<p><code>sql-with-terraform</code> ディレクトリに移動します。</p>
</li>
</ol>
<pre><code class="language-bash prettyprint">cd ~/sql-with-terraform&#x000A;</code></pre>
<ol start="4">
<li>
<p>生成された MYSQL のパスワードを取得します。</p>
</li>
</ol>
<pre><code class="language-bash prettyprint">echo MYSQL_PASSWORD=$(terraform output -json | jq -r '.generated_user_password.value')&#x000A;</code></pre>
<ol start="5">
<li>
<p>MySQL への接続をテストします。</p>
</li>
</ol>
<pre><code class="language-bash prettyprint">mysql -udefault -p --host 127.0.0.1 default&#x000A;</code></pre>
<ol start="6">
<li>
<p>プロンプトが表示されたら、上の出力で確認した <code>MYSQL_PASSWORD</code> の値を入力し、<strong>Enter</strong> キーを押します。</p>
</li>
<li>
<p>MYSQL のコマンドラインにログインできるはずです。<strong>Ctrl</strong>+<strong>D</strong> キーを押して MYSQL を終了します。</p>
</li>
<li>
<p>最初の Cloud Shell タブに戻ると、Cloud SQL Proxy に対する接続のログが表示されています。</p>
</li>
</ol>
