**データ分析のためのBigQuery**

データアナリストのスキルであるSQLやビジュアリゼーションの学習に興味がありますか？ペタバイトサイズのデータセットに拡張できるクエリを書きたいですか？本クエストを受講し、クエリ、インジェスト、最適化、ビジュアル化、そして機械学習モデルの構築を学びましょう。

<h2 id="step2">概要</h2>
<p>SQL（構造化クエリ言語）とは、データ操作のための標準言語であり、構造化データセットを照会して分析情報を得ることができます。データベースの管理に一般的に使用されており、リレーショナル データベースへのトランザクション レコードの入力、ペタバイト規模のデータ分析といった作業を行うことができます。</p>
<p>このラボは SQL の入門編となっており、Qwiklabs でこれからデータ サイエンスに関する多数のラボやクエストに取り組むための足がかりとなります。このラボは 2 部に分かれています。前半では SQL クエリの基本的なキーワードについて学び、ロンドン市内のシェア自転車に関する一般公開データセットに対して BigQuery コンソールでクエリを実行します。</p>
<p>後半では、ロンドン市内のシェア自転車に関するデータセットのサブセットを CSV ファイルにエクスポートする方法を学んでから、そのファイルを Cloud SQL にアップロードします。その後、Cloud SQL を使用して、データベースやテーブルを作成、管理する方法を学びます。最後に、データを操作、編集するその他の SQL キーワードを実際に試してみます。</p>
<h3>目標</h3>
<p>このラボでは、次の方法について学びます。</p>
<ul>
<li>データベースをテーブルやプロジェクトと区別する。</li>
<li>
<code>SELECT</code>、<code>FROM</code>、<code>WHERE</code> の各キーワードを使って簡単なクエリを組み立てる。</li>
<li>BigQuery コンソール内のコンポーネントや階層構造を確認する。</li>
<li>データベースやテーブルを BigQuery に読み込む。</li>
<li>テーブルに対して簡単なクエリを実行する。</li>
<li>
<code>COUNT</code>、<code>GROUP BY</code>、<code>AS</code>、<code>ORDER BY</code> の各キーワードについて理解する。</li>
<li>上記のコマンドを実行、連結して、データセットから意味のあるデータを pull する。</li>
<li>データのサブセットを CSV ファイルにエクスポートし、そのファイルを Cloud Storage の新しいバケットに格納する。</li>
<li>新しい Cloud SQL インスタンスを作成し、エクスポートした CSV ファイルを新しいテーブルとして読み込む。</li>
<li>
<code>CREATE DATABASE</code>、<code>CREATE TABLE</code>、<code>DELETE</code>、<code>INSERT INTO</code>、<code>UNION</code> の各クエリを Cloud SQL で実行する。</li>
</ul>
<h3>前提条件</h3>
<p><strong>重要:</strong> ラボを開始する前に、個人用 Gmail アカウントからログアウトしてください。</p>
<p>これは<strong>入門レベル</strong>のラボです。これまでに SQL を使用した経験がほとんど、またはまったくない方を対象としています。Cloud Storage や Cloud Shell の知識があれば役立ちますが、必須ではありません。このラボでは、SQL でのクエリの読み書きの基礎について学び、その知識を BigQuery や Cloud SQL で実際に試してみます。</p>
<p>ラボを始める前に、ご自身の SQL の習熟度をご検討ください。以下のラボは、このラボよりも難易度が高くなっており、お持ちの知識をより高度なユースケースに応用していただけます。</p>
<ul>
<li><a href="https://google.qwiklabs.com/catalog_lab/476">BigQuery の天気データ</a></li>
<li><a href="https://google.qwiklabs.com/catalog_lab/479">Datalab と BigQuery による出生率データの分析</a></li>
</ul>
<p>準備ができたら、下にスクロールし、以下に示す手順に沿ってラボ環境をセットアップします。</p>

<h2 id="step4">SQL の基礎</h2>
<h3>データベースとテーブル</h3>
<p>前述のように、SQL では「構造化データセット」から情報を取り出すことができます。構造化データセットには明確なルールと書式があり、通常はテーブル形式（行と列のデータ）になっています。</p>
<p>非構造化データ<em></em>には、たとえば画像ファイルがあります。非構造化データは SQL で操作できず、BigQuery のデータセットやテーブルに格納できません（少なくともネイティブでは格納できません）。たとえば画像データを操作するには、<a href="https://google.qwiklabs.com/catalog_lab/1241">API</a> を通じて <a href="https://google.qwiklabs.com/catalog_lab/1112">Cloud Vision</a> のようなサービスを直接利用します。</p>
<p>構造化データセットの例（単純なテーブル）を以下に示します。</p>
<table>
<tr>
<td colspan="1" rowspan="1"><p><strong>ユーザー</strong></p></td>
<td colspan="1" rowspan="1"><p><strong>料金</strong></p></td>
<td colspan="1" rowspan="1"><p><strong>発送済み</strong></p></td>
</tr>
<tr>
<td colspan="1" rowspan="1"><p>山田</p></td>
<td colspan="1" rowspan="1"><p>3,500 円</p></td>
<td colspan="1" rowspan="1"><p>○</p></td>
</tr>
<tr>
<td colspan="1" rowspan="1"><p>佐藤</p></td>
<td colspan="1" rowspan="1"><p>5,000 円</p></td>
<td colspan="1" rowspan="1"><p>×</p></td>
</tr>
</table>
<p>Google スプレッドシートを使ったことがある場合は、このようなテーブルに見覚えがあるでしょう。テーブルには「ユーザー」、「料金」、「発送済み」の列があり、各列に値が入力された行が 2 行あります。</p>
<p>データベースは基本的に 1 つまたは複数のテーブルの集合です。<em></em>SQL は構造化データベースの管理ツールですが、このラボのように、データベース全体ではなく、1 つのテーブルや、結合された複数のテーブルに対してクエリを実行することも一般的です。</p>
<h3>SELECT と FROM</h3>
<p>SQL のキーワードは文字どおりの意味を持ちますが、クエリを実行する前に、データへの質問を組み立てておくと役立ちます（ただし、楽しみのためにデータを調べてみたいだけ場合は別です）。</p>
<p>SQL にはあらかじめ定義されたキーワード<em></em>があります。これらのキーワードを使用して、質問を英語に似た SQL 構文に変換することで、求める答えをデータベース エンジンから受け取ることができます。</p>
<p>特に重要なキーワードに <code>SELECT</code> と <code>FROM</code> があります。</p>
<ul>
<li>
<code>SELECT</code> は、データセットから pull するフィールドを指定します。</li>
<li>
<code>FROM</code> は、データを pull する 1 つまたは複数のテーブルを指定します。</li>
</ul>
<p>わかりやすいように例を使って説明します。以下にテーブル <code>example_table</code> があります。USER、PRICE、SHIPPED の列があるのがわかります。</p>
<p><img alt="14422cb7144f3ae.png" src="https://cdn.qwiklabs.com/XOxiZPGbXJ2GBjG164aXcFmKI35BtpMO0%2FiWcbIYLfQ%3D"></p>
<p>pull するのは USER 列のデータだけだとします。この場合、<code>SELECT</code> と <code>FROM</code> を使用して以下のクエリを実行します。</p>
<pre><code>SELECT USER FROM example_table&#x000A;</code></pre>
<p>このコマンドを実行したすると、<code>example_table</code> から <code>USER</code> 列の名前がすべて選択されます。</p>
<p>SQL の <code>SELECT</code> キーワードでは、複数の列を選択することもできます。USER 列と SHIPPED 列からデータを pull するとしましょう。この場合、前のクエリを変更して <code>SELECT</code> クエリに列を追加します（必ずカンマで区切ります）。</p>
<pre><code>SELECT USER, SHIPPED FROM example_table&#x000A;</code></pre>
<p>このコマンドを実行すると、メモリから <code>USER</code> と <code>SHIPPED</code> のデータを取得できます。</p>
<p><img alt="a4027fb83edf734.png" src="https://cdn.qwiklabs.com/u4knQcxxNRrBjThYXtSccEq9%2BroxOz0FL0Jg6zCzIAE%3D"></p>
<p>これで、基本的な SQL キーワードを 2 つ学ぶことができました。では、もう少し複雑な内容に進みましょう。</p>
<h3>WHERE</h3>
<p><code>WHERE</code> というキーワードも SQL コマンドのひとつで、特定の列の値でテーブルをフィルタできます。<code>example_table</code> から、商品が発送済みのユーザーの名前を pull するとしましょう。この場合は、次のようにクエリに <code>WHERE</code> を追加します。</p>
<pre><code>SELECT USER FROM example_table WHERE SHIPPED='YES'&#x000A;</code></pre>
<p>このコマンドを実行すると、商品が発送済みのすべてのユーザーがメモリから返されます。</p>
<p><img alt="5566150a165277e8.png" src="https://cdn.qwiklabs.com/wst%2BjF1gd%2FvqJaN8OPxTgoWdUbT%2BcWKMVipwc2wlYWw%3D"></p>
<p>SQL の主なキーワードを理解できたところで、BigQuery コンソールでこれらを使ってクエリを実行し、学んだことを試してみましょう。</p>

<h2 id="step5">BigQuery コンソールの操作</h2>
<h3>BigQuery の枠組み</h3>
<p><a href="https://cloud.google.com/bigquery/">BigQuery</a> は Google Cloud Platform 上で動作する、ペタバイト規模のフルマネージド データ ウェアハウスです。データ アナリストやデータ サイエンティストは、サーバーを設定、管理することなく、大規模なデータセットに対するクエリやフィルタの実行、結果の集計、複雑な操作の実行が可能です。コマンドライン ツール（Cloud Shell にインストール済み）またはウェブ コンソールを使用して、GCP プロジェクトに格納されているデータを管理、照会できます。</p>
<p>このラボでは、ウェブ コンソールを使用して SQL クエリを実行します。</p>
<h3>BigQuery Console を開く</h3>
<p>Google Cloud Console で [<strong>ナビゲーション メニュー</strong>] &gt; [<strong>BigQuery</strong>] を選択します。</p>
<p><img alt="BigQuery_menu.png" src="https://cdn.qwiklabs.com/QwUX6algdXPgIThqBlgB2rnqGkoQUUANkl3xicD06uc%3D"></p>
<p>[<strong>Cloud Console の BigQuery へようこそ</strong>] メッセージボックスが開きます。このメッセージボックスには、クイックスタート ガイドとリリースノートへのリンクが表示されています。</p>
<p>[<strong>完了</strong>] をクリックします。</p>
<p>BigQuery コンソールが開きます。</p>
<p>
</p>
<p><img alt="bq-console.png" src="https://cdn.qwiklabs.com/vd3QNHGB4BAyHjAvOHw9qD0iqCPNaHAzY657z%2FGWLtY%3D"></p>

<p>この UI の主な機能を確認しておきましょう。コンソールの右側には [クエリエディタ] があります。ここで、前述のような SQL コマンドを記述、実行します。その下に [クエリの履歴] があります。ここには、以前に実行したクエリの一覧が表示されます。</p>
<p>コンソールの左側には「ナビゲーション パネル」があります。クエリ履歴、保存済みクエリ、ジョブ履歴のほかに、[リソース<em></em>] タブがあります。</p>
<p>リソースの最上位には GCP プロジェクトがあります。これは、Qwiklabs でログインし、使用する一時的な GCP プロジェクトと同じようなものです。作業中のコンソールとこのスクリーンショットを見るとわかるように、現在は [リソース] タブに Qwiklabs プロジェクトしかありません。プロジェクト名の横にある矢印をクリックしても、何も表示されません。</p>
<p>これは、プロジェクトにデータセットやテーブルが含まれておらず、クエリを実行できるものが何もないからです。データセットにはテーブルが含まれることは前に説明しました。プロジェクトにデータを追加すると、BigQuery では、<em>プロジェクトにデータセットが含まれ、データセットにテーブルが含まれます</em>。
「プロジェクト → データセット → テーブル」という枠組みと、コンソールの詳細について理解できたところで、クエリを実行できるデータを読み込みます。</p>
<h3>クエリ可能なデータをアップロードする</h3>
<p>このセクションでは、一般公開データをプロジェクトに pull し、BigQuery で SQL コマンドの実行を練習できるようにします。</p>
<p>[<strong>+ データを追加</strong>] リンクをクリックし、[<strong>一般公開データセットを調べる</strong>] を選択します。</p>
<p><img alt="BQ_adddata.png" src="https://cdn.qwiklabs.com/F9lhtxyIiwttzF%2FyortRrX8dOAJ60sTN%2BKCNyuXHcfU%3D"></p>
<p>検索バーに「london」と入力し、<strong>London Bicycle Hires</strong> のタイルを選択して、[<strong>データセットを表示</strong>] をクリックします。</p>
<p>新しいタブが開いて、<code>bigquery-public-data</code> という新しいプロジェクトが [リソース] パネルに追加されます。</p>
<p><img alt="BQ_pubdata.png" src="https://cdn.qwiklabs.com/f1yJh%2FCYs%2FKZ1D9Ta%2FU1mWAQh3VsOzu0Z%2Becb5CYimY%3D"></p>
<p>この新しいタブでも、引き続き Qwiklabs プロジェクトを操作している点に注意してください。データセットやテーブルを含む一般公開されているプロジェクトを、分析のために BigQuery に pull しただけであり、そのプロジェクトに切り替えた<em></em>わけではありません。ジョブやサービスは Qwiklabs アカウントに関連付けられています。これは、コンソール上部のプロジェクト フィールドで確認できます。</p>
<p><img alt="BQ_proj_check.png" src="https://cdn.qwiklabs.com/eUyizy8IKV89BwSWBtOLM5MhJLemWB%2BCzgSHeaCbn1w%3D"></p>
<p>[<strong>bigquery-public-data</strong>] &gt; [<strong>london_bicycles</strong>] &gt; [<strong>cycle_hire</strong>] をクリックします。データは以下のように BigQuery の枠組みに従っています。</p>
<ul>
<li>GCP プロジェクト → <code>bigquery-public-data</code>
</li>
<li>データセット → <code>london_bicycles</code>
</li>
<li>テーブル → <code>cycle_hire</code>
</li>
</ul>
<p><code>cycle_hire</code> テーブルが開いたら、コンソールの中央で [<strong>プレビュー</strong>] タブをクリックします。ページは次のようになります。</p>
<p><img alt="cycle_hire.png" src="https://cdn.qwiklabs.com/foj4wmiY8F%2FcK1BpqVCM%2BP4CgU7Sf5WAk7fNaaeiaZM%3D"></p>
<p>列や、行に入力されている値を確認します。これで、<code>cycle_hire</code> テーブルに対して SQL クエリを実行できる状態になりました。</p>
<h3>BigQuery で SELECT、FROM、WHERE を実行する</h3>
<p>これで、SQL クエリのキーワードと BigQuery のデータの枠組みについて理解でき、使用するデータを用意できました。GCP サービスを使用していくつか SQL コマンドを実行しましょう。</p>
<p>コンソールの右下を見ると、<strong>24,369,201</strong> 行のデータがあることがわかります。これは、2015 年から 2017 年の間にロンドン市内で利用されたシェア自転車の件数を示しています（決して少ない数ではありません）。</p>
<p>7 列目のキー <code>end_station_name</code> に注目します。これは、シェア自転車の最終目的地を示しています。詳細に進む前に、<code>end_station_name</code> 列を分離する簡単なクエリを実行します。次のコマンドをコピーして、クエリエディタに貼り付けます。</p>
<pre><code>SELECT end_station_name FROM `bigquery-public-data.london_bicycles.cycle_hire`;&#x000A;</code></pre>
<p>[<strong>実行</strong>] をクリックします。</p>
<p>20 秒ほど経過すると、クエリで指定した列 <code>end_station_name</code> を含む 24,369,201 行が返されます。</p>
<p>それでは、乗車時間が 20 分以上だった件数を確認してみましょう。</p>
<p>[<strong>クエリを新規作成</strong>] をクリックしてクエリエディタの表示をクリアし、<code>WHERE</code> キーワードを使用した、次のクエリを実行します。</p>
<pre><code>SELECT * FROM `bigquery-public-data.london_bicycles.cycle_hire` WHERE duration&gt;=1200;&#x000A;</code></pre>
<p>このクエリの実行には 1 分程度かかる場合があります。</p>
<p><code>SELECT *</code> により、テーブルからすべての列の値が返されます。duration は秒単位になっているため、1200（60 x 20）という値を使用しています。</p>
<p>右下を見ると、<strong>7,334,890</strong> 行が返されたことがわかります。全体に占める割合（7334890/24369201）で見ると、ロンドン市内のシェア自転車利用件数のうち 30% が 20 分以上だった（長時間の利用が多い）ことがわかります。</p>

