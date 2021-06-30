# App Engine フレキシブル環境に Python Flask ウェブ アプリケーションをデプロイする

<h3><strong>概要</strong></h3>
<p>このラボでは、Python Flask ウェブ アプリケーションを App Engine フレキシブル環境にデプロイする方法について学びます。デプロイするアプリケーションはサンプル アプリケーションです。人の顔の写真をアップロードして、その人が楽しい気分である可能性を調べることができます。このアプリケーションは、Vision、Storage、Datastore の Google Cloud API を使用しています。</p>
<h3>App Engine について</h3>
<p>Google App Engine アプリケーションは作成や管理が簡単で、トラフィックやデータ ストレージの変動に合わせて容易にスケーリングできます。App Engine 環境では、サーバー管理の手間がかかりません。必要な作業は、アプリケーションをアップロードすることだけです。</p>
<p>Google App Engine のアプリケーションは、受信したトラフィック量に応じて自動的にスケールします。また、Google App Engine は負荷分散やマイクロサービス、認証、SQL および NoSQL データベース、トラフィック分散、ロギング、検索、バージョニング、ロールアウトとロールバック、セキュリティ スキャニングをネイティブでサポートしており、どれも高度にカスタマイズすることができます。</p>
<p>App Engine の<a href="https://cloud.google.com/appengine/docs/flexible/">フレキシブル環境</a>は、Java、Python、PHP、NodeJS、Ruby、Go など、数多くのプログラミング言語をサポートしています。Python などの一部の言語は、App Engine の<a href="https://cloud.google.com/appengine/docs/about-the-standard-environment">スタンダード環境</a>でもサポートされています。2 つの環境はそれぞれに固有の長所があるため、ユーザーは自身のアプリケーションの特徴に応じて柔軟に選ぶことができます。詳しくは、<a href="https://cloud.google.com/appengine/docs/the-appengine-environments">App Engine 環境の選択</a>をご確認ください。</p>
<h3><strong>ラボの内容</strong></h3>
<ul>
<li>App Engine フレキシブル環境にシンプルなウェブ アプリケーションをデプロイする方法</li>
<li>Vision、Storage、Datastore の Google Cloud クライアント ライブラリにアクセスする方法</li>
<li>Cloud Shell を使用する方法</li>
</ul>
<h3>前提条件</h3>
<ul>
<li>Python の基本知識</li>
<li>Linux の標準的なテキスト エディタ（vim、emacs、nano など）を使い慣れていること</li>
<li>人の顔が写っている写真</li>
</ul>

<h2 id="step4">サンプルコードを取得する</h2>
<p>Cloud Shell のコマンドラインで次のコマンドを実行して、GitHub リポジトリのクローンを作成します。</p>
<pre><code>git clone https://github.com/GoogleCloudPlatform/python-docs-samples.git&#x000A;</code></pre>
<p><code>python-docs-samples/codelabs/flex_and_vision</code> ディレクトリに移動します。</p>
<pre><code>cd python-docs-samples/codelabs/flex_and_vision&#x000A;</code></pre>
<h2 id="step5">API リクエストを認証する</h2>
<p>このラボでは、Datastore、Storage、Vision の各 API が自動的に有効になります。これらの API にリクエストを行うには、サービス アカウントの認証情報が必要です。この認証情報は、Cloud Shell で <em>gcloud</em> を使用してプロジェクトから生成することができます。プロジェクト ID は、このラボを開始した [Qwiklabs] タブで確認できます。</p>
<p><code>[YOUR_PROJECT_ID]</code> の環境変数を設定します。<code>[YOUR_PROJECT_ID]</code> を実際のプロジェクト ID に置き換えてください。</p>
<pre><code>export PROJECT_ID=[YOUR_PROJECT_ID]&#x000A;</code></pre>
<p>ローカルでのテストの際、Google Cloud API にアクセスするためのサービス アカウントを作成してください。</p>
<pre><code>gcloud iam service-accounts create qwiklab \&#x000A;  --display-name "My Qwiklab Service Account"&#x000A;</code></pre>
<p>新しく作成したサービス アカウントに適切な権限を付与します。</p>
<pre><code>gcloud projects add-iam-policy-binding ${PROJECT_ID} \&#x000A;--member serviceAccount:qwiklab@${PROJECT_ID}.iam.gserviceaccount.com \&#x000A;--role roles/owner&#x000A;</code></pre>
<p>サービス アカウントを作成したら、サービス アカウントキーを生成します。</p>
<pre><code>gcloud iam service-accounts keys create ~/key.json \&#x000A;--iam-account qwiklab@${PROJECT_ID}.iam.gserviceaccount.com&#x000A;</code></pre>
<p>このコマンドで、サービス アカウントキーが生成され、ホーム ディレクトリの <code>key.json</code> という名前の JSON ファイルに保存されます。</p>
<p>生成されたキーの絶対パスを使用して、サービス アカウントキーの環境変数を設定します。</p>
<pre><code>export GOOGLE_APPLICATION_CREDENTIALS="/home/${USER}/key.json"&#x000A;</code></pre>
<p>Vision API に対する認証の詳細については、<a href="https://cloud.google.com/vision/docs/common/auth">こちら</a>をご確認ください。</p>

