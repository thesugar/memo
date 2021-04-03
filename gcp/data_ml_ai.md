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


# Dataprep: Qwik Start

**Dataprocではないので注意**

<p><a href="https://cloud.google.com/dataprep/">Google Cloud Dataprep</a> は、分析用データの可視的な探索、クリーニング、準備を行うためのサーバーレスなインテリジェント データ サービスで、どんな規模でも稼働します。インフラストラクチャのデプロイや管理は必要ありません。コードも不要で、クリックするだけで簡単にデータを準備できます。</p>
<p>このラボでは Dataprep を使って、データセットのインポート、不一致データの修正、データの変換、データの結合を行います。初めてご利用の場合でも、ラボの終了時にはすべての操作を行えるようになります。</p>

<h2 id="step3">プロジェクトでの Cloud Storage バケットの作成</h2>
<ol>
<li>
<p>Cloud Platform で、<strong>ナビゲーション メニュー</strong> &gt; [<strong>Storage</strong>] &gt; [<strong>ブラウザ</strong>] の順に選択します。
<img alt="nav_storage.png" src="https://cdn.qwiklabs.com/OOFTVa9%2B3ahdzF6URCmTlQEFBPAOVXfNAn9vyAde3Jc%3D"></p>
</li>
<li>
<p>[<strong>バケットを作成</strong>] をクリックします。</p>
</li>
<li>
<p>[<strong>バケットを作成</strong>] ダイアログの [<strong>名前</strong>] でバケットに一意の名前を付けます。<a href="https://cloud.google.com/storage/docs/bucket-naming#requirements">バケット名の要件</a>を参照してください。その他の設定はすべてデフォルト値のままにします。</p>
<img alt="my-bucket.png" src="https://cdn.qwiklabs.com/gjo2TbsHELeXCzGV2p9a1RK5m8p5XdQ90wQz3yz5fF4%3D">
</li>
<aside>バケットの命名の詳細については、<a href="https://cloud.google.com/storage/docs/bucket-naming%5C#requirements" target="_blank">バケット名の要件</a>を参照してください</aside>
<li>
<p>[<strong>作成</strong>] をクリックします。</p>
</li>
</ol>
<p>バケットを作成できました。後の手順で使用するため、バケット名をメモしておきます。</p>

<h2 id="step4">Cloud Dataprep の初期化</h2>
<ol>
<li>
<strong>[ナビゲーション メニュー]</strong> &gt; [<strong>Dataprep</strong>] の順に選択します。</li>
<li>チェック ボックスをオンにして、Google Dataprep の利用規約に同意し、[<strong>同意する</strong>] をクリックします。</li>
<li>アカウント情報を Trifacta と共有することを承認するため、[<strong>同意して続行</strong>] をクリックします。</li>
<li>[<strong>許可</strong>] をクリックして、Trifacta がプロジェクトのデータにアクセスすることを許可します。</li>
<li>GCP のユーザー名をクリックして、Cloud Dataprep by Trifacta にログインします。GCP のユーザー名は、[Connection Details] パネルの [<strong>Username</strong>] です。</li>
<li>[<strong>許可</strong>] をクリックして、GCP ラボのアカウントに Cloud Dataprep へのアクセスを許可します。</li>
<li>チェックボックスをオンにし、[<strong>Accept</strong>] をクリックして Trifacta の利用規約に同意します。</li>
<li>[First time set up] 画面で [<strong>Continue</strong>] をクリックして、デフォルトのストレージの場所を作成します。</li>
</ol>
<p><img alt="937d6677b5e75d9d.png" src="https://cdn.qwiklabs.com/683VTyhfFDoqvOtciyH73it%2FEkDN5JW%2FemL9jXT%2Bqew%3D"></p>
<p>Dataprepが新しいブラウザータブで開きます。 ようこそページで、右上の <strong>Hide tour</strong> をクリックします。</p>

