**Kubernetes in Google Cloud**

# Docker の概要
<h2 id="step2">概要</h2>
<p>Docker は、アプリケーションを開発、リリース、実行するためのオープン プラットフォームです。Docker を使用すると、アプリケーションをインフラストラクチャから分離して、インフラストラクチャをマネージド アプリケーションのように扱うことができます。また、コードのリリース、テスト、デプロイを高速化し、コード作成とコード実行のサイクルを短縮できます。</p>
<p>Docker は、カーネル コンテナ化機能と、アプリの管理とデプロイをサポートするワークフローおよびツールを組み合わせることでこれを実現します。</p>
<p>Docker コンテナは Kubernetes で直接使用できるため、Kubernetes Engine で簡単に実行できます。Docker の基本を学ぶことで、Kubernetes とコンテナ化アプリケーションを開発するスキルが身に付きます。</p>
<h3>ラボの内容</h3>
<p>このラボでは、次の方法について学びます。</p>
<ul>
<li>Docker コンテナをビルド、実行、デバッグする方法</li>
<li>Docker Hub と Google Container Registry から Docker イメージを pull する方法</li>
<li>Docker イメージを Google Container Registry に push する方法</li>
</ul>

<h2 id="step4">Hello World</h2>
<p>まずは、Cloud Shell を開いて次のコマンドを入力し、hello world コンテナを実行します。</p>
<pre><code>docker run hello-world&#x000A;</code></pre>
<p>（コマンド出力）</p>
<pre><code class="language-bash prettyprint">Unable to find image 'hello-world:latest' locally&#x000A;latest: Pulling from library/hello-world&#x000A;9db2ca6ccae0: Pull complete&#x000A;Digest: sha256:4b8ff392a12ed9ea17784bd3c9a8b1fa3299cac44aca35a85c90c5e3c7afacdc&#x000A;Status: Downloaded newer image for hello-world:latest&#x000A;&#x000A;Hello from Docker!&#x000A;This message shows that your installation appears to be working correctly.&#x000A;...&#x000A;</code></pre>
<p>この単純なコンテナから返された <code>Hello from Docker!</code> が 表示されます。コマンド自体はシンプルですが、出力内で、コマンドによって実行されたステップ数に注目してください。Docker デーモンは hello-world イメージを検索し、ローカルでは見つけることができませんでした。そこで Docker Hub という一般公開レジストリからイメージを pull し、そのイメージからコンテナを作成して実行しています。</p>
<p>次のコマンドを実行して、Docker Hub から pull されたコンテナ イメージを確認します。</p>
<pre><code>docker images&#x000A;</code></pre>
<p>（コマンド出力）</p>
<pre><code class="language-bash prettyprint">REPOSITORY     TAG      IMAGE ID       CREATED       SIZE&#x000A;hello-world    latest   1815c82652c0   6 days ago    1.84 kB&#x000A;</code></pre>
<p>これは、Docker Hub 一般公開レジストリから pull されたイメージです。イメージ ID は <a href="https://www.movable-type.co.uk/scripts/sha256.html">SHA256 ハッシュ</a>形式で示され、このフィールドではプロビジョニング済みの Docker イメージが指定されます。Docker デーモンがローカルでイメージを見つけることができない場合、デフォルトで一般公開レジストリを検索します。もう一度コンテナを実行してみましょう。</p>
<pre><code>docker run hello-world&#x000A;</code></pre>
<p>（コマンド出力）</p>
<pre><code class="language-bash prettyprint">Hello from Docker!&#x000A;This message shows that your installation appears to be working correctly.&#x000A;&#x000A;To generate this message, Docker took the following steps:&#x000A;...&#x000A;</code></pre>
<p>この 2 回目の実行で、Docker デーモンはローカル レジストリでイメージを見つけ出し、そのイメージからコンテナを実行します。Docker Hub からイメージを pull する必要はありません。</p>
<p>最後に、次のコマンドを使用して実行中のコンテナを確認します。</p>
<pre><code>docker ps&#x000A;</code></pre>
<p>（コマンド出力）</p>
<pre><code class="language-bash prettyprint">CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES&#x000A;</code></pre>
<p>実行中のコンテナはありません。以前実行した hello-world コンテナはすでに終了しています。終了したものを含むすべてのコンテナを表示するには、<code>docker ps -a</code> を実行します。</p>
<pre><code>docker ps -a&#x000A;</code></pre>
<p>（コマンド出力）</p>
<pre><code class="language-bash prettyprint">CONTAINER ID      IMAGE           COMMAND      ...     NAMES&#x000A;6027ecba1c39      hello-world     "/hello"     ...     elated_knuth&#x000A;358d709b8341      hello-world     "/hello"     ...     epic_lewin&#x000A;</code></pre>
<p>この出力では、<code>コンテナ ID</code>（コンテナの識別用に Docker によって生成された UUID）と、実行に関する補足のメタデータが表示されます。コンテナの<code>名前</code>はランダムに生成されますが、<code>docker run --name [コンテナ名] hello-world</code> で指定することもできます。</p>
<h2 id="step5">ビルド</h2>
<p>単純な node アプリケーションに基づく Docker イメージをビルドしましょう。次のコマンドを実行して <code>test</code> という名前のフォルダを作成し、このフォルダに移動します。</p>
<pre><code>mkdir test &amp;&amp; cd test&#x000A;</code></pre>
<p><code>Dockerfile</code> を作成します。</p>
<pre><code>cat &gt; Dockerfile &lt;&lt;EOF&#x000A;# 上位イメージとして正式な Node ランタイムを使用します&#x000A;FROM node:6&#x000A;&#x000A;# コンテナの作業ディレクトリを /app に設定します&#x000A;WORKDIR /app&#x000A;&#x000A;# 現行ディレクトリの内容を /app のコンテナにコピーします&#x000A;ADD . /app&#x000A;&#x000A;# コンテナのポート 80 で外部からアクセスできるようにします&#x000A;EXPOSE 80&#x000A;&#x000A;# コンテナの起動時に node を使用して app.js を実行します&#x000A;CMD ["node", "app.js"]&#x000A;EOF&#x000A;</code></pre>
<p>このファイルは、Docker デーモンにイメージのビルド方法を指示するものです。</p>
<ul>
<li>最初の行で、ベースとなる上位イメージを指定します。この例では、node バージョン 6 の正式な Docker イメージです。</li>
<li>2 行目でコンテナの作業（現行）ディレクトリを設定します。</li>
<li>3 行目では、現行ディレクトリの内容（<code>"."</code> で示す）をコンテナに追加します。</li>
<li>次にコンテナのポートを公開してこのポートでの接続を受け入れ、最後に node コマンドを実行してアプリケーションを起動します。</li>
</ul>
<aside class="special"><p><code>Dockerfile</code> の各行については、<a href="https://docs.docker.com/engine/reference/builder/#known-issues-run" target="blank">Dockerfile コマンド リファレンス</a>をご確認ください。</p>
</aside>
<p>次に node アプリケーションを作成し、その後はイメージのビルドに進みます。</p>
<p>次のコマンドを実行して node アプリケーションを作成します。</p>
<pre><code>cat &gt; app.js &lt;&lt;EOF&#x000A;const http = require('http');&#x000A;&#x000A;const hostname = '0.0.0.0';&#x000A;const port = 80;&#x000A;&#x000A;const server = http.createServer((req, res) =&gt; {&#x000A;    res.statusCode = 200;&#x000A;    res.setHeader('Content-Type', 'text/plain');&#x000A;    res.end('Hello World\n');&#x000A;});&#x000A;&#x000A;server.listen(port, hostname, () =&gt; {&#x000A;    console.log('Server running at http://%s:%s/', hostname, port);&#x000A;});&#x000A;&#x000A;process.on('SIGINT', function() {&#x000A;    console.log('Caught interrupt signal and will exit');&#x000A;    process.exit();&#x000A;});&#x000A;EOF&#x000A;</code></pre>
<p>これは、ポート 80 でリッスンし、「Hello World」を返す単純な HTTP サーバーです。</p>
<p>ではイメージのビルドに進みましょう。</p>
<p>前述したように、<code>“.”</code> は現行ディレクトリを示すため、Dockerfile が含まれるディレクトリ内から次のコマンドを実行する必要があります。</p>
<pre><code>docker build -t node-app:0.1 .&#x000A;</code></pre>
<p>このコマンドが完了するまでに数分かかる場合があります。完了すると、次のような出力が表示されます。</p>
<pre><code>Sending build context to Docker daemon 3.072 kB&#x000A;Step 1 : FROM node:6&#x000A;6: Pulling from library/node&#x000A;...&#x000A;...&#x000A;...&#x000A;Step 5 : CMD node app.js&#x000A; ---&gt; Running in b677acd1edd9&#x000A; ---&gt; f166cd2a9f10&#x000A;Removing intermediate container b677acd1edd9&#x000A;Successfully built f166cd2a9f10&#x000A;</code></pre>
<p><code>-t</code> は、<code>name:tag</code> 構文を使用してイメージに名前とタグを付けることを意味します。イメージの名前は <code>node-app</code> で<code>タグ</code>は <code>0.1</code> です。Docker イメージをビルドする際には、タグの指定を強くおすすめします。指定しないと、タグはデフォルトで <code>latest</code> になり、新しいイメージと古いイメージを区別するのが難しくなります。また、上記の <code>Dockerfile</code> の各行で、イメージのビルドに伴って中間コンテナレイヤが生成されている点もご確認ください。</p>
<p>次のコマンドを実行して、ビルドしたイメージを確認します。</p>
<pre><code>docker images&#x000A;</code></pre>
<p>出力は次のようになります。</p>
<pre><code>REPOSITORY     TAG      IMAGE ID        CREATED            SIZE&#x000A;node-app       0.1      f166cd2a9f10    25 seconds ago     656.2 MB&#x000A;node           6        5a767079e3df    15 hours ago       656.2 MB&#x000A;hello-world    latest   1815c82652c0    6 days ago         1.84 kB&#x000A;</code></pre>
<p><code>node</code> はベースイメージで、ビルドしたイメージは <code>node-app</code> です。<code>node</code> を削除する場合は、先に <code>node-app</code> を削除する必要があります。イメージのサイズは VM と比較すると相対的に小さくなっており、<code>node:slim</code> や <code>node:alpine</code> といった他のバージョンの node イメージはさらに小さいため、簡単に移植できます。コンテナのサイズを小さくする方法は、上級者向けトピックに詳しく記載されています。<a href="https://hub.docker.com/_/node">こちら</a>の公式リポジトリですべてのバージョンを確認できます。</p>
<h2 id="step6">実行</h2>
<p>このモジュールでは次のコードを使用して、ビルドしたイメージに基づいてコンテナを実行します。</p>
<pre><code>docker run -p 4000:80 --name my-app node-app:0.1&#x000A;</code></pre>
<p>（コマンド出力）</p>
<pre><code class="language-bash prettyprint">Server running at http://0.0.0.0:80/&#x000A;</code></pre>
<p><code>--name </code>フラグを使用すると、必要に応じてコンテナに名前を付けることができます。<code>-p</code> で、ホストのポート 4000 をコンテナのポート 80 に割り当てることを Docker に指示します。これで <code>http://localhost:4000</code> のサーバーにアクセスできるようになります。ポートのマッピングが適用されていないと、localhost のコンテナにアクセスできません。</p>
<p>別のターミナルを開き（Cloud Shell で [<code>+</code>] アイコンをクリック）、サーバーをテストします。</p>
<pre><code>curl http://localhost:4000&#x000A;</code></pre>
<p>（コマンド出力）</p>
<pre><code class="language-bash prettyprint">Hello World&#x000A;</code></pre>
<p>コンテナは、最初のターミナルが稼働している限り実行されます。コンテナをバックグラウンドで実行する（ターミナルのセッションに関連付けない）場合は、<code>-d</code> フラグを指定する必要があります。</p>
<p>最初のターミナルを閉じてから、次のコマンドを実行してコンテナを停止、削除します。</p>
<pre><code>docker stop my-app &amp;&amp; docker rm my-app&#x000A;</code></pre>
<p>次のコマンドを実行してバックグラウンドでコンテナを起動します。</p>
<pre><code>docker run -p 4000:80 --name my-app -d node-app:0.1&#x000A;&#x000A;docker ps&#x000A;</code></pre>
<p>（コマンド出力）</p>
<pre><code class="language-bash prettyprint">CONTAINER ID   IMAGE          COMMAND        CREATED         ...  NAMES&#x000A;xxxxxxxxxxxx   node-app:0.1   "node app.js"  16 seconds ago  ...  my-app&#x000A;</code></pre>
<p><code>docker ps</code> の出力で、コンテナが実行されていることがわかります。ログを確認する場合は、<code>docker logs [コンテナ ID]</code> を実行します。</p>
<aside class="special"><p><strong>ヒント: </strong>先頭の数文字でコンテナを一意に識別できれば、コンテナ ID 全体を記述する必要はありません。たとえば、コンテナ ID が「<code>17bcaca6f...</code>」の場合は、「<code>docker logs 17b</code>」を実行します。</p>
</aside>
<pre><code>docker logs [コンテナ ID]&#x000A;</code></pre>
<p>（コマンド出力）</p>
<pre><code class="language-bash prettyprint">Server running at http://0.0.0.0:80/&#x000A;</code></pre>
<p>アプリケーションを修正してみましょう。Cloud Shell で、このラボで作成したテスト ディレクトリを開きます。</p>
<pre><code>cd test&#x000A;</code></pre>
<p>任意のテキスト エディタ（nano や vim など）で <code>app.js</code> を編集し、「Hello World」を別の文字列に置き換えます。</p>
<pre><code class="language-bash prettyprint">....&#x000A;const server = http.createServer((req, res) =&gt; {&#x000A;    res.statusCode = 200;&#x000A;      res.setHeader('Content-Type', 'text/plain');&#x000A;        res.end('Welcome to Cloud\n');&#x000A;});&#x000A;....&#x000A;</code></pre>
<p>新しいイメージをビルドして、タグ <code>0.2</code> を指定します。</p>
<pre><code>docker build -t node-app:0.2 .&#x000A;</code></pre>
<p>（コマンド出力）</p>
<pre><code class="language-bash prettyprint">Step 1/5 : FROM node:6&#x000A; ---&gt; 67ed1f028e71&#x000A;Step 2/5 : WORKDIR /app&#x000A; ---&gt; Using cache&#x000A; ---&gt; a39c2d73c807&#x000A;Step 3/5 : ADD . /app&#x000A; ---&gt; a7087887091f&#x000A;Removing intermediate container 99bc0526ebb0&#x000A;Step 4/5 : EXPOSE 80&#x000A; ---&gt; Running in 7882a1e84596&#x000A; ---&gt; 80f5220880d9&#x000A;Removing intermediate container 7882a1e84596&#x000A;Step 5/5 : CMD node app.js&#x000A; ---&gt; Running in f2646b475210&#x000A; ---&gt; 5c3edbac6421&#x000A;Removing intermediate container f2646b475210&#x000A;Successfully built 5c3edbac6421&#x000A;Successfully tagged node-app:0.2&#x000A;</code></pre>
<p>ステップ 2 では、既存のキャッシュ レイヤを使用しています。<code>app.js</code> に変更を加えているため、ステップ 3 以降ではレイヤが修正されています。</p>
<p>新しいイメージ バージョンで別のコンテナを実行します。80 の代わりにホストのポート 8080 を割り当てる方法に注目してください。ホストポート 4000 はすでに使われているため、使用できません。</p>
<pre><code>docker run -p 8080:80 --name my-app-2 -d node-app:0.2&#x000A;docker ps&#x000A;</code></pre>
<p>（コマンド出力）</p>
<pre><code class="language-bash prettyprint">CONTAINER ID     IMAGE             COMMAND            CREATED&#x000A;xxxxxxxxxxxx     node-app:0.2      "node app.js"      53 seconds ago      ...&#x000A;xxxxxxxxxxxx     node-app:0.1      "node app.js"      About an hour ago   ...&#x000A;</code></pre>
<p>コンテナをテストします。</p>
<pre><code>curl http://localhost:8080&#x000A;</code></pre>
<p>（コマンド出力）</p>
<pre><code class="language-bash prettyprint">Welcome to Cloud&#x000A;</code></pre>
<p>それでは、自分で作った最初のコンテナをテストしてみましょう。</p>
<pre><code>curl http://localhost:4000&#x000A;</code></pre>
<p>（コマンド出力）</p>
<pre><code class="language-bash prettyprint">Hello World&#x000A;</code></pre>
<h2 id="step7">デバッグ</h2>
<p>コンテナのビルドと実行について理解できたところで、デバッグの演習に進みましょう。</p>
<p><code>docker logs [コンテナ ID]</code> を使用して、コンテナのログを確認できます。コンテナの実行中にログの出力を監視する場合は、<code>-f</code> オプションを使用します。</p>
<pre><code>docker logs -f [コンテナ ID]&#x000A;</code></pre>
<p>（コマンド出力）</p>
<pre><code class="language-bash prettyprint">Server running at http://0.0.0.0:80/&#x000A;</code></pre>
<p>実行中のコンテナ内で対話型 Bash セッションを開始する必要がある場合は、docker exec を使用してください。別のターミナルを開き（Cloud Shell で [+] アイコンをクリック）、次のコマンドを入力します。</p>
<pre><code>docker exec -it [コンテナ ID] bash&#x000A;</code></pre>
<p><code>-it</code> フラグを使用することで、pseudo-tty を割り当てて stdin を開いたままにし、コンテナとやり取りすることができます。bash は <code>Dockerfile</code> で指定された <code>WORKDIR</code> ディレクトリ（/app）で実行されることに注意してください。これ以降は、デバッグするコンテナ内に対話型のシェル セッションがあります。</p>
<p>（コマンド出力）</p>
<pre><code class="language-bash prettyprint">root@xxxxxxxxxxxx:/app#&#x000A;</code></pre>
<p>ディレクトリを確認します。</p>
<pre><code>ls&#x000A;</code></pre>
<p>（コマンド出力）</p>
<pre><code class="language-bash prettyprint">Dockerfile  app.js&#x000A;root@xxxxxxxxxxxx:/app#&#x000A;</code></pre>
<p>Bash セッションを終了します。新しいターミナルで、次のように入力します。</p>
<pre><code>exit&#x000A;</code></pre>
<p>Docker inspect を使用して、Docker 内でコンテナのメタデータを調べることができます。</p>
<pre><code>docker inspect [コンテナ ID]&#x000A;</code></pre>
<p>（コマンド出力）</p>
<pre><code>[&#x000A;    {&#x000A;        "Id": "xxxxxxxxxxxx....",&#x000A;        "Created": "2017-08-07T22:57:49.261726726Z",&#x000A;        "Path": "node",&#x000A;        "Args": [&#x000A;            "app.js"&#x000A;        ],&#x000A;...&#x000A;</code></pre>
<p>返された JSON の特定のフィールドを調査するには、<code>--format</code> を使用します。次に例を示します。</p>
<pre><code>docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' [コンテナ ID]&#x000A;</code></pre>
<p>（出力例）</p>
<pre><code>192.168.9.3&#x000A;</code></pre>
<p>以下のリソースでデバッグの詳細を必ずご確認ください。</p>
<ul>
<li>Docker inspect <a href="https://docs.docker.com/engine/reference/commandline/inspect/#examples" target="_blank">リファレンス</a>
</li>
<li>Docker exec <a href="https://docs.docker.com/engine/reference/commandline/exec/" target="_blank">リファレンス</a>
</li>
</ul>
<h2 id="step8">公開</h2>
<p>次は、イメージを <a href="https://cloud.google.com/container-registry/">Google Container Registry</a>（gcr）にプッシュします。その後、すべてのコンテナとイメージを削除して新規の環境をシミュレートしてから、コンテナを pull して実行します。これによって、Docker コンテナの移植性がわかります。</p>
<p>gcr がホストする非公開レジストリにイメージを push する場合、レジストリ名を使ってイメージにタグを付ける必要があります。形式は <code>[ホスト名]/[プロジェクト ID]/[イメージ]:[タグ]</code> です。</p>
<p>gcr の場合</p>
<ul>
<li>
<code>[ホスト名]</code>= gcr.io</li>
<li>
<code>[プロジェクト ID]</code>= プロジェクトの ID</li>
<li>
<code>[イメージ]</code>= イメージ名</li>
<li>
<code>[タグ]</code>= 任意の文字列タグ（指定しない場合はデフォルトで「latest」に設定されます）</li>
</ul>
<p>プロジェクト ID は次のコマンドを実行して確認できます。</p>
<pre><code>gcloud config list project&#x000A;</code></pre>
<p>（コマンド出力）</p>
<pre><code class="language-bash prettyprint">[core]&#x000A;project = [プロジェクト ID]&#x000A;&#x000A;Your active configuration is: [default]&#x000A;</code></pre>
<p><code>node-app:0.2</code> にタグを付けます。<code>[プロジェクト ID]</code> は実際の構成に置き換えてください。</p>
<pre><code>docker tag node-app:0.2 gcr.io/[プロジェクト ID]/node-app:0.2&#x000A;</code></pre>
<pre><code>docker images&#x000A;</code></pre>
<p>（コマンド出力）</p>
<pre><code class="language-bash prettyprint">REPOSITORY                      TAG         IMAGE ID          CREATED&#x000A;node-app                        0.2         76b3beef845e      22 hours ago&#x000A;gcr.io/[プロジェクト ID]/node-app    0.2         76b3beef845e      22 hours ago&#x000A;node-app                        0.1         f166cd2a9f10      26 hours ago&#x000A;node                            6           5a767079e3df      7 days ago&#x000A;hello-world                     latest      1815c82652c0      7 weeks ago&#x000A;</code></pre>
<p>このイメージを gcr に push します。<code>[プロジェクト ID]</code> を忘れずに置き換えてください。</p>
<pre><code>docker push gcr.io/[プロジェクト ID]/node-app:0.2&#x000A;</code></pre>
<p>コマンド出力（実際の出力とは異なる場合があります）</p>
<pre><code class="language-bash prettyprint">The push refers to a repository [gcr.io/[プロジェクト ID]/node-app]&#x000A;057029400a4a: Pushed&#x000A;342f14cb7e2b: Pushed&#x000A;903087566d45: Pushed&#x000A;99dac0782a63: Pushed&#x000A;e6695624484e: Pushed&#x000A;da59b99bbd3b: Pushed&#x000A;5616a6292c16: Pushed&#x000A;f3ed6cb59ab0: Pushed&#x000A;654f45ecb7e3: Pushed&#x000A;2c40c66f7667: Pushed&#x000A;0.2: digest: sha256:25b8ebd7820515609517ec38dbca9086e1abef3750c0d2aff7f341407c743c46 size: 2419&#x000A;</code></pre>
<p>ウェブブラウザのイメージ レジストリにアクセスし、イメージが gcr に存在していることを確認します。コンソールで [<strong>ツール</strong>] &gt; [<strong>Container Registry</strong>] に移動するか、<code>http://gcr.io/[プロジェクト ID]/node-app</code> にアクセスしてください。以下のようなページが表示されます。</p>
<p><img alt="8afb07930833e781.png" src="https://cdn.qwiklabs.com/08T98M2PxiDZKimWDIIeUHF4ds7Ux2JH5KEnk4GobII%3D"></p>
<p>このイメージをテストするために、新しい VM を起動して ssh で接続し、gcloud をインストールしてみましょう。わかりやすいように、すべてのコンテナとイメージを削除して新規の環境をシミュレートします。</p>
<p>すべてのコンテナを停止して削除します。</p>
<pre><code>docker stop $(docker ps -q)&#x000A;docker rm $(docker ps -aq)&#x000A;</code></pre>
<p>node イメージを削除する前に、（<code>node:6</code> の）下位のイメージを削除する必要があります。<code>[プロジェクト ID]</code> は置き換えてください。</p>
<pre><code>docker rmi node-app:0.2 gcr.io/[プロジェクト ID]/node-app node-app:0.1&#x000A;docker rmi node:6&#x000A;docker rmi $(docker images -aq) # remove remaining images&#x000A;docker images&#x000A;</code></pre>
<p>（コマンド出力）</p>
<pre><code class="language-bash prettyprint">REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE&#x000A;</code></pre>
<p>この時点で新規の疑似環境が用意できたことになります。イメージを pull して実行します。<code>[プロジェクト ID]</code> を忘れずに置き換えてください。</p>
<pre><code>docker pull gcr.io/[プロジェクト ID]/node-app:0.2&#x000A;docker run -p 4000:80 -d gcr.io/[プロジェクト ID]/node-app:0.2&#x000A;curl http://localhost:4000&#x000A;</code></pre>
<p>（コマンド出力）</p>
<pre><code class="language-bash prettyprint">Welcome to Cloud&#x000A;</code></pre>

<p>これで、コンテナの移植性を実際に確認できました。ホスト（オンプレミスまたは VM）に Docker がインストールされていれば、一般公開レジストリまたは非公開レジストリからイメージを pull し、そのイメージに基づいてコンテナを実行できます。Docker 以外には、ホストにインストールする必要のあるアプリケーションの依存関係はありません。</p>
<h2 id="step9">まとめ</h2>
<p>Docker の概要については以上です。演習の内容を振り返ってみましょう。</p>
<ul>
<li>Docker Hub の公開イメージに基づいてコンテナを実行しました。</li>
<li>独自のコンテナ イメージをビルドし、Google Container Registry に push しました。</li>
<li>実行中のコンテナをデバッグする方法を学びました。</li>
<li>Google Container Registry から pull したイメージに基づいてコンテナを実行しました。</li>
</ul>

# Hello Node Kubernetes
<h2 id="step2">概要</h2>
<p>このハンズオンラボでは、これまで作成してきたコードを使って <a href="http://kubernetes.io">Kubernetes</a> で実行される（<a href="https://cloud.google.com/container-engine/">Kubernetes Engine</a> で動作する）複製アプリケーションを作成します。シンプルな Hello World node.js アプリケーションです。</p>
<p>次の図には、このラボで学習するさまざまな内容が示されており、各部がどう関連しているかの理解に役立ちます。図を参照しながら進めれば、終了する頃には内容をすべて理解できるようになります（現時点で理解できなくても問題ありません）。</p>
<p><img alt="ba830277f2d92e04.png" src="https://cdn.qwiklabs.com/ZB%2FLTu%2FfOBuu7svaY0%2Fier%2FSvbJdfF3Lrb%2F5woXeecI%3D"></p>
<p>Kubernetes はオープンソースのプロジェクト（<a href="http://kubernetes.io/">kubernetes.io</a> から入手可能）で、ノートパソコンから可用性の高いマルチノード クラスタ、パブリック クラウドからオンプレミスのデプロイ、仮想マシンからベアメタルまで、さまざまな環境で扱うことができます。</p>
<p>このラボでは、Kubernetes Engine（Compute Engine で稼働する Kubernetes の Google ホスト バージョン）などのマネージド環境を使用して、基盤となるインフラストラクチャの構築ではなく Kubernetes の体験を重視しています。</p>
<h4>演習内容</h4>
<ul>
<li>
<p>Node.js サーバーを作成する</p>
</li>
<li>
<p>Docker コンテナ イメージを作成する</p>
</li>
<li>
<p>コンテナ クラスタを作成する</p>
</li>
<li>
<p>Kubernetes ポッドを作成する</p>
</li>
<li>
<p>サービスをスケールアップする</p>
</li>
</ul>
<h3>要件</h3>
<ul>
<li>Linux の標準的なテキスト エディタ（<code>vim</code>、<code>emacs</code>、<code>nano</code> など）の使用経験があると役に立ちます。</li>
</ul>
<p>本質的なコンセプトをよりよく理解できるよう、コマンドは手動で入力することをおすすめしますが、多くのラボには、必要なコマンドが含まれたコードブロックがあります。ラボでは、コードブロックのコマンドをコピーして、該当する場所に貼り付けることもできます。</p>

<h2 id="step4">Node.js アプリケーションを作成する</h2>
<p>Cloud Shell を使用して、Kubernetes Engine にデプロイするシンプルな Node.js サーバーを作成します。</p>
<pre><code>vi server.js&#x000A;</code></pre>
<p>エディタを開始します。</p>
<pre><code>i&#x000A;</code></pre>
<p>次の内容をファイルに加えます。</p>

```js
const http = require('http')
http.createServer((req, res) => {
    res.writeHead(200)
    res.end('Hello World')
}).liten(8080)
```

<aside>ここでは <code>vi</code> を使用しますが、<code>nano</code> と <code>emacs</code> も Cloud Shell で使用できます。また、<a href="https://cloud.google.com/shell/docs/features#web_editor">こちらで説明</a>されているように、Cloud Shell のウェブエディタ機能を使うこともできます。

</aside>
<p><code>server.js</code> ファイルを保存します。<strong>Esc キー</strong>を押してから、次のように入力します。</p>
<pre><code>:wq&#x000A;</code></pre>
<p>Cloud Shell には <code>node</code> 実行可能ファイルがインストールされており、このコマンドでノードサーバーを起動することができます（このコマンドによる出力はありません）。</p>
<pre><code>node server.js&#x000A;</code></pre>
<p>Cloud Shell に組み込まれたウェブ <a href="https://cloud.google.com/cloud-shell/docs/features#web_preview">プレビュー機能</a>を使用して、新しいブラウザタブを開き、起動したインスタンスに対するリクエストをポート 8080 でプロキシサーバーに送信します。</p>
<p><img alt="bde9fe42e27656fb.png" src="https://cdn.qwiklabs.com/a6YnJv8GlGae4rnJIbjA27J8c7YApa%2B6noPFkkKxZjk%3D"></p>
<p>新しいブラウザタブが開き、結果が表示されます。</p>
<p><img alt="24aab6bb51533e91.png" src="https://cdn.qwiklabs.com/Si2q8aZMVmxlt1eZELd271lVh9W24mbVd5YnZ8iS37g%3D"></p>
<p>続行する前に、Cloud Shell に戻り、<strong>Ctrl+c キー</strong>を押して実行中のノードサーバーを停止します。</p>
<p>次に、このアプリケーションを Docker コンテナでパッケージ化します。</p>
<h2 id="step5">Docker コンテナ イメージを作成する</h2>
<p>構築するイメージについて記述した <code>Dockerfile</code> を作成します。Docker コンテナ イメージは他の既存のイメージから拡張することができるため、今回も既存の Node イメージから拡張します。</p>
<pre><code>vi Dockerfile&#x000A;</code></pre>
<p>エディタを起動します。</p>
<pre><code>i&#x000A;</code></pre>
<p>次の内容を入力します。</p>
<pre><code>FROM node:6.9.2&#x000A;EXPOSE 8080&#x000A;COPY server.js .&#x000A;CMD node server.js&#x000A;</code></pre>
<p>この Docker イメージの「レシピ」は次のようになります。</p>
<ul>
<li>Docker Hub にある <code>node</code> イメージから起動する</li>
<li>ポート <code>8080</code> を開く</li>
<li>
<code>server.js</code> ファイルを対象のイメージにコピーする</li>
<li>以前に行ったように、手動でノードサーバーを起動する</li>
</ul>
<p><strong>Esc キー</strong>を押してから、次のように入力し、この <code>Dockerfile</code> を保存します。</p>
<pre><code>:wq&#x000A;</code></pre>
<p>次のようにイメージを作成します。<code>PROJECT_ID</code> は、コンソールやラボの [<strong>接続の詳細</strong>] セクションに表示される GCP のプロジェクト ID に置き換えます。</p>
<pre><code>docker build -t gcr.io/PROJECT_ID/hello-node:v1 .&#x000A;</code></pre>

> `-t` でタグ付け（`名前:tag`）ができる。ここでは `gcr.io/PROJECT_ID/hello-node:v1` とタグつけしている。gcr.io〜という長ったらしい命名は Google Container Registory を使う際のルール。`.` はカレントディレクトリを指す。個人的には `docker build . -t gcr.io/~~` という順番のほうがわかりやすいと思う。

<p>すべてをダウンロードして展開するには時間がかかりますが、イメージ作成の進行状況は進捗バーで確認できます。</p>
<p>完了したら、次のコマンドを使用して、ローカルでイメージをテストします。新しく作成したコンテナ イメージから、Docker コンテナを daemon としてポート 8080 で実行します（<code>PROJECT_ID</code> はコンソール、またはラボの [<strong>接続の詳細</strong>] セクションに表示される GCP のプロジェクト ID に置き換えます）。</p>
<pre><code>docker run -d -p 8080:8080 gcr.io/PROJECT_ID/hello-node:v1&#x000A;</code></pre>
<p>次のように表示されます。</p>
<pre><code class="language-bash prettyprint">325301e6b2bffd1d0049c621866831316d653c0b25a496d04ce0ec6854cb7998&#x000A;</code></pre>
<p>Cloud Shell のウェブ プレビュー機能を使用すれば、この結果を確認することができます。</p>
<p><img alt="bde9fe42e27656fb.png" src="https://cdn.qwiklabs.com/a6YnJv8GlGae4rnJIbjA27J8c7YApa%2B6noPFkkKxZjk%3D"></p>
<p>または Cloud Shell で <code>curl</code> を使用します。</p>
<pre><code>curl http://localhost:8080&#x000A;</code></pre>
<p>次のように表示されます。</p>
<pre><code class="language-bash prettyprint">Hello World！&#x000A;</code></pre>
<aside>
<strong>注</strong>:
<code>docker run</code> コマンドについて詳しくは、<a href="https://docs.docker.com/engine/reference/run/">こちら</a>をご確認ください。
</aside>
<p>次に、実行中のコンテナを停止します。</p>
<p>次のコマンドを実行して、Docker コンテナ ID を確認します。</p>
<pre><code>docker ps&#x000A;</code></pre>
<p>次のように表示されます。</p>
<pre><code>CONTAINER ID        IMAGE                              COMMAND&#x000A;2c66d0efcbd4        gcr.io/PROJECT_ID/hello-node:v1    "/bin/sh -c 'node&#x000A;</code></pre>
<p>次のコマンドの CONTAINER ID を前のステップで取得したコンテナ ID に置き換えて実行し、コンテナを停止します。</p>
<pre><code>docker stop [CONTAINER ID]&#x000A;</code></pre>
<p>コンソールには次のように表示されます（コンテナ ID）。</p>
<pre><code class="language-bash prettyprint">2c66d0efcbd4&#x000A;</code></pre>
<p>イメージが意図したとおりに機能するようになりましたので、これを Docker イメージ用の非公開リポジトリである <a href="https://cloud.google.com/tools/container-registry/">Google Container Registry</a> に push します。このリポジトリには Google Cloud プロジェクトからアクセスできます。</p>
<p><code>PROJECT_ID</code> をコンソール、またはラボの [<strong>接続の詳細</strong>] セクションに表示される GCP のプロジェクト ID に置き換えて、次のコマンドを実行します。</p>
<pre><code>gcloud auth configure-docker&#x000A;</code></pre>
<pre><code>docker push gcr.io/PROJECT_ID/hello-node:v1&#x000A;</code></pre>
<p>初回の push は、完了するまで数分かかることがあります。進捗状況は進捗バーで確認できます。</p>
<pre><code class="language-bash prettyprint">The push refers to a repository [gcr.io/qwiklabs-gcp-6h281a111f098/hello-node]&#x000A;ba6ca48af64e: Pushed&#x000A;381c97ba7dc3: Pushed&#x000A;604c78617f34: Pushed&#x000A;fa18e5ffd316: Pushed&#x000A;0a5e2b2ddeaa: Pushed&#x000A;53c779688d06: Pushed&#x000A;60a0858edcd5: Pushed&#x000A;b6ca02dfe5e6: Pushed&#x000A;v1: digest: sha256:8a9349a355c8e06a48a1e8906652b9259bba6d594097f115060acca8e3e941a2 size: 2002&#x000A;</code></pre>
<p>コンテナ イメージがコンソールに一覧で表示されます。[<strong>ナビゲーション メニュー</strong>] &gt; [<strong>Container Registry</strong>] を選択します。</p>
<p><img alt="Kubernetes_Container_reg.png" src="https://cdn.qwiklabs.com/01saPtda19enDOnSFx3tQ6w0N5FuM4uER8GDsC8n5cA%3D"></p>
<p>これで、プロジェクト全体で使用できる Docker イメージができました。このイメージには Kubernetes でアクセスし、オーケストレートすることができます。</p>
<p><img alt="container_reg.png" src="https://cdn.qwiklabs.com/dQgWvGqTs5%2BVCSfmbpL2lpTlQ7dd19FwSIKTEBS3poA%3D"></p>
<aside>
<strong>注:</strong> レジストリ（<code>gcr.io</code>）用の汎用ドメインを使用しています。使用するゾーンとバケットは限定することができます。詳しくは、<a href="https://cloud.google.com/container-registry/docs/#pushing_to_the_registry">こちらのドキュメント</a>でご確認ください。

</aside>
<h2 id="step6">クラスタを作成する</h2>
<p>ここまでの作業で Container Engine クラスタを作成する準備が整っています。クラスタは Google がホストしている Kubernetes マスター API サーバーと一連のワーカーノードで構成されます。ワーカーノードは Compute Engine 仮想マシンです。</p>
<p><code>gcloud</code> を使用してプロジェクトを設定していることを確認してください（<code>PROJECT_ID</code> をコンソール、またはラボの [<strong>接続の詳細</strong>] セクションに表示される GCP のプロジェクト ID に置き換えます）。</p>
<pre><code>gcloud config set project PROJECT_ID&#x000A;</code></pre>
<p>2 つの <a href="https://cloud.google.com/compute/docs/machine-types">n1-standard-1</a> ノードがあるクラスタを作成します（完了するまでに数分かかります）。</p>
<pre><code>gcloud container clusters create hello-world \&#x000A;                --num-nodes 2 \&#x000A;                --machine-type n1-standard-1 \&#x000A;                --zone us-central1-a&#x000A;</code></pre>
<p>クラスターの構築時に表示される警告は無視しても問題ありません。</p>
<p>コンソールに次のように表示されます。</p>
<pre><code class="language-bash prettyprint">Creating cluster hello-world...done.&#x000A;Created [https://container.googleapis.com/v1/projects/PROJECT_ID/zones/us-central1-a/clusters/hello-world].&#x000A;kubeconfig entry generated for hello-world.&#x000A;NAME         ZONE           MASTER_VERSION  MASTER_IP       MACHINE_TYPE   STATUS&#x000A;hello-world  us-central1-a  1.5.7           146.148.46.124  n1-standard-1  RUNNING&#x000A;</code></pre>
<p><strong>注:</strong> このクラスタは、イメージが表示されている上述のコンソールから作成することもできます（[<strong>Kubernetes Engine</strong>] &gt; [<strong>Kubernetes クラスタ</strong>] &gt; [<strong>クラスタを作成</strong>]）。</p>
<aside> <strong>注</strong>:クラスタは、コンテナ レジストリで使用されるストレージ バケットと同じゾーンに作成することをおすすめします（前のステップを参照してください）。
</aside>
<p>[<strong>ナビゲーション メニュー</strong>] &gt; [<strong>Kubernetes Engine</strong>] を選択すると、Kubernetes Engine を搭載した、完全版の Kubernetes クラスタができていることを確認できます。</p>
<p><img alt="kubernetes_cluster.png" src="https://cdn.qwiklabs.com/wMR0qhPxxgayDTSjDq1oXmoXEy4OeuQ%2FUtc6CZnvFT0%3D"></p>
<p>それでは、コンテナ化されたアプリケーションを Kubernetes クラスタにデプロイしてみましょう。ここからは <code>kubectl</code> コマンドライン（Cloud Shell 環境で設定済み）を使用します。</p>

<h2 id="step7">ポッドを作成する</h2>
<p>Kubernetes <strong>ポッド</strong>は管理とネットワーキングを目的に結合されたコンテナのグループで、1 つ以上のコンテナを含めることができます。ここでは、プライベート コンテナ レジストリに保存されている Node.js イメージで構築した 1 つのコンテナを使用します。このコンテナは、コンテンツの提供にポート 8080 を使用します。</p>
<p><code>kubectl run</code> コマンドでポッドを作成します（<code>PROJECT_ID</code> をコンソール、またはラボの [<strong>接続の詳細</strong>] セクションに表示される GCP のプロジェクト ID に置き換えます）。</p>
<pre><code>kubectl create deployment hello-node \&#x000A;    --image=gcr.io/PROJECT_ID/hello-node:v1&#x000A;</code></pre>
<p>次のように表示されます。</p>
<pre><code class="language-bash prettyprint">deployment.apps/hello-node created&#x000A;</code></pre>
<p>これで<strong>デプロイメント</strong> オブジェクトが作成されました。デプロイメント オブジェクトは、ポッドを作成してスケーリングするためのおすすめの方法です。ここでは、<code>hello-node:v1</code> イメージを実行する 1 つのポッドレプリカを新しいデプロイメントで管理します。</p>
<p>デプロイメントを表示するには、次のコマンドを実行します。</p>
<pre><code>kubectl get deployments&#x000A;</code></pre>
<p>次のように表示されます。</p>
<pre><code class="language-bash prettyprint">NAME         READY   UP-TO-DATE   AVAILABLE   AGE&#x000A;hello-node   1/1     1            1           1m36s&#x000A;</code></pre>
<p>デプロイメントによって作成されたポッドを表示するには、次のコマンドを実行します。</p>
<pre><code>kubectl get pods&#x000A;</code></pre>
<p>次のように表示されます。</p>
<pre><code class="language-bash prettyprint">NAME                         READY     STATUS    RESTARTS   AGE&#x000A;hello-node-714049816-ztzrb   1/1       Running   0          6m&#x000A;</code></pre>
<p>ここで、便利な <code>kubectl</code> コマンドをいくつかお伝えします。いずれもクラスタの状態を変えるものではありません。詳しくは、<a href="https://cloud.google.com/container-engine/docs/kubectl/">こちら</a>をご確認ください。</p>
<pre><code>kubectl cluster-info&#x000A;</code></pre>
<pre><code>kubectl config view&#x000A;</code></pre>
<p>トラブルシューティング用:</p>
<pre><code>kubectl get events&#x000A;</code></pre>
<pre><code>kubectl logs &lt;pod-name&gt;&#x000A;</code></pre>
<p>ここで、ポッドに外部からアクセスできるようにする必要があります。</p>

<h2 id="step8">外部トラフィックを許可する</h2>
<p>デフォルトでは、ポッドにはクラスタ内の内部 IP からしかアクセスできません。<code>hello-node</code> コンテナを Kubernetes 仮想ネットワークの外部からアクセスできるようにするには、ポッドを Kubernetes <strong>サービス</strong>として公開する必要があります。</p>
<p>Cloud Shell から、<code>kubectl expose</code> コマンドを <code>--type="LoadBalancer"</code> フラグと組み合わせて使用すると、ポッドをインターネットで公開できます。外部からアクセスできる IP を作成するには、このフラグが必要です。</p>

> **以下の `hello-node` は deployment の名前。**

<pre><code>kubectl expose deployment hello-node --type="LoadBalancer" --port=8080&#x000A;</code></pre>
<p>次のように表示されます。</p>
<pre><code class="language-bash prettyprint">service/hello-node exposed&#x000A;</code></pre>
<p>このコマンドで使用しているフラグは、基盤となるインフラストラクチャで提供されるロードバランサ（この場合は <a href="https://cloud.google.com/compute/docs/load-balancing/">Compute Engine のロードバランサ</a>）を使うよう指定しています。ポッドを直接公開するのではなく、デプロイメントを公開していることに注意してください。これによって、デプロイメントで管理されるすべてのポッドにまたがってトラフィックの負荷を分散するサービスが作成されます（ここでは 1 つのポッドだけですが、後でレプリカを追加します）。</p>
<p>Kubernetes マスターによってロードバランサ、それに関連する Compute Engine 転送ルール、ターゲット プール、ファイアウォール ルールが作成され、Google Cloud Platform の外部からこのサービスに完全にアクセスできるようになります。</p>
<p>対象サービスの公開 IP アドレスを確認するには、<code>kubectl</code> をリクエストしてすべてのクラスタ サービスを一覧で表示します。</p>
<pre><code>kubectl get services&#x000A;</code></pre>
<p>次のように表示されます。</p>
<pre><code class="language-bash prettyprint">NAME         CLUSTER-IP     EXTERNAL-IP      PORT(S)    AGE&#x000A;hello-node   10.3.250.149   104.154.90.147   8080/TCP   1m&#x000A;kubernetes   10.3.240.1     &lt;none&gt;           443/TCP    5m&#x000A;</code></pre>
<p>hello-node サービスに対して 2 つの IP アドレスが表示されます。いずれもポート <code>8080</code> を使用しています。CLUSTER-IP は内部 IP で、Cloud Virtual Network 内でのみ表示されます。EXTERNAL-IP は負荷分散を行う外部 IP です。</p>
<aside class="special"><p><strong>注</strong>:<code>EXTERNAL-IP</code> が表示され、使用できるようになるまでには数分かかることがあります。<code>EXTERNAL-IP</code> が見つからない場合は、数分待ってからもう一度試してみてください。</p>
</aside>
<p>ブラウザでアドレス <code>http://&lt;EXTERNAL_IP&gt;:8080</code> と入力すれば、対象のサービスにアクセスできるようになっています。</p>
<p><img alt="67cfa8c674f8c708.png" src="https://cdn.qwiklabs.com/C36gw9A9wyqza3PveBYBCsMggISATH80A18fqFBck7U%3D"></p>
<p>コンテナと Kubernetes を利用することで、ワークロードを負担するホストを指定がなかったり、サービスのモニタリングと再起動ができるなどのメリットが得られるようになりました。それでは、新しい Kubernetes インフラストラクチャを利用することで得られるその他のメリットについて見ていきましょう。</p>

<h2 id="step9">サービスをスケールアップする</h2>
<p>Kubernetes の優れた特長の 1 つは、アプリケーションのスケールアップが容易なことです。急にアプリケーションの容量を増やす必要が生じたとしても、レプリケーション コントローラに、ポッドの新しいレプリカを管理するよう指示することができます。</p>
<pre><code>kubectl scale deployment hello-node --replicas=4&#x000A;</code></pre>
<p>次のように表示されます。</p>
<pre><code class="language-bash prettyprint">deployment.extensions/hello-node scaled&#x000A;</code></pre>
<p>更新されたデプロイメントの詳細をリクエストできます。</p>
<pre><code>kubectl get deployment&#x000A;</code></pre>
<p>次のように表示されます。</p>
<pre><code class="language-bash prettyprint">NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE&#x000A;hello-node   4         4         4            4           16m&#x000A;</code></pre>
<p>また、すべてのポッドを一覧で表示することもできます。</p>
<pre><code>kubectl get pods&#x000A;</code></pre>
<p>次のように表示されます。</p>
<pre><code class="language-bash prettyprint">NAME                         READY     STATUS    RESTARTS   AGE&#x000A;hello-node-714049816-g4azy   1/1       Running   0          1m&#x000A;hello-node-714049816-rk0u6   1/1       Running   0          1m&#x000A;hello-node-714049816-sh812   1/1       Running   0          1m&#x000A;hello-node-714049816-ztzrb   1/1       Running   0          16m&#x000A;</code></pre>
<p>ここでは<strong>宣言的アプローチ</strong>を使用しています。新しいインスタンスの起動や停止を行うのではなく、実行するインスタンスの数を宣言します。Kubernetes の突合せループによって、リクエストした内容と実際の状態が一致しているかどうかが照合され、必要に応じて調整が行われます。</p>
<p>次の図に、Kubernetes クラスタの状態をまとめます。</p>
<p><img alt="587f7f0a097aaa2.png" src="https://cdn.qwiklabs.com/xcK5q7mZsGBWS%2BPbytdmF0W%2BdsZxvNXdOIEPBXX13X4%3D"></p>

<h2 id="step10">サービスへのアップグレードをロールアウトする</h2>
<p>本番環境にデプロイしたアプリケーションは、いずれかの時点でバグ修正や追加機能の実装が必要になります。Kubernetes を使用すると、ユーザーに影響を及ぼすことなく、新しいバージョンを本番環境にデプロイすることができます。</p>
<p>まず、アプリケーションに何らかの変更を加えてみましょう。<code>server.js</code> を編集します:</p>
<pre><code>vi server.js&#x000A;</code></pre>
<pre><code>i&#x000A;</code></pre>
<p>レスポンス メッセージを更新します。</p>
<pre><code class="language-bash prettyprint">response.end("Hello Kubernetes World!");&#x000A;</code></pre>
<p><code>server.js</code> ファイルを保存します。<strong>Esc キー</strong>を押してから、次のように入力します。</p>
<pre><code>:wq&#x000A;</code></pre>
<p>バージョン番号を変えたタグ（この場合は <code>v2</code>）を含む新しいコンテナ イメージを構築し、レジストリに公開します。</p>
<p><code>PROJECT_ID</code> をコンソール、またはラボの [<strong>接続の詳細</strong>] セクションに表示されるラボのプロジェクト ID に置き換えて、次のコマンドを実行します。</p>
<pre><code>docker build -t gcr.io/PROJECT_ID/hello-node:v2 .&#x000A;</code></pre>
<pre><code>docker push gcr.io/PROJECT_ID/hello-node:v2&#x000A;</code></pre>
<aside><strong>注</strong>:キャッシュを活用できるため、この更新版イメージの構築と展開は迅速に行われます。
</aside>
<p>Kubernetes によって、対象のアプリケーションでレプリケーション コントローラがスムーズに更新されます。既存の <code>hello-node deployment</code> を編集し、イメージを <code>gcr.io/PROJECT_ID/hello-node:v1</code> から <code>gcr.io/PROJECT_ID/hello-node:v2</code> に変更して、実行しているコンテナのイメージラベルを更新します。</p>
<p>これは <code>kubectl edit</code> コマンドで実行します。テキスト エディタが開かれ、デプロイメントの yaml 構成全体が表示されます。ここでは yaml 構成全体を理解する必要はありません。構成の <code>spec.template.spec.containers.image</code> フィールドを更新すれば、新しいイメージでポッドを更新するよう指示することになるというポイントが重要です。</p>
<pre><code>kubectl edit deployment hello-node&#x000A;</code></pre>
<p><code>Spec</code> &gt; <code>containers</code> &gt; <code>image</code> を探し、バージョン番号をv2に変更します。</p>

```yaml
# 下のオブジェクトを編集してください。「#」が付いている行はコメントであり、無視されます。
# 空白のファイルを使用すると編集内容が破棄されます。ファイルの保存中にエラーが発生した場合は
# ファイルが再び開かれますが、エラーが存在する状態になります。
#
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: 2016-03-24T17:55:28Z
  generation: 3
  labels:
    run: hello-node
  name: hello-node
  namespace: default
  resourceVersion: "151017"
  selfLink: /apis/extensions/v1beta1/namespaces/default/deployments/hello-node
  uid: 981fe302-f1e9-11e5-9a78-42010af00005
spec:
  replicas: 4
  selector:
    matchLabels:
      run: hello-node
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: hello-node
    spec:
      containers:
      - image: gcr.io/PROJECT_ID/hello-node:v1 ## Update this line ##
        imagePullPolicy: IfNotPresent
        name: hello-node
        ports:
        - containerPort: 8080
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      securityContext: {}
      terminationGracePeriodSeconds: 30
```

<p>変更を行ったら、このファイルを保存して閉じます。<strong>Esc キー</strong>を押してから、次のように入力します。</p>
<pre><code>:wq&#x000A;</code></pre>
<p>次のように表示されます。</p>
<pre><code>deployment.extensions/hello-node edited&#x000A;</code></pre>
<p>次のコマンドを実行して、新しいイメージでデプロイメントを更新します。</p>
<pre><code>kubectl get deployments&#x000A;</code></pre>
<p>新しいイメージで新しいポッドが作成され、古いポッドは削除されます。</p>
<p>次のように表示されます。</p>
<pre><code>NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE&#x000A;hello-node   4         4         4            4           1h&#x000A;</code></pre>
<p>更新の実行中、ユーザーがサービスを中断する必要はありません。しばらくすると、アプリケーションの新しいバージョンにアクセスできるようになります。更新のロールアウトについてさらに詳しくは、<a href="https://cloud.google.com/container-engine/docs/rolling-updates">こちらのドキュメント</a>をご確認ください。</p>
<p>Kubernetes Engine クラスタを使うと、以上のようなデプロイ、スケーリング、更新の機能のおかげで、インフラストラクチャを気にせずアプリケーションに集中できるようになります。</p>
<h2 id="step11">Kubernetes グラフィカル ダッシュボード（オプション）</h2>
<p>最近の Kubernetes には、グラフィカル ウェブ ユーザー インターフェース（ダッシュボード）が実装されています。このダッシュボードはすぐに使うことができ、システムにアプローチしやすくやり取りしやすい手段として CLI の機能性が組み込まれています。</p>
<p>すぐに使うために、クラスタレベル許可を与えるように以下のコマンドを実行します。</p>
<pre><code>kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=$(gcloud config get-value account)&#x000A;</code></pre>
<p>適当許可は設定された後で、新ダッシュボードサービスを作成するように以下のコマンドを実行します。</p>
<pre><code>kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml&#x000A;</code></pre>
<p>出力例:</p>
<pre><code>secret "kubernetes-dashboard-certs" created&#x000A;serviceaccount "kubernetes-dashboard" created&#x000A;role.rbac.authorization.k8s.io "kubernetes-dashboard-minimal" created&#x000A;rolebinding.rbac.authorization.k8s.io "kubernetes-dashboard-minimal" created&#x000A;deployment.apps "kubernetes-dashboard" created&#x000A;service "kubernetes-dashboard" created&#x000A;</code></pre>
<p>ダッシュボードサービスの <code>yaml</code> 表現を編集するように以下のコマンドを実行します。</p>
<pre><code>kubectl -n kube-system edit service kubernetes-dashboard&#x000A;</code></pre>
<p>編集モードにに入るように <code>i</code> を押します。<code>type: ClusterIP</code> から <code>type: NodePort</code> に変更します。変更してから、ファイルを保存し、終了します。その後で、<strong>Esc</strong> を押します。</p>
<pre><code>:wq&#x000A;</code></pre>
<p>Kubernetes ダッシュボードにログインするには、トークンを使用して認証する必要があります。トークンは、サービス アカウントに割り当てられているもの たとえば、<code>namespace-controller</code> を使用してください。</p>
<p> 次のコマンドで、トークンの値を取得できます。</p>
<pre><code>kubectl -n kube-system describe $(kubectl -n kube-system \&#x000A;get secret -n kube-system -o name | grep namespace) | grep token:&#x000A;</code></pre>
<p>出力例:</p>
<pre><code class="language-bash prettyprint">token:      eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJuYW1lc3BhY2UtY29udHJvbGxlci10b2tlbi1kOTZyNCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJuYW1lc3BhY2UtY29udHJvbGxlciIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImU2ZmFkNGQ5LTJjNjYtMTFlOC05NDFiLTQyMDEwYTgwMDFlYiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTpuYW1lc3BhY2UtY29udHJvbGxlciJ9.AY3Fp-T_4wxTzvo4kiWi4zxojVTSr1Wy7BL_-HmIRlWTRAUmy_1RAJS19zn4BbSkxlV13Y9Bv3NoVcG01jKd4QoM172OXo2TqSU5v2B62i3-_CDZtf3CVgQIp9jiuxACcR5zg3w-4ewGfH4C3ospoKCuayyRaADLq0ThWLGaTQv9e7UjSfWAPir3XPXQut3mMRYrSiHcFNiEGeztSfF3cyhuvL2I5Lfh20yYuqW5j-w72BLnlqQGPuhJXJgH1_35XUCU8WtnkEK-qYX40ajDWJYa1s9_R-MWzF6Zwji2Gh5txOvxG3lZuIq9GSAOBp85617wB3eCGio6Nu3L9TwWXA&#x000A;</code></pre>
<p>Kubernetes ダッシュボードへのログインに使用するため、トークンをコピーします。</p>
<pre><code>kubectl proxy --port 8081&#x000A;</code></pre>
<p>次に、Cloud Shell のウェブ プレビュー機能を使用して、ポートを <strong>8081</strong> に変更します。</p>
<p><img alt="CloudShell_web_preview_changeport.png" src="https://cdn.qwiklabs.com/SBq6c7%2Fh9Y7hCaC5sg4icIk%2FLssTrTseMY10PdL6tpo%3D"></p>
<p>これで、リクエストが API エンドポイントに送られるようになります。ダッシュボードを開くには、<code>/?authuser=0</code> を削除して、以下のように URL を追加します。</p>
<pre><code>/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/overview?namespace=default&#x000A;</code></pre>
<p>URL は最終的に次のようになります。</p>
<pre><code>https://8081-dot-5177448-dot-devshell.appspot.com/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/overview?namespace=default&#x000A;</code></pre>
<p>ウェブ プレビューは次のようになります。</p>
<p><img alt="e28f04ef7f77f1e6.png" src="https://cdn.qwiklabs.com/TQXFLFhWIKiwNS5WfFzqbo7zYaukguShztoxNFwxzYc%3D"></p>
<p>[<strong>Token</strong>] ラジオボタンをオンにし、前のステップでコピーしたトークンを貼り付けます。[<strong>Sign In</strong>] をクリックします。</p>
<p>Kubernetes グラフィカル ダッシュボードを使用すれば、コンテナ化されたアプリケーションのデプロイや、クラスタのモニタリングと管理を行うことができます。ぜひご活用ください。</p>
<p><img alt="dashboard.png" src="https://cdn.qwiklabs.com/poLdd9NpGhinl5apCl1cmRaKLYQj5bb79wpFUqrLFsE%3D"></p>
<p>ウェブ コンソールでは、開発用のマシンやローカルのマシンからダッシュボードにアクセスできます。[<strong>ナビゲーション メニュー</strong>] &gt; [<strong>Kubernetes Engine</strong>] を選択し、モニタリングするクラスタの [<strong>接続</strong>] ボタンをクリックします。</p>
<p><img alt="kubernetes_cluster_connect.png" src="https://cdn.qwiklabs.com/wgf9r%2Bl04aq9GdhtmDqN3Q8RGaomIdLf6LU%2F6owm%2Bm0%3D"></p>
<p><img alt="d580f036b7c50a.png" src="https://cdn.qwiklabs.com/lHbjivR2ApkRJo1Szf9HePkfbiJ2Yog2ep29%2B8M8nLE%3D"></p>
<p>Kubernetes ダッシュボードについてさらに詳しくは、<a href="http://kubernetes.io/docs/user-guide/ui/">ダッシュボード ツアー</a>をご利用ください。</p>

# [Kubernetes を使った Cloud のオーケストレーション](https://github.com/thesugar/memo/blob/master/gcp/cloud_architecture.md#kubernetes-%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%9F-cloud-%E3%81%AE%E3%82%AA%E3%83%BC%E3%82%B1%E3%82%B9%E3%83%88%E3%83%AC%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3)

# [Kubernetes Engine: クイックスタート](https://github.com/thesugar/memo/blob/master/gcp/basic-of-GCP-handson.md#kubernetes-engine-%E3%82%AF%E3%82%A4%E3%83%83%E3%82%AF%E3%82%B9%E3%82%BF%E3%83%BC%E3%83%88)

# Kubernetes Engine によるデプロイの管理
<h2 id="step2">概要</h2>
<p>DevOps のプラクティスでは、「継続的デプロイ」、「Blue / Green デプロイ」、「カナリア デプロイ」といったアプリケーション デプロイのシナリオを管理するために、複数のデプロイが定期的に利用されます。こうした複数の異種混合デプロイが使用される一般的なシナリオに対応できるように、このラボではコンテナのスケーリングと管理の演習を行います。</p>
<h3><strong>演習内容</strong></h3>
<ul>
<li>kubectl ツールを使用する</li>
<li>デプロイの yaml ファイルを作成する</li>
<li>デプロイをリリース、更新、スケーリングする</li>
<li>デプロイとデプロイ スタイルを更新する</li>
</ul>
<h2 id="step3">デプロイの概要</h2>
<p>異種混合デプロイでは通常、特定の技術的ニーズや運用上のニーズに対応するため、2 つ以上の異なるインフラストラクチャ環境またはリージョンを接続します。異種混合デプロイは、デプロイの仕様に応じて「ハイブリッド」、「マルチクラウド」、「パブリック-プライベート」と呼ばれます。このラボでは、単一のクラウド環境、複数のパブリック クラウド環境（マルチクラウド）、またはオンプレミス環境とパブリック クラウド環境の組み合わせ（ハイブリッドまたはパブリック-プライベート）において、複数のリージョンにまたがる異種混合デプロイを扱います。</p>
<p>単一の環境またはリージョンに限定されたデプロイでは、ビジネス上や技術上のさまざまな課題が発生する可能性があります。</p>
<ul>
<li>
<strong>リソースの上限</strong>: 単一の環境、特にオンプレミス環境では、本番環境のニーズを満たすコンピューティング リソース、ネットワーキング リソース、ストレージ リソースを確保できない場合があります。</li>
<li>
<strong>地理的範囲の制限</strong>: 単一環境のデプロイでは、地理的に離れた場所にいるユーザーが互いに 1 つのデプロイにアクセスする必要があります。つまり、ユーザーのトラフィックは、中心となる場所に到達するまでに世界中を移動する可能性があります。</li>
<li>
<strong>限られた可用性</strong>: ウェブ規模のトラフィック パターンを処理するアプリケーションは、フォールト トレランスと復元力を維持する必要があります。</li>
<li>
<strong>ベンダー ロックイン</strong>: プラットフォームとインフラストラクチャがベンダーレベルで抽象化されるため、アプリケーションの移植が妨げられる場合があります。</li>
<li>
<strong>柔軟性の低いリソース</strong>: リソースが特定のコンピューティング、ストレージ、ネットワーキング サービスに限定される場合があります。</li>
</ul>
<p>異種混合デプロイは、これらの課題に対処するのに役立ちますが、プログラマティックで確定的なプロセスと手順を使用して設計する必要があります。一度限りまたはその場しのぎのデプロイ手順では、デプロイやプロセスが脆弱になり、障害に耐えられない可能性があります。また、その場しのぎのプロセスはデータの消失やトラフィックのドロップを招く場合があります。優れたデプロイ プロセスは再現可能でなければならず、プロビジョニング、構成、メンテナンスの管理には、実績のあるアプローチが使用される必要があります。</p>
<p>異種混合デプロイの一般的なシナリオは、マルチクラウド デプロイ、オンプレミス データの外部接続、継続的インテグレーション / 継続的デリバリー（CI / CD）プロセスの 3 つです。</p>
<p>次の演習では、Kubernetes とその他のインフラストラクチャ リソースを使用して適切に設計されたアプローチとともに、異種混合デプロイの一般的な使用事例を実践します。</p>

<h3>ゾーンを設定する</h3>
<p>ローカルゾーンとして <code>us-central1-a</code> を指定して次のコマンドを実行し、使用する GCP ゾーンを設定します。</p>
<pre><code>gcloud config set compute/zone us-central1-a&#x000A;</code></pre>
<h3>このラボのサンプルコードを入手する</h3>
<p>コンテナとデプロイを作成して実行するためのサンプルコードを入手します。</p>
<pre><code>git clone https://github.com/googlecodelabs/orchestrate-with-kubernetes.git&#x000A;cd orchestrate-with-kubernetes/kubernetes&#x000A;</code></pre>
<p>5 つの <code>n1-standard-1</code> ノードを含むクラスタを作成します（完了するまでに数分かかります）。</p>
<pre><code>gcloud container clusters create bootcamp --num-nodes 5 --scopes "https://www.googleapis.com/auth/projecthosting,storage-rw"&#x000A;</code></pre>
<h2 id="step5">Deployment オブジェクトの詳細</h2>
<p>Deployment を使ってみましょう。まずは Deployment オブジェクトについて確認します。<code>kubectl</code> の <code>explain</code> コマンドを使用すると、Deployment オブジェクトに関する情報が表示されます。</p>
<pre><code>kubectl explain deployment&#x000A;</code></pre>
<p><code>--recursive</code> オプションを使用してすべてのフィールドを確認することもできます。</p>
<pre><code>kubectl explain deployment --recursive&#x000A;</code></pre>
<p>ラボを進めながら explain コマンドを使用すると、Deployment オブジェクトの構造や、各フィールドの機能を理解するのに役立ちます。</p>
<pre><code>kubectl explain deployment.metadata.name&#x000A;</code></pre>
<h2 id="step6">Deployment の作成</h2>
<p><code>deployments/auth.yaml</code> ファイルを更新します。</p>
<pre><code>vi deployments/auth.yaml&#x000A;</code></pre>
<p>エディタを起動します。</p>
<pre><code>i&#x000A;</code></pre>
<p>Deployment の containers セクションにある <code>image</code> を次のように変更します。</p>
<pre><code>…&#x000A;containers:&#x000A;- name: auth&#x000A;  image: kelseyhightower/auth:1.0.0&#x000A;...&#x000A;</code></pre>
<p><code>auth.yaml</code> ファイルを保存します（<code>Esc</code> キーを押してから、次のように入力します）。</p>
<pre><code>:wq&#x000A;</code></pre>
<p>次に、簡単なデプロイを作成してみましょう。デプロイ構成ファイルを確認します。</p>
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
          image: "kelseyhightower/auth:1.0.0"
          ports:
            - name: http
              containerPort: 80
            - name: health
              containerPort: 81
...
```

<p>Deployment でレプリカが 1 つ作成され、バージョン 1.0.0 の auth コンテナが使用されているのがわかります。</p>
<p>auth デプロイを作成するために <code>kubectl create</code> コマンドを実行すると、Deployment マニフェスト内のデータに準拠するポッドが 1 つ作成されます。つまり、<code>replicas</code> フィールドで指定された数字を変更することで、ポッドの数を増減できます。</p>
<p>では、<code>kubectl create</code> を使用して Deployment オブジェクトを作成しましょう。</p>
<pre><code>kubectl create -f deployments/auth.yaml&#x000A;</code></pre>
<p>Deployment が実際に作成されていることを確認します。</p>
<pre><code>kubectl get deployments&#x000A;</code></pre>
<p>Deployment が作成されると、Kubernetes はその ReplicaSet を作成します。次のように入力すると、Deployment の ReplicaSet が作成されたことを確認できます。</p>
<pre><code>kubectl get replicasets&#x000A;</code></pre>
<p><code>auth-xxxxxxx</code> のような名前の ReplicaSet が表示されます。</p>
<p>最後に、Deployment の一部として作成されたポッドを確認します。ReplicaSet が作成されると、Kubernetes によってポッドが 1 つ作成されます。</p>
<pre><code>kubectl get pods&#x000A;</code></pre>
<p>次に、auth デプロイ用のサービスを作成します。サービス マニフェスト ファイルについてはすでにご存知のはずですから、ここでは説明しません。<code>kubectl create</code> コマンドを使用して auth サービスを作成します。</p>
<pre><code>kubectl create -f services/auth.yaml&#x000A;</code></pre>
<p>次に、同じ手順で hello Deployment を作成して公開します。</p>
<pre><code>kubectl create -f deployments/hello.yaml&#x000A;kubectl create -f services/hello.yaml&#x000A;</code></pre>
<p>さらに、同じ手順でフロントエンド Deployment を作成して公開します。</p>
<pre><code>kubectl create secret generic tls-certs --from-file tls/&#x000A;kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf&#x000A;kubectl create -f deployments/frontend.yaml&#x000A;kubectl create -f services/frontend.yaml&#x000A;</code></pre>
<aside class="special"><p><strong>注:</strong> フロントエンドの ConfigMap が作成されました。</p>
</aside>
<p>フロントエンドの外部 IP を取得して curl を実行し、フロントエンドを操作します。</p>
<pre><code>kubectl get services frontend&#x000A;curl -ks https://&lt;EXTERNAL-IP&gt;&#x000A;</code></pre>
<p>hello レスポンスが返されます。</p>
<p><code>kubectl</code> の出力テンプレート機能を使用して、curl を 1 行で実行することもできます。</p>
<pre><code>curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`&#x000A;</code></pre>

<h3>Deployment をスケールする</h3>
<p>作成した Deployment をスケーリングするには、<code>spec.replicas</code> フィールドを更新します。<code>kubectl explain</code> コマンドをもう一度使用して、このフィールドの説明を確認してください。</p>
<pre><code>kubectl explain deployment.spec.replicas&#x000A;</code></pre>
<p>replicas フィールドは、<code>kubectl scale</code> コマンドを使って簡単に更新できます。</p>
<pre><code>kubectl scale deployment hello --replicas=5&#x000A;</code></pre>
<aside class="warning"><p><strong>注:</strong> 新しいポッドがすべて起動するまでに数分かかる場合があります。</p>
</aside>
<p>Deployment が更新されると、Kubernetes は関連する ReplicaSet を自動的に更新し、ポッドの総数が 5 になるように新しいポッドを起動します。</p>
<p>5 つの <code>hello</code> ポッドが実行されていることを確認します。</p>
<pre><code>kubectl get pods | grep hello- | wc -l&#x000A;</code></pre>
<p>今度はアプリケーションを縮小します。</p>
<pre><code>kubectl scale deployment hello --replicas=3&#x000A;</code></pre>
<p>ここでも、正しい数のポッドが起動していることを確認します。</p>
<pre><code>kubectl get pods | grep hello- | wc -l&#x000A;</code></pre>
<p>Kubernetes のデプロイと、ポッドのグループを管理してスケーリングする方法について学びました。</p>
<h2 id="step7">ローリング アップデート</h2>
<p>Deployment では、ローリング アップデートのメカニズムを使用してイメージを新しいバージョンに更新できます。Deployment を新しいバージョンで更新すると、ReplicaSet が新たに作成され、古い ReplicaSet のレプリカ数が減少するにつれて、新しい ReplicaSet のレプリカ数が徐々に増加します。</p>
<p><img alt="8d107e36763fd5c1.png" src="https://cdn.qwiklabs.com/uc6D9jQ5Blkv8wf%2FccEcT35LyfKDHz7kFpsI4oHUmb0%3D"></p>
<h3>ローリング アップデートをトリガーする</h3>
<p>Deployment を更新するには、次のコマンドを実行します。</p>
<pre><code>kubectl edit deployment hello&#x000A;</code></pre>
<p>Deployment の containers セクションにある <code>image</code> を次のように変更します。</p>
<pre><code>…&#x000A;containers:&#x000A;- name: hello&#x000A;  image: kelseyhightower/hello:2.0.0&#x000A;...&#x000A;</code></pre>
<p>保存して終了します。</p>
<p>保存してエディタを終了すると、更新された Deployment がクラスタに保存され、Kubernetes がローリング アップデートを開始します。</p>
<p>Kubernetes によって作成された新しい ReplicaSet を確認します。</p>
<pre><code>kubectl get replicaset&#x000A;</code></pre>

**出力例（イメージ。ひととおりハンズオン終わったあとでまたコマンド実行した結果を貼り付けてるので多少違うかも）**
```
NAME                      DESIRED   CURRENT   READY   AGE
auth-65965d94d6           1         1         1       29m
frontend-5dd6cbb4dd       1         1         1       22m
hello-56ff65f9bf          3         3         3       24m
hello-5745d7776d          0         0         0       16m
```

<p>ロールアウト履歴にも新しいエントリが表示されます。</p>
<pre><code>kubectl rollout history deployment/hello&#x000A;</code></pre>

**出力例（これもイメージで、多少違う可能性あり）**

```
deployment.extensions/hello
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

<h3>ローリング アップデートを一時停止する</h3>
<p>実行中のロールアウトで問題が検出された場合は、一時停止してアップデートを中止し、次の手順を試してください。</p>
<pre><code>kubectl rollout pause deployment/hello&#x000A;</code></pre>

**出力**
```
deployment.extensions/hello paused
```

<p>ロールアウトの現在の状態を確認します。</p>
<pre><code>kubectl rollout status deployment/hello&#x000A;</code></pre>

**出力（例）**

```
deployment "hello" successfully rolled out
```

<p>ポッドで直接確認することもできます。</p>
<pre><code>kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'&#x000A;</code></pre>

**出力例（イメージ）**
```
auth-65965d94d6-6s274           kelseyhightower/auth:1.0.0
frontend-5dd6cbb4dd-9stkq               nginx:1.9.14
hello-56ff65f9bf-8z6sc          kelseyhightower/hello:1.0.0
hello-56ff65f9bf-l6lbl          kelseyhightower/hello:1.0.0
hello-56ff65f9bf-vcjzr          kelseyhightower/hello:1.0.0
```

<h3>ローリング アップデートを再開する</h3>
<p>ロールアウトが一時停止されるということは、一部のポッドが新しいバージョンで、その他のポッドは古いバージョンであることを意味します。<code>resume</code> コマンドを使用してロールアウトを続行してください。</p>
<pre><code>kubectl rollout resume deployment/hello&#x000A;</code></pre>

**出力**

```
deployment.extensions/hello resumed
```

<p>ロールアウトの完了後に <code>status</code> コマンドを実行すると、次のように表示されます。</p>
<pre><code>kubectl rollout status deployment/hello&#x000A;</code></pre>
<p>（出力）</p>
<pre><code>deployment "hello" successfully rolled out&#x000A;</code></pre>
<h3>アップデートをロールバックする</h3>
<p>新しいバージョンでバグが検出されたと仮定します。この場合は新しいバージョンに問題があると考えられるため、新しいポッドに接続したユーザーにこれらの問題が発生することになります。</p>
<p>調査後にきちんと修正したバージョンをリリースするために、以前のバージョンにロールバックする必要があります。</p>
<p><code>rollout</code> コマンドを使用して、以前のバージョンにロールバックします。</p>
<pre><code>kubectl rollout undo deployment/hello&#x000A;</code></pre>

**出力**

```
deployment.extensions/hello rolled back
```

<p>履歴でロールバックを確認します。</p>
<pre><code>kubectl rollout history deployment/hello&#x000A;</code></pre>

**出力（例、イメージ）**

```
REVISION  CHANGE-CAUSE
2         <none>
3         <none>
```

<p>最後に、すべてのポッドが以前のバージョンにロールバックされていることを確認します。</p>
<pre><code>kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'&#x000A;</code></pre>
<p>これで完了です。Kubernetes デプロイのローリング アップデートと、ダウンタイムなしでアプリケーションを更新する方法について学びました。</p>
<h2 id="step8">カナリア デプロイ</h2>
<p>本番環境で一部のユーザーを対象に新しいデプロイをテストする場合は、カナリア デプロイを使用します。カナリア デプロイでは、一部のユーザーに変更をリリースすることで、新しいリリースに伴うリスクを軽減できます。</p>
<h3>カナリア デプロイを作成する</h3>
<p>カナリア デプロイは、新しいバージョンを含む独立したデプロイと、通常の安定したデプロイとカナリア デプロイの両方を対象とするサービスで構成されています。</p>
<p><img alt="48190cf58fdf2eeb.png" src="https://cdn.qwiklabs.com/qSrgIP5FyWKEbwOk3PMPAALJtQoJoEpgJMVwauZaZow%3D"></p>
<p>まず、新しいバージョン用の新しいカナリア デプロイを作成します。</p>
<pre><code>cat deployments/hello-canary.yaml&#x000A;</code></pre>
<p>（出力）</p>

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: hello-canary
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: hello
        track: canary
        # Use ver 1.0.0 so it matches version on service selector
        version: 1.0.0
    spec:
      containers:
        - name: hello
          image: kelseyhightower/hello:2.0.0
          ports:
            - name: http
              containerPort: 80
            - name: health
              containerPort: 81
...
```

<p>バージョンを 1.0.0 に更新してください（他のバージョンが指定されている場合）。</p>
<p>カナリア デプロイを作成します。</p>
<pre><code>kubectl create -f deployments/hello-canary.yaml&#x000A;</code></pre>
<p>カナリア デプロイが作成されると、<code>hello</code> と <code>hello-canary</code> の 2 つのデプロイが存在することになります。次の <code>kubectl</code> コマンドを使用して確認してください。</p>
<pre><code>kubectl get deployments&#x000A;</code></pre>
<p><code>hello</code> サービスのセレクタは、本番環境デプロイメントとカナリア デプロイメントの<strong>両方</strong>のポッドに一致する <code>app:hello</code> セレクタを使用します。ただし、カナリア デプロイのポッド数は少ないため、少数のユーザーにしか表示されません。</p>
<h3>カナリア デプロイを確認する</h3>
<p>リクエストに応じて提供される <code>hello</code> バージョンを確認できます。</p>
<pre><code>curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version&#x000A;</code></pre>
<p>これを数回実行すると、一定のリクエストが hello 1.0.0 によって処理され、ごく一部（1/4 = 25%）が 2.0.0 によって処理されることがわかります。</p>

<h3>本番環境でのカナリア デプロイメント - セッション アフィニティ</h3>
<p>このラボで Nginx サービスに送信された各リクエストは、カナリア デプロイで処理される可能性がありましたが、カナリア デプロイでユーザーが処理されることを避けたい場合もあります。たとえば、アプリケーションの UI が変更されるユースケースで、ユーザーを混乱させたくないときなどです。このような場合は、ユーザーをいずれかのデプロイに「固定」できます。</p>
<p>これは、セッション アフィニティを指定してサービスを作成することで実現できます。これにより、同じユーザーが常に同じバージョンで処理されます。以下の例では、サービスは以前と同じですが、新しく追加された <code>sessionAffinity</code> フィールドが ClientIP に設定されているため、同じ IP アドレスを持つすべてのクライアントのリクエストは、同じバージョンの <code>hello</code> アプリケーションに送信されることになります。</p>

```yaml
kind: Service
apiVersion: v1
metadata:
  name: "hello"
spec:
  sessionAffinity: ClientIP
  selector:
    app: "hello"
  ports:
    - protocol: "TCP"
      port: 80
      targetPort: 80
```

<p>このテスト環境の設定は簡単ではないためここでは行いませんが、本番環境でのカナリア デプロイには <code>sessionAffinity</code> を使用することをおすすめします。</p>
<h2 id="step9">Blue / Green デプロイ</h2>
<p>オーバーヘッド、パフォーマンスへの影響、ダウンタイムを最小限に抑えながら徐々にアプリケーションをデプロイできるローリング アップデートは、理想的な方法です。さらに、完全にデプロイされるまで新しいバージョンを指定しないようにロードバランサを変更すると効果的な場合があります。これには、Blue / Green デプロイが有効です。</p>
<p>Kubernetes は、古い「Blue」バージョン用と新しい「Green」バージョン用に 2 つの異なるデプロイを作成することでこれを実現します。「Blue」バージョンには既存の <code>hello</code> デプロイを使用します。このデプロイには、ルータとして機能するサービスを介してアクセスします。新しい「Green」バージョンが稼働したら、サービスを更新してそのバージョンを使用するように切り替えます。</p>
<p><img alt="9e624196fdaf4534.png" src="https://cdn.qwiklabs.com/POW8Q247ZKNY%2ByHIartCsoEu8MAih7k4u1twusCx6pw%3D"></p>
<aside class="warning"><p>Blue / Green デプロイの欠点は、アプリケーションをホストするために必要なクラスタのリソースが少なくとも 2 倍になることです。アプリケーションの両方のバージョンを同時にデプロイする前に、クラスタに十分なリソースがあることを確認してください。</p>
</aside>
<h3>サービス</h3>
<p>既存の hello サービスを使用しますが、<code>app:hello</code>、<code>version: 1.0.0</code> のセレクタを持つように更新してください。 このセレクタは、既存の「Blue」デプロイとは一致しますが、「Green」デプロイとは一致しません。使用するバージョンが異なるからです。</p>
<p>まずはサービスを更新します。</p>
<pre><code>kubectl apply -f services/hello-blue.yaml&#x000A;</code></pre>
<h3>Blue / Green デプロイを使用したアップデート</h3>
<p>Blue / Green デプロイ スタイルをサポートするために、新しいバージョン用に「Green」デプロイを新たに作成します。この Green デプロイによって、バージョン ラベルとイメージパスが更新されます。</p>

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: hello-green
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: hello
        track: stable
        version: 2.0.0
    spec:
      containers:
        - name: hello
          image: kelseyhightower/hello:2.0.0
          ports:
            - name: http
              containerPort: 80
            - name: health
              containerPort: 81
          resources:
            limits:
              cpu: 0.2
              memory: 10Mi
          livenessProbe:
            httpGet:
              path: /healthz
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            periodSeconds: 15
            timeoutSeconds: 5
          readinessProbe:
            httpGet:
              path: /readiness
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            timeoutSeconds: 1
```

<p>Green デプロイを作成します。</p>
<pre><code>kubectl create -f deployments/hello-green.yaml&#x000A;</code></pre>
<p>Green デプロイを作成して適切に起動したら、現在のバージョンである 1.0.0 がまだ使用されていることを確認します。</p>
<pre><code>curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version&#x000A;</code></pre>
<p>次に、新しいバージョンを指定するようにサービスを更新します。</p>
<pre><code>kubectl apply -f services/hello-green.yaml&#x000A;</code></pre>
<p>サービスが更新されると、すぐに「Green」デプロイが使用されるようになり、常に新しいバージョンが使用されていることが確認できます。</p>
<pre><code>curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version&#x000A;</code></pre>
<h3>Blue / Green ロールバック</h3>
<p>必要に応じて、同じ方法で古いバージョンにロールバックできます。「Blue」デプロイがまだ実行されている間に、サービスを古いバージョンに戻すだけです。</p>
<pre><code>kubectl apply -f services/hello-blue.yaml&#x000A;</code></pre>
<p>サービスを更新すると、ロールバックが完了します。再度、正しいバージョンが使用されていることを確認してください。</p>
<pre><code>curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version&#x000A;</code></pre>
<p>これで完了です。Blue / Green デプロイと、バージョンをすべて一度に切り替える必要があるアプリケーションにアップデートをデプロイする方法について学びました。</p>
<h2 id="step10">これで完了です。</h2>
<p>これで Kubernetes によってデプロイを管理するハンズオンラボが終了しました。このラボでは、<code>kubectl</code> コマンドライン ツールと、YAML ファイルに設定されたさまざまなスタイルのデプロイ構成を活用して、デプロイをリリース、更新、スケーリングできました。この基礎演習で培ったスキルは、実際の DevOps 環境で問題なく役立つことでしょう。</p>

# Kubernetes Engine での Jenkins を使用した継続的デリバリー
<h2 id="step2">概要</h2>
<p>このラボでは、Kubernetes Engine で <code>Jenkins</code> を使用して継続的デリバリー パイプラインを設定する方法を学びます。Jenkins とは、共有リポジトリに頻繁にコードを統合するデベロッパーによく利用されているオートメーション サーバーです。このラボでは、次の図のようなソリューションを構築します。</p>
<p><img alt="overview.png" src="https://cdn.qwiklabs.com/1b%2B9D20QnfRjAF8c6xlXmexot7TDcOsYzsRwp%2FH4ErE%3D"></p>
<p>Kubernetes で Jenkins を実行する方法について詳しくは、<a href="https://cloud.google.com/solutions/jenkins-on-container-engine">こちら</a>をご覧ください。</p>
<h3><strong>演習内容</strong></h3>
<p>このラボでは、次のタスクを行います。</p>
<ul>
<li>Jenkins アプリケーションを Kubernetes Engine クラスタにプロビジョニングする</li>
<li>Helm パッケージ マネージャーを使用して Jenkins アプリケーションを設定する</li>
<li>Jenkins アプリケーションの機能を試す</li>
<li>Jenkins パイプラインを作成して実行する</li>
</ul>
<h3>前提事項</h3>
<p>このラボは<strong>上級者向け</strong>です。受講する前に、少なくともシェル プログラミング、Kubernetes、Jenkins の基礎を理解しておく必要があります。これらの基礎は、次の Qwiklabs で効率的に習得できます。</p>
<ul>
<li><a href="https://google.qwiklabs.com/labs/944/">Docker の概要</a></li>
<li><a href="https://google.qwiklabs.com/catalog_lab/468">Hello Node Kubernetes</a></li>
<li><a href="https://google.qwiklabs.com/catalog_lab/572">Kubernetes Engine によるデプロイの管理</a></li>
<li><a href="https://google.qwiklabs.com/catalog_lab/1093">Kubernetes Engine での Jenkins の設定</a></li>
</ul>
<p>準備ができたら下にスクロールして、Kubernetes、Jenkins、継続的デリバリーについて詳しく学んでいきましょう。</p>
<h3>Kubernetes Engine とは</h3>
<p>Kubernetes Engine は、高度なクラスタ マネージャーでありコンテナのオーケストレーション システムである <code>Kubernetes</code> の GCP ホスト バージョンです。Kubernetes はオープンソースのプロジェクトで、ノートパソコンや可用性の高いマルチノード クラスタ、仮想マシンからベアメタルまで、さまざまな環境で扱うことができます。前述のように、Kubernetes アプリは<code>コンテナ</code>上に構築される、実行に必要なすべての依存関係とライブラリがバンドルされた軽量アプリケーションです。この基本的な構造により、Kubernetes アプリケーションの高可用性、安全性、迅速なデプロイといった、クラウド デベロッパーが理想とするフレームワークが実現します。</p>
<h3>Jenkins とは</h3>
<p><a href="https://jenkins.io/">Jenkins</a> は、ビルド、テスト、デプロイ用パイプラインの柔軟なオーケストレーションを可能にするオープンソースのオートメーション サーバーです。Jenkins を使用すれば、継続的デリバリーに起因するオーバーヘッドの問題を気に掛けることなく、プロジェクトに対して迅速に反復処理を行うことができます。</p>
<h3>継続的デリバリーと継続的デプロイについて</h3>
<p>継続的デリバリー（CD）パイプラインを設定する必要がある場合、Kubernetes Engine での Jenkins のデプロイには、標準の VM ベースのデプロイよりも大きなメリットがあります。</p>
<p>ビルドプロセスでコンテナを使用すると、1 つの仮想ホストが複数のオペレーティング システムでジョブを実行できます。Kubernetes Engine が提供する<code>エフェメラル ビルド エグゼキュータ</code>は、ビルドがアクティブに実行されている場合にのみ使用されるため、バッチ処理ジョブなどの他のクラスタタスク用にリソースを確保できます。エフェメラル ビルド エグゼキュータのもう 1 つの利点は速度の速さで、わずか数秒で起動します。<em></em></p>
<p>また、Kubernetes Engine には Google のグローバル ロードバランサがすでに組み込まれており、これを使用してインスタンスへのウェブ トラフィックのルーティングを自動化できます。ロードバランサは SSL ターミネーションを処理し、Google のバックボーン ネットワーク（ユーザーのウェブフロントに結合）で設定されたグローバル IP アドレスを構成します。このロードバランサにより、ユーザーは常に最短パスでアプリケーション インスタンスに接続されます。</p>
<p>Kubernetes と Jenkins、そして CD パイプラインにおけるこれらの関係についてある程度理解したところで、実際にビルドしてみましょう。</p>

<h2 id="step4">リポジトリのクローンを作成する</h2>
<p>まず、Cloud Shell で新しいセッションを開き、次のコマンドを実行してゾーン <code>us-east1-d</code> を設定します。</p>
<pre><code>gcloud config set compute/zone us-east1-d&#x000A;</code></pre>
<p>ラボのサンプルコードのクローンを作成します。</p>
<pre><code>git clone https://github.com/GoogleCloudPlatform/continuous-deployment-on-kubernetes.git&#x000A;</code></pre>
<p>適切なディレクトリに移動します。</p>
<pre><code>cd continuous-deployment-on-kubernetes&#x000A;</code></pre>
<h2 id="step5">Jenkins をプロビジョニングする</h2>
<h3><strong>Kubernetes クラスタを作成する</strong></h3>
<p>次に、以下のコマンドを実行して Kubernetes クラスタをプロビジョニングします。</p>
<pre><code>gcloud container clusters create jenkins-cd \&#x000A;--num-nodes 2 \&#x000A;--machine-type n1-standard-2 \&#x000A;--scopes "https://www.googleapis.com/auth/source.read_write,cloud-platform"&#x000A;</code></pre>
<p>この手順は完了までに数分かかる場合があります。追加のスコープで、Jenkins が Cloud Source Repositories と Container Registry にアクセスできるようになります。</p>

<p>続行する前に、次のコマンドを使用してクラスタが実行されていることを確認します。</p>
<pre><code>gcloud container clusters list&#x000A;</code></pre>
<p>次に、クラスタの認証情報を取得します。</p>
<pre><code>gcloud container clusters get-credentials jenkins-cd&#x000A;</code></pre>
<p>Kubernetes Engine はこれらの認証情報を使用して、新たにプロビジョニングされたクラスタにアクセスします。次のコマンドを実行して、クラスタに接続できることを確認してください。</p>
<pre><code>kubectl cluster-info&#x000A;</code></pre>
<h2 id="step6">Helm をインストールする</h2>
<p>このラボでは、Helm を使用して Charts リポジトリから Jenkins をインストールします。Helm は、Kubernetes アプリケーションの構成とデプロイを容易にするパッケージ マネージャーです。Jenkins をインストールすると、CI / CD パイプラインを設定できるようになります。</p>
<ol>
<li>
<p>Helm バイナリをダウンロードしてインストールします。</p>
</li>
</ol>
<pre><code>wget https://storage.googleapis.com/kubernetes-helm/helm-v2.14.1-linux-amd64.tar.gz&#x000A;</code></pre>
<ol start="2">
<li>
<p>Cloud Shell でファイルを展開します。</p>
</li>
</ol>
<pre><code>tar zxfv helm-v2.14.1-linux-amd64.tar.gz&#x000A;cp linux-amd64/helm .&#x000A;</code></pre>
<ol start="3">
<li>
<p>クラスタの RBAC に自分をクラスタ管理者として追加します。これにより、クラスタ内での権限を Jenkins に付与できるようになります。</p>
</li>
</ol>
<pre><code>kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=$(gcloud config get-value account)&#x000A;</code></pre>
<ol start="4">
<li>
<p>Helm のサーバー側である Tiller に、クラスタの cluster-admin ロールを付与します。</p>
</li>
</ol>
<pre><code>kubectl create serviceaccount tiller --namespace kube-system&#x000A;kubectl create clusterrolebinding tiller-admin-binding --clusterrole=cluster-admin --serviceaccount=kube-system:tiller&#x000A;</code></pre>

<ol start="5">
<li>
<p>Helm を初期化します。これにより、Helm のサーバー側（Tiller）がクラスタ内に正常にインストールされます。</p>
</li>
</ol>
<pre><code>./helm init --service-account=tiller&#x000A;./helm update&#x000A;</code></pre>

<ol start="6">
<li>
<p>次のコマンドを実行して、Helm が正しくインストールされていることを確認します。クライアントとサーバーのどちらのバージョンも <code>v2.14.1</code> と表示される必要があります。</p>
</li>
</ol>
<pre><code>./helm version&#x000A;</code></pre>
<p><strong>出力例:</strong></p>
<pre><code>Client: &amp;version.Version{SemVer:"v2.14.1", GitCommit:"5270352a09c7e8b6e8c9593002a73535276507c0", GitTreeState:"clean"}&#x000A;Server: &amp;version.Version{SemVer:"v2.14.1", GitCommit:"5270352a09c7e8b6e8c9593002a73535276507c0", GitTreeState:"clean"}&#x000A;</code></pre>
<h2 id="step7">Jenkins を構成、インストールする</h2>
<p>サービス アカウントの認証情報で Cloud Source Repositories にアクセスするために必要な GCP 固有のプラグインを、カスタムの値ファイルを使って追加します。</p>
<ol>
<li>
<p>Helm CLI を使用して、構成設定とともにチャートをデプロイします。</p>
</li>
</ol>
<pre><code>./helm install -n cd stable/jenkins -f jenkins/values.yaml --version 1.2.2 --wait&#x000A;</code></pre>
<p>このコマンドは完了までに数分かかる場合があります。</p>

<ol start="2">
<li>
<p>コマンドが完了したら、Jenkins Pod が Running 状態になり、コンテナが READY 状態になっていることを確認します。</p>
</li>
</ol>
<pre><code>kubectl get pods&#x000A;</code></pre>
<p><strong>出力例:</strong></p>
<pre><code>NAME                          READY     STATUS    RESTARTS   AGE&#x000A;cd-jenkins-7c786475dd-vbhg4   1/1       Running   0          1m&#x000A;</code></pre>
<ol start="3">
<li>
<p>クラスタにデプロイできるように Jenkins サービス アカウントを構成します。</p>
</li>
</ol>
<pre><code>kubectl create clusterrolebinding jenkins-deploy --clusterrole=cluster-admin --serviceaccount=default:cd-jenkins&#x000A;</code></pre>
<p>次の出力が表示されます。</p>
<pre><code class="language-output prettyprint">clusterrolebinding.rbac.authorization.k8s.io/jenkins-deploy created&#x000A;</code></pre>
<ol start="4">
<li>
<p>次のコマンドを実行して、Cloud Shell から Jenkins UI へのポート転送を設定します。</p>
</li>
</ol>
<pre><code>export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/component=jenkins-master" -l "app.kubernetes.io/instance=cd" -o jsonpath="{.items[0].metadata.name}")&#x000A;kubectl port-forward $POD_NAME 8080:8080 &gt;&gt; /dev/null &amp;&#x000A;</code></pre>
<ol start="5">
<li>
<p>Jenkins サービスが適切に作成されたことを確認します。</p>
</li>
</ol>
<pre><code>kubectl get svc&#x000A;</code></pre>
<p><strong>出力例:</strong></p>
<pre><code>NAME               CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE&#x000A;cd-jenkins         10.35.249.67   &lt;none&gt;        8080/TCP    3h&#x000A;cd-jenkins-agent   10.35.248.1    &lt;none&gt;        50000/TCP   3h&#x000A;kubernetes         10.35.240.1    &lt;none&gt;        443/TCP     9h&#x000A;</code></pre>
<p>Jenkins マスターがリクエストしたときに必要に応じて自動的にビルダーノードが起動するように、<a href="https://wiki.jenkins-ci.org/display/JENKINS/Kubernetes+Plugin">Kubernetes プラグイン</a>を使用しています。処理が完了するとノードが自動的に終了し、リソースがクラスタのリソースプールに戻されます。</p>
<p>このサービスでは、<code>selector</code> に一致するすべての Pod のポート <code>8080</code> と <code>50000</code> が公開されることに注意してください。ここでは、Kubernetes クラスタ内の Jenkins ウェブ UI ポートとビルダー / エージェント登録ポートが公開されます。また、<code>jenkins-ui</code> サービスは ClusterIP を使用して公開されるため、クラスタ外からはアクセスできません。</p>
<h2 id="step8">Jenkins に接続する</h2>
<ol>
<li>
<p>Jenkins チャートによって管理者パスワードが自動的に作成されます。このパスワードを取得するには、以下を実行します。</p>
</li>
</ol>
<pre><code>printf $(kubectl get secret cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo&#x000A;</code></pre>
<ol start="2">
<li>Jenkins ユーザー インターフェースを表示するには、Cloud Shell で [ウェブでプレビュー] をクリックし、[ポート 8080 でプレビュー] をクリックします。</li>
</ol>
<p><img alt="web_prev.png" src="https://cdn.qwiklabs.com/wy13PEPdV6ZbYMJR2tmk3iKe%2FEyVDXVWtrWFVJeZUXk%3D"></p>
<ol start="3">
<li>ユーザー名「admin」と自動生成されたパスワードでログインします。</li>
</ol>
<p>これで、Kubernetes クラスタに Jenkins が設定されました。次のセクションでは、自動化した CI / CD パイプラインを Jenkins によって実行します。</p>
<h2 id="step9">アプリケーションについて理解する</h2>
<p>継続的デプロイ パイプラインにサンプル アプリケーション <code>gceme</code> をデプロイします。このアプリケーションは Go 言語で作成され、リポジトリの sample-app ディレクトリに保存されます。Compute Engine インスタンスで gceme バイナリを実行すると、アプリの情報カードにインスタンスのメタデータが表示されます。</p>
<p><img alt="90fb779a65809b9e.png" src="https://cdn.qwiklabs.com/ceqFX6Vwtd12NSBtnNhrkemKfRSLHbCmFZiLn8WmC98%3D"></p>
<p>2 つのオペレーション モードに対応するこのアプリケーションは、マイクロサービスに似た機能を提供します。</p>
<ul>
<li>
<strong>バックエンド モード</strong>では、gceme がポート 8080 をリッスンし、Compute Engine インスタンスのメタデータを JSON 形式で返します。</li>
<li>
<strong>フロントエンド モード</strong>では、gceme がバックエンド gceme サービスにクエリを送信し、ユーザー インターフェースに JSON を表示します。</li>
</ul>
<p><img alt="fc22b68ab20dfe0e.png" src="https://cdn.qwiklabs.com/P1T5JBWWprA4iLf%2FB5%2BO6as7otLE25YBde57gzZwSz4%3D"></p>
<h2 id="step10">アプリケーションをデプロイする</h2>
<p>アプリケーションを 2 つの異なる環境にデプロイします。</p>
<ul>
<li>
<strong>本番環境</strong>: ユーザーがアクセスする公開サイト。</li>
<li>
<strong>カナリア環境</strong>: 少量のユーザー トラフィックが発生する、キャパシティの小さいサイト。この環境は、ソフトウェアをすべてのユーザーにリリースする前に、ライブ トラフィックを使って検証する場合に使用します。</li>
</ul>
<p>Google Cloud Shell で、サンプル アプリケーションのディレクトリに移動します。</p>
<pre><code>cd sample-app&#x000A;</code></pre>
<p>Kubernetes 名前空間を作成して、デプロイする環境を論理的に隔離します。</p>
<pre><code>kubectl create ns production&#x000A;</code></pre>

<p><code>kubectl apply</code> コマンドを使用して、本番デプロイメント、カナリア デプロイメント、サービスを作成します。</p>
<pre><code>kubectl apply -f k8s/production -n production&#x000A;</code></pre>
<pre><code>kubectl apply -f k8s/canary -n production&#x000A;</code></pre>
<pre><code>kubectl apply -f k8s/services -n production&#x000A;</code></pre>

<p>デフォルトでは、フロントエンドのレプリカが 1 つだけデプロイされます。<code>kubectl scale</code> コマンドを使用して、少なくとも 4 つのレプリカが常時実行されるようにします。</p>
<p>次のコマンドを実行して、本番環境のフロントエンドをスケールアップします。</p>
<pre><code>kubectl scale deployment gceme-frontend-production -n production --replicas 4&#x000A;</code></pre>
<p>フロントエンド用に 5 つのポッド（本番トラフィック用に 4 つのポッド、カナリア リリース用に 1 つのポッド）が実行されていることを確認します。カナリア リリースへの変更は 5 人中 1 人（20%）のユーザーにのみ影響します。</p>
<pre><code>kubectl get pods -n production -l app=gceme -l role=frontend&#x000A;</code></pre>
<p>また、バックエンド用に 2 つのポッド（1 つは本番用、もう 1 つはカナリア用）があることも確認してください。</p>
<pre><code>kubectl get pods -n production -l app=gceme -l role=backend&#x000A;</code></pre>
<p>本番環境サービスの外部 IP を取得します。</p>
<pre><code>kubectl get service gceme-frontend -n production&#x000A;</code></pre>
<aside class="warning">
<p><strong>注: </strong>ロードバランサの外部 IP アドレスが表示されるまでに数分かかる場合があります。</p>
</aside>
<p><strong>出力例:</strong></p>
<pre><code class="language-bash prettyprint">NAME            TYPE          CLUSTER-IP     EXTERNAL-IP     PORT(S)  AGE&#x000A;gceme-frontend  LoadBalancer  10.79.241.131  104.196.110.46  80/TCP   5h&#x000A;</code></pre>
<p><strong>外部 IP</strong> をブラウザに貼り付けると、以下のような情報カードが表示されます。</p>
<p><img alt="backend_card.png" src="https://cdn.qwiklabs.com/dfXfZfxRk0UMSTZm8FyIaLFsHla7AlUmxHO70UTOFjk%3D"></p>
<p>次に、環境変数にフロントエンド サービスのロードバランサの IP を保存して、後で使用できるようにします。<em></em></p>
<pre><code>export FRONTEND_SERVICE_IP=$(kubectl get -o jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)&#x000A;</code></pre>
<p>ブラウザでフロントエンドの外部 IP アドレスを開き、両方のサービスが正常に動作していることを確認します。次のコマンドを実行して、サービスのバージョンを確認します（1.0.0 と表示されるはずです）。</p>
<pre><code>curl http://$FRONTEND_SERVICE_IP/version&#x000A;</code></pre>
<p>これで、サンプル アプリケーションが正常にデプロイされました。次は、継続的かつ確実に変更をデプロイするためのパイプラインを設定します。</p>
<h2 id="step11">Jenkins パイプラインを作成する</h2>
<h3><strong>サンプルアプリのソースコードをホストするリポジトリを作成する</strong></h3>
<p><code>gceme</code> サンプルアプリのコピーを作成して、<a href="https://cloud.google.com/source-repositories/docs/">Cloud Source Repositories</a> に push します。</p>
<pre><code>gcloud source repos create default&#x000A;</code></pre>
<p>表示される警告は無視してください。このリポジトリの使用料を請求されることはありません。</p>

<pre><code>git init&#x000A;</code></pre>
<p>sample-app ディレクトリを独自の Git リポジトリとして初期化します。</p>
<pre><code>git config credential.helper gcloud.sh&#x000A;</code></pre>
<p>次のコマンドを実行します。</p>
<pre><code>git remote add origin https://source.developers.google.com/p/$DEVSHELL_PROJECT_ID/r/default&#x000A;</code></pre>
<p>Git commit にユーザー名とメールアドレスを設定します。<code>[EMAIL_ADDRESS]</code> を Git メールアドレスに置き換え、<code>[USERNAME]</code> を Git ユーザー名に置き換えてください。</p>
<pre><code>git config --global user.email "[EMAIL_ADDRESS]"&#x000A;</code></pre>
<pre><code>git config --global user.name "[USERNAME]"&#x000A;</code></pre>
<p>ファイルの追加、commit、push を行います。</p>
<pre><code>git add .&#x000A;</code></pre>
<pre><code>git commit -m "Initial commit"&#x000A;</code></pre>
<pre><code>git push origin master&#x000A;</code></pre>
<h3><strong>サービス アカウントの認証情報を追加する</strong></h3>
<p>Jenkins がコード リポジトリにアクセスできるように認証情報を構成します。Jenkins はクラスタのサービス アカウント認証情報を使用して、Cloud Source Repositories からコードをダウンロードします。</p>
<p><strong>手順 1</strong>: Jenkins のユーザー インターフェースで、左側のナビゲーションにある [<strong>Credentials</strong>] をクリックします。</p>
<p><strong>手順 2</strong>: [<strong>Jenkins</strong>] をクリックします。<img alt="stored_scoped_cred.png" src="https://cdn.qwiklabs.com/SiqLluBt531RSaLWnUZm28LRRVJCVBbINUZz7vaSyfo%3D"></p>
<p><strong>手順 3</strong>: [<strong>Global credentials (unrestricted)</strong>] をクリックします。</p>
<p><strong>手順 4</strong>: 左側のナビゲーションで [<strong>Add Credentials</strong>] をクリックします。</p>
<p><strong>手順 5</strong>: [<strong>Kind</strong>] プルダウンから [<strong>Google Service Account from metadata</strong>] を選択し、[<strong>OK</strong>] をクリックします。</p>
<p>これでグローバル認証情報が追加されました。認証情報の名前<code></code>は、ラボの「<code>接続の詳細</code>」にあるご自分の GCP プロジェクト ID です。</p>
<p><img alt="global_cred.png" src="https://cdn.qwiklabs.com/AADN9HYavaPKH3MPQMY8P9qhACY47HgPjAOuRLZz97M%3D"></p>
<p><strong>Jenkins ジョブを作成する</strong></p>
<p>Jenkins のユーザー インターフェースに移動し、次の手順でパイプライン ジョブを設定します。</p>
<p><strong>手順 1</strong>: 左側のナビゲーションで [<strong>Jenkins</strong>] &gt; [<strong>New Item</strong>] をクリックします。</p>
<p><img alt="86e92964a4599231.png" src="https://cdn.qwiklabs.com/KXB9gx8B77F27XAgu6bgXle0HvnoERwlPf9f2WYVqQM%3D"></p>
<p><strong>手順 2</strong>: プロジェクトに「<strong>sample-app</strong>」という名前を付けて [<strong>Multibranch Pipeline</strong>] オプションを選択し、[<strong>OK</strong>] をクリックします。</p>
<p><strong>手順 3</strong>: 次のページの [<strong>Branch Sources</strong>] のセクションで [<strong>Add Source</strong>] をクリックし、[<strong>git</strong>] を選択します。</p>
<p><strong>手順 4</strong>: Cloud Source Repositories 内にある sample-app リポジトリの <strong>HTTPS クローン URL</strong> を [<strong>Project Repository</strong>] 欄に貼り付けます。<code>[PROJECT_ID]</code> は実際の <strong>GCP プロジェクト ID</strong> に置き換えてください。</p>
<pre><code>https://source.developers.google.com/p/[PROJECT_ID]/r/default&#x000A;</code></pre>
<p><strong>手順 5</strong>: 前の手順でサービス アカウントの追加時に作成した認証情報の名前を、[<strong>Credentials</strong>] プルダウンから選択します。</p>
<p><strong>手順 6</strong>: [<strong>Scan Multibranch Pipeline Triggers</strong>] の [<strong>Periodically if not otherwise run</strong>] ボックスをオンにして、[<strong>Interval</strong>] の値を [1 minute] に設定します。</p>
<p><strong>手順 7</strong>: ジョブの構成は次のようになります。</p>
<p><img alt="general_tab_jenkins.png" src="https://cdn.qwiklabs.com/Eke%2BxaE%2BRpitZRG%2FFifw9jmNwAgCLMmnU21mdq8KDco%3D"></p>
<p><img alt="branch_source_jenkins.png" src="https://cdn.qwiklabs.com/Kp4KwWsjF4YOP1PHIs7i4OPV19GvBQMGqWkqG7UTpOg%3D"></p>
<p><img alt="build_conf_jenkins.png" src="https://cdn.qwiklabs.com/MmwLl5ZgWpBlrD%2B9%2Bfr97UhBrnVuqlpmMvmBofd49YM%3D"></p>
<p><strong>手順 8</strong>: 他の設定はすべてデフォルトのままにして [<strong>Save</strong>] をクリックします。</p>
<p>これらの手順を完了すると、「Branch indexing」という名前のジョブが実行されます。このメタジョブはリポジトリのブランチを認識し、既存のブランチで変更が行われていないことを確認します。左上の sample-app をクリックすると、マスタージョブが表示されます。</p>
<aside class="warning"><p><strong>注: </strong>次の手順でコードを変更しないと、マスタージョブの初回実行は失敗します。</p>
</aside>
<p>これで、Jenkins パイプラインの作成が完了しました。次に、継続的インテグレーションのための開発環境を作成します。</p>
<h2 id="step12">開発環境を作成する</h2>
<p>開発ブランチは、デベロッパーがコードの変更を公開サイトに統合する前にテストを行うための一連の環境です。これは実際の環境をスケールダウンしたものですが、ライブ環境と同じ方法でデプロイする必要があります。</p>
<h3>開発ブランチを作成する</h3>
<p>機能ブランチから開発環境を作成する場合は、ブランチを Git サーバーに push して、Jenkins で環境をデプロイできます。</p>
<p>開発ブランチを作成して、Git サーバーに push します。</p>
<pre><code>git checkout -b new-feature&#x000A;</code></pre>
<h3>パイプラインの定義を変更する</h3>
<p>パイプラインを定義する <code>Jenkinsfile</code> は <a href="https://jenkins.io/doc/book/pipeline/syntax/">Jenkins Pipeline Groovy 構文</a>で記述されます。<code>Jenkinsfile</code> を使用すると、ビルド パイプライン全体を 1 つのファイルで表し、ソースコードと一緒に公開できます。パイプラインは並列化などの機能をサポートしていますが、ユーザーによる承認が必要です。</p>
<p>パイプラインを想定どおりに機能させるには、<code>Jenkinsfile</code> ファイルを編集してプロジェクト ID を設定する必要があります。</p>
<p><code>vi</code> などのターミナル エディタで Jenkinsfile を開きます。</p>
<pre><code>vi Jenkinsfile&#x000A;</code></pre>
<p>エディタを起動します。</p>
<pre><code>i&#x000A;</code></pre>
<p><code>REPLACE_WITH_YOUR_PROJECT_ID</code> 値を実際の <code>PROJECT_ID</code> に置き換えます（<code>PROJECT_ID</code> はご自分の GCP プロジェクト ID で、ラボの「<code>接続の詳細</code>」に表示されます。<code>gcloud config get-value project</code> を実行して確認することもできます）。</p>
<pre><code>def project = 'REPLACE_WITH_YOUR_PROJECT_ID'&#x000A;def appName = 'gceme'&#x000A;def feSvcName = "${appName}-frontend"&#x000A;def imageTag = "gcr.io/${project}/${appName}:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"&#x000A;</code></pre>
<p><code>Jenkinsfile</code> ファイルを保存します（<code>vi</code> では <strong>Esc</strong> キーを押して以下を実行します）。</p>
<pre><code>:wq&#x000A;</code></pre>
<h3>サイトを変更する</h3>
<p>アプリケーションの変更を演習するために、gceme カードを<strong>青</strong>から<strong>オレンジ</strong>に変更します。</p>
<p><code>html.go:</code> を開きます。</p>
<pre><code>vi html.go&#x000A;</code></pre>
<p>エディタを起動します。</p>
<pre><code>i&#x000A;</code></pre>
<p>次のように <code>&lt;div class="card blue"&gt;</code> の 2 つのインスタンスを変更します。</p>
<pre><code>&lt;div class="card orange"&gt;&#x000A;</code></pre>
<p><code>html.go</code> ファイルを保存します（<strong>Esc</strong> キーを押して以下を実行します）。</p>
<pre><code>:wq&#x000A;</code></pre>
<p><code>main.go:</code> を開きます。</p>
<pre><code>vi main.go&#x000A;</code></pre>
<p>エディタを起動します。</p>
<pre><code>i&#x000A;</code></pre>
<p>バージョンが次の行で定義されています。</p>
<pre><code>const version string = "1.0.0"&#x000A;</code></pre>
<p>以下のように更新します。</p>
<pre><code>const version string = "2.0.0"&#x000A;</code></pre>
<p>main.go ファイルをもう一度保存します（<strong>Esc</strong> キーを押して以下を実行します）。</p>
<pre><code>:wq&#x000A;</code></pre>
<h2 id="step13">デプロイを開始する</h2>
<p>変更を commit して push します。</p>
<pre><code>git add Jenkinsfile html.go main.go&#x000A;</code></pre>
<pre><code>git commit -m "Version 2.0.0"&#x000A;</code></pre>
<pre><code>git push origin new-feature&#x000A;</code></pre>
<p>これにより、開発環境のビルドが開始されます。</p>
<p>変更が Git リポジトリに push されたら、Jenkins のユーザー インターフェースに移動し、new-feature ブランチのビルドが開始されていることを確認します。変更が反映されるまでに最長で 1 分ほどかかります。</p>
<p><img alt="757c7d394c91b309.png" src="https://cdn.qwiklabs.com/HxgBZDiCt3jzVvCYPfufV6y97cT8ggU4GXrC5D0qKZ8%3D"></p>
<p>ビルドの実行後、左側のナビゲーションのビルドの横にある下矢印をクリックして、[<strong>Console output</strong>] を選択します。</p>
<p><img alt="6ea3b2ed776e3b29.png" src="https://cdn.qwiklabs.com/3SL2TROBOgHPAAXRLhLCMEmyJ4h0LJGZAH2aNrLsToQ%3D"></p>
<p>ビルドの出力を数分間観察し、「<code>kubectl --namespace=new-feature apply...</code>」メッセージを確認します。これで、new-feature ブランチがクラスタにデプロイされます。</p>
<aside class="special"><p><strong>注:</strong> 開発では、外部に接続しているロードバランサは使用しません。アプリケーションを保護するには、<a href="http://kubernetes.io/docs/user-guide/connecting-to-applications-proxy/" target='blank"'>kubectl プロキシ</a>を使用します。プロキシは Kubernetes API で自らの認証を行い、ローカルマシンからのリクエストをクラスタ内のサービスにリダイレクトします。サービスがインターネットに公開されることはありません。</p>
</aside>
<p><code>Build Executor</code> に何も表示されなくても問題はありません。Jenkins ホームページ --&gt; sample-app に移動して、<code>new-feature</code> パイプラインが作成されていることを確認できます。</p>
<p>すべて完了したら、バックグラウンドでプロキシを開始します。</p>
<pre><code>kubectl proxy &amp;&#x000A;</code></pre>
<p>プロキシが停止した場合は、<strong>Ctrl+C</strong> キーを押して終了します。<code>localhost</code> にリクエストを送信して <code>kubectl</code> プロキシからサービスに転送し、アプリケーションがアクセス可能であることを確認します。</p>
<pre><code>curl \&#x000A;http://localhost:8001/api/v1/namespaces/new-feature/services/gceme-frontend:80/proxy/version&#x000A;</code></pre>
<p>現在実行中のバージョンである 2.0.0 が返されます。</p>
<p>次のようなエラーが表示された場合:</p>
<pre><code class="language-output__ prettyprint">{&#x000A;  "kind": "Status",&#x000A;  "apiVersion": "v1",&#x000A;  "metadata": {&#x000A;  },&#x000A;  "status": "Failure",&#x000A;  "message": "no endpoints available for service \"gceme-frontend:80\"",&#x000A;  "reason": "ServiceUnavailable",&#x000A;  "code": 503&#x000A;</code></pre>
<p>これは、frontend エンドポイントがまだ伝搬していないことを意味します。少し待ってから再度 <code>curl</code> コマンドを試してください。以下の出力が表示されたら次に進みます。</p>
<pre><code>2.0.0&#x000A;</code></pre>
<p>これで、開発環境の設定が完了しました。次は、前のモジュールで学んだことを踏まえ、カナリア リリースをデプロイして新しい機能をテストします。</p>
<h2 id="step14">カナリア リリースをデプロイする</h2>
<p>開発環境でアプリが最新のコードを実行することが確認できたので、今度はそのコードをカナリア環境にデプロイしてみましょう。</p>
<p>カナリア ブランチを作成して、Git サーバーに push します。</p>
<pre><code>git checkout -b canary&#x000A;</code></pre>
<pre><code>git push origin canary&#x000A;</code></pre>
<p>カナリア パイプラインが開始されたことが Jenkins に表示されます。完了したら、サービス URL を確認して、一部のトラフィックが新しいバージョンで処理されていることを確かめます。5 つのリクエストがあればそのうち約 1 つ（順序は不規則）でバージョン 2.0.0 が返されるはずです。</p>
<pre><code>export FRONTEND_SERVICE_IP=$(kubectl get -o \&#x000A;jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)&#x000A;</code></pre>
<pre><code>while true; do curl http://$FRONTEND_SERVICE_IP/version; sleep 1; done&#x000A;</code></pre>
<p>1.0.0 と表示される場合は、上記のコマンドをもう一度実行してください。上記の動作を確認したら、<strong>Ctrl+C</strong> キーでコマンドを終了します。</p>
<p>これでカナリア リリースのデプロイが完了しました。次は、新しいバージョンを本番環境にデプロイします。</p>
<h2 id="step15">本番環境にデプロイする</h2>
<p>カナリア リリースが成功し、お客様からも苦情が寄せられていないと想定して、本番環境へのデプロイを開始します。</p>
<p>カナリア ブランチを作成して、Git サーバーに push します。</p>
<pre><code>git checkout master&#x000A;</code></pre>
<pre><code>git merge canary&#x000A;</code></pre>
<pre><code>git push origin master&#x000A;</code></pre>
<p>マスター パイプラインが開始されたことが Jenkins に表示されます。<em></em>完了したら（数分かかる場合があります）、サービス URL を確認して、すべてのトラフィックが新しいバージョン 2.0.0 で処理されていることを確かめます。</p>
<pre><code>export FRONTEND_SERVICE_IP=$(kubectl get -o \&#x000A;jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)&#x000A;</code></pre>
<pre><code>while true; do curl http://$FRONTEND_SERVICE_IP/version; sleep 1; done&#x000A;</code></pre>
<p>1.0.0 のインスタンスが表示される場合は、上記のコマンドを再度実行してください。このコマンドを停止するには、<strong>Ctrl+C</strong> キーを押します。</p>
<p><strong>出力例:</strong></p>
<pre><code class="language-bash prettyprint">gcpstaging9854_student@qwiklabs-gcp-df93aba9e6ea114a:~/continuous-deployment-on-kubernetes/sample-app$ while true; do curl http://$FRONTEND_SERVICE_IP/version; sleep 1; done&#x000A;2.0.0&#x000A;2.0.0&#x000A;2.0.0&#x000A;2.0.0&#x000A;2.0.0&#x000A;2.0.0&#x000A;^C&#x000A;</code></pre>
<p>gceme アプリケーションの情報カードが表示されるサイトに移動することもできます。カードの色は青からオレンジに変わっています。外部 IP アドレスを再度取得して確認する場合は、次のコマンドを実行します。</p>
<pre><code>kubectl get service gceme-frontend -n production&#x000A;</code></pre>
<p><strong>出力例:</strong></p>
<p><img alt="orange_card.png" src="https://cdn.qwiklabs.com/nFJlHtZPI%2F8HvtCNSGRmCyqRkMDylftq%2F200xRXtUKc%3D"></p>
<p>本番環境にアプリケーションが正常にデプロイされました。</p>