<h2 id="step6">その他の SQL キーワード: GROUP BY、COUNT、AS、ORDER BY</h2>
<h3>GROUP BY</h3>
<p><code>GROUP BY</code> キーワードは、一定の基準（列値など）を満たす結果セットの行を集計し、その基準で見つかった一意のエントリをすべて返します。</p>
<p>このキーワードは、テーブルを分類する情報を理解するのに役立ちます。このキーワードの機能をさらに理解するには、[<strong>クエリを新規作成</strong>] をクリックし、次のコマンドをコピーして、クエリエディタに貼り付けます。</p>
<pre><code>SELECT start_station_name FROM `bigquery-public-data.london_bicycles.cycle_hire` GROUP BY start_station_name;&#x000A;</code></pre>
<p>[<strong>実行</strong>] をクリックします。</p>
<p>以下のような出力が返されます（行の値は異なる場合があります）。</p>
<p><img alt="30b04fc6f21c544c.png" src="https://cdn.qwiklabs.com/WcheomnmNVxpZflsIhCmoSmxbakCLWCugBjaQ6ARrLI%3D"></p>
<p><code>GROUP BY</code> がなければ、<strong>24,369,201</strong> 行がすべて返されたはずです。<code>GROUP BY</code> は、テーブルで見つかった一意の（重複しない）列値を出力します。これは、右下を見ると確認できます。行数は <strong>880</strong> になっています。これは、ロンドン市内のシェア自転車には 880 か所の異なる出発地があることを示しています。</p>
<h3>COUNT</h3>
<p><code>COUNT()</code> 関数は、同じ基準（列値など）を満たす行の数を返します。これは、<code>GROUP BY</code> と一緒に使用すると便利です。</p>
<p>前のクエリに <code>COUNT</code> 関数を追加して、出発地ごとの乗車件数を求めます。[<strong>クエリを新規作成</strong>] をクリックし、次のコマンドをコピーしてクエリエディタに貼り付け、[<strong>クエリを実行</strong>] をクリックします。</p>
<pre><code>SELECT start_station_name, COUNT(*) FROM `bigquery-public-data.london_bicycles.cycle_hire` GROUP BY start_station_name;&#x000A;</code></pre>
<p>以下のような出力が返されます（行の値は異なる場合があります）。</p>
<p><img alt="b62aaf26b7a35e35.png" src="https://cdn.qwiklabs.com/rvBPMCHcoM0etV%2BNXOvBwHnfh6LWgAAzBI5o0%2FKM97Y%3D"></p>
<p>結果は、出発地ごとの乗車件数を示しています。</p>
<h3>AS</h3>
<p>SQL には <code>AS</code> キーワードもあります。これは、テーブルまたは列のエイリアス<em></em>を作成します。エイリアスとは、返される列またはテーブルにつける新しい名前です。その名前を <code>AS</code> で指定します。</p>
<p>前のクエリに <code>AS</code> キーワードを追加して、実際の処理を見てみましょう。[<strong>クエリを新規作成</strong>] をクリックし、次のコマンドをコピーして、クエリエディタに貼り付けます。</p>
<pre><code>SELECT start_station_name, COUNT(*) AS num_starts FROM `bigquery-public-data.london_bicycles.cycle_hire` GROUP BY start_station_name;&#x000A;</code></pre>
<p>[<strong>実行</strong>] をクリックします。</p>
<p>以下のような出力が返されます（行の値は異なる場合があります）。</p>
<p><img alt="a45a8b122dbfea2b.png" src="https://cdn.qwiklabs.com/tne5HErHLjL7JXpNCIHadDKX1n70jKGnDJcdgwKdHXM%3D"></p>
<p>この結果からわかるように、返されるテーブルの <code>COUNT(*)</code> 列がエイリアス名 <code>num_starts</code> に設定されています。このキーワードは、大規模なデータセットを操作する場合に特に便利です。あいまいなテーブル名や列名が何を指しているのかわからなくなることは、想像以上によくあります。</p>
<h3>ORDER BY</h3>
<p><code>ORDER BY</code> キーワードは、指定の基準または列値に応じて、クエリから返されるデータを昇順または降順で並べ替えます。前のクエリにこのキーワードを追加して、以下の処理を行います。</p>
<ul>
<li>各ステーションを出発地とする乗車件数を含み、出発ステーションがアルファベット順になっているテーブルを返す。</li>
<li>各ステーションを出発地とする乗車件数を含み、件数が昇順になっているテーブルを返す。</li>
<li>各ステーションを出発地とする乗車件数を含み、件数が降順になっているテーブルを返す。</li>
</ul>
<p>以下のコマンドは、それぞれが 1 つのクエリです。コマンドごとに、クエリエディタをクリアし、コマンドをコピーしてクエリエディタに貼り付け、[<strong>実行</strong>] をクリックします。結果を確認してみましょう。</p>
<pre><code>SELECT start_station_name, COUNT(*) AS num FROM `bigquery-public-data.london_bicycles.cycle_hire` GROUP BY start_station_name ORDER BY start_station_name;&#x000A;</code></pre>
<pre><code>SELECT start_station_name, COUNT(*) AS num FROM `bigquery-public-data.london_bicycles.cycle_hire` GROUP BY start_station_name ORDER BY num;&#x000A;</code></pre>
<pre><code>SELECT start_station_name, COUNT(*) AS num FROM `bigquery-public-data.london_bicycles.cycle_hire` GROUP BY start_station_name ORDER BY num DESC;&#x000A;</code></pre>
<p>最後のクエリは以下の結果を返します。</p>
<p><img alt="368742bde0aa54d6.png" src="https://cdn.qwiklabs.com/L3uDkzg%2B3uiiBgBwaGSnhVmqVpXP9waOgMIg9zLDG2c%3D"></p>
<p>「Belgrove Street, King's Cross」からの出発が最も多いことがわかります。ただし、全体に占める割合（234458/24369201）を見ると、このステーションから出発している件数は 1% に満たないことがわかります。</p>

<h2 id="step7">Cloud SQL の操作</h2>
<h3>クエリを CSV ファイルとしてエクスポートする</h3>
<p><a href="https://cloud.google.com/sql/">Cloud SQL</a> はクラウド上の PostgreSQL と MySQL のリレーショナル データベースを簡単に設定、維持、運用、管理できるようにするフルマネージド データベース サービスです。Cloud SQL が対応しているデータ形式には、ダンプファイル（.sql）と CSV ファイル（.csv）があります。ここでは、<code>cycle_hire</code> テーブルのサブセットを CSV ファイルにエクスポートし、一時的な場所として Cloud Storage にアップロードする方法を説明します。</p>
<p>BigQuery コンソールでは、以下のコマンドを最後に実行しました。</p>
<pre><code>SELECT start_station_name, COUNT(*) AS num FROM `bigquery-public-data.london_bicycles.cycle_hire` GROUP BY start_station_name ORDER BY num DESC;&#x000A;</code></pre>
<p>[クエリ結果] セクションで、[<strong>結果を保存する</strong>] &gt; [<strong>CSV（ローカル ファイル）</strong>] をクリックします。これにより、ダウンロードが実行され、このクエリが CSV ファイルとして保存されます。ダウンロードされたファイルの場所と名前は、後で使用するためメモしておきます。</p>
<p>[<strong>クエリを新規作成</strong>] をクリックし、次のコマンドをコピーして、クエリエディタに貼り付けます。</p>
<pre><code>SELECT end_station_name, COUNT(*) AS num FROM `bigquery-public-data.london_bicycles.cycle_hire` GROUP BY end_station_name ORDER BY num DESC;&#x000A;</code></pre>
<p>このクエリは、各ステーションを到着地とする乗車件数を含み、件数が降順になっているテーブルを返します。次の出力が表示されます。</p>
<p><img alt="814554dc82c60a74.png" src="https://cdn.qwiklabs.com/mAesoVBfXplwi1b5bh1nIRILSAi7%2FBbebkAhpzb2CZ4%3D"></p>
<p>[クエリ結果] セクションで [<strong>結果を保存する</strong>] &gt; [<strong>CSV</strong>] をクリックします。これにより、ダウンロードが実行され、このクエリが CSV ファイルとして保存されます。ダウンロードされたファイルの場所と名前は、この後のセクションで使用するためメモしておきます。</p>
<h3>CSV ファイルを Cloud Storage にアップロードする</h3>
<p>GCP Console に移動し、ストレージ バケットを作成します。作成したファイルは、ストレージ バケットにアップロードできます。</p>
<p><strong>ナビゲーション メニュー</strong>で [<strong>Storage</strong>] &gt; [<strong>ブラウザ</strong>] をクリックし、[<strong>バケットを作成</strong>] をクリックします。</p>
<p>バケットに一意の名前を入力し、その他の設定は変更せずに [<strong>作成</strong>] をクリックします。</p>
<p><img alt="359fdfd56ad5c454.png" src="https://cdn.qwiklabs.com/MFrZ8g98FGK%2FgBtWhPVHBAvbMqkeIZK%2FC%2FtKy6r0a4w%3D"></p>

<p>GCP Console で、新しく作成された Cloud Storage バケットが表示されます。</p>
<p>[<strong>ファイルをアップロード</strong>] をクリックし、<code>start_station_name</code> のデータが含まれる CSV ファイルを選択します。次に [<strong>開く</strong>] をクリックします。<code>end_station_name</code> のデータについても同様に操作します。</p>
<p><code>start_station_name</code> ファイルの名前を変更します。ファイル名の端にあるその他アイコンをクリックして、[<strong>名前を変更</strong>] をクリックします。ファイル名を「<code>start_station_data.csv</code>」に変更します。</p>
<p><code>end_station_name</code> ファイルの名前を変更します。ファイル名の端にあるその他アイコンをクリックして、[<strong>名前を変更</strong>] をクリックします。ファイル名を「<code>end_station_data.csv</code>」に変更します。</p>
<p>バケットは以下のようになります。</p>
<p><img alt="4ca41c9e381d94f.png" src="https://cdn.qwiklabs.com/O0gGDUAw3%2BKFgvwpeQvYtmRFgfAlChH09mZMXpztL%2FM%3D"></p>

<h3>Cloud SQL インスタンスを作成する</h3>
<p>コンソールで、<strong>ナビゲーション メニュー</strong> &gt; [<strong>SQL</strong>] を選択します。</p>
<p>[<strong>インスタンスを作成</strong>] をクリックします。</p>
<p>データベース エンジンンの選択を求められたら、[<strong>MySQL</strong>] を選択します。</p>
<p>インスタンス名（「qwiklabs-demo」など）を入力し、[<strong>root パスワード</strong>] フィールドに安全なパスワードを入力して、[<strong>作成</strong>] をクリックします。入力したパスワードを忘れないようにしてください。</p>
<p><img alt="cloud-sql-instance.gif" src="https://cdn.qwiklabs.com/WUXtEyQAsB3cC2zATXIR1P8kq2BvnrT88egc0y%2F0sFo%3D"></p>
<p>インスタンスが作成されるまで数分かかることがあります。作成されたら、インスタンス名の横に緑色のチェックマークが表示されます。</p>
<p>Cloud SQL インスタンスをクリックします。以下のようなページが表示されます。</p>
<p><img alt="daeb5119c32adea.png" src="https://cdn.qwiklabs.com/vo4A5hMKLopnx9%2B%2FuNs%2FhMM4y53QY8XEpzCBsvJ88t0%3D"></p>

<h2 id="step8">Cloud SQL の新しいクエリ</h2>
<h3>CREATE キーワード（データベースとテーブル）</h3>
<p>Cloud SQL インスタンスが起動して実行中になったので、Cloud Shell コマンドラインを使用してそのインスタンス内にデータベースを作成します。</p>

<p>Cloud Shell で以下のコマンドを実行して SQL インスタンスに接続します。インスタンスに <code>qwiklabs-demo</code> 以外の名前を使用した場合はその名前に置き換えます。</p>
<pre><code>gcloud sql connect  qwiklabs-demo --user=root&#x000A;</code></pre>
<p>インスタンスへの接続には 1 分程度かかる場合があります。</p>
<p>プロンプトが表示されたら、インスタンスに指定した root パスワードを入力します。</p>
<p>次のような出力が表示されます。</p>
<pre><code>Welcome to the MariaDB monitor.  Commands end with ; or \g.&#x000A;Your MySQL connection id is 494&#x000A;Server version: 5.7.14-google-log (Google)&#x000A;&#x000A;Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.&#x000A;&#x000A;Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.&#x000A;&#x000A;MySQL [(none)]&gt;&#x000A;</code></pre>
<p>Cloud SQL インスタンスにはデータベースが事前に構成されていますが、ここでは独自のデータベースを作成して、ロンドン市内のシェア自転車のデータを格納します。</p>
<p>MySQL サーバーのプロンプトで以下のコマンドを実行して、<code>bike</code> というデータベースを作成します。</p>
<pre><code>CREATE DATABASE bike;&#x000A;</code></pre>
<p>次の出力が表示されます。</p>
<pre><code>Query OK, 1 row affected (0.05 sec)&#x000A;&#x000A;MySQL [(none)]&gt;&#x000A;</code></pre>

<p>次のコマンドを実行して、bike データベース内にテーブルを作成します。</p>
<pre><code>USE bike;&#x000A;CREATE TABLE london1 (start_station_name VARCHAR(255), num INT);&#x000A;</code></pre>
<p>この文では前と同じ <code>CREATE</code> キーワードを使用していますが、今回は <code>TABLE</code> 句を使用して、データベースではなくテーブルを作成することを指定しています。<code>USE</code> キーワードは、接続先のデータベースを指定しています。これで、「start_station_name」と「num」の 2 つの列を含む「london1」というテーブルが作成されます。<code>VARCHAR(255)</code> は、最大 255 文字を格納できる可変長の文字列型の列を指定し、<code>INT</code> は整数型の列です。</p>
<p>以下のコマンドを実行して、「london2」という別のテーブルを作成します。</p>
<pre><code>USE bike;&#x000A;CREATE TABLE london2 (end_station_name VARCHAR(255), num INT);&#x000A;</code></pre>
<p>空のテーブルが作成されたことを確認します。MySQL サーバーのプロンプトで以下のコマンドを実行します。</p>
<pre><code>SELECT * FROM london1;&#x000A;SELECT * FROM london2;&#x000A;</code></pre>
<p>どちらのコマンドも、出力は以下のようになります。</p>
<pre><code>Empty set (0.04 sec)&#x000A;</code></pre>
<p>「Empty set」と出力されるのは、まだデータを読み込んでいないためです。</p>
<h3>テーブルに CSV ファイルをアップロードする</h3>
<p>Cloud SQL コンソールに戻ります。ここで、CSV ファイル <code>start_station_name</code> と <code>end_station_name</code> を、新しく作成した london1 テーブルと london2 テーブルにアップロードします。</p>
<ol>
<li>Cloud SQL のインスタンス ページで、[<strong>インポート</strong>] をクリックします。</li>
<li>Cloud Storage ファイル フィールドで、[<strong>参照</strong>] をクリックし、バケット名と反対側にある矢印をクリックして、[<code>start_station_data.csv</code>] をクリックします。[<strong>選択</strong>] をクリックします。</li>
<li>
<code>bike</code> データベースを選択し、「london1」をテーブルとして入力します。</li>
<li>[<strong>インポート</strong>] をクリックします。</li>
</ol>
<p><img alt="3aab448fcc8a8a6f.png" src="https://cdn.qwiklabs.com/AmE%2BAIYHcE48Xt0k7wmRrB%2FJYPoe8dS%2BAmNFP2eZm5g%3D"></p>
<p>もう 1 つの CSV ファイルについても同様の操作を行います。</p>
<ol>
<li>Cloud SQL のインスタンス ページで、[<strong>インポート</strong>] をクリックします。</li>
<li>Cloud Storage ファイル フィールドで、[<strong>参照</strong>] をクリックし、バケット名と反対側にある矢印をクリックして、[<code>end_station_data.csv</code>] をクリックします。[<strong>選択</strong>] をクリックします。</li>
<li>bike データベースを選択し、「london2」をテーブルとして入力します。</li>
<li>[<strong>インポート</strong>] をクリックします。</li>
</ol>
<p>これで、両方の CSV ファイルが、<code>bike</code> データベース内のテーブルにアップロードされました。</p>
<p>Cloud Shell セッションに戻り、MySQL サーバーのプロンプトで以下のコマンドを実行して、london1 の内容を確認します。</p>
<pre><code>SELECT * FROM london1;&#x000A;</code></pre>
<p>出力は、ステーション名ごとに 1 行ずつ、計 881 行になります。出力の形式は以下のようになります。</p>
<p><img alt="48c3c74603692827.png" src="https://cdn.qwiklabs.com/gHiOlNC7sj3nJvU6dgHAVtU4bzDaVWvK6MBEaKRwLH8%3D"></p>
<p>以下のコマンドを実行して、london2 にデータが入力されていることを確認します。</p>
<pre><code>SELECT * FROM london2;&#x000A;</code></pre>
<p>出力は、ステーション名ごとに 1 行ずつ、計 883 行になります。出力の形式は以下のようになります。</p>
<p><img alt="85a788ec7971f8a0.png " src="https://cdn.qwiklabs.com/mUE0WbdWa8RP6L4nH%2BJIOBd3M6TBJB3Yi7%2FQje8zcZM%3D"></p>
<h3>DELETE キーワード</h3>
<p>データ管理に役立つ SQL キーワードをさらにいくつか紹介します。最初は <code>DELETE</code> キーワードです。</p>
<p>以下のコマンドを MySQL セッションで実行し、london1 と london2 の 1 行目を削除します。</p>
<pre><code>DELETE FROM london1 WHERE num=0;&#x000A;DELETE FROM london2 WHERE num=0;&#x000A;</code></pre>
<p>どちらのコマンドを実行しても、以下が出力されます。</p>
<pre><code>Query OK, 1 row affected (0.04 sec)&#x000A;</code></pre>
<p>削除された行は CSV ファイルの列見出しでした。この DELETE キーワードは、必ずファイルの 1 行目を削除するというものではなく、列名（この例では「num」）に特定の値（この例では「0」）が含まれるすべての行<code></code><em></em>をテーブルから削除します。<code>SELECT * FROM london1;</code> と <code>SELECT * FROM london2;</code> の各クエリを実行して、テーブルの先頭までスクロールしてみると、その行がもうないことがわかります。</p>
<h3>INSERT INTO キーワード</h3>
<p><code>INSERT INTO</code> キーワードを使用して、テーブルに値を挿入することもできます。次のコマンドを実行して、london1 に新しい行を挿入します。<code>start_station_name</code> は「test destination」に、<code>num</code> は「1」に設定されます。</p>
<pre><code>INSERT INTO london1 (start_station_name, num) VALUES ("test destination", 1);&#x000A;</code></pre>
<p><code>INSERT INTO</code> キーワードはテーブル（「london1」）を必要とし、最初のかっこ内の用語（この例では「start_station_name」と「num」）で指定された列が含まれる新しい行を作成します。「VALUES」句に続く値が新しい行に挿入されます。</p>
<p>次の出力が表示されます。</p>
<pre><code>Query OK, 1 row affected (0.05 sec)&#x000A;</code></pre>
<p><code>SELECT * FROM london1;</code> クエリを実行すると、「london1」テーブルの末尾に新しい行が追加されていることがわかります。</p>
<p><img alt="b067eb36e63b9e68.png" src="https://cdn.qwiklabs.com/eYhqa3ycQ83rA1PDALG5rffOmKQ5OkLYuduwGhjejK4%3D"></p>
<h3>UNION キーワード</h3>
<p>ここで紹介する最後の SQL キーワードは <code>UNION</code> です。このキーワードは、複数の <code>SELECT</code> クエリの出力を結果セットに結合します。ここでは、<code>UNION</code> を使用して、「london1」と「london2」の各テーブルのサブセットを結合します。</p>
<p>以下の連結クエリは、両方のテーブルから特定のデータを pull し、<code>UNION</code> 演算子で結合します。</p>
<p>MySQL サーバーのプロンプトで次のコマンドを実行します。</p>
<pre><code>SELECT start_station_name AS top_stations, num FROM london1 WHERE num&gt;100000&#x000A;UNION&#x000A;SELECT end_station_name, num FROM london2 WHERE num&gt;100000&#x000A;ORDER BY top_stations DESC;&#x000A;</code></pre>
<p>最初の <code>SELECT</code> クエリは、「london1」テーブルから 2 つの列を選択し、「start_station_name」にはエイリアス「top_stations」を作成しています。<code>WHERE</code> キーワードを使用して、出発地になっている回数が 10 万回を超えるステーション名のみを pull しています。</p>
<p>2 番目の <code>SELECT</code> クエリは、「london2」テーブルから 2 つの列を選択し、<code>WHERE</code> キーワードを使用して、到着地になっている回数が 10 万回を超えるステーション名のみを pull しています。</p>
<p>間にある <code>UNION</code> キーワードは、「london2」のデータを「london1」と融合することで、これらのクエリの出力を結合します。「london1」に「london2」を結合するため、優先される列値は「top_stations」と「num」になります。</p>
<p><code>ORDER BY</code> は、最終的に結合されたテーブルを、「top_stations」列の値を使ってアルファベット降順に並べ替えます。</p>
<p>次の出力が表示されます。</p>
<p><img alt="aec2cf3f207027a0.png" src="https://cdn.qwiklabs.com/z9UUN0K%2FyW1UWaCn0M%2BU1b0srEzOO7EO%2BZ1xHk5aNq0%3D"></p>
<p>14 のうち 13 のステーションが、出発地としても到着地としても上位に入っていることがわかります。基本的な SQL キーワードを使って大規模なデータセットに対してクエリを実行し、データポイントと、具体的な質問への答えを受け取ることができました。</p>