<h2 id="step5">フローの作成</h2>
<p>Cloud Dataprep では、<code>flow</code> ワークスペースを使用してデータセットにアクセスし、操作します。</p>
<ol>
<li>右上にある [<strong>Create Flow</strong>] をクリックします。</li>
</ol>
<p><img alt="dataprep_create_flow.png" src="https://cdn.qwiklabs.com/pPXJS3TfQZ5k9vXvmEugmFZV0jdPnNY%2F0IqJQzR7hfg%3D"></p>
<ol start="2">
<li>フローの名前と説明を入力します。このラボでは、<a href="https://www.fec.gov/data/browse-data/?tab=bulk-data">米国連邦選挙委員会 2016</a> のデータを使用しているため、[Flow Name] に「FEC-2016」、[Flow Description] に「米国連邦選挙委員会 2016」と入力します。</li>
</ol>
<p><img alt="656369ed7d46b2dc.png" src="https://cdn.qwiklabs.com/1JxXr%2BI%2B2XHwGjcZpgyUZVb28RkeNNLhlhfBRRYFEd8%3D"></p>
<ol start="3">
<li>[<strong>Create</strong>] をクリックします。</li>
</ol>
<p>FEC-2016 フローのページが表示されます。[What's a flow?] のスライドをスクロールすると、次の作業の概要を確認できます。[<strong>Don't show me any helpers</strong>] をクリックしてスキップすることも可能です。</p>
<h2 id="step6">データセットのインポート</h2>
<p>このセクションでは、データをインポートして、FEC-2016 フローに追加します。</p>
<ol>
<li>[<strong>Import &amp; Add Datasets</strong>] をクリックします。</li>
</ol>
<p><img alt="Dataprep_importandadd.png" src="https://cdn.qwiklabs.com/UHSReu92K7XpbpMDU1e7Rj7413ICk%2B8legdU12McZjE%3D"></p>
<ol start="2">
<li>左のメニューパネルで、Google Cloud Storage からデータセットをインポートするために [<strong>GCS</strong>] を選択します。次に、鉛筆のアイコンをクリックしてファイルのパスを編集します。</li>
</ol>
<p><img alt="dataprep_choose_file1.png" src="https://cdn.qwiklabs.com/TSeO9tpjFfoVja0DJjURWYWSZByIFAuz3j0uX8GvPnc%3D"></p>
<ol start="3">
<li>[<strong>Choose a file or folder</strong>] テキスト ボックスに「<code>gs://spls/gsp105</code>」と入力して、[<strong>Go</strong>] をクリックします。</li>
</ol>
<p>[<strong>Go</strong>] および [<strong>Cancel</strong>] のボタンが表示されるように、必要に応じてブラウザ ウィンドウを広げてください。</p>
<ol start="4">
<li>
<p>cn-2016.txt の横にある [<strong>+</strong>] アイコンをクリックすると、データセットが作成されて右側のパネルに表示されます。データセットのタイトルをクリックして、名前を「Candidate Master 2016」に変更します。</p>
</li>
<li>
<p>同様に、itcont-2016.txt のデータセットを追加し、名前を「Campaign Contributions 2016」に変更します。</p>
</li>
<li>
両方のデータセットが右側のペインに表示されたら、[<strong>Import &amp; Add to Flow</strong>] をクリックします。
</li>
</ol>
<p><img alt="4e126e9b671a7722.png" src="https://cdn.qwiklabs.com/dQ3zTrEHhwVZnUJTvV3U%2Bazq5YT15qPiuq0lf38bRAw%3D"></p>
<p>2 つのデータセットがフローとして表示されます。</p>
<p><img alt="FEC-2016.png" src="https://cdn.qwiklabs.com/aWqb85gyr84iT6ed629r4E4xnyr3wqECzVSLRvW4KYA%3D"></p>
<h2 id="step7">候補者ファイルの準備</h2>
<ol>
<li>デフォルトで、[Candidate Master 2016] データセットが選択されています。右側のパネルで、[<strong>Add New Recipe</strong>] をクリックします。</li>
</ol>
<p><img alt="dataprep_add_new_rec.png" src="https://cdn.qwiklabs.com/PhqJTmQAcBFK9TlzwKnIuktxlSTwvac6B476qDBUeJY%3D"></p>
<ol start="2">
<li>[<strong>Edit Recipe</strong>] をクリックします。</li>
</ol>
<p><img alt="dataprep_edit_rec.png" src="https://cdn.qwiklabs.com/lUQ2xOXyveFwtba3rszgh%2BDL61KrjmOYMMtKvbsWj6k%3D"></p>
<p>[Candidate Master 2016-2 Transformer] ページがグリッドビューで開きます。</p>
<p><img alt="dataprep_transformer.png" src="https://cdn.qwiklabs.com/i%2B7KE1QO7TyaaGGqrfQIH1B%2FWUIX1MMPSpGbPD5ora8%3D"></p>
<p>Transformer ページでは、変換レシピを作成して、サンプルに適用された結果を確認できます。表示内容に問題がなければ、データセットに対してジョブを実行します。</p>
<p>各列の見出しには、データ型が指定された名前と値があります。データ型は、旗のアイコンをクリックすると表示されます。</p>
<p><img alt="datatypes.png" src="https://cdn.qwiklabs.com/np%2B84KqotZFP2Vm9pfBSUhQ2QApRI3El5anSZ%2B02Rfs%3D"></p>
<p>フラグオプションをクリックすると、右側に[<strong>Details</strong>]パネルも開きます。</p>
<p><img alt="details_panel.png" src="https://cdn.qwiklabs.com/woQN8qYyLVpoB4xAf99Othuc7kz8t%2BeD2HfL%2BgbfL1Q%3D"></p>
<p>詳細パネルの右上にある[<strong>X</strong>] をクリックして、詳細パネルを閉じます。</p>
<p>次の手順では、グリッドビューでデータを探索し、レシピに変換手順を適用します。</p>
<ol>
<li>Column5 には、1990～2064 年のデータがあります。スプレッドシートで行うように [column5] の幅を広げると、年を細かく分けることができます。最も背の高いビン（2016 年）をクリックして選択します。</li>
</ol>
<p><img alt="Dataflow_column_5.png" src="https://cdn.qwiklabs.com/F1UOclmgih%2BHGec3LqFKr8VTzJuBQex3CZSbivabPC8%3D"></p>
<p>すると、これらの値が選択されたステップが作成されます。</p>
<ol start="2">
<li>右側の [<strong>Suggestions</strong>] パネルの [<strong>Keep rows</strong>] セクションで [<strong>Add</strong>] をクリックし、このステップをレシピに追加します。</li>
</ol>
<p><img alt="9a6a7c2c7888f3c6.png" src="https://cdn.qwiklabs.com/lplTJs%2BDH%2F8o82y2C%2FTFRQq7BhJwMkUpV2K62aU1pF4%3D"></p>
<p>右側のレシピパネルに次のステップが追加されます。</p>
<p><code>Keep rows where(date(2016, 1, 1) &lt;= column5) &amp;&amp; (column5 &lt; date(2018, 1, 1))</code></p>
<ol start="3">
<li>Column6（State）で、ヘッダーの不一致（赤色）部分にカーソルを合わせてクリックし、不一致の行を選択します。</li>
</ol>
<p><img alt="3cdb3803ef49636b.png" src="https://cdn.qwiklabs.com/J4An%2BqPwdGNzAdgSvOWakFOE%2FnrgJ%2B7ijW4SGJBch38%3D"></p>
<p>下にスクロールして不一致の値を見つけます。ほとんどのレコードで column7 には「P」、column6 には「US」という値があることがわかります。不一致が発生するのは、column6 が「State」列（フラグアイコンで示される）としてマークされている一方で、State 以外（「US」など）の値があるからです。</p>
<ol start="4">
<li>不一致を修正するには、提案パネルの上部にある[<strong>X</strong>]をクリックして変換をキャンセルし、Column6のフラグアイコンをクリックしてString列に変更します。</li>
</ol>
<p><img alt="84cfd42fcab33662.png" src="https://cdn.qwiklabs.com/lZtnUvD5I6KCDVJ4Zot5dmDC1KkektpgBlQgF7OO4pc%3D"></p>
<p>これで不一致がなくなり、列マーカーが緑色になります。</p>
<ol start="5">
<li>大統領候補者のみになるようにフィルタします。column7 の値が「P」のレコードが大統領候補者です。column7 のヒストグラムで 2 つのビンにカーソルを合わせ、どちらが「H」でどちらが「P」かを確認します。「P」のビンをクリックします。</li>
</ol>
<p><img alt="328626b128b93f1.png" src="https://cdn.qwiklabs.com/CIUuZWVWsWWQD3xJhQfJhm0qSoFcBK0cYUTpAnxpTbc%3D"></p>
<ol start="6">
<li>右側の提案パネルで、[<strong>Add</strong>] をクリックして、レシピへのステップを受け入れます。</li>
</ol>
<p><img alt="Dataprep_row_7" src="https://cdn.qwiklabs.com/vPIQ8e25%2FibBMpAF9J%2Bq9tWd%2FEqW1C6Tbj6xM0OlZAo%3D"></p>
<h2 id="step8">献金ファイルの結合</h2>
<p>結合ページでは、現在のデータセットを別のデータセットまたはレシピに追加できます。この処理は両方のデータセットに共通する情報に基づいて行われます。</p>
<p>献金のファイルを候補者のファイルと結合するには、まず候補者のファイルをクリーンアップします。</p>
<ol>
<li>グリッド ビュー ページの上部で [<strong>FEC-2016</strong>]（データセット セレクタ）をクリックします。</li>
</ol>
<p><img alt="dataprep_fec2016.png" src="https://cdn.qwiklabs.com/HjbiWzAPc2RlYFro3r4llT65MBJjJkcA%2FklDCnLHn%2BU%3D"></p>
<ol start="2">
<li>
<p>グレー表示の [<strong>Campaign Contributions</strong>] をクリックして選択します。</p>
</li>
<li>
<p>右のパネルで [<strong>Add New Recipe</strong>] をクリックし、[<strong>Edit Recipe</strong>] をクリックします。</p>
</li>
<li>
<p>ページの右上にある[<strong>recipe</strong>]アイコンをクリックし、[<strong>Add New Step</strong>] をクリックします。</p>
</li>
</ol>
<p><img alt="dataprep_2nd_recipe.png" src="https://cdn.qwiklabs.com/RSjdCHmcco6eqx2PxMao6QLRwsGdN17OuuKWuKScdJA%3D"></p>
<p>データセット内の余分な区切り文字を削除します。</p>
<ol>
<li>
<p>検索ボックスに次の Wrangle 言語のコマンドを入力します。</p>
</li>
</ol>
<pre><code>replacepatterns col: * with: '' on: `{start}"|"{end}` global: true&#x000A;</code></pre>
<p>Transformation Builder により Wrangle コマンドが解析され、検索と置換の変換フィールドにデータが入力されます。</p>
<p><img alt="recipe.png" src="https://cdn.qwiklabs.com/xpmu12D%2F%2Fa4JOxNT8%2FJVhSw4DAZH6m7tdBdNqRKoL4s%3D"></p>
<ol start="2">
<li>
<p>[<strong>Add</strong>] をクリックして、変換をレシピに追加します。</p>
</li>
<li>
<p>新しいステップをもう 1 つレシピに追加します。[<strong>New Step</strong>] をクリックし、検索ボックスに「Join」と入力します。</p>
</li>
</ol>
<p><img alt="Dataprep_join" src="https://cdn.qwiklabs.com/R0Fk6rYEUC7dlUnXULpsr4UXwyI38nKt9koliI%2BmVeo%3D"></p>
<ol start="4">
<li>
<p>[<strong>Join datasets</strong>] をクリックして結合のページを開きます。</p>
</li>
<li>
<p>[Candidate Master 2016-2] をクリックして [Campaign Contributions-2] と結合し、右下の [<strong>Accept</strong>] をクリックします。</p>
</li>
</ol>
<p><img alt="dataprep_candidate_master_2.png" src="https://cdn.qwiklabs.com/s3%2F9d3GqcQR0LKz%2FBa8VVU%2BqM62sZ430%2Fyf7c9lf2EE%3D"></p>
<ol start="6">
<li>結合キーのセクションにカーソルを移動し、鉛筆（編集のアイコン）をクリックします。</li>
</ol>
<p><img alt="edit_join.png" src="https://cdn.qwiklabs.com/H4yPICUZhn0v03sH8lME2m%2FOnPRm2EK28AYf1CqwWK0%3D"></p>
<p>Dataprep は共通のキーを推定します。Dataprep が結合キーとして提案する共通の値は多数あります。</p>
<ol start="7">
<li>[Add Key] パネルの [Suggested join keys] セクションで、[<strong>column2 = column11</strong>] をクリックします。</li>
</ol>
<p><img alt="join_conditions.png" src="https://cdn.qwiklabs.com/PVJawQJsD8zidQznUENkTaUpglQkmkZUoOe8n7dDMmE%3D"></p>
<ol start="8">
<li>[<strong>Save and Continue</strong>] をクリックします。</li>
</ol>
<p>Column2 と Column11 が確認用に開きます。</p>
<ol start="9">
<li>[<strong>Next</strong>] をクリックし、[Columns] ラベルの左側のチェック ボックスをオンにします。これにより、両方のデータセットのすべての列が、結合されるデータセットに追加されます。</li>
</ol>
<p><img alt="type_checkbox.png" src="https://cdn.qwiklabs.com/Y0yG7pQbqD9xQ%2FaaawSVMTCspoeJ3GlWrllws8dNzZ0%3D"></p>
<ol start="10">
<li>
<p><strong>Review</strong> をクリックし、<strong>Add to Recipe</strong> をクリックしてグリッドビューに戻ります。</p>
</li>
</ol>
<h2 id="step9">データのサマリー</h2>
<p>Column 16 の献金を集計、平均化、カウントし、Column 2、24、8 の ID、名前、所属政党別に候補者をグループ化して、有用なサマリーを生成します。</p>
<ol>
<li>
<p>[<strong>New Step</strong>] をクリックし、[<strong>Transformation</strong>] 検索ボックスに次の式を入力して、集計データをプレビューします。</p>
</li>
</ol>
<pre><code>pivot value:sum(column16),average(column16),countif(column16 &gt; 0) group: column2,column24,column8&#x000A;</code></pre>
<p>結合された集計データの初期サンプルが表示されます。これは、米国大統領候補と 2016 年の選挙献金指標のサマリー テーブルになっています。</p>
<p></p>
<p><img alt="6f4fba772aa0a141.png" src="https://cdn.qwiklabs.com/KWn%2BlaBdq7hmCkvWKzDVVGl0JYFKCEXuKEJ2GkfF7uA%3D"></p>
<ol start="2">
<li>[<strong>Add</strong>] をクリックして、主な米国大統領候補と、その 2016 年の選挙献金指標のサマリー テーブルを開きます。</li>
</ol>
<p><img alt="summary_table.png" src="https://cdn.qwiklabs.com/m0%2FLJXc5GImSno9sE25%2FlYcEx4sM0LfiY%2Bed8raUpnI%3D"></p>
<h2 id="step10">列名の変更</h2>
<p>列名を変更すると、データをさらに解析しやすくなります。そのためには、名前の変更と概数にする処理の各ステップをそれぞれレシピに追加します。[<strong>New Step</strong>] をクリックして、次のとおり入力します。</p>
<pre><code>rename type: manual mapping: [column24,'Candidate_Name'], [column2,'Candidate_ID'],[column8,'Party_Affiliation'], [sum_column16,'Total_Contribution_Sum'], [average_column16,'Average_Contribution_Sum'], [countif,'Number_of_Contributions']&#x000A;</code></pre>
<p>[<strong>Add</strong>] をクリックします。</p>
<p>平均献金額を概数にするには、次の行を [<strong>New Step</strong>] の末尾に貼り付けます。</p>
<pre><code>set col: Average_Contribution_Sum value: round(Average_Contribution_Sum)</code></pre>
<p>[<strong>Add</strong>] をクリックします。</p>
<p>結果は次のようになります。</p>
<p><img alt="2b3dc976f95952a5.png" src="https://cdn.qwiklabs.com/lTAt9SOx1C2HMSt%2B3TZqoAfdPqeDXiD9c2L4rzVPGs0%3D"></p>

