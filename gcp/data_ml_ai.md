**Baseline: Data, ML, AI**

# AI Platform: Qwik Start
<h2 id="step2">概要</h2>
<p>このラボでは、ローカルおよび <a href="https://cloud.google.com/ml-engine/docs/">AI Platform</a> で <a href="http://tensorflow.org/">TensorFlow</a> モデルをトレーニングする方法について実践演習を行います。また、トレーニング終了後に、そのモデルを AI Platform にデプロイしてサービス（予測）を提供する方法について学びます。ここでは、<a href="https://archive.ics.uci.edu/ml/datasets/Census+Income">米国国勢調査所得データセット</a>を使用して任意の人物の所得階層を予測するモデルをトレーニングします。</p>
<p>このラボでは、AI Platform でのトレーニングと予測の初歩を網羅しており、国勢調査データセットを使用して次の作業を行います。</p>
<ul>
<li>TensorFlow トレーニング アプリケーションを作成し、それをローカルで検証します。</li>
<li>クラウドの単一ワーカー インスタンスでトレーニング ジョブを実行します。</li>
<li>クラウド内の分散トレーニング ジョブとしてトレーニング ジョブを実行します。</li>
<li>ハイパーパラメータ チューニングを使用して、ハイパーパラメータを最適化します。</li>
<li>予測をサポートするモデルをデプロイします。</li>
<li>オンライン予測をリクエストし、レスポンスを確認します。</li>
<li>バッチ予測をリクエストします。</li>
</ul>
<h3>作業内容</h3>
<p>このサンプルでは、米国国勢調査所得データセットに基づいて所得階層を予測するためにワイド＆ディープモデルを構築します。2 つの所得階層（ラベルとも呼ばれる）は次のとおりです。</p>
<ul>
<li>
<strong>&gt;50K</strong>: 50,000 ドル超</li>
<li>
<strong>&lt;=50K</strong>: 50,000 ドル以下</li>
</ul>
<p>ワイド＆ディープモデルでは、ディープ ニューラル ネット（DNN）を使用して、複雑な特徴や、それらの特徴間の相互作用に関して概要レベルの抽象化を学習します。次に、DNN からの出力を、比較的簡単な特徴に対して実行された線形回帰と結合します。これにより、多くの構造化データの問題に有効な、効果とスピードのバランスを実現できます。</p>
<p>このサンプルでは、TensorFlow の既製の <code>DNNCombinedLinearClassifier</code> クラスを使用してモデルを定義します。国勢調査データセットに特化したデータ変換を定義してから、これらの（場合によっては変換された）特徴をモデルの DNN または線形部分に割り当てます。</p>

<h2 id="step4">バーチャル環境を作成する</h2>
<p>次のコマンドを実行して、パッケージ リストをダウンロードし、更新します。</p>
<pre><code>sudo apt-get update</code></pre>
<p>システムからパッケージ インストレーションを分離するため、Python バーチャル環境を使用します。</p>
<pre><code>sudo apt-get install virtualenv</code></pre>
<ql-infobox>[Y/n] が表示されたら、<code>Y</code> と入力し、<code>Enter</code> キーを押します。</ql-infobox>
<pre><code>virtualenv -p python3 venv</code></pre>
<p>バーチャル環境をアクティブにします。</p>
<pre><code>source venv/bin/activate</code></pre>