# BigQuery: Qwik Start - コマンドライン

<h2 id="step2">概要</h2>
<p>適切なハードウェアとインフラストラクチャを用意せずに大規模なデータセットを保存したり、それに対してクエリを実行したりすると、時間と費用がかかってしまいます。Google BigQuery は、こうした問題を解決するために Google のインフラストラクチャの処理能力を使用して SQL クエリを超高速で実行する、<a href="https://cloud.google.com/solutions/bigquery-data-warehouse" target="_blank">エンタープライズ データ ウェアハウス</a>です。データを BigQuery に読み込んだら、後の処理は Google にお任せください。他のユーザーにデータの表示やクエリを許可するなど、ビジネスニーズに基づいてプロジェクトとデータ両方へのアクセスを制御できます。</p>
<p>BigQuery には、<a href="https://bigquery.cloud.google.com/" target="_blank">ウェブ UI</a> または<a href="https://cloud.google.com/bigquery/docs/cli_tool" target="_blank">コマンドライン ツール</a>から、あるいは、さまざまな<a href="https://cloud.google.com/bigquery/docs/reference/v2" target="_blank">クライアント ライブラリ</a>（Java、.NET、Python など）を使って <a href="https://cloud.google.com/bigquery/docs/reference/libraries" target="_blank">BigQuery REST API</a> を呼び出すことでアクセスできます。また、さまざまな<a href="https://cloud.google.com/bigquery/third-party-tools" target="_blank">サードパーティ製ツール</a>を使用して BigQuery と通信し、データを可視化したり、データを読み込んだりすることができます。</p>
<p>このハンズオンラボでは、コマンドライン インターフェースを使用して一般公開テーブルにクエリを実行する方法と、サンプルデータを BigQuery に読み込む方法を説明します。</p>