# Dataflow: Qwik Start - テンプレート

<h2 id="step2">概要</h2>
<p>このラボでは、<a href="https://cloud.google.com/dataflow/docs/templates/provided-templates">Google の Cloud Dataflow テンプレート</a>の 1 つを使用してストリーミング パイプラインを作成する方法を学習します。具体的には、Cloud Pub/Sub to BigQuery テンプレートを使用します。このテンプレートは、Pub/Sub トピックから JSON 形式のメッセージを読み取り、BigQuery テーブルに push します。このテンプレートのドキュメントについては、<a href="https://cloud.google.com/dataflow/docs/templates/provided-templates#cloudpubsubtobigquery">こちら</a>をご覧ください。</p>
<p>BigQuery でデータセットとテーブルを作成するには、Cloud Shell コマンドラインまたは GCP Console を使用します。<strong>いずれかの方法を選択し</strong>、ラボの作業を進めてください。両方の方法を試したい場合は、このラボを 2 回行ってください。</p>

<h2 id="step4">Cloud Shell を使用して Cloud BigQuery データセットとテーブルを作成する</h2>
<p>まず BigQuery のデータセットとテーブルを作成します。</p>
<aside>
<strong>注: </strong>このセクションでは <code>bq</code> コマンドライン ツールを使用します。Console を使ってこのラボを行う場合は、<strong>スキップ</strong>してください。

</aside>
<p>次のコマンドを実行して、<code>taxirides</code> というデータセットを作成します。</p>
<pre><code>bq mk taxirides&#x000A;</code></pre>
<p>出力は次のようになります。</p>
<pre><code class="language-bash prettyprint">Dataset '&lt;myprojectid:taxirides&gt;' successfully created&#x000A;</code></pre>

<p>これでデータセットが作成されました。次の手順でこれを使用して、BigQuery テーブルをインスタンス化します。以下のコマンドを実行します。</p>
<pre><code>bq mk \&#x000A;--time_partitioning_field timestamp \&#x000A;--schema ride_id:string,point_idx:integer,latitude:float,longitude:float,\&#x000A;timestamp:timestamp,meter_reading:float,meter_increment:float,ride_status:string,\&#x000A;passenger_count:integer -t taxirides.realtime&#x000A;</code></pre>
<p>出力は次のようになります。</p>
<pre><code class="language-bash prettyprint">Table 'myprojectid:taxirides.realtime' successfully created&#x000A;</code></pre>

<p>一見すると、<code>bq mk</code> コマンドは少し複雑に見えます。しかし、<a href="https://cloud.google.com/bigquery/docs/reference/bq-cli-reference">BigQuery コマンドラインのドキュメントを参照すると</a>、全体像を把握できます。たとえば、<strong>スキーマ</strong>については次のような説明が記載されています。</p>
<ul>
<li>「ローカルの JSON スキーマ ファイルへのパス、または <code>[FIELD]</code>:<code>[DATA_TYPE]</code>, <code>[FIELD]</code>:<code>[DATA_TYPE]</code> 形式の列定義のカンマ区切りのリスト」。</li>
</ul>
<p>今回は、カンマ区切りのリスト（後者）を使用しています。</p>
<h3><strong>Storage バケットを作成する</strong></h3>
<p>テーブルがインスタンス化されたので、バケットを作成します。作成するには、次のコマンドを実行します。</p>
<pre><code>export BUCKET_NAME=&lt;your-unique-name&gt;&#x000A;</code></pre>
<pre><code>gsutil mb gs://$BUCKET_NAME/&#x000A;</code></pre>

<p>バケットを作成したら、<strong>「パイプラインを実行する」セクションまで下にスクロール</strong>します。</p>
<h2 id="step5">GCP Console を使用して Cloud BigQuery データセットとテーブルを作成する</h2>
<aside> <strong>注: </strong>コマンドラインによる設定が完了している場合は、このセクションの手順を実行しないでください。
</aside>
<p>左側のメニューの [ビッグデータ] セクションで [<strong>BigQuery</strong>] をクリックします。</p>
<p>左側のナビゲーションでプロジェクト名をクリックしてから、コンソールの右側にある [<strong>データセットを作成</strong>] をクリックします。データセット ID として「<code>taxirides</code>」を入力します。</p>
<p><img alt="dataset.png" src="https://cdn.qwiklabs.com/bRb1qVZ2z54AcpeDuFy%2B85MuvYJY4AVLbyNSjyGBITw%3D"></p>
<p>その他のデフォルト設定はすべてそのままにし、[<strong>データセットを作成</strong>] をクリックします。</p>

<p>左側のコンソールで、プロジェクト ID の下に taxirides データセットが表示されます。それをクリックして、コンソールの右側にある [<strong>テーブルを作成</strong>] を選択します。</p>
<p>[<strong>宛先テーブル</strong>] に「<code>realtime</code>」と入力します。</p>
<p>[スキーマ] で、[<strong>テキストとして編集</strong>] スライダーを切り替えて次のように入力します。</p>
<pre><code>ride_id:string,point_idx:integer,latitude:float,longitude:float,timestamp:timestamp,&#x000A;meter_reading:float,meter_increment:float,ride_status:string,passenger_count:integer&#x000A;</code></pre>

<p>コンソールは次のようになります。</p>
<p><img alt="new-table.png" src="https://cdn.qwiklabs.com/gYNzUZhtzX8OAdnoYoNY31oG6lZWz97RxhclS4pX9uM%3D"></p>
<p>次に、[<strong>テーブルを作成</strong>] をクリックします。</p>

