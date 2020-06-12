**GCP の基礎**
# Qwiklabs と Google Cloud Platform の概要
## GCP プロジェクトとは
GCP プロジェクトは、Google Cloud のリソースを 1 つにまとめているエンティティです。ほとんどの場合、ここにはリソースとサービスが含まれます。たとえば、仮想マシンプール、データベース セット、それらを相互に接続するネットワークなどが保持されます。プロジェクトには各種設定とアクセス許可も含まれており、これを使用して、セキュリティ ルールと誰がどのリソースにアクセスできるかを指定します。

一度に複数のプロジェクトにアクセスできます。大企業の、あるいは経験豊富な GCP ユーザーの場合、GCP プロジェクトが数十から数千あるのは珍しいことではありません。組織によって GCP の使用方法は異なるため、プロジェクトを使ってチームやプロダクト別にクラウド コンピューティング サービスを区別すると便利でしょう。

## ロールと権限
GCP にはクラウド コンピューティング サービスのほかに権限とロールの集合が格納されており、誰がどのリソースにアクセスできるかを定義できるようになっています。このロールと権限は、Cloud Identity and Access Management（IAM）サービスを使用して検査および変更ができます。

次の表は、[ロールについて](https://cloud.google.com/iam/docs/understanding-roles#primitive_roles) に書かれている定義の抜粋です。閲覧者、編集者、オーナーそれぞれのロールの権限が簡潔にまとめられています。

|ロール名|権限|
|--|--|
|roles/viewer|既存のリソースやデータを表示する（ただし変更は不可能）など、状態に影響しない読み取り専用アクションに必要な権限。|
|roles/editor|すべての閲覧者権限と、状態を変更するアクション（既存のリソースの変更など）に必要な権限。
|roles/owner|すべての編集者権限と、以下のアクションを実行するために必要な権限。<ul><li>プロジェクトおよびプロジェクト内のすべてのリソースの権限とロールを管理する。</li><li>プロジェクトの課金情報を設定する。</li></ul>

# 仮想マシンの作成

<h3>リージョンとゾーンについて</h3>
<p>一部の Compute Engine リソースは、リージョン内またはゾーン内にのみ存在します。リージョンとは、リソースを実行できる特定の地理的な場所です。各リージョンには、1 つまたは複数のゾーンがあります。たとえば、リージョン us-central1 は米国中部を指し、ゾーン <code>us-central1-a</code>、<code>us-central1-b</code>、<code>us-central1-c</code>、<code>us-central1-f</code> が含まれています。</p>
<p><img alt="regions_and_zones.png" src="https://cdn.qwiklabs.com/BErmNT8ZIzd5yqxO0lEJj8lAlKT3jKC%2BtI%2Byj3OSKDA%3D"></p>
<p>ゾーン内で動作するリソースはゾーンリソースと呼びます。仮想マシン インスタンスと永続ディスクは 1 つのゾーン内で動作します。永続ディスクを仮想マシン インスタンスに接続するには、両方のリソースを同じゾーン内に配置する必要があります。同様に、インスタンスに静的 IP アドレスを割り当てるには、インスタンスが静的 IP と同じリージョンに存在している必要があります。</p>
<aside>
	リージョンとゾーンの詳細については、<a href="https://cloud.google.com/compute/docs/regions-zones/">リージョンとゾーンのドキュメント</a>にあるリストをご覧ください。
	</aside>

<h2 id="step4">Cloud Console から新しいインスタンスを作成する</h2>
<p>ここでは、Cloud Console から Google Compute Engine を使用して、事前定義済みの新しいマシンタイプを作成する方法について学習します。</p>
<p>GCP Console の画面左上にある<strong>ナビゲーション メニュー</strong>で、[<strong>Compute Engine</strong>] &gt; [<strong>VM インスタンス</strong>] の順に選択します。</p>
<p><img alt="caf8a9546fcdb4d8.png" src="https://cdn.qwiklabs.com/cazq0JhVyKhbnPWikoaworD5UjdeV07A6ZwhItC3Idk%3D"></p>
<p>最初に初期化する際は 1 分ほどかかることがあります。</p>
<p>新しいインスタンスを作成するには、[<strong>作成</strong>] をクリックします。</p>
<p><img alt="9aaec89ac3d4c6a2.png" src="https://cdn.qwiklabs.com/%2BPseSeyW06BbibSolAef75j7hGYa4vBNp1uyXVaDvR8%3D"></p>
<p>新しいインスタンスの作成時に構成できるパラメータは多数ありますが、このラボでは以下を使用します。</p>
<table>

<tr>
<th>項目</th>
<th>値</th>
<th>その他の情報</th>
</tr>


<tr>
<td><p><strong>名前</strong></p></td>
<td><p><code>gcelab</code></p></td>
<td>VM インスタンスの名前</td>
</tr>
<tr>
<td><p><strong>リージョン</strong></p></td>
<td><p><code>us-central1（アイオワ）</code></p></td>
<td><p>リージョンの詳細については、<a href="https://cloud.google.com/compute/docs/zones" target="blank">リージョンとゾーン</a>のドキュメントをご確認ください。</p></td>
</tr>
<tr>
<td><p><strong>ゾーン</strong></p></td>
<td>
<p><code>us-central1-c </code></p>
<p><strong>注: </strong>後で必要になるため、選択したゾーンを控えておいてください。</p>
</td>
<td><p>ゾーンの詳細については、<a href="https://cloud.google.com/compute/docs/zones" target="blank">リージョンとゾーン</a>のドキュメントをご確認ください。</p></td>
</tr>
<tr>
<td><p><strong>マシンタイプ</strong></p></td>
<td>
<p><code>2 vCPUs</code></p> <p>これは（n1-standard-2）、</p> <p>2 CPU、7.5 GB RAM のインスタンスです。</p> <p>マイクロ インスタンス タイプから 32 コア、208 GB RAM のインスタンス タイプまで、多数のマシンタイプがあります。詳しくは、<a href="https://cloud.google.com/compute/docs/machine-types" target="blank">マシンタイプ</a>のドキュメントをご確認ください。</p>
</td>
<td><p><strong>注</strong>: 新しいプロジェクトにはデフォルトの<a href="https://cloud.google.com/compute/docs/resource-quotas" target="blank">リソース割り当て</a>が適用されるため、CPU コアの数が制限される場合があります。このラボ以外のプロジェクトで作業を行う際は、より多くの CPU コアをリクエストできます。</p></td>
</tr>
<tr>
<td><p><strong>ブートディスク</strong></p></td>
<td>
<p><code>新しい 10 GB の標準永続ディスク</code></p> <p><code>OS イメージ: Debian GNU/Linux 9（Stretch）</code></p>
</td>
<td><p>Debian、Ubuntu、CoreOS から、Red Hat Enterprise Linux や Windows Server などのプレミアム イメージまで、選択できるイメージは多数あります。詳しくは、オペレーション システムのドキュメントをご確認ください。</p></td>
</tr>
<tr>
<td><p><strong>ファイアウォール</strong></p></td>
<td>
<p>[<code>HTTP トラフィックを許可する</code>] をオンにします</p> <p>このオプションをオンにすると、後でインストールするウェブサーバーにアクセスできるようになります。</p>
</td>
<td><p><strong>注: </strong>これによって、ポート 80 に HTTP トラフィックを許可するファイアウォール ルールが自動的に作成されます。</p></td>
</tr>

</table>
<p>[<strong>作成</strong>] をクリックします。</p>
<p>完了するまで 1 分ほどお待ちください。</p>
<p>完了すると、[<strong>VM インスタンス</strong>] ページに新しい仮想マシンが表示されます。</p>
<p>仮想マシンに SSH 接続するには、右側の [<strong>SSH</strong>] をクリックします。これによって、ブラウザから直接 SSH クライアントを起動できます。</p>
<p><img alt="d64685c5118c91c7.png" src="https://cdn.qwiklabs.com/CX17Ly71sbbBa4PoO1YQpyM%2Fq3hTSrzJwnQSChEBoTc%3D"></p>
<ql-infobox>
<strong>注:</strong> 詳しくは、<a href="https://cloud.google.com/compute/docs/instances/connecting-to-instance" target="blank">インスタンスへの接続</a>のドキュメントをご確認ください。
</ql-infobox>
<h3>NGINX ウェブサーバーをインストールする</h3>
<p>お使いの仮想マシンを接続するために、世界的に利用されているウェブ サーバーである NGINX ウェブサーバーをインストールします。</p>
<p>SSH 接続を確立したら、<code>sudo</code> を使用して <code>root</code> アクセスを取得します。</p>
<pre><code>sudo su -&#x000A;</code></pre>
<p><code>root</code> ユーザーとして、OS を更新します。</p>
<pre><code>apt-get update&#x000A;</code></pre>
<p>（出力）</p>
<pre><code class="language-bash prettyprint">Get:1 http://security.debian.org stretch/updates InRelease [94.3 kB]&#x000A;Ign http://deb.debian.org strech InRelease&#x000A;Get:2 http://deb.debian.org strech-updates InRelease [91.0 kB]&#x000A;...&#x000A;</code></pre>
<p>NGINX をインストールします。</p>
<pre><code>apt-get install nginx -y&#x000A;</code></pre>
<p>（出力）</p>
<pre><code class="language-bash prettyprint">Reading package lists... Done&#x000A;Building dependency tree&#x000A;Reading state information... Done&#x000A;The following additional packages will be installed:&#x000A;...&#x000A;</code></pre>
<p>NGINX が動作していることを確認します。</p>
<pre><code>ps auwx | grep nginx&#x000A;</code></pre>
<p>（出力）</p>
<pre><code class="language-bash prettyprint">root      2330  0.0  0.0 159532  1628 ?        Ss   14:06   0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;&#x000A;www-data  2331  0.0  0.0 159864  3204 ?        S    14:06   0:00 nginx: worker process&#x000A;www-data  2332  0.0  0.0 159864  3204 ?        S    14:06   0:00 nginx: worker process&#x000A;root      2342  0.0  0.0  12780   988 pts/0    S+   14:07   0:00 grep nginx&#x000A;</code></pre>
<p>問題ないようです。ウェブページを表示するには、Cloud Console に移動し、仮想マシン インスタンスの<code>外部 IP</code> リンクをクリックします。このウェブページは、新しいブラウザ ウィンドウまたはタブで <code>http://EXTERNAL_IP/</code> に外部 IP<code></code> を入れても表示できます。</p>
<p><img alt="fa24dc2b865d88cf.png" src="https://cdn.qwiklabs.com/fEO7THcBhNClASAGR3HC3y%2Fk2IUVsmS89ggQC0L3wmU%3D"></p>
<p>このデフォルトのウェブページが表示されます。</p>
<p><img alt="c0908a068b419647.png" src="https://cdn.qwiklabs.com/%2BOL6V6asztPiJZBeMO0xqc%2FiAX8iJfSTr4wDkJmKQu4%3D"></p>
<p>このラボの進捗状況を確認するには、下の [<strong>進行状況を確認</strong>] をクリックします。チェックマークが表示されていれば、予定通りに進んでいます。</p>
<ql-activity-tracking step="1">
     Compute Engine インスタンスを作成し、必要なファイアウォール ルールとともに NGINX サーバーをインスタンスに追加する
</ql-activity-tracking>
<h2 id="step5">gcloud で新しいインスタンスを作成する</h2>
<p>仮想マシン インスタンスは、GCP Console ではなく、<a href="https://cloud.google.com/developer-shell/#how_do_i_get_started">Google Cloud Shell</a> にあらかじめインストールされているコマンドライン ツール <code>gcloud</code> を使用して作成することもできます。Cloud Shell は Debian ベースの仮想マシンで、さまざまな開発ツール（<code>gcloud</code>、<code>git</code> など）や、永続的な 5 GB のホーム ディレクトリが提供されています。</p>
<ql-infobox>
<p>今後ご自身のマシンで使用する場合は、<a href="https://cloud.google.com/sdk/gcloud/" target="blank">gcloud コマンドライン ツールの概要</a>をご確認ください。</p>
</ql-infobox>
<p>Cloud Shell で、<code>gcloud</code> を使用してコマンドラインから新しい仮想マシン インスタンスを作成します。</p>
<pre><code>gcloud compute instances create gcelab2 --machine-type n1-standard-2 --zone us-central1-c&#x000A;</code></pre>
<p>（出力）</p>
<pre><code class="language-bash prettyprint">Created [...gcelab2].&#x000A;NAME     ZONE           MACHINE_TYPE  ...    STATUS&#x000A;gcelab2  us-central1-c  n1-standard-2 ...    RUNNING&#x000A;</code></pre>
<p>下の [<strong>進行状況を確認</strong>] をクリックして、このラボの進捗状況を確認します。</p>
<ql-activity-tracking step="2">
     gcloud で新しいインスタンスを作成する
</ql-activity-tracking>
<p>作成されるインスタンスは、デフォルトで次のようになります。</p>
<ul>
<li>最新の <a href="https://cloud.google.com/compute/docs/images#debian">Debian 9（stretch）</a>イメージが使用されます。</li>
<li>
<a href="https://cloud.google.com/compute/docs/machine-types">マシンタイプ</a>は <code>n1-standard-2</code> になります。このラボでは、<code>n1-highmem-4</code> または <code>n1-highcpu-4</code> を選択することも可能です。Qwiklabs 以外のプロジェクトでは、<a href="https://cloud.google.com/compute/docs/instances/creating-instance-with-custom-machine-type">カスタム マシンタイプ</a>を指定することもできます。</li>
<li>インスタンスと同じ名前のルート永続ディスクが自動的にインスタンスに組み込まれます。</li>
</ul>
<p><code>gcloud compute instances create --help</code> を実行すると、すべてのデフォルト設定が表示されます。</p>
<ql-infobox>
<p><strong>注: </strong>常に同じリージョンまたはゾーンを使用しており、毎回 <code>--zone</code> フラグを追加しなくてすむようにしたい場合は、<code>gcloud</code> で使用されるデフォルトのリージョンとゾーンを設定することもできます。以下のコマンドで設定できます。</p>
<p><code>gcloud config set compute/zone ...</code></p>
<p><code>gcloud config set compute/region ...</code></p>
</ql-infobox>
<p><code>help</code> を終了するには、<strong>Ctrl</strong>+<strong>C</strong> キーを押します。</p>
<p>インスタンスを確認します。<strong>ナビゲーション メニュー</strong>から [<strong>Compute Engine</strong>] &gt; [<strong>VM インスタンス</strong>] の順に選択すると、このラボで作成した 2 つのインスタンスが表示されます。</p>
<p><img alt="vm-instances.png" src="https://cdn.qwiklabs.com/162OtnL0LAR6EnPSMqNojHEw9nPfAh9SU4ZPRzwK%2B%2Fw%3D"></p>
<p>これで、<code>gcloud</code> を使用してインスタンスに SSH 接続を行うことができるようになりました。適切なゾーンを追加するよう注意してください。オプションをグローバルに設定した場合は <code>--zone</code> フラグを省略します。</p>
<pre><code>gcloud compute ssh gcelab2 --zone us-central1-c&#x000A;</code></pre>
<p>（出力）</p>
<pre><code class="language-bash prettyprint">WARNING: The public SSH key file for gcloud does not exist.&#x000A;WARNING: The private SSH key file for gcloud does not exist.&#x000A;WARNING: You do not have an SSH key for gcloud.&#x000A;WARNING: [/usr/bin/ssh-keygen] will be executed to generate a key.&#x000A;This tool needs to create the directory&#x000A;[/home/gcpstaging306_student/.ssh] before being able to generate SSH&#x000A;Keys.&#x000A;</code></pre>
<p>ここで、「<strong>Y</strong>」と入力して続行します。</p>
<pre><code>Do you want to continue? (Y/n)&#x000A;</code></pre>
<p>パスフレーズ セクションでは <strong>Enter</strong> キーを押し、パスフレーズを空白のままにします。</p>
<pre><code class="language-bash prettyprint">Generating public/private rsa key pair.&#x000A;Enter passphrase (empty for no passphrase)&#x000A;</code></pre>
<p>接続後、リモートシェルを終了して SSH 接続を解除します。</p>
<pre><code>exit&#x000A;</code></pre>

# Cloud Shell と gcloud のスタートガイド

<h2 id="step2">概要</h2>
<p>Google Cloud Shell では、<code>gcloud</code> コマンドラインを使って、Google Cloud Platform でホストされているコンピューティング リソースにアクセスすることができます。Cloud Shell は Debian ベースの仮想マシンであり、5 GB の永続的なホーム ディレクトリを備えているため、GCP プロジェクトやリソースを簡単に管理できます。Cloud SDK の <code>gcloud</code> やその他の必要なユーティリティは Cloud Shell にプリインストールされているため、すぐに利用できます。</p>
<p>このハンズオンラボでは、Cloud Shell で <code>gcloud</code> コマンドラインを使って、Google Cloud Platform でホストされているコンピューティング リソースに接続する方法を学びます。</p>

<h2 id="step4">Cloud Shell の起動</h2>
<p>GCP Console の右上にあるアイコンをクリックして、Cloud Shell セッションを開きます。</p>
<p><img alt="console_topbar_icon_cloudshell.png" src="https://cdn.qwiklabs.com/vdY5e%2Fan9ZGXw5a%2FZMb1agpXhRGozsOadHURcR8thAQ%3D"></p>
<p>次に、[Cloud Shell の起動] をクリックします。</p>
<p><img alt="start_cloud_shell.png" src="https://cdn.qwiklabs.com/lr3PBRjWIrJ%2BMQnE8kCkOnRQQVgJnWSg4UWk16f0s%2FA%3D"></p>
<p>Cloud Shell を有効にしたら、コマンドラインを使用して、Cloud SDK の <code>gcloud</code> コマンドや、仮想マシン インスタンスで利用可能なその他のツールを実行することができます。このラボでは、後ほど <code>$HOME</code> ディレクトリを使用します。このディレクトリを使用すると、ファイルを永続ディスク ストレージに保存して、複数のプロジェクトや Cloud Shell セッションで使用できます。<code>$HOME</code> はプライベート ディレクトリであるため、他のユーザーはアクセスできません。</p>
<h2 id="step5">リージョンとゾーンについて</h2>
<p>一部の Compute Engine リソースは、リージョンまたはゾーンに属します。リージョンとは、リソースを実行できる特定の地理的な場所です。各リージョンには、1 つまたは複数のゾーンがあります。たとえば、リージョン us-central1 は米国中部のリージョンを指し、ゾーン <code>us-central1-a</code>、<code>us-central1-b</code>、<code>us-central1-c</code>、<code>us-central1-f</code> が含まれています。</p>
<p><img alt="regions_and_zones.png" src="https://cdn.qwiklabs.com/BErmNT8ZIzd5yqxO0lEJj8lAlKT3jKC%2BtI%2Byj3OSKDA%3D"></p>
<p>ゾーン内にあるリソースを、ゾーンリソースと呼びます。仮想マシン インスタンスと永続ディスクはゾーンに属します。永続ディスクを仮想マシン インスタンスに接続するには、両方のリソースを同じゾーン内に配置する必要があります。同様に、インスタンスに静的 IP アドレスを割り当てるには、インスタンスが静的 IP と同じリージョンに存在している必要があります。</p>
<aside>
<a href="https://cloud.google.com/compute/docs/regions-zones/">リージョンとゾーンについてのドキュメント</a>で、リージョンとゾーンの詳細と一覧をご確認ください。

</aside>
<p>デフォルトのリージョンとゾーンは、以下の値を使用して設定されます。</p>
<p><code>google-compute-default-zone</code>
<code>google-compute-default-region</code></p>
<p>デフォルトのリージョンとゾーンの設定を確認するには、次の <code>gcloud</code> コマンドを実行します。&lt;your_project_id&gt; は実際のプロジェクト ID に置き換えてください。実際のプロジェクト ID は、GCP Console のホームページか、このラボを開始した [Qwiklabs] タブで確認できます。</p>
<pre><code>gcloud compute project-info describe --project &lt;your_project_ID&gt;&#x000A;</code></pre>
<p>後ほど、この出力のゾーン（<code>google-compute-default-zone</code>）を使用します。</p>
<p>レスポンスで返されるデフォルトのゾーンとリージョンのメタデータ値を確認します。レスポンスに <code>google-compute-default-region</code> キーと <code>google-compute-default-zone</code> キーおよび値が含まれていない場合、それはデフォルトのゾーンまたはリージョンが設定されていないことを意味します。</p>
<h2 id="step6">Cloud SDK の初期化</h2>
<p><code>gcloud</code> CLI は Google Cloud SDK の一部です。<code>gcloud</code> コマンドライン ツールを使用する際は、事前にシステムに SDK をダウンロードしてインストールし、初期化（<code>gcloud init</code> を実行）しておく必要があります。</p>
<p>Cloud Shell では、<code>gcloud</code> CLI が自動的に利用可能になります。このラボでは Cloud Shell を使用しているため、<code>gcloud</code> を手動でインストールする必要はありません。</p>
<h2 id="step7">環境変数の設定</h2>
<p>環境変数とは、環境を定義する変数です。独自の変数を定義すると、API や実行可能ファイルを含むスクリプトの作成時に時間を節約できます。</p>
<p>環境変数をいくつか作成してみましょう。</p>
<pre><code>export PROJECT_ID=&lt;your_project_ID&gt;&#x000A;</code></pre>
<p>ZONE 環境変数を設定します（前のコマンドで取得したゾーンの値を使用します）。</p>
<pre><code>export ZONE=&lt;your_zone&gt;&#x000A;</code></pre>
<p>変数が正しく設定されていることを確認します。</p>
<pre><code>echo $PROJECT_ID&#x000A;echo $ZONE&#x000A;</code></pre>
<h2 id="step8">gcloud による仮想マシンの作成</h2>
<p><code>gcloud</code> を使用して新しい仮想マシン インスタンスを作成します。次のコマンドを使用します。</p>
<ul>
<li>
<code>gcloud compute</code>: Compute Engine API よりわかりやすい形式で Google Compute Engine リソースを簡単に管理できます。</li>
<li>
<code>instances create</code>: 新しいインスタンスを作成します。</li>
</ul>
<p>次のコマンドを実行して VM を作成します。</p>
<pre><code>gcloud compute instances create gcelab2 --machine-type n1-standard-2 --zone $ZONE&#x000A;</code></pre>
<ul>
<li>VM の名前は「gcelab2」です。</li>
<li>
<code>--machine-type</code> フラグを使用して、マシンタイプに「n1-standard-2」を指定しています。</li>
<li>
<code>--zone</code> フラグを使用して、環境変数で定義したゾーンに作成するように指定しています。</li>
</ul>
<p>（出力）</p>
<p><img alt="gcloud_vm.png" src="https://cdn.qwiklabs.com/K%2FhXlIfrP4aHv%2FMvTKhHkx7ClOkBp4pXVsv6i7hYT4Y%3D"></p>
<p><code>--zone</code> フラグを省略した場合、<code>gcloud</code> はデフォルト プロパティに基づいてユーザーの希望のゾーンを推定します。<code>マシンタイプ</code>や<code>イメージ</code>など、インスタンスに必要なその他の設定が create コマンドで指定されていない場合はデフォルト値に設定されます。</p>
<h4>完了したタスクをテストする</h4>
<p>[<strong>進行状況を確認</strong>] をクリックして、実行したタスクを確認します。仮想マシンが gcloud で正常に作成されている場合は、評価スコアが表示されます。</p>
<ql-activity-tracking step="1">
    gcloud による仮想マシンの作成
</ql-activity-tracking>
<p>デフォルト値は、<code>create</code> コマンドのヘルプを表示することで確認できます。</p>
<pre><code>gcloud compute instances create --help&#x000A;</code></pre>
<h2 id="step9">gcloud コマンドの使用</h2>
<p><code>gcloud</code> コマンドの実行時に末尾に <code>-h</code>（help（ヘルプ）の「h」）フラグを付けると、使用方法の簡単なガイドラインが表示されます。<code></code></p>
<p>Cloud Shell で次のコマンドを実行します。</p>
<pre><code class="language-bash prettyprint">gcloud -h&#x000A;</code></pre>
<p>さらに詳細なヘルプを表示するには、<code>--help</code> フラグを付けるか、<code>gcloud help</code> コマンドを実行します。Cloud Shell で次のコマンドを実行します。</p>
<pre><code class="language-bash prettyprint">gcloud config --help&#x000A;</code></pre>
<p><strong>Enter</strong> キーや <strong>Space</strong> キーを押すと、ヘルプ情報を下にスクロールできます。</p>
<p>「<code>q</code>」と入力して終了します。</p>
<p>次のコマンドを実行します。</p>
<pre><code class="language-bash prettyprint">gcloud help config&#x000A;</code></pre>
<p><code>gcloud config --help</code> コマンドも <code>gcloud help config</code> コマンドも同じように機能することがわかります。どちらのコマンドでも詳細なヘルプ情報が表示されます。</p>
<p><a href="https://cloud.google.com/sdk/gcloud/reference/" target="_blank"><code>gcloud</code> グローバル フラグ</a>は、呼び出しごとにコマンドの動作を制御します。フラグは、SDK のプロパティで設定された値よりも優先されます。</p>
<p>使用環境の構成リストを表示してみましょう。</p>
<pre><code class="language-bash prettyprint">gcloud config list&#x000A;</code></pre>
<p>他のプロパティの設定内容を確認するには、以下を実行してすべてのプロパティを表示します。</p>
<pre><code class="language-bash prettyprint">gcloud config list --all&#x000A;</code></pre>
<p><code>components</code> でコンポーネントのリストを表示してみましょう。</p>
<pre><code>gcloud components list&#x000A;</code></pre>
<p>このラボでどのコンポーネントを使用できるのかがわかります。次のセクションでは新しいコンポーネントをインストールします。</p>
<h2 id="step10">オートコンプリート</h2>
<p><code>gcloud interactive</code> ではコマンドとフラグの自動プロンプトが表示され、コマンド入力時に下のセクションにインライン ヘルプ スニペットが表示されます。</p>
<p>コマンド名、サブコマンド名、フラグ名、列挙フラグ値などの静的情報は、プルダウンを使ってオートコンプリートされます。</p>

<p>ベータ版コンポーネントをインストールします。</p>
<pre><code>gcloud components install beta&#x000A;</code></pre>

<p><code>gcloud interactive</code> モードに移行します。</p>
<pre><code>gcloud beta interactive&#x000A;</code></pre>
<p>interactive モードの使用時に <strong>Tab</strong> キーを押すと、ファイルパスやリソースの引数の入力が補完されます。プルダウン メニューが表示されたら、<strong>Tab</strong> キーを使用してリスト内を移動し、<strong>Space</strong> キーを使用して候補を選択します。</p>
<p>試してみましょう。次のコマンドの先頭部分を入力し、オートコンプリートを使って入力を補完します。</p>
<pre><code>gcloud compute instances describe &lt;your_vm&gt;&#x000A;</code></pre>
<p>Cloud Shell の下部に、この機能の有効 / 無効を切り替えるショートカットが表示されています。F2 キーを押して試してみてください。</p>
<p>F2:help:STATE は、有効なヘルプを切り替えます。有効な場合は ON、無効な場合は OFF です。</p>
<h2 id="step11">VM インスタンスに SSH 接続する</h2>
<p><code>gcloud compute</code> を使用すると、インスタンスに簡単に接続できます。<code>gcloud compute ssh</code> コマンドは、SSH に対するラッパーを提供して、認証と、インスタンス名から IP アドレスへのマッピングを処理します。</p>
<p><code>gcloud compute ssh</code> を使用して SSH で VM に接続してみましょう。</p>
<pre><code>gcloud compute ssh gcelab2 --zone $ZONE&#x000A;</code></pre>
<p>（出力）</p>
<pre><code>WARNING: The public SSH key file for gcloud does not exist.&#x000A;WARNING: The private SSH key file for gcloud does not exist.&#x000A;WARNING: You do not have an SSH key for gcloud.&#x000A;WARNING: [/usr/bin/ssh-keygen] will be executed to generate a key.&#x000A;This tool needs to create the directory&#x000A;[/home/gcpstaging306_student/.ssh] before being able to generate SSH Keys.&#x000A;</code></pre>
<p>「Y」と入力して続行します。</p>
<pre><code>Do you want to continue? (Y/n)&#x000A;</code></pre>
<p>パスフレーズ セクションでは <strong>Enter</strong> キーを押して、パスフレーズを空白のままにします。</p>
<pre><code>Generating public/private rsa key pair.&#x000A;Enter passphrase (empty for no passphrase)&#x000A;</code></pre>
<p>ここでは特に何もする必要はないため、「exit」と入力してリモートシェルを終了し、SSH 接続を解除します。</p>
<pre><code>exit&#x000A;</code></pre>
<p>プロジェクトのコマンド プロンプトに戻るはずです。</p>
<h2 id="step12">ホーム ディレクトリの使用</h2>
<p>次にホーム ディレクトリを確認してみましょう。Cloud Shell ホーム ディレクトリの内容は、仮想マシンを終了して再起動した後でも、プロジェクトをまたいですべての Cloud Shell セッション間で保持されます。</p>
<p>現在の作業ディレクトリを変更します。</p>
<pre><code class="language-bash prettyprint">cd $HOME&#x000A;</code></pre>
<p><code>vi</code> テキスト エディタを使って、<code>.bashrc</code> 構成ファイルを開きます。</p>
<pre><code class="language-bash prettyprint">vi ./.bashrc&#x000A;</code></pre>
<p>エディタが開き、ファイルの内容が表示されます。<code>Esc</code> キーを押してから「<code>:wq</code>」と入力し、エディタを終了します。</p>

# Kubernetes Engine: クイックスタート

<h2 id="step2">概要</h2>
<p><a href="https://cloud.google.com/kubernetes-engine/" target="_blank">Google Kubernetes Engine</a>（GKE）は、Google のインフラストラクチャを使用して、コンテナ化されたアプリケーションのデプロイ、管理、スケーリングを行うマネージド環境を提供します。Kubernetes Engine 環境は複数のマシン（具体的には、<a href="https://cloud.google.com/compute" target="_blank">Google Compute Engine</a> インスタンス）で構成されており、これらのマシンがグループ化されて<a href="https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-architecture" target="_blank">コンテナ クラスタ</a>を形成します。このラボでは、GKE を使用してコンテナを作成し、アプリケーションをデプロイする方法を実践形式で学びます。</p>
<h3>Kubernetes Engine によるクラスタ オーケストレーション</h3>
<p>Kubernetes Engine クラスタは、<a href="https://kubernetes.io/" target="_blank">Kubernetes</a> オープンソース クラスタ管理システムを利用します。Kubernetes は、コンテナ クラスタを操作するメカニズムを提供します。Kubernetes のコマンドとリソースを使用して、アプリケーションのデプロイや管理、管理タスクの実行やポリシーの設定、デプロイ済みワークロードの状態のモニタリングを行うことができます。</p>
<p>Kubernetes では、頻繁に利用される Google サービスと同じ設計原理が使われており、自動管理、アプリケーション コンテナのモニタリングと稼働状況のプローブ、自動スケーリング、ローリング アップデートなどの機能が、Google サービスと同じように提供されています。コンテナ クラスタでアプリケーションを実行する際は、Google の 10 年以上の経験に裏付けられたテクノロジーを活かしてコンテナで本番環境のワークロードが処理されます。</p>
<h3>Google Cloud Platform における Kubernetes</h3>
<p>Kubernetes Engine クラスタを実行すると、Google Cloud Platform が提供する高度なクラスタ管理機能のメリットも得られます。たとえば、次のようなものです。</p>
<ul>
<li>Compute Engine インスタンスのための<a href="https://cloud.google.com/compute/docs/load-balancing-and-autoscaling" target="_blank">負荷分散</a>
</li>
<li>クラスタ内のノードのサブセットを定義する<a href="https://cloud.google.com/kubernetes-engine/docs/node-pools" target="_blank">ノードプール</a>による柔軟性の向上</li>
<li>クラスタノードのインスタンス数の<a href="https://cloud.google.com/kubernetes-engine/docs/cluster-autoscaler" target="_blank">自動スケーリング</a>
</li>
<li>クラスタノード ソフトウェアの<a href="https://cloud.google.com/kubernetes-engine/docs/node-auto-upgrade" target="_blank">自動アップグレード</a>
</li>
<li>ノードの正常稼働と可用性を保つ<a href="https://cloud.google.com/kubernetes-engine/docs/how-to/node-auto-repair" target="_blank">ノードの自動修復</a>
</li>
<li>Stackdriver の<a href="https://cloud.google.com/kubernetes-engine/docs/how-to/logging" target="_blank">ロギングとモニタリング</a>によるクラスタの状況確認</li>
</ul>
<p>Kubernetes の基本事項を理解したところで、今度は Kubernetes Engine を使用して、コンテナ化されたアプリケーションを短時間（30 分程度）でデプロイする方法を学習しましょう。下にスクロールし、以下に示す手順に沿ってラボ環境を設定してください。</p>

<h2 id="step4">デフォルトのコンピューティング ゾーンの設定</h2>
<p><a href="https://cloud.google.com/compute/docs/regions-zones/#available" target="_blank">コンピューティング ゾーン</a>とは、クラスタとそのリソースが存在するリージョンのおおよその場所を示すものです。たとえば、<code>us-central1-a</code> は <code>us-central1</code> リージョンのゾーンです。</p>
<p>Cloud Shell で新しいセッションを開始し、次のコマンドを実行してデフォルトのコンピューティング ゾーンを <code>us-central1-a</code> に設定します。</p>
<pre><code>gcloud config set compute/zone us-central1-a&#x000A;</code></pre>
<p>次の出力が表示されます。</p>
<pre><code class="language-output prettyprint">Updated property [compute/zone].&#x000A;</code></pre>
<h2 id="step5">Kubernetes Engine クラスタの作成</h2>
<p><a href="https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-architecture" target="_blank">クラスタ</a>は、少なくとも 1 つのクラスタ マスター<em></em>マシンと、ノード<em></em>と呼ばれる複数のワーカーマシンで構成されます。ノードは <a href="https://cloud.google.com/compute/docs/instances/">Compute Engine 仮想マシン（VM）インスタンス</a>であり、自身をクラスタの一部にするために必要な Kubernetes プロセスを実行します。</p>
<p>クラスタを作成するには、次のコマンドを実行します。<code>[CLUSTER-NAME]</code> は、クラスタに付ける名前（たとえば <code>my-cluster</code>）に置き換えてください。クラスタ名は、文字で始まり英数字で終わる 40 文字以下のものにする必要があります。</p>
<pre><code>gcloud container clusters create [CLUSTER-NAME]&#x000A;</code></pre>
<p>出力に含まれる警告は無視してかまいません。クラスタの作成が完了するまで数分かかることがあります。間もなく次のような出力が返されます。</p>
<pre><code class="language-output prettyprint">NAME        LOCATION       ...   NODE_VERSION  NUM_NODES  STATUS&#x000A;my-cluster  us-central1-a  ...   1.13.11-gke.9  3          RUNNING&#x000A;</code></pre>
<p>[<strong>進行状況を確認</strong>] をクリックして、目標に沿って進んでいることを確認します。
<ql-activity-tracking step="1">
Kubernetes Engine クラスタを作成する
</ql-activity-tracking></p>
<h2 id="step6">クラスタの認証情報を取得する</h2>
<p>クラスタの作成が完了したら、そのクラスタとやり取りするために必要な認証情報を取得します。</p>
<p>クラスタの認証情報を取得するには、次のコマンドを実行します。<code>[CLUSTER-NAME]</code> はクラスタの名前に置き換えてください。</p>
<pre><code>gcloud container clusters get-credentials [CLUSTER-NAME]&#x000A;</code></pre>
<p>次のような出力が返されます。</p>
<pre><code class="language-output prettyprint">Fetching cluster endpoint and auth data.&#x000A;kubeconfig entry generated for my-cluster.&#x000A;</code></pre>
<h2 id="step7">アプリケーションをクラスタにデプロイする</h2>
<p>作成したクラスタに、<a href="https://cloud.google.com/kubernetes-engine/docs/concepts/kubernetes-engine-overview" target="_blank">コンテナ化されたアプリケーション</a>をデプロイします。ここではクラスタで <code>hello-app</code> アプリケーションを実行します。</p>
<p>Kubernetes Engine では Kubernetes オブジェクトを使用して、クラスタのリソースを作成、管理します。Kubernetes には、ウェブサーバーのようなステートレス アプリケーションをデプロイするための <a href="https://kubernetes.io/docs/concepts/workloads/controllers/deployment/" target="_blank">Deployment</a> オブジェクトが用意されています。インターネットからアプリケーションにアクセスする際のルールと負荷分散を定義するには、<a href="https://kubernetes.io/docs/concepts/services-networking/service/" target="_blank">Service</a> オブジェクトを使用します。</p>
<p>Cloud Shell で次の <a href="https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create" target="_blank">kubectl create</a> コマンドを実行して、<code>hello-app</code> コンテナ イメージから <code>hello-server</code> という新しい Deployment を作成します。</p>
<pre><code>kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0&#x000A;</code></pre>
<p>次の出力が表示されます。</p>
<pre><code class="language-output prettyprint">deployment.apps/hello-server created&#x000A;</code></pre>
<p>これで、<code>hello-server</code> を表す Deployment オブジェクトが作成されます。この場合、<code>--image</code> にはデプロイするコンテナ イメージを指定します。上のコマンドでは、<a href="https://cloud.google.com/container-registry/docs" target="_blank">Google Container Registry</a> バケットからサンプル イメージが呼び出されます。<code>gcr.io/google-samples/hello-app:1.0</code> は、呼び出すイメージのバージョンを指定しています。バージョンが指定されていない場合は、最新バージョンが使用されます。</p>
<p>ここで、次の <a href="https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#expose" target="_blank">kubectl expose</a> コマンドを実行して Kubernetes Service を作成します。このサービスは、アプリケーションを外部トラフィックに公開するための Kubernetes リソースです。</p>
<pre><code>kubectl expose deployment hello-server --type=LoadBalancer --port 8080&#x000A;</code></pre>
<p>コマンドの内容:</p>
<ul>
<li>
<code>--port</code> にはコンテナで公開するポートを指定します。</li>
<li>
<code>type="LoadBalancer"</code> を渡すことで、コンテナの Compute Engine ロードバランサが作成されます。</li>
</ul>
<p>次の出力が表示されます。</p>
<pre><code class="language-output prettyprint">service/hello-server exposed&#x000A;</code></pre>
<p>[<strong>進行状況を確認</strong>] をクリックして、目標に沿って進んでいることを確認します。
<ql-activity-tracking step="2">
新しい Deployment の hello-server を作成する
</ql-activity-tracking></p>
<p><a href="https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get" target="_blank">kubectl get</a> を実行して、<code>hello-server</code> Service について確認します。</p>
<pre><code>kubectl get service&#x000A;</code></pre>
<p>次のような出力が返されます。</p>
<p><img alt="get_service_output.png" src="https://cdn.qwiklabs.com/lQsiuHvWAOnxttrz%2BrVtQjA56YbwUrpqei8X0XeUfVQ%3D"></p>
<aside class="special"><p><strong>注:</strong> 外部 IP アドレスの生成には数分かかることがあります。<code>EXTERNAL-IP</code> 列のステータスが「pending」の場合は、もう一度上のコマンドを実行してください。</p>
</aside>
<p>出力結果の <code>EXTERNAL IP</code> 列に示されている、Service の外部 IP アドレスをコピーします。</p>
<p>外部 IP アドレスと公開ポートを指定して、ウェブブラウザでアプリケーションを表示します。</p>
<pre><code>http://[EXTERNAL-IP]:8080&#x000A;</code></pre>
<p>ページは次のようになります。</p>
<p><img alt="output.png" src="https://cdn.qwiklabs.com/Et91dORVgSJkoFOa6UVbdtwzKaFzmliTSYhOrj8ONbw%3D"></p>

<h2 id="step8">クリーンアップ</h2>
<p>次のコマンドを実行してクラスタを削除します。</p>
<pre><code>gcloud container clusters delete [CLUSTER-NAME]&#x000A;</code></pre>
<p>プロンプトが表示されたら、「<strong>Y</strong>」と入力して確定します。クラスタの削除には数分かかる場合があります。Google Kubernetes Engine クラスタの削除については、<a href="https://cloud.google.com/kubernetes-engine/docs/how-to/deleting-a-cluster">こちらのドキュメント</a>をご覧ください。</p>

# ネットワーク ロードバランサと HTTP ロードバランサを設定する

<h2 id="step2">概要</h2>
<p>このハンズオンラボでは、ネットワーク ロードバランサと HTTP ロードバランサの違いや、Google Compute Engine 仮想マシンで実行されるアプリケーションでの設定方法について学びます。</p>
<p><a href="https://cloud.google.com/load-balancing/docs/load-balancing-overview#a_closer_look_at_cloud_load_balancers">Google Cloud Platform で負荷を分散する</a>には、いくつかの方法があります。このラボでは、次のロードバランサの設定について詳しく説明します。</p>
<ul>
<li>L3 <a href="https://cloud.google.com/compute/docs/load-balancing/network/">ネットワーク ロードバランサ</a>
</li>
<li>L7 <a href="https://cloud.google.com/compute/docs/load-balancing/http/">HTTP(S) ロードバランサ</a>
</li>
</ul>
<p>根底にあるコンセプトを理解できるよう、コマンドは手動で入力することをおすすめしますが、多くのラボでは必要なコマンドが含まれたコードブロックが提供されています。ラボでは、コードブロックのコマンドをコピーして該当する場所に貼り付けることもできます。</p>
<h4>演習内容</h4>
<ul>
<li>
<p>ネットワーク ロードバランサを設定する。</p>
</li>
<li>
<p>HTTP(S) ロードバランサを設定する。</p>
</li>
<li>
<p>ネットワーク ロードバランサと HTTP ロードバランサの違いを実際に体験して学ぶ。</p>
</li>
</ul>
<h3>前提条件</h3>
<p>Linux の標準的なテキスト エディタ（<code>vim</code>、<code>emacs</code>、<code>nano</code> など）の使用経験があると役に立ちます。</p>

<h2 id="step4">すべてのリソースにデフォルトのリージョンとゾーンを設定する</h2>
<p>Cloud Shell でデフォルト ゾーンを設定します。</p>
<pre><code>gcloud config set compute/zone us-central1-a&#x000A;</code></pre>
<p>デフォルト リージョンを設定します。</p>
<pre><code>gcloud config set compute/region us-central1&#x000A;</code></pre>
<p>ゾーンとリージョンの選択について詳しくは、<a href="https://cloud.google.com/compute/docs/zones">リージョンとゾーンのドキュメント</a>をご覧ください。</p>
<aside class="special"><p><strong>注: </strong>自分のマシンで <code>gcloud</code> を実行した場合は、セッション間で <code>config</code> 設定が維持されます。Cloud Shell では、新しいセッションまたは再接続のたびに設定し直す必要があります。</p>
</aside>
<h2 id="step5">複数のウェブサーバー インスタンスの作成</h2>
<p>マシンクラスタの挙動をシミュレートするため、<a href="https://cloud.google.com/compute/docs/instance-templates">インスタンス テンプレート</a>と<a href="https://cloud.google.com/compute/docs/instance-groups/">マネージド インスタンス グループ</a>を使用して、静的コンテンツを提供するシンプルな Nginx ウェブサーバー クラスタを作成します。インスタンス テンプレートでは、クラスタ内の各仮想マシンの状態（ディスク、CPU、メモリなど）が定義されます。マネージド インスタンス グループでは、インスタンス テンプレートを使用して多数の仮想マシンのインスタンスが作成されます。</p>
<p>Nginx ウェブサーバー クラスタを作成するため、次のものを作成します。</p>
<ul>
<li>起動時に Nginx サーバーを構成するすべての仮想マシン インスタンスで使用される、起動スクリプト</li>
<li>起動スクリプトを使用するインスタンス テンプレート</li>
<li>ターゲット プール</li>
<li>インスタンス テンプレートを使用するマネージド インスタンス グループ</li>
</ul>
<p>引き続き Cloud Shell で、すべての仮想マシン インスタンスで使用される起動スクリプトを作成します。このスクリプトの起動時に Nginx サーバーが構成されます。</p>
<pre><code>cat &lt;&lt; EOF &gt; startup.sh&#x000A;#! /bin/bash&#x000A;apt-get update&#x000A;apt-get install -y nginx&#x000A;service nginx start&#x000A;sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html&#x000A;EOF&#x000A;</code></pre>
<p>起動スクリプトを使用するインスタンス テンプレートを作成します。</p>
<pre><code>gcloud compute instance-templates create nginx-template \&#x000A;         --metadata-from-file startup-script=startup.sh&#x000A;</code></pre>
<p>（出力）</p>
<pre><code class="language-bash prettyprint">Created [...].&#x000A;NAME           MACHINE_TYPE  PREEMPTIBLE CREATION_TIMESTAMP&#x000A;nginx-template n1-standard-1             2015-11-09T08:44:59.007-08:00&#x000A;</code></pre>
<p>ターゲット プールを作成します。ターゲット プールは、グループ内のすべてのインスタンスに 1 か所からアクセスできるようにするもので、この後のステップで負荷分散のために必要になります。</p>
<pre><code>gcloud compute target-pools create nginx-pool&#x000A;</code></pre>
<p>（出力）</p>
<pre><code class="language-bash prettyprint">Created [...].&#x000A;NAME       REGION       SESSION_AFFINITY BACKUP HEALTH_CHECKS&#x000A;nginx-pool us-central1&#x000A;</code></pre>
<p>インスタンス テンプレートを使用してマネージド インスタンス グループを作成します。</p>
<pre><code>gcloud compute instance-groups managed create nginx-group \&#x000A;         --base-instance-name nginx \&#x000A;         --size 2 \&#x000A;         --template nginx-template \&#x000A;         --target-pool nginx-pool&#x000A;</code></pre>
<p>（出力）</p>
<pre><code class="language-bash prettyprint">Created [...].&#x000A;NAME         LOCATION       SCOPE  BASE_INSTANCE_NAME  SIZE  TARGET_SIZE  INSTANCE_TEMPLATE  AUTOSCALED&#x000A;nginx-group  us-central1-a  zone   nginx               0     2            nginx-template     no&#x000A;</code></pre>
<p>これで、名前が <code>nginx-</code> で始まる 2 つの仮想マシン インスタンスが作成されます。これには数分かかります。</p>
<p>コンピューティング エンジン インスタンスの一覧を表示し、すべてのインスタンスが作成されたことを確認します。</p>
<pre><code>gcloud compute instances list&#x000A;</code></pre>
<p>（出力）</p>
<pre><code class="language-bash prettyprint">NAME       ZONE           MACHINE_TYPE  PREEMPTIBLE INTERNAL_IP EXTERNAL_IP    STATUS&#x000A;nginx-7wvi us-central1-a n1-standard-1             10.240.X.X  X.X.X.X           RUNNING&#x000A;nginx-9mwd us-central1-a n1-standard-1             10.240.X.X  X.X.X.X           RUNNING&#x000A;</code></pre>
<p>次にファイアウォールを構成し、<code>EXTERNAL_IP</code> アドレス経由でマシンのポート 80 に接続できるようにします。</p>
<pre><code>gcloud compute firewall-rules create www-firewall --allow tcp:80&#x000A;</code></pre>
<p>前述のコマンドの実行結果として示される外部 IP アドレスを使用すると、<code>http://EXTERNAL_IP/</code> で各インスタンスに接続することができます。</p>
<p>ラボの進行状況を確認します。下の [<strong>進行状況を確認</strong>] をクリックして、ウェブサーバーのグループを作成したことを確認します。</p>
<ql-activity-tracking step="1">
    ウェブサーバーのグループを作成する
</ql-activity-tracking>
<h2 id="step6">ネットワーク ロードバランサの作成</h2>
<p>ネットワーク ロードバランサを利用すると、システムの負荷を受信 IP プロトコル データ（アドレス、ポート、プロトコル タイプなど）に基づいて分散することができます。HTTP(S) ロード バランシングにはないオプションもいくつか利用できます。たとえば、SMTP トラフィックなど、TCP/UDP ベースの付加的なプロトコルの負荷を分散することができます。自身のアプリケーションで TCP 接続関連の仕組みが使用されている場合、ネットワーク ロード バランシングでは、HTTP(S) ロード バランシングとは異なり、アプリケーションパケットをチェックすることができます。</p>
<aside>
さらに詳しくは、<a href="https://cloud.google.com/compute/docs/load-balancing/network/">ネットワーク負荷分散の設定</a>をご確認ください。

</aside>
<p>実習用のインスタンス グループを対象とした L3 ネットワーク ロードバランサを作成します。</p>
<pre><code>gcloud compute forwarding-rules create nginx-lb \&#x000A;         --region us-central1 \&#x000A;         --ports=80 \&#x000A;         --target-pool nginx-pool&#x000A;</code></pre>
<p>（出力）</p>
<pre><code class="language-bash prettyprint">Created [https://www.googleapis.com/compute/v1/projects/...].&#x000A;</code></pre>
<p>プロジェクト内のすべての Google Compute Engine 転送ルールを一覧表示します。</p>
<pre><code>gcloud compute forwarding-rules list&#x000A;</code></pre>
<p>（出力）</p>
<pre><code class="language-bash prettyprint">NAME     REGION       IP_ADDRESS     IP_PROTOCOL TARGET&#x000A;nginx-lb us-central1 X.X.X.X        TCP         us-central1/targetPools/nginx-pool&#x000A;</code></pre>
<p>次に、ブラウザでロードバランサ <code>http://IP_ADDRESS/</code> にアクセスします（<code>IP_ADDRESS</code> は前述のコマンドの実行結果に示されたアドレスを示します）。</p>
<p>ラボの進行状況を確認します。下の [<strong>進行状況を確認</strong>] をクリックして、ウェブサーバーを指す L3 ネットワーク ロードバランサが作成されたことを確認します。</p>
<ql-activity-tracking step="2">
    ウェブサーバーを指す L3 ネットワーク ロードバランサを作成する
</ql-activity-tracking>
<h2 id="step7">HTTP(S) ロードバランサの作成</h2>
<p>HTTP(S) ロードバランサでは、インスタンスに対する HTTP(S) リクエストの負荷をグローバルに分散します。あるセットのインスタンスに一部の URL をルーティングし、別のインスタンスには別の URL をルーティングする URL ルールを構成することができます。インスタンス グループに十分な容量があり、リクエストに対して適切であれば、ユーザーに最も近いインスタンス グループにリクエストが常に割り振られます。最も近いグループに十分な容量がない場合、容量が十分ある最寄りのグループにリクエストが送信されます。</p>
<aside class="special"><p>さらに詳しくは、<a href="https://cloud.google.com/compute/docs/load-balancing/http/" target="blank">ドキュメントの HTTP(S) ロードバランサ</a>をご確認ください。</p>
</aside>
<p>最初に、<a href="https://cloud.google.com/compute/docs/load-balancing/health-checks">ヘルスチェック</a>を作成します。ヘルスチェックでは、インスタンスが HTTP または HTTPS トラフィックにレスポンスすることが確認されます。</p>
<pre><code>gcloud compute http-health-checks create http-basic-check&#x000A;</code></pre>
<p>（出力）</p>
<pre><code class="language-bash prettyprint">Created [https://www.googleapis.com/compute/v1/projects/...].&#x000A;NAME             HOST PORT REQUEST_PATH&#x000A;http-basic-check      80   /&#x000A;</code></pre>
<p>HTTP サービスを定義し、ポート名をインスタンス グループの該当するポートにマッピングします。これで、名前を指定したポートに負荷分散サービスがトラフィックを転送できるようになります。</p>
<pre><code>gcloud compute instance-groups managed \&#x000A;       set-named-ports nginx-group \&#x000A;       --named-ports http:80&#x000A;</code></pre>
<p>（出力）</p>
<pre><code class="language-bash prettyprint">Updated [https://www.googleapis.com/compute/v1/projects/...].&#x000A;</code></pre>
<p><a href="https://cloud.google.com/compute/docs/reference/latest/backendServices">バックエンド サービス</a>を作成します。</p>
<pre><code>gcloud compute backend-services create nginx-backend \&#x000A;      --protocol HTTP --http-health-checks http-basic-check --global&#x000A;</code></pre>
<p>（出力）</p>
<pre><code class="language-bash prettyprint">Created [https://www.googleapis.com/compute/v1/projects/...].&#x000A;NAME          BACKENDS PROTOCOL&#x000A;nginx-backend          HTTP&#x000A;</code></pre>
<p>バックエンド サービスにインスタンス グループを追加します。</p>
<pre><code>gcloud compute backend-services add-backend nginx-backend \&#x000A;    --instance-group nginx-group \&#x000A;    --instance-group-zone us-central1-a \&#x000A;    --global&#x000A;</code></pre>
<p>（出力）</p>
<pre><code class="language-bash prettyprint">Updated [https://www.googleapis.com/compute/v1/projects/...].&#x000A;</code></pre>
<p>あらゆるインスタンスへの受信リクエストをすべて振り向けるデフォルトの URL マップを作成します。</p>
<pre><code>gcloud compute url-maps create web-map \&#x000A;    --default-service nginx-backend&#x000A;</code></pre>
<p>（出力）</p>
<pre><code class="language-bash prettyprint">Created [https://www.googleapis.com/compute/v1/projects/...].&#x000A;NAME    DEFAULT_SERVICE&#x000A;Web-map nginx-backend&#x000A;</code></pre>
<aside class="special"><p>リクエストされた URL に基づいてトラフィックを異なるインスタンスに振り向けるには、<a href="https://cloud.google.com/compute/docs/load-balancing/http/content-based-example" target="_blank">コンテンツ ベースのルーティング</a>をご確認ください。</p>
</aside>
<p>自身の URL マップにリクエストを振り向けるターゲット HTTP プロキシを作成します。</p>
<pre><code>gcloud compute target-http-proxies create http-lb-proxy \&#x000A;    --url-map web-map&#x000A;</code></pre>
<p>（出力）</p>
<pre><code class="language-bash prettyprint">Created [https://www.googleapis.com/compute/v1/projects/...].&#x000A;NAME          URL_MAP&#x000A;http-lb-proxy web-map&#x000A;</code></pre>
<p>受信リクエストを処理し、振り向ける、グローバルの転送ルールを作成します。転送ルールにより、IP アドレス、IP プロトコル、ポートの指定に応じて、特定のターゲット HTTP プロキシまたは HTTPS プロキシにトラフィックが送信されるようになります。なお、グローバル転送ルールでは複数のポートはサポートされていません。</p>
<pre><code>gcloud compute forwarding-rules create http-content-rule \&#x000A;        --global \&#x000A;        --target-http-proxy http-lb-proxy \&#x000A;        --ports 80&#x000A;</code></pre>
<p>（出力）</p>
<pre><code class="language-bash prettyprint">Created [https://www.googleapis.com/compute/v1/projects/...].&#x000A;</code></pre>
<p>グローバル転送ルールの作成後、設定が反映されるまで数分かかることがあります。</p>
<pre><code>gcloud compute forwarding-rules list&#x000A;</code></pre>
<p>（出力）</p>
<pre><code class="language-bash prettyprint">NAME              REGION IP_ADDRESS    IP_PROTOCOL TARGET&#x000A;http-content-rule        X.X.X.X       TCP         http-lb-proxy&#x000A;nginx-lb   us-central1  X.X.X.X       TCP         us-central1/....&#x000A;</code></pre>
<p>転送ルールの http-content-rule の IP_ADDRESS をメモしておきます。</p>
<p>ブラウザから <code>http://IP_ADDRESS/</code> に接続できるようになります。これには 3～5 分ほどかかる場合があります。接続できない場合は、少し待ってからブラウザを再読み込みしてください。</p>

----

コードまとめ

```
gcloud container clusters create nucleus-backend \
          --num-nodes 1 \
          --network nucleus-vpc \
          --region us-east1

gcloud container clusters get-credentials nucleus-backend \
          --region us-east1

kubectl create deployment hello-server \
          --image=gcr.io/google-samples/hello-app:2.0

kubectl expose deployment hello-server \
          --type=LoadBalancer \
          --port 8080
```

```
cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF

gcloud compute instance-templates create web-server-template \
          --metadata-from-file startup-script=startup.sh \
          --network nucleus-vpc \
          --machine-type g1-small \
          --region us-east1

gcloud compute instance-groups managed create web-server-group \
          --base-instance-name web-server \
          --size 2 \
          --template web-server-template \
          --region us-east1

gcloud compute firewall-rules create web-server-firewall \
          --allow tcp:80 \
          --network nucleus-vpc

gcloud compute http-health-checks create http-basic-check

gcloud compute instance-groups managed \
          set-named-ports web-server-group \
          --named-ports http:80 \
          --region us-east1

gcloud compute backend-services create web-server-backend \
          --protocol HTTP \
          --http-health-checks http-basic-check \
          --global

gcloud compute backend-services add-backend web-server-backend \
          --instance-group web-server-group \
          --instance-group-region us-east1 \
          --global

gcloud compute url-maps create web-server-map \
          --default-service web-server-backend

gcloud compute target-http-proxies create http-lb-proxy \
          --url-map web-server-map

gcloud compute forwarding-rules create http-content-rule \
        --global \
        --target-http-proxy http-lb-proxy \
        --ports 80

gcloud compute forwarding-rules list
```