<h2 id="step6">アプリケーションをローカルでテストする</h2>
<h3><strong>仮想環境を起動して依存関係をインストールする</strong></h3>
<p><a href="https://virtualenv.pypa.io/en/stable/">virtualenv</a> を使用して、<code>env</code> という名前の隔離された Python 3 環境を作成します。</p>
<pre><code>virtualenv -p python3 env&#x000A;</code></pre>
<p>新たに作成した <em>virtualenv</em> の <code>env</code> に入ります。</p>
<pre><code>source env/bin/activate&#x000A;</code></pre>
<p><code>pip</code> を使用して、プロジェクトの依存関係を <code>requirements.txt</code> ファイルからインストールします。</p>
<pre><code>pip install -r requirements.txt&#x000A;</code></pre>
<p><code>requirements.txt</code> ファイルは、このプロジェクトに必要なパッケージ依存関係のリストです。上のコマンドを実行すると、リストのすべてのパッケージ依存関係がこの <em>virtualenv</em> にダウンロードされます。</p>
<h3>App Engine アプリを作成する</h3>
<p>次に、下のコマンドを使用して App Engine インスタンスを作成します。</p>
<pre><code>gcloud app create&#x000A;</code></pre>
<p>リージョンのリストが表示されます。Python の App Engine フレキシブル環境をサポートしているリージョンを選択し、<strong>Enter</strong> キーを押します。リージョンとゾーンの詳細については、<a href="https://cloud.google.com/docs/geography-and-regions">こちら</a>をご確認ください。</p>
<h3><strong>ストレージ バケットを作成する</strong></h3>
<p>まず、環境変数 <em>CLOUD_STORAGE_BUCKET</em> を <em>PROJECT_ID</em> の名前に設定します（通常は利便性のため、<em>PROJECT_ID</em> と同じ名前をバケットに付けることをおすすめします）。</p>
<pre><code>export CLOUD_STORAGE_BUCKET=${PROJECT_ID}&#x000A;</code></pre>
<p>次に、下のコマンドを実行し、<em>PROJECT_ID</em> と同じ名前のバケットを作成します。</p>
<pre><code>gsutil mb gs://${PROJECT_ID}&#x000A;</code></pre>