<h3><strong>Storage バケットを作成する</strong></h3>
<p>GCP Console に戻り、[<strong>Storage</strong>] &gt; [<strong>ブラウザ</strong>] &gt; [<strong>バケットを作成</strong>] に移動します。</p>
<p><img alt="new-bucket.png" src="https://cdn.qwiklabs.com/kOdGE5oSj2FLlqRFgHSobVa8dbytB9uLsahqTsoSuYY%3D"></p>
<p>バケットに一意の名前を付けます。その他のデフォルト設定はそのままにして、[<strong>作成</strong>] をクリックします。</p>

<h2 id="step6">パイプラインを実行する</h2>
<p><strong>ナビゲーション メニュー</strong>の [ビッグデータ] セクションで、[<strong>Dataflow</strong>] をクリックします。</p>
<p>画面上部の [<strong>+ テンプレートからジョブを作成</strong>] をクリックします。</p>
<p>Cloud Dataflow ジョブの [<strong>ジョブ名</strong>] を入力します。</p>
<p>[<strong>Cloud Dataflow テンプレート</strong>] で、[Cloud Pub/Sub Topic to BigQuery]<em></em> テンプレートを選択します。</p>
<p>[<strong>Cloud Pub/Sub 入力トピック</strong>] に、次のように入力します。</p>
<pre><code>projects/pubsub-public-data/topics/taxirides-realtime&#x000A;</code></pre>
<p>[<strong>BigQuery 出力テーブル</strong>] に、作成されたテーブルの名前を入力します。</p>
<pre><code>&lt;myprojectid&gt;:taxirides.realtime&#x000A;</code></pre>
<p>バケットを [<strong>一時的なロケーション</strong>] として追加します。</p>
<pre><code>gs://Your_Bucket_Name/temp&#x000A;</code></pre>
<p><img alt="console.png" src="https://cdn.qwiklabs.com/kJiAXaVD5HH1IYxC0CWj%2BmR%2B9CqSFNJm3aDa%2Fm%2FEIco%3D"></p>
<p>[<strong>ジョブを実行</strong>] ボタンをクリックします。</p>

<p>リソースがビルドされ、使用できる状態になります。</p>
<p>それでは、ナビゲーション メニューにある [<strong>BigQuery</strong>] をクリックして、BigQuery に書き込まれたデータを見てみましょう。</p>
<p>BigQuery UI が開くと、プロジェクト名の下に <strong>taxirides</strong> テーブル、その下に <strong>realtime</strong> が表示されます。</p>
<p><img alt="bq-screenshot.png" src="https://cdn.qwiklabs.com/qO%2BbkqpwX5%2Begoc38sfCCc1Q3OmjzMYVg%2B8CQgmEnUA%3D"></p>
<h2 id="step7">クエリを送信する</h2>
<p>クエリは、標準 SQL を使用して送信できます。</p>
<p>[クエリエディタ] フィールドに以下を追加します。<em>myprojectid</em> は、Qwiklabs ページの GCP プロジェクト ID に置き換えます。</p>
<pre><code>SELECT * FROM `myprojectid.taxirides.realtime` LIMIT 1000&#x000A;</code></pre>
<p>[<strong>クエリを実行</strong>] をクリックします。</p>
<p>問題やエラーが発生した場合は、クエリを再実行してください（パイプラインの起動には数分かかります）。</p>
<p>クエリが正常に実行されると、次のように [クエリ結果] パネルに出力が表示されます。</p>
<p><img alt="query-results.png" src="https://cdn.qwiklabs.com/JIACp2MGHfBDfVaUOojawE0nqwdx18el4zjgmqABSuc%3D"></p>
<p>これで Pub/Sub トピックから 1,000 件のタクシー乗車データが pull され、BigQuery テーブルに push されました。実際に確認したように、テンプレートは Dataflow ジョブを実行するのに実用的で使いやすい方法です。<a href="https://cloud.google.com/dataflow/docs/templates/provided-templates">こちら</a>から、他の Google テンプレートを必ず確認するようにしてください。</p>

# Dataproc: Qwik Start - Console

<h2 id="step2">概要</h2>
<p>Cloud Dataproc は、<a href="http://spark.apache.org/">Apache Spark</a> および <a href="http://hadoop.apache.org/">Apache Hadoop</a> クラスタをより簡単かつ低コストで実行できるようにする、高速で使いやすいフルマネージド クラウド サービスです。このサービスでは、これまで数時間から数日かかっていたオペレーションを数秒から数分で処理できます。Cloud Dataproc クラスタは迅速に作成できるうえ、いつでもサイズ変更が可能です。このため、データ パイプラインの成長にクラスタが追いつかないことを心配する必要はありません。</p>

→　Dataproc はユーザーが大量のデータを処理、変換、理解することに役立つ。

<p>このラボでは、Google Cloud Platform（GCP）Console を使用して Google Cloud Dataproc クラスタを作成し、クラスタで簡単な <a href="http://spark.apache.org/">Apache Spark</a> ジョブを実行してクラスタのワーカーの数を変更する方法を学びます。</p>

<h3><strong>Cloud Dataproc API が有効であること</strong>を確認</h3>
<p>GCP で Dataproc クラスタを作成するには、Cloud Dataproc API を有効にする必要があります。API が有効になっていることを確認するには:</p>
<p><strong>ナビゲーション メニュー</strong> &gt; [<strong>API とサービス</strong>] &gt; [<strong>ライブラリ</strong>] をクリックします。</p>
<p><img alt="nav_to_library.png" src="https://cdn.qwiklabs.com/FLkG9hzTo3TKxjdyHxdzFcp0Ig2LyAbMW%2Blku5xArp8%3D"></p>
<p>[<strong>API とサービスを検索</strong>] ダイアログに「<strong>Cloud Dataproc</strong>」と入力します。検索結果としてコンソールに Cloud Dataproc API が表示されます。</p>
<p><strong>Cloud Dataproc API</strong> をクリックすると、API のステータスが表示されます。API がまだ有効になっていない場合は、[<strong>有効にする</strong>] をクリックします。</p>
<p>API が有効になっている場合は、次の手順に移ります。</p>
<p><img alt="api.png" src="https://cdn.qwiklabs.com/7Ajj25TBn7XLmDweXY458W77pFyOXYDkFpgceNNo5rk%3D"></p>
<h2 id="step4">クラスタの作成</h2>
<p>Cloud Platform Console で<strong>ナビゲーション メニュー</strong> &gt; [<strong>Dataproc</strong>] &gt; [<strong>クラスタ</strong>] を選択し、[<strong>クラスタの作成</strong>] をクリックします。</p>
<p>クラスタの各項目を以下のように設定します。これ以外の項目はすべてデフォルト値のままにします。</p>
<table>

<tr>
<th>項目</th>
<th>値</th>
</tr>


<tr>
<td>名前</td>
<td>example-cluster</td>
</tr>
<tr>
<td>リージョン</td>
<td>global</td>
</tr>
<tr>
<td>ゾーン</td>
<td>us-central1-a</td>
</tr>

</table>
<aside>
<strong>注:</strong> <em>ゾーン</em>は特別なマルチリージョンの名前空間であり、すべての Google Compute ゾーンに対してグローバルにインスタンスをデプロイできます。また、個別のリージョン（<code>us-east1</code> や <code>europe-west1</code> など）を複数指定することで、Cloud Dataproc によって利用されるリソース（VM インスタンスや Google Cloud Storage など）やメタデータの保存場所をリージョンごとに分離することもできます。

</aside>
<p><img alt="ccc5b8f862ec3a4f.png" src="https://cdn.qwiklabs.com/nj3Y5D%2BthYWN%2Bw%2BulCuLwrS%2BWAs5faDSDR%2BKLN5NuUo%3D"></p>
<p>[<strong>作成</strong>] をクリックしてクラスタを作成します。</p>
<p>新しいクラスタが [クラスタ] リストに表示されます。作成には数分かかる場合があります。クラスタが使用できるようになるまで、ステータスには [<strong>プロビジョニング</strong>] と表示され、その後 [<strong>実行中</strong>] に変わります。</p>

## コマンドラインでクラスタを作成する場合

```bash
gcloud config set dataproc/region global
```

```bash
gcloud dataproc clusters create example-cluster
```

<h2 id="step5">ジョブの送信</h2>
<p>サンプルの Spark ジョブを実行するには:</p>
<p>左側のナビゲーション メニューで [<strong>ジョブ</strong>] を選択して Dataproc ジョブの表示に切り替え、[<strong>ジョブの送信</strong>] をクリックします。</p>
<p><img alt="fe78cb5282f3f914.png" src="https://cdn.qwiklabs.com/10Y1a%2B4f6tEAFIqZuxfwNze2jCayn7ZW%2FZqPMn2sDms%3D"></p>
<p>ジョブを更新するために以下の項目を設定します。これ以外の項目はすべてデフォルト値のままにします。</p>
<table>

