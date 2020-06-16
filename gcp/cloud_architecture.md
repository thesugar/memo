# Kubernetes を使った Cloud のオーケストレーション
<h2 id="step2">概要</h2>
<p>このラボでは、次の方法について学びます。</p>
<ul>
<li>
<a href="https://cloud.google.com/container-engine">Kubernetes Engine</a> を使用して完全な <a href="http://kubernetes.io">Kubernetes</a> クラスタをプロビジョニングする。</li>
<li>kubectl を使用して Docker コンテナをデプロイおよび管理する。</li>
<li>Kubernetes のデプロイとサービスを使用してアプリケーションをマイクロサービスに分割する。</li>
</ul>
<p>Kubernetes はアプリケーションを取り扱うプラットフォームです。ラボのこのパートでは、「app」という名前のアプリケーション例を使用して演習を行います。</p>
<ql-infobox>
<p><a href="https://github.com/kelseyhightower/app" target="blank">app</a> は GitHub 上でホストされる、12 要素のサンプル アプリケーションです。このラボでは、次の Docker イメージを使用して作業します。</p>
<ul>
<li>
<a href="https://hub.docker.com/r/kelseyhightower/monolith" target="blank">kelseyhightower/monolith</a> - monolith には、auth サービスと hello サービスが含まれます。</li>
<li>
<a href="https://hub.docker.com/r/kelseyhightower/auth" target="blank">kelseyhightower/auth</a> - auth マイクロサービス。認証されたユーザーに対して JWT トークンを生成します。</li>
<li>
<a href="https://hub.docker.com/r/kelseyhightower/hello" target="blank">kelseyhightower/hello</a> - hello マイクロサービス。認証されたユーザーに挨拶します。</li>
<li>
<a href="https://hub.docker.com/_/nginx" target="blank">ngnix</a> - auth サービスと hello サービスのフロントエンド。</li>
</ul>
</ql-infobox>
<p>Kubernetes はオープンソースのプロジェクト（<a href="http://kubernetes.io/">kubernetes.io</a> から入手可能）で、ノートパソコンから可用性の高いマルチノード クラスタ、パブリック クラウドからオンプレミスのデプロイ、仮想マシンからベアメタルまで、さまざまな環境で実行できます。</p>
<p>このラボでは、Kubernetes Engine などのマネージド環境を使用することで、基盤となるインフラストラクチャの設定ではなく、Kubernetes を体験することに焦点を当てます。</p>

<h3><strong>Google Kubernetes Engine</strong></h3>
<p>Cloud Shell 環境で次のコマンドを入力して、ゾーンを設定します。</p>
<pre><code>gcloud config set compute/zone us-central1-b&#x000A;</code></pre>
<p>ゾーンを設定後、このラボで使用するクラスタを起動します。</p>
<pre><code>gcloud container clusters create io&#x000A;</code></pre>
<ql-infobox>
<p><strong>注:</strong> Kubernetes Engine がバックグラウンドでいくつかの仮想マシンをプロビジョニングしているため、クラスタの作成にはしばらく時間がかかります。</p>
</ql-infobox>
<h2 id="step4">サンプルコードを取得する</h2>
<p>次の Cloud Shell コマンドラインで、GitHub レポジトリのクローンを作成します。</p>
<pre><code>git clone https://github.com/googlecodelabs/orchestrate-with-kubernetes.git&#x000A;</code></pre>
<pre><code>cd orchestrate-with-kubernetes/kubernetes&#x000A;</code></pre>
<p>ファイルを一覧表示して、作業対象を確認します。</p>
<pre><code>ls&#x000A;</code></pre>
<p>サンプルは次のレイアウトになっています。</p>
<pre><code class="language-bash prettyprint">deployments/  /* デプロイ マニフェスト */&#x000A;  ...&#x000A;nginx/        /* nginx 構成ファイル */&#x000A;  ...&#x000A;pods/         /* Pod マニフェスト*/&#x000A;  ...&#x000A;services/     /* サービス マニフェスト */&#x000A;  ...&#x000A;tls/          /* TLS 証明書 */&#x000A;  ...&#x000A;cleanup.sh    /* クリーンアップ スクリプト */&#x000A;</code></pre>
<p>コードが入手できたので、今度は Kubernetes を試してみましょう。</p>
<h2 id="step5">Kubernetes のクイックデモ</h2>
<p><code>kubectl create</code> コマンドを使用すると、Kubernetes を簡単に使い始めることができます。このコマンドを使用して、nginx コンテナのインスタンスを 1 つ起動します。</p>
<pre><code>kubectl create deployment nginx --image=nginx:1.10.0&#x000A;</code></pre>
<p>Kubernetes が Deployment を 1 つ作成します。Deployment については後ほど詳しく説明します。ここでは、ポッドが実行されているノードで障害が発生した場合でも、Deployment はポットを稼働状態に保つことを知っていれば十分です。</p>
<p>Kubernetes では、すべてのコンテナがポッドで実行されます。<code>kubectl get pods</code> コマンドを使用して、実行中の nginx コンテナを表示します。</p>
<pre><code>kubectl get pods&#x000A;</code></pre>
<p>nginx コンテナが稼働すると、<code>kubectl expose</code> コマンドを使用して、それを Kubernetes の外部に公開できます。</p>
<pre><code>kubectl expose deployment nginx --port 80 --type LoadBalancer&#x000A;</code></pre>
<p>ここでは、パブリック IP アドレスが関連付けられた外部ロードバランサが、Kubernetes によって背後で作成されました。そのパブリック IP アドレスをヒットするクライアントは、サービスの背後にあるポッドにルーティングされます。この場合は nginx ポッドです。</p>
<p>ここで、<code>kubectl get</code> コマンドを使用して、サービスを一覧表示します。</p>
<pre><code>kubectl get services&#x000A;</code></pre>
<ql-infobox>
<p><strong>注:</strong> サービスの <code>ExternalIP</code> フィールドに値が入力されるまでに数秒かかる場合がありますが、これは正常な動作です。このフィールドに値が入力されるまで、数秒ごとに <code>kubectl get services</code> コマンドを再実行してください。</p>
</ql-infobox>
<p>リモートから Nginx コンテナにアクセスできるように、このコマンドに外部 IP を追加します。</p>
<pre><code>curl http://&lt;External IP&gt;:80&#x000A;</code></pre>
<p>このように、Kubernetes はそのまま簡単に使用できるワークフローに対応しており、実行および公開用の <code>kubectl</code> コマンドを使用してすぐに開始できます。</p>
<h3>完了したタスクをテストする</h3>
<p>下の [<strong>進行状況を確認</strong>] をクリックして、ラボの進捗状況を確認します。Kubernetes クラスタの作成と Nginx コンテナのデプロイが正常に行われている場合は、評価スコアが表示されます。</p>
<ql-activity-tracking step="1">
    Kubernetes クラスタを作成し、Nginx コンテナを起動する

</ql-activity-tracking>
<p>ここまで kubernetes について簡単に説明してきましたが、ここからは各コンポーネントと要約について見ていきます。</p>
<h2 id="step6">ポッド</h2>
<p>Kubernetes の中核は<a href="http://kubernetes.io/docs/user-guide/pods/">ポッド</a>です。</p>
<p>ポッドは、1 つ以上のコンテナのコレクションを表しており、また保持しています。一般に、互いに強い依存関係を持つコンテナが複数存在する場合、それらのコンテナを単一のポッド内にパッケージ化します。</p>
<p><img alt="fb02d86798243fcb.png" src="https://cdn.qwiklabs.com/tzvM5wFnfARnONAXX96nz8OgqOa1ihx6kCk%2BelMakfw%3D"></p>
<p>この例では、monolith コンテナと nginx コンテナを含むポッドがあります。</p>
<p>また、ポッドには<a href="http://kubernetes.io/docs/user-guide/volumes/">ボリューム</a>があります。ボリュームとはポッドが存在する限り存続するデータディスクで、そのポッド内のコンテナによって使用できます。ポッドは、そのコンテンツに対して共有の名前空間を提供します。これは、この例のポッド内の 2 つのコンテナが互いに通信できることと、接続されたボリュームを共有できることを意味します。</p>
<p>また、ポッドはネットワーク名前空間も共有します。これは、ポッドごとに 1 つの IP アドレスがあることを意味しています。</p>
<p>ポッドについて、さらに詳しく説明します。</p>
<h2 id="step7">ポッドを作成する</h2>
<p>ポッドは、ポッド構成ファイルを使用して作成できます。少し時間を取って monolith ポッド構成ファイルを詳しく見てみましょう。以下のコマンドを実行します。</p>
<pre><code>cat pods/monolith.yaml&#x000A;</code></pre>
<p>出力には、開いている構成ファイルが表示されます。</p>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: monolith
  labels:
    app: monolith
spec:
  containers:
    - name: monolith
      image: kelseyhightower/monolith:1.0.0
      args:
        - "-http=0.0.0.0:80"
        - "-health=0.0.0.0:81"
        - "-secret=secret"
      ports:
        - name: http
          containerPort: 80
        - name: health
          containerPort: 81
      resources:
        limits:
          cpu: 0.2
          memory: "10Mi"
```

<p>ここで、いくつか注目すべき点として以下のことがわかります。</p>
<ul>
<li>ポッドは 1 つのコンテナ（monolith）で構成されている。</li>
<li>コンテナの起動時に引数がいくつか渡されている。</li>
<li>http トラフィック用にポート 80 を開いている。</li>
</ul>
<p><code>kubectl</code> を使用して monolith ポッドを作成します。</p>
<pre><code>kubectl create -f pods/monolith.yaml&#x000A;</code></pre>
<p>ポッドを調べてみましょう。<code>kubectl get pods</code> コマンドを使用して、デフォルトの名前空間で動作するすべてのポッドを一覧表示します。</p>
<pre><code>kubectl get pods&#x000A;</code></pre>
<ql-infobox>
<p><strong>注:</strong> monolith ポッドが起動して動作するまでに数秒かかることがあります。実行する前に、monolith コンテナ イメージを Docker Hub から pull する必要があります。</p>
</ql-infobox>
<p>ポッドが実行されたら、<code>kubectl</code> <code>describe</code> コマンドを使用して、monolith ポッドに関する情報をさらに取得します。</p>
<pre><code>kubectl describe pods monolith&#x000A;</code></pre>
<p>ポッド IP アドレスやイベントログなど、monolith ポッドに関する情報は多数あります。これらの情報は、トラブルシューティングで役立ちます。</p>
<p>Kubernetes は、ポッドを構成ファイルに記述することで簡単に作成できるようにし、実行時にその情報を簡単に表示できるようにしています。この時点で、デプロイに必要なすべてのポッドを作成できます。</p>
<h2 id="step8">ポッドの操作</h2>
<p>デフォルトでは、ポッドにプライベート IP アドレスが割り当てられ、クラスタの外部から到達することはできません。ローカルポートを monolith ポッド内のポートにマッピングするには、<code>kubectl port-forward</code> コマンドを使用します。</p>
<ql-infobox>
<p>ここからは、複数の Cloud Shell タブを使ってポッド間の通信を設定します。第 2 または第 3 のコマンドシェルで実行されるコマンドについては、それぞれの説明の中でその旨が示されています。</p>
</ql-infobox>
<p>2 つの Cloud Shell ターミナルを開きます。1 つで <code>kubectl port-forward</code> コマンドを実行し、もう 1 つで <code>curl</code> コマンドを実行します。</p>
<p><strong>第 2 のターミナル</strong>で、次のコマンドを実行してポート転送を設定します。</p>
<pre><code>kubectl port-forward monolith 10080:80&#x000A;</code></pre>
<p>今度は、<strong>最初のターミナル</strong>で <code>curl</code> を使用してポッドとの通信を開始します。</p>
<pre><code>curl http://127.0.0.1:10080&#x000A;</code></pre>
<p>これで、コンテナから挨拶メッセージ「hello」が返されます。</p>
<p>今度は、<code>curl</code> コマンドを使用して、安全なエンドポイントをヒットするとどうなるか確認します。</p>
<pre><code>curl http://127.0.0.1:10080/secure&#x000A;</code></pre>

```
# 出力
authorization failed
```
<p>こうなりました。</p>
<p>では、ログインして認証トークンを monolith から取得してみましょう。</p>
<pre><code>curl -u user http://127.0.0.1:10080/login&#x000A;</code></pre>
<p>ログイン プロンプトで、秘密のパスワード「password」を使用してログインします。</p>
<p>ログインすると JWT トークンが表示されます。Cloud Shell は長い文字列のコピーをうまく処理できないため、トークンの環境変数を作成します。</p>
<pre><code>TOKEN=$(curl http://127.0.0.1:10080/login -u user|jq -r '.token')&#x000A;</code></pre>
<p>ホスト パスワードの入力を求められたら、秘密のパスワード「password」をもう一度入力します。</p>
<p>このコマンドを使用してトークンをコピーした後、そのトークンを使用して <code>curl</code> で安全なエンドポイントをヒットします。</p>
<pre><code>curl -H "Authorization: Bearer $TOKEN" http://127.0.0.1:10080/secure&#x000A;</code></pre>

```
# 出力
{"message":"Hello"}
```

<p>この時点でアプリケーションからレスポンスが返され、すべてが正しく実行されていることがわかります。</p>
<p><code>kubectl logs</code> コマンドを使用して、monolith ポッドのログを表示します。</p>
<pre><code>kubectl logs monolith&#x000A;</code></pre>
<p><strong>3 つ目のターミナル</strong>を開き、<code>-f</code> フラグを使用して、リアルタイムで発生しているログのストリームを取得します。</p>
<pre><code>kubectl logs -f monolith&#x000A;</code></pre>
<p><strong>最初のターミナル</strong>で <code>curl</code> を使用して monolith を操作すると、（<strong>3 つ目のターミナル</strong>で）ログが更新されるのを確認できます。</p>
<pre><code>curl http://127.0.0.1:10080&#x000A;</code></pre>
<p><code>kubectl exec</code> コマンドを使用して、monolith ポッド内で対話型シェルを実行します。これは、コンテナ内からトラブルシューティングするときに便利な場合があります。</p>
<pre><code>kubectl exec monolith --stdin --tty -c monolith /bin/sh&#x000A;</code></pre>
<p>たとえば、monolith コンテナ内にシェルを設定すると、<code>ping</code> コマンドを使用して外部接続をテストできます。</p>
<pre><code>ping -c 3 google.com&#x000A;</code></pre>
<p>この対話型シェルの作業を完了したら、必ずログアウトしてください。</p>
<pre><code>exit&#x000A;</code></pre>
<p>ご覧のように、ポッドの操作は <code>kubectl</code> コマンドを使用するのと同様に簡単です。リモートでコンテナをヒットする必要がある場合、またはログインシェルを取得する必要がある場合、開始して実行するのに必要なものはすべて Kubernetes から提供されます。</p>
<h2 id="step9">サービス</h2>
<p>ポッドは永続的なものではありません。実行チェックや準備チェックに失敗するなど、さまざまな理由で停止または開始することがあり、それによって問題が発生します。</p>
<p>ポッドのセットと通信したい場合にポッドを再起動すると、IP アドレスが変わることがあります。</p>
<p>そのために<a href="http://kubernetes.io/docs/user-guide/services/">サービス</a>が必要になります。サービスは、ポッドに安定したエンドポイントを提供します。</p>
<p><img alt="393e02e1d49f3b37.png" src="https://cdn.qwiklabs.com/Jg0T%2F326ASwqeD1vAUPBWH5w1D%2F0oZn6z5mQ5MubwL8%3D"></p>
<p>サービスはラベルを使用して、どのポッドを操作するかを決定します。ポッドに正しいラベルが付いている場合、サービスによって自動的に取得されて公開されます。</p>
<p>サービスがポッドのセットに提供するアクセスのレベルは、サービスの種類によって異なります。現在、次の 3 つの種類があります。</p>
<ul>
<li>
<code>ClusterIP</code>（内部） -- デフォルトの種類です。このサービスはクラスタ内からのみ見えます。</li>
<li>
<code>NodePort</code> は、クラスタ内の各ノードに外部からアクセス可能な IP を与えます。</li>
<li>
<code>LoadBalancer</code> は、サービスからのトラフィックをクラスタ内のノードに転送するクラウド プロバイダからロードバランサを追加します。</li>
</ul>
<p>ここでは、以下の方法を説明します。</p>
<ul>
<li>
<p>サービスを作成する</p>
</li>
<li>
<p>ラベルセレクタを使用して、限定されたポッドのセットを外部に公開する</p>
</li>
</ul>
<h2 id="step10">サービスの作成</h2>
<p>サービスを作成する前に、まず https トラフィックを処理できる安全なポッドを作成しましょう。</p>
<p>ディレクトリを移動していた場合は、<code>~/orchestrate-with-kubernetes/kubernetes</code> ディレクトリに戻ったことを確認してください。</p>
<pre><code>cd ~/orchestrate-with-kubernetes/kubernetes&#x000A;</code></pre>
<p>monolith サービス構成ファイルを詳しく調べます。</p>
<pre><code>cat pods/secure-monolith.yaml&#x000A;</code></pre>
<p>安全な monolith ポッドとその構成データを作成します。</p>
<pre><code>kubectl create secret generic tls-certs --from-file tls/&#x000A;kubectl create configmap nginx-proxy-conf --from-file nginx/proxy.conf&#x000A;kubectl create -f pods/secure-monolith.yaml&#x000A;</code></pre>
<p>今度は安全なポッドがあるので、安全な monolith ポッドを外部に公開します。そのために、Kubernetes サービスを作成します。</p>
<p>monolith サービス構成ファイルを詳しく調べます。</p>
<pre><code>cat services/monolith.yaml&#x000A;</code></pre>
<p>（出力）:</p>

```yaml
kind: Service
apiVersion: v1
metadata:
  name: "monolith"
spec:
  selector:
    app: "monolith"
    secure: "enabled"
  ports:
    - protocol: "TCP"
      port: 443
      targetPort: 443
      nodePort: 31000
  type: NodePort