<h3><strong>アプリケーションを実行する</strong></h3>
<p>次のコマンドを実行してアプリケーションを起動します。</p>
<pre><code>python main.py&#x000A;</code></pre>
<p>アプリケーションが起動したら、Cloud Shell ツールバーの「ウェブでプレビュー」アイコンをクリックして [プレビューのポート: 8080] を選択します。</p>
<p><img alt="web_prev_menu.png" src="https://cdn.qwiklabs.com/a6YnJv8GlGae4rnJIbjA27J8c7YApa%2B6noPFkkKxZjk%3D"></p>
<p>ブラウザのタブが開き、起動したサーバーに接続されます。次のように表示されます。</p>
<p><img alt="app_page.png" src="https://cdn.qwiklabs.com/3XEUm%2FJ1jhdalpSARYV57vCM70Cl3hpJ1QZkHqTOfhA%3D"></p>
<p>ここからさらに面白くなっていきます。[<strong>Choose File</strong>] ボタンをクリックし、人の顔が写っている写真をパソコンで選択して、[<strong>Submit</strong>] をクリックします。</p>
<p>写真をアップロードすると、次のように表示されます。</p>
<p><img alt="face_detection.png" src="https://cdn.qwiklabs.com/42S6HnaFnnUPxkY9p7TiBsfsJO15TrON45trEZOxppE%3D"></p>
<aside class="special"><p><strong>注:</strong> アプリケーションのローカルテストが完了したら、Cloud Shell コマンドラインで <strong>Ctrl+C</strong> キーを押してローカル ウェブサーバーをシャットダウンしてください。</p>
</aside>