<tr>
<th>項目</th>
<th>値</th>
</tr>


<tr>
<td>クラスタ</td>
<td>example-cluster</td>
</tr>
<tr>
<td>ジョブタイプ</td>
<td>Spark</td>
</tr>
<tr>
<td>メインクラスまたは JAR</td>
<td>org.apache.spark.examples.SparkPi</td>
</tr>
<tr>
<td>引数</td>
<td>1000（タスクの数を設定します）</td>
</tr>
<tr>
<td>JAR ファイル</td>
<td>file:///usr/lib/spark/examples/jars/spark-examples.jar</td>
</tr>

</table>
<p><img alt="66a806709011b870.png" src="https://cdn.qwiklabs.com/UOaUkpetQkKD%2BLM8Rw6KvLmmxLtD5zJNkHu6Jj%2BCgfk%3D"></p>
<p>[<strong>送信</strong>] をクリックします。</p>
<aside class="special"><p><strong>ジョブによる Pi の計算方法:</strong> Spark ジョブは、<a href="https://en.wikipedia.org/wiki/Monte_Carlo_method" target="_blank">モンテカルロ法</a>を使用して Pi の値を推定します。これによって、単位正方形で囲まれた円をモデル化した座標平面上に x、y 点が生成されます。入力引数（1000）は、生成する x と y のペア数を決定します。生成するペアが多いほど、推定の精度が向上します。この推定では、Cloud Dataproc ワーカーノードを利用して計算が並列化されます。詳細については、<a href="https://academo.org/demos/estimating-pi-monte-carlo/" target="_blank">モンテカルロ法を使用した Pi の推定</a>と、<a href="https://github.com/apache/spark/blob/master/examples/src/main/java/org/apache/spark/examples/JavaSparkPi.java" target="_blank">GitHub の JavaSparkPi.java</a> をご覧ください。</p>
</aside>
<p>ジョブが [<strong>ジョブ</strong>] リストに表示されます。このリストには、プロジェクトのジョブがクラスタ、タイプ、現在のステータスとともに表示されます。ジョブ ステータスは [<strong>実行中</strong>] と表示され、その後ジョブが完了すると [<strong>完了</strong>] になります。</p>
<p><img alt="9985555bb3c1543.png" src="https://cdn.qwiklabs.com/T5Qtm5IbKDxZAAOGCu2acHOg1d%2Bkn%2FcLg%2FE6T7DAnbY%3D"></p>

## コマンドラインでジョブを送信する場合

```bash
gcloud dataproc jobs submit spark --cluster example-cluster \
  --class org.apache.spark.examples.SparkPi \
  --jars file:///usr/lib/spark/examples/jars/spark-examples.jar -- 1000
```

このコマンドでは、以下が指定されています。

- Spark ジョブを example-cluster クラスタで実行すること
- ジョブの Pi 計算アプリケーションの main メソッドが含まれる class
- ジョブのコードが含まれる jar ファイルの場所
- ジョブに渡すパラメータ。このケースでは、タスクの個数（1000）

<h2 id="step6">ジョブ出力の確認</h2>
<p>完了したジョブの出力を表示する手順は次のとおりです。</p>
<p>[<strong>ジョブ</strong>] リストのジョブ ID をクリックします。</p>
<p>[<strong>行の折り返し</strong>] をオンにするか、Pi の計算値まで右にスクロールします。[<strong>行の折り返し</strong>] をオンにすると、次のように表示されます。</p>
<p><img alt="output.png" src="https://cdn.qwiklabs.com/zoDB6bN%2FfzWp9P7ltO26xgW%2FanpJ2LScnaHtbttd0tU%3D"></p>
<p>Pi のおおよその値が正しく計算されました。</p>
<h3>クラスタの更新</h3>
<p>クラスタのワーカー インスタンスの数を変更するには:</p>
<ol>
<li>左側のナビゲーション ペインで [<strong>クラスタ</strong>] を選択し、Dataproc クラスタの表示に戻ります。</li>
<li>[<strong>クラスタ</strong>] リストで [<strong>example-cluster</strong>] をクリックします。デフォルトでは、クラスタの CPU 使用率の概要が表示されます。</li>
<li>[<strong>設定</strong>] をクリックし、クラスタの現在の設定を表示します。</li>
</ol>
<p><img alt="ec14bf04e8f43d88.png" src="https://cdn.qwiklabs.com/Fvipbc9ytyszFNKtzc3tLtVHdWy1Ya3kOzmvLA1lMWs%3D"></p>
<ol start="4">
<li>[<strong>編集</strong>] をクリックします。ここでワーカーノードの数を編集できます。</li>
<li>[<strong>ワーカーノード</strong>] に、「<strong>4</strong>」と入力します。</li>
<li>[<strong>保存</strong>] をクリックします。</li>
</ol>
<p><img alt="944887f76f36e447.png" src="https://cdn.qwiklabs.com/HbuwmChVD%2FKHAFx7XyZK1KCkCyDkpytcSjcQjVszFM4%3D"></p>
<p>クラスタが更新されました。クラスタの VM インスタンスの数を確認してください。</p>
<p><img alt="vm-instances.png" src="https://cdn.qwiklabs.com/odOIZ7betFCnE4NmF5XfYPog0%2FkAbeHeAZbOjifAcSA%3D"></p>

<p>更新されたクラスタのジョブを再度実行するには、左側のナビゲーション メニューで [<strong>ジョブ</strong>] をクリックしてから [<strong>ジョブの送信</strong>] をクリックします。</p>
<p>[<strong>ジョブの送信</strong>] で設定した項目と同じ項目を設定します。</p>
<table>

<tr>
<th>項目</th>
<th>値</th>
</tr>


<tr>
<td>クラスタ</td>
<td>example-cluster</td>
</tr>
<tr>
<td>ジョブタイプ</td>
<td>Spark</td>
</tr>
<tr>
<td>メインクラスまたは JAR</td>
<td>org.apache.spark.examples.SparkPi</td>
</tr>
<tr>
<td>引数</td>
<td>1000（タスクの数を設定します）</td>
</tr>
<tr>
<td>JAR ファイル</td>
<td>file:///usr/lib/spark/examples/jars/spark-examples.jar
</td>
</tr>

</table>
<p><img alt="66a806709011b870.png" src="https://cdn.qwiklabs.com/UOaUkpetQkKD%2BLM8Rw6KvLmmxLtD5zJNkHu6Jj%2BCgfk%3D"></p>
<p>[<strong>送信</strong>] をクリックします。</p>

## コマンドラインでクラスタを更新する場合

```bash
gcloud dataproc clusters update example-cluster --num-workers 4
```

# 強化学習: Qwik Start
<h2 id="step2">概要</h2>
<h3>はじめに</h3>
<p>機械学習に関する研究の多くの分野と同様に、<a href="https://ja.wikipedia.org/wiki/%E5%BC%B7%E5%8C%96%E5%AD%A6%E7%BF%92">強化学習（RL: Reinforcement Learning）</a>は、猛烈なスピードで進歩しています。他の研究分野もそうですが、研究者たちはディープ ラーニングを活用して最先端の成果を生み出しています。</p>
<p>特に強化学習は、ゲーム分野で従来の機械学習テクニックの性能を大幅に上回り、Atari ゲームでは人間のレベルに追いつくどころか、世界最高水準にまで達しました。囲碁では人間のチャンピオンに勝ち、Starcraft II のような難易度の高いゲームでも今後が期待される結果を出しています。</p>
<p>このラボでは、<a href="https://gym.openai.com/">OpenAI Gym</a> が提供するサンプルからモデル化された、簡単なゲームを構築することによって強化学習の基礎を学びます。</p>
<h3>目標</h3>
<p>このラボでは、次の作業を行います。</p>
<ul>
<li>強化学習の基本的なコンセプトについて理解する。</li>
<li>AI Platform Tensorflow 2.1 Notebook を作成する。</li>
<li>GitHub にある training data analyst リポジトリからサンプル リポジトリのクローンを作成する。</li>
<li>ノートブックの手順を読み、理解し、実行する。</li>
</ul>