<h2 id="step4">テーブルを確認する</h2>
<p>BigQuery には、クエリを実行できる<a href="https://cloud.google.com/bigquery/docs/sample-tables" target="_blank">サンプル テーブル</a>がいくつかあります。このクイックスタートでクエリを実行する Shakespeare テーブルには、シェイクスピアの全戯曲のすべての単語のエントリが含まれています。</p>
<p>サンプル データセットの Shakespeare テーブルのスキーマを調べるには、次のコマンドを実行します。</p>
<pre><code>bq show bigquery-public-data:samples.shakespeare&#x000A;</code></pre>
<p>このコマンドでは下記を行います：</p>
<ul>
<li>BigQueryコマンドラインを呼び出すため、 <code>bq</code>を使用します。</li>
<li>
<code>show</code>はアクションです</li>
<li>その後で、bigqueryで拝見したい<code>project:public dataset.table</code> の名前をリストします。</li>
</ul>
<p>出力:</p>
<pre><code class="language-bash prettyprint">Table bigquery-public-data:samples.shakespeare&#x000A;&#x000A;   Last modified                  Schema                 Total Rows   Total Bytes   Expiration&#x000A; ----------------- ------------------------------------ ------------ ------------- ------------&#x000A;  26 Aug 14:43:49   |- word: string (required)           164656       6432064&#x000A;                    |- word_count: integer (required)&#x000A;                    |- corpus: string (required)&#x000A;                    |- corpus_date: integer (required)&#x000A;</code></pre>
<h2 id="step5">help コマンドを実行する</h2>
<p>help コマンドにコマンド名を指定すると、そのコマンドに関する情報が得られます。たとえば、次の <code>bq help</code> の呼び出しでは、<code>query</code> コマンドについての情報が表示されます。</p>
<pre><code>bq help query&#x000A;</code></pre>
<p><code>bq</code> で使用される全コマンドの一覧を表示するには、<code>bq help</code> とだけ実行します。</p>
<h2 id="step6">クエリの実行</h2>
<p>次に、クエリを実行して、Shakespeare の作品に 「raisin」 という部分文字列が何回出現するかを確認します。</p>
<p>Query を実行するには、<code>bq query "[SQL_STATEMENT]"</code> コマンドを実行します。</p>
<ul>
<li>
<code>[SQL_STATEMENT]</code> 内の引用符を \ マークでエスケープする、または</li>
<li>全体を囲んでいる記号とは異なる種類の引用符 (<code>"</code>, <code>'</code>) を使用します。</li>
</ul>
<p>Cloud Shell で次の標準 SQL クエリを実行して、Shakespeare のすべての作品に部分文字列<strong>「raisin」</strong>が表示された回数をカウントします。</p>
<pre><code>bq query --use_legacy_sql=false \&#x000A;'SELECT&#x000A;   word,&#x000A;   SUM(word_count) AS count&#x000A; FROM&#x000A;   `bigquery-public-data`.samples.shakespeare&#x000A; WHERE&#x000A;   word LIKE "%raisin%"&#x000A; GROUP BY&#x000A;   word'&#x000A;</code></pre>
<p>このコマンドでは：</p>
<ul>
<li>クエリで <code>--use_legacy_sql=false</code> 部分はスタンダード SQL をデフォルト クエリ シンタックスに作成します。</li>
</ul>
<p>出力:</p>
<pre><code class="language-bash prettyprint">Waiting on job_e19 ... (0s) Current status: DONE&#x000A;+---------------+-------+&#x000A;|     word      | count |&#x000A;+---------------+-------+&#x000A;| praising      |     8 |&#x000A;| Praising      |     4 |&#x000A;| raising       |     5 |&#x000A;| dispraising   |     2 |&#x000A;| dispraisingly |     1 |&#x000A;| raisins       |     1 |&#x000A;</code></pre>
<p>このテーブルは、実際には「<strong>raisin</strong>」という単語が出現していないものの、いくつかのシェイクスピアの作品にこの文字の並びが出現することを示しています。</p>

<p>Shakespeareの作品にはない言葉を探すと、何もが探しません。</p>
<p>次のように「<strong>huzzah</strong>」を検索すると、一致するものが返りません。</p>
<pre><code>bq query --use_legacy_sql=false \&#x000A;'SELECT&#x000A;  word&#x000A;FROM&#x000A;  `bigquery-public-data`.samples.shakespeare&#x000A;WHERE&#x000A;  word = "huzzah"'&#x000A;</code></pre>

<h2 id="step7">新しいテーブルを作成する</h2>
<p>次に、新しいテーブルを作成します。すべてのテーブルは、データセット内に存在する必要があります。データセットとはテーブルの単純なグループのことで、単一のプロジェクトに割り当てられます。</p>
<h3><strong>新しいデータセットの作成</strong></h3>
<p>デフォルトのプロジェクトに既存のデータセットがあるかどうかを確認するには、<code>bq ls</code> コマンドを使用します。</p>
<pre><code>bq ls&#x000A;</code></pre>
<p>デフォルトのプロジェクト内にはまだデータセットがないため、コマンドラインに戻ります。</p>
<p><code>bq ls</code> を再び実行し、プロジェクト ID <code>bigquery-public-data</code> と末尾にコロン（:）を指定して、特定のプロジェクトに含まれるデータセットの一覧を取得します。</p>
<pre><code>bq ls bigquery-public-data:&#x000A;</code></pre>
<p>出力:</p>
<pre><code class="language-bash prettyprint">          datasetId&#x000A; ------------------------------&#x000A; austin_311&#x000A; austin_bikeshare&#x000A; austin_crime&#x000A; austin_incidents&#x000A; austin_waste&#x000A; baseball&#x000A; bitcoin_blockchain&#x000A; bls&#x000A; census_bureau_construction&#x000A; census_bureau_international&#x000A; census_bureau_usa&#x000A; census_utility&#x000A; chicago_crime&#x000A; ...&#x000A;</code></pre>
<p>次は、データセットを作成します。データセット名は最大 1,024 文字で、A〜Z、a〜z、0〜9、およびアンダースコアで構成できますが、数字またはアンダースコアで始めたり、スペースを使用したりすることはできません。</p>
<p><code>bq mk</code> コマンドを使用して、<code>babynames</code> という名前の新しいデータセットを作成します。</p>
<pre><code>bq mk babynames&#x000A;</code></pre>
<p>出力例:</p>
<pre><code class="language-bash prettyprint">Dataset 'qwiklabs-gcp-ba3466847fe3cec0:babynames' successfully created.&#x000A;</code></pre>

<p><code>bq ls</code> を実行して、デフォルトのプロジェクトに新しいデータセットが表示されることを確認します。</p>
<pre><code>bq ls&#x000A;</code></pre>
<p>出力例:</p>
<pre><code class="language-bash prettyprint"> datasetId&#x000A; -------------&#x000A;  babynames&#x000A;</code></pre>
<h3><strong>テーブルのアップロード</strong></h3>
<p>テーブルを作成するには、その前にデータセットをプロジェクトに追加しておく必要があります。ここで使用するカスタムデータ ファイルには、米国社会保障局から提供された、人気のある赤ちゃんの名前に関する約 7 MB のデータが含まれています。</p>
<p>このコマンドを実行し、データファイルの URL を使用して、<a href="http://www.ssa.gov/OACT/babynames/names.zip" target="_blank">赤ちゃんの名前の zip ファイル</a>をプロジェクトに追加します。</p>
<pre><code>wget http://www.ssa.gov/OACT/babynames/names.zip&#x000A;</code></pre>
<p>ファイルの一覧を表示します。</p>
<pre><code>ls&#x000A;</code></pre>
<p>プロジェクトに追加されたファイルの名前を確認できます。</p>
<p>ファイルを展開します。</p>
<pre><code>unzip names.zip&#x000A;</code></pre>
<p>これは非常に大きなテキスト ファイルです。再びファイルの一覧を表示します。</p>
<pre><code>ls&#x000A;</code></pre>
<p><code>bq load</code> コマンドは、テーブルの作成または更新とデータの読み込みを 1 回のステップで実行します。</p>
<p><code>bq load</code> コマンドを使用して、今作成した babynames データセット内の names2010 という名前の新しいテーブルにソースファイルを読み込みます。デフォルトでは、このコマンドは同期的に実行し、完了するまでに数秒かかります。</p>
<p>実行するコード用の <code>bq load</code> コマンドの引数は次のとおりです。</p>
<pre><code class="language-bash prettyprint">datasetID: babynames&#x000A;tableID: names2010&#x000A;source: yob2010.txt&#x000A;schema: name:string,gender:string,count:integer&#x000A;</code></pre>
<p>テーブルを作成します。</p>
<pre><code>bq load babynames.names2010 yob2010.txt name:string,gender:string,count:integer&#x000A;</code></pre>
<p>出力例:</p>
<pre><code class="language-bash prettyprint">Waiting on job_4f0c0878f6184119abfdae05f5194e65 ... (35s) Current status: DONE&#x000A;</code></pre>

<p><code>「bq ls」</code>と<code>「babynames」</code>を実行して、テーブルがデータセットに表示されることを確認します。</p>
<pre><code>bq ls babynames&#x000A;</code></pre>
<p>出力:</p>
<pre><code class="language-bash prettyprint">  tableId    Type&#x000A; ----------- -------&#x000A;  names2010   TABLE&#x000A;</code></pre>
<p><code>「bq show」</code> と <code>「dataset.table」</code> を実行してスキーマを確認します。</p>
<pre><code>bq show babynames.names2010&#x000A;</code></pre>
<p>出力:</p>
<pre><code class="language-bash prettyprint">Table myprojectid:babynames.names2010&#x000A;&#x000A;   Last modified         Schema         Total Rows   Total Bytes   Expiration&#x000A; ----------------- ------------------- ------------ ------------- ------------&#x000A;  13 Mar 15:31:00   |- name: string     34041        653855&#x000A;                    |- gender: string&#x000A;                    |- count: integer&#x000A;</code></pre>
<ql-infobox>デフォルトでは、BigQuery は読み込まれるデータが UTF-8 エンコード データであるものと想定します。ISO-8859-1（または Latin-1）でエンコードされたデータがあり、読み込んだデータに問題がある場合は、<code>-E</code> フラグを使用して、Latin-1 としてデータを処理するように BigQuery に対して明示的に指定できます。詳細については、<a href="https://cloud.google.com/bigquery/bq-command-line-tool#characterencodings" target="_blank">文字エンコード</a>をご覧ください。
</ql-infobox>
<h2 id="step8">クエリを実行する</h2>
<p>これで、データをクエリして興味深い結果を返す準備が整いました。</p>
<p>次のコマンドを実行すると、女の子の名前で人気がある上位 5 つが返されます。</p>
<pre><code>bq query "SELECT name,count FROM babynames.names2010 WHERE gender = 'F' ORDER BY count DESC LIMIT 5"&#x000A;</code></pre>
<p>出力:</p>
<pre><code class="language-bash prettyprint">Waiting on job_58c0f5ca52764ef1902eba611b71c651 ... (0s) Current status: DONE&#x000A;+----------+-------+&#x000A;|   name   | COUNT |&#x000A;+----------+-------+&#x000A;| Isabella | 22898 |&#x000A;| Sophia   | 20634 |&#x000A;| Emma     | 17333 |&#x000A;| Olivia   | 17019 |&#x000A;| Ava      | 15424 |&#x000A;+----------+-------+&#x000A;</code></pre>
<p>次のコマンドを実行して、最も珍しい男の子の名前トップ 5 を確認します。</p>
<pre><code>bq query "SELECT name,count FROM babynames.names2010 WHERE gender = 'M' ORDER BY count ASC LIMIT 5"&#x000A;</code></pre>
<p><strong>注:</strong> ソースデータでは、出現回数が 5 回未満の名前が省略されるため、最小カウントは5です。（<code>LIMIT 5</code> の 5 は上位 5 件を表示するという意味の 5 であり、出現回数が 5 回未満が省略どうこうとはまた別の話）</p>
<p>出力:</p>
<pre><code class="language-bash prettyprint">Waiting on job_556ba2e5aad340a7b2818c3e3280b7a3 ... (1s) Current status: DONE&#x000A;+----------+-------+&#x000A;|   name   | COUNT |&#x000A;+----------+-------+&#x000A;| Amonti   |     5 |&#x000A;| Adyant   |     5 |&#x000A;| Ahmani   |     5 |&#x000A;| Anes     |     5 |&#x000A;| Alin     |     5 |&#x000A;+----------+-------+&#x000A;</code></pre>

<h2 id="step10">クリーンアップ</h2>
<p><code>-r</code>フラグを付けている<code>babynames</code>というデータセットを<code>bq rm</code>コマンドを実行して、取り除きます。そして、データセットですべてのテーブルを解除します。</p>
<pre><code>bq rm -r babynames&#x000A;</code></pre>
<p>解除のコマンドを確認ため、<strong>ｙ</strong>を入力します。</p>

# 

<h2 id="step4">BigQueryを開く</h2>
<p>BigQuery コンソールには、テーブルに対してクエリを実行するためのインターフェースが用意されており、BigQuery が提供する<a href="https://cloud.google.com/bigquery/public-data">一般公開データセット</a>も利用できます。</p>
<h3>BigQuery Console を開く</h3>
<p>Google Cloud Console で [<strong>ナビゲーション メニュー</strong>] &gt; [<strong>BigQuery</strong>] を選択します。</p>
<p><img alt="BigQuery_menu.png" src="https://cdn.qwiklabs.com/QwUX6algdXPgIThqBlgB2rnqGkoQUUANkl3xicD06uc%3D"></p>
<p>[<strong>Cloud Console の BigQuery へようこそ</strong>] メッセージボックスが開きます。このメッセージボックスには、クイックスタート ガイドとリリースノートへのリンクが表示されています。</p>
<p>[<strong>完了</strong>] をクリックします。</p>
<p>BigQuery コンソールが開きます。</p>
<p>
</p>
<p><img alt="bq-console.png" src="https://cdn.qwiklabs.com/vd3QNHGB4BAyHjAvOHw9qD0iqCPNaHAzY657z%2FGWLtY%3D"></p>

<h2 id="step5">公開データセットをクエリする</h2>
<p>このセクションでは、パブリックデータセットUSA NamesをBigQueryに読み込み、データセットをクエリして1910年から2013年の間に米国で最も一般的な名前を判断します。</p>
<h3>USA Nameデータセットを読み込む</h3>
<p>1. 左ペインで、[<strong>データを追加</strong>] &gt;  [<strong>一般公開データセットを調べる</strong>] をクリックします。</p>
<p><img alt="add-dataset.png" src="https://cdn.qwiklabs.com/KyiruOPDE7RgllidFqFTuHBmnL%2F4l%2Fe%2FEjZ%2BQymC5Wc%3D"></p>
<p>[データセット]ウィンドウが開きます。</p>
<p>2. 検索ボックスに「USA Names」と入力して入力します。</p>
<p>3. 検索結果に表示される<strong>USA Names</strong>タイルをクリックします</p>
<p><img alt="usa-names.png" src="https://cdn.qwiklabs.com/rOwZVSMvXuBMyJVOJpGa%2BX1GLNpViIvZ7dis%2BA4ffiA%3D"></p>
<p>4. <strong>データセットを表示</strong> をクリック</p>
<p>BigQueryが新しいブラウザタブに表示されます。プロジェクトbigquery-public-dataがリソースに追加され、Resourcesツリーの左ペインにデータセットusa_namesが一覧表示されます。</p>
<p><img alt="usa-names-dataset.png" src="https://cdn.qwiklabs.com/BALtI920nnVX5uSAP7DqFDopgDGSYN0whQsYDL3KHz8%3D"></p>
<h3>USA Nameデータセットを問い合わせる</h3>
<p>このデータセット内の赤ちゃんの名前と性別についてbigquery-public-data.usa_names.usa_1910_2013を照会してから、上位10名を降順でリストします。</p>
<p>1. 次のクエリをコピーして（クエリエディタ）テキスト領域に貼り付けます。</p>
<pre><code>SELECT&#x000A;  name, gender,&#x000A;  SUM(number) AS total&#x000A;FROM&#x000A;  `bigquery-public-data.usa_names.usa_1910_2013`&#x000A;GROUP BY&#x000A;  name, gender&#x000A;ORDER BY&#x000A;  total DESC&#x000A;LIMIT&#x000A;  10&#x000A;</code></pre>
<p>2. ウィンドウの右下に、クエリバリデータを表示します。</p>
<p><img alt="queryresult.png" src="https://cdn.qwiklabs.com/w4CbrHuw%2FbR2q8JmeJ7MWqym9rIXMyahmpNSAMNkStM%3D"></p>
<p>クエリが有効な場合、BigQueryは緑色のチェックマークアイコンを表示します。クエリが無効な場合は、赤い感嘆符アイコンが表示されます。クエリが有効な場合、バリデータはクエリを実行したときに処理されるデータ量も表示します。これは、クエリを実行するコストを判断するのに役立ちます。</p>
<p>3. <strong>実行 </strong>をクリックします。</p>
<p>クエリ結果がクエリエディタの下に表示されます。 [クエリ結果]セクションの上部に、B​​igQueryは経過時間とクエリによって処理されたデータを表示します。時間の下には、照会結果を表示する表があります。ヘッダー行には、クエリのGROUP BYで指定された列の名前が含まれています。</p>
<p><img alt="79dfab8f4004d7a7.png" src="https://cdn.qwiklabs.com/u%2B0%2FsSmu4twVfwREoLlV9NxlZQiz%2BueyYffr1hpRD7A%3D"></p>

<h2 id="step6">カスタム テーブルを作成する</h2>
<p>このセクションでは、カスタム テーブルを作成し、そこにデータを読み込んで、それに対してクエリを実行します。</p>
<h3>ローカルのパソコンにデータをダウンロードする</h3>
<p>ダウンロードするファイルには、米国社会保障局から提供された、人気のある赤ちゃんの名前に関する約 7 MB のデータが含まれています。</p>
<ol>
<li>
<p><a href="http://www.ssa.gov/OACT/babynames/names.zip">赤ちゃんの名前の zip ファイル</a>をローカルのパソコンにダウンロードします。</p>
</li>
<li>
<p>自分のパソコン上でファイルを解凍します。</p>
</li>
<li>
<p>この zip ファイルには、データセットについて説明した <code>NationalReadMe.pdf</code> ファイルが含まれています。<a href="http://www.ssa.gov/OACT/babynames/background.html">このデータセットの詳細を確認します</a>。</p>
</li>
<li>
<p><code>yob2014.txt</code> というファイルを開いて、データの内容を確認します。このファイルは、名前、性別（<code>M</code> または <code>F</code>）、およびその名前の子供の数を示す 3 つの列を含んだ、カンマ区切り値（CSV）ファイルです。このファイルにヘッダー行はありません。</p>
</li>
<li>
<p>後で確認できるように、<code>yob2014.txt</code> ファイルの場所をメモします。</p>
</li>
</ol>
<h2 id="step7">データセットを作成する</h2>
<p>このセクションでは、テーブルを格納するデータセットを作成し、プロジェクトにデータを追加して、クエリの対象となるデータテーブルを作成します。</p>
<p>データセットは、プロジェクト内のテーブルおよびビューへのアクセス制御に役立ちます。このラボではテーブルを 1 つしか使用しませんが、テーブルを格納するデータセットは必要です。</p>
<ol>
<li>コンソールに戻り、左側のパネルの [<strong>リソース</strong>] セクションで、該当する GCP プロジェクト ID（qwiklabs で始まる ID）をクリックします。</li>
</ol>
<p><img alt="bq-console.png" src="https://cdn.qwiklabs.com/6SeNR3lv9wLFe%2BxZc21E9ot%2Fby5zRiv1hY56s7tqqrA%3D"></p>
<p>クエリエディタの下にプロジェクトが表示されます。</p>
<ol start="2">
<li>
<p>右側のプロジェクト セクションで、[<strong>データセットを作成</strong>] をクリックします。</p>
</li>
<li>
<p>[<strong>データセットを作成</strong>] ページで次の操作を行います。</p>
</li>
</ol>
<ul>
<li>[<strong>データセット ID</strong>] に「<code>babynames</code>」と入力します。</li>
<li>[<strong>データのロケーション</strong>] で [<strong>米国（US）</strong>] を選択します。</li>
<li>[<strong>デフォルトのテーブルの有効期限</strong>] はデフォルト値のままにしておきます。</li>
</ul>
<p>現在、一般公開データセットは US マルチリージョン <a href="https://cloud.google.com/bigquery/docs/dataset-locations">ロケーション</a>に保存されています。わかりやすくするため、データセットを同じロケーションに配置します。</p>
<p><img alt="create-dataset.png" src="https://cdn.qwiklabs.com/cY3RLVckaR19518nzekwwCuDBLcRIj7xT0hoRKdGQ3Y%3D"></p>
<ol start="4">
<li>
<p>パネルの下部にある [<strong>データセットを作成</strong>] をクリックします。</p>
</li>
</ol>

<h2 id="step8">新しいテーブルにデータを読み込む</h2>
<p>このセクションでは、作成したテーブルにデータを読み込みます。</p>
<ol>
<li>左側のパネルの [<strong>リソース</strong>] セクションで [<strong>babynames</strong>] を見つけてクリックし、[<strong>テーブルを作成</strong>] をクリックします。</li>
</ol>
<p>別途指定のない限り、すべての設定にデフォルト値を使用します。</p>
<ol start="2">
<li>
<p>[<strong>テーブルの作成</strong>] ページで次の操作を行います。</p>
</li>
</ol>
<ul>
<li>
<p>[<strong>ソース</strong>] で [<strong>空のテーブル</strong>] をクリックし、プルダウン メニューで [<strong>アップロード</strong>] を選択します。</p>
</li>
<li>
<p>[<strong>ファイルを選択</strong>] で [<strong>参照</strong>] をクリックし、<code>yob2014.txt</code> ファイルを選択して [<strong>開く</strong>] をクリックします。</p>
</li>
<li>
<p>[<strong>ファイル形式</strong>] で [<strong>Avro</strong>] をクリックし、プルダウン メニューで [<strong>CSV</strong>] を選択します。</p>
</li>
<li>
<p>[<strong>テーブル名</strong>] に「<code>names_2014</code>」と入力します。</p>
</li>
<li>
<p>[<strong>スキーマ</strong>] セクションで [<strong>テキストとして編集</strong>] をクリックし、次のスキーマ定義をボックスに貼り付けます。</p>
</li>
</ul>
<pre><code>name:string,gender:string,count:integer&#x000A;</code></pre>
<p><img alt="create-table.png" src="https://cdn.qwiklabs.com/lGwpa5oXQD8ndm0ifPEBhex0MoGx%2BuzuluWpswRC6rU%3D"></p>
<ol start="3">
<li>
<p>[<strong>テーブルを作成</strong>] をクリックします（ウインドウの下部にあります）。</p>
</li>
<li>
<p>BigQuery によってテーブルが作成され、データが読み込まれるのを待ちます。BigQuery がデータを読み込んでいる間は、左側のパネルの [<strong>ジョブ履歴</strong>] の横に [<strong>（1 件が実行中）</strong>] という文字列が表示されます。データが読み込まれると、この文字列は消えます。</p>
</li>
</ol>
<h3>テーブルをプレビューする</h3>
<ol>
<li>左側のナビゲーション パネルで、[<strong>babynames</strong>] &gt; <strong>names_2014</strong> を選択します。</li>
<li>詳細パネルで [<strong>プレビュー</strong>] タブをクリックします。</li>
</ol>
<p><img alt="87469f59db3ee2ab.png" src="https://cdn.qwiklabs.com/K8qmzq%2Bv5GkyPNw3%2BmjeLQMt75wjZZWzIeNO7DZZ98M%3D"></p>

<h2 id="step9">テーブルに対してクエリを実行する</h2>
<p>テーブルにデータが読み込まれたので、クエリを実行できます。手順は前の例とまったく同じです。ただし今回は、一般公開テーブルではなく自分のテーブルに対してクエリを実行します。</p>
<ol>
<li>
<p>クエリエディタで、[<strong>クエリを新規作成</strong>] をクリックします。</p>
</li>
<li>
<p>次のクエリをコピーして、[<strong>クエリエディタ</strong>] テキスト ボックスに貼り付けます。このクエリは、2014 年に米国で人気が高かった男の子の名前、上位 5 つを取得します。</p>
</li>
</ol>
<pre><code>SELECT&#x000A; name, count&#x000A;FROM&#x000A; `babynames.names_2014`&#x000A;WHERE&#x000A; gender = 'M'&#x000A;ORDER BY count DESC LIMIT 5&#x000A;</code></pre>
<ol start="3">
<li>[<strong>実行</strong>] をクリックします。結果はクエリ ウィンドウの下に表示されます。</li>
</ol>
<p><img alt="a77eadeafdf06a84.png" src="https://cdn.qwiklabs.com/h04o8xozweSRtqn3MgPL2r78n3537TaJqCfqda6bq8c%3D"></p>
<h2 id="step10">これで完了です。</h2>

# Google BigQuery で SQL を使用して e コマース データセットを操作する
[この URL から（無料）](https://google.qwiklabs.com/focuses/3618?parent=catalog)。

# BigQuery でのよくある SQL エラーのトラブルシューティング
[この URL から（無料）](https://google.qwiklabs.com/focuses/3642?parent=catalog)

# データポータルを使ったデータ探索とレポート作成
[この URL から（無料）](https://google.qwiklabs.com/focuses/3614?parent=catalog)

# JOIN と UNION を使用してデータ ウェアハウスを構築する
<h2 id="step2">概要</h2>
<p><a href="http://bigquery.cloud.google.com/">BigQuery</a> は、Google が低料金で提供する NoOps のフルマネージド分析データベースです。インフラストラクチャを所有して管理したりデータベース管理者を配置したりすることなく、テラバイト単位の大規模なデータをクエリできます。BigQuery は SQL を採用しており、従量課金制モデルで利用できます。このような特徴を活かし、お客様は有用な情報を得るためのデータ分析に専念できます。</p>
<p>ここでは、<a href="https://shop.googlemerchandisestore.com/">Google Merchandise Store</a> に関する数百万件の Google アナリティクスのレコードを含み、BigQuery に読み込まれている <a href="https://www.en.advertisercommunity.com/t5/Articles/Introducing-the-Google-Analytics-Sample-Dataset-for-BigQuery/ba-p/1676331#">e コマースのデータセット</a>を使用します。このデータセットのコピーを使用して、フィールドや行からどのような分析情報が得られるのかを確認します。</p>
<p>このラボでは、SQL の JOIN と UNION を使用して新しいレポート テーブルを作成する方法について詳しく学習します。</p>
<h3>演習内容</h3>
<p>このラボでは、次のタスクの実行方法について学びます。</p>
<ul>
<li>
<p>新しい e コマースデータで感情分析を行う</p>
</li>
<li>
<p>データセットを結合して新しいテーブルを作成する</p>
</li>
<li>
<p>UNION とテーブル ワイルドカードを使用して履歴データを連結する</p>
</li>
</ul>

<h2 id="step3">BigQuery コンソール</h2>
<h3>BigQuery Console を開く</h3>
<p>Google Cloud Console で [<strong>ナビゲーション メニュー</strong>] &gt; [<strong>BigQuery</strong>] を選択します。</p>
<p><img alt="BigQuery_menu.png" src="https://cdn.qwiklabs.com/QwUX6algdXPgIThqBlgB2rnqGkoQUUANkl3xicD06uc%3D"></p>
<p>[<strong>Cloud Console の BigQuery へようこそ</strong>] メッセージボックスが開きます。このメッセージボックスには、クイックスタート ガイドとリリースノートへのリンクが表示されています。</p>
<p>[<strong>完了</strong>] をクリックします。</p>
<p>BigQuery コンソールが開きます。</p>
<p>
</p>
<p><img alt="bq-console.png" src="https://cdn.qwiklabs.com/vd3QNHGB4BAyHjAvOHw9qD0iqCPNaHAzY657z%2FGWLtY%3D"></p>

<h2 id="step4">テーブルを保存するための新しいデータセットを作成する</h2>
<p>BigQuery プロジェクトで、「<strong>ecommerce</strong>」という名前の新しいデータセットを作成します。</p>
<ol>
<li>左側のパネルの [<strong>リソース</strong>] で、BigQuery プロジェクト（<code>qwiklabs-gcp-xxxx</code>）をクリックします。</li>
<li>右側のクエリエディタで、[<strong>データセットを作成</strong>] をクリックします。</li>
</ol>
<p>[<strong>データセットを作成</strong>] ダイアログが開きます。</p>
<ol start="3">
<li>[<strong>データセット ID</strong>] に「<code>ecommerce</code>」と入力します。他のオプションはすべてデフォルト値のままにします。</li>
</ol>
<p><img alt="5fce8105cf381420.png" src="https://cdn.qwiklabs.com/VLtojAawcq9BhGzT7Bu%2FQstCbmn1j3hglRUI0XZfZjM%3D"></p>
<p>[<strong>データセットを作成</strong>] をクリックします。</p>
<p><strong>シナリオ</strong>: マーケティング チームから、e コマース ウェブサイトのすべての商品レビューが提供されました。あなたは、データ サイエンス チームと協力して、以下の 3 つのソースのデータを結合するデータ ウェアハウスを BigQuery で構築します。</p>
<ul>
<li>ウェブサイトの e コマースデータ</li>
<li>商品在庫のストックレベルとリードタイム</li>
<li>商品レビューの感情分析</li>
</ul>
<p>このラボでは、商品レビューに基づく新しいデータセットについて検討します。</p>
<h2 id="step5">プロジェクトをリソースツリーに固定する</h2>
<p>マーケティング チームの新しいデータセットを含むプロジェクトは、<strong>data-to-insights</strong> です。</p>
<p>BigQuery の一般公開データセットは、デフォルトでは BigQuery のウェブ UI には表示されません。<strong>data-to-insights</strong> は一般公開データセットのプロジェクトであるため、[リソース] のツリーに固定する必要があります。</p>
<ol>
<li>
<p>新しいブラウザ ウィンドウで、この一般公開データセットのプロジェクト（<a href="https://console.cloud.google.com/bigquery?p=data-to-insights&amp;page=ecommerce">https://console.cloud.google.com/bigquery?p=data-to-insights&amp;page=ecommerce</a>）を開きます。</p>
</li>
<li>
<p>左側のパネルの [リソース] で、[<strong>data-to-insights</strong>] をクリックします。右側のパネルで、[<strong>プロジェクトを固定</strong>] をクリックします。</p>
</li>
</ol>
<p><img alt="pin_project.png" src="https://cdn.qwiklabs.com/Aij0fKC0qe2yt%2BgH8tD2AgpQllVHBM27a2LEgZq3%2BgA%3D"></p>
<ol start="3">
<li>
<p>このブラウザ ウィンドウを閉じます。</p>
</li>
<li>
<p>最初の BigQuery のブラウザ ウィンドウに戻り、ページを更新します（これによりウェブ UI が更新されます）。</p>
</li>
</ol>
<p><code>data-to-insights</code> プロジェクトが [リソース] に表示されます。</p>
<h2 id="step6">e コマースデータを機械学習で拡充する</h2>
<p>このセクションでは、商品レビューの感情分析を行ううえで Google の感情分析 API がどのように機能するのかを見ていきます。</p>
<p>感情分析は、テキスト内で表現されている全体的な態度（ポジティブかネガティブか）を特定します。結果は、score と magnitude の数値によって表されます。</p>
<ul>
<li>
<em>score</em>: <code>-1.0</code>（ネガティブ）～<code>1.0</code>（ポジティブ）のスコアで感情が表されます。これは、テキストの全体的な感情の傾向に相当します。</li>
<li>
<em>magnitude</em>: 指定したテキストの全体的な感情の強度（ポジティブとネガティブの両方）が <code>0.0</code>～<code>+infinity</code> の値で示されます。score と違って magnitude は正規化されていないため、テキスト内で感情（ポジティブとネガティブの両方）が表現されるたびにテキストの magnitude の値が増加します（そのため、長いテキスト ブロックで値が高くなる傾向があります）。</li>
</ul>
<p>架空の商品レビューをいくつか使用して、感情分析 API がどのように機能するのかを確認します。</p>
<ol>
<li>ブラウザの新しいウィンドウまたはタブで <a href="https://cloud.google.com/natural-language/">Cloud Natural Language</a> のページを開きます。</li>
<li>下にスクロールして [<strong>Try the API</strong>] テキスト ボックスを見つけます。</li>
</ol>
<p><img alt="5eb2128531c7e948.png" src="https://cdn.qwiklabs.com/1YW3vIkcP7YfB7ztaaZxWUsf%2FKCZpCsxzksj81itKhY%3D">
3. テキスト ボックス内のテキストを次のテキストに置き換えます。</p>
<pre><code>I liked this flashlight at first. It worked ok for about a month but then just it just stopped working. So I'm quite disappointed with it.&#x000A;</code></pre>
<p><img alt="34aa0f0753f86cc7.png" src="https://cdn.qwiklabs.com/2p%2BXl957iU8MF10I6w5Sy9baHxEWGZk9M6KdoBHOum0%3D"></p>
<ol start="4">
<li>[<strong>Analyze</strong>] をクリックします。</li>
<li>[<strong>Sentiment</strong>] タブをクリックします。</li>
</ol>
<p><img alt="267c40bd24a7c953.png" src="https://cdn.qwiklabs.com/nE%2FeToQhhKDbnlgXm2HOKaQr7j16E3e6bVF9NWDYdrA%3D"></p>

<p>以下の商品レビューを分析します。後で質問に答えられるように Score の値を記録しておいてください。これらの商品レビューの中で感情が最もポジティブなもの、ネガティブなもの、中立的なものはそれぞれどれでしょうか。</p>
<p><strong>レビュー 1:</strong></p>
<pre><code>The three dog frisbees we ordered unfortunately didn't do well with our bigger German Shepherd dogs.&#x000A;</code></pre>
<p><strong>レビュー 2:</strong></p>
<pre><code>The three dog frisbees we ordered unfortunately didn't do well with our bigger German Shepherd dogs. Firstly, they had a tough time catching them in the air since the light blue models matched the color of the sky and secondly the material they were made out of wasn't strong enough to withstand more than a couple months of use before they got chewed up.&#x000A;</code></pre>
<p><strong>レビュー 3:</strong></p>
<pre><code>Honestly I've gone through quite a few umbrellas in the past but this new red Executive Umbrella is one of the best. We ended up going with red but you have a wide variety of colors to choose from. The umbrella material was excellent and didn't degrade after heavy use (we're in Seattle!).&#x000A;</code></pre>
<p><strong>レビュー 4:</strong></p>
<pre><code>I love these sunglasses. They are sturdy, look nice, and are functional. Highly recommended!&#x000A;</code></pre>
<p><strong>レビュー 5:</strong></p>
<pre><code>I got one of the microfleece jackets as a gift and wear it most days. The material is good and not itchy.&#x000A;</code></pre>
<p><strong>レビュー 6:</strong></p>
<pre><code>I pre-ordered a few yoga blocks but my shipment kept getting delayed because of supplier delays. Not sure what's going on there but would be great to speed up shipment.&#x000A;</code></pre>

<h2 id="step7">商品に対する感情のデータセットを調べる</h2>
<p>データ サイエンス チームは、すべての商品レビューに対して感情分析 API を実行し、各商品の感情スコア（score）と感情強度（magnitude）の平均値を調べました。</p>
<p>ここではそのデータについて検討します。</p>
<ol>
<li>
<strong>data-to-insight</strong> &gt; <strong>ecommerce</strong> &gt; <strong>products</strong> データセットに移動して、[<strong>プレビュー</strong>] タブをクリックします。データが表示されます。</li>
</ol>

<ol start="2">
<li>[<strong>スキーマ</strong>] タブをクリックします。</li>
</ol>

<h4>レビューの感情がポジティブだった商品をスコアの高い順に 5 つ表示するクエリを作成する</h4>
<p><strong>クエリエディタ</strong>に SQL クエリを入力します。</p>
<p>正解例:</p>
<pre><code>#standardSQL&#x000A;SELECT&#x000A;  SKU,&#x000A;  name,&#x000A;  sentimentScore,&#x000A;  sentimentMagnitude&#x000A;FROM&#x000A;  `data-to-insights.ecommerce.products`&#x000A;ORDER BY&#x000A;  sentimentScore DESC&#x000A;LIMIT 5&#x000A;</code></pre>

<p>上で入力したクエリを、レビューの感情がネガティブだった商品をスコアの低い順に 5 つ表示するように変更します。NULL 値は除外します。</p>
<p>正解例:</p>
<pre><code>#standardSQL&#x000A;SELECT&#x000A;  SKU,&#x000A;  name,&#x000A;  sentimentScore,&#x000A;  sentimentMagnitude&#x000A;FROM&#x000A;  `data-to-insights.ecommerce.products`&#x000A;WHERE sentimentScore IS NOT NULL&#x000A;ORDER BY&#x000A;  sentimentScore&#x000A;LIMIT 5&#x000A;</code></pre>

<h2 id="step8">データセットを結合して有用な情報を得る</h2>
<p>あなたは、月初めに在庫管理チームから、商品在庫データセットの <code>orderedQuantity</code> フィールドの値が古くなっているという知らせを受けました。2017 年 8 月 1 日の商品別の合計販売数を調べて現在のストックレベルと照合し、どの商品から補充すればよいかわかるようにする必要があります。</p>
<h4>productSKU 別の日次販売数を計算する</h4>
<p><strong>ecommerce</strong> データセットに新しいテーブルを作成します。要件は次のとおりです。</p>
<ul>
<li>
<code>sales_by_sku_20170801</code> という名前を付ける</li>
<li>データのソースに <code>data-to-insights.ecommerce.all_sessions_raw</code> を使用する</li>
<li>重複する結果は含めない</li>
<li>
<code>productSKU</code> を返す</li>
<li>合計注文数（<code>productQuantity</code>）を返す（ヒント: <code>SUM()</code> で IFNULL 条件を使用します）</li>
<li>
<code>20170801</code> の販売数のみを含める</li>
<li>
<code>ORDER BY</code> で SKU を注文数の多い順に並べる</li>
</ul>
<p>正解例:</p>
<pre><code># pull what sold on 08/01/2017&#x000A;CREATE OR REPLACE TABLE ecommerce.sales_by_sku_20170801 AS&#x000A;SELECT DISTINCT&#x000A;  productSKU,&#x000A;  SUM(IFNULL(productQuantity,0)) AS total_ordered&#x000A;FROM&#x000A;  `data-to-insights.ecommerce.all_sessions_raw`&#x000A;WHERE date = '20170801'&#x000A;GROUP BY productSKU&#x000A;ORDER BY total_ordered DESC #462 skus sold&#x000A;</code></pre>
<p>[<strong>プレビュー</strong>] タブをクリックします。販売された商品の SKU の数はいくつですか（重複したものはカウントしません）。</p>
<p>答え: 462</p>

<p>次に、この販売データを商品の在庫情報で拡充するために、2 つのデータセットを結合します。</p>
<h4>販売データと在庫データを結合する</h4>
<p>結合を使用して、ウェブサイトの e コマースデータを商品在庫データセットの以下のフィールドで拡充します。</p>
<ul>
<li><code>name</code></li>
<li><code>stockLevel</code></li>
<li><code>restockingLeadTime</code></li>
<li><code>sentimentScore</code></li>
<li><code>sentimentMagnitude</code></li>
</ul>
<p>部分的に作成済みの次のクエリを完成させてください。</p>
<pre><code># standardSQL&#x000A;# join against product inventory to get name&#x000A;SELECT DISTINCT&#x000A;  website.productSKU,&#x000A;  website.total_ordered,&#x000A;  inventory.name,&#x000A;  inventory.stockLevel,&#x000A;  inventory.restockingLeadTime,&#x000A;  inventory.sentimentScore,&#x000A;  inventory.sentimentMagnitude&#x000A;FROM&#x000A;  ecommerce.sales_by_sku_20170801 AS website&#x000A;  LEFT JOIN `data-to-insights.ecommerce.products` AS inventory&#x000A;&#x000A;ORDER BY total_ordered DESC&#x000A;</code></pre>
<p>正解例:</p>
<pre><code># standardSQL&#x000A;# join against product inventory to get name&#x000A;SELECT DISTINCT&#x000A;  website.productSKU,&#x000A;  website.total_ordered,&#x000A;  inventory.name,&#x000A;  inventory.stockLevel,&#x000A;  inventory.restockingLeadTime,&#x000A;  inventory.sentimentScore,&#x000A;  inventory.sentimentMagnitude&#x000A;FROM&#x000A;  ecommerce.sales_by_sku_20170801 AS website&#x000A;  LEFT JOIN `data-to-insights.ecommerce.products` AS inventory&#x000A;  ON website.productSKU = inventory.SKU&#x000A;ORDER BY total_ordered DESC&#x000A;</code></pre>
<p>上で記述したクエリをさらに次のように変更します。</p>
<ul>
<li>(<code>total_ordered / stockLevel</code>) の計算フィールドを追加して「<code>ratio</code>」というエイリアスを割り当てる（ヒント: ストックレベルが 0 の場合に 0 除算エラーが発生しないように <code>SAFE_DIVIDE(field1,field2)</code> を使用します）</li>
<li>結果をフィルタして、月初めにすでに在庫が 50% を切っている商品のみを含める</li>
</ul>
<p>正解例:</p>
<pre><code>#standardSQL&#x000A;# calculate ratio and filter&#x000A;SELECT DISTINCT&#x000A;  website.productSKU,&#x000A;  website.total_ordered,&#x000A;  inventory.name,&#x000A;  inventory.stockLevel,&#x000A;  inventory.restockingLeadTime,&#x000A;  inventory.sentimentScore,&#x000A;  inventory.sentimentMagnitude,&#x000A;&#x000A;  SAFE_DIVIDE(website.total_ordered, inventory.stockLevel) AS ratio&#x000A;FROM&#x000A;  ecommerce.sales_by_sku_20170801 AS website&#x000A;  LEFT JOIN `data-to-insights.ecommerce.products` AS inventory&#x000A;  ON website.productSKU = inventory.SKU&#x000A;&#x000A;# gone through more than 50% of inventory for the month&#x000A;WHERE SAFE_DIVIDE(website.total_ordered,inventory.stockLevel) &gt;= .50&#x000A;&#x000A;ORDER BY total_ordered DESC&#x000A;</code></pre>


<h2 id="step9">追加のレコードを連結する</h2>
<p>海外のチームの 2017 年 8 月 2 日の店舗販売データがすでにあるため、それを日次販売テーブルに記録する必要があります。</p>
<h4>2017 年 8 月 2 日の productSKU 別販売数を保存するための新しい空のテーブルを作成する</h4>
<p>スキーマで以下のフィールドを指定します。</p>
<ul>
<li>テーブル名: <code>ecommerce.sales_by_sku_20170802</code>
</li>
<li><code>productSKU STRING</code></li>
<li>
<code>total_ordered</code> <code>INT64</code>
</li>
</ul>
<p>正解例:</p>
<pre><code>#standardSQL&#x000A;CREATE OR REPLACE TABLE ecommerce.sales_by_sku_20170802&#x000A;(&#x000A;productSKU STRING,&#x000A;total_ordered INT64&#x000A;);&#x000A;</code></pre>
<p>日付を共有する販売テーブルが 2 つになったことを確認します（表示されない場合はブラウザ ウィンドウを更新してください）。</p>
<p><img alt="d569d313f9d0b3d.png" src="https://cdn.qwiklabs.com/rlbkLpDDuV1Hgm355AOsMWtGgEsMGkX4YCQqA6d8F8w%3D"></p>
<p>セールスチームから提供された販売レコードを挿入します。</p>
<pre><code>#standardSQL&#x000A;INSERT INTO ecommerce.sales_by_sku_20170802&#x000A;(productSKU, total_ordered)&#x000A;VALUES('GGOEGHPA002910', 101)&#x000A;</code></pre>
<p>テーブルをプレビューして、このレコードが表示されることを確認します。</p>
<h4>履歴データを連結する</h4>
<p>同じスキーマを持つデータを連結するにはさまざまな方法があります。中でもよく使われるのは、UNION を使用する方法と、テーブル ワイルドカードを使用する方法の 2 つです。</p>
<ul>
<li>
<strong>UNION</strong> は SQL 演算子のひとつで、複数の結果セットの行を連結します。</li>
<li>
<strong>テーブル ワイルドカード</strong>を使用すると、簡潔な SQL ステートメントを使用して複数のテーブルをクエリできます。ワイルドカード テーブルは標準 SQL でのみ使用できます。</li>
</ul>
<p>以下の 2 つのテーブルのすべてのレコードを返す UNION クエリを記述します。</p>
<ul>
<li>
<p><code>ecommerce.sales_by_sku_20170801</code></p>
</li>
<li>
<p><code>ecommerce.sales_by_sku_20170802</code></p>
</li>
</ul>
<pre><code>#standardSQL&#x000A;SELECT * FROM ecommerce.sales_by_sku_20170801&#x000A;UNION ALL&#x000A;SELECT * FROM ecommerce.sales_by_sku_20170802&#x000A;</code></pre>
<p>注: <code>UNION</code> と <code>UNION ALL</code> の違いは、<code>UNION</code> では重複レコードが含まれないことです。</p>
<p>日次販売テーブルの数が増えるとどのような問題が起こりますか。</p>
<p>答え: 互いに連結された多数の <code>UNION</code> ステートメントを記述しなければならなくなります。</p>
<p>この問題を解決するには、テーブル ワイルドカード フィルタと <code>_TABLE_SUFFIX</code> フィルタを使用します。</p>
<p>テーブル ワイルドカード（*）を使用して、2017 年の <code>ecommerce.sales_by_sku_</code> テーブルのすべてのレコードを選択するクエリを記述します。</p>
<p>正解例:</p>
<pre><code>#standardSQL&#x000A;SELECT * FROM `ecommerce.sales_by_sku_2017*`&#x000A;</code></pre>
<p>前のクエリを変更して、結果を 2017 年 8 月 2 日のレコードのみに制限するフィルタを追加します。</p>
<p>正解例:</p>
<pre><code>#standardSQL&#x000A;SELECT * FROM `ecommerce.sales_by_sku_2017*`&#x000A;WHERE _TABLE_SUFFIX = '0802'&#x000A;</code></pre>
<p>注: 分割テーブルを作成して、日次販売データが自動的に正しいパーティションに取り込まれるようにする方法もあります。</p>

## まとめ
<p>これでこのハンズオンラボは終了です。ここでは、サンプル e コマースデータを調べるためにレポート テーブルを作成し、SQL の JOIN と UNION を使用してビューを操作しました。</p>

# BigQuery で日付分割テーブルを作成する
<h3>BigQuery Console を開く</h3>
<p>Google Cloud Console で [<strong>ナビゲーション メニュー</strong>] &gt; [<strong>BigQuery</strong>] を選択します。</p>
<p><img alt="BigQuery_menu.png" src="https://cdn.qwiklabs.com/QwUX6algdXPgIThqBlgB2rnqGkoQUUANkl3xicD06uc%3D"></p>
<p>[<strong>Cloud Console の BigQuery へようこそ</strong>] メッセージボックスが開きます。このメッセージボックスには、クイックスタート ガイドとリリースノートへのリンクが表示されています。</p>
<p>[<strong>完了</strong>] をクリックします。</p>
<p>BigQuery コンソールが開きます。</p>
<p>
</p>
<p><img alt="bq-console.png" src="https://cdn.qwiklabs.com/vd3QNHGB4BAyHjAvOHw9qD0iqCPNaHAzY657z%2FGWLtY%3D"></p>

<h2 id="step4">新しいデータセットを作成する</h2>
<p>まず、テーブルを保存するためのデータセットを作成します。</p>
<p>プロジェクト名をクリックし、[<strong>データセットを作成</strong>] をクリックします。</p>
<p><img alt="5fce8105cf381420.png" src="https://cdn.qwiklabs.com/VLtojAawcq9BhGzT7Bu%2FQstCbmn1j3hglRUI0XZfZjM%3D"></p>
<p>データセットに <strong>ecommerce</strong> という名前を付けます。その他のオプションはデフォルト値のままにします（[データのロケーション]、[デフォルトのテーブルの有効期限]）。</p>
<p>[<strong>データセットを作成</strong>] をクリックします。</p>
<p>[<strong>進行状況を確認</strong>] をクリックして、目標に沿って進んでいることを確認します。</p>

<h2 id="step5">日付パーティションを持つテーブルを作成する</h2>
<p>分割テーブルは、パーティションと呼ばれるセグメントに分割されたテーブルで、データの管理やクエリをより簡単に行えるという利点があります。大きなテーブルを小さなパーティションに分割することで、クエリのパフォーマンスを高めたり、クエリによって読み取られるバイト数を減らしてコストを抑えたりすることができます。</p>
<p>ここでは、新しいテーブルを作成し、日付またはタイムスタンプの列をパーティションとしてバインドします。その前に、分割されていないテーブルのデータを調べてみましょう。</p>
<h3>ウェブページ分析データで 2017 年の訪問者のサンプルをクエリする</h3>
<p><strong>クエリエディタ</strong>に以下のクエリを追加します。実行する前に、処理されるデータの総量を確認します。クエリ検証ツールのアイコンの横に、「このクエリを実行すると、1.74 GB が処理されます」と表示されます。</p>
<pre><code>#standardSQL&#x000A;SELECT DISTINCT&#x000A;  fullVisitorId,&#x000A;  date,&#x000A;  city,&#x000A;  pageTitle&#x000A;FROM `data-to-insights.ecommerce.all_sessions_raw`&#x000A;WHERE date = '20170708'&#x000A;LIMIT 5&#x000A;</code></pre>
<p>[<strong>実行</strong>] をクリックします。</p>
<p>このクエリは 5 件の結果を返します。</p>
<h3>ウェブページ分析データで 2018 年の訪問者のサンプルをクエリする</h3>
<p>次に、このクエリを変更して 2018 年の訪問者を調べてみましょう。</p>
<p>[<strong>クエリエディタ</strong>]ーを削除するため、[<strong>クエリを新規作成</strong>]をくりっくして、新しいクエリを追加します。WHERE dateが２０１８０７０８に変更します。</p>
<pre><code>#standardSQL&#x000A;SELECT DISTINCT&#x000A;  fullVisitorId,&#x000A;  date,&#x000A;  city,&#x000A;  pageTitle&#x000A;FROM `data-to-insights.ecommerce.all_sessions_raw`&#x000A;WHERE date = '20180708'&#x000A;LIMIT 5&#x000A;</code></pre>
<p><strong>クエリ検証ツール</strong>により、処理されるデータの量が表示されます。</p>
<p>[<strong>実行</strong>] をクリックします。</p>
<p>返される結果は 0 件なのに、処理されるデータの量は 1.74 GB で前のクエリと変わりません。なぜでしょう。それは、クエリエンジンがデータセット内のすべてのレコードをスキャンして、WHERE 句の日付に一致しているかどうかを確認する必要があるためです。レコードの日付をひとつひとつ '20180708' という条件と比較しなければなりません。</p>
<p>なお、よく誤解されていますが、LIMIT 5 を指定しても処理されるデータの総量が減ることはありません。</p>

<h4>日付分割テーブルの一般的なユースケース</h4>
<p>行を WHERE 条件と比較するために毎回データセット全体をスキャンするのは、無駄の多い作業です。以下のように特定の期間のレコードのみを対象とする場合は特にそうです。</p>
<ul>
<li>昨年のすべてのトランザクション</li>
<li>過去 7 日間のすべてのユーザー インタラクション</li>
<li>先月販売したすべての商品</li>
</ul>
<p>今度は、前のクエリのようにデータセット全体をスキャンして日付フィールドでフィルタする代わりに、日付分割テーブルを作成します。これにより、クエリに関係のないパーティションのレコードは一切スキャンする必要がなくなります。</p>
<h4>日付に基づいて新しい分割テーブルを作成する</h4>
<p>[<strong>クエリを新規作成</strong>] をクリックし、以下のクエリを追加して [<strong>実行</strong>] をクリックします。</p>
<pre><code>#standardSQL&#x000A; CREATE OR REPLACE TABLE ecommerce.partition_by_day&#x000A; PARTITION BY date_formatted&#x000A; OPTIONS(&#x000A;   description="a table partitioned by date"&#x000A; ) AS&#x000A; SELECT DISTINCT&#x000A; PARSE_DATE("%Y%m%d", date) AS date_formatted,&#x000A; fullvisitorId&#x000A; FROM `data-to-insights.ecommerce.all_sessions_raw`&#x000A;</code></pre>
<p>このクエリには、PARTITION BY &lt;a field&gt; という新しいオプションが追加されています。パーティショニングに使用できるデータ型は DATE と TIMESTAMP の 2 つです。ここでは、文字列として保存されている日付フィールドをパーティショニングに適した DATE 型に変換するために、PARSE_DATE 関数が使用されています。</p>
<p><strong>ecommerce</strong> データセットをクリックし、新しい <strong>partiton_by_day</strong> テーブルを選択します。</p>
<p><img alt="f15327a7d4da4db9.png" src="https://cdn.qwiklabs.com/MOKf1mKRDJjhlD0WxypQMvRsMV93A4PPOjC4aD%2BcpgM%3D"></p>
<p>[<strong>詳細</strong>] タブをクリックします。</p>
<p>次のようになっていることを確認します。</p>
<ul>
<li>分割基準: Day</li>
<li>フィールドで分割: date_formatted</li>
</ul>
<p><img alt="BQ_partbyday_details.png" src="https://cdn.qwiklabs.com/wG%2BZE0Q0eUDjzxZgnlk4J4DUHqbe4eByeL6UR1EIxdM%3D"></p>
<aside>
注: Qwiklabs アカウントの分割テーブル内のパーティションは、日付列の値から 60 日後に自動的に期限切れになります。課金を有効にした個人の GCP アカウントでは、期限切れにならない分割テーブルを作成できます。このラボでこれから扱うクエリは、すでに作成されている分割テーブルに対して実行します。

</aside>

<h2 id="step6">分割テーブルを使用して処理されたデータを表示する</h2>
<p>以下のクエリを実行します。処理される合計バイト数を確認してください。</p>
<pre><code>#standardSQL&#x000A;SELECT *&#x000A;FROM `data-to-insights.ecommerce.partition_by_day`&#x000A;WHERE date_formatted = '2016-08-01'&#x000A;</code></pre>
<p>処理されるバイト数が約 25 KB（0.025 MB）になりました。これは、前のクエリに比べるとごくわずかです。</p>
<p>次に、以下のクエリを実行します。処理される合計バイト数を確認してください。</p>
<pre><code>#standardSQL&#x000A;SELECT *&#x000A;FROM `data-to-insights.ecommerce.partition_by_day`&#x000A;WHERE date_formatted = '2018-07-08'&#x000A;</code></pre>
<p>「このクエリを実行すると、0 B が処理されます<code></code>」と表示されます。</p>
<p>処理されるバイト数が 0 バイトになるのはなぜでしょうか。</p>
<p>クエリエンジンは既存のパーティションを把握しており、2018-07-08 に対応するパーティションは存在しないことがわかっているからです（ecommerce データセットのデータの範囲は 2016-08-01 から 2017-08-01 までです）。</p>

<h2 id="step7">自動的に期限切れになる分割テーブルを作成する</h2>
<p>自動的に期限切れになる分割テーブルは、データ プライバシーに関する法令を遵守するために使用されます。ストレージの不必要な浪費を防ぐために使用することもできます（本番環境ではコストの節約になります）。データのローリング ウィンドウを作成するには、使い終わったパーティションが自動的に消去されるように有効期限を追加します。</p>
<h3>公開されている NOAA 気象データのテーブルを調べる</h3>
<p>左側のメニューの [リソース] で、[<strong>データを追加</strong>] をクリックして [<strong>一般公開データセットを調べる</strong>] を選択します。</p>
<p><img alt="fe6252fb6b824196.png" src="https://cdn.qwiklabs.com/Sdj%2F9%2BxXNN4eDPRO5LdtBglwmXSY99B%2Fu11SzxBjqhg%3D"></p>
<p>「GSOD NOAA」を検索して、このデータセットを選択します。</p>
<p>[<strong>データセットを表示</strong>] をクリックします。</p>
<p><strong>noaa_gsod</strong> データセットのテーブルのリストを<strong>スクロール</strong>します（これらのテーブルは分割テーブルではなく、手動でシャーディングされています）。</p>
<p><img alt="8537f8afdbf8824a.png" src="https://cdn.qwiklabs.com/sB6M27wRVkHxmhHs8oTMDORQVOYcRJABzdzzrtCAlyc%3D"></p>
<p>ここでの目標は、次のようなテーブルを作成することです。</p>
<ul>
<li>2018 年以降の気象データに対してクエリを実行する</li>
<li>降水量（雨、雪など）が観測された日のみを含める</li>
<li>各パーティションの保存期間を 90 日に制限する（ローリング ウィンドウ）</li>
</ul>
<p>まず、以下のクエリを<strong>コピーして貼り付けます</strong>。</p>
<pre><code>#standardSQL&#x000A; SELECT&#x000A;   DATE(CAST(year AS INT64), CAST(mo AS INT64), CAST(da AS INT64)) AS date,&#x000A;   (SELECT ANY_VALUE(name) FROM `bigquery-public-data.noaa_gsod.stations` AS stations&#x000A;    WHERE stations.usaf = stn) AS station_name,  -- Stations may have multiple names&#x000A;   prcp&#x000A; FROM `bigquery-public-data.noaa_gsod.gsod*` AS weather&#x000A; WHERE prcp &lt; 99.9  -- Filter unknown values&#x000A;   AND prcp &gt; 0      -- Filter stations/days with no precipitation&#x000A;   AND CAST(_TABLE_SUFFIX AS int64) &gt;= 2018&#x000A; ORDER BY date DESC -- Where has it rained/snowed recently&#x000A; LIMIT 10&#x000A;</code></pre>
<p><em>TABLE_SUFFIX</em> フィルタで参照されるテーブルの数を制限するために、FROM 句でテーブル ワイルドカード* が使用されています。</p>
<p>LIMIT 10 が追加されていますが、まだパーティションがないため、スキャンされるデータの総量は減りません（約 1.83 GB）。</p>
<p>[<strong>実行</strong>] をクリックします。</p>
<p>日付の形式が正しいこと、降水量のフィールドに 0 の値がないことを確認します。</p>
<h2 id="step8">実習: 分割テーブルを作成する</h2>
<p>前のクエリを変更してテーブルを作成します。次のように指定します。</p>
<ul>
<li>テーブル名: ecommerce.days_with_rain</li>
<li>PARTITION BY に date フィールドを使用</li>
<li>OPTIONS に partition_expiration_days = 60 を指定</li>
<li>description = "weather stations with precipitation, partitioned by day" を追加</li>
</ul>
<p>変更したクエリは次のようになります。</p>
<pre><code>#standardSQL&#x000A; CREATE OR REPLACE TABLE ecommerce.days_with_rain&#x000A; PARTITION BY date&#x000A; OPTIONS (&#x000A;   partition_expiration_days=60,&#x000A;   description="weather stations with precipitation, partitioned by day"&#x000A; ) AS&#x000A; SELECT&#x000A;   DATE(CAST(year AS INT64), CAST(mo AS INT64), CAST(da AS INT64)) AS date,&#x000A;   (SELECT ANY_VALUE(name) FROM `bigquery-public-data.noaa_gsod.stations` AS stations&#x000A;    WHERE stations.usaf = stn) AS station_name,  -- Stations may have multiple names&#x000A;   prcp&#x000A; FROM `bigquery-public-data.noaa_gsod.gsod*` AS weather&#x000A; WHERE prcp &lt; 99.9  -- Filter unknown values&#x000A;   AND prcp &gt; 0      -- Filter&#x000A;   AND CAST(_TABLE_SUFFIX AS int64) &gt;= 2018&#x000A;</code></pre>

<h4>データ パーティションの有効期限が機能していることを確認する</h4>
<p>60 日前より古いデータが保存されていないことを確認するために、DATE_DIFF クエリを実行してパーティションの経過日数を取得します。パーティションは 60 日後に期限切れになるように設定されています。</p>
<p>以下のクエリは、非常に降水量の多い<a href="https://en.wikipedia.org/wiki/Wakayama,_Wakayama#Climate">和歌山市</a>にある NOAA の気象観測所の平均降水量を追跡します。</p>
<p>このクエリを追加して実行します。</p>
<pre><code>#standardSQL&#x000A;# avg monthly precipitation&#x000A;SELECT&#x000A;  AVG(prcp) AS average,&#x000A;  station_name,&#x000A;  date,&#x000A;  CURRENT_DATE() AS today,&#x000A;  DATE_DIFF(CURRENT_DATE(), date, DAY) AS partition_age,&#x000A;  EXTRACT(MONTH FROM date) AS month&#x000A;FROM ecommerce.days_with_rain&#x000A;WHERE station_name = 'WAKAYAMA' #Japan&#x000A;GROUP BY station_name, date, today, month, partition_age&#x000A;ORDER BY date DESC; # most recent days first&#x000A;</code></pre>
<h2 id="step9">partition_age が 60 日を超えていないことを確認する</h2>
<p>ORDER BY 句を更新して、パーティションを古い順に表示します。以下のクエリを追加して実行します。</p>
<pre><code>#standardSQL&#x000A;# avg monthly precipitation&#x000A;&#x000A;SELECT&#x000A;  AVG(prcp) AS average,&#x000A;  station_name,&#x000A;  date,&#x000A;  CURRENT_DATE() AS today,&#x000A;  DATE_DIFF(CURRENT_DATE(), date, DAY) AS partition_age,&#x000A;  EXTRACT(MONTH FROM date) AS month&#x000A;FROM ecommerce.days_with_rain&#x000A;WHERE station_name = 'WAKAYAMA' #Japan&#x000A;GROUP BY station_name, date, today, month, partition_age&#x000A;ORDER BY partition_age DESC&#x000A;</code></pre>
<aside>
注: 気象データ（およびこれらのパーティション）は継続的に更新されるため、将来このクエリを再実行すると結果が変わります。

</aside>
<h2 id="step10">まとめ</h2>
<p>ここでは、BigQuery で分割テーブルを作成してクエリを実行しました。</p>

# BigQuery での JSON、配列、構造体の操作
<h2 id="step2">概要</h2>
<p><a href="http://bigquery.cloud.google.com/">BigQuery</a> は、Google が低価格で提供する NoOps、フルマネージドの分析データベースです。インフラストラクチャを所有して管理したり、データベース管理者を置いたりすることなく、テラバイト単位の大規模なデータでクエリを実行できます。また、SQL を使用することができ、お支払いモデルは従量課金制となります。このような利点を活かし、ユーザーは有用な情報を得るためのデータの分析に専念することができます。</p>
<p>このラボでは、BigQuery での半構造化データの操作（JSON や配列データ型の取り込み）について詳しく学習します。スキーマを非正規化し、ネストされた繰り返しフィールドを持つ単一のテーブルにすることで、パフォーマンスが改善する場合があります。ただし、配列データを操作する SQL 構文は複雑になることがあります。ここでは、さまざまな半構造化データセットに対する読み込み、クエリ実行、トラブルシューティング、ネスト解除を実際に行います。</p>

<h3>BigQuery Console を開く</h3>
<p>Google Cloud Console で [<strong>ナビゲーション メニュー</strong>] &gt; [<strong>BigQuery</strong>] を選択します。</p>
<p><img alt="BigQuery_menu.png" src="https://cdn.qwiklabs.com/QwUX6algdXPgIThqBlgB2rnqGkoQUUANkl3xicD06uc%3D"></p>
<p>[<strong>Cloud Console の BigQuery へようこそ</strong>] メッセージボックスが開きます。このメッセージボックスには、クイックスタート ガイドとリリースノートへのリンクが表示されています。</p>
<p>[<strong>完了</strong>] をクリックします。</p>
<p>BigQuery コンソールが開きます。</p>
<p>
</p>
<p><img alt="bq-console.png" src="https://cdn.qwiklabs.com/vd3QNHGB4BAyHjAvOHw9qD0iqCPNaHAzY657z%2FGWLtY%3D"></p>

<h2 id="step4">テーブルを保存するための新しいデータセットを作成する</h2>
<p>BigQuery で対象のプロジェクト名をクリックし、[<strong>データセットを作成</strong>] をクリックします。</p>
<p><img alt="5fce8105cf381420.png" src="https://cdn.qwiklabs.com/VLtojAawcq9BhGzT7Bu%2FQstCbmn1j3hglRUI0XZfZjM%3D"></p>
<p>新しいデータセットに「<code>fruit_store</code>」という名前を付けます。その他のオプション（[データのロケーション]、[デフォルトのテーブルの有効期限]）はデフォルト値のままにします。[<strong>データセットを作成</strong>] をクリックします。</p>
<h2 id="step5">SQL で配列を操作する</h2>
<p>通常、SQL では、以下の果物リストのように各行に値が 1 つ含まれます。</p>
<table>
<tr>
<td colspan="1" rowspan="1">
<p><strong>行</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>果物</strong></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>1</p>
</td>
<td colspan="1" rowspan="1">
<p>raspberry</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>2</p>
</td>
<td colspan="1" rowspan="1">
<p>blackberry</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>3</p>
</td>
<td colspan="1" rowspan="1">
<p>strawberry</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>4</p>
</td>
<td colspan="1" rowspan="1">
<p>cherry</p>
</td>
</tr>
</table>
<p>果物リストに店舗の担当者名が必要な場合はどうすればよいでしょうか。次のようになります。</p>
<table>
<tr>
<td colspan="1" rowspan="1">
<p><strong>行</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>果物</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>担当者</strong></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>1</p>
</td>
<td colspan="1" rowspan="1">
<p>raspberry</p>
</td>
<td colspan="1" rowspan="1">
<p>sally</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>2</p>
</td>
<td colspan="1" rowspan="1">
<p>blackberry</p>
</td>
<td colspan="1" rowspan="1">
<p>sally</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>3</p>
</td>
<td colspan="1" rowspan="1">
<p>strawberry</p>
</td>
<td colspan="1" rowspan="1">
<p>sally</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>4</p>
</td>
<td colspan="1" rowspan="1">
<p>cherry</p>
</td>
<td colspan="1" rowspan="1">
<p>sally</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>5</p>
</td>
<td colspan="1" rowspan="1">
<p>orange</p>
</td>
<td colspan="1" rowspan="1">
<p>frederick </p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>6</p>
</td>
<td colspan="1" rowspan="1">
<p>apple</p>
</td>
<td colspan="1" rowspan="1">
<p>frederick</p>
</td>
</tr>
</table>
<p>従来のリレーショナル データベースの SQL では、同じ名前が複数回出現する場合、上記のテーブルを果物と担当者の 2 つの別個のテーブルに分割することを考えるでしょう。これを<a href="https://ja.wikipedia.org/wiki/%E9%96%A2%E4%BF%82%E3%81%AE%E6%AD%A3%E8%A6%8F%E5%8C%96" target="_blank">正規化</a>と呼びます（1 つのテーブルを多数のテーブルに分割）。mySQL のようなトランザクション データベースでよく行われます。</p>
<p>データ ウェアハウジングでよく行われるのはその逆の操作（非正規化）で、多数のテーブルを 1 つの大きなレポート テーブルにまとめます。</p>

<p>ここでは、繰り返しフィールドを使用して、粒度の異なるデータをすべて 1 つのテーブルに格納する方法を学びます。</p>
<table>
<tr>
<td colspan="1" rowspan="1">
<p><strong>行</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>果物（配列）</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>担当者</strong></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="4">
<p>1</p>
</td>
<td colspan="1" rowspan="1">
<p>raspberry</p>
</td>
<td colspan="1" rowspan="1">
<p>sally</p>
</td>
</tr>
<tr><td colspan="1" rowspan="1">
<p>blackberry</p>
</td></tr>
<tr><td colspan="1" rowspan="1">
<p>strawberry</p>
</td></tr>
<tr><td colspan="1" rowspan="1">
<p>cherry</p>
</td></tr>
<tr>
<td colspan="1" rowspan="2">
<p>2</p>
</td>
<td colspan="1" rowspan="1">
<p>orange</p>
</td>
<td colspan="1" rowspan="1">
<p>frederick </p>
</td>
</tr>
<tr><td colspan="1" rowspan="1">
<p>apple</p>
</td></tr>
</table>
<p>上のテーブルの不自然な点はどこでしょうか。</p>
<ul>
<li>行が 2 つだけである。</li>
<li>[果物] 列では、1 つの行に複数のフィールド値がある。</li>
<li>担当者がすべてのフィールド値に関連付けられている。</li>
</ul>
<p>ここからわかるのは、<code>array</code> データ型が使用されているということです。</p>
<p>以下のように記述すると、果物の配列について理解しやすくなります。</p>
<table>
<tr>
<td colspan="1" rowspan="1">
<p><strong>行</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>果物（配列）</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>担当者</strong></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>1</p>
</td>
<td colspan="1" rowspan="1">
<p>[raspberry, blackberry, strawberry, cherry]</p>
</td>
<td colspan="1" rowspan="1">
<p>sally</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>2</p>
</td>
<td colspan="1" rowspan="1">
<p>[orange, apple]</p>
</td>
<td colspan="1" rowspan="1">
<p>frederick </p>
</td>
</tr>
</table>
<p>これら両方のテーブルは同じ内容を表します。主な学習のポイントは 2 つあります。</p>
<ul>
<li>配列は単純に [ ] 内の項目のリストである</li>
<li>BigQuery では配列が「フラット化<em></em>」されて表示され、配列の値が単純に一列にリスト化される（ここでも値は単一の行に含まれる）</li>
</ul>
<p>実際に試してみましょう。BigQuery のクエリエディタに次のクエリを入力します。</p>
<pre><code class="language-sql prettyprint">#standardSQL&#x000A;SELECT&#x000A;['raspberry', 'blackberry', 'strawberry', 'cherry'] AS fruit_array&#x000A;</code></pre>
<p>[<strong>実行</strong>] をクリックします。</p>
<p>次のクエリを実行します。</p>
<pre><code class="language-sql prettyprint">#standardSQL&#x000A;SELECT&#x000A;['raspberry', 'blackberry', 'strawberry', 'cherry', 1234567] AS fruit_array&#x000A;</code></pre>
<p>次のようなエラーが表示されます。</p>
<p><strong>Error:</strong> <code>Array elements of types {INT64, STRING} do not have a common supertype at [3:1]</code></p>

<p>配列内では同じデータ型を使用する必要があります（すべて文字列、すべて数値など）。</p>
<p>最後に、テーブルに対して次のクエリを実行します。</p>
<pre><code class="language-sql prettyprint">#standardSQL&#x000A;SELECT person, fruit_array, total_cost FROM `data-to-insights.advanced.fruit_store`;&#x000A;</code></pre>
<p>[<strong>実行</strong>] をクリックします。</p>
<p>結果が表示されたら [<strong>JSON</strong>] タブをクリックして、ネストされた結果の構造を確認します。</p>
<p><img alt="e66b02ee06dd462.png" src="https://cdn.qwiklabs.com/V0j%2FsiBB1RlvfpG6x%2Bl9P6xn5YaSKjcRwasQde5WRTs%3D"></p>
<h3>半構造化 JSON を BigQuery に読み込む</h3>
<p>BigQuery に JSON ファイルを取り込む必要がある場合はどうすればよいでしょうか。</p>
<p>データセットで [<code>fruit_details</code>] をクリックします。</p>
<p>次のようにテーブルの詳細を設定します。</p>
<ul>
<li>[<strong>ソース</strong>]: [<strong>テーブルの作成元</strong>] プルダウンで [<strong>Google Cloud Storage</strong>] を選択します。</li>
<li>[<strong>GCS バケットからファイルを選択</strong>]: <code>gs://cloud-training/gsp416/shopping_cart.json</code>
</li>
<li>[<strong>ファイル形式</strong>]: JSON（改行区切り）</li>
</ul>
<p>新しいテーブルの名前を「<code>fruit_details</code>」にします。</p>
<p>[<strong>スキーマと入力パラメータ</strong>] のチェックボックスをオンにします。</p>
<p>[<strong>テーブルを作成</strong>] をクリックします。</p>
<p>[スキーマ] で <code>fruit_array</code> が「REPEATED」に設定されているため、このフィールドが配列であることがわかります。</p>
<p><strong>内容のまとめ</strong></p>
<ul>
<li>BigQuery は配列をネイティブにサポートする</li>
<li>配列値のデータ型はすべて同じでなければならない</li>
<li>BigQuery では配列を繰り返しフィールド（REPEATED）と呼ぶ</li>
</ul>

<h2 id="step6">ARRAY_AGG() を使用して独自の配列を作成する</h2>
<p>今度は自分で配列を作成してみましょう。</p>
<p>以下のクエリを<strong>コピーして貼り付けて</strong>、この一般公開データセットを探索します。</p>
<pre><code class="language-sql prettyprint">SELECT&#x000A;  fullVisitorId,&#x000A;  date,&#x000A;  v2ProductName,&#x000A;  pageTitle&#x000A;  FROM `data-to-insights.ecommerce.all_sessions`&#x000A;WHERE visitId = 1501570398&#x000A;ORDER BY date&#x000A;</code></pre>
<p>[<strong>実行</strong>] をクリックして結果を確認します。</p>

<p>次に、<code>ARRAY_AGG()</code> 関数を使用して、これらの文字列値を 1 つの配列にまとめます。</p>
<p>以下のクエリを<strong>コピーして貼り付けて</strong>、この一般公開データセットを探索します。</p>
<pre><code class="language-sql prettyprint">SELECT&#x000A;  fullVisitorId,&#x000A;  date,&#x000A;  ARRAY_AGG(v2ProductName) AS products_viewed,&#x000A;  ARRAY_AGG(pageTitle) AS pages_viewed&#x000A;  FROM `data-to-insights.ecommerce.all_sessions`&#x000A;WHERE visitId = 1501570398&#x000A;GROUP BY fullVisitorId, date&#x000A;ORDER BY date&#x000A;</code></pre>
<p>[<strong>実行</strong>] をクリックして結果を確認します。</p>

<p>次に、<code>ARRAY_LENGTH()</code> 関数を使用して、閲覧されたページと商品の数を調べます。</p>
<pre><code class="language-sql prettyprint">SELECT&#x000A;  fullVisitorId,&#x000A;  date,&#x000A;  ARRAY_AGG(v2ProductName) AS products_viewed,&#x000A;  ARRAY_LENGTH(ARRAY_AGG(v2ProductName)) AS num_products_viewed,&#x000A;  ARRAY_AGG(pageTitle) AS pages_viewed,&#x000A;  ARRAY_LENGTH(ARRAY_AGG(pageTitle)) AS num_pages_viewed&#x000A;  FROM `data-to-insights.ecommerce.all_sessions`&#x000A;WHERE visitId = 1501570398&#x000A;GROUP BY fullVisitorId, date&#x000A;ORDER BY date&#x000A;</code></pre>

<p>次に、ページと商品の重複を除去して、閲覧された一意の商品の数を調べます。そのためには <code>ARRAY_AGG()</code> に <code>DISTINCT</code> を追加します。</p>
<pre><code class="language-sql prettyprint">SELECT&#x000A;  fullVisitorId,&#x000A;  date,&#x000A;  ARRAY_AGG(DISTINCT v2ProductName) AS products_viewed,&#x000A;  ARRAY_LENGTH(ARRAY_AGG(DISTINCT v2ProductName)) AS distinct_products_viewed,&#x000A;  ARRAY_AGG(DISTINCT pageTitle) AS pages_viewed,&#x000A;  ARRAY_LENGTH(ARRAY_AGG(DISTINCT pageTitle)) AS distinct_pages_viewed&#x000A;  FROM `data-to-insights.ecommerce.all_sessions`&#x000A;WHERE visitId = 1501570398&#x000A;GROUP BY fullVisitorId, date&#x000A;ORDER BY date&#x000A;</code></pre>

<p><strong>内容のまとめ</strong></p>
<p>配列には、次のような便利な機能があります。</p>
<ul>
<li>
<p><code>ARRAY_LENGTH(&lt;array&gt;)</code> で要素の数を調べる</p>
</li>
<li>
<p><code>ARRAY_AGG(DISTINCT &lt;field&gt;)</code> で要素の重複を除去する</p>
</li>
<li>
<p><code>ARRAY_AGG(&lt;field&gt; ORDER BY &lt;field&gt;)</code> で要素を並べ替える</p>
</li>
<li>
<p><code>ARRAY_AGG(&lt;field&gt; LIMIT 5)</code> で配列の要素数を制限する</p>
</li>
</ul>
<h2 id="step7">すでに配列が含まれているデータセットをクエリする</h2>
<p>Google アナリティクス向けの BigQuery 一般公開データセット <code>bigquery-public-data.google_analytics_sample</code> には、このコースのデータセット <code>data-to-insights.ecommerce.all_sessions</code> より多くのフィールドと行が含まれています。さらに、商品、ページ、トランザクションなどのフィールド値が、配列としてネイティブに格納されています。</p>
<p>以下のクエリを<strong>コピーして貼り付けて</strong>、どのようなデータがあるか調べます。繰り返し値（配列）を含むフィールドがないか探してみてください。</p>
<pre><code class="language-sql prettyprint">SELECT&#x000A;  *&#x000A;FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`&#x000A;WHERE visitId = 1501570398&#x000A;</code></pre>
<p>[<strong>実行</strong>] をクリックしてクエリを実行します。</p>
<p>結果を<strong>右にスクロール</strong>して、<code>hits.product.v2ProductName</code> フィールドを探します（複数フィールドのエイリアスについてはこの後で説明します）。</p>

<p>Google アナリティクス スキーマに含まれているフィールドは、ここで分析するには多すぎます。前と同じように、訪問者とページ名のフィールドだけをクエリしてみましょう。</p>
<pre><code class="language-sql prettyprint">SELECT&#x000A;  visitId,&#x000A;  hits.page.pageTitle&#x000A;FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`&#x000A;WHERE visitId = 1501570398&#x000A;</code></pre>
<p>「<code>Cannot access field product on a value with type ARRAY&gt; at [5:8]</code>」というエラーが表示されます。</p>
<p>繰り返しフィールド（配列）を通常どおりにクエリするには、まず配列を分割して複数の行に戻す必要があります。</p>
<p>たとえば、<code>hits.page.pageTitle</code> の配列は、次のように 1 つの行として格納されています。</p>
<pre><code>['homepage','product page','checkout']&#x000A;</code></pre>
<p>これを次のようにする必要があります。</p>
<pre><code>['homepage',&#x000A;'product page',&#x000A;'checkout']&#x000A;</code></pre>
<p>これを SQL で行うにはどうすればよいでしょうか。</p>
<p>解答: 配列フィールドで UNNEST() 関数を使用します。</p>
<pre><code class="language-sql prettyprint">SELECT DISTINCT&#x000A;  visitId,&#x000A;  h.page.pageTitle&#x000A;FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`,&#x000A;UNNEST(hits) AS h&#x000A;WHERE visitId = 1501570398&#x000A;LIMIT 10&#x000A;</code></pre>
<p>UNNEST() については後ほど詳しく説明します。ここでは差し当たり、次のことを覚えておいてください。</p>
<ul>
<li>配列要素を行に戻すには UNNEST() を使用する</li>
<li>UNNEST() は常に FROM 句のテーブル名の後に指定する（概念的には、事前に結合されたテーブルに似ています）</li>
</ul>

<h2 id="step8">構造体の概要</h2>
<p>フィールド エイリアスの <code>hit.page.pageTitle</code> について、3 つのフィールドをドットで区切って 1 つにまとめたように見えるのが気になった方もいらっしゃるでしょう。配列値を使用すると、フィールドの粒度をより細かく「掘り下げる<em></em>」ことができますが、これと同様に、関連するフィールドをグループ化してスキーマを「広げる<em></em>」ことができるデータ型があります。それが、SQL データ型の <a href="https://cloud.google.com/bigquery/docs/reference/standard-sql/data-types#struct-type">（構造体）</a>です。</p>
<p>概念的には、構造体はメインテーブルに事前に結合された別テーブルのようなものと考えるとわかりやすくなります。</p>
<p>構造体には次のような特徴があります。</p>
<ul>
<li>1 つ以上のフィールドを含めることができる</li>
<li>フィールドのデータ型は同じでなくてもよい</li>
<li>固有のエイリアスを割り当てることができる</li>
</ul>
<p>このように、構造体はテーブルによく似ています。</p>
<h3>構造体を含むデータセットを探索する</h3>
<p>[<strong>リソース</strong>] で <strong>bigquery-public-data</strong> データセットを見つけます（このデータセットがまだない場合は、この<a href="https://console.cloud.google.com/bigquery?p=bigquery-public-data&amp;d=google_analytics_sample&amp;t=ga_sessions_20170801&amp;page=table" target="_blank">リンク</a>を使って固定してください）。</p>
<p><strong>bigquery-public-data</strong> をクリックして開きます。</p>
<p><strong>google_analytics_sample</strong> を見つけて開きます。</p>
<p><strong>ga_sessions</strong> テーブルをクリックします。</p>
<p>スキーマをスクロールし、ブラウザの検索機能（Ctrl+F キー）を使って次の質問に答えてください。</p>

<p>ご想像のとおり、今日の e コマースサイトで格納されるウェブサイトのセッション データは膨大な量になります。</p>
<p>1 つのテーブルで 32 個もの構造体を使用する最大のメリットは、結合を一切行わずに次のようなクエリを実行できることです。</p>
<pre><code class="language-sql prettyprint">SELECT&#x000A;  visitId,&#x000A;  totals.*,&#x000A;  device.*&#x000A;FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`&#x000A;WHERE visitId = 1501570398&#x000A;LIMIT 10&#x000A;</code></pre>
<p>注: <code>.*</code> という構文を使用すると、その構造体のすべてのフィールドが返されます（別のテーブルを結合した場合によく似ています）。</p>
<p>大きなレポート テーブルを構造体（事前に結合された「テーブル」）や配列（よりきめ細かい粒度）として格納すると、次のようなメリットがあります。</p>
<ul>
<li>
<p>32 個ものテーブルを結合する必要がなくなるため、パフォーマンスが大幅に向上する</p>
</li>
<li>
<p>配列から必要に応じてより詳細なデータを取得でき、その必要がないときのデメリットもない（BigQuery では各列が個別にディスクに保存されます）</p>
</li>
<li>
<p>すべてのビジネスデータが 1 つのテーブルに含まれるようになるため、結合キーに煩わされたり、必要なデータがどのテーブルにあるか調べたりする必要がなくなる</p>
</li>
</ul>
<h2 id="step9">構造体と配列の使い方を練習する</h2>
<p>次のデータセットは、トラックを走るランナーのラップタイムです。各ラップは「スプリット」と呼ばれます。</p>
<p><img alt="e271abf591541acc.png" src="https://cdn.qwiklabs.com/7SKxQcYHcPl4LFyErCHfGcASWgeQSpH%2FHbtJcGyfAEY%3D"></p>
<p>このクエリで STRUCT 構文を試します。構造体コンテナ内に、異なるデータ型のフィールドが存在する点に注意してください。</p>
<pre><code class="language-sql prettyprint">#standardSQL&#x000A;SELECT STRUCT("Rudisha" as name, 23.4 as split) as runner&#x000A;</code></pre>
<table>
<tr>
<td colspan="1" rowspan="1">
<p><strong>行</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>runner.name</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>runner.split</strong></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>1</p>
</td>
<td colspan="1" rowspan="1">
<p>Rudisha</p>
</td>
<td colspan="1" rowspan="1">
<p>23.4</p>
</td>
</tr>
</table>
<p>フィールドのエイリアスについて、どのようなことがわかりますか。構造体内でフィールドがネストされているため（name と split が runner のサブセット）、ドットを使用して表されています。</p>
<p>1 つのレースにランナーのスプリット タイムが複数ある場合はどうなるでしょうか（ラップごとのタイムなど）。</p>

<p>その場合は配列を使用します。次のクエリを実行して確認します。</p>
<pre><code class="language-sql prettyprint">#standardSQL&#x000A;SELECT STRUCT("Rudisha" as name, [23.4, 26.3, 26.4, 26.1] as splits) AS runner&#x000A;</code></pre>
<table>
<tr>
<td colspan="1" rowspan="1">
<p><strong>行</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>runner.name</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>runner.splits</strong></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="4">
<p>1</p>
</td>
<td colspan="1" rowspan="4">
<p>Rudisha</p>
</td>
<td colspan="1" rowspan="1">
<p>23.4</p>
</td>
</tr>
<tr><td colspan="1" rowspan="1">
<p>26.3</p>
</td></tr>
<tr><td colspan="1" rowspan="1">
<p>26.4</p>
</td></tr>
<tr><td colspan="1" rowspan="1">
<p>26.1</p>
</td></tr>
</table>
<p>まとめると次のようになります。</p>
<ul>
<li>
<p>構造体は、内部に複数のフィールド名とデータ型を保持できるコンテナです。</p>
</li>
<li>
<p>構造体内のフィールドの型には配列を使用できます（上記の splits フィールドを参照）。</p>
</li>
</ul>
<h3>JSON データを実際に取り込む</h3>
<p>「<strong>racing</strong>」という名前の新しいデータセットを作成します。</p>
<p>「<strong>race_results</strong>」と名前の新しいテーブルを作成します。</p>
<p>以下の Google Cloud Storage の JSON ファイルを取り込みます。</p>
<pre><code>gs://data-insights-course/labs/optimizing-for-performance/race_results.json&#x000A;</code></pre>
<ul>
<li>
<p>[<strong>ソース</strong>] で [<strong>テーブルの作成元</strong>] プルダウン メニューから [<strong>Google Cloud Storage</strong>] を選択します。</p>
</li>
<li>
<p>[<strong>GCS バケットからファイルを選択</strong>]: <code>gs://data-insights-course/labs/optimizing-for-performance/race_results.json</code></p>
</li>
<li>
<p>[<strong>ファイル形式</strong>]: JSON（改行区切り）</p>
</li>
<li>
<p>[<strong>スキーマ</strong>] の [<strong>テキストとして編集</strong>] スライダーをクリックし、次の内容を追加します。</p>
</li>
</ul>
<pre><code class="language-bash prettyprint">[&#x000A;    {&#x000A;        "name": "race",&#x000A;        "type": "STRING",&#x000A;        "mode": "NULLABLE"&#x000A;    },&#x000A;    {&#x000A;        "name": "participants",&#x000A;        "type": "RECORD",&#x000A;        "mode": "REPEATED",&#x000A;        "fields": [&#x000A;            {&#x000A;                "name": "name",&#x000A;                "type": "STRING",&#x000A;                "mode": "NULLABLE"&#x000A;            },&#x000A;            {&#x000A;                "name": "splits",&#x000A;                "type": "FLOAT",&#x000A;                "mode": "REPEATED"&#x000A;            }&#x000A;        ]&#x000A;    }&#x000A;]&#x000A;</code></pre>
<p>[<strong>テーブルを作成</strong>] をクリックします。</p>
<p>読み込みジョブが完了したら、新しく作成されたテーブルをプレビューします。</p>
<p><img alt="3e306d62a708e186.png" src="https://cdn.qwiklabs.com/uS4tIn1NVXc08s%2FVZXp4nll64cbAlYrnAmx6MlMyt1w%3D"></p>
<p>構造体のフィールドはどれでしょうか。</p>
<p><strong>participants</strong> フィールドは RECORD 型なので構造体です。</p>
<p>配列のフィールドはどれでしょうか。</p>
<p><code>participants.splits</code> フィールドは、親である <code>participants</code> 構造体内の FLOAT の配列です。モードが REPEATED であるため、配列であることがわかります。この配列の値は、単一のフィールドに複数の値が含まれるため「ネストされた値」と呼ばれます。</p>

<h3>ネストされた繰り返しフィールドに対してクエリを実行する</h3>
<p>800M レースのすべてのランナーを確認しましょう。</p>
<pre><code class="language-sql prettyprint">#standardSQL&#x000A;SELECT * FROM racing.race_results&#x000A;</code></pre>
<p>何件の行が返されましたか。</p>
<p>解答: 1</p>
<p><img alt="2bd4ee3bd800b483.png" src="https://cdn.qwiklabs.com/8ygOJAxnxJxkoRxOSBabLh0T7%2BIwx%2FaE%2BSK%2F46cYexg%3D"></p>
<p>各ランナーの名前とレースの種類を一覧表示するにはどうすればよいでしょうか。</p>
<p>次のクエリを実行するとどうなるか確認しましょう。</p>
<pre><code class="language-sql prettyprint">#standardSQL&#x000A;SELECT race, participants.name&#x000A;FROM racing.race_results&#x000A;</code></pre>
<p><code>Error: Cannot access field name on a value with type ARRAY&lt;STRUCT&lt;name STRING, splits ARRAY&lt;FLOAT64&gt;&gt;&gt;&gt; at [1:21]</code></p>
<p>集計関数を使う際に GROUP BY を忘れた状態に似ています。ここでは、粒度の異なる 2 つの項目があります。レースが 1 行、参加者名が 3 行あります。これをどのように変更すればよいでしょうか。</p>
<table>
<tr>
<td colspan="1" rowspan="1">
<p><strong>行</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>race</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>participants.name</strong></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>1</p>
</td>
<td colspan="1" rowspan="1">
<p>800M</p>
</td>
<td colspan="1" rowspan="1">
<p>Rudisha</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>2</p>
</td>
<td colspan="1" rowspan="1">
<p>???</p>
</td>
<td colspan="1" rowspan="1">
<p>Makhloufi</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>3</p>
</td>
<td colspan="1" rowspan="1">
<p>???</p>
</td>
<td colspan="1" rowspan="1">
<p>Murphy</p>
</td>
</tr>
</table>
<p>これを次のようにします。</p>
<table>
<tr>
<td colspan="1" rowspan="1">
<p><strong>行</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>race</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>participants.name</strong></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>1</p>
</td>
<td colspan="1" rowspan="1">
<p>800M</p>
</td>
<td colspan="1" rowspan="1">
<p>Rudisha</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>2</p>
</td>
<td colspan="1" rowspan="1">
<p>800M</p>
</td>
<td colspan="1" rowspan="1">
<p>Makhloufi</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>3</p>
</td>
<td colspan="1" rowspan="1">
<p>800M</p>
</td>
<td colspan="1" rowspan="1">
<p>Murphy</p>
</td>
</tr>
</table>
<p>従来のリレーショナル SQL では、レースのテーブルと参加者のテーブルがある場合、両方のテーブルから情報を取得するにはテーブルを結合します。ここでは、参加者の構造体（概念的にはテーブルに似ています）は、すでにレースのテーブルに含まれていますが、構造体ではない「race」のフィールドとまだ適切に関連付けられていません。</p>
<p>最初のテーブルで 800M レースを各ランナーと関連付けるために使用する SQL コマンドは何ですか。</p>
<p>解答: CROSS JOIN</p>
<p>次のクエリを実行してみましょう。</p>
<pre><code class="language-sql prettyprint">#standardSQL&#x000A;SELECT race, participants.name&#x000A;FROM racing.race_results&#x000A;CROSS JOIN&#x000A;participants  # これは構造体です（テーブル内のテーブルのようなものです）&#x000A;</code></pre>
<p><code>Error: Table name "participants" cannot be resolved: dataset name is missing</code>.</p>
<p>参加者の構造体はテーブルに似ていますが、技術的には <code>racing.race_results</code> テーブル内のフィールドの 1 つです。</p>
<p>クエリにデータセット名を追加します。</p>
<pre><code class="language-sql prettyprint">#standardSQL&#x000A;SELECT race, participants.name&#x000A;FROM racing.race_results&#x000A;CROSS JOIN&#x000A;race_results.participants # 完全な構造体名&#x000A;</code></pre>
<p>次に [<strong>実行</strong>] をクリックします。</p>
<p>結果はどうなりましたか。各レースのランナーがすべて一覧表示されました。</p>
<table>
<tr>
<td colspan="1" rowspan="1">
<p><strong>行</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>race</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>name</strong></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>1</p>
</td>
<td colspan="1" rowspan="1">
<p>800M</p>
</td>
<td colspan="1" rowspan="1">
<p>Rudisha</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>2</p>
</td>
<td colspan="1" rowspan="1">
<p>800M</p>
</td>
<td colspan="1" rowspan="1">
<p>Makhloufi</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>3</p>
</td>
<td colspan="1" rowspan="1">
<p>800M</p>
</td>
<td colspan="1" rowspan="1">
<p>Murphy</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>4</p>
</td>
<td colspan="1" rowspan="1">
<p>800M</p>
</td>
<td colspan="1" rowspan="1">
<p>Bosse</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>5</p>
</td>
<td colspan="1" rowspan="1">
<p>800M</p>
</td>
<td colspan="1" rowspan="1">
<p>Rotich</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>6</p>
</td>
<td colspan="1" rowspan="1">
<p>800M</p>
</td>
<td colspan="1" rowspan="1">
<p>Lewandowski</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>7</p>
</td>
<td colspan="1" rowspan="1">
<p>800M</p>
</td>
<td colspan="1" rowspan="1">
<p>Kipketer</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>8</p>
</td>
<td colspan="1" rowspan="1">
<p>800M</p>
</td>
<td colspan="1" rowspan="1">
<p>Berian</p>
</td>
</tr>
</table>
<p>以下の方法で最後のクエリを簡素化できます。</p>
<ul>
<li>元のテーブルのエイリアスを追加する</li>
<li>「CROSS JOIN」の句をカンマで置き換える（カンマは暗黙的にクロス結合を表す）</li>
</ul>
<p>これで同じクエリ結果が得られます。</p>
<pre><code class="language-sql prettyprint">#standardSQL&#x000A;SELECT race, participants.name&#x000A;FROM racing.race_results AS r, r.participants&#x000A;</code></pre>
<p>レースの種類が複数ある場合（800M、100M、200M）、クロス結合では、デカルト積のように各ランナーの名前がすべてのレースと関連付けられることはないのでしょうか。</p>
<p><strong>解答</strong>: そのようにはなりません。これは相関<em></em>クロス結合であり、個々の行に関連付けられた要素のみが展開されます。詳しくは、<a href="https://cloud.google.com/bigquery/docs/reference/standard-sql/arrays#flattening-arrays">配列と構造体の操作</a>をご覧ください。</p>
<p>構造体のまとめ:</p>
<ul>
<li>
<p>SQL の<a href="https://cloud.google.com/bigquery/docs/reference/standard-sql/data-types#struct-type">構造体</a>は、単純に他のデータ フィールドのコンテナです（異なるデータ型を格納可能）。構造体という言葉はデータが構造化されていることを表します。先ほどの例を思い出してください。</p>
</li>
<li>
<p><code>STRUCT(``"Rudisha" as name, [23.4, 26.3, 26.4, 26.1] as splits``)`` AS runner</code></p>
</li>
<li>
<p>構造体にエイリアス（上記の runner）が指定され、概念的にはメインテーブル内に含まれるテーブルと考えることができます。</p>
</li>
<li>
<p>構造体（および配列）で要素を操作するには、要素を展開する必要があります。構造体自体の名前または構造体内の配列のフィールドを UNNEST() で囲み、展開してフラット化します。</p>
</li>
</ul>
<h2 id="step10">ラボの質問: STRUCT()</h2>
<p>先ほど作成した <code>racing.race_results</code> テーブルを使用して以下の質問に解答してください。</p>
<p><strong>タスク:</strong> 参加したランナーの合計数を取得するクエリを作成してください。</p>
<p>まず、部分的に作成済みの次のクエリを使用します。</p>
<pre><code class="language-sql prettyprint">#standardSQL&#x000A;SELECT COUNT(participants.name) AS racer_count&#x000A;FROM racing.race_results&#x000A;</code></pre>
<p><strong>ヒント:</strong> FROM の後に追加のデータソースとして、構造体名でクロス結合する必要があります。</p>
<p>正解例:</p>
<pre><code class="language-sql prettyprint">#standardSQL&#x000A;SELECT COUNT(p.name) AS racer_count&#x000A;FROM racing.race_results AS r, UNNEST(r.participants) AS p&#x000A;</code></pre>
<table>
<tr>
<td colspan="1" rowspan="1">
<p><strong>行</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>racer_count</strong></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>1</p>
</td>
<td colspan="1" rowspan="1">
<p>8</p>
</td>
</tr>
</table>
<p>解答: レースに参加したランナーは 8 人です。</p>

<h2 id="step11">ラボの質問: 配列を UNNEST( ) で展開する</h2>
<p>名前が「R」で始まるランナーの合計レース時間を一覧表示するクエリを作成します。合計時間が短いランナーが先に表示されるように並べ替えます。UNNEST() 演算子を使って、部分的に作成済みの次のクエリで開始します。</p>
<p>クエリを完成させてください。</p>
<pre><code class="language-sql prettyprint">#standardSQL&#x000A;SELECT&#x000A;  p.name,&#x000A;  SUM(split_times) as total_race_time&#x000A;FROM racing.race_results AS r&#x000A;, r.participants AS p&#x000A;, p.splits AS split_times&#x000A;WHERE&#x000A;GROUP BY&#x000A;ORDER BY&#x000A;;&#x000A;</code></pre>
<p>ヒント:</p>
<ul>
<li>FROM 句で、構造体および構造体内の配列の両方をデータソースとして展開する必要があります。</li>
<li>必要に応じてエイリアスを使用します。</li>
</ul>
<p>正解例:</p>
<pre><code class="language-sql prettyprint">#standardSQL&#x000A;SELECT&#x000A;  p.name,&#x000A;  SUM(split_times) as total_race_time&#x000A;FROM racing.race_results AS r&#x000A;, UNNEST(r.participants) AS p&#x000A;, UNNEST(p.splits) AS split_times&#x000A;WHERE p.name LIKE 'R%'&#x000A;GROUP BY p.name&#x000A;ORDER BY total_race_time ASC;&#x000A;</code></pre>
<table>
<tr>
<td colspan="1" rowspan="1">
<p><strong>行</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>name</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>total_race_time</strong></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>1</p>
</td>
<td colspan="1" rowspan="1">
<p>Rudisha</p>
</td>
<td colspan="1" rowspan="1">
<p>102.19999999999999</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>2</p>
</td>
<td colspan="1" rowspan="1">
<p>Rotich</p>
</td>
<td colspan="1" rowspan="1">
<p>103.6</p>
</td>
</tr>
</table>

<h2 id="step12">配列内の値でフィルタする</h2>
<p>800M のレースで最も早いラップタイムは、23.2 秒でした。ただし、それがどのランナーの記録であるかは確認できていません。その結果を返すクエリを作成します。</p>
<p>部分的に作成済みの次のクエリを完成させてください。</p>
<pre><code class="language-sql prettyprint">#standardSQL&#x000A;SELECT&#x000A;  p.name,&#x000A;  split_time&#x000A;FROM racing.race_results AS r&#x000A;, r.participants AS p&#x000A;, p.splits AS split_time&#x000A;WHERE split_time = ;&#x000A;</code></pre>
<p>正解例:</p>
<pre><code class="language-sql prettyprint">#standardSQL&#x000A;SELECT&#x000A;  p.name,&#x000A;  split_time&#x000A;FROM racing.race_results AS r&#x000A;, UNNEST(r.participants) AS p&#x000A;, UNNEST(p.splits) AS split_time&#x000A;WHERE split_time = 23.2;&#x000A;</code></pre>
<table>
<tr>
<td colspan="1" rowspan="1">
<p><strong>行</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>name</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>split_time</strong></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>1</p>
</td>
<td colspan="1" rowspan="1">
<p>Kipketer</p>
</td>
<td colspan="1" rowspan="1">
<p>23.2</p>
</td>
</tr>
</table>