```

<p>留意点:</p>
<ol>
<li>「app=monolith」および「secure=enabled」のラベルを持つすべてのポッドを自動的に検出して公開するセレクタが存在します。</li>
<li>この方法により、外部トラフィックをポート 31000 から nginx （ポート 443 上）にどのように転送できるか決まるため、ここでノードポートを公開する必要があります。</li>
</ol>
<p><code>kubectl create</code> コマンドを使用して、monolith サービス構成ファイルから monolith サービスを作成します。</p>
<pre><code>kubectl create -f services/monolith.yaml&#x000A;</code></pre>
<p>（出力）:</p>
<pre><code class="language-bash prettyprint">service "monolith" created&#x000A;</code></pre>
<h3>完了したタスクをテストする</h3>
<p>下の [<strong>進行状況を確認</strong>] をクリックして、ラボの進捗状況を確認します。Monolith ポッドとサービスが正常に作成されている場合は、評価スコアが表示されます。</p>
<ql-activity-tracking step="2">
    Monolith ポッドとサービスを作成する
</ql-activity-tracking>
<p>サービスはポートを使用して公開します。そのため、別の app がユーザーのいずれかのサーバー上のポート 31000 にバインドしようとすると、ポートの衝突が発生する可能性があります。</p>
<p>通常は、Kubernetes がこのポートの割り当てを処理します。このラボでは、後でヘルスチェックを簡単に構成できるポートを選択します。</p>
<p><code>gcloud compute firewall-rules</code> コマンドを使用して、公開されたノードポート上の monolith サービスへのトラフィックを許可します。</p>
<pre><code>gcloud compute firewall-rules create allow-monolith-nodeport \&#x000A;  --allow=tcp:31000&#x000A;</code></pre>
<h3>完了したタスクをテストする</h3>
<p>下の [<strong>進行状況を確認</strong>] をクリックして、ラボの進捗状況を確認します。TCP トラフィックを 31000 ポートで許可するためのファイアウォール ルールが正常に作成されている場合は、評価スコアが表示されます。</p>
<ql-activity-tracking step="3">
    公開されたノードポート上で monolith サービスへのトラフィックを許可する
</ql-activity-tracking>
<p>すべての設定が完了したので、ポート転送を使用せずに、クラスタの外部から安全な monolith サービスをヒットできるはずです。</p>
<p>最初に、1 つのノードの外部 IP アドレスを取得します。</p>
<pre><code>gcloud compute instances list&#x000A;</code></pre>
<p>ここで、<code>curl</code> を使用して安全な monolith サービスをヒットしてみます。</p>
<pre><code>curl -k https://&lt;EXTERNAL_IP&gt;:31000&#x000A;</code></pre>
<p>タイムアウトしました。何が問題でしょうか。</p>
<ql-infobox>
<p>ここで簡単な理解度チェックを行いましょう。次のコマンドを使用して、以下の質問に答えてください。</p>
<p><code>kubectl get services monolith</code></p>
<p><code>kubectl describe services monolith</code></p>
<p><strong>質問:</strong></p>
<ul>
<li>monolith サービスからレスポンスが得られないのはなぜでしょうか。</li>
<li>monolith サービスにはいくつのエンドポイントがありますか。</li>
<li>monolith サービスでは、どのラベルによってポッドがピックアップされる必要がありますか。</li>
</ul>
</ql-infobox>
<p>ヒント: ラベルと関係があります。次のセクションで問題を修正します。</p>
<h2 id="step11">ポッドにラベルを追加する</h2>
<p>現在、monolith サービスにはエンドポイントがありません。このような問題をトラブルシューティングする方法の 1 つは、ラベルクエリで <code>kubectl get pods</code> コマンドを使用することです。</p>
<p>monolith ラベルで実行されているポッドがかなりあることがわかります。</p>
<pre><code>kubectl get pods -l "app=monolith"&#x000A;</code></pre>
<p>しかし、「app = monolith」および「secure = enabled」の場合はどうでしょうか。</p>
<pre><code>kubectl get pods -l "app=monolith,secure=enabled"&#x000A;</code></pre>
<p>このラベルクエリでは結果が出力されません。「secure = enabled」ラベルを追加する必要があるようです。</p>
<p><code>kubectl label</code> コマンドを使用して、安全な monolith ポッドに足りない <code>secure=enabled</code> ラベルを追加します。その後、ラベルが更新されたことを確認します。</p>
<pre><code>kubectl label pods secure-monolith 'secure=enabled'&#x000A;kubectl get pods secure-monolith --show-labels&#x000A;</code></pre>
<p>ポッドに正しくラベルが付けられたので、monolith サービスのエンドポイントのリストを見てみましょう。</p>
<pre><code>kubectl describe services monolith | grep Endpoints&#x000A;</code></pre>
<p>確かに表示されています。</p>
<p>ノードの 1 つをもう一度ヒットすることで、これをテストしてみましょう。</p>
<pre><code>gcloud compute instances list&#x000A;curl -k https://&lt;EXTERNAL_IP&gt;:31000&#x000A;</code></pre>
<p>やりました。ヒューストン、コンタクトがありました。</p>
<h3>完了したタスクをテストする</h3>
<p>下の [<strong>進行状況を確認</strong>] をクリックして、ラボの進捗状況を確認します。monolith ポッドにラベルが正常に追加されている場合は、評価スコアが表示されます。</p>
<ql-activity-tracking step="4">
    ポッドにラベルを追加する
</ql-activity-tracking>
<h2 id="step12">Kubernetes でアプリケーションをデプロイする</h2>
<p>このラボの目標は、本番環境でコンテナのスケーリングと管理を行えるようにすることですが、ここで必要になるのが<a href="http://kubernetes.io/docs/user-guide/deployments/#what-is-a-deployment">デプロイ</a>です。デプロイは、実行中のポッド数とユーザーによって指定された必要なポッド数が同じであることを保証する宣言的な方法です。</p>
<p><img alt="f96989028fa7d280.png" src="https://cdn.qwiklabs.com/1UD7MTP0ZxwecE%2F64MJSNOP8QB7sU9rTI0PSv08OVz0%3D">Deployment の主な利点は、ポッドを管理するときにシステムレベルについて細かく意識する必要がないことです。Deployment は、バックグラウンドで<a href="http://kubernetes.io/docs/user-guide/replicasets/">レプリカセット</a>を使ってポッドの開始と停止を管理します。ポッドの更新またはスケーリングが必要な場合は、Deployment がその処理を行います。ポッドがなんらかの理由で停止した場合、デプロイはポッドの再起動も処理します。</p>
<p>簡単な例を見てみましょう。</p>
<p><img alt="b2e31eed284e4cfe.png" src="https://cdn.qwiklabs.com/fH4ZxGNxg5KLBy5ykbwKNIS9MIJ9cgcMEDuhB0a9uBo%3D"></p>
<p>ポッドは、作成されたノードの存続期間に関連付けられています。上記の例では、Node3 が停止しています（ポッドも停止）。手動で新しいポッドを作成して、そのためのノードを見つける代わりに、デプロイによって新しいポッドが作成されて Node2 で起動されました。</p>
<p>これは良い方法です。</p>
<p>ここで、ポッドとサービスについて学んだことをすべて組み合わせて、デプロイを使って monolith アプリケーションを小さなサービスに分割しましょう。</p>
<h2 id="step13">デプロイを作成する</h2>
<p>monolith アプリを次の 3 つの部分に分割します。</p>
<ul>
<li>
<strong>auth</strong> - 認証済みユーザー用に JWT トークンを生成します。</li>
<li>
<strong>hello</strong> - 認証されたユーザーに挨拶します。</li>
<li>
<strong>frontend</strong> - トラフィックを auth サービスと hello サービスにルーティングします。</li>
</ul>
<p>サービスごとにデプロイを 1 つ作成する準備が整いました。次に、auth デプロイと hello デプロイのための内部サービスと、フロントエンド デプロイのための外部サービスを定義します。これが完了すると、monolith のみの場合と同様にマイクロサービスを操作して、各部分を個別にスケーリングおよびデプロイできるようになります。</p>
<p>最初に、auth デプロイ構成ファイルを検査します。</p>
<pre><code>cat deployments/auth.yaml&#x000A;</code></pre>
<p>（出力）</p>

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: auth
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: auth
        track: stable
    spec:
      containers:
        - name: auth
          image: "kelseyhightower/auth:2.0.0"
          ports:
            - name: http
              containerPort: 80
            - name: health
              containerPort: 81
...
```

<p>デプロイはレプリカを 1 つ作成し、バージョン 2.0.0 の auth コンテナが使用されています。</p>
<p><code>kubectl create</code> コマンドを使用して auth デプロイを作成すると、デプロイ マニフェスト内のデータに準拠するポッドが 1 つ作成されます。これは、replicas フィールドに指定された数を変更することにより、ポッドの数をスケーリングできることを意味しています。</p>
<p>いずれにしても、デプロイ オブジェクトを作成しましょう。</p>
<pre><code>kubectl create -f deployments/auth.yaml&#x000A;</code></pre>
<p>auth デプロイ用のサービスを作成します。ここでは、<code>kubectl create</code> コマンドを使用して auth サービスを作成します。</p>
<pre><code>kubectl create -f services/auth.yaml&#x000A;</code></pre>
<p>次に、同じ手順で hello デプロイを作成して公開します。</p>
<pre><code>kubectl create -f deployments/hello.yaml&#x000A;kubectl create -f services/hello.yaml&#x000A;</code></pre>
<p>もう一度、フロントエンド デプロイを作成して公開します。</p>
<pre><code>kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf&#x000A;kubectl create -f deployments/frontend.yaml&#x000A;kubectl create -f services/frontend.yaml&#x000A;</code></pre>
<ql-infobox>
<p>フロントエンドを作成するためのステップがもう 1 つあります。一部の構成データをコンテナで保存する必要があるからです。</p>
</ql-infobox>
<p>外部 IP を取得し、それに対して curl を実行してフロントエンドを操作します。</p>
<pre><code>kubectl get services frontend&#x000A;curl -k https://&lt;EXTERNAL-IP&gt;&#x000A;</code></pre>
<p>hello レスポンスが返されます。</p>
<h3>完了したタスクをテストする</h3>
<p>下の [<strong>進行状況を確認</strong>] をクリックして、ラボの進捗状況を確認します。Auth、Hello、およびフロントエンド Deployment が正常に作成されている場合は、評価スコアが表示されます。</p>
<ql-activity-tracking step="5">
    Deployment（Auth、Hello、およびフロントエンド）を作成する
</ql-activity-tracking>
<h2 id="step14">まとめ</h2>
<p>これで完了です。Kubernetes を使用してマルチサービス アプリケーションを開発しました。ここで習得したスキルを活用することで、デプロイとサービスのコレクションを使って Kubernetes に複雑なアプリケーションをデプロイできるようになります。</p>

# Deployment Manager - 完全な本番環境
<h2 id="step2">概要</h2>
<p>このラボでは、インフラストラクチャ オーケストレーション ツールである Deployment Manager を使用してサービスを起動し、Cloud Monitoring でサービスをモニタリングします。Cloud Monitoring では、Cloud Monitoring ダッシュボードで基本的なブラック ボックス モニタリングをセットアップし、インシデント対応をトリガーする稼働時間チェック（アラート通知）を設定します。</p>
<p>具体的には、次の作業を行います。</p>
<ol>
<li>Deployment Manager のサンプル テンプレートを使用して高度なデプロイメントをインストール、構成する。</li>
<li>Cloud Monitoring を有効にする。</li>
<li>Cloud Monitoring の稼働時間チェックと通知を構成する。</li>
<li>Cloud Monitoring ダッシュボードに、CPU 使用率と上り（内向きの）トラフィックを示す 2 つのグラフを構成する。</li>
<li>負荷テストを行い、サービスの一時停止をシミュレーションする。</li>
</ol>
<p>Cloud Monitoring では、クラウド対応アプリケーションのパフォーマンス、稼働時間、全体的な動作状況を視覚的に確認できます。Cloud Monitoring は、Google Cloud Platform、Amazon Web Services、ホストされた稼働時間プローブ、アプリケーション インストゥルメンテーション、その他の一般的なアプリケーション コンポーネント（Cassandra、Nginx、Apache Web Server、Elasticsearch など）から、指標、イベント、メタデータを収集し、これらのデータを取り込んでダッシュボード、グラフ、アラートを介して分析情報を提供します。Cloud Monitoring のアラート機能を Slack、PagerDuty、HipChat、Campfire などと統合すると、共同作業に便利です。</p>
<h2 id="step3">目標</h2>
<p>このラボでは、次のタスクの実行方法を学びます。</p>
<ul>
<li>
<p>テンプレートのコレクションからクラウド サービスを開始します。</p>
</li>
<li>
<p>アプリケーションの基本的なブラック ボックス モニタリングを構成します。</p>
</li>
<li>
<p>稼働時間チェックを作成し、サービスの停止を確認します。</p>
</li>
<li>
<p>アラート ポリシーを設定し、インシデント レスポンス手順をトリガーします。</p>
</li>
<li>
<p>グラフを動的に更新し、ダッシュボードを作成および構成します。</p>
</li>
<li>
<p>サービスに負荷を掛けて、モニタリングとアラートの方法をテストします。</p>
</li>
<li>
<p>サービスの停止をシミュレーションし、モニタリングとアラートの方法をテストします。</p>
</li>
</ul>

<h2 id="step5">バーチャル環境を作成する</h2>
<p>次のコマンドを実行して、パッケージ リストをダウンロードし、更新します。</p>
<pre><code>sudo apt-get update</code></pre>
<p>システムからパッケージ インストレーションを分離するため、Python バーチャル環境を使用します。</p>
<pre><code>sudo apt-get install virtualenv</code></pre>
<ql-infobox>[Y/n] が表示されたら、<code>Y</code> と入力し、<code>Enter</code> キーを押します。</ql-infobox>
<pre><code>virtualenv -p python3 venv</code></pre>
<p>バーチャル環境をアクティブにします。</p>
<pre><code>source venv/bin/activate</code></pre>