<h2 id="step4">強化学習 101</h2>
<p>強化学習は機械学習の形態の 1 つであり、エージェントが環境に対する行動を選択しながら、その一連の選択を通じて得られる目標（報酬）を最大化する方法を学習していくというものです。従来の教師あり学習のテクニックとは異なり、データポイントはすべてがラベル付けされるというわけではなく、エージェントは「疎」な報酬にアクセスできるだけです。</p>
<p><a href="http://www.incompleteideas.net/book/ebook/node12.html">強化学習の歴史</a>は 1950 年代にまでさかのぼることができます。そのアルゴリズムは数多く存在しますが、最近では、簡単に実装できる強力な深層強化学習アルゴリズム、DQN（Deep Q-Network）と DDPG（Deep Deterministic Policy Gradient）の 2 つが注目されています。このセクションでは、アルゴリズムとその変種について簡単に紹介します。</p>
<p><img alt="8c84b4dbb56d882e.png" src="https://cdn.qwiklabs.com/cDBDy0wLYFlwkAnG0PrdbCg7UAEngRYH%2BORdWseL14A%3D"></p>
<p>強化学習のプロセス概念図<em></em></p>
<p>DQN は、Google DeepMind グループが 2015 年に <a href="https://storage.googleapis.com/deepmind-media/dqn/DQNNaturePaper.pdf">Nature の論文</a>で発表したアルゴリズムです。論文の著者らは、画像認識分野でのディープ ラーニングの成功を励みに、ディープ ニューラル ネットワークを Q 学習に組み込み、観測空間が非常に高次元な <a href="https://gym.openai.com/envs/#atari">Atari Game Engine Simulator</a> でアルゴリズムをテストしました。</p>
<p>ディープ ニューラル ネットワークは、特定の入力状態に基づいて、出力 Q 値、すなわちある行動を取ることがどの程度望ましいかを予測する関数近似器として機能します。つまり、DQN は価値ベースのアルゴリズムです。DQN はトレーニング アルゴリズムの中でベルマン方程式に従い Q 値を更新していきますが、動くターゲットに合わせる難しさを避けるために、ターゲットの値を予測する、第 2 のディープ ニューラル ネットワークを使います。</p>
<p>より実用的なレベルとして、次のモデルでは、GCP で実行されている強化学習ジョブを取得するために、ソースファイル、シェルコマンド、エンドポイントを強調表示しています。</p>
<p><img alt="62687b03fa144178.png" src="https://cdn.qwiklabs.com/FQvwxiTxO%2FJ5baJVEDsj0tKHG1hvn27YHmaa0FHFbS4%3D"></p>
<h2 id="step5">AI Platform Notebook を作成する</h2>
<p>このラボに必要なすべてのファイルは、この<a href="https://github.com/GoogleCloudPlatform/training-data-analyst/pull/745">リポジトリ</a>にあります。これらすべてのコマンドを実行するために、AI Platform Tensorflow Notebook を作成します。</p>
<p>左側のナビゲーション メニューで、[<strong>AI Platform</strong>] &gt; [<strong>ノートブック</strong>] の順に選択します。上部のメニューで、[<strong>+ 新しいインスタンス</strong>] &gt; [<strong>Tensorflow 2.1</strong>] &gt; [<strong>Without GPUs</strong>] の順に選択します。</p>
<p><img alt="5b41852a0a7cc19b.png" src="https://cdn.qwiklabs.com/SMSqSEVSsQ9PrCIKKpdb1bZ30cqeytJeh4gj5KWoCGY%3D"></p>
<p>次に [<strong>作成</strong>] をクリックします。数分で AI Platform Notebook のプロビジョニングが完了します。ページを適宜更新してください。ノートブックがビルドされたら、[<strong>JupyterLab を開く</strong>] ボタンをクリックします。</p>
<p><img alt="5fd20811e4ca5ec7.png" src="https://cdn.qwiklabs.com/1gDxdk8TVF0S0RL9VA8isFHv7DODEipQ0Of6b%2FPuaX0%3D"></p>
<p>JupyterLab が読み込まれた新しいタブが開きます。</p>

<h2 id="step6">サンプルコードのクローンを作成する</h2>
<p>[<strong>ターミナル</strong>] アイコンをクリックします。コマンドを入力するための一時的なシェルが用意されます。次のコマンドを入力して、training data analyst リポジトリからサンプル レポジトリのクローンを作成します。</p>
<pre><code>git clone https://github.com/GoogleCloudPlatform/training-data-analyst.git&#x000A;</code></pre>
<p>このコマンドが反映されるまで待ちます。左側のメニューで、[<strong>training-data-analyst</strong>] &gt; [<strong>quests</strong>] &gt; [<strong>rl</strong>] &gt; [<strong>early_rl</strong>] &gt; [<strong>early_rl.ipynb</strong>] の順に選択します。新しいタブが開きます。</p>

<h2 id="step7">ノートブックを実行する</h2>
<p>新しいタブは次のようになります。</p>
<p><img alt="6827f6f1a1eae75.png" src="https://cdn.qwiklabs.com/eIiugfg275%2BwduGfjCmVnCwb%2B6FnsIZDN4I16yT0wCY%3D"></p>
<p>次のノートブックを読み、<strong>Shift + Enter</strong> を押して、すべてのコードブロックを実行します。</p>

# Cloud Composer （Airflow）: Qwik Start - Console

<h2 id="step2">概要</h2>
<p>ワークフローはデータ分析における一般的なテーマのひとつで、データの取り込み、変換、分析によってデータから有益な情報を見つけるために使用されます。Google Cloud Platform（GCP）には、ワークフローをホストするためのツールとして Cloud Composer が用意されています。これは、よく利用されているオープンソース ワークフロー ツールの Apache Airflow をホスト型にしたものです。</p>
<p>このラボでは、GCP Console を使用して Cloud Composer 環境を作成し、Cloud Composer を使ってシンプルなワークフローを実行します。このワークフローは、データファイルが存在することを確認し、Cloud Dataproc クラスタを作成して Apache Hadoop ワードカウント ジョブを実行した後、Cloud Dataproc クラスタを削除します。</p>
<h3>演習内容</h3>
<ul>
<li>
<p>GCP Console を使用して Cloud Composer 環境を作成する</p>
</li>
<li>
<p>Airflow ウェブ インターフェースで DAG（有向非巡回グラフ）を表示して実行する</p>
</li>
<li>
<p>保存されたワードカウント ジョブの結果を表示する</p>
</li>
</ul>

<h2 id="step4">Cloud Composer 環境を作成する</h2>
<p>このセクションでは、Cloud Composer 環境を作成します。</p>
<ol>
<li>
<strong>ナビゲーション メニュー</strong> &gt; [<strong>Composer</strong>] に移動します。</li>
</ol>
<p><img alt="6d5cc1e126272384.png" src="https://cdn.qwiklabs.com/xM%2Bvs%2B5wRKY0WR6BrAN8XQ2nvFiHEgLN7kltTL9BzZI%3D"></p>
<ol start="2">
<li>[<strong>環境の作成</strong>] をクリックし、次の環境設定を指定します。</li>
</ol>
<p><strong>名前:</strong> highcpu</p>
<p><strong>場所:</strong> us-central1</p>
<p><strong>ゾーン:</strong> us-central1-a</p>
<p><strong>マシンタイプ:</strong> n1-highcpu-4</p>
<p><strong>Python バージョン:</strong> 3</p>
<p>その他の設定はすべてデフォルトのままにします。</p>
<ol start="3">
<li>[<strong>作成</strong>] をクリックします。</li>
</ol>
<p>GCP コンソールの [環境] ページで環境の名前の左側に緑色のチェックマークが表示されれば、環境作成プロセスは完了しています。</p>
<p>設定プロセスが完了するまで 10～15 分かかることがあります。その間にラボの作業を進めてください。</p>
<h3>Cloud Storage バケットを作成する</h3>
<p>プロジェクトの Cloud Storage バケットを作成します。このバケットは、Dataproc の Hadoop ジョブの出力に使用されます。</p>
<ol>
<li>
<p><strong>ナビゲーション メニュー</strong> &gt; [<strong>Storage</strong>] &gt; [<strong>ブラウザ</strong>] に移動して [<strong>バケットを作成</strong>] をクリックします。</p>
</li>
<li>
<p>バケットにユニバーサルに一意な名前を付けてから、[<strong>作成</strong>] をクリックします。</p>
</li>
</ol>
<p>この Cloud Storage バケット名は後ほど Airflow 変数として使用するため、覚えておいてください。</p>
<p>[<strong>進行状況を確認</strong>] をクリックして、目標に沿って進んでいることを確認します。</p>