<h2 id="step7">コードを確認する</h2>
<aside class="special"><strong>サンプル コード レイアウト</strong>
<p>このサンプルは次のようなレイアウトになっています。</p>
</aside>
<pre><code>templates/&#x000A;  homepage.html   /* Jinja2 を使用する HTML テンプレート */&#x000A;app.yaml          /* App Engine アプリケーション構成ファイル */&#x000A;main.py           /* Python Flask ウェブ アプリケーション */&#x000A;requirements.txt  /* プロジェクトの依存関係のリスト */&#x000A;</code></pre>
<aside class="special"><strong>main.py</strong>
<p>この Python ファイルは Flask ウェブ アプリケーションです。このアプリケーションでは、写真（できれば顔の写真）を送信すると、その写真が Cloud Storage に保存され、Cloud Vision API の顔検出機能を使用して分析されます。また、写真の主な情報が Datastore（Google Cloud Platform の NoSQL データベース）に保存されて、ユーザーがウェブサイトを訪れるたびに参照されます。</p>
<p>このアプリケーションは、Storage、Datastore、Vision の Google Cloud Platform クライアント ライブラリを使用しています。これらのクライアント ライブラリを使用すると、任意のプログラミング言語から簡単に Cloud API にアクセスできます。</p>
<p>コードの重要なスニペットについていくつか見てみましょう。</p>
<p>一番上の imports セクションは、コードに必要な各種パッケージをインポートする場所です。Datastore、Storage、Vision の Google Cloud クライアント ライブラリはこちらでインポートします。</p>
</aside>
<pre><code>from google.cloud import datastore&#x000A;from google.cloud import storage&#x000A;from google.cloud import vision&#x000A;</code></pre>
<aside class="special"><p>次のコードは、ユーザーがウェブサイトのルート URL にアクセスしたときに実行される処理を表しています。まず、Datastore クライアント ライブラリへのアクセスに使用される Datastore クライアント オブジェクトが作成されます。次に、Datastore に対するクエリが実行されて、種類が <em>Faces</em> のエンティティが取得されます。最後に、HTML テンプレートがレンダリングされて、Datastore から抽出した <em>image_entities</em> が変数として渡されます。</p>
</aside>
<pre><code>@app.route('/')&#x000A;def homepage():&#x000A;    # Cloud Datastore クライアントを作成します。&#x000A;    datastore_client = datastore.Client()&#x000A;&#x000A;    # 作成した Cloud Datastore クライアントを使用して、Datastore から&#x000A;    # 写真の情報を取得します。&#x000A;    query = datastore_client.query(kind='Faces')&#x000A;    image_entities = list(query.fetch())&#x000A;&#x000A;    # Jinja2 HTML テンプレートを返して、image_entities をパラメータとして渡します。&#x000A;    return render_template('homepage.html', image_entities=image_entities)&#x000A;</code></pre>
<aside class="special"><p>次に、<a href="https://cloud.google.com/datastore/docs/concepts/entities" target="blank">エンティティ</a>がどのように Datastore に保存されるかについて見ていきましょう。Datastore は、Google Cloud の NoSQL データベース ソリューションです。データは「エンティティ」<em></em>と呼ばれるオブジェクトに保存され、各エンティティに固有の識別「キー」<em></em>が割り当てられます。このキーは、「種類」<em></em>と「キー名」<em></em>の文字列を使用して作成します。種類<em></em>はエンティティ<em></em>を分類するためのもので、たとえば、写真、人、動物などの種類<em></em>を作ることができます。</p>
<p>各エンティティ<em></em>には、デベロッパーが定義する複数の「プロパティ」<em></em>を割り当てることができます。プロパティでは、整数、浮動小数点数、文字列、日付、バイナリデータなど、さまざまな型の値を使用できます。</p>
</aside>
<pre><code>    # Cloud Datastore クライアントを作成します。&#x000A;    datastore_client = datastore.Client()&#x000A;&#x000A;    # 現在の日時を取得します。&#x000A;    current_datetime = datetime.now()&#x000A;&#x000A;    # 新しいエンティティの種類。&#x000A;    kind = 'Faces'&#x000A;&#x000A;    # 新しいエンティティの名前 / ID。&#x000A;    name = blob.name&#x000A;&#x000A;    # 新しいエンティティの Cloud Datastore キーを作成します。&#x000A;    key = datastore_client.key(kind, name)&#x000A;&#x000A;    # このキーを使用して新しいエンティティを作成します。エンティティ キー blob_name、&#x000A;    # storage_public_url、timestamp、joy のディクショナリ値を設定します。&#x000A;    entity = datastore.Entity(key)&#x000A;    entity['blob_name'] = blob.name&#x000A;    entity['image_public_url'] = blob.public_url&#x000A;    entity['timestamp'] = current_datetime&#x000A;    entity['joy'] = face_joy&#x000A;&#x000A;    # この新しいエンティティを Datastore に保存します。&#x000A;    datastore_client.put(entity)&#x000A;</code></pre>
<aside class="special"><p>Storage と Vision のクライアント ライブラリにプログラムでアクセスする方法も Datastore と同様です。<em>vim</em>、<em>emacs</em>、<em>nano</em> を使用して <em>main.py</em> ファイルを開くと、すべてのサンプルコードを確認できます。</p>
<strong>homepage.html</strong>
<p>Flask ウェブ フレームワークでは、テンプレート エンジンに Jinja2 を使用しています。そのため、<em>main.py</em> から <em>homepage.html</em> に変数と式を渡して、ページがレンダリングされる際に値に置き換えることができます。</p>
<p>Jinja2 の詳細については、<a href="http://jinja.pocoo.org/docs/2.9/templates/" target="blank">http://jinja.pocoo.org/docs/2.9/templates/</a> をご確認ください。</p>
<p>次の Jinja2 HTML テンプレートは、データベースに写真を送信するためのフォームを表示します。このフォームには、以前に送信された写真も表示され、そのファイル名およびアップロード日時と、Vision API によって検出された顔の人物が楽しい気分である可能性が示されます。</p>
</aside>
<h4><strong>homepage.html</strong></h4>
<pre><code>&lt;h1&gt;Google Cloud Platform - Face Detection Sample&lt;/h1&gt;&#x000A;&#x000A;&lt;p&gt;This Python Flask application demonstrates App Engine Flexible, Google Cloud&#x000A;Storage, Datastore, and the Cloud Vision API.&lt;/p&gt;&#x000A;&#x000A;&lt;br&gt;&#x000A;&#x000A;&lt;html&gt;&#x000A;  &lt;body&gt;&#x000A;    &lt;form action="upload_photo" method="POST" enctype="multipart/form-data"&gt;&#x000A;      Upload File: &lt;input type="file" name="file"&gt;&lt;br&gt;&#x000A;      &lt;input type="submit" name="submit" value="Submit"&gt;&#x000A;    &lt;/form&gt;&#x000A;    {% for image_entity in image_entities %}&#x000A;      &lt;img src="{{image_entity['image_public_url']}}" width=200 height=200&gt;&#x000A;      &lt;p&gt;{{image_entity['blob_name']}} was uploaded {{image_entity['timestamp']}}.&lt;/p&gt;&#x000A;      &lt;p&gt;Joy Likelihood for Face: {{image_entity['joy']}}&lt;/p&gt;&#x000A;    {% endfor %}&#x000A;  &lt;/body&gt;&#x000A;&lt;/html&gt;&#x000A;</code></pre>
<h2 id="step8">アプリを App Engine フレキシブル環境にデプロイする</h2>
<p>App Engine フレキシブル環境では、アプリケーションのデプロイ構成が記述された <code>app.yaml</code> というファイルが使用されます。このファイルが存在しない場合、App Engine はデプロイ構成の推定を試みますが、このファイルを指定することをおすすめします。</p>
<p>次に、<em>vim</em>、<em>nano</em>、<em>emacs</em> などの任意のエディタを使用して <code>app.yaml</code> に変更を加えます。ここでは <code>nano</code> エディタを使用します。</p>
<pre><code>nano app.yaml&#x000A;</code></pre>
<p><code>app.yaml</code> を開いたら、<code>&lt;your-cloud-storage-bucket&gt;</code> を実際の Cloud Storage バケットの名前に置き換えます（Cloud Storage バケットの名前を忘れてしまった場合は、[Qwiklabs] タブから「GCP プロジェクト ID」<em></em>をコピーします）（<code>gs://</code> は不要で、バケット名だけを使う。<code>CLOUD_STORAGE_BUCKET: my-bucket-name</code> のような形になる）。
<code>env_variables</code> セクションでは、アプリケーションのデプロイ後に <code>main.py</code> で使用される環境変数を設定します。</p>
<pre><code>runtime: python&#x000A;env: flex&#x000A;entrypoint: gunicorn -b :$PORT main:app&#x000A;&#x000A;runtime_config:&#x000A;    python_version: 3&#x000A;&#x000A;env_variables:&#x000A;    CLOUD_STORAGE_BUCKET: &lt;your-cloud-storage-bucket&gt;&#x000A;</code></pre>
<p>これは、App Engine フレキシブル環境の Python 3 アプリケーションをデプロイするために必要な基本構成です。App Engine を構成する方法の詳細については、<a href="https://cloud.google.com/appengine/docs/flexible/python/configuring-your-app-with-app-yaml">こちら</a>をご確認ください。</p>
<p>完了したらファイルを保存して閉じます。<code>nano</code> の場合は <strong>Ctrl+X</strong> キーを押します。次のようなプロンプトが表示されます。</p>
<p><img alt="nano_changes.png" src="https://cdn.qwiklabs.com/iYcYjs1zRkiAt0bzVK5kJSapGihzmT6SHNQW1RqD1gU%3D"></p>
<p>文字 <strong>Y</strong> を入力し、次のプロンプトでもう一度 <strong>Enter<em></em></strong> キーを押してファイル名を確認します。</p>
<p><img alt="nano_confirm.png" src="https://cdn.qwiklabs.com/shsCi7GUrcNEQHX7F0HIPy8io0clqbxkf4RHUhtcrTo%3D"></p>

また、`requirements.txt` で以下のように指定する必要がある。

```
grpcio==1.27.2
setuptools==40.3.0
```

<p><code>gcloud</code> を使用してアプリを App Engine にデプロイします。</p>
<pre><code>gcloud app deploy&#x000A;</code></pre>
<p>Cloud Shell でアプリケーションがビルドされることを確認します。最大で 5 分ほどかかります。その間に、App Engine フレキシブル環境がバックグラウンドで Google Compute Engine 仮想マシンを自動的にプロビジョニングし、その後アプリケーションをインストールして起動します。</p>
<p><img alt="app_regions_output.png" src="https://cdn.qwiklabs.com/WnPxqnhBYUuXxJ1fmvPGjc1zy%2BfWANAT07E5FHoW7CQ%3D"></p>
<p>アプリケーションがデプロイされたら、次の URL を使用してウェブブラウザで開きます。</p>
<pre><code>https://&lt;PROJECT_ID&gt;.appspot.com&#x000A;</code></pre>
<aside class="special"><p><em>PROJECT_ID</em> を忘れてしまった場合は、Cloud Shell コマンドラインから <code>gcloud config list project</code> を実行します。</p>
</aside>