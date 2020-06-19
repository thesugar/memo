# Cloud Source Repositories: Qwik Start

## 概要
Google Cloud Source Repositories には、アプリケーションやサービスの開発におけるコラボレーションを支援する Git バージョン管理機能が備わっています。このラボでは、サンプル ファイルを含むローカル Git リポジトリを作成し、Cloud Source Repositories をリモート リポジトリとして追加してローカル リポジトリのコンテンツを push します。また、Source Repositories に含まれるソースブラウザを使用して、Google Cloud Platform Console からリポジトリ ファイルを表示します。

<h2 id="step2">概要</h2>
<p><a href="https://cloud.google.com/source-repositories/">Google Cloud Source Repositories</a> には、アプリケーションやサービスの開発におけるコラボレーションを支援する Git バージョン管理機能が備わっています。このラボでは、サンプル ファイルを含むローカル Git リポジトリを作成し、Cloud Source Repositories をリモート リポジトリとして追加してローカル リポジトリのコンテンツを push します。また、Source Repositories に含まれるソースブラウザを使用して、Google Cloud Platform Console からリポジトリ ファイルを表示します。</p>

<h2 id="step4">新しいリポジトリを作成する</h2>
<p>Cloud Shell で新しいセッションを開始し、次のコマンドを実行して <code>REPO_DEMO</code> という新しい Cloud Source Repositories リポジトリを作成します。</p>
<pre><code>gcloud source repos create REPO_DEMO&#x000A;</code></pre>
<p>リポジトリ作成に対する請求についての警告は無視して構いません。</p>

<h2 id="step5">新しいリポジトリのクローンを Cloud Shell セッションに作成する</h2>
<p>新しい Cloud Source Repositories リポジトリのコンテンツのクローンを、Cloud Shell セッション内のローカル リポジトリに作成します。</p>
<pre><code>gcloud source repos clone REPO_DEMO&#x000A;</code></pre>
<p><code>gcloud source repos clone</code> コマンドを実行すると、origin という名前のリモート リポジトリとして Cloud Source Repositories が追加され、ローカル Git リポジトリにクローンが作成されます。</p>
<h2 id="step6">Cloud Source Repositories のリポジトリに push する</h2>
<p>作成したローカル リポジトリに移動します。</p>
<pre><code>cd REPO_DEMO&#x000A;</code></pre>
<p>次のコマンドを実行して、ローカル リポジトリに <code>myfile.txt</code> ファイルを作成します。</p>
<pre><code>echo "Hello World!" &gt; myfile.txt&#x000A;</code></pre>
<p>次の Git コマンドを使用してファイルを commit します。</p>
<pre><code>git config --global user.email "you@example.com"&#x000A;</code></pre>
<pre><code>git config --global user.name "Your Name"&#x000A;</code></pre>
<pre><code>git add myfile.txt&#x000A;</code></pre>
<pre><code>git commit -m "First file using Cloud Source Repositories" myfile.txt&#x000A;</code></pre>
<p>出力は次のようになります。</p>
<pre><code class="language-output prettyprint">[master (root-commit) c072ab6] First file using Cloud Source Repositories&#x000A; 1 file changed, 1 insertion(+)&#x000A; create mode 100644 myfile.txt&#x000A;</code></pre>
<p>ローカル リポジトリにコードを commit したら、<code>git push</code> コマンドを使用してコンテンツを Cloud Source Repositories に追加します。</p>
<pre><code>git push origin master&#x000A;</code></pre>
<p>git によって <code>master</code> ブランチから <code>origin</code> リモート サーバーにサンプル アプリケーション ファイルが push されます。</p>
<pre><code class="language-output prettyprint">Counting objects: 3, done.&#x000A;Writing objects: 100% (3/3), 247 bytes | 0 bytes/s, done.&#x000A;Total 3 (delta 0), reused 0 (delta 0)&#x000A;To https://source.developers.google.com/p/qwiklabs-gcp-ba5b4dcd/r/REPO_DEMO&#x000A; * [new branch]      master -&gt; master&#x000A;</code></pre>
<h2 id="step7">Google Cloud Source Repositories のリポジトリのファイルを参照する</h2>
<p>Google Cloud Source Repositories のソースコード ブラウザを使用してリポジトリ ファイルを表示します。特定のブランチ、タグ、commit ごとに表示をフィルタできます。</p>
<p>ナビゲーション メニューを開いて [<strong>Source Repositories</strong>] &gt; [<strong>ソースコード</strong>] を選択し、リポジトリに push したサンプル ファイルを参照します。</p>
<p><img alt="source_code.png" src="https://cdn.qwiklabs.com/BQhgfKbYsv2zq%2BEmHz9NfzcGw%2FPYuLBVTW7cqlwTUno%3D"></p>
<p>コンソールに、最新の commit 時点のマスター ブランチ内のファイルが表示されます。</p>
<h2 id="step8">GCP リポジトリのファイルを表示する</h2>
<p>[<code>REPO_DEMO</code>] &gt; [<code>myfile.txt</code>] をクリックして、ソースコード ブラウザでファイルの内容を表示します。</p>
<p><img alt="myfile_content.png" src="https://cdn.qwiklabs.com/Lwp%2FsCGnvAycES1Jk9%2BsYqBlHXVyIUKYhIARixjsWJk%3D"></p>