<h2 id="step5">Airflow と主要なコンセプト</h2>
<p>Composer 環境が作成されるまでの間に、Airflow で使用される用語を確認しましょう。</p>
<p><a href="https://airflow.apache.org/">Airflow</a> とは、ワークフローの作成、スケジューリング、モニタリングをプログラマティックに行うためのプラットフォームです。</p>
<p>Airflow を使用して、ワークフローをタスクの有向非循環グラフ（DAG）として作成します。Airflow スケジューラは、指定された依存関係に従って一連のワーカーでタスクを実行します。</p>
<h3>基本コンセプト</h3>
<p><a href="https://airflow.apache.org/concepts.html#dags">DAG</a></p>
<p>有向非循環グラフ（DAG）とは、実行したいすべてのタスクの集まりであり、それらの関係や依存状態を反映するように編成されます。</p>
<p><a href="https://airflow.apache.org/concepts.html#operators">オペレーター</a></p>
<p>単一のタスクを記述したもので、通常はアトミックなタスクです。たとえば、BashOperator<em></em> は bash コマンドの実行に使用されます。</p>
<p><a href="https://airflow.apache.org/concepts.html#tasks">タスク</a></p>
<p>オペレーターのパラメータ化されたインスタンスであり、DAG 内のノードです。</p>
<p><a href="https://airflow.apache.org/concepts.html#task-instances">タスク インスタンス</a></p>
<p>特定のタスクの実行です。DAG、タスク、ある時点を表し、実行中<em></em>、成功<em></em>、失敗<em></em>、スキップ<em></em>などのステータスを示します。</p>
<p>その他のコンセプトについては、<a href="https://airflow.apache.org/concepts.html#">こちら</a>をご覧ください。</p>
<h2 id="step6">ワークフローの定義</h2>
<p>これから使用するワークフローについて説明します。Cloud Composer ワークフローは <a href="https://airflow.incubator.apache.org/concepts.html#dags">DAG（Directed Acyclic Graph、有向非循環グラフ）</a>で構成されます。DAG の定義には標準 Python ファイルを使用し、これらのファイルは Airflow の <code>DAG_FOLDER</code> に配置されます。Airflow では各ファイル内のコードを実行して動的に <code>DAG</code> オブジェクトをビルドします。任意の数のタスクを記述した DAG を必要なだけ作成できます。一般に、DAG と論理ワークフローは 1 対 1 で対応している必要があります。</p>
<p>次のコードは <a href="https://github.com/GoogleCloudPlatform/python-docs-samples/blob/master/composer/workflows/hadoop_tutorial.py" target="_blank">hadoop_tutorial.py</a> ワークフロー（DAG）です。</p>

```python
# Copyright 2018 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# [START composer_hadoop_tutorial]
"""Example Airflow DAG that creates a Cloud Dataproc cluster, runs the Hadoop
wordcount example, and deletes the cluster.

This DAG relies on three Airflow variables
https://airflow.apache.org/concepts.html#variables
* gcp_project - Google Cloud Project to use for the Cloud Dataproc cluster.
* gce_zone - Google Compute Engine zone where Cloud Dataproc cluster should be
  created.
* gcs_bucket - Google Cloud Storage bucket to use for result of Hadoop job.
  See https://cloud.google.com/storage/docs/creating-buckets for creating a
  bucket.
"""

import datetime
import os

from airflow import models
from airflow.contrib.operators import dataproc_operator
from airflow.utils import trigger_rule

# Output file for Cloud Dataproc job.
output_file = os.path.join(
    models.Variable.get('gcs_bucket'), 'wordcount',
    datetime.datetime.now().strftime('%Y%m%d-%H%M%S')) + os.sep
# Path to Hadoop wordcount example available on every Dataproc cluster.
WORDCOUNT_JAR = (
    'file:///usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar'
)
# Arguments to pass to Cloud Dataproc job.
input_file = 'gs://pub/shakespeare/rose.txt'
wordcount_args = ['wordcount', input_file, output_file]

yesterday = datetime.datetime.combine(
    datetime.datetime.today() - datetime.timedelta(1),
    datetime.datetime.min.time())

default_dag_args = {
    # Setting start date as yesterday starts the DAG immediately when it is
    # detected in the Cloud Storage bucket.
    'start_date': yesterday,
    # To email on failure or retry set 'email' arg to your email and enable
    # emailing here.
    'email_on_failure': False,
    'email_on_retry': False,
    # If a task fails, retry it once after waiting at least 5 minutes
    'retries': 1,
    'retry_delay': datetime.timedelta(minutes=5),
    'project_id': models.Variable.get('gcp_project')
}

# [START composer_hadoop_schedule]
with models.DAG(
        'composer_hadoop_tutorial',
        # Continue to run DAG once per day
        schedule_interval=datetime.timedelta(days=1),
        default_args=default_dag_args) as dag:
    # [END composer_hadoop_schedule]

    # Create a Cloud Dataproc cluster.
    create_dataproc_cluster = dataproc_operator.DataprocClusterCreateOperator(
        task_id='create_dataproc_cluster',
        # Give the cluster a unique name by appending the date scheduled.
        # See https://airflow.apache.org/code.html#default-variables
        cluster_name='composer-hadoop-tutorial-cluster-{{ ds_nodash }}',
        num_workers=2,
        zone=models.Variable.get('gce_zone'),
        master_machine_type='n1-standard-1',
        worker_machine_type='n1-standard-1')

    # Run the Hadoop wordcount example installed on the Cloud Dataproc cluster
    # master node.
    run_dataproc_hadoop = dataproc_operator.DataProcHadoopOperator(
        task_id='run_dataproc_hadoop',
        main_jar=WORDCOUNT_JAR,
        cluster_name='composer-hadoop-tutorial-cluster-{{ ds_nodash }}',
        arguments=wordcount_args)

    # Delete Cloud Dataproc cluster.
    delete_dataproc_cluster = dataproc_operator.DataprocClusterDeleteOperator(
        task_id='delete_dataproc_cluster',
        cluster_name='composer-hadoop-tutorial-cluster-{{ ds_nodash }}',
        # Setting trigger_rule to ALL_DONE causes the cluster to be deleted
        # even if the Dataproc job fails.
        trigger_rule=trigger_rule.TriggerRule.ALL_DONE)

    # [START composer_hadoop_steps]
    # Define DAG dependencies.
    create_dataproc_cluster >> run_dataproc_hadoop >> delete_dataproc_cluster
    # [END composer_hadoop_steps]

# [END composer_hadoop]
```