<h2 id="step5">サンプル リポジトリのクローンの作成</h2>
<p>Cloud Shell で次のコマンドを実行して、<a href="https://github.com/GoogleCloudPlatform/cloudml-samples">cloudml-samples リポジトリ</a>のクローンを作成します。</p>
<pre><code>git clone https://github.com/GoogleCloudPlatform/cloudml-samples.git&#x000A;</code></pre>
<p><code>cloudml-samples</code> &gt; <code>census</code> &gt; <code>estimator</code> ディレクトリに移動します。このラボ内のコマンドは <code>estimator</code> ディレクトリから実行してください。</p>
<pre><code>cd cloudml-samples/census/estimator&#x000A;</code></pre>
<p>これで、現在のディレクトリは <code>~/cloudml-samples/census/estimator</code> になります。</p>
<p>使用するモデルはこのディレクトリに存在するため、ラボの完了まで常にこのディレクトリから作業を行います。</p>
<aside class="special"><strong>注:</strong><p>このラボを進めるうえで Python コードを確認する必要はありませんが、関心がある場合は Cloud Shell エディタでリポジトリ内のファイルをご覧ください。</p>
</aside>
<h2 id="step6">トレーニング アプリケーションをローカルで開発して検証する</h2>
<p>トレーニング アプリケーションをクラウドで実行する前に、まずローカルで実行します。ローカル環境では、効率的な開発と検証のワークフローを実現できるため、迅速な反復処理が可能です。また、アプリケーションをローカルでデバッグすれば、クラウド リソースの課金が発生することもありません。</p>
<h3>トレーニング データの入手</h3>
<p>ここで使用するデータファイルの <code>adult.data</code> と <code>adult.test</code> は、一般公開されている Google Cloud Storage バケットでホストされています。</p>
<p>これらのファイルは Cloud Storage から直接読み取ることも、ローカル環境にコピーすることもできます。このラボでは、まずローカル トレーニングのためにサンプルをダウンロードし、その後クラウド トレーニングのために自分の Cloud Storage バケットにアップロードします。</p>
<p>次のコマンドを実行して、データをローカルのファイル ディレクトリにダウンロードし、ダウンロードしたデータファイルを指す変数を設定します。</p>
<pre><code>mkdir data&#x000A;gsutil -m cp gs://cloud-samples-data/ml-engine/census/data/* data/&#x000A;</code></pre>
<p>さらに次のコマンドを実行して、<code>TRAIN_DATA</code> 変数と <code>EVAL_DATA</code> 変数をローカルのファイルパスに設定します。</p>
<pre><code>export TRAIN_DATA=$(pwd)/data/adult.data.csv&#x000A;export EVAL_DATA=$(pwd)/data/adult.test.csv&#x000A;</code></pre>
<p><code>adult.data.csv</code> ファイルを開くには、次のコマンドを実行します。</p>
<pre><code>head data/adult.data.csv&#x000A;</code></pre>
<p>データが次のようなコンマ区切り値の形式で格納されているのが確認できます。</p>
<pre><code class="language-bash prettyprint">39, State-gov, 77516, Bachelors, 13, Never-married, Adm-clerical, Not-in-family, White, Male, 2174, 0, 40, United-States, &lt;=50K&#x000A;50, Self-emp-not-inc, 83311, Bachelors, 13, Married-civ-spouse, Exec-managerial, Husband, White, Male, 0, 0, 13, United-States, &lt;=50K&#x000A;38, Private, 215646, HS-grad, 9, Divorced, Handlers-cleaners, Not-in-family, White, Male, 0, 0, 40, United-States, &lt;=50K&#x000A;53, Private, 234721, 11th, 7, Married-civ-spouse, Handlers-cleaners, Husband, Black, Male, 0, 0, 40, United-States, &lt;=50K&#x000A;...&#x000A;</code></pre>
<p>トレーニング データをダウンロードし、問題がないことを確認したので、次は必要な依存関係をインストールします。</p>
<h2 id="step7">依存関係のインストール</h2>
<p>TensorFlow は Cloud Shell にインストールされていますが、サンプルの <code>requirements.txt</code> ファイルを実行して、サンプルで必要な TensorFlow と同じバージョンを使用していることを確認する必要があります。</p>
<pre><code>pip install -r ../requirements.txt&#x000A;</code></pre>
<p>このコマンドが完了するまでに数分かかります。完了すると、次のような出力が返されます。</p>
<pre><code class="language-output prettyprint">Successfully installed Keras-2.3.1 absl-py-0.8.0 astor-0.8.0 future-0.16.0 gast-0.3.2 google-pasta-0.1.7 grpcio-1.24.1 h5py-2.10.0 keras-applications-1.0.8 keras-preprocessing-1.1.0 markdown-3.1.1 numexpr-2.7.0 numpy-1.17.2 pandas-0.25.1 protobuf-3.10.0 python-dateutil-2.8.0 pytz-2019.3 pyyaml-5.1.2 scipy-1.3.1 six-1.12.0 tensorboard-1.14.0 tensorflow-1.14.0 tensorflow-estimator-1.14.0 termcolor-1.1.0 werkzeug-0.16.0 wrapt-1.11.2.&#x000A;</code></pre>
<p>pandas には安定したバージョンを使用します(&lt; 0.25)</p>
<pre><code>pip install pandas==0.24.2</code></pre>
<p>以下のコマンドでインストールを確認します。</p>
<pre><code>python -c "import tensorflow as tf; print('TensorFlow version {} is installed.'.format(tf.__version__))"</code></pre>
<p>TensorFlowライブラリが特別のインストラクションを使用するようにコンパイルされていないので何か警告の場合は見逃せます。</p>
<h2 id="step8">ローカル トレーニング ジョブを実行する</h2>
<p>ローカル トレーニング ジョブは Python で記述されたトレーニング プログラムを読み込み、AI Platform でのライブのクラウド トレーニング ジョブと似た環境でトレーニング プロセスを開始します。</p>
<p>次のコマンドを実行して、出力ディレクトリを指定し、<code>MODEL_DIR</code> 変数を設定します。</p>
<pre><code>export MODEL_DIR=output&#x000A;</code></pre>
<p>次のコマンドを実行して、このトレーニング ジョブをローカルで実行します。</p>
<pre><code>gcloud ai-platform local train \&#x000A;    --module-name trainer.task \&#x000A;    --package-path trainer/ \&#x000A;    --job-dir $MODEL_DIR \&#x000A;    -- \&#x000A;    --train-files $TRAIN_DATA \&#x000A;    --eval-files $EVAL_DATA \&#x000A;    --train-steps 1000 \&#x000A;    --eval-steps 100&#x000A;</code></pre>
<aside class="special"><strong>注:</strong><p>後で同じトレーニング ジョブを AI Platform で実行しますが、そのときに使用するコマンドも上記と大きくは異なりません。</p>
</aside>
<p>デフォルトで、詳細ログはオフになっています。詳細ログを有効にするには、<code>--verbosity</code> タグを <code>DEBUG</code> に設定します。後のコマンド例で有効にする方法を示します。</p>
<h3>TensorBoard を使用した要約ログの検査</h3>
<p>評価結果を確認するには、<a href="https://www.tensorflow.org/programmers_guide/summaries_and_tensorboard">TensorBoard</a> という可視化ツールを使用します。TensorBoard を使用すると、TensorFlow グラフを可視化したり、グラフの実行に関する量的な指標をプロットしたり、グラフには表示されないイメージなどの追加データを表示したりできます。Tensorboard は、TensorFlow の一部としてインストールされます。</p>
<p>以下の手順に従って TensorBoard を起動し、トレーニング中（実行中および実行後の両方）に生成された概要ログを対象に指定します。</p>
<p>TensorBoard の起動:</p>
<pre><code>tensorboard --logdir=$MODEL_DIR --port=8080&#x000A;</code></pre>
<p><strong>ウェブでプレビュー</strong> アイコンをクリックし、[<strong>プレビューのポート: 8080</strong>] を選択します。TensorBoard が実行されると新しいタブが開きます。</p>
<p><img alt="bde9fe42e27656fb.png" src="https://cdn.qwiklabs.com/a6YnJv8GlGae4rnJIbjA27J8c7YApa%2B6noPFkkKxZjk%3D"></p>
<p>[<strong>Accuracy</strong>] をクリックすると、ジョブの進行状況に沿った精度の変化がグラフィカルに表示されます。</p>
<p>Cloud Shell で <code>Ctrl+C</code> キーを押し、TensorBoard を終了します。</p>
<p>output/export/census ディレクトリに、ローカルでトレーニングを行った結果としてエクスポートされたモデルが格納されています。このディレクトリをリスト表示して、生成されたタイムスタンプ サブディレクトリを確認します。</p>
<pre><code>ls output/export/census/&#x000A;</code></pre>
<p>次のような出力が返されます。</p>
<pre><code>1547669983&#x000A;</code></pre>
<p>生成されたタイムスタンプをコピーします。</p>
<p><code>&lt;timestamp&gt;</code> を自分のタイムスタンプに置き換えて、Cloud Shell で次のコマンドを実行します。</p>
<pre><code>gcloud ai-platform local predict \&#x000A;--model-dir output/export/census/&lt;timestamp&gt; \&#x000A;--json-instances ../test.json</code></pre>
<aside>注意：<strong>RuntimeError: Bad magic number in .pyc file</strong>を表示する場合に、次のコマンドを実行し、<code>sudo rm -rf /google/google-cloud-sdk/lib/googlecloudsdk/command_lib/ml_engine/*.pyc</code>、および上のコマンドをもう再実行します。</aside>
<p>これにより、次のような結果が得られます。</p>
<pre><code>CLASS_IDS  CLASSES  LOGISTIC              LOGITS                PROBABILITIES&#x000A;[0]        [u'0']   [0.0583675391972065]  [-2.780855178833008]  [0.9416324496269226, 0.0583675354719162]&#x000A;</code></pre>
<p>ここで、クラス 0 は所得 &lt;= 50k を意味し、クラス 1 は所得 &gt; 50k を意味します。</p>
<h2 id="step9">トレーニング済みモデルを使用した予測</h2>
<p>TensorFlow モデルのトレーニングが完了すると、そのモデルを新しいデータの予測に使用できます。この例では、国勢調査モデルのトレーニングが完了し、与えられた人物情報から所得階層を予測する準備が整っています。</p>
<p>このセクションでは、GitHub リポジトリの <code>census</code> ディレクトリにある定義済みの予測リクエスト ファイル（<code>test.json</code>）を使用します。詳細を確認したい場合は、Cloud Shell エディタで JSON ファイルを調べることができます。</p>
<h2 id="step10">クラウドでのトレーニング ジョブの実行</h2>
<p>ローカルで実行することによりモデルを検証できたため、次は AI Platform を使用したトレーニングの演習を行います。このセクションでも、Cloud Shell から作業を行います。</p>
<aside class="special"><p><strong>注: </strong>最初のジョブ リクエストは開始に数分かかりますが、その後のジョブはより短時間で実行されます。これにより、トレーニング ジョブを開発する際、ジョブを迅速に反復して検証できます。</p>
</aside>
<h3>Google Cloud Storage バケットのセットアップ</h3>
<p>AI Platform サービスは、モデルのトレーニングやバッチ予測の実行時に、データを読み書きするために <a href="https://cloud.google.com/storage/">Google Cloud Storage</a>（GCS）へのアクセスを必要とします。</p>
<p>まず、以下の変数を設定します。</p>
<pre><code>PROJECT_ID=$(gcloud config list project --format "value(core.project)")&#x000A;BUCKET_NAME=${PROJECT_ID}-mlengine&#x000A;echo $BUCKET_NAME&#x000A;REGION=us-central1&#x000A;</code></pre>
<p>新しいバケットを作成します。</p>
<pre><code>gsutil mb -l $REGION gs://$BUCKET_NAME&#x000A;</code></pre>

<p>データファイルを Cloud Storage バケットにアップロードします。</p>
<p><code>gsutil</code> を使用して、2 つのファイルを Cloud Storage バケットにコピーします。</p>
<pre><code>gsutil cp -r data gs://$BUCKET_NAME/data&#x000A;</code></pre>
<p>TRAIN_DATA 変数および EVAL_DATA 変数がファイルを指すように設定します。</p>
<pre><code>TRAIN_DATA=gs://$BUCKET_NAME/data/adult.data.csv&#x000A;EVAL_DATA=gs://$BUCKET_NAME/data/adult.test.csv&#x000A;</code></pre>
<p>再度 <code>gsutil</code> を使用して JSON テストファイル <code>test.json</code> を Cloud Storage バケットにコピーします。</p>
<pre><code>gsutil cp ../test.json gs://$BUCKET_NAME/data/test.json&#x000A;</code></pre>
<p>このファイルを指すように <code>TEST_JSON</code> 変数を設定します。</p>
<pre><code>TEST_JSON=gs://$BUCKET_NAME/data/test.json&#x000A;</code></pre>

<h2 id="step11">クラウド内で単一インスタンスのトレーナーを実行する</h2>
<p>単一インスタンスと分散モードの両方で動作する検証済みトレーニング ジョブを使用して、クラウド内でトレーニング ジョブを実行する準備が整いました。まず、単一インスタンスのトレーニング ジョブをリクエストします。</p>
<p>単一インスタンスのトレーニング ジョブを実行するには、デフォルトの BASIC スケール階層を使用します。最初のジョブ リクエストは開始に数分かかる場合がありますが、その後のジョブはより短時間で実行されます。これにより、トレーニング ジョブを開発する際、ジョブを迅速に反復して検証できます。</p>
<p>最初のトレーニングに、その後のトレーニングと区別できるような名前をつけます。たとえば、反復を表す番号を追加できます。</p>
<pre><code>JOB_NAME=census_single_1&#x000A;</code></pre>
<p>AI Platform によって生成された出力の保存先ディレクトリを指定します。これには、トレーニング ジョブと予測ジョブをリクエストする際に含める OUTPUT_PATH 変数を設定します。OUTPUT_PATH は、モデルのチェックポイント、概要、およびエクスポート用の Cloud Storage の場所を完全修飾パスで表します。前の手順で定義した BUCKET_NAME 変数を使用できます。ジョブ名を出力ディレクトリとして使用することをおすすめします。</p>
<pre><code>OUTPUT_PATH=gs://$BUCKET_NAME/$JOB_NAME&#x000A;</code></pre>
<p>以下のコマンドを実行して、クラウドで単一のプロセスを使用するトレーニング ジョブを送信します。ここでは、<code>--verbosity</code> タグを <code>DEBUG</code> に設定することにより、ロギング出力全体を検査して精度、損失、その他の指標を取得できるようにします。出力に含まれる他の多くの警告メッセージは、このサンプルでは無視してかまいません。</p>
<pre><code>gcloud ai-platform jobs submit training $JOB_NAME \&#x000A;    --job-dir $OUTPUT_PATH \&#x000A;    --runtime-version 1.14 \&#x000A;    --python-version 3.5 \&#x000A;    --module-name trainer.task \&#x000A;    --package-path trainer/ \&#x000A;    --region $REGION \&#x000A;    -- \&#x000A;    --train-files $TRAIN_DATA \&#x000A;    --eval-files $EVAL_DATA \&#x000A;    --train-steps 1000 \&#x000A;    --eval-steps 100 \&#x000A;    --verbosity DEBUG&#x000A;</code></pre>
<p>コマンドラインでログを監視することで、トレーニング ジョブの進行状況をモニタリングできます。</p>
<pre><code>gcloud ai-platform jobs stream-logs $JOB_NAME&#x000A;</code></pre>
<p>または、コンソールでモニタリングすることもできます: [<strong>AI プラットフォーム</strong>] &gt; [<strong>ジョブ</strong>]</p>

<h3>出力の検査</h3>
<p>Cloud Shell でジョブをモニタリングしている場合は、次の出力で完了が確認できます。</p>
<pre><code>INFO    2019-01-16 12:58:34 -0800     master-replica-0      Task completed successfully.&#x000A;</code></pre>
<p>クラウドでのトレーニングでは、出力は Cloud Storage 内に生成されます。このサンプルでは、出力は <code>OUTPUT_PATH</code> に保存されます。出力を一覧表示するには、次のコマンドを実行します。</p>
<pre><code>gsutil ls -r $OUTPUT_PATH&#x000A;</code></pre>
<p>ローカルでのトレーニングからの出力（前のセクション）に類似した出力が表示されます。</p>
<p><strong>オプション:</strong> トレーニング中またはトレーニング後に TensorBoard でこのディレクトリを指定できます。Cloud Shell の上部にある [<strong>ウェブでプレビュー</strong>] メニューを使用して、再度 [<strong>プレビューのポート: 8080</strong>] を選択します。</p>
<pre><code>tensorboard --logdir=$OUTPUT_PATH --port=8080&#x000A;</code></pre>
<p>TensorBoard を終了するには、コマンドラインで <strong>Ctrl</strong>+<strong>C</strong> キーを押します。</p>
<h2 id="step12">予測をサポートするモデルをデプロイする</h2>
<p>トレーニング済みモデルを AI Platform にデプロイしてオンライン予測リクエストを処理できるようにすると、スケーラブルにサービスを提供できるという利点があります。これは、トレーニング済みモデルで短期間に多くの予測リクエストを受ける場合に役立ちます。</p>
<p>AI Platform トレーニング ジョブが完了するまで待ちます。Cloud Console でジョブ名の横に緑色のチェックマークが表示されたら、または Cloud Shell コマンドラインに「<strong>Job completed successfully</strong>」というメッセージが表示されたらトレーニング ジョブは完了です。</p>
<p>AI Platform モデルを作成します。</p>
<pre><code>MODEL_NAME=census&#x000A;</code></pre>
<p>次のコマンドを実行します。</p>
<pre><code>gcloud ai-platform models create $MODEL_NAME --regions=$REGION&#x000A;</code></pre>

<p>エクスポートされたトレーニング済みモデルのバイナリのフルパスを参照して、使用するエクスポート済みモデルを選択します。</p>
<pre><code>gsutil ls -r $OUTPUT_PATH/export&#x000A;</code></pre>
<p>出力をスクロールして、<code>$OUTPUT_PATH/export/census/&lt;timestamp&gt;/</code> の値を探します。タイムスタンプをコピーして次のコマンドの timestamp の部分に貼り付け、環境変数 <code>MODEL_BINARIES</code> の値を設定します。</p>
<pre><code>MODEL_BINARIES=$OUTPUT_PATH/export/census/&lt;timestamp&gt;/&#x000A;</code></pre>
<aside class="special"><p><strong>注: </strong>このトレーニングのタイムスタンプは、前述のローカル トレーニング中に生成されたタイムスタンプとは異なります。必ず <code>gsutil ls</code> の出力をスクロールして、この新しいタイムスタンプを探してください。</p>
</aside>
<p>このトレーニング済みモデルをデプロイします。</p>
<p>次のコマンドを実行して、モデルのバージョン <code>v1</code> を作成します。</p>
<pre><code>gcloud ai-platform versions create v1 \&#x000A;--model $MODEL_NAME \&#x000A;--origin $MODEL_BINARIES \&#x000A;--runtime-version 1.14 \&#x000A;--python-version 3.5&#x000A;</code></pre>

<p>トレーニング済みモデルのデプロイには数分かかることがあります。完了したら、<code>models list</code> コマンドを使用してモデルの一覧を表示できます。</p>
<pre><code>gcloud ai-platform models list&#x000A;</code></pre>
<h3>デプロイしたモデルにオンライン予測リクエストを送信する</h3>
<p>これで、デプロイしたモデルに予測リクエストを送信できます。次のコマンドは、GitHub リポジトリに含まれる <code>test.json</code> ファイルを使用して予測リクエストを送信します。</p>
<pre><code>gcloud ai-platform predict \&#x000A;--model $MODEL_NAME \&#x000A;--version v1 \&#x000A;--json-instances ../test.json&#x000A;</code></pre>
<p>レスポンスには、test.json のデータエントリに基づく各ラベル（<strong>&gt;50K</strong> と <strong>&lt;=50K</strong>）の確率が含まれます。これは、予測される所得が 50,000 ドル超と 50,000 ドル以下のどちらであるかを示しています。</p>
<pre><code class="language-bash prettyprint">CLASS_IDS  CLASSES  LOGISTIC               LOGITS                PROBABILITIES&#x000A;[0]        [u'0']   [0.06142739951610565]  [-2.726504325866699]  [0.9385725855827332, 0.061427392065525055]&#x000A;</code></pre>
<p>ここで、クラス <code>0</code> は<em>所得 &lt;= 50k</em> を意味し、クラス <code>1</code> は<em>所得 &gt; 50k</em> を意味します。</p>
<aside class="special"><p><strong>注:</strong> AI Platform はバッチ予測もサポートしていますが、このラボでは説明しません。詳細については、バッチ予測に関するドキュメントをご覧ください。</p>
</aside>