<h2 id="step6">Deployment Manager のサンプル テンプレートのクローンを作成する</h2>
<p>Google は、学習や構築に確実に使用できる Deployment Manager のサンプル テンプレート セットを提供しています。</p>
<p>リポジトリのクローンを作成するには、以下のコマンドを Cloud Shell コマンドラインに入力し、Deployment Manager サンプル テンプレートを格納するディレクトリを作成します。</p>
<pre><code>mkdir ~/dmsamples&#x000A;</code></pre>
<p>そのディレクトリに移動します。</p>
<pre><code>cd ~/dmsamples&#x000A;</code></pre>
<p>作成したディレクトリにリポジトリのクローンを作成します。</p>
<pre><code>git clone https://github.com/GoogleCloudPlatform/deploymentmanager-samples.git&#x000A;</code></pre>
<p><strong>出力例:</strong></p>
<pre><code class="language-bash prettyprint">remote: Counting objects: 1917, done.&#x000A;remote: Compressing objects: 100% (31/31), done.&#x000A;remote: Total 1917 (delta 11), reused 29 (delta 7), pack-reused 1874&#x000A;Receiving objects: 100% (1917/1917), 426.86 KiB | 0 bytes/s, done.&#x000A;Resolving deltas: 100% (1060/1060), done.&#x000A;</code></pre>
<h2 id="step7">サンプル ファイルを調べる</h2>
<p>サンプル テンプレートのコレクションをディレクトリにダウンロードしました。次は、どのようなファイルがあるか見てみましょう。</p>
<h3>サンプル テンプレートを一覧表示する</h3>
<p>以下のコマンドを実行し、version2 のサンプルを見つけて一覧を表示します。</p>
<pre><code>cd ~/dmsamples/deploymentmanager-samples/examples/v2&#x000A;</code></pre>
<pre><code>ls&#x000A;</code></pre>
<p>次のように表示されます。</p>
<pre><code class="language-bash prettyprint">access_context_manager  cloud_functions  common               folder_creation  iam_custom_role  internal_lb_haproxy  quick_start   ssl                 vm_with_disks&#x000A;access_control          cloudkms         container_igm        gke              igm-updater      metadata_from_file   regional_igm  step_by_step_guide  vpn_auto_subnet&#x000A;bigtable                cloud_router     container_vm         ha-service       image_based_igm  nodejs               saltstack     template_modules    waiter&#x000A;build_configuration     cloudsql         custom_machine_type  htcondor         instance_pool    nodejs_l7            single_vm     vlan_attachment&#x000A;cloudbuild              cloudsql_import  dataproc             iam              internal_lb      project_creation     sqladmin      vm_startup_script&#x000A;</code></pre>
<p>すべてのサブディレクトリが独立したプロジェクトになっているわけではありません。たとえば、<strong>common</strong> という名前のディレクトリには、他の複数のプロジェクトで使用されるテンプレートが含まれています。後で復習するときには、<code>README</code> ファイルをガイドとして使用してください。</p>
<p><code>nodejs</code> ディレクトリに、このラボの実施に必要なものがすべて含まれています。なお、<code>nodejs</code> ディレクトリと <code>nodejs_l7</code> ディレクトリがありますが、ここでは <code>nodejs</code> を使用します。</p>
<h3>nodejs デプロイメントの一覧表示と調査</h3>
<p>version2 のサンプルを見つけて一覧を表示します。</p>
<pre><code>cd nodejs/python&#x000A;</code></pre>
<pre><code>ls&#x000A;</code></pre>
<p><strong>出力例:</strong></p>
<pre><code class="language-bash prettyprint">frontend.py  frontend.py.schema  nodejs.py  nodejs.py.schema  nodejs.yaml&#x000A;</code></pre>
<p>Deployment Manager の主要な構成ファイルは <code>nodejs.yaml</code> です。このファイルで、テンプレートを使用してインフラストラクチャを生成します。残りのファイルはテンプレートです。テンプレートでは、<code>nodejs</code>.<code>yaml</code> 構成ファイルに定義されている変数を使ってカスタマイズされた結果を生成します。</p>
<p><img alt="bb0ba9fa2750ba43.png" src="https://cdn.qwiklabs.com/40UqXM4XpfPlZ%2BOQWaVadL5q1cNctNoC2OHvAS3pxsw%3D"></p>
<h3>frontend.py </h3>
<p><code>frontend.py</code> には <code>frontend.py.schema</code> が含まれ、これにより <code>container_instance_template.py</code> に基づいてインスタンス テンプレートが作成されます。このテンプレートから、マネージド インスタンス グループとオートスケーラーが作成されます。また、単一の公開 IP アドレスによる転送ルールを持つネットワーク ロードバランサも作成されます。以下も作成されます。</p>
<ul>
<li>
<p>マネージド インスタンス グループを参照するターゲット プール。</p>
</li>
<li>
<p>ターゲット プールにアタッチされたヘルスチェック。</p>
</li>
</ul>
<h3>nodejs.py</h3>
<p><code>nodejs.py</code> には <code>nodejs.py.schema</code> が含まれ、これによりフロントエンドとバックエンドのテンプレートが結びつけられます。</p>
<ul>
<li>
<p>フロントエンドは <code>frontend.py</code> です。</p>
</li>
<li>
<p>バックエンドは <code>/common/python/container_vm.py</code> です。</p>
</li>
<li>
<p>これは、MySQL を持つ Docker コンテナを実行している VM であるため、カスタム テンプレートは必要ありません。</p>
</li>
</ul>
<h3>その他のファイル</h3>
<ul>
<li>
<p><code>/common/python/container_instance_template.py</code></p>
</li>
<li>
<p><code>/common/python/container_vm.py</code></p>
</li>
<li>
<p><code>/common/python/container_helper.py</code></p>
</li>
</ul>
<h2 id="step8">デプロイメントをカスタマイズする</h2>
<p><code>nodejs</code> Deployment Manager テンプレートをダウンロードして内容を確認しました。次は、デプロイメントをカスタマイズしてみましょう。</p>
<h3>ゾーンの指定</h3>
<p><code>nodejs.yaml</code> ファイルにはゾーンが必要です。ここで、ファイルにゾーンを追加します。</p>
<ol>
<li>
<p>次のコマンドを入力し、ゾーンのリストを表示します。</p>
</li>
</ol>
<pre><code>gcloud compute zones list&#x000A;</code></pre>
<p>構成ファイルで使用するゾーンの名前をコピーします。</p>
<ol start="2">
<li>
<p><code>nano</code> で <code>nodejs.yaml</code> を開きます。これで、<code>zone</code> の値を編集できます。</p>
</li>
</ol>
<pre><code>nano nodejs.yaml&#x000A;</code></pre>
<p><code>nodejs.yaml</code> ファイルには、以下が含まれています。</p>
<pre><code class="language-bash prettyprint">resources:&#x000A;- name: nodejs&#x000A;  type: nodejs.py&#x000A;  properties:&#x000A;    zone: ZONE_TO_RUN&#x000A;</code></pre>
<ol start="3">
<li>
<code>ZONE_TO_RUN</code> の部分を近くのゾーンの名前に置き換えてから、nano を終了してファイルを保存します。</li>
</ol>
<p>この例では、<code>ZONE_TO_RUN</code> が <code>us-east1-d</code> に設定されています。</p>
<pre><code class="language-bash prettyprint">resources:&#x000A;- name: nodejs&#x000A;  type: nodejs.py&#x000A;  properties:&#x000A;    zone: us-east1-d&#x000A;</code></pre>
<h3>インスタンス グループ内のインスタンス最大数を変更する</h3>
<p><code>nodejs.py</code> ファイルを編集します。</p>
<ol>
<li>
<p>次のコマンドを入力して、<code>nano</code> で <code>nodejs.py</code> を開きます。</p>
</li>
</ol>
<pre><code>nano nodejs.py&#x000A;</code></pre>
<ol start="2">
<li>
<p><code>nodejs.py</code> でフロントエンドの現在のスケーリングの制限を確認します。</p>
</li>
</ol>
<pre><code>{&#x000A;     'name': frontend,&#x000A;     'type': 'frontend.py',&#x000A;     'properties': {&#x000A;         'zone': context.properties['zone'],&#x000A;         'dockerImage': 'gcr.io/deployment-manager-examples/nodejsservice',&#x000A;         'port': application_port,&#x000A;         # コンテナに公開されている変数を env 変数として定義&#x000A;         'dockerEnv': {&#x000A;             'SEVEN_SERVICE_MYSQL_PORT': mysql_port,&#x000A;             'SEVEN_SERVICE_PROXY_HOST': '$(ref.' + backend&#x000A;                                         + '.networkInterfaces[0].networkIP)'&#x000A;         },&#x000A;         # 未入力の場合のデフォルトは 1&#x000A;         'size': 2,&#x000A;         # 未入力の場合のデフォルトは 1&#x000A;         'maxSize': 20&#x000A;     }&#x000A; },&#x000A;</code></pre>
<p>現在のスケーリングの制限は 20 です（maxSize を参照）。</p>
<ol start="3">
<li>
<p><code>maxSize</code> を変更して 4 に設定します。</p>
</li>
</ol>
<pre><code>{&#x000A;     'name': frontend,&#x000A;     'type': 'frontend.py',&#x000A;     'properties': {&#x000A;         'zone': context.properties['zone'],&#x000A;         'dockerImage': 'gcr.io/deployment-manager-examples/nodejsservice',&#x000A;         'port': application_port,&#x000A;         # コンテナに公開されている変数を env 変数として定義&#x000A;         'dockerEnv': {&#x000A;             'SEVEN_SERVICE_MYSQL_PORT': mysql_port,&#x000A;             'SEVEN_SERVICE_PROXY_HOST': '$(ref.' + backend&#x000A;                                         + '.networkInterfaces[0].networkIP)'&#x000A;         },&#x000A;         # 未入力の場合のデフォルトは 1&#x000A;         'size': 2,&#x000A;         # 未入力の場合のデフォルトは 1&#x000A;         'maxSize': 4&#x000A;     }&#x000A; },&#x000A;</code></pre>
<p>作業が終わったらファイルを保存し、nano を終了します。</p>
<h2 id="step9">アプリケーションを実行する</h2>
<p>次は、Deployment Manager を使用してアプリケーションをデプロイし、動作可能にします。これによってインフラストラクチャが構築されますが、トラフィックは許可されません。Deployment Manager によってインフラストラクチャがセットアップされた後、サービスラベルを適用できます。</p>
<h3>アプリケーションをデプロイする</h3>
<p>最初に、次のコマンドを入力してアプリケーション <code>advanced-configuration</code> を指定し、Deployment Manager に構成ファイル（<code>nodejs.yaml</code>）を渡します。</p>
<pre><code>gcloud deployment-manager deployments create advanced-configuration --config nodejs.yaml&#x000A;</code></pre>
<p>出力:</p>
<pre><code class="language-bash prettyprint">The fingerprint of the deployment is PiYc6OsIFkWzQpCDklHvaA==&#x000A;Waiting for create [operation-1529913842103-56f72d31872d9-90070017-aec5761d]...done.&#x000A;Create operation operation-1529913842103-56f72d31872d9-90070017-aec5761d completed successfully.&#x000A;NAME                                   TYPE                             STATE      ERRORS  INTENT&#x000A;advanced-configuration-application-fw  compute.v1.firewall              COMPLETED  []&#x000A;advanced-configuration-backend         compute.v1.instance              COMPLETED  []&#x000A;advanced-configuration-frontend-as     compute.v1.autoscaler            COMPLETED  []&#x000A;advanced-configuration-frontend-hc     compute.v1.httpHealthCheck       COMPLETED  []&#x000A;advanced-configuration-frontend-igm    compute.v1.instanceGroupManager  COMPLETED  []&#x000A;advanced-configuration-frontend-it     compute.v1.instanceTemplate      COMPLETED  []&#x000A;advanced-configuration-frontend-lb     compute.v1.forwardingRule        COMPLETED  []&#x000A;advanced-configuration-frontend-tp     compute.v1.targetPool            COMPLETED  []&#x000A;</code></pre>
<p>下の [<strong>進捗状況を確認</strong>] をクリックして、ラボの進捗状況を確認します。</p>
<ql-activity-tracking step="1">
    アプリケーションをデプロイする
</ql-activity-tracking>
<p>インスタンス数の上限を確認します。</p>
<ol>
<li>
<p>[<strong>Compute Engine</strong>] &gt; [<strong>インスタンス グループ</strong>] に移動します。</p>
<p><img alt="ComputeEngine_InstanceGroups.png" src="https://cdn.qwiklabs.com/VSe6QJCURyuBRtdwI1VmPbmtVO6lr%2BMBOVqSxBQ%2FR1o%3D"></p>
</li>
<li>
<p>[<strong>advanced-configuration-frontend-igm</strong>] をクリックします。<img alt="11990217fdad313e.png" src="https://cdn.qwiklabs.com/%2Bp3hZFvK7pLEz8wDzim4v9wdElPNp53neYOUXYA0OV0%3D"></p>
</li>
<li>
<p>[詳細] タブをクリックし、インスタンスの最大数を確認します。</p>
<img alt="max-num.png" src="https://cdn.qwiklabs.com/P%2BEn0rirUUkLCkoMCWp2QJtHtGjaVqydNJNAphyJNZI%3D">
</li>
</ol>
<p>4 に設定されているはずです。</p>
<h2 id="step10">アプリケーションが動作していることを確認する</h2>
<p>アプリケーションの起動には数分かかります。アプリケーションの動作は Cloud Console の Deployment Manager（<strong>ナビゲーション メニュー</strong> &gt; [<strong>Deployment Manager</strong>]）で確認できます。あるいは、Console の Compute Engine（<strong>ナビゲーション メニュー</strong> &gt; [<strong>Compute Engine</strong>] &gt; [<strong>VM インスタンス</strong>]）でインスタンスを確認することもできます。</p>
<p>アプリケーションが実行されていることを確認するには、ブラウザを開き、<code>port 8080</code> にアクセスしてサービスを表示します。IP アドレスは Deployment Manager が（テンプレートで指定されている）グローバル転送ルールを実装したときに動的に確立されているため、そのアドレスを見つけてアプリケーションをテストする必要があります。</p>
<h3>グローバル ロードバランサ転送ルールの IP アドレスを見つける</h3>
<ol>
<li>
<p>Cloud コマンドラインに以下のコマンドを入力し、転送用 IP アドレスを見つけます。</p>
</li>
</ol>
<pre><code>gcloud compute forwarding-rules list&#x000A;</code></pre>
<p><strong>出力例:</strong></p>
<p><img alt="369c66095dd87ce8.png" src="https://cdn.qwiklabs.com/iTFyoWP0LBGcSOSQLoSTyfXuGX7UQdYCQNJLoXu415Q%3D">
転送用 IP アドレスは、出力に一覧表示された IP アドレスです。このラボでは何回かこのアドレスを使用するため、書き留めておいてください。</p>
<ol start="2">
<li>
<p>ブラウザで新しいウィンドウを開きます。アドレス フィールドに以下の URL を入力します。<code>&lt;your IP address&gt;</code> の部分は、転送用 IP アドレスに置き換えてください。</p>
</li>
</ol>
<pre><code>http://&lt;your IP address&gt;:8080&#x000A;</code></pre>
<p>以下のような空白ページに、ご自分の IP アドレスが表示されます。</p>
<p><img alt="b0313bc2bc83fd85.png" src="https://cdn.qwiklabs.com/STpmChPe39xBddGAasxfBm%2BTsT5A425F6HG%2FMKQeBz0%3D"></p>
<p>サービスが動作するまで、最大で 10 分かかることがあります。404 などのエラーが発生した場合や、タイムアウトになった場合は、2 分ほど待ってから再試行してください。</p>
<ol start="3">
<li>
<p>次に、ブラウザのアドレス行にログメッセージを追加して、ログ情報を入力します。</p>
</li>
</ol>
<pre><code>http://&lt;your forwarding IP address&gt;:8080/?msg=&lt;enter_a_message&gt;&#x000A;</code></pre>
<p><code>enter_a_message</code> の部分は、任意のメッセージに置き換えてください。</p>
<p>以下はログメッセージの例です（実際の IP アドレスは異なります）。</p>
<pre><code class="language-bash prettyprint">http://35.196.56.153:8080/?msg=my dog has spots&#x000A;</code></pre>
<p><img alt="dcbc4d400e96e650.png" src="https://cdn.qwiklabs.com/BaDXAVrIgQau%2F%2F1ym99jFfrWBd8xwAn7qZnOYGEgkLY%3D"></p>
<p>ログを入力すると、ブラウザから <code>added</code> が返されます。</p>
<p><img alt="d816761e8572a251.png" src="https://cdn.qwiklabs.com/9iHvic1MnaKJUnI%2Bf1pwgTnIdhHnYEcWLiGexXu5IJA%3D"></p>
<ol start="4">
<li>
<code>http://&lt;your IP address&gt;:8080</code> に移動してログを表示します。次に例を示します。</li>
</ol>
<p><img alt="7ce7917b8519c96d.png" src="https://cdn.qwiklabs.com/m7fbmMH6Z5acpT9%2F2ueDX5hOVMtdP7tlqAs0Axmj34I%3D"></p>
<p>作業を続行してさらにログを作成し、<code>http://&lt;your IP address&gt;:8080</code> で確認します。</p>
<h2 id="step11">Cloud Monitoring で稼働時間チェックとアラート ポリシーを構成する</h2>
<h3>モニタリング ワークスペースを作成する</h3>
<p>ここで、Qwiklabs GCP プロジェクトに関連付けられたモニタリング ワークスペースを設定します。次の手順にて、モニタリングの無料トライアル付きの新しいアカウントを作成します。</p>
<p>1. Google Cloud Platform コンソールで、[<strong>ナビゲーション メニュー </strong>] &gt; [<strong>モニタリング</strong>] をクリックします。</p>
<p>2. ワークスペースがプロビジョニングされるのを待ちます。</p>
<p>モニタリング ダッシュボードが開いたら、モニタリング ワークスペースが利用できます。</p>
<p><img alt="monitoring_overview.png" src="https://cdn.qwiklabs.com/BGS8QF9AXRWwP0pKwugIg7rSBz9hP%2B2r88V4czj0m4o%3D"></p>

<h3>稼働時間チェックを構成する</h3>
<p>Cloud Monitoring ウィンドウが開きます。</p>
<ol>
<li>[Cloud Monitoring] タブで [<strong>稼働時間チェック</strong>] をクリックします。</li>
<li>[<strong>稼働時間チェックの作成</strong>] をクリックします。</li>
<li>[<strong>New uptime check</strong>] ページで次のプロパティ値を指定します。</li>
</ol>
<table>

<tr>
<th>プロパティ</th>
<th>値</th>
</tr>


<tr>
<td>Title</td>
<td>Check1</td>
</tr>
<tr>
<td>Check Type</td>
<td>TCP</td>
</tr>
<tr>
<td>Resource Type</td>
<td>URL</td>
</tr>
<tr>
<td>Hostname</td>
<td>
<code>&lt;転送先アドレス&gt;</code>（記録しておいたアドレス）</td>
</tr>
<tr>
<td>Port</td>
<td>8080</td>
</tr>
<tr>
<td>Response Content</td>
<td><code>&lt;空白のまま&gt;</code></td>
</tr>
<tr>
<td>Check every</td>
<td>1 minute</td>
</tr>

</table>
<ol start="4">
<li>[<strong>TEST</strong>] をクリックしてチェックをテストします。</li>
</ol>
<p><img alt="7f60fe39e946f680.png" src="https://cdn.qwiklabs.com/eay9YeJ5k9vIdLIv62KrdE3aOgp3kCbtwGZE3zwAnZI%3D"></p>
<aside class="special"><p>テストに失敗した場合は、サービスが稼働中であることを確認します。また、ファイアウォール ルールが存在し、正しく設定されていることも確認します。</p>
</aside>
<ol start="5">
<li>テストに成功したら、[<strong>SAVE</strong>] をクリックして保存します。</li>
</ol>
<p>稼働時間チェックを保存すると、Cloud Monitoring によってアラート ポリシーの作成が提案されます。[<strong>No, Thanks</strong>] をクリックします。</p>
<p><img alt="bfe4607c7305436d.png" src="https://cdn.qwiklabs.com/5P8EvAIM64KkuQrUlbLCndLuh85pVz%2BUrIX6MpBRhT0%3D"></p>
<h2 id="step12">アラート ポリシーと通知を構成する</h2>
<ol>
<li>
<p>[<strong>アラート</strong>] &gt; [<strong>CREATE POLICY</strong>] をクリックし、新しいポリシーを作成します。</p>
</li>
<li>
<p>ポリシーに「<strong>Test</strong>」という名前を付けます。</p>
</li>
<li>
<p>条件を追加するために [<strong>ADD CONDITION</strong>] をクリックします。</p>
</li>
<li>
<p>[<strong>Find resource type and metric</strong>] で、リソースの種類として [<strong>GCE VM Instance</strong>] を選択します。</p>
</li>
<li>
<p><strong>Metrics</strong> として、[<strong>CPU usage</strong>] や [<strong>CPU utilization</strong>] などの評価対象の指標を選択します。</p>
</li>
<li>
<p>[<strong>Condition</strong>] で、[<strong>is above</strong>] を選択します。</p>
</li>
<li>
<p>しきい値と、この指標が設定値に達しない場合にアラートをトリガーするまでの期間を指定します。たとえば、[<strong>Threshold</strong>] に「<strong>20</strong>」と入力し、[<strong>For</strong>] には [<strong>1 minute</strong>] を設定します。</p>
</li>
<li>
<p>[<strong>ADD</strong>] をクリックして追加します。</p>
</li>
<li>
<p>[<strong>Notifications</strong>] で [<strong>ADD NOTIFICATION CHANNEL</strong>] をクリックします。[Notification Channel Type] として [Email] を選択し、自分のメールアドレスを追加します。[<strong>ADD</strong>] をクリックします。</p>
<p>後のステップで、メールで通知されるイベントをトリガーします。</p>
</li>
<li>
<p>[<strong>SAVE</strong>] をクリックして保存します。</p>
</li>
</ol>
<p>下の [<strong>進捗状況を確認</strong>] をクリックして、ラボの進捗状況を確認します。</p>
<ql-activity-tracking step="2">
    Cloud Monitoring で稼働時間チェックとアラート ポリシーを構成する
</ql-activity-tracking>
<h2 id="step13">便利なグラフをダッシュボードで構成する</h2>
<h3>ダッシュボードを構成する</h3>
<ol>
<li>
<p>左側のメニューで、[<strong>ダッシュボード</strong>] &gt; [<strong>CREATE DASHBOARD</strong>] をクリックします。</p>
</li>
<li>
<p>新しいダッシュボードに「<strong>DMDash</strong>」という名前を付けてから [<strong>確認</strong>] をクリックします。</p>
</li>
<li>
<p>右上にある [<strong>ADD CHART</strong>] をクリックし、新しいグラフを追加します。</p>
<p><img alt="ef98bfe7497869ed.png" src="https://cdn.qwiklabs.com/7mqY6WhK4LAtBMDffF1udM%2FkvqUHeJJPO1IL1dV3iok%3D"></p>
</li>
<li>
<p>次のプロパティ値を設定します。</p>
</li>
</ol>
<table>

<tr>
<th>プロパティ</th>
<th>値</th>
</tr>


<tr>
<td>Title</td>
<td>Sample</td>
</tr>
<tr>
<td>Resource type</td>
<td>GCE VM Instance</td>
</tr>
<tr>
<td>Metric Type</td>
<td>CPU Usage</td>
</tr>

</table>
<ol start="5">
<li>[<strong>Save</strong>] をクリックします。</li>
<li>[<strong>Add Chart</strong>] をクリックし、以下のプロパティを設定して、ダッシュボードに別のグラフを追加します。</li>
</ol>
<table>

<tr>
<th>プロパティ</th>
<th>値</th>
</tr>


<tr>
<td>Title</td>
<td>Sample 2</td>
</tr>
<tr>
<td>Resource type</td>
<td>GCE VM Instance</td>
</tr>
<tr>
<td>Metric Type</td>
<td>Sent bytes</td>
</tr>

</table>
<ol start="7">
<li>
<p>[<strong>SAVE</strong>] をクリックします。保存された DMDash は以下のようになります。</p>
<p><img alt="8b18deae69fb8132.png" src="https://cdn.qwiklabs.com/NO8Ga0SKlB8dId7XxLbrRrVrA3DDYObh8xGsfoPPdOM%3D"></p>
</li>
</ol>
<h2 id="step14">ApacheBench を使用してテスト用 VM を作成する</h2>
<p>指定したリージョン内のトラフィック用にモニタリングを構成したので、それが機能するか確認してみましょう。ApacheBench をインストールし、それを使ってサービスに 3 段階の負荷をかけ、セットアップした Cloud Monitoring ダッシュボードを確認します。</p>
<h3>VM を作成する</h3>
<ol>
<li>
<p>Cloud Console で、[<strong>Compute Engine</strong>] &gt; [<strong>VM インスタンス</strong>] をクリックします。</p>
<p><img alt="cf68a77143674661.png" src="https://cdn.qwiklabs.com/vbSlqad%2BVrl178fWsGAaH%2BmPyCI%2Fdjo0hGtwvn793mg%3D"></p>
</li>
<li>
<p>[<strong>インスタンスを作成</strong>] をクリックし、[<strong>インスタンスの作成</strong>] ダイアログですべてデフォルト設定を使用します。</p>
<p><img alt="Create_Instance.png" src="https://cdn.qwiklabs.com/J%2BpSMLd%2FmiylLCHiefiff22NU0a9k1S1SuI8Du16%2F50%3D"></p>
</li>
<li>
<p>[<strong>作成</strong>] をクリックします。</p>
</li>
</ol>
<h3>ApacheBench をインストールする</h3>
<ol>
<li>
<p>[VM インスタンス] ウィンドウで、[instance-1] の [<strong>SSH</strong>] ボタンをクリックして、作成した VM に SSH 接続します。</p>
</li>
<li>
<p>以下のコマンドを入力し、最新の ApacheBench をインストールします。</p>
</li>
</ol>
<pre><code>sudo apt-get update&#x000A;</code></pre>
<pre><code>sudo apt-get -y install apache2-utils&#x000A;</code></pre>
<p>下の [<strong>進捗状況を確認</strong>] をクリックして、ラボの進捗状況を確認します。</p>
<ql-activity-tracking step="3">
    ApacheBench を使用してテスト用 VM を作成する
</ql-activity-tracking>
<h3>負荷をかけてモニタリングする</h3>
<p>次は、ApacheBench を使用してサービスに負荷をかけます。ここでは、Cloud Monitoring の DMDash ダッシュボードで CPU 使用率とネットワーク受信トラフィックをモニタリングします。Cloud Monitoring ではインスタンスの数も追跡できます。それには、行にカーソルを合わせるか、Cloud Console でインスタンスを表示します（<strong>ナビゲーション メニュー</strong> &gt; [<strong>Compute Engine</strong>] &gt; [<strong>VM インスタンス</strong>]）。</p>
<ol>
<li>
<p>SSH ウィンドウで次のコマンドを入力し、ApacheBench でサービスに負荷をかけます。<code>Your_IP</code> の部分は、転用用 ID（記録しておいたもの）に置き換えてください。次のコマンドを 2、3 回実行し、トラフィックを生成します。</p>
</li>
</ol>
<pre><code>ab -n 1000 -c 100 http://&lt;Your_IP&gt;:8080/&#x000A;</code></pre>
<p><strong>出力例:</strong></p>
<pre><code>This is ApacheBench, Version 2.3 &lt;$Revision: 1757674 $&gt;&#x000A;Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/&#x000A;Licensed to The Apache Software Foundation, http://www.apache.org/&#x000A;&#x000A;Benchmarking 35.196.195.26 (be patient)&#x000A;Completed 100 requests&#x000A;Completed 200 requests&#x000A;Completed 300 requests&#x000A;Completed 400 requests&#x000A;Completed 500 requests&#x000A;Completed 600 requests&#x000A;Completed 700 requests&#x000A;Completed 800 requests&#x000A;Completed 900 requests&#x000A;Completed 1000 requests&#x000A;Finished 1000 requests&#x000A;&#x000A;Server Software:&#x000A;Server Hostname:        35.196.195.26&#x000A;Server Port:            8080&#x000A;&#x000A;Document Path:          /&#x000A;Document Length:        40 bytes&#x000A;&#x000A;Concurrency Level:      100&#x000A;Time taken for tests:   0.824 seconds&#x000A;Complete requests:      1000&#x000A;Failed requests:        0&#x000A;Total transferred:      140000 bytes&#x000A;HTML transferred:       40000 bytes&#x000A;Requests per second:    1213.57 [#/sec] (mean)&#x000A;Time per request:       82.402 [ms] (mean)&#x000A;Time per request:       0.824 [ms] (mean, across all concurrent requests)&#x000A;Transfer rate:          165.92 [Kbytes/sec] received&#x000A;&#x000A;Connection Times (ms)&#x000A;              min  mean[+/-sd] median   max&#x000A;Connect:       36   37   0.5     37      40&#x000A;Processing:    36   43   8.7     38      73&#x000A;Waiting:       36   43   8.7     38      73&#x000A;Total:         73   80   9.1     75     112&#x000A;&#x000A;Percentage of the requests served within a certain time (ms)&#x000A;  50%     75&#x000A;  66%     78&#x000A;  75%     81&#x000A;  80%     84&#x000A;  90%     95&#x000A;  95%    101&#x000A;  98%    106&#x000A;  99%    110&#x000A; 100%    112 (longest request)&#x000A;</code></pre>
<p>数分間待って、負荷を 5,000 まで増やします。</p>
<ol start="2">
<li>
<p>次のコマンドを 2、3 回実行してトラフィックを生成します。</p>
</li>
</ol>
<pre><code>ab -n 5000 -c 100 http://&lt;Your_IP&gt;:8080/&#x000A;</code></pre>
<p>数分間待って、負荷を 10,000 まで増やします。</p>
<ol start="3">
<li>
<p>次のコマンドを 2、3 回実行してトラフィックを生成します。</p>
</li>
</ol>
<pre><code>ab -n 10000 -c 100 http://&lt;Your_IP&gt;:8080/&#x000A;</code></pre>
<p>今度は、インスタンスごとの CPU 使用率を下げるとどうなるか確認します。</p>
<ol>
<li>Cloud Console で、<strong>ナビゲーション メニュー</strong> &gt; [<strong>Compute Engine</strong>] &gt; [<strong>インスタンス グループ</strong>] をクリックします。</li>
<li>インスタンス グループの名前をクリックし、[詳細] タブをクリックしてから、[<strong>グループを編集</strong>] をクリックします。</li>
<li>CPU 使用率を変更します。[<strong>CPU 使用率</strong>] をクリックし、[<strong>ターゲットの CPU 使用率</strong>] を <strong>20</strong> に変更してから [<strong>完了</strong>] をクリックします。</li>
<li>[<strong>保存</strong>] をクリックします。</li>
</ol>
<aside class="special"><p>ターゲットの CPU 使用率は、インスタンス グループ内のすべての VM に対する CPU 使用率の合計値です。これによって、自動スケーリングが実行されるタイミングが制御されます。通常、本番環境ではこの値を 60% 以上に設定しますが、この演習では短時間で自動スケーリングを確認できるよう 20% に設定します。</p>
</aside>
<p>次のコマンドを 2、3 回実行してトラフィックを生成します。</p>
<pre><code>ab -n 10000 -c 100 http://&lt;Your_IP&gt;:8080/&#x000A;</code></pre>
<p><strong>予想される挙動:</strong> 負荷によりグループ内の累積 CPU 消費量が 20% を超え、自動スケーリングがトリガーされます。そのため、新しいインスタンスが開始されています。</p>
<p>次に、自動スケーリングをオフにするとどうなるか確認します。</p>
<ol>
<li>[<strong>Compute Engine</strong>] &gt; [<strong>インスタンス グループ</strong>] に移動します。</li>
<li>インスタンス グループの名前をクリックし、[<strong>グループを編集</strong>] をクリックします。</li>
<li>[<strong>自動スケーリング モード</strong>] を [<strong>自動スケーリングしない</strong>] に変更します。</li>
<li>[<strong>保存</strong>] をクリックします。</li>
</ol>
<p>数分間待ってから、次のコマンドを 2、3 回実行してトラフィックを生成します。</p>
<pre><code>ab -n 10000 -c 100 http://&lt;Your_IP&gt;:8080/&#x000A;</code></pre>
<p><strong>予想される挙動:</strong> 自動スケーリングをオフにすると、新しいインスタンスは作成されず、累積 CPU 使用率が増加します。</p>
<h3>結果</h3>
<p>Cloud Monitoring ダッシュボードで変更を確認できるようになるまで、5 分ほどかかります。</p>
<h2 id="step15">サービスの一時停止をシミュレーションする</h2>
<p>一時停止をシミュレーションするために、ファイアウォールを削除します。</p>
<ol>
<li>
<p><strong>ナビゲーション メニュー</strong> &gt; [<strong>VPC ネットワーク</strong>] &gt; [<strong>ファイアウォール ルール</strong>] をクリックします。</p>
</li>
<li>
<p><strong>tcp:8080</strong> が含まれるファイアウォール ルールの横にあるチェックボックスをオンにしてから、ページの上部にある [<strong>削除</strong>] をクリックします。もう一度 [<strong>削除</strong>] をクリックして確定します。</p>
</li>
<li>
<p>15～30 分で通知メールが届きます。</p>
</li>
</ol>

# Spinnaker と Kubernetes Engine を使用した継続的デリバリー パイプライン
<p>このハンズオンラボでは、Google Kubernetes Engine、Google Cloud Source Repositories、Google Cloud Container Builder、Spinnaker を使用して継続的デリバリー パイプラインを作成する方法を示します。サンプル アプリケーションを作成したら、そのサービスを自動的にビルド、テスト、デプロイするように構成します。アプリケーション コードを変更すると、継続的デリバリー パイプラインがトリガーされ、新しいバージョンが自動的に再ビルド、再テスト、再デプロイされます。</p>
<h2 id="step2">目標</h2>
<ul>
<li>
<p><a href="https://cloud.google.com/shell/">Google Cloud Shell</a> を起動して、Kubernetes Engine クラスタを作成し、ID とユーザー管理スキームを構成することによって、環境をセットアップします。</p>
</li>
<li>
<p>サンプル アプリケーションをダウンロードして、Git レポジトリを作成してから、Google Cloud Source Repositories のレポジトリにアップロードします。</p>
</li>
<li>
<p><a href="https://github.com/kubernetes/helm">Helm</a> を使用して Spinnaker を Kubernetes Engine にデプロイします。</p>
</li>
<li>
<p>Docker イメージをビルドします。</p>
</li>
<li>
<p>アプリケーションが変更されたときに Docker イメージを作成するためのトリガーを作成します。</p>
</li>
<li>
<p>アプリケーションを Kubernetes Engine に確実かつ継続的にデプロイするように Spinnaker パイプラインを構成します。</p>
</li>
<li>
<p>コード変更をデプロイしてパイプラインをトリガーし、本番環境へのロールアウトを監視します。</p>
</li>
</ul>
<h2 id="step3">パイプライン アーキテクチャ</h2>
<p>アプリケーション更新を継続的にユーザーに配布するには、ソフトウェアを確実にビルド、テスト、更新する自動プロセスが必要です。コード変更は、アーティファクトの作成、単体テスト、機能テスト、本番環境ロールアウトで構成されたパイプラインを自動的に通過する必要があります。場合によっては、コード更新をユーザーベース全体に配布する前にユーザーのサブセットにのみ適用し、実際の動作を確認したいケースもあるでしょう。これらの<a href="https://martinfowler.com/bliki/CanaryRelease.html">カナリアテスト</a>のいずれかで不十分なことが判明した場合は、ソフトウェア変更を自動的な手順ですばやくロールバックできる必要があります。</p>
<p><img alt="8cda078d8123f982.png" src="https://cdn.qwiklabs.com/3W%2FV9FteNU4TA%2BCcIV81TBodme5yGTuaxcf%2BR0NtcH4%3D"></p>
<p>Kubernetes Engine と Spinnaker を使用すると、堅牢な継続的デリバリー フローを作成できます。これにより、ソフトウェアを開発して検証したらすぐに出荷できるようになります。迅速な繰り返しが最終目標ですが、各アプリケーション リビジョンが本番環境ロールアウトの候補になる前に、必ず自動検証をすべて通過する必要があります。特定の変更を自動化によって検査してから、アプリケーションを手動で検証し、さらにプレリリース テストを実施することもできます。</p>
<p>アプリケーションが本番環境用として準備が整ったとチームが判断したら、チームメンバーの 1 人が本番環境デプロイメント用として承認できます。</p>
<h3>アプリケーション デリバリー パイプライン</h3>
<p>このラボでは、次の図に示す継続的デリバリー パイプラインを構築します。</p>
<p><img alt="7a1802b4383b4905.png" src="https://cdn.qwiklabs.com/PJktFlNGrmyiEiLlnCrd%2B2r%2FggA3LaUDPQn5frY7ikc%3D"></p>

<h2 id="step5">環境の設定</h2>
<p>このラボに必要なインフラストラクチャと ID を構成します。最初に、Spinnaker とサンプル アプリケーションをデプロイするための Kubernetes Engine クラスタを作成します。</p>
<ol>
<li>
<p>デフォルトのコンピューティング ゾーンを設定します。</p>
</li>
</ol>
<pre><code>gcloud config set compute/zone us-central1-f&#x000A;</code></pre>
<ol start="2">
<li>
<p>Spinnaker チュートリアルのサンプル アプリケーションを使用して Kubernetes Engine を作成します。</p>
</li>
</ol>
<pre><code>gcloud container clusters create spinnaker-tutorial \&#x000A;    --machine-type=n1-standard-2&#x000A;</code></pre>
<p>このデプロイは完了するまでに <strong>5～10 分</strong>程度かかります。デフォルトのスコープに関する警告が表示される場合がありますが、このラボには影響しないため無視して構いません。デプロイが完了したら操作を続行します。</p>
<p>デプロイが完了すると、クラスタの詳細（名前、ロケーション、バージョン、IP アドレス、マシンタイプ、ノード バージョン、ノード数、実行中であることを示すステータス）が記載されたレポートが表示されます。</p>
<h3>ID およびアクセス管理を構成する</h3>
<p>Cloud Identity Access Management（Cloud IAM）<a href="https://cloud.google.com/iam/docs/service-accounts" target="_blank">サービス アカウント</a>を作成して Spinnaker に権限を委任し、Cloud Storage にデータを保存できるようにします。Spinnaker は、パイプライン データを Cloud Storage に保存することで、信頼性と復元力を確保します。Spinnaker のデプロイメントが予期せず失敗した場合は、オリジナルと同じパイプライン データへのアクセス権を持つ同じデプロイメントを数分で作成できます。</p>
<p>次の手順で起動スクリプトを Cloud Storage バケットにアップロードします。</p>
<ol start="3">
<li>
<p>サービス アカウントを作成します。</p>
</li>
</ol>
<pre><code>gcloud iam service-accounts create spinnaker-account \&#x000A;    --display-name spinnaker-account&#x000A;</code></pre>
<ol start="4">
<li>
<p>後のコマンドで使用するために、サービス アカウントのメールアドレスと現在のプロジェクト ID を環境変数に格納します。</p>
</li>
</ol>
<pre><code>export SA_EMAIL=$(gcloud iam service-accounts list \&#x000A;    --filter="displayName:spinnaker-account" \&#x000A;    --format='value(email)')&#x000A;</code></pre>
<pre><code>export PROJECT=$(gcloud info --format='value(config.project)')&#x000A;</code></pre>
<ol start="5">
<li>
<p><code>storage.admin</code> ロールをサービス アカウントにバインドします。</p>
</li>
</ol>
<pre><code>gcloud projects add-iam-policy-binding $PROJECT \&#x000A;    --role roles/storage.admin \&#x000A;    --member serviceAccount:$SA_EMAIL&#x000A;</code></pre>
<ol start="6">
<li>
<p>サービス アカウント キーをダウンロードします。後のステップで Spinnaker をインストールして、このキーを Kubernetes Engine にアップロードします。</p>
</li>
</ol>
<pre><code>gcloud iam service-accounts keys create spinnaker-sa.json \&#x000A;     --iam-account $SA_EMAIL&#x000A;</code></pre>
<p>（出力）</p>
<pre><code class="language-bash prettyprint">created key [12f224e036437704b91a571792462ca6fc4cd438] of type [json] as [spinnaker-sa.json] for [spinnaker-account@qwiklabs-gcp-gcpd-f5e16da10e5d.iam.gserviceaccount.com]&#x000A;</code></pre>
<h2 id="step6">Spinnaker パイプラインをトリガーする Cloud Pub/Sub を設定する</h2>
<ol>
<li>
<p>Container Registry からの通知に使用する Cloud Pub/Sub トピックを作成します。</p>
</li>
</ol>
<pre><code>gcloud pubsub topics create projects/$PROJECT/topics/gcr&#x000A;</code></pre>
<ol start="2">
<li>
<p>イメージの push についての通知を受け取れるように、Spinnaker から読み取ることができるサブスクリプションを作成します。</p>
</li>
</ol>
<pre><code>gcloud pubsub subscriptions create gcr-triggers \&#x000A;    --topic projects/${PROJECT}/topics/gcr&#x000A;</code></pre>
<ol start="3">
<li>
<p>Spinnaker のサービス アカウントに、gcr-triggers サブスクリプションからの読み取り権限を付与します。</p>
</li>
</ol>
<pre><code>export SA_EMAIL=$(gcloud iam service-accounts list \&#x000A;    --filter="displayName:spinnaker-account" \&#x000A;    --format='value(email)')&#x000A;</code></pre>
<pre><code>gcloud beta pubsub subscriptions add-iam-policy-binding gcr-triggers \&#x000A;    --role roles/pubsub.subscriber --member serviceAccount:$SA_EMAIL&#x000A;</code></pre>
<h3>完了したタスクをテストする</h3>
<p>[<strong>進行状況を確認</strong>] をクリックして、実行したタスクを確認します。環境が正常に設定されている場合は、評価スコアが表示されます。</p>
<ql-activity-tracking step="1">
  環境の設定
</ql-activity-tracking>
<h2 id="step7">Helm を使用した Spinnaker のデプロイ</h2>
<p>このセクションでは、<a href="https://github.com/kubernetes/helm" target="_blank">Helm</a> を使用して <a href="https://github.com/kubernetes/charts" target="_blank">Charts</a> リポジトリから Spinnaker をデプロイします。Helm は、<a href="http://kubeapps.com" target="_blank">Kubernetes アプリケーション</a>の構成とデプロイに使用可能なパッケージ マネージャです。</p>
<h3>Helm をインストールする</h3>
<ol>
<li>
<p><code>helm</code> バイナリをダウンロードしてインストールします。</p>
</li>
</ol>
<pre><code class="language-bash prettyprint">wget https://get.helm.sh/helm-v3.1.0-linux-amd64.tar.gz&#x000A;</code></pre>
<ol start="2">
<li>
<p>ファイルをローカル システムに解凍します。</p>
</li>
</ol>
<pre><code>tar zxfv helm-v3.1.0-linux-amd64.tar.gz&#x000A;</code></pre>
<pre><code>cp linux-amd64/helm .&#x000A;</code></pre>
<ol start="3">
<li>
<p>Helm にクラスタの cluster-admin ロールを付与します。</p>
</li>
</ol>
<pre><code>kubectl create clusterrolebinding user-admin-binding \&#x000A;    --clusterrole=cluster-admin --user=$(gcloud config get-value account)&#x000A;</code></pre>
<ol start="4">
<li>
<p>Spinnaker に <code>cluster-admin</code> ロールを付与し、すべての名前空間にリソースをデプロイできるようにします。</p>
</li>
</ol>
<pre><code>kubectl create clusterrolebinding --clusterrole=cluster-admin \&#x000A;    --serviceaccount=default:default spinnaker-admin&#x000A;</code></pre>
<ol start="5">
<li>
<p>Helm の使用可能なリポジトリに stable チャートのデプロイを追加します（Spinnaker も含まれています）。</p>
</li>
</ol>
<pre><code>./helm repo add stable https://kubernetes-charts.storage.googleapis.com&#x000A;./helm repo update&#x000A;</code></pre>
<h3>Spinnaker を構成する</h3>
<ol start="6">
<li>
<p>引き続き Cloud Shell で、Spinnaker のパイプライン構成を保存するためのバケットを作成します。</p>
</li>
</ol>
<pre><code>export PROJECT=$(gcloud info \&#x000A;    --format='value(config.project)')&#x000A;</code></pre>
<pre><code>export BUCKET=$PROJECT-spinnaker-config&#x000A;</code></pre>
<pre><code>gsutil mb -c regional -l us-central1 gs://$BUCKET&#x000A;</code></pre>
<ol start="7">
<li>
<p>次のコマンドを実行して、Spinnaker のインストール方法を記述する <code>spinnaker-config.yaml</code> ファイルを作成します。</p>
</li>
</ol>
<pre><code>export SA_JSON=$(cat spinnaker-sa.json)&#x000A;export PROJECT=$(gcloud info --format='value(config.project)')&#x000A;export BUCKET=$PROJECT-spinnaker-config&#x000A;cat &gt; spinnaker-config.yaml &lt;&lt;EOF&#x000A;gcs:&#x000A;  enabled: true&#x000A;  bucket: $BUCKET&#x000A;  project: $PROJECT&#x000A;  jsonKey: '$SA_JSON'&#x000A;&#x000A;dockerRegistries:&#x000A;- name: gcr&#x000A;  address: https://gcr.io&#x000A;  username: _json_key&#x000A;  password: '$SA_JSON'&#x000A;  email: 1234@5678.com&#x000A;&#x000A;# デフォルトのストレージ バックエンドである minio を無効にします&#x000A;minio:&#x000A;  enabled: false&#x000A;&#x000A;# GCP サービスを有効にするように Spinnaker を構成します&#x000A;halyard:&#x000A;  additionalScripts:&#x000A;    create: true&#x000A;    data:&#x000A;      enable_gcs_artifacts.sh: |-&#x000A;        \$HAL_COMMAND config artifact gcs account add gcs-$PROJECT --json-path /opt/gcs/key.json&#x000A;        \$HAL_COMMAND config artifact gcs enable&#x000A;      enable_pubsub_triggers.sh: |-&#x000A;        \$HAL_COMMAND config pubsub google enable&#x000A;        \$HAL_COMMAND config pubsub google subscription add gcr-triggers \&#x000A;          --subscription-name gcr-triggers \&#x000A;          --json-path /opt/gcs/key.json \&#x000A;          --project $PROJECT \&#x000A;          --message-format GCR&#x000A;EOF&#x000A;</code></pre>
<h3>Spinnaker チャートをデプロイする</h3>
<ol start="8">
<li>
<p>Helm コマンドライン インターフェースを使用して、構成セットとともにチャートをデプロイします。</p>
</li>
</ol>
<pre><code>./helm install -n default cd stable/spinnaker -f spinnaker-config.yaml \&#x000A;           --version 1.23.0 --timeout 10m0s --wait&#x000A;</code></pre>
<ql-infobox>
インストールが完了するまでに <b>5～8 分</b>ほどかかります。
</ql-infobox>
<ol start="9">
<li>
<p>コマンドが完了したら次のコマンドを実行して、Cloud Shell から Spinnaker へのポート転送を設定します。</p>
</li>
</ol>
<pre><code>export DECK_POD=$(kubectl get pods --namespace default -l "cluster=spin-deck" \&#x000A;    -o jsonpath="{.items[0].metadata.name}")&#x000A;</code></pre>
<pre><code>kubectl port-forward --namespace default $DECK_POD 8080:9000 &gt;&gt; /dev/null &amp;&#x000A;</code></pre>
<ol start="10">
<li>Spinnaker ユーザー インターフェースを開くには、Cloud Shell ウィンドウの最上部で [<strong>ウェブでプレビュー</strong>] をクリックし、[<strong>ポート 8080 でプレビュー</strong>] をクリックします。</li>
</ol>
<p><img alt="bde9fe42e27656fb.png" src="https://cdn.qwiklabs.com/a6YnJv8GlGae4rnJIbjA27J8c7YApa%2B6noPFkkKxZjk%3D"></p>
<p>ようこそ画面に続いて、Spinnaker のユーザー インターフェースが表示されます。</p>
<p><img alt="8c03705c17bf3b7f.png" src="https://cdn.qwiklabs.com/78xnSiBC3%2B6wTlFb3dk%2FPGGw0%2BWMeBhMVkQpnHfWFiY%3D"></p>
<p>このタブは開いたままにしておいてください。後ほど、ここから Spinnaker UI にアクセスします。</p>
<h3>完了したタスクをテストする</h3>
<p>[<strong>進行状況を確認</strong>] をクリックして、実行したタスクを確認します。Kubernetes Helm を使用して Spinnaker チャートが正常にデプロイされている場合は、評価スコアが表示されます。</p>
<ql-activity-tracking step="2">
  Kubernetes Helm を使用して Spinnaker チャートをデプロイする
</ql-activity-tracking>
<h2 id="step8">Docker イメージのビルド</h2>
<p>このセクションでは、アプリ ソース コードの変更を検出して Docker イメージをビルドしてから、Container Registry に push するように Cloud Build を構成します。</p>
<h3>ソースコード リポジトリを作成する</h3>
<ol>
<li>
<p>Cloud Shell のタブで、サンプル アプリケーションのソースコードをダウンロードします。</p>
</li>
</ol>
<pre><code>wget https://gke-spinnaker.storage.googleapis.com/sample-app-v2.tgz&#x000A;</code></pre>
<ol start="2">
<li>
<p>ソースコードを展開します。</p>
</li>
</ol>
<pre><code>tar xzfv sample-app-v2.tgz&#x000A;</code></pre>
<ol start="3">
<li>
<p>ディレクトリをソースコードに変更します。</p>
</li>
</ol>
<pre><code>cd sample-app&#x000A;</code></pre>
<ol start="4">
<li>
<p>このリポジトリの Git commit にユーザー名とメールアドレスを設定します。<code>[USERNAME]</code> は作成するユーザー名で置き換えます。</p>
</li>
</ol>
<pre><code>git config --global user.email "$(gcloud config get-value core/account)"&#x000A;</code></pre>
<pre><code>git config --global user.name "[USERNAME]"&#x000A;</code></pre>
<ol start="5">
<li>
<p>ソースコード レポジトリへの最初の commit を行います。</p>
</li>
</ol>
<pre><code>git init&#x000A;</code></pre>
<pre><code>git add .&#x000A;</code></pre>
<pre><code>git commit -m "Initial commit"&#x000A;</code></pre>
<ol start="6">
<li>
<p>コードをホストするレポジトリを作成します。</p>
</li>
</ol>
<pre><code>gcloud source repos create sample-app&#x000A;</code></pre>
<ql-infobox>
<p>「you may be billed for this repository」というメッセージは無視してかまいません。</p>
</ql-infobox>
<pre><code class="language-bash prettyprint">git config credential.helper gcloud.sh&#x000A;</code></pre>
<ol start="7">
<li>
<p>新しく作成したレポジトリをリモート レポジトリとして追加します。</p>
</li>
</ol>
<pre><code>export PROJECT=$(gcloud info --format='value(config.project)')&#x000A;</code></pre>
<pre><code>git remote add origin https://source.developers.google.com/p/$PROJECT/r/sample-app&#x000A;</code></pre>
<ol start="8">
<li>
<p>コードを新しいレポジトリのマスター ブランチに push します。</p>
</li>
</ol>
<pre><code>git push origin master&#x000A;</code></pre>
<ol start="9">
<li>
<strong>ナビゲーション メニュー</strong> &gt; [<strong>Source Repositories</strong>] をクリックして、Console にソースコードが表示されることを確認します。</li>
</ol>
<p><img alt="nav-source-repo.png" src="https://cdn.qwiklabs.com/y1jULn3nMeoUGcf1aBnSZ7u9KT5HLYqIhd%2FGa78LbtY%3D"></p>
<ol start="10">
<li>[<strong>すべてのリポジトリを表示</strong>] をクリックし、[<strong>sample-app</strong>] を選択します。</li>
</ol>
<p><img alt="source-repo.png" src="https://cdn.qwiklabs.com/r2BMszyuyimucqRYkLOpNqc1FMXQ9W9VXxt2hXnSeO0%3D"></p>
<h3>ビルドトリガーを構成する</h3>
<p><a href="https://git-scm.com/book/en/v2/Git-Basics-Tagging" target="_blank">Git タグ</a>がソース リポジトリに push されるたびに Docker イメージをビルドして push するように Container Builder を構成します。Container Builder はソースコードを自動的にチェックアウトして、リポジトリ内の Dockerfile から Docker イメージをビルドし、そのイメージを Google Cloud Container Registry に push します。</p>
<p><img alt="bcab2f07975c8776.png" src="https://cdn.qwiklabs.com/5mt%2Fk3dCd%2FgfYPln9TjLQoZkjh%2FZiiOnBWQ5MLLJVyI%3D"></p>
<ol start="11">
<li>
<p>Cloud Platform Console で、<strong>ナビゲーション メニュー</strong> &gt; [<strong>Cloud Build</strong>] &gt; [<strong>トリガー</strong>] をクリックします。</p>
</li>
<li>
<p>[<strong>トリガーを作成</strong>] をクリックします。</p>
</li>
</ol>
<p><img alt="CloudBuild_BuildTriggers.png" src="https://cdn.qwiklabs.com/ZYUwl%2BcDxAArWV1hG08gJoFIOIuImwfe0N4H025tubQ%3D"></p>
<ol start="13">
<li>
<p>新しく作成した <code>sample-app</code> リポジトリを選択します。</p>
</li>
<li>
<p>次のトリガー設定を指定します。</p>
</li>
</ol>
<ul>
<li>
<p><strong>名前</strong>: <code>sample-app-tags</code></p>
</li>
<li>
<p><strong>トリガーの種類</strong>: タグ</p>
</li>
<li>
<p><strong>タグ（正規表現）</strong>: <code>v.*</code></p>
</li>
<li>
<p><strong>ビルド構成</strong>: <code>Cloud Build 構成ファイル（yaml または json）</code></p>
</li>
<li>
<p><strong>Cloud Build 構成ファイルの場所</strong>: <code>/cloudbuild.yaml</code></p>
</li>
</ul>
<ol start="15">
<li>[<strong>トリガーを作成</strong>] をクリックします。</li>
</ol>
<p><img alt="create_trigger_console.png" src="https://cdn.qwiklabs.com/nuJhjbQskef1vJPuwHPiDncAnbGtiHWNBPVHkGR95q8%3D"></p>
<p>これ以降、文字 "v" が先頭に付いている Git タグをソースコード リポジトリに push すると、Container Builder が自動的にアプリケーションをビルドして、Docker イメージとして Container Registry に push します。</p>
<h3>Spinnaker で使用するために Kubernetes マニフェストを準備する</h3>
<p>Spinnaker は、クラスタにデプロイするために Kubernetes マニフェストにアクセスする必要があります。このセクションでは、Cloud Build での CI プロセス中にマニフェストを保存する Cloud Storage バケットを作成します。マニフェストが Cloud Storage に保存されると、Spinnaker はパイプラインの実行中にマニフェストをダウンロードして適用できます。</p>
<ol start="16">
<li>
<p>バケットを作成します。</p>
</li>
</ol>
<pre><code>export PROJECT=$(gcloud info --format='value(config.project)')&#x000A;</code></pre>
<pre><code>gsutil mb -l us-central1 gs://$PROJECT-kubernetes-manifests&#x000A;</code></pre>
<ol start="17">
<li>
<p>バケットに対するバージョニングを有効にして、マニフェストの履歴を取得できるようにします。</p>
</li>
</ol>
<pre><code>gsutil versioning set on gs://$PROJECT-kubernetes-manifests&#x000A;</code></pre>
<ol start="18">
<li>
<p>kubernetes デプロイメント マニフェストに正しいプロジェクト ID を設定します。</p>
</li>
</ol>
<pre><code>sed -i s/PROJECT/$PROJECT/g k8s/deployments/*&#x000A;</code></pre>
<ol start="19">
<li>
<p>リポジトリに変更を commit します。</p>
</li>
</ol>
<pre><code>git commit -a -m "Set project ID"&#x000A;</code></pre>
<h3>イメージをビルドする</h3>
<p>次の手順で最初のイメージを push します。</p>
<ol start="20">
<li>
<p>Cloud Shell で、引き続き <code>sample-app</code> ディレクトリから、Git タグを作成します。</p>
</li>
</ol>
<pre><code>git tag v1.0.0&#x000A;</code></pre>
<ol start="21">
<li>
<p>タグを push します。</p>
</li>
</ol>
<pre><code>git push --tags&#x000A;</code></pre>
<p>（出力）</p>
<pre><code class="language-bash prettyprint">To https://source.developers.google.com/p/qwiklabs-gcp-ddf2925f84de0b16/r/sample-app&#x000A;* [new tag]      v1.0.0 -&gt; v1.0.0&#x000A;</code></pre>
<ol start="22">
<li>GCP Console に移動します。引き続き Cloud Build で、左側のペインの [<strong>履歴</strong>] をクリックして、ビルドがトリガーされたことを確認します。トリガーされていない場合は、前のセクションでトリガーが正しく構成されていることを確認します。</li>
</ol>
<p><img alt="CloudBuild_BuildHistory.png" src="https://cdn.qwiklabs.com/hOy0rerMcmxMDs8Qs23dG2ZPGIzt78TOV9adkp1M5dQ%3D"></p>
<p>ビルドが完了するまで<strong>このページで待機</strong>します。完了したら、次のセクションに進みます。</p>
<h3>完了したタスクをテストする</h3>
<p>[<strong>進行状況を確認</strong>] をクリックして、実行したタスクを確認します。Docker イメージが正常にビルドされている場合は、評価スコアが表示されます。</p>
<ql-activity-tracking step="3">
  Docker イメージをビルドする
</ql-activity-tracking>
<h2 id="step9">デプロイメント パイプラインの構成</h2>
<p>これでイメージが自動的にビルドされるようになりました。イメージは Kubernetes クラスタにデプロイする必要があります。</p>
<p>統合テストに備えてスケールダウンされた環境にデプロイします。統合テストに合格したら、コードを本番環境サービスにデプロイするための変更を手動で承認する必要があります。</p>
<h3>Spinnaker を管理する spin CLI をインストールする</h3>
<p><a href="https://www.spinnaker.io/guides/spin/cli/" target="_blank">spin</a> は、Spinnaker のアプリケーションとパイプラインを管理するためのコマンドライン ユーティリティです。</p>
<ol>
<li>
<p>バージョン 1.14.0 の <code>spin</code> をダウンロードします。</p>
</li>
</ol>
<pre><code>curl -LO https://storage.googleapis.com/spinnaker-artifacts/spin/1.14.0/linux/amd64/spin&#x000A;</code></pre>
<ol start="2">
<li>
<p><code>spin</code> を実行可能にします。</p>
</li>
</ol>
<pre><code>chmod +x spin&#x000A;</code></pre>
<h3>デプロイメント パイプラインを作成する</h3>
<ol start="3">
<li>
<p>spin を使用して、Spinnaker 内に sample という名前のアプリを作成します。このアプリのオーナーのメールアドレスを設定します。</p>
</li>
</ol>
<pre><code>./spin application save --application-name sample \&#x000A;                        --owner-email "$(gcloud config get-value core/account)" \&#x000A;                        --cloud-providers kubernetes \&#x000A;                        --gate-endpoint http://localhost:8080/gate&#x000A;</code></pre>
<p>次に、継続的デリバリー パイプラインを作成します。このチュートリアルでは、文字「v」で始まるタグが付いた Docker イメージが Container Registry に到着したことを検出するようにパイプラインが構成されます。</p>
<ol start="4">
<li>
<p><code>sample-app</code> ソースコード ディレクトリから次のコマンドを実行します。これにより、サンプル パイプラインが Spinnaker インスタンスにアップロードされます。</p>
</li>
</ol>
<pre><code>export PROJECT=$(gcloud info --format='value(config.project)')&#x000A;sed s/PROJECT/$PROJECT/g spinnaker/pipeline-deploy.json &gt; pipeline.json&#x000A;./spin pipeline save --gate-endpoint http://localhost:8080/gate -f pipeline.json&#x000A;</code></pre>
<h3>パイプラインの実行を手動でトリガーして確認する</h3>
<p>ここまでの手順で作成した構成では、新しくタグ付けされたイメージが push された際の通知を利用して Spinnaker パイプラインをトリガーします。前のステップでは、タグを Cloud Source Repositories に push しました。これによって、Cloud Build でイメージのビルドと Container Registry への push がトリガーされます。このパイプラインを確認するために、<code>手動でトリガー</code>してみましょう。</p>
<ol start="5">
<li>Spinnaker UI で、画面の上部にある [<strong>Applications</strong>] をクリックしてマネージド アプリケーションのリストを表示します。この手順で作成したアプリケーションは、<strong>sample</strong> です。<strong>sample</strong> が表示されない場合は、Spinnaker の [Applications] タブを更新してみてください。</li>
</ol>
<p><img alt="app-sample.png" src="https://cdn.qwiklabs.com/l1SLrdit4XYqd636IlNhYubL13Y3jvvOO7AvQ2Df0DA%3D"></p>
<ol start="6">
<li>[<strong>sample</strong>] をクリックしてアプリケーション デプロイメントを表示します。</li>
<li>上部にある [<strong>パイプライン</strong>] をクリックして、アプリケーションのパイプラインのステータスを表示します。</li>
<li>パイプラインを初めてトリガーするために、[<strong>手動実行を開始</strong>] をクリックします。</li>
</ol>
<p><img alt="start-man-ex.png" src="https://cdn.qwiklabs.com/WSy8LMXa74ovYfYogBCA6J%2FXzQ77Qjp4jKIyX8jpa9o%3D"></p>
<ol start="9">
<li>
<p>[<strong>実行</strong>] をクリックします。</p>
</li>
<li>
<p>[<strong>Execution Details</strong>] をクリックして、パイプラインの進行状況の詳細を表示します。</p>
</li>
</ol>
<p><img alt="man-start-detail.png" src="https://cdn.qwiklabs.com/ZJXzykscsE1mOzsubdbDgPBWisZN%2BkV1Sj7LITrikF8%3D"></p>
<p>進行状況バーで、デプロイメント パイプラインのステータスとそのステップを確認できます。</p>
<p><img alt="progress-bar.png" src="https://cdn.qwiklabs.com/t%2BWtKxCdqL8czxdWYVvy%2BM2Cuqi7DcUbj5c6OR6k9OA%3D"></p>
<p>青色のステップは現在実行中です。緑色は正常に完了したステップ、赤色は失敗したステップを示します。</p>
<ol start="11">
<li>ステージをクリックして、その詳細を表示します。</li>
</ol>
<p><strong>3～5 分</strong>後に統合テストフェーズが完了すると、パイプラインがデプロイメントの続行を手動で承認するよう求めてきます。</p>
<ol start="12">
<li>黄色の「人物」アイコンにカーソルを合わせ、[<strong>続行</strong>] をクリックします。</li>
</ol>
<p><img alt="continue-to-deploy.png" src="https://cdn.qwiklabs.com/1eDinR4BooeXbRTiOJTyKtvh2nWTkJWX4ddhhiyFuYk%3D"></p>
<p>ロールアウトが本番環境フロントエンドと本番環境バックエンドのデプロイメントに進みます。これは数分後に完了します。</p>
<ol start="13">
<li>Spinnaker UI の上部で [<strong>Infrastructure</strong>] &gt; [<strong>Load Balancers</strong>] をクリックして、アプリを表示します。</li>
</ol>
<p><img alt="infrasturcture-lb.png" src="https://cdn.qwiklabs.com/ans47fs3%2B2qcS6gjsDvYb3WW6%2BoRUX4RhLKCIIum7NA%3D"></p>
<ol start="14">
<li>
<p>ロードバランサのリストを下にスクロールし、[<code>service sample-frontend-production</code>] の下の [デフォルト] をクリックします。</p>
</li>
<li>
<p>右側の詳細ペインをスクロール ダウンし、<strong>Ingress</strong> IP のクリップボード ボタンをクリックしてアプリの IP アドレスをコピーします。Spinnaker UI から Ingress IP へのリンクではデフォルトで HTTPS が使用される場合がありますが、このアプリケーションは HTTP を使用するように構成されています。</p>
</li>
</ol>
<p><img alt="ingress.png" src="https://cdn.qwiklabs.com/q%2BkaykoQWgcrCfZtJ%2B7amfMqq0qguh%2B8rFDcpl1C2sI%3D"></p>
<ol start="16">
<li>コピーしたアドレスを新しいブラウザタブに貼り付けて、アプリケーションを表示します。canary バージョンが表示される場合がありますが、画面を更新すると production バージョンも表示されます。</li>
</ol>
<p><img alt="5350848ad1354d1c.png" src="https://cdn.qwiklabs.com/VteuqPqhtXAAocO0DyXy2QM%2B%2BPbOGMyXs114MYCVIHI%3D"></p>
<p>これで、アプリケーションをビルド、テスト、デプロイするためのパイプラインを手動でトリガーしました。</p>
<h3>完了したタスクをテストする</h3>
<p>[<strong>進行状況を確認</strong>] をクリックして、実行したタスクを確認します。サービス ロードバランサが正常に作成されている場合は、評価スコアが表示されます。</p>
<ql-activity-tracking step="4">
  サービス ロードバランサを作成する
</ql-activity-tracking>
<h3>完了したタスクをテストする</h3>
<p>[<strong>進行状況を確認</strong>] をクリックして、実行したタスクを確認します。イメージが正常に本番環境にデプロイされている場合は、評価スコアが表示されます。</p>
<ql-activity-tracking step="5">
  イメージを本番環境にデプロイする
</ql-activity-tracking>
<h2 id="step10">コードの変更によるパイプラインのトリガー</h2>
<p>ここで、コードの変更、Git タグの push、レスポンスでのパイプライン実行の監視を通して、パイプラインをエンドツーエンドでテストします。"v" で始まる Git タグを push すると、新しい Docker イメージをビルドして Container Registry に push するための Container Builder がトリガーされます。Spinnaker は新しいイメージタグが "v" で始まることを検出すると、イメージをカナリアにデプロイしてテストを実行し、同じイメージをデプロイメント内のすべての Pod にロールアウトするためのパイプラインをトリガーします。</p>
<ol>
<li>
<p>sample-app ディレクトリから次のコマンドを実行して、アプリの色をオレンジ色から青色に変更します。</p>
</li>
</ol>
<pre><code>sed -i 's/orange/blue/g' cmd/gke-info/common-service.go&#x000A;</code></pre>
<ol start="2">
<li>
<p>変更にタグを付け、ソースコード リポジトリに push します。</p>
</li>
</ol>
<pre><code>git commit -a -m "Change color to blue"&#x000A;</code></pre>
<pre><code>git tag v1.0.1&#x000A;</code></pre>
<pre><code>git push --tags&#x000A;</code></pre>
<ol start="3">
<li>
<p>GCP Console で [<strong>Cloud Build</strong>] &gt; [<strong>履歴</strong>] を開き、新しいビルドが表示されるまで数分待ちます。必要であればページを更新します。新しいビルドが完了するまで待ってから次のステップに進みます。</p>
</li>
<li>
<p>Spinnaker UI に戻り、[<strong>Pipelines</strong>] をクリックして、イメージのデプロイを開始するパイプラインの動作を監視します。自動的にトリガーされたパイプラインは、表示されるまでに数分かかります。必要であればページを更新します。</p>
</li>
</ol>
<p><img alt="d006977feef1a15d.png" src="https://cdn.qwiklabs.com/zTxZCs10j1jYtRzTBKH8XTzaTCiqO2UdtDyg7%2BqQ2c4%3D"></p>
<h3>完了したタスクをテストする</h3>
<p>[<strong>進行状況を確認</strong>] をクリックして、実行したタスクを確認します。コードの変更によるパイプラインのトリガーが正常に行われた場合は、評価スコアが表示されます。</p>
<ql-activity-tracking step="6">
  コードの変更によるパイプラインのトリガー
</ql-activity-tracking>
<h2 id="step11">カナリア デプロイメントを観察する</h2>
<ol>
<li>
<p>デプロイが一時停止し、本番環境へのロールアウトを待機している間に、実行中のアプリケーションを表示しているウェブページに戻ります。アプリが表示されているタブを繰り返し更新してください。バックエンドのうちの 4 つがアプリの以前のバージョンを実行しており、カナリアを実行しているバックエンドは 1 つだけです。5 回更新するごとにアプリの新しい青色バージョンが表示されるはずです。</p>
</li>
<li>
<p>パイプラインが完了すると、アプリは次のスクリーンショットのようになります。コードを変更したため、色は青色に変わっています。また、[<strong>バージョン</strong>] フィールドには <code>canary</code> と示されるようになっています。</p>
</li>
</ol>
<p><img alt="Blue_Application_Output.png" src="https://cdn.qwiklabs.com/h9zibqT3MUXBu8FWjVE%2B07ZdbXMaGFsmMCLKgALX8nE%3D"></p>
<p>これで、アプリを本番環境全体に正常にロールアウトできました。</p>
<ol start="3">
<li>
<p>必要に応じて前回の commit を元に戻し、この変更をロールバックできます。ロールバックすると新しいタグ　(v1.0.2）が追加され、v1.0.1 をデプロイするときに使用したのと同じパイプラインを通してタグが戻されます。</p>
</li>
</ol>
<pre><code>git revert v1.0.1&#x000A;</code></pre>
<pre><code>git tag v1.0.2&#x000A;</code></pre>
<pre><code>git push --tags&#x000A;</code></pre>
<ol start="4">
<li>ビルドとパイプラインが完了したら、ロールバックを確認します。[<strong>インフラストラクチャ</strong>] &gt; [<strong>ロードバランサ</strong>] の順にクリックしてから、[<strong>service sample-frontend-production</strong>] の [<strong>デフォルト</strong>] をクリックして Ingress IP アドレスをコピーします。このアドレスを新しいタブに貼り付けます。</li>
</ol>
<p>アプリがオレンジ色に戻り、<code>production</code> バージョン番号を確認できます。</p>
<p><img alt="revert.png" src="https://cdn.qwiklabs.com/LtAUqeMxVZbGZv4tBVncyhlMXO5JR9q7vN51oHPlNvU%3D"></p>
<h2 id="step12">お疲れさまでした</h2>

# Cloud Monitoring APM によるサイトの信頼性のトラブルシューティング
<h2 id="step2">概要</h2>
<p>このラボの目標は、GKE クラスタ インフラストラクチャ、Istio、およびこのインフラストラクチャにデプロイされたアプリケーションをモニタリングする Cloud モニタリング の特定の機能を習得することです。</p>
<h3>演習内容</h3>
<ul>
<li>
<p>GKE クラスタを作成する</p>
</li>
<li>
<p>GKE クラスタにマイクロサービス アプリケーションをデプロイする</p>
</li>
<li>
<p>アプリケーションにレイテンシとエラーの SLI および SLO を定義する</p>
</li>
<li>
<p>SLI をモニタリングするように Cloud モニタリングを構成する</p>
</li>
<li>
<p>アプリケーションに破壊的変更をデプロイし、その結果生じる問題に対して Cloud モニタリングを使って解決策を講じる</p>
</li>
<li>
<p>解決策によって SLO 違反に対処できたことを確認する</p>
</li>
</ul>
<h3>ラボの内容</h3>
<ul>
<li>
<p>マイクロサービス アプリケーションを既存の GKE クラスタにデプロイする方法</p>
</li>
<li>
<p>アプリケーションに適した SLI と SLO を選択する方法</p>
</li>
<li>
<p>Cloud モニタリング の機能を使用して SLI を実装する方法</p>
</li>
<li>
<p>Cloud Trace、Profiler、および Debugger を使用してソフトウェアの問題を特定する方法</p>
</li>
</ul>
<h3>前提条件</h3>
<ul>
<li>
<p>Kubernetes の基礎知識</p>
</li>
<li>
<p>Cloud モニタリング の基礎知識</p>
</li>
<li>
<p>トラブルシューティング プロセスの基礎知識</p>
</li>
</ul>

<h2 id="step4">インフラストラクチャ設定</h2>
<p>このラボでは、Google Kubernetes Engine クラスタに接続し、クラスタが正しく作成されていることを確認します。</p>
<p><code>gcloud</code> でゾーンを設定します。</p>
<pre><code class="language-bash prettyprint">gcloud config set compute/zone us-west1-b&#x000A;</code></pre>
<p>プロジェクト ID を設定します。</p>
<pre><code class="language-bash prettyprint">export PROJECT_ID=$(gcloud info --format='value(config.project)')&#x000A;</code></pre>
<p><code>shop-cluster</code> という名前のクラスタが作成されていることを確認します。</p>
<pre><code class="language-bash prettyprint">gcloud container clusters list&#x000A;</code></pre>
<p>クラスタのステータスが「PROVISIONING」である場合は、少し待ってから上記のコマンドを再び実行します。ステータスが「RUNNING」になるまでこれを繰り返してください。</p>
<p>待っている間に、クラスタ上のアプリケーションをモニタリングするために Cloud モニタリング ワークスペースを設定します。</p>
<h3>モニタリング ワークスペースを作成する</h3>
<p>ここで、Qwiklabs GCP プロジェクトに関連付けられたモニタリング ワークスペースを設定します。次の手順にて、モニタリングの無料トライアル付きの新しいアカウントを作成します。</p>
<p>1. Google Cloud Platform コンソールで、[<strong>ナビゲーション メニュー </strong>] &gt; [<strong>モニタリング</strong>] をクリックします。</p>
<p>2. ワークスペースがプロビジョニングされるのを待ちます。</p>
<p>モニタリング ダッシュボードが開いたら、モニタリング ワークスペースが利用できます。</p>
<p><img alt="monitoring_overview.png" src="https://cdn.qwiklabs.com/BGS8QF9AXRWwP0pKwugIg7rSBz9hP%2B2r88V4czj0m4o%3D"></p>

<h3>クラスタを確認する</h3>
<p>Cloud Shell に戻ります。クラスタのステータスが「RUNNING」になったら、クラスタの認証情報を取得します。</p>
<pre><code class="language-bash prettyprint">gcloud container clusters get-credentials shop-cluster --zone us-west1-b&#x000A;</code></pre>
<p>（出力）</p>
<pre><code class="language-bash prettyprint">Fetching cluster endpoint and auth data.&#x000A;kubeconfig entry generated for shop-cluster.&#x000A;</code></pre>
<p>ノードが作成されていることを確認します。</p>
<pre><code class="language-bash prettyprint">kubectl get nodes&#x000A;</code></pre>
<p>出力は次のようになります。</p>
<pre><code class="language-bash prettyprint">NAME                                  STATUS    ROLES     AGE       VERSION&#x000A;gke-shop-cluster-demo-default-pool1-24748028-3nwh   Ready     &lt;none&gt;    4m        v1.13.7-gke.8&#x000A;gke-shop-cluster-demo-default-pool1-24748028-3z1g   Ready     &lt;none&gt;    4m        v1.13.7-gke.8&#x000A;gke-shop-cluster-demo-default-pool1-24748028-4ksd   Ready     &lt;none&gt;    4m        v1.13.7-gke.8&#x000A;gke-shop-cluster-demo-default-pool1-24748028-f2f2   Ready     &lt;none&gt;    4m        v1.13.7-gke.8&#x000A;gke-shop-cluster-demo-default-pool1-24748028-gcb3   Ready     &lt;none&gt;    4m        v1.13.7-gke.8&#x000A;</code></pre>
<h2 id="step5">アプリケーションをデプロイする</h2>
<p>このセクションでは、Hipster Shop というマイクロサービス アプリケーションをクラスタにデプロイして、モニタリングする実際のワークロードを作成します。</p>
<p>次のコマンドを実行してリポジトリのクローンを作成します。</p>
<pre><code class="language-bash prettyprint">git clone -b APM-Troubleshooting-Demo-2 https://github.com/blipzimmerman/microservices-demo-1&#x000A;</code></pre>
<p><code>skaffold</code> をダウンロードしてインストールします。</p>
<pre><code>curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/v0.36.0/skaffold-linux-amd64 &amp;&amp; chmod +x skaffold &amp;&amp; sudo mv skaffold /usr/local/bin&#x000A;</code></pre>
<p><code>skaffold</code> を使用してアプリをインストールします。</p>
<pre><code class="language-bash prettyprint">cd microservices-demo-1&#x000A;skaffold run&#x000A;</code></pre>
<p>すべてが正しく動作していることを確認します。</p>
<pre><code class="language-bash prettyprint">kubectl get pods&#x000A;</code></pre>
<p>次のような出力のように表示されるはずです。</p>
<pre><code class="language-bash prettyprint">NAME                                     READY     STATUS    RESTARTS   AGE&#x000A;adservice-55f94cfd9c-4lvml               1/1       Running   0          20m&#x000A;cartservice-6f4946f9b8-6wtff             1/1       Running   2          20m&#x000A;checkoutservice-5688779d8c-l6crl         1/1       Running   0          20m&#x000A;currencyservice-665d6f4569-b4sbm         1/1       Running   0          20m&#x000A;emailservice-684c89bcb8-h48sq            1/1       Running   0          20m&#x000A;frontend-67c8475b7d-vktsn                1/1       Running   0          20m&#x000A;loadgenerator-6d646566db-p422w           1/1       Running   0          20m&#x000A;paymentservice-858d89d64c-hmpkg          1/1       Running   0          20m&#x000A;productcatalogservice-bcd85cb5-d6xp4     1/1       Running   0          20m&#x000A;recommendationservice-685d7d6cd9-pxd9g   1/1       Running   0          20m&#x000A;redis-cart-9b864d47f-c9xc6               1/1       Running   0          20m&#x000A;shippingservice-5948f9fb5c-vndcp         1/1       Running   0          20m&#x000A;</code></pre>
<p>すべての Pod が「Running」になるまでコマンドを再実行してから、次のステップに進んでください。</p>
<p>アプリケーションの<strong>外部 IP</strong> を取得します。</p>
<pre><code class="language-bash prettyprint">export EXTERNAL_IP=$(kubectl get service frontend-external | awk 'BEGIN { cnt=0; } { cnt+=1; if (cnt &gt; 1) print $4; }')&#x000A;</code></pre>
<p>最後に、アプリが起動して稼働していることを確認します。</p>
<pre><code class="language-bash prettyprint">curl -o /dev/null -s -w "%{http_code}\n"  http://$EXTERNAL_IP&#x000A;</code></pre>
<p><strong>注:</strong> 500 エラーが発生した場合は、このコマンドを再度実行してください。</p>
<p>正常であれば、200 が表示されます。</p>
<p>ソースをダウンロードして、コードを Cloud Source Repositories に挿入します。</p>
<pre><code class="language-bash prettyprint">./setup_csr.sh&#x000A;</code></pre>
<p>以上でアプリケーションのデプロイが完了したので、アプリケーションのモニタリングを設定します。</p>
<h3>リソース</h3>
<ul>
<li><a href="https://github.com/GoogleCloudPlatform/microservices-demo">マイクロサービス デモ アプリケーション</a></li>
</ul>
<p>注: このラボでは、トラブルシューティング演習を補助するため、このアプリケーション ビルドの<a href="https://github.com/blipzimmerman/microservices-demo-1">フォーク</a>を使用します。</p>
<ul>
<li>
<p><a href="http://skaffold.dev">Skaffold</a></p>
</li>
</ul>
<h2 id="step6">SLO と SLI のサンプルを作成する</h2>
<p>モニタリングを実装する前に、Google の書籍『<a href="https://landing.google.com/sre/books/">Site Reliability Engineering</a>』のサービスレベル目標<em></em>に関する章の冒頭部分をご確認ください。</p>
<p>「サービスを適切に管理するには、そのサービスでは実際にどのような動作が重要なのか、またそうした動作をどのように測定し評価するかを理解することが不可欠です。この目的のために、Google では、ユーザーが内部 API を使用しているか一般公開されたプロダクトを使用しているかにかかわらず、一定レベルのサービスを定義してユーザーに提供したいと考えています。」</p>
<p>「Google は、直感と経験、およびユーザーニーズの理解を通して、<strong>サービスレベル指標（SLI）</strong>、<strong>サービスレベル目標（SLO）</strong>、<strong>サービスレベル契約（SLA）</strong>を定義しています。これらの尺度は、重要な指標の基本的な特性、それらの指標の望ましい値、期待されるサービスを提供できない場合の Google の対応を示しています。最終的には、適切な指標を選択することにより、問題が発生したときに適切な対処が可能になります。また、それによってサービスの健全性に対する SRE チームの信頼も深まります。」</p>
<p>「SLI はサービスレベルの指標です。つまり、提供されるサービスのレベルのいくつかの要素について、注意深く定義された定量的尺度です。」</p>
<p>「ほとんどのサービスでは、<strong>リクエストのレイテンシ</strong>（リクエストに対してレスポンスを返すのにかかる時間）が主要な SLI と見なされます。他の一般的な SLI には、<strong>エラー率</strong>（多くの場合、受信したすべてのリクエストのわずかな割合で表されます）、<strong>システム スループット</strong>（一般的には 1 秒あたりのリクエスト数で表します）があります。SRE にとって重要なもう 1 つの SLI は、<strong>可用性</strong>（サービスが使用可能な時間の割合）です。多くの場合、可用性は成功した整形式のリクエストの割合で定義されます。<strong>耐久性</strong>（データが長期間にわたって保持される可能性）も、データ ストレージ システムでは同程度に重要です。測定値はしばしば集計値で表されます。つまり、一定の測定ウィンドウで生データが収集され、レート、平均値、またはパーセンタイル値に変換されます。」</p>
<p>基本的な概念について確認したところで、今度はアプリケーションの SLI と SLO を定義します。エンドユーザーの e コマース トラフィックを処理するのはアプリケーションそのものであることを考えると、ユーザー エクスペリエンスが安定していてパフォーマンスが優れていることが非常に重要になります。SLI をモニタリングして、リクエストのレイテンシ、エラー率、スループット、可用性を調べましょう。</p>
<h3>アプリケーション アーキテクチャ</h3>
<p><img alt="f818a68bd5b98622.png" src="https://cdn.qwiklabs.com/NYo2MRjXDjHGhUiLEoQL9MG8gLMn%2B3wrhz53emKj06g%3D"></p>
<p>SLI を作成するうえでは、該当のアプリケーションがどのように構築されているかを理解することが不可欠です。詳細な情報は元の<a href="https://github.com/GoogleCloudPlatform/microservices-demo">リポジトリ</a>にありますが、このラボでは次のことを理解すれば十分です。</p>
<ul>
<li>
<p>ユーザーはフロントエンドを介してアプリケーションにアクセスします。</p>
</li>
<li>
<p>購入は CheckoutService によって処理されます。</p>
</li>
<li>
<p>CheckoutService は、CurrencyService を使用して変換を処理します。</p>
</li>
<li>
<p>RecommendationService、ProductCatalogService、Adservice などの他のサービスは、ページのレンダリングに必要なコンテンツをフロントエンドに提供するために使用されます。</p>
</li>
</ul>
<h3>サービスレベル指標とサービスレベル目標</h3>
<p>次の SLI と SLO は、エンドユーザー エクスペリエンスと、ユーザーとビジネス目標への理論的影響に基づいて選択されたものです。</p>
<table>
<tr>
<td colspan="1" rowspan="1">
<p><strong>SLI</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>指標</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>説明</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>SLO</strong></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>リクエストのレイテンシ</p>
</td>
<td colspan="1" rowspan="1">
<p>フロントエンドのレイテンシ</p>
</td>
<td colspan="1" rowspan="1">
<p>ユーザーがページの読み込みを待機している時間を測定します。一般的に、レイテンシが長くなるとユーザー エクスペリエンスが低下します。</p>
</td>
<td colspan="1" rowspan="1">
<p>過去 60 分間で 99% のリクエストが 3 秒以内に処理されている</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="3">
<p>エラー率</p>
</td>
<td colspan="1" rowspan="1">
<p>フロントエンドのエラー率</p>
</td>
<td colspan="1" rowspan="1">
<p>ユーザーが経験したエラー率を測定します。エラー率が高い場合、問題が発生している可能性があります。</p>
</td>
<td colspan="1" rowspan="1">
<p>過去 60 分間のエラー数が 0</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>チェックアウトのエラー率</p>
</td>
<td colspan="1" rowspan="1">
<p>チェックアウト サービスを呼び出す他のサービスが経験したエラー率を測定します。エラー率が高い場合、問題が発生している可能性があります。</p>
</td>
<td colspan="1" rowspan="1">
<p>過去 60 分間のエラー数が 0</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>通貨サービスのエラー率</p>
</td>
<td colspan="1" rowspan="1">
<p>通貨サービスを呼び出す他のサービスが経験したエラー率を測定します。エラー率が高い場合、問題が発生している可能性があります。</p>
</td>
<td colspan="1" rowspan="1">
<p>過去 60 分間のエラー数が 0</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>可用性</p>
</td>
<td colspan="1" rowspan="1">
<p>フロントエンドの成功率</p>
</td>
<td colspan="1" rowspan="1">
<p>サービスの可用性の判断基準として、成功したリクエストの割合を測定します。成功率が低い場合、ユーザー エクスペリエンスが低下していることが考えられます。</p>
</td>
<td colspan="1" rowspan="1">
<p>過去 60 分間で 99% のリクエストが成功している</p>
</td>
</tr>
</table>
<h2 id="step7">レイテンシの SLI を構成する</h2>
<p>各 SLO のアラート ポリシーを作成します。モニタリング対象の指標はすでに収集中です。</p>
<h3>フロントエンドのレイテンシ</h3>
<p>Cloud Platform Console の Cloud モニタリング（<strong>ナビゲーション メニュー</strong> &gt; [<strong>モニタリング</strong>]）で、左側のメニューにある [<strong>アラート</strong>]、[<strong>CREATE POLICY</strong>] の順にクリックします。</p>
<p>アラート ポリシーに「Latency Policy」という名前を付けます。</p>
<p>[<strong>ADD CONDITION</strong>] をクリックします。次に、アラート ポリシーをトリガーするために使用する指標と条件を指定します。</p>
<aside>
ユーザー エクスペリエンスに影響するパフォーマンス問題が発生すると、この条件に基づいてアラートが通知されます。上記のサービスレベル指標とサービスレベル目標<em></em>の表に記載されているとおり、99 パーセンタイルのフロントエンドのレイテンシを SLI として使用します。</aside>
<p>[<strong>Find resource type and metric</strong>] に次の内容を追加した後、プルダウン メニューから次の項目を選択します。</p>
<pre><code class="language-bash prettyprint">custom.googleapis.com/opencensus/grpc.io/client/roundtrip_latency&#x000A;</code></pre>
<p>指標が見つからない場合は、少し待ってからもう一度試してください。</p>
<p><img alt="1st_metric.png" src="https://cdn.qwiklabs.com/Q9mCoMm8SbV8OpM9hYyUTHq4q%2BgeRzSdjVN6JSh2bpg%3D"></p>
<p>[Resource Type] に「global」と入力して選択します。</p>
<p>[<strong>Filter</strong>] をクリックして、[<strong>opencensus_task</strong>] を選択します。最初のデフォルト値をクリックし、[<strong>Apply</strong>] をクリックします。</p>
<p>次に、[Aggregator] を [<strong>99th percentile</strong>] に設定します。</p>
<p>画面は次のようになります。</p>
<p><img alt="stackdriver_latency_policy2.png" src="https://cdn.qwiklabs.com/L%2FqomYsdV7d2UL%2FJPB4TBxcIsX%2FQuu9CDJxeCprryVo%3D"></p>
<p>次に、[<strong>Configuration</strong>] で次のようにオプションを設定します。</p>
<ul>
<li>[Condition triggers if]: <strong>Any time series violates</strong>
</li>
<li>[Condition]: <strong>is above</strong>
</li>
<li>[Threshold]: <strong>500</strong>
</li>
<li>[For]: <strong>Most recent value</strong>
</li>
</ul>
<p><img alt="stackdriver_latency_alert.png" src="https://cdn.qwiklabs.com/hBWdNPw48H0tBaFsb8eUN7XoYBByFl%2F9Tu8TIOOWK3U%3D"></p>
<p>[<strong>Add</strong>] をクリックして通知チャンネルをアラート ポリシーに追加してから、[<strong>Save</strong>] をクリックします。</p>
<p>これで、フロントエンドのレイテンシの SLI をモニタリングするように Cloud モニタリングを構成できました。</p>
<h2 id="step8">可用性の SLI を構成する</h2>
<p>次に、別のアラート ポリシーを作成して、サービスの可用性をモニタリングします。</p>
<h3>フロントエンドの可用性</h3>
<p>まず、ユーザー エクスペリエンスに最も直接的に影響する、フロントエンド サービスのエラー率をモニタリングします。前述のとおり、SLO 違反と見なされるエラーを検討します。エラーが検出された場合にインシデントをトリガーするアラート ポリシーを作成します。</p>
<p>特定のエラーに基づいてインシデントをトリガーする簡単な方法は、ログベースの指標を使用することです。</p>
<p>Cloud Platform Console で、<strong>ナビゲーション メニュー</strong> &gt; [<strong>Logging</strong>] の順に選択します。</p>
<p>次のようにフィルタを構成します。</p>
<ul>
<li>
<p>リソースタイプ（最初のプルダウン メニュー）で [<strong>Kubernetes コンテナ</strong>] を選択する</p>
</li>
<li>
<p>ログレベル（3 番目のプルダウン メニュー）で [<strong>エラー</strong>] を選択する</p>
</li>
<li>
<p>フィルタ検索バーに次のコードを入力します。</p>
</li>
</ul>
<pre><code>label:k8s-pod/app:"currencyservice"&#x000A;</code></pre>
<p><img alt="log_filter.png" src="https://cdn.qwiklabs.com/bdM6GZfnlgDzMBBB9WR0y51CjKgIBVKlcxUuQp0smfU%3D"></p>
<p><strong>注:</strong> サービスが正しく実行されているため、ページには何の結果も表示されません。この後で行う変更により、この結果は変更されます。</p>
<p>[<strong>指標を作成</strong>] をクリックします。</p>
<p>指標に「Error_Rate_SLI」という名前を付け、[<strong>指標を作成</strong>] をクリックしてログベースの指標を保存します。</p>
<p><img alt="log_metric.png" src="https://cdn.qwiklabs.com/fYS9GwP3ML9Y0P9MerLy5yRNiyfv50KwxEmIM%2BTxcqE%3D"></p>
<p>[ログベースの指標] で一番下までスクロールすると、[ユーザー定義の指標] に新しい指標が表示されています。次に、この指標に関するアラートを作成します。行の端にあるその他アイコンをクリックし、[<strong>指標に基づいて通知を作成する</strong>] を選択します。</p>
<p><img alt="stackdriver_log_metric_sli.png" src="https://cdn.qwiklabs.com/iSCnIy88BMHUzG5sROtM6xr%2BhPgTKDFPdDYtg78fMYs%3D"></p>
<p>リソースのタイプと指標はすでに入力されています。</p>
<p>指標に「invalid」と表示されている場合は、ブラウザの画面を更新します。</p>
<p>条件に「Error Rate SLI」という名前を付けます。</p>
<p>[<strong>SHOW ADVANCED OPTIONS</strong>] をクリックし、以下のように設定します。</p>
<ul>
<li>Aligner: <strong>rate</strong>
</li>
</ul>
<p><strong>Configuration</strong> で <strong>Threshold</strong> の値を <strong>1 分間、<strong>[<strong>0.5</strong>] に設定します。</strong></strong></p>
<p>[<strong>SAVE</strong>] をクリックして条件を保存します。</p>
<p>その次の画面で、新しいポリシーに「Error Rate SLI」という名前を付けて<strong>保存</strong>します。</p>
<p>期待どおりエラーはなく、現在のところアプリケーションは可用性の SLO を達成しています。</p>
<h2 id="step9">新しいリリースをデプロイする</h2>
<p>以上で SLI のモニタリングを構成できたので、アプリケーションの変更がユーザー エクスペリエンスに与える影響を測定する準備が整いました。アプリケーションの新しいリリースをデプロイすると何が起こるかを見てみましょう。</p>
<p>次に、新しいリリースを含む Service の Kubernetes マニフェストを変更した後、skaffold を実行してアプリケーションを再度デプロイします。</p>
<h3>YAML ファイルを更新する</h3>
<p>必要に応じて Cloud Shell を再度有効にしてから、Cloud Shell でコードエディタを起動します。</p>
<p><img alt="ba731110a97f468f.png" src="https://cdn.qwiklabs.com/O1pSCpaSe6p5nxOMmjQg3Vsf0YwHaqW2bT56hM6Iym0%3D"></p>
<p><strong>microservices-demo-1</strong> フォルダを開いて、その中の <strong>kubernetes-manifests</strong> フォルダを開きます。</p>
<p><code>kubernetes_manifests/recommendationservice.yaml</code> を開いて、28 行目を次の内容で置き換えます。</p>
<p>次の 3 つのファイルを更新します。</p>
<ul>
<li><code>kubernetes_manifests/recommendationservice.yaml</code></li>
<li><code>kubernetes_manifests/currencyservice.yaml</code></li>
<li><code>kubernetes_manifests/frontend.yaml</code></li>
</ul>
<p><code>image</code> を <code>rel013019</code> に変更して、「<code>imagePullPolicy: Always</code>」という行を追加します。</p>
<p>例: <code>recommendationservice.yaml</code> ファイル（更新前）</p>
<p><img alt="be6fd76ca59d64db.png" src="https://cdn.qwiklabs.com/F1zJer0%2B38xl4Jl%2Bkjn76heBtcWLi86wtxkwHyWn8SQ%3D"></p>
<p>更新後には次のようになります。</p>
<p><img alt="caf0b2c0654e5973.png" src="https://cdn.qwiklabs.com/aoTXKMxnoxXUGW%2BgYE6Ag5TfHggrmv9epBzbU3B6KT4%3D"></p>
<p>他の 2 つのファイルも同様に更新します。</p>
<ul>
<li>
<code>kubernetes_manifests/currencyservice.yaml</code>（28 行目）</li>
<li>
<code>kubernetes_manifests/frontend.yaml</code>（27 行目）</li>
</ul>
<p>更新した各ファイルを保存して閉じます。これで、新しいバージョンをデプロイする準備が整いました。</p>
<h3>新しいバージョンをデプロイする</h3>
<p>Cloud Shell で、デプロイメントを更新して新しいコンテナ イメージをデプロイします。</p>
<pre><code class="language-bash prettyprint">skaffold run&#x000A;</code></pre>
<p>新しいバージョンのサービスが実行されていることを確認します。</p>
<pre><code class="language-bash prettyprint">kubectl get pods&#x000A;</code></pre>
<p>（出力）</p>
<pre><code class="language-bash prettyprint">NAME                                     READY     STATUS    RESTARTS   AGE&#x000A;adservice-55f94cfd9c-4lvml               1/1       Running   0          17d&#x000A;cartservice-6f4946f9b8-6wtff             1/1       Running   197        17d&#x000A;checkoutservice-5688779d8c-l6crl         1/1       Running   0          17d&#x000A;currencyservice-665d6f4569-b4sbm         1/1       Running   0          1m&#x000A;emailservice-684c89bcb8-h48sq            1/1       Running   0          17d&#x000A;frontend-5f889fc7bb-wvfvv                1/1       Running   0          1m&#x000A;loadgenerator-6d646566db-p422w           1/1       Running   0          17d&#x000A;paymentservice-858d89d64c-hmpkg          1/1       Running   0          17d&#x000A;productcatalogservice-bcd85cb5-d6xp4     1/1       Running   0          17d&#x000A;recommendationservice-57cb4559f9-bdgj7   1/1       Running   0          1m&#x000A;redis-cart-9b864d47f-c9xc6               1/1       Running   0          17d&#x000A;shippingservice-5948f9fb5c-vndcp         1/1       Running   0          17d&#x000A;</code></pre>
<h2 id="step10">データを送信する</h2>
<p>アプリケーションが実行されているので、デプロイしたものを確認します。</p>
<p>Cloud Platform Console で、[<strong>Kubernetes Engine</strong>] &gt; [<strong>Services と Ingress</strong>] に移動します。<code>frontend-external</code> サービスを見つけて、エンドポイント URL をクリックします。</p>
<p>Hipster Shop ウェブサイトが表示されたら、トラフィックを送信するために、いくつかの商品を選んで [<strong>Buy</strong>] または [<strong>Add to Cart</strong>] をクリックします。十分な量のレイテンシ データが生成されるまで、約 60 秒間このウェブサイトにとどまります。</p>
<h2 id="step11">レイテンシの SLO 違反 - 問題を見つける</h2>
<p>この演習では、Cloud モニタリング アプリケーション パフォーマンス管理（APM）ツールを使用して、アプリケーションのレイテンシの増加を引き起こす問題を特定して解決します。</p>
<p>はじめに、新しいバージョンをデプロイした後、アプリケーションに問題が起こっていないかを確認します。</p>
<p>Cloud Platform Console で <strong>Cloud モニタリング</strong> に戻ります（<strong>ナビゲーション メニュー</strong> &gt; [<strong>モニタリング</strong>]）。上部のリボンにある<strong>自動更新の矢印</strong>をクリックすると、常に最新の情報が表示されます。</p>
<p>レイテンシのポリシーに関するインシデントが表示されていなくても、しばらくすると表示されます。数分お待ちください。</p>
<p><img alt="Stackdriver_latency_policy.png" src="https://cdn.qwiklabs.com/d2mEd%2BUDydg6BT%2BnqrMSNhsQd66OCEN9S3JtXjeU6W0%3D"></p>
<p>状況を把握するには、左側のメニューで [アラート] をクリックし、[Incidents] でアラートをクリックします。場合によっては、[<strong>ACKNOWLEDGED</strong>] タブをクリックしてアラートの発生を確認する必要があります。</p>
<p>レイテンシの問題を分析する最良の方法は、Trace を使用することです。<strong>ナビゲーション メニュー</strong> &gt; [<strong>トレース</strong>] の順にクリックします。</p>
<p>最初の概要には役立つ情報が記載されていますが、ここでは次の詳細レベルに進みます。左側のメニューで [<strong>トレースリスト</strong>] をクリックします。</p>
<p>[<strong>自動再読み込み</strong>] をクリックします。ページ上部の散布図に注目してください。アラートが表示されている時間帯には、リクエストに多数の外れ値があります。</p>
<p>データが収集されるまで 1～2 分待ってから、いずれかの外れ値トレースをクリックして詳細を確認します。</p>
<p><img alt="6a05ee60a0daa87.png" src="https://cdn.qwiklabs.com/ZQJTzVEsrg0SpmO%2BtTjxjCJWAF8ahvmZYoy%2FqzcAMpU%3D"></p>
<p>スパン名（呼び出されているサービスまたは関数を表します）が /cart/ または /cart/checkout/ のいずれかであることに注目してください。</p>
<p>このトレースと問題発生前の類似のトレースを比較して違いを理解できるよう、すべてのカート オペレーションの左下にある「Recv./cart」サマリーで類似のトレースを探します。</p>
<p>上部で期間を [<strong>1 時間</strong>] に設定して、問題発生前のトレースが含まれるようにします。</p>
<p><img alt="64c6982957db838c.png" src="https://cdn.qwiklabs.com/xQGX3Q%2B4iSOhxvp6R58eQlmoy4Zzc3ssulS3ymiFRxo%3D"></p>
<p>問題発生前のトレースのいずれかをクリックします。</p>
<p><img alt="9b446892a7770fd8.png" src="https://cdn.qwiklabs.com/H2VPCEvIhUGweaqHH0nlD68a%2FoOs7r%2FO9y2nFiZTM2M%3D"></p>
<p>この類似のトレースでは、ListRecommendations が一度だけ呼び出されています。しかし、最新のデプロイの後では、ListRecommendations はリクエストごとに何度も呼び出されており、そのためにかなりのレイテンシが発生しています。</p>
<p>これにより、上記の外れ値の問題は ListRecommendations への複数回の呼び出しによって引き起こされていると結論付けられます。</p>
<h2 id="step12">変更をデプロイしてレイテンシを解決する</h2>
<p>最新のリリースで発生したレイテンシの問題を解決するには、誤りのあるコードを修正した別のバージョンをロールアウトする必要があります。誤りのあるコードを含む Service の Kubernetes マニフェストを修正します。</p>
<p>修正をデプロイするには、<strong>ソースコード エディタ</strong>が開いている Cloud Shell タブに戻ります。次のファイルを修正します。</p>
<ul>
<li><code>kubernetes_manifests/recommendationservice.yaml</code></li>
<li><code>kubernetes_manifests/frontend.yaml</code></li>
</ul>
<p>イメージタグ <code>rel013019</code> を <code>rel013019fix</code> に置き換えて修正し、<code>image</code> が次のようになるようにします。</p>
<pre><code class="language-bash prettyprint">      containers:&#x000A;      - name: server&#x000A;        image: gcr.io/accl-19-dev/frontend:rel013019fix&#x000A;        imagePullPolicy: Always&#x000A;</code></pre>
<p><img alt="5602175997ba2cd0.png" src="https://cdn.qwiklabs.com/e6Sr8zxShjB4NnUhmqbj1%2FhVnLtDzOdrInmDsXOlR7s%3D"></p>
<p>ファイルを<strong>保存</strong>します。</p>
<p>Cloud Shell プロンプトに戻り、修正したイメージを再デプロイします。</p>
<pre><code class="language-bash prettyprint">skaffold run&#x000A;</code></pre>
<h3>修正を検証する</h3>
<p>以上で破壊的変更をロールバックできたので、アプリケーションが正常な状態に戻ったことを確認します。</p>
<p><strong>Metrics Explorer</strong> に戻ります（<strong>ナビゲーション メニュー</strong> &gt; [<strong>モニタリング</strong>] &gt; [<strong>Metrics Explorer</strong>]）。</p>
<p>検索欄に「<strong>latency</strong>」と入力し、<strong>roundtrip_latency</strong> をクリックします。</p>
<p><img alt="185dd877b3d55bf.png" src="https://cdn.qwiklabs.com/Uql7RZQJybUqQ1rDJrrLFLlsqwEqxkgL4NXrZo8L6Fk%3D"></p>
<p>グラフの種類を [<strong>Line</strong>] に変更します。レイテンシがすぐに短縮されたことがグラフから見てとれるはずです（そうでない場合は少しお待ちください）。</p>
<p><img alt="69ba9bd7c87115f9.png" src="https://cdn.qwiklabs.com/QQhDauFF%2FFqZy10MoK1u1mS0S9SWnFYc9PdjQyVPcbk%3D"></p>
<p>これでインシデントが解決されました。[<strong>モニタリングの概要</strong>] に戻ります。</p>
<p>ここであなたは 2 つの点に気づくはずです。それは、インシデントがもうなくなったことと、インシデントが解決されたことをあなたに知らせるイベントがあることです。インシデント解決のメッセージが届かない場合は、数分お待ちください。</p>
<p><img alt="ec64e9f0da94d7ed.png" src="https://cdn.qwiklabs.com/K5qWQ8LJ8rZTuZwbn46pe3jS6%2BFWZi9GC58%2FcqRTsyw%3D"></p>
<p>以上で、ユーザー エクスペリエンス（尺度はレイテンシ）が低下する原因となった変更をモニタリングによって正確に特定し、根本原因を見つけて破壊的変更をロールバックすることができました。次のセクションでは、Cloud モニタリングが可用性の問題の解決にどのように役立つかを見ていきましょう。</p>
<h2 id="step13">エラー率の SLO 違反 - 問題を見つける</h2>
<p>この演習では、Cloud モニタリング アプリケーション パフォーマンス管理ツール（APM）を使用して、アプリケーションがエラー バジェットに違反するエラーを引き起こしている問題のトラブルシューティングを行います。</p>
<p>まず、Cloud Platform Console で <strong>Cloud モニタリング</strong> に移動します。</p>
<p>エラー率の SLI に関するインシデントを見つけてその<strong>インシデント</strong>をクリックし、何が起こっているかを調べます。インシデントが確認されてインシデントとしてリストされるまでに、数分かかることがあります。インシデントがまだ表示されていない場合は、以下のインシデント手順をスキップして、左側のメニューの [<strong>Error Reporting</strong>] をクリックしてください。</p>
<p><strong>currencyservice</strong> Pod で、以前よりはるかに多くのエラーがロギングされていることがわかります。</p>
<p>[<strong>確認</strong>] をクリックして、これ以上通知のエスカレーションが発生しないようにします。</p>
<p>この種のアラートに対処する方法は数多くありますが、最も簡単なのは Cloud Error Reporting です。<strong>ナビゲーション メニュー</strong> &gt; [<strong>Error Reporting</strong>] の順に選択します。</p>
<p>[Error Reporting を開く] インシデントが最近急増していることに注目してください。問題になっているエラーを詳しく調べるため、[<strong>Error: Conversion is Zero</strong>] をクリックします。</p>
<p><img alt="error_reporting1.png" src="https://cdn.qwiklabs.com/psHiDwAJDAwtp9h1rO8jHChNs7kI4payYMrmrfeGCLE%3D"></p>
<p>右側の Stack Trace サンプルをご覧ください。どの呼び出しがエラーに関連しているかを確認できます。<img alt="error_reporting3.png" src="https://cdn.qwiklabs.com/XrL6I8hLQtzZJHgwFHC9el0QZKJ72WmwZ2plBz9hymQ%3D"></p>
<p>表示されている一番下の呼び出し（<strong>/usr/src/app/server/js:131</strong>）を<strong>クリック</strong>します。</p>
<p>これにより、Cloud Debug に誘導されます。上部のバーで通貨サービスが選択されていることを確認します。</p>
<p><img alt="dcb90f0fce431ec.png" src="https://cdn.qwiklabs.com/8gHA8t%2FlCE42mcXMHzJvlG2HLXIYGNazuvJepHQTpAQ%3D"></p>
<p>次に、<strong>Cloud Source Repositories</strong> から実行されているソースコードを選択します。</p>
<p><img alt="97ba51694adc967b.png" src="https://cdn.qwiklabs.com/EaOhOK3AZYY%2B3MSSkP9UCsm5Ns%2FF57bmLU%2FNpSBP6aI%3D"></p>
<p>次のソースを選択します。</p>
<ul>
<li>
<strong>リポジトリ:</strong> apm-qwiklabs-demo</li>
<li>
<strong>タグ付きのバージョンまたはブランチ:</strong> APM-Troubleshooting-Demo-2</li>
</ul>
<p>[<strong>ソースを選択</strong>] をクリックします。</p>
<p>左側のメニューで <code>/src/currencyservice/server.js</code> をブラウジングします。</p>
<p><img alt="267a2773a8fbce8.png" src="https://cdn.qwiklabs.com/CFPsZG0ikug%2FaeXQIkDhzsb5XQjUGjt5KKniOq3e7fQ%3D"></p>
<p>155 行目付近まで下にスクロールし、例外がスローされた関数を見つけます。エラー報告に出現するログの行（<strong>Conversion is Zero</strong>）を確認できます。</p>
<p><img alt="6bc6f0c7f2f436e1.png" src="https://cdn.qwiklabs.com/ogdwAif0tx%2BpUuYyxwCUO4thqdhF71SrHaM64npBVLM%3D"> <img alt="fff6e7e964cf3ac6.png" src="https://cdn.qwiklabs.com/mapFAd1G9zG3wfX%2BAtv%2BpBJ0lPs0bDmbBcIU9ZPmswk%3D"></p>
<p>上のコード スニペットから、<code>result.units</code> <code>&lt; 0</code> の場合にこのエラーがロギングされたことがわかります。この問題をトラブルシューティングするには、スナップショットを使用して、アプリケーションの実行中に変数を調査します。</p>
<p>右上で [<strong>スナップショット</strong>] が選択されていることを確認します。</p>
<p><img alt="97347c0fd561f8a3.png" src="https://cdn.qwiklabs.com/ZS9ugTt%2BvJWd0oZ5BZeqo%2B6eikt217P8EEbutFGUQso%3D"></p>
<p>スナップショットを作成する行番号（155）をクリックします。</p>
<p><img alt="774be2495268d15d.png" src="https://cdn.qwiklabs.com/FDN%2BGH9MQRInb0LMlUbrFnq7pj3fltXiMExN9jwOBNs%3D"></p>
<p>この演習では、155、141、149 の各行でスナップショットを作成します。適切と思われる場所にスナップショット ポイントを追加してください。次にコードが実行されたとき、変数のスナップショットが作成されます。次にコードが実行されるまでアプリケーションが待機している間、「スナップショットがヒットするまで待機しています…」という通知が表示されます。</p>
<p>スナップショットの作成が完了すると、右側のペインに該当スナップショットの変数が表示されます。</p>
<p><img alt="74a796c219d2ad97.png" src="https://cdn.qwiklabs.com/k%2FEuNZmYh6u3SO2JzzpQLrAtKWvzGd0OwcnfpMOBKMg%3D"></p>
<p>変数とコールスタックの情報に注目してください。この情報は、コードがたどるパスと、パスをたどる際にコードが遭遇する変数や構造を理解するために大変役立ちます。しかも、アプリケーションの再起動やコードの変更は不要です。</p>
<p><strong>結果</strong>をクリックして、155 行目までの 3 つのスナップショットをすべて調査します。<code>result.units</code> が NOT &gt; 0 の場合にエラーがトリガーされることを思い出してください。変数を調べると、<code>result.units = NaN</code>（not a number）である、つまり数値でないことがわかります。これがエラーを引き起こしている原因です。</p>
<p><img alt="cf9717dbcf09659a.png" src="https://cdn.qwiklabs.com/NDRrhVbUN22y2Rr1OCCb38q3giI20oJ%2FNb3CBpCdSjw%3D"></p>
<p>この時点で、<code>result.units</code> を <code>0</code> に設定する変換（または子）関数のバグにより、変換中の商品に 0.00 の価格タグが付けられたことがエラーの原因であると結論付けられます。スナップショット情報とログを使用したトラブルシューティングは、問題の確実な診断を可能にします。</p>
<p>では、この問題を引き起こしたバグはどのようなものでしょうか。コードを見ると、<code>result.units</code> は 144 行目のユーロで設定されており、それは 136 行目の <code>from.units</code> の演算で設定されています。</p>
<p><img alt="bef885a4cb6d54f7.png" src="https://cdn.qwiklabs.com/BTmylaJpMc7Djtk8wXIrOHaTkaDaB%2BBERl2p7zh8xBY%3D"></p>
<p>スナップショットを調べると、<code>euros.units</code> も <code>NaN</code> ですが、<code>from.units</code> は有効な数値です。そのために、<code>from.units</code> をユーロに変換するときに問題が発生したのです。</p>
<p><img alt="1cbc42017f0e4e76.png" src="https://cdn.qwiklabs.com/fq%2FCfblgUKVisU3UnHAlEamKSbABMrYAaY85UdTRydk%3D"></p>
<p>根本原因は、137 行目の <code>from.units</code> を <code>euros.units</code> に変換する処理のバグであると結論付けることができます。「<code>8</code>」は <code>Data[]</code> に渡されますが、これは実際には通貨単位（EUR など）から為替レートへの Key-Value マッピングです。137 行目は、<code>from.units</code>（つまり 8）の代わりに <code>from.to_currency</code>（つまり USD）を使用するように修正できます。</p>
<p><img alt="e79ad235feef0dcf.png" src="https://cdn.qwiklabs.com/XaieKVHTltMlamxM2K2fY3GxDUVuP%2F6ZWWFxUePpLXI%3D"></p>
<p>以上でバグの原因が特定できたので、適切な変更を加えることができます。アラートの時期からすると、この問題は最新のデプロイメントによって引き起こされた可能性があります。</p>
<p>以前のブランチ「Master」の 137 行目にこのコードエラーがあるかどうかを見てみましょう。</p>
<p>コンソールに戻り、<strong>Source Repositories</strong>（ナビゲーション メニューの [ツール] 下にあります）でコードを調べます。</p>
<p><strong>apm-qwiklabs-demo</strong> リポジトリを開いて <strong>master</strong> ブランチを選択します。</p>
<p>左側で <strong>src</strong> &gt; <strong>currencyservice</strong> &gt; <strong>server.js</strong> をブラウジングします。137 行目では、適切な被除数 <code>data[from.currency_code]</code> が使用されていることがわかります。</p>
<p><img alt="13375108606a76d7.png" src="https://cdn.qwiklabs.com/sUa8V3WgufkeQT6KR69jqXf8gC9bNetgbNYAPlP%2F1d0%3D"></p>
<p>これで、このバグが最新のデプロイにより取り込まれたことが確認できました。この問題に対処するには、以前のバージョンにロールバックする必要があります。</p>
<h2 id="step14">変更をデプロイしてエラー率を解決する</h2>
<p>この問題を解決するには、アプリケーションを修正する必要があります。そのためには、誤りのあるコードを含むサービスの Kubernetes マニフェストを修正する必要があります。</p>
<h3>修正をデプロイする</h3>
<p><strong>Cloud Shell</strong> に戻り、<strong>ソースコード エディタ</strong>で <strong>kubernetes_manifests</strong> フォルダの <strong>currencyservice.yaml</strong> ファイルを開きます。</p>
<p><img alt="78698b11c80697f.png" src="https://cdn.qwiklabs.com/6q7HreHq0%2FMF4qfqXmHiNWEPEESa3gDbiVOkOylq4Qg%3D"></p>
<p>イメージタグ __<code>rel013019</code>__ を __<code>rel013019fix</code>__ に置き換え、<code>image</code> が次のようになるようにします。</p>
<pre><code class="language-bash prettyprint">      containers:&#x000A;      - name: server&#x000A;        image: gcr.io/accl-19-dev/frontend:rel013019fix&#x000A;        imagePullPolicy: Always&#x000A;</code></pre>
<p>ファイルを<strong>閉じて</strong>保存し、Cloud Shell プロンプトに<strong>戻ります</strong>。</p>
<p>修正したイメージを再デプロイします。</p>
<pre><code class="language-bash prettyprint">skaffold run&#x000A;</code></pre>
<h3>修正を検証する</h3>
<p>これで破壊的変更をロールバックできたので、アプリケーションが正常な状態に戻ったことを確認します。</p>
<p>前と同様に、まずはインシデントが解決されているかどうかを検証します。Cloud Platform Console で <strong>Cloud モニタリング</strong> に移動し、エラー率インシデントが解決されたことを確認します。この時点で、オープンなインシデントはなくなっているはずです。</p>
<p><img alt="monitoring.png" src="https://cdn.qwiklabs.com/JtV9vu0TysQVEXjtSJ4vVHoQFo2DNKrDYvepROis%2BiE%3D"></p>
<p>次に、[<strong>Error Reporting</strong>] に戻ります。前に見つけたエラーを開いて、今では発生していないことを確認します（タイムラインを見れば、最新のデプロイ以降は発生していないことがわかります）。</p>
<p><img alt="error_reporting.png" src="https://cdn.qwiklabs.com/upQrgC4g8%2Fh075tdKOp6rJ0KK%2F6Nh2rDnZICarLzuMk%3D"></p>
<p>以上で、モニタリングは成功です。つまり、ユーザー エクスペリエンス（尺度はアプリケーション エラー数）の低下を引き起こした変更を正確に特定し、根本原因を突き止め、破壊的変更をロールバックできました。</p>
<p>Cloud モニタリング を使用してリソース使用率を最適化する方法を学んでください。</p>
<h2 id="step15">Cloud モニタリング APM によるアプリケーションの最適化</h2>
<p>この演習では、Cloud モニタリング アプリケーション パフォーマンス管理ツール（APM）を使用し、アプリケーションの実行速度を上げてコンピューティング リソースの使用量を減らすための改善策を見つけます。</p>
<p>このシナリオに登場するクラウド オペレーション部門の責任者は、最近のコンピューティング コストの上昇に不満を抱いています。具体的には、<strong>currencyservice</strong> サービスによる CPU の使用量が、システムの使用状況から予想される量より多くなっていることが問題です。</p>
<p>あなたのチームは最適化の方策を見出すタスクを任されています。ここでは APM ツールを使用してサービスを分析し、アプリケーションの適切な領域で作業を進める必要があります。</p>
<p>Cloud Platform Console の左側のメニューで、Cloud モニタリング の [<strong>プロファイラ</strong>] を開きます。</p>
<p><img alt="1607c72d45a41947.png" src="https://cdn.qwiklabs.com/Qif%2B5Hn6VWW9DAV7vuErMACDWO5mKfpqyEhLNTUS%2FHw%3D"></p>
<p>右上の [期間] を [30 分] に変更します。データがない場合は、データが取り込まれるまで 2 分ほど待ちます。</p>
<p><strong>注:</strong> プロファイラは、集計コールスタックを構築するために、呼び出しのランダムなサンプルを選択します。予想したデータが表示されない場合がありますが、それはこのラボで十分な時間が経過しておらず、サンプルの選択が予想よりも早く完了したためです。演習では、代わりに下のスクリーンショットを使用してください。</p>
<p>フィルタのオプションで、<strong>フロントエンド</strong> サービスと、<strong>CPU 時間</strong>プロファイル タイプを選択します。</p>
<p><img alt="profiler.png" src="https://cdn.qwiklabs.com/LyXkORgSj9biwIiO1gP4auLf8OWUcpNJQh%2Bu3oEKJy4%3D"></p>
<p>プロファイラは、システムのランダムなサンプル プロファイルを選択し、データを組み合わせて、どの関数が最も多くリソースを使用しているかを表示します。以下のフレームグラフは、リソース（この場合は CPU）の使用状況別にグループ化された関数呼び出しを示しています。X 軸は CPU 使用量、Y 軸は親子関係を表します。</p>
<p><img alt="profiler1.png" src="https://cdn.qwiklabs.com/OCBFVc4vEm6tGaP0YngaWumK5ZKSi%2FQ89U%2FAMLMx8aM%3D">この場合、CPU の大部分は左側の ServeHTTP 呼び出しによって使用されています。この呼び出しをクリックして原因を詳しく調べます。</p>
<p><img alt="profiler2.png" src="https://cdn.qwiklabs.com/fsb34xta9X4R5bokB1UYE37sTx2ox%2FqdqtQwTldhlrM%3D"></p>
<p>ビューを展開すると、その半数近くが <strong>viewCartHandler</strong> によるもので、これは主に <strong>getRecommendations</strong> によって発生していることがわかります。</p>
<p>改善の対象となる箇所は <strong>getRecommendations</strong> であり、さらには <strong>getProduct</strong> です。前の演習を振り返ってください。そこでは、再試行ロジックのエラーのため、推奨サービスと getProduct がループ内で頻繁に呼び出されていました。この問題を解決すれば、コンピューティング コストを 20% ほど削減できる可能性があります。</p>
<h2 id="step16">お疲れさまでした</h2>