<p>この DAG では、3 つのワークフロー タスクをオーケストレートするために次のオペレーターをインポートしています。</p>
<ol>
<li>
<code>DataprocClusterCreateOperator</code>: Cloud Dataproc クラスタを作成します。</li>
<li>
<code>DataProcHadoopOperator</code>: Hadoop ワードカウント ジョブを送信し、結果を Cloud Storage バケットに書き込みます。</li>
<li>
<code>DataprocClusterDeleteOperator</code>: クラスタを削除して、Compute Engine の利用料金が発生しないようにします。</li>
</ol>
<p>タスクは順番に実行されます。ファイルの次の部分で確認できます。</p>
<pre><code># DAG の依存関係を定義します&#x000A;create_dataproc_cluster &gt;&gt; run_dataproc_hadoop &gt;&gt; delete_dataproc_cluster&#x000A;</code></pre>
<p>この DAG の名前は <code>hadoop_tutorial</code> で、1 日 1 回実行されます。</p>
<pre><code>with models.DAG(&#x000A;        'composer_hadoop_tutorial',&#x000A;        # DAG を 1 日に 1 回、継続して実行します&#x000A;        schedule_interval=datetime.timedelta(days=1),&#x000A;        default_args=default_dag_args) as dag:&#x000A;</code></pre>
<p><code>default_dag_args</code> に渡される <code>start_date</code> は <code>yesterday</code> に設定されているので、Cloud Composer ではワークフローの開始が DAG のアップロード直後に設定されます。</p>
<h2 id="step7">環境情報の表示</h2>
<ol>
<li>
<p><strong>Composer</strong> に戻り、環境のステータスを確認します。</p>
</li>
<li>
<p>環境が作成されたら、環境の名前（highcpu）をクリックして詳細を確認します。</p>
</li>
</ol>
<p>[<strong>環境の詳細</strong>] で、Airflow ウェブ インターフェース URL、Kubernetes Engine クラスタ ID、バケットに保存されている DAG フォルダへのリンクなどの情報を確認できます。</p>
<aside class="special"><p><strong>注:</strong> Cloud Composer がスケジュール設定を行うのは、<code>/dags</code> フォルダ内のワークフローのみです。</p>
</aside>
<p>[<strong>進行状況を確認</strong>] をクリックして、目標に沿って進んでいることを確認します。</p>

<h2 id="step8">Airflow UI の使用</h2>
<p>GCP Console で Airflow ウェブ インターフェースにアクセスするには:</p>
<ol>
<li>
<p>[<strong>環境</strong>] ページに戻ります。</p>
</li>
<li>
<p>環境の [<strong>Airflow ウェブサーバー</strong>] 列で、[<strong>Airflow</strong>] をクリックします。</p>
</li>
<li>
<p>ラボの認証情報をクリックします。</p>
</li>
<li>
<p>新しいウィンドウに Airflow ウェブ インターフェースが表示されます。</p>
</li>
</ol>
<h2 id="step9">Airflow 変数の設定</h2>
<p>Airflow 変数は Airflow 固有の概念であり、<a href="https://cloud.google.com/composer/docs/how-to/managing/environment-variables">環境変数</a>とは異なります。</p>
<ol>
<li>Airflow メニューバーで [<strong>Admin</strong>] &gt; [<strong>Variables</strong>] を選択し、[<strong>Create</strong>] をクリックします。</li>
</ol>
<p><img alt="4a38ba78af97a898.png" src="https://cdn.qwiklabs.com/O8uLOjzf0%2BIexBYTTlXK1SeU%2BX2yEiX02jjOzwRSx9s%3D"></p>
<ol start="2">
<li>
<code>gcp_project</code>、<code>gcs_bucket</code>、<code>gce_zone</code> の 3 つの Airflow 変数を作成します。</li>
</ol>
<table>
<tr>
<td colspan="1" rowspan="1">
<p><strong>キー</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>値</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>詳細</strong></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p><code>gcp_project</code></p>
</td>
<td colspan="1" rowspan="1">
<p>プロジェクト ID</p>
</td>
<td colspan="1" rowspan="1">
<p>この Qwik Start で使用している Google Cloud Platform プロジェクト。</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p><code>gcs_bucket</code></p>
</td>
<td colspan="1" rowspan="1">
<p>gs://</p>
</td>
<td colspan="1" rowspan="1">
<p> は、先ほど作成した Cloud Storage バケットの名前に置き換えます。このバケットに Dataproc の Hadoop ジョブの出力が保存されます。</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p><code>gce_zone</code></p>
</td>
<td colspan="1" rowspan="1">
<p>us-central1-a</p>
</td>
<td colspan="1" rowspan="1">
<p>Cloud Dataproc クラスタを作成する Compute Engine ゾーン。別のゾーンを選択するには、<a href="https://cloud.google.com/compute/docs/regions-zones/regions-zones#available" target="_blank">使用可能なリージョンとゾーン</a>をご覧ください。</p>
</td>
</tr>
</table>
<p>完了すると、Variables のテーブルは次のようになります。</p>
<p><img alt="6069c7a4f191b5a3.png" src="https://cdn.qwiklabs.com/1YBb%2F%2F98z%2Fc1FLI6JFa0AHmt1UC3ngg8cpXSZazsR%2F0%3D"></p>
<h2 id="step10">DAG の Cloud Storage へのアップロード</h2>
<p>DAG をアップロードするには:</p>
<ol>
<li>
<p>Cloud Shell で、hadoop_tutorial.py をコピーしてローカル仮想マシンに保存します。</p>
</li>
</ol>
<pre><code>git clone https://github.com/GoogleCloudPlatform/python-docs-samples&#x000A;</code></pre>
<ol start="2">
<li>
<p><code>python-docs-samples</code> ディレクトリに移動します。</p>
</li>
</ol>
<pre><code class="language-bash prettyprint">cd python-docs-samples/composer/workflows&#x000A;</code></pre>
<ol start="3">
<li>
<p>次に <code>hadoop_tutorial.py</code> ファイルのコピーを Cloud Storage バケットにアップロードします。このバケットは、環境を作成したときに自動的に作成されています。これを確認するには、[<strong>Composer</strong>] &gt; [<strong>環境</strong>] に移動して、先ほど作成した環境をクリックします。作成した環境の説明が表示されます。[<code>DAG のフォルダ</code>] を見つけて値をコピーし、次のコマンドの <code>DAGs_folder_path</code> をその値に置き換えて、ファイルをアップロードします。</p>
</li>
</ol>
<pre><code>gsutil cp hadoop_tutorial.py DAGs_folder_path&#x000A;</code></pre>
<p>Cloud Composer により、DAG が Airflow に追加されて自動的にスケジュール設定されます。DAG の変更が 3～5 分で行われます。これ以降、このワークフローを <code>composer_hadoop_tutorial</code> と呼びます。</p>
<p>このタスクのステータスは、Airflow ウェブ インターフェースで確認できます。</p>

<h3>DAG の実行状況の確認</h3>
<p>DAG ファイルを Cloud Storage の <code>dags</code> フォルダにアップロードすると、ファイルが Cloud Composer によって解析されます。エラーが見つからなければ、このワークフローの名前が DAG のリストに表示され、即時実行されるようキューに登録されます。</p>
<p>Airflow ウェブ インターフェースの [DAGs] タブが表示されていることを確認してください。このプロセスが完了するまで数分かかります。ブラウザの画面を更新して、最新情報を表示してください。</p>
<p><img alt="802e34f53ff823f7.png" src="https://cdn.qwiklabs.com/gEFZWSXpeYO%2Fu8otUagVB%2Bg9DVzkN1eF2sepg9DErEk%3D"></p>
<ol>
<li>Airflow で [<strong>composer_hadoop_tutorial</strong>] をクリックして DAG の詳細ページを開きます。このページでは、ワークフローのタスクと依存関係が図で示されています。</li>
</ol>
<p><img alt="36269bfc2aa177b0.png" src="https://cdn.qwiklabs.com/8XEkWjYsF%2FyLN9OwhnUw617grlu1YYfEneShhCsUsZY%3D"></p>
<ol start="2">
<li>ツールバーの [<strong>Graph View</strong>] をクリックします。各タスクのグラフィックにカーソルを合わせると、そのタスクのステータスが表示されます。各タスクを囲む線の色もステータスを表しています（緑は実行中、赤は失敗など）。</li>
</ol>
<p><img alt="7ff3ff1262891e06.png" src="https://cdn.qwiklabs.com/6oFp1AbGNvE2AGYELR14lma8fICFtNk2MqghT89PBxQ%3D"></p>
<ol start="3">
<li>[Refresh] をクリックして最新情報を表示すると、プロセスを囲む線の色がステータスに応じて変化するのを確認できます。</li>
</ol>
<p>プロセスのステータスが「Success」（成功）になったら、もう一度ワークフローを [<strong>Graph View</strong>] から実行します。</p>
<ol>
<li>[<strong>create_dataproc_cluster</strong>] グラフィックをクリックします。</li>
<li>[<strong>Clear</strong>] をクリックして 3 つのタスクをリセットします。
<img alt="afc5474e843a0a8c.png" src="https://cdn.qwiklabs.com/rF%2FsanIZyrcXKOUyCNYnyNrx0mGZaaVlUWDyLkOdDVQ%3D">
</li>
<li>次に、[<strong>OK</strong>] をクリックして確定します。</li>
</ol>
<p><strong>create_dataproc_cluster</strong> を囲む線の色が変化し、ステータスが「Running」（実行中）になったことに注目してください。</p>
<p>GCP Console でプロセスをモニタリングすることもできます。</p>
<ol>
<li>
<p><strong>create_dataproc_cluster</strong> のステータスが「Running」に変わったら、<strong>ナビゲーション メニュー</strong> &gt; [<strong>Dataproc</strong>] に移動し、次の操作を行います。</p>
</li>
</ol>
<ul>
<li>
<p>[<strong>Clusters</strong>] をクリックして、クラスタの作成と削除をモニタリングします。ワークフローによって作成されたクラスタは一時的なものです。ワークフローの実行中にのみ存在し、最後のワークフロー タスクで削除されます。</p>
</li>
<li>
<p>[<strong>Jobs</strong>] をクリックして、Apache Hadoop ワードカウント ジョブをモニタリングします。ジョブ ID をクリックすると、ジョブのログ出力を確認できます。</p>
</li>
</ul>
<ol start="2">
<li>Dataproc のステータスが「Running」になったら Airflow に戻って [<strong>Refresh</strong>] をクリックすると、クラスタが完成したことを確認できます。</li>
</ol>
<p><code>run_dataproc_hadoop</code> プロセスが完了したら、<strong>ナビゲーション メニュー</strong> &gt; [<strong>Storage</strong>] &gt; [<strong>ブラウザ</strong>] に移動し、バケット名をクリックして <code>wordcount</code> フォルダでワードカウントの結果を確認できます。</p>
