**BigQuery for Machine Learning**

機械学習を学んで実践し、SQL だけを使用して、数時間ではなく数分でモデルをビルドしたいとお考えの場合、BigQuery の新機能である BigQuery ML を使用すれば、最小限のコーディングで機械学習モデルの作成、トレーニング、評価、予測が可能になります。この一連のラボでは、さまざまなモデルタイプを試して、優れたモデルを作成する方法を学習します。

<ul>
<li>BQML の詳細については、<a href="https://cloud.google.com/bigquery/docs/bigqueryml-intro">ドキュメント</a>をご覧ください。</li>
<li><a href="https://cloud.google.com/bigquery/docs/bigqueryml-scientist-start">データ サイエンティスト向けの BigQuery ML スタートガイド</a></li>
<li><a href="https://cloud.google.com/bigquery/docs/bigqueryml-analyst-start">データ アナリスト向けの BigQuery ML スタートガイド</a></li>
<li>すでに Google アナリティクス アカウントをお持ちで、BigQuery で独自のデータセットをクエリすることをご希望の場合は、こちらの<a href="https://support.google.com/analytics/answer/3416092">エクスポート ガイド</a>に従ってください。</li>
<li>詳細な BigQuery SQL リファレンス ガイドは、追加のリソースとしてこちらからご覧いただけます。<a href="https://cloud.google.com/bigquery/docs/reference/standard-sql/query-syntax">https://cloud.google.com/bigquery/docs/reference/standard-sql/query-syntax</a>
</li>
</ul>

# BQML スタートガイド
<h2 id="step2">概要</h2>
<p><a href="https://cloud.google.com/bigquery/">BigQuery Machine Learning</a>（BQML、ベータ版）を使用すると、BigQuery で SQL クエリを使用して機械学習モデルを作成し実行できます。その目的は、SQL のユーザーが簡単に機械学習を利用できるようにすることです。使い慣れたツールを使用してモデルを構築でき、データを移動する必要もないため、開発スピードを向上させることができます。</p>
<p><a href="https://shop.googlemerchandisestore.com/">Google Merchandise Store</a> に関する数百万件にのぼる Google アナリティクスのレコードが格納された <a href="https://www.en.advertisercommunity.com/t5/Articles/Introducing-the-Google-Analytics-Sample-Dataset-for-BigQuery/ba-p/1676331#">e コマース データセット</a>が新たに BigQuery に読み込まれ、利用できるようになりました。このラボでは、このデータを使用して訪問者が取引を実行するかどうかを予測するモデルを作成します。</p>
<h3>ラボの内容</h3>
<p>BigQuery で機械学習モデルを作成、評価、使用する方法</p>
<h3>必要なもの</h3>
<ul>
<li>
<p>ブラウザ（<a href="https://www.google.com/chrome/browser/desktop/">Chrome</a>、<a href="https://www.mozilla.org/firefox/">Firefox</a> など）</p>
</li>
<li>
<p>SQL または BigQuery の基本的な知識</p>
</li>
</ul>

<h3>BigQuery Console を開く</h3>
<p>Google Cloud Console で [<strong>ナビゲーション メニュー</strong>] &gt; [<strong>BigQuery</strong>] を選択します。</p>
<p><img alt="BigQuery_menu.png" src="https://cdn.qwiklabs.com/QwUX6algdXPgIThqBlgB2rnqGkoQUUANkl3xicD06uc%3D"></p>
<p>[<strong>Cloud Console の BigQuery へようこそ</strong>] メッセージボックスが開きます。このメッセージボックスには、クイックスタート ガイドとリリースノートへのリンクが表示されています。</p>
<p>[<strong>完了</strong>] をクリックします。</p>
<p>BigQuery コンソールが開きます。</p>
<p>
</p>
<p><img alt="bq-console.png" src="https://cdn.qwiklabs.com/vd3QNHGB4BAyHjAvOHw9qD0iqCPNaHAzY657z%2FGWLtY%3D"></p>

<h2 id="step4">データセットを作成する</h2>
<p>データセットを作成するには、プロジェクト名の横にある矢印をクリックして [<strong>データセットを作成</strong>] を選択します。</p>
<p><img alt="b5416ccaffe4b195.png" src="https://cdn.qwiklabs.com/vB7JumPZxSBDwJJfux363NFVnHpm2Z4XtZCeJfRe4gU%3D"></p>
<p>次に、データセットに「<strong>bqml_lab</strong>」という名前を付けて [<strong>データセットを作成</strong>] をクリックします。</p>
<p><img alt="1fb788b51c77d4b6.png" src="https://cdn.qwiklabs.com/omQLG7rA4gLj8dg%2BcrdSRFrrMQdtD25a9aaRRU32y1A%3D"></p>

<h2 id="step5">モデルを作成する</h2>
<p>次に進みましょう。</p>
<p>[<strong>クエリエディター</strong>] をクリックして以下のクエリを追加し、訪問者が取引を実行するかどうかを予測するモデルを作成します。</p>

```sql
#standardSQL
CREATE OR REPLACE MODEL `bqml_lab.sample_model`
OPTIONS(model_type='logistic_reg') AS
SELECT
  IF(totals.transactions IS NULL, 0, 1) AS label,
  IFNULL(device.operatingSystem, "") AS os,
  device.isMobile AS is_mobile,
  IFNULL(geoNetwork.country, "") AS country,
  IFNULL(totals.pageviews, 0) AS pageviews
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`
WHERE
  _TABLE_SUFFIX BETWEEN '20160801' AND '20170631'
LIMIT 100000;
```

<p>ここでは、取引が行われたかどうかの基準として、訪問者のデバイスのオペレーティング システム、そのデバイスがモバイル デバイスであるかどうか、訪問者の国、ページビューの回数が使用されます。</p>
<p>ここで、<code>bqml_lab</code> はデータセットの名前、<code>sample_model</code> はモデルの名前です。指定されたモデルタイプは 2 項ロジスティック回帰です。<code>label</code> はフィッティングの対象です。</p>
<aside class="special"><p><strong>注:</strong> 1列のみに関心がある場合に、これは<code>input_label_cols</code>を設定するの互い違い方法です。</p>
</aside>
<p>トレーニング データは、2016 年 8 月 1 日から 2017 年 6 月 30 日の間に収集されたものに限定しています。これは「予測」用に最後の月のデータを残しておくためです。さらに、時間の節約のため 100,000 データポイントに制限します。</p>
<p><code>CREATE MODEL</code> コマンドを実行すると非同期で実行されるクエリジョブが作成されるため、BigQuery UI ウィンドウを閉じたり、更新したりといったことができます。</p>

<h3><strong>（省略可）モデル情報とトレーニング統計</strong></h3>
<p>興味があれば、UI で <code>bqml_lab</code>、<code>sample_model</code> データセットの順にクリックすると、モデルに関する情報が得られます。[<strong>詳細</strong>] タブに、モデルの生成に使用される基本的なモデル情報とトレーニング オプションが表示されます。[<strong>トレーニング統計</strong>] の下に、次のような表が表示されます。</p>
<p><img alt="sm-table.png" src="https://cdn.qwiklabs.com/RasutC3NgQjgeiyL2hevVZhEdiBvohhhasf3eTE4%2FFw%3D"></p>
<p><img alt="sm-graph.png" src="https://cdn.qwiklabs.com/Q9RCxurY6RIrVbhLNoJjWFZ2zgo2ndVS8tIjJ2gx51w%3D"></p>
<h2 id="step6">モデルを評価する</h2>
<p>今度は、クエリを以下に置き換えます。</p>

```sql
#standardSQL
SELECT
  *
FROM
  ml.EVALUATE(MODEL `bqml_lab.sample_model`, (
SELECT
  IF(totals.transactions IS NULL, 0, 1) AS label,
  IFNULL(device.operatingSystem, "") AS os,
  device.isMobile AS is_mobile,
  IFNULL(geoNetwork.country, "") AS country,
  IFNULL(totals.pageviews, 0) AS pageviews
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`
WHERE
  _TABLE_SUFFIX BETWEEN '20170701' AND '20170801'));
```

<p>線形回帰モデルで使用した場合、上記のクエリは以下の列を返します。</p>
<ul>
<li>
<code>mean_absolute_error</code>、<code>mean_squared_error</code>、<code>mean_squared_log_error</code>、</li>
<li>
<code>median_absolute_error</code>、<code>r2_score</code>、<code>explained_variance</code>
</li>
</ul>
<p>ロジスティック回帰モデルで使用した場合、上記のクエリは以下の列を返します。</p>
<ul>
<li>
<code>precision</code>、<code>recall</code>
</li>
<li>
<code>accuracy</code>、<code>f1_score</code>
</li>
<li>
<code>log_loss</code>、<code>roc_auc</code>
</li>
</ul>
<p>各指標の算出方法や意味を理解するには、<a href="https://developers.google.com/machine-learning/glossary/">機械学習の用語集</a>をご覧になるか、Google 検索をご利用ください。</p>
<p>クエリの <code>SELECT</code> と <code>FROM</code> の部分はトレーニング中に使用したものと同じであることがわかります。<code>WHERE</code> の部分は期間の違いを反映し、<code>FROM</code> の部分は <code>ml.EVALUATE</code> の呼び出しを示しています。</p>
<p>以下のような表が表示されます。</p>
<p><img alt="40b9e88efb1252e6.png" src="https://cdn.qwiklabs.com/F4xj1Rol9xUq416nkDE%2FaruyNryT1P9vhfZLpRbXWxI%3D"></p>

<h2 id="step7">モデルを使用する</h2>
<h3><strong>国ごとの購入数を予測する</strong></h3>
<p>このクエリでは、各国の訪問者による取引の数を予測し、結果を並べ替えて、購入数の上位 10 か国を抽出します。</p>

```sql
#standardSQL
SELECT
  country,
  SUM(predicted_label) as total_predicted_purchases
FROM
  ml.PREDICT(MODEL `bqml_lab.sample_model`, (
SELECT
  IFNULL(device.operatingSystem, "") AS os,
  device.isMobile AS is_mobile,
  IFNULL(totals.pageviews, 0) AS pageviews,
  IFNULL(geoNetwork.country, "") AS country
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`
WHERE
  _TABLE_SUFFIX BETWEEN '20170701' AND '20170801'))
GROUP BY country
ORDER BY total_predicted_purchases DESC
LIMIT 10;
```

<p>このクエリは、前のセクションで示した評価クエリと非常によく似ています。<code>ml.EVALUATE</code> の代わりに、<code>ml.PREDICT</code> を使用し、クエリの BQML の部分は標準 SQL コマンドでラップされています。このラボでは、国とその国の購入の合計が必要なので、<code>SELECT</code>、<code>GROUP BY</code>、<code>ORDER BY</code> を使用します。<code>LIMIT</code> は結果を上位 10 件に制限するために使用しています。</p>
<p>以下のような表が表示されます。</p>
<p><img alt="5ea34ba0da80443a.png" src="https://cdn.qwiklabs.com/87OvTSvXnVFthzil2BzMoTlTz8KUtWYezliUJhZ1m6s%3D"></p>

<h3><strong>ユーザーごとの購入数を予測する</strong></h3>
<p>もう一つ例を示します。今度は、各訪問者による取引の数を予測し、結果をソートして、取引数の上位 10 人の訪問者を抽出します。</p>

```sql
#standardSQL
SELECT
  fullVisitorId,
  SUM(predicted_label) as total_predicted_purchases
FROM
  ml.PREDICT(MODEL `bqml_lab.sample_model`, (
SELECT
  IFNULL(device.operatingSystem, "") AS os,
  device.isMobile AS is_mobile,
  IFNULL(totals.pageviews, 0) AS pageviews,
  IFNULL(geoNetwork.country, "") AS country,
  fullVisitorId
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`
WHERE
  _TABLE_SUFFIX BETWEEN '20170701' AND '20170801'))
GROUP BY fullVisitorId
ORDER BY total_predicted_purchases DESC
LIMIT 10;
```

<p>以下のような表が表示されます。</p>
<p><img alt="e8f461a65fc57f2b.png" src="https://cdn.qwiklabs.com/%2Bzk0mE%2Bw87K3wW2%2FZJSrb0b9xoKiLSgnLe6gVtugr1I%3D"></p>

# BQML で分類モデルを使用して訪問者の購入を予測する
<h2 id="step2">概要</h2>
<p>BigQuery は、低コストでテラバイト規模のデータをクエリできる分析データベースです。Google による一貫管理が行われ、インフラストラクチャの管理やデータベース管理者の設置など、お客様による管理運用の必要がないため、有用な情報取得のためのデータ分析に専念できます。BigQuery は SQL を使用しており、従量課金制モデルでも利用できます。</p>
<p>BigQuery の新機能である <a href="https://cloud.google.com/bigquery/docs/bigqueryml-analyst-start">BigQuery ML</a>（BQML、ベータ版）を使用すれば、データ アナリストによる最小限のコーディングで、機械学習モデルの作成、トレーニング、評価、予測が可能になります。</p>
<p><a href="https://www.en.advertisercommunity.com/t5/Articles/Introducing-the-Google-Analytics-Sample-Dataset-for-BigQuery/ba-p/1676331#">Google Merchandise Store</a> に関する数百万件にのぼる Google アナリティクスのレコードが格納された <a href="https://shop.googlemerchandisestore.com/">e コマース データセット</a>が新たに BigQuery に読み込まれ、利用できるようになりました。このラボでは、このデータを使用して一般的なクエリを実行し、企業が知りたい顧客の購買習慣に関する情報を取得します。</p>
<h3>目標</h3>
<p>このラボでは、次のタスクの実行方法について学びます。</p>
<ul>
<li>
<p>BigQuery を使用して一般公開データセットを見つける</p>
</li>
<li>
<p>e コマース データセットをクエリし、探索する</p>
</li>
<li>
<p>バッチ予測に使用するトレーニングと評価のデータセットを作成する</p>
</li>
<li>
<p>分類（ロジスティック回帰）モデルを BQML に作成する</p>
</li>
<li>
<p>機械学習モデルの性能を評価する</p>
</li>
<li>
<p>訪問者が購入する見込みを予測し、ランクを付ける</p>
</li>
</ul>

<h3>BigQuery Console を開く</h3>
<p>Google Cloud Console で [<strong>ナビゲーション メニュー</strong>] &gt; [<strong>BigQuery</strong>] を選択します。</p>
<p><img alt="BigQuery_menu.png" src="https://cdn.qwiklabs.com/QwUX6algdXPgIThqBlgB2rnqGkoQUUANkl3xicD06uc%3D"></p>
<p>[<strong>Cloud Console の BigQuery へようこそ</strong>] メッセージボックスが開きます。このメッセージボックスには、クイックスタート ガイドとリリースノートへのリンクが表示されています。</p>
<p>[<strong>完了</strong>] をクリックします。</p>
<p>BigQuery コンソールが開きます。</p>
<p>
</p>
<p><img alt="bq-console.png" src="https://cdn.qwiklabs.com/vd3QNHGB4BAyHjAvOHw9qD0iqCPNaHAzY657z%2FGWLtY%3D"></p>

<h3>コース用データセットへのアクセス</h3>
<p>BigQuery を開いたら、新しいタブで次の直接リンクを開いて、一般公開の <strong>data-to-insights</strong> プロジェクトを BigQuery プロジェクト パネルに取り込みます。</p>
<ul>
<li><a href="https://console.cloud.google.com/bigquery?p=data-to-insights&amp;d=ecommerce&amp;t=web_analytics&amp;page=table">https://console.cloud.google.com/bigquery?p=data-to-insights&amp;d=ecommerce&amp;t=web_analytics&amp;page=table</a></li>
</ul>
<p><img alt="dataset-added.png" src="https://cdn.qwiklabs.com/wAOpgsMVrpanjH%2F8OqKMFHWaIUfTMjG6C1O8HBYRlzM%3D"></p>
<p><strong>data-to-insights</strong> e コマース データセットのフィールド定義は<a href="https://support.google.com/analytics/answer/3437719?hl=ja">こちら</a>をご覧ください。このリンク先のページは、参照用に新しいタブで開いたままにしておきます。</p>
<h2 id="step4">e コマースデータを探索する</h2>
<p><strong>シナリオ:</strong> データ アナリスト チームが e コマース ウェブサイトに関する Google アナリティクスのログを BigQuery にエクスポートし、e コマース訪問者のセッションの生データをすべて含むテーブルを作成してデータを探索できるようにしました。このデータを使用して、いくつかの質問に対する答えを見つけていきます。</p>
<p><strong>質問:</strong> ウェブサイトを訪れた訪問者の何パーセントが実際に購入したか。</p>
<p>BigQuery ページで [<strong>クエリエディタ</strong>] をクリックして、クエリエディタで次をコピーして入力します。</p>

```sql
#standardSQL
WITH visitors AS(
SELECT
COUNT(DISTINCT fullVisitorId) AS total_visitors
FROM `data-to-insights.ecommerce.web_analytics`
),

purchasers AS(
SELECT
COUNT(DISTINCT fullVisitorId) AS total_purchasers
FROM `data-to-insights.ecommerce.web_analytics`
WHERE totals.transactions IS NOT NULL
)

SELECT
  total_visitors,
  total_purchasers,
  total_purchasers / total_visitors AS conversion_rate
FROM visitors, purchasers
```

<p>[<strong>実行</strong>] をクリックします。</p>
<p>結果: 2.69%</p>
<p><strong>質問:</strong> 売上の上位 5 つの商品は何か。</p>
<p>前のクエリを次のように置き換え、<strong>クエリを実行</strong>します。</p>

```sql
SELECT
  p.v2ProductName,
  p.v2ProductCategory,
  SUM(p.productQuantity) AS units_sold,
  ROUND(SUM(p.localProductRevenue/1000000),2) AS revenue
FROM `data-to-insights.ecommerce.web_analytics`,
UNNEST(hits) AS h,
UNNEST(h.product) AS p
GROUP BY 1, 2
ORDER BY revenue DESC
LIMIT 5;
```

<p>[<strong>実行</strong>] をクリックします。</p>
<p>結果:</p>
<table>
<tr>
<td colspan="1" rowspan="1">
<p><strong>Row</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>v2ProductName</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>v2ProductCategory</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>units_sold</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>revenue</strong></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>1</p>
</td>
<td colspan="1" rowspan="1">
<p>Nest® Learning Thermostat 3rd Gen-USA - Stainless Steel</p>
</td>
<td colspan="1" rowspan="1">
<p>Nest-USA</p>
</td>
<td colspan="1" rowspan="1">
<p>17651</p>
</td>
<td colspan="1" rowspan="1">
<p>870976.95</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>2</p>
</td>
<td colspan="1" rowspan="1">
<p>Nest® Cam Outdoor Security Camera - USA</p>
</td>
<td colspan="1" rowspan="1">
<p>Nest-USA</p>
</td>
<td colspan="1" rowspan="1">
<p>16930</p>
</td>
<td colspan="1" rowspan="1">
<p>684034.55</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>3</p>
</td>
<td colspan="1" rowspan="1">
<p>Nest® Cam Indoor Security Camera - USA</p>
</td>
<td colspan="1" rowspan="1">
<p>Nest-USA</p>
</td>
<td colspan="1" rowspan="1">
<p>14155</p>
</td>
<td colspan="1" rowspan="1">
<p>548104.47</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>4</p>
</td>
<td colspan="1" rowspan="1">
<p>Nest® Protect Smoke + CO White Wired Alarm-USA</p>
</td>
<td colspan="1" rowspan="1">
<p>Nest-USA</p>
</td>
<td colspan="1" rowspan="1">
<p>6394</p>
</td>
<td colspan="1" rowspan="1">
<p>178937.6</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>5</p>
</td>
<td colspan="1" rowspan="1">
<p>Nest® Protect Smoke + CO White Battery Alarm-USA</p>
</td>
<td colspan="1" rowspan="1">
<p>Nest-USA</p>
</td>
<td colspan="1" rowspan="1">
<p>6340</p>
</td>
<td colspan="1" rowspan="1">
<p>178572.4</p>
</td>
</tr>
</table>
<p><strong>質問:</strong> ウェブサイトに再訪問して購入した訪問者は何人か。</p>
<p>次のクエリを実行して調べます。</p>

```sql
# 再訪問で購入した訪問者数（最初の訪問でも購入した場合も含まれる）
WITH all_visitor_stats AS (
SELECT
  fullvisitorid, # 741,721 人のユニーク訪問者
  IF(COUNTIF(totals.transactions > 0 AND totals.newVisits IS NULL) > 0, 1, 0) AS will_buy_on_return_visit
  FROM `data-to-insights.ecommerce.web_analytics`
  GROUP BY fullvisitorid
)

SELECT
  COUNT(DISTINCT fullvisitorid) AS total_visitors,
  will_buy_on_return_visit
FROM all_visitor_stats
GROUP BY will_buy_on_return_visit
```

<p>結果:</p>
<table>
<tr>
<td colspan="1" rowspan="1">
<p><strong>Row</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>total_visitors</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>will_buy_on_return_visit</strong></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>1</p>
</td>
<td colspan="1" rowspan="1">
<p>729848</p>
</td>
<td colspan="1" rowspan="1">
<p>0</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>2</p>
</td>
<td colspan="1" rowspan="1">
<p>11873</p>
</td>
<td colspan="1" rowspan="1">
<p>1</p>
</td>
</tr>
</table>
<p>結果を分析すると、総訪問者の 1.6%（11,873÷7,41,721）がウェブサイトに戻ってきて、購入していることがわかります。この人数には、最初のセッションで購入し、再度訪問してもう一度購入した訪問者のサブセットも含まれます。</p>
<p><strong>質問:</strong> e コマースの利用者の多くが、閲覧するものの再訪問するまで購入しない理由は何か。</p>
<p><strong>回答:</strong> 唯一の正解はありませんが、よくある理由ひとつは、最終的に購入を決定する前に、別の e コマースサイトと比較検討してから購入するから、というものです。これは、決定する前に入念な事前調査と比較が必要になる高額な商品（自動車など）の購入の際に顕著ですが、比較的程度は低いものの、このサイトの商品（T シャツやアクセサリなど）の購入にも当てはまります。</p>
<p>オンライン マーケティング業界では今後、初回訪問で観察された特徴に基づいて将来購入に至るユーザーを識別してマーケティング活動を行うことが、コンバージョン率を上げ、競合他社のサイトへの流出を抑える鍵となるでしょう。</p>
<h2 id="step5">対象を特定する</h2>
<p>ここからは、BigQuery で機械学習モデルを作成し、新しいユーザーが将来購入しそうかどうかを予測します。こうした高い価値を持つユーザーを識別することで、マーケティング チームがそれらのユーザーにターゲットを絞って特別プロモーションや広告キャンペーンを実施することが可能になり、それらのユーザーが自社の e コマースサイトを再度訪問するまでの間に他のサイトと比較していたとしても、コンバージョンにつなげることができます。</p>
<h2 id="step6">特徴を選択し、トレーニング データセットを作成する</h2>
<p>Google アナリティクスは、さまざまなディメンションを捉え、この e コマース ウェブサイトのユーザーの訪問を計測します。フィールドの完全なリストを<a href="https://support.google.com/analytics/answer/3437719?hl=ja">こちら</a>で確認してから<a href="https://bigquery.cloud.google.com/table/data-to-insights:ecommerce.web_analytics?tab=preview">デモ データセットをプレビュー</a>し、訪問者のウェブサイトへの初回訪問と、その訪問者が戻ってきて購入するかどうかの関係を、機械学習モデルが理解するのに役立つ特徴を見つけます。</p>
<p>次の 2 つのフィールドが分類モデルに適した入力であるかどうかをテストすることにします。</p>
<ul>
<li>
<code>totals.bounces</code>（訪問者がウェブサイトをすぐに離れるかどうか）</li>
<li>
<code>totals.timeOnSite</code>（訪問者がウェブサイトに留まった期間）</li>
</ul>
<p><strong>質問:</strong> 上記の 2 つのフィールドのみを使用することにはどんなリスクがあるか。</p>
<p><strong>回答:</strong> 機械学習の良し悪しは、提供されるトレーニング データにかかっています。入力した特徴とラベル（この場合、訪問者が将来購入するかどうか）の関係をモデルに判断、学習させるための十分な情報が揃わない場合、精度の高いモデルを確立することはできません。これら 2 つのフィールドでモデルをトレーニングすることは出発点にはなりますが、精度の高いモデルを生成するために 2 つのフィールドだけで十分かどうかは、これから見極めていきます。</p>
<p>クエリエディタで次のクエリを実行します。</p>

```sql
SELECT
  * EXCEPT(fullVisitorId)
FROM

  # 特徴
  (SELECT
    fullVisitorId,
    IFNULL(totals.bounces, 0) AS bounces,
    IFNULL(totals.timeOnSite, 0) AS time_on_site
  FROM
    `data-to-insights.ecommerce.web_analytics`
  WHERE
    totals.newVisits = 1)
  JOIN
  (SELECT
    fullvisitorid,
    IF(COUNTIF(totals.transactions > 0 AND totals.newVisits IS NULL) > 0, 1, 0) AS will_buy_on_return_visit
  FROM
      `data-to-insights.ecommerce.web_analytics`
  GROUP BY fullvisitorid)
  USING (fullVisitorId)
ORDER BY time_on_site DESC
LIMIT 10;
```

<p>[<strong>実行</strong>] をクリックします。</p>
<p>結果:</p>
<table>
<tr>
<td colspan="1" rowspan="1">
<p><strong>Row</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>bounces</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>time_on_site</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>will_buy_on_return_visit</strong></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>1</p>
</td>
<td colspan="1" rowspan="1">
<p>0</p>
</td>
<td colspan="1" rowspan="1">
<p>15047</p>
</td>
<td colspan="1" rowspan="1">
<p>0</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>2</p>
</td>
<td colspan="1" rowspan="1">
<p>0</p>
</td>
<td colspan="1" rowspan="1">
<p>12136</p>
</td>
<td colspan="1" rowspan="1">
<p>0</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>3</p>
</td>
<td colspan="1" rowspan="1">
<p>0</p>
</td>
<td colspan="1" rowspan="1">
<p>11201</p>
</td>
<td colspan="1" rowspan="1">
<p>0</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>4</p>
</td>
<td colspan="1" rowspan="1">
<p>0</p>
</td>
<td colspan="1" rowspan="1">
<p>10046</p>
</td>
<td colspan="1" rowspan="1">
<p>0</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>5</p>
</td>
<td colspan="1" rowspan="1">
<p>0</p>
</td>
<td colspan="1" rowspan="1">
<p>9974</p>
</td>
<td colspan="1" rowspan="1">
<p>0</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>6</p>
</td>
<td colspan="1" rowspan="1">
<p>0</p>
</td>
<td colspan="1" rowspan="1">
<p>9564</p>
</td>
<td colspan="1" rowspan="1">
<p>0</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>7</p>
</td>
<td colspan="1" rowspan="1">
<p>0</p>
</td>
<td colspan="1" rowspan="1">
<p>9520</p>
</td>
<td colspan="1" rowspan="1">
<p>0</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>8</p>
</td>
<td colspan="1" rowspan="1">
<p>0</p>
</td>
<td colspan="1" rowspan="1">
<p>9275</p>
</td>
<td colspan="1" rowspan="1">
<p>1</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>9</p>
</td>
<td colspan="1" rowspan="1">
<p>0</p>
</td>
<td colspan="1" rowspan="1">
<p>9138</p>
</td>
<td colspan="1" rowspan="1">
<p>0</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>10</p>
</td>
<td colspan="1" rowspan="1">
<p>0</p>
</td>
<td colspan="1" rowspan="1">
<p>8872</p>
</td>
<td colspan="1" rowspan="1">
<p>0</p>
</td>
</tr>
</table>
<p>どのフィールドがモデルの特徴になりますか。どれがラベル（正解）ですか。</p>
<p>入力は、<strong>bounces</strong> と <strong>time_on_site</strong> です。ラベルは、<strong>will_buy_on_return_visit</strong> です。</p>
<p><strong>質問:</strong> 訪問者の最初のセッションの後にわかる 2 つのフィールドはどれか。</p>
<p><strong>回答:</strong> <strong>bounces</strong> と <strong>time_on_site</strong> は、訪問者の最初のセッションの後に明らかになります。</p>
<p><strong>質問:</strong> 後になるまで判明しないフィールドはどれか。</p>
<p><strong>回答:</strong> <strong>will_buy_on_return_visit</strong> は、初回の訪問後にはわかりません。繰り返しになりますが、ここでは、ウェブサイトに戻ってきて購入するユーザーのサブセットを予測します。予測時には将来のことはわからないため、新しい訪問者が後から戻ってきて購入するかどうかについて確かなことは言えません。ML モデルを構築する価値は、最初のセッションについて集められたデータに基づいて、将来購入してもらえる確率を予測できる点にあります。</p>
<p><strong>質問:</strong> 最初のデータ結果に注目すると、<strong>time_on_site</strong> と <strong>bounces</strong> は、ユーザーが戻ってきて購入するかどうかを示す良い指標になると思うか。</p>
<p><strong>回答:</strong> モデルのトレーニングと評価を行う前に結論を出すのは早すぎるかもしれませんが、<code>time_on_site</code> の上位 10 項目を見ると、戻ってきて購入したユーザーは 1 人だけでした。あまり精度は高くなさそうです。モデルがどのように役立っているかに注目してみましょう。</p>
<h2 id="step7">モデルを格納する BigQuery データセットを作成する</h2>
<p>次に、新しい BigQuery データセットを作成します。このデータセットに ML モデルも格納します。</p>
<ol>
<li>左側のペインで、プロジェクト名（qwiklabs-gcp -...で始まる）をクリックしてから、クリック [<strong>データセットを作成</strong>]。</li>
</ol>
<p><img alt="BQ_CreateDataset.png" src="https://cdn.qwiklabs.com/QZ8JT8gSgMzUxN%2B7ZxxLmlnMDDAc8WARu5H8m2JhSbQ%3D"></p>
<ol start="2">
<li>
<p>[<strong>データセットを作成</strong>] ダイアログで、次の操作を行います。</p>
</li>
</ol>
<ul>
<li>[<strong>データセット ID</strong>] に「<strong>ecommerce</strong>」と入力します。</li>
<li>その他の値はデフォルトのままにします。</li>
</ul>
<p><img alt="create-dataset.png" src="https://cdn.qwiklabs.com/iLtK62bwChP7Z9ECizLQ1y%2FSe4QIld%2FKQKekZ69V%2FLg%3D"></p>
<ol start="3">
<li>[<strong>データセットを作成</strong>] をクリックします。</li>
</ol>

<h2 id="step8">BQML モデルタイプを選択し、オプションを指定する</h2>
<p>最初の特徴を選択したので、最初の ML モデルを BigQuery に作成する準備ができました。</p>
<p>モデルタイプは次の 2 つから選択します。</p>
<table>
<tr>
<td colspan="1" rowspan="1">
<p><strong>モデル</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>モデルタイプ</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>ラベルのデータ型</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>例</strong></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>予測</p>
</td>
<td colspan="1" rowspan="1">
<p>linear_reg</p>
</td>
<td colspan="1" rowspan="1">
<p>数値（通常は整数または浮動小数点数）</p>
</td>
<td colspan="1" rowspan="1">
<p>過去の売上データから翌年の売上を予測。</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>分類</p>
</td>
<td colspan="1" rowspan="1">
<p>logistic_reg</p>
</td>
<td colspan="1" rowspan="1">
<p>0 または 1 のバイナリ分類</p>
</td>
<td colspan="1" rowspan="1">
<p>コンテキストに応じて、メールを迷惑メールまたは迷惑メール以外に分類。</p>
</td>
</tr>
</table>
<p><strong>注:</strong> 機械学習で使用されているモデルタイプは他にも多数あります（ニューラル ネットワークやディシジョン ツリーなど）。これらは <a href="https://www.tensorflow.org/tutorials/">TensorFlow</a> などのライブラリで利用できます。執筆時点で、BQML は上記の 2 つをサポートしています。</p>
<p>どのモデルタイプを選択すればいいでしょうか。</p>
<p>訪問者を「将来購入する」か「将来購入しない」にバケット化しているため、分類モデルで <code>logistic_reg</code> を使用します。</p>
<p>次のクエリを入力して、モデルを作成し、モデル オプションを指定します。</p>

```sql
CREATE OR REPLACE MODEL `ecommerce.classification_model`
OPTIONS
(
model_type='logistic_reg',
labels = ['will_buy_on_return_visit']
)
AS

#standardSQL
SELECT
  * EXCEPT(fullVisitorId)
FROM

  # 特徴
  (SELECT
    fullVisitorId,
    IFNULL(totals.bounces, 0) AS bounces,
    IFNULL(totals.timeOnSite, 0) AS time_on_site
  FROM
    `data-to-insights.ecommerce.web_analytics`
  WHERE
    totals.newVisits = 1
    AND date BETWEEN '20160801' AND '20170430') # 最初の 9 か月分でトレーニング
  JOIN
  (SELECT
    fullvisitorid,
    IF(COUNTIF(totals.transactions > 0 AND totals.newVisits IS NULL) > 0, 1, 0) AS will_buy_on_return_visit
  FROM
      `data-to-insights.ecommerce.web_analytics`
  GROUP BY fullvisitorid)
  USING (fullVisitorId)
;
```

<p>次に、[<strong>実行</strong>] をクリックしてモデルのトレーニングを行います。</p>
<p>モデルのトレーニングが終わるのを待ちます（5～10 分）。</p>
<aside>
<b>注:</b> 利用可能なデータのすべてをモデルのトレーニングに使用することはできません。モデルの評価とテストのために、モデルにとって未知のデータポイントを別途残しておく必要があるためです。これを行うために、WHERE 句の条件を追加し、12 か月のデータセットのうち最初の 9 か月のセッション データのみをフィルタリングしてトレーニングに使用するようにします。

</aside>

<p>モデルのトレーニングが終わったら、「This was a CREATE operation. Results will not be shown」というメッセージが表示されます。これはモデルのトレーニングが正常に終了したことを示します。</p>
<p>ecommerce データセットの内部に注目し、<strong>classification_model</strong> が表示されていることを確かめます。</p>
<p>次に、未知の評価データに対してモデルの性能を評価します。</p>
<h2 id="step9">分類モデルの性能を評価する</h2>
<h3>性能の基準を選択する</h3>
<p>ML での分類に関する問題について、偽陽性率（ユーザーが戻ってきて購入すると予測したが、実際には購入しない）を最小限に抑え、真陽性率（ユーザーが戻ってきて購入すると予測し、実際に購入する）を最大限にすることを目指します。</p>
<p>次に示すような ROC（受信者操作特性）曲線でこの関係を視覚化します。ここでは、曲線の下の領域（AUC）を最大限にしようと試みています。</p>
<p><img alt="42fc12b6077e4784.png" src="https://cdn.qwiklabs.com/GNW5Bw%2B8bviep9OK201QGPzaAEnKKyoIkDChUHeVdFw%3D"></p>
<p>BQML では、<strong>roc_auc</strong> は、トレーニングを行った ML モデルを評価するときにクエリ可能なフィールドです。</p>
<p>トレーニングが完了したので、<code>ML.EVALUATE</code> を使用して、このクエリに対するモデルの性能を評価できます。</p>

```sql
SELECT
  roc_auc,
  CASE
    WHEN roc_auc > .9 THEN 'good'
    WHEN roc_auc > .8 THEN 'fair'
    WHEN roc_auc > .7 THEN 'decent'
    WHEN roc_auc > .6 THEN 'not great'
  ELSE 'poor' END AS model_quality
FROM
  ML.EVALUATE(MODEL ecommerce.classification_model,  (

SELECT
  * EXCEPT(fullVisitorId)
FROM

  # 特徴
  (SELECT
    fullVisitorId,
    IFNULL(totals.bounces, 0) AS bounces,
    IFNULL(totals.timeOnSite, 0) AS time_on_site
  FROM
    `data-to-insights.ecommerce.web_analytics`
  WHERE
    totals.newVisits = 1
    AND date BETWEEN '20170501' AND '20170630') # 2 か月分で評価
  JOIN
  (SELECT
    fullvisitorid,
    IF(COUNTIF(totals.transactions > 0 AND totals.newVisits IS NULL) > 0, 1, 0) AS will_buy_on_return_visit
  FROM
      `data-to-insights.ecommerce.web_analytics`
  GROUP BY fullvisitorid)
  USING (fullVisitorId)

));
```

<p>次のような結果が表示されます。</p>
<table>
<tr>
<td colspan="1" rowspan="1">
<p>Row</p>
</td>
<td colspan="1" rowspan="1">
<p>roc_auc</p>
</td>
<td colspan="1" rowspan="1">
<p>model_quality</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>1</p>
</td>
<td colspan="1" rowspan="1">
<p>0.724588</p>
</td>
<td colspan="1" rowspan="1">
<p>decent</p>
</td>
</tr>
</table>
<p>モデルを評価すると、<strong>roc_auc</strong> で 0.72 を取得します。これは、モデルが妥当（decent）なレベルではあるものの、予測力が特に優れているわけではないことを示しています。目標は、曲線の下の領域を可能な限り 1.0 に近づけることであるため、改善の余地はまだ残されています。</p>

<h2 id="step10">特徴エンジニアリングでモデル性能を強化する</h2>
<p>これまでもそれとなくふれてきましたが、訪問者の最初のセッションと再度訪問して購入する可能性の関係をモデルに理解させるうえで役立つデータセットの特徴は他にもたくさん存在します。</p>
<p>新しい特徴をいくつか追加し、<code>classification_model_2</code> という名前の 2 番目の機械学習モデルを作成します。</p>
<ul>
<li>初回訪問時に訪問者は購入手続きをどこまで進めていたか</li>
<li>訪問者はどこからアクセスしたか（トラフィック ソース: オーガニック検索、参照元サイトなど）</li>
<li>デバイスのカテゴリ（モバイル、タブレット、パソコン）</li>
<li>地理情報（国）</li>
</ul>
<p>次のクエリを実行して、この 2 番目のモデルを作成します。</p>

```sql
CREATE OR REPLACE MODEL `ecommerce.classification_model_2`
OPTIONS
  (model_type='logistic_reg', labels = ['will_buy_on_return_visit']) AS

WITH all_visitor_stats AS (
SELECT
  fullvisitorid,
  IF(COUNTIF(totals.transactions > 0 AND totals.newVisits IS NULL) > 0, 1, 0) AS will_buy_on_return_visit
  FROM `data-to-insights.ecommerce.web_analytics`
  GROUP BY fullvisitorid
)

# 新しい特徴に追加
SELECT * EXCEPT(unique_session_id) FROM (

  SELECT
      CONCAT(fullvisitorid, CAST(visitId AS STRING)) AS unique_session_id,

      # ラベル
      will_buy_on_return_visit,

      MAX(CAST(h.eCommerceAction.action_type AS INT64)) AS latest_ecommerce_progress,

      # サイトでの行動
      IFNULL(totals.bounces, 0) AS bounces,
      IFNULL(totals.timeOnSite, 0) AS time_on_site,
      IFNULL(totals.pageviews, 0) AS pageviews,

      # 訪問経路
      trafficSource.source,
      trafficSource.medium,
      channelGrouping,

      # モバイルか PC か
      device.deviceCategory,

      # 地域
      IFNULL(geoNetwork.country, "") AS country

  FROM `data-to-insights.ecommerce.web_analytics`,
     UNNEST(hits) AS h

    JOIN all_visitor_stats USING(fullvisitorid)

  WHERE 1=1
    # 初回訪問のみ予測
    AND totals.newVisits = 1
    AND date BETWEEN '20160801' AND '20170430' # 9 か月分でトレーニング

  GROUP BY
  unique_session_id,
  will_buy_on_return_visit,
  bounces,
  time_on_site,
  totals.pageviews,
  trafficSource.source,
  trafficSource.medium,
  channelGrouping,
  device.deviceCategory,
  country
);
```

<aside>
<b>注:</b> これは新しいモデルですが、ここでも同じ最初の 9 か月のデータを用いてトレーニングします。優れたモデル出力は優れた入力特徴に起因することを確信できるように、新しいトレーニング データや異なるトレーニング データではなく、同じトレーニング データセットを使用することが重要です。

</aside>
<p>データセット クエリのトレーニングに追加された重要な新しい特徴は、各訪問者がセッションで到達した購入手続きの進捗状況で、フィールド <code>hits.eCommerceAction.action_type</code> に記録されます。<a href="https://support.google.com/analytics/answer/3437719?hl=ja">フィールド定義</a>でそのフィールドを検索すると、「6 = Completed Purchase」のフィールド マッピングが表示されます。</p>
<p>余談ですが、ウェブ解析データセットには、<a href="https://cloud.google.com/bigquery/docs/reference/standard-sql/arrays">ARRAYS</a> のようなネストされ繰り返されたフィールドがあり、データセットではこれらを別個の行に分ける必要があります。これは、UNNEST() 関数を使用して行います（上記のクエリで確認できます）。</p>
<p>新しいモデルのトレーニングが終わるのを待ちます（5～10 分）。</p>

<p>この新しいモデルを評価し、より優れた予測能力が備わっているかどうかを確認します。</p>

```sql
#standardSQL
SELECT
  roc_auc,
  CASE
    WHEN roc_auc > .9 THEN 'good'
    WHEN roc_auc > .8 THEN 'fair'
    WHEN roc_auc > .7 THEN 'decent'
    WHEN roc_auc > .6 THEN 'not great'
  ELSE 'poor' END AS model_quality
FROM
  ML.EVALUATE(MODEL ecommerce.classification_model_2,  (

WITH all_visitor_stats AS (
SELECT
  fullvisitorid,
  IF(COUNTIF(totals.transactions > 0 AND totals.newVisits IS NULL) > 0, 1, 0) AS will_buy_on_return_visit
  FROM `data-to-insights.ecommerce.web_analytics`
  GROUP BY fullvisitorid
)

# 新しい特徴に追加
SELECT * EXCEPT(unique_session_id) FROM (

  SELECT
      CONCAT(fullvisitorid, CAST(visitId AS STRING)) AS unique_session_id,

      # ラベル
      will_buy_on_return_visit,

      MAX(CAST(h.eCommerceAction.action_type AS INT64)) AS latest_ecommerce_progress,

      # サイトでの行動
      IFNULL(totals.bounces, 0) AS bounces,
      IFNULL(totals.timeOnSite, 0) AS time_on_site,
      totals.pageviews,

      # 訪問経路
      trafficSource.source,
      trafficSource.medium,
      channelGrouping,

      # モバイルか PC か
      device.deviceCategory,

      # 地域
      IFNULL(geoNetwork.country, "") AS country

  FROM `data-to-insights.ecommerce.web_analytics`,
     UNNEST(hits) AS h

    JOIN all_visitor_stats USING(fullvisitorid)

  WHERE 1=1
    # 初回訪問のみ予測
    AND totals.newVisits = 1
    AND date BETWEEN '20170501' AND '20170630' # 2 か月分で評価

  GROUP BY
  unique_session_id,
  will_buy_on_return_visit,
  bounces,
  time_on_site,
  totals.pageviews,
  trafficSource.source,
  trafficSource.medium,
  channelGrouping,
  device.deviceCategory,
  country
)
));
```

<p>（出力）</p>
<table>
<tr>
<td colspan="1" rowspan="1">
<p>Row</p>
</td>
<td colspan="1" rowspan="1">
<p>roc_auc</p>
</td>
<td colspan="1" rowspan="1">
<p>model_quality</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>1</p>
</td>
<td colspan="1" rowspan="1">
<p>0.910382</p>
</td>
<td colspan="1" rowspan="1">
<p>good</p>
</td>
</tr>
</table>
<p>この新しいモデルでは、<strong>roc_auc</strong> で 0.91 を取得するようになりました。これは、最初のモデルよりも著しく向上しています。</p>
<p>モデルをトレーニングしたので、今度は予測を行います。</p>

<h2 id="step11">新しい訪問者のうち、戻ってきて購入するユーザーを予測する</h2>
<p>次に、新しい訪問者がどのくらい戻ってきて購入するかを予測するためのクエリを作成します。</p>
<p>以下の予測クエリでは、強化済みの分類モデルを使用して、Google Merchandise Store への初めての訪問者が後の訪問で購入する確率を予測します。</p>

```sql
#standardSQL
SELECT
*
FROM
  ml.PREDICT(MODEL `ecommerce.classification_model_2`,
   (

WITH all_visitor_stats AS (
SELECT
  fullvisitorid,
  IF(COUNTIF(totals.transactions > 0 AND totals.newVisits IS NULL) > 0, 1, 0) AS will_buy_on_return_visit
  FROM `data-to-insights.ecommerce.web_analytics`
  GROUP BY fullvisitorid
)

  SELECT
      CONCAT(fullvisitorid, '-',CAST(visitId AS STRING)) AS unique_session_id,

      # ラベル
      will_buy_on_return_visit,

      MAX(CAST(h.eCommerceAction.action_type AS INT64)) AS latest_ecommerce_progress,

      # サイトでの行動
      IFNULL(totals.bounces, 0) AS bounces,
      IFNULL(totals.timeOnSite, 0) AS time_on_site,
      totals.pageviews,

      # 訪問経路
      trafficSource.source,
      trafficSource.medium,
      channelGrouping,

      # モバイルか PC か
      device.deviceCategory,

      # 地域
      IFNULL(geoNetwork.country, "") AS country

  FROM `data-to-insights.ecommerce.web_analytics`,
     UNNEST(hits) AS h

    JOIN all_visitor_stats USING(fullvisitorid)

  WHERE
    # 初回訪問のみ予測
    totals.newVisits = 1
    AND date BETWEEN '20170701' AND '20170801' # 1 か月分でテスト

  GROUP BY
  unique_session_id,
  will_buy_on_return_visit,
  bounces,
  time_on_site,
  totals.pageviews,
  trafficSource.source,
  trafficSource.medium,
  channelGrouping,
  device.deviceCategory,
  country
)

)

ORDER BY
  predicted_will_buy_on_return_visit DESC;
```

<p>予測は、最後の 1 か月（12 か月中）のデータセットで行われます。</p>

<p>モデルは、2017 年 7 月の e コマース セッションに対する予測を出力するようになります。新たに追加された 3 つのフィールドを確認できます。</p>
<ul>
<li>predicted_will_buy_on_return_visit: 訪問者が後で購入することを、モデルが予測しているかどうか（1 = yes）</li>
<li>predicted_will_buy_on_return_visit_probs.label: yes または no に関するバイナリ分類子</li>
<li>predicted_will_buy_on_return_visit.prob: その予測に対するモデルの信頼度（1 = 100%）</li>
</ul>
<p><img alt="710c0c0b9abf827b.png" src="https://cdn.qwiklabs.com/fEttKDSQAqoD3sZWAo04MVkXGfM%2ByRDnOQST%2FaCRZwg%3D"></p>
<h2 id="step12">結果</h2>
<ul>
<li>
<p>上位 6% の初回訪問者（予測された確率を降順で並べ替え済み）の 6% 以上が後の訪問で購入しました。</p>
</li>
<li>
<p>これらのユーザーは、後の訪問で購入に至った初回訪問者全体のほぼ 50% に相当します。</p>
</li>
<li>
<p>全体として、初回訪問者の 0.7% しか後の訪問で購入しませんでした。</p>
</li>
<li>
<p>初回訪問者の上位 6% にターゲットを絞ると、マーケティング ROI は、全員をターゲットにした場合に比べ 9 倍も向上します。</p>
</li>
</ul>
<h2 id="step13">追加情報</h2>
<p><strong>ヒント:</strong> 新しいデータで既存のモデルの再トレーニングを行う場合、トレーニング時間を短縮するには、<code>warm_start = true</code> をモデルのオプションに追加します。特徴の列を変更することはできません（変更すると、新しいモデルが必要になります）。</p>
<p><strong>roc_auc</strong> は、モデル評価で使用できるパフォーマンス指標のひとつにすぎず、他にも <a href="https://en.wikipedia.org/wiki/Precision_and_recall">accuracy、precision、recall</a> を使用できます。信頼すべきパフォーマンス指標を知ることは、全体の目標が何かによって大きく左右されます。</p>
<h2 id="step14">探索できるその他のデータセット</h2>
<p>タクシー運賃を予測する場合など、他のデータセットに対するモデルの構築を試してみるには、以下のリンクから <strong>bigquery-public-data</strong> プロジェクトを利用できます。</p>
<ul>
<li>
<p><a href="https://bigquery.cloud.google.com/table/bigquery-public-data:chicago_taxi_trips.taxi_trips">https://bigquery.cloud.google.com/table/bigquery-public-data:chicago_taxi_trips.taxi_trips</a></p>
</li>
</ul>

<h2 id="step16">これで完了です。</h2>
<p>e コマースの訪問者を分類する ML モデルを BigQuery で無事構築できました。</p>

# BigQuery ML 予測モデルによるタクシー運賃の予測
<h2 id="step2">概要</h2>
<p><a href="https://cloud.google.com/bigquery/">BigQuery</a> は、Google が低料金で提供する NoOps のフルマネージド分析データベースです。インフラストラクチャを所有して管理したりデータベース管理者を配置したりすることなく、テラバイト単位の大規模なデータをクエリできます。</p>
<p>BigQuery の新機能である <a href="https://cloud.google.com/bigquery/docs/bigqueryml-analyst-start">BigQuery ML</a>（BQML、ベータ版）を使用すれば、最小限のコーディングで機械学習モデルの作成、トレーニング、評価、予測が可能になります。</p>
<p>このラボでは、BigQuery の一般公開データセットの中から、数百万件に及ぶニューヨーク市内のタクシー賃走データを探索します。その後、機械学習モデルを BigQuery 内に作成し、モデル入力に基づいてタクシー運賃を予測します。最後に、モデルの性能を評価し、そのモデルで予測を行います。</p>
<h3>目標</h3>
<p>このラボでは、次のタスクの実行方法について学びます。</p>
<ul>
<li>BigQuery を使用して一般公開データセットを見つける。</li>
<li>タクシーの一般公開データセットをクエリし、探索する。</li>
<li>バッチ予測に使用するトレーニングと評価のデータセットを作成する。</li>
<li>予測（線形回帰）モデルを BQML に作成する。</li>
<li>機械学習モデルの性能を評価する。</li>
</ul>

<h3>BigQuery Console を開く</h3>
<p>Google Cloud Console で [<strong>ナビゲーション メニュー</strong>] &gt; [<strong>BigQuery</strong>] を選択します。</p>
<p><img alt="BigQuery_menu.png" src="https://cdn.qwiklabs.com/QwUX6algdXPgIThqBlgB2rnqGkoQUUANkl3xicD06uc%3D"></p>
<p>[<strong>Cloud Console の BigQuery へようこそ</strong>] メッセージボックスが開きます。このメッセージボックスには、クイックスタート ガイドとリリースノートへのリンクが表示されています。</p>
<p>[<strong>完了</strong>] をクリックします。</p>
<p>BigQuery コンソールが開きます。</p>
<p>
</p>
<p><img alt="bq-console.png" src="https://cdn.qwiklabs.com/vd3QNHGB4BAyHjAvOHw9qD0iqCPNaHAzY657z%2FGWLtY%3D"></p>

<h2 id="step4">ニューヨーク市のタクシーデータを探索する</h2>
<p><strong>質問: </strong>2015 年のイエロー タクシーの毎月の賃走回数はどれくらいですか。</p>
<p>次の SQL コードをコピーしてクエリエディタに貼り付けます。</p>

```sql
#standardSQL
SELECT
  TIMESTAMP_TRUNC(pickup_datetime,
    MONTH) month,
  COUNT(*) trips
FROM
  `bigquery-public-data.new_york.tlc_yellow_trips_2015`
GROUP BY
  1
ORDER BY
  1
```

<p>[<strong>実行</strong>] をクリックします。</p>
<p>次の結果が表示されます。</p>
<p><img alt="BQML_taxi_mo_trips.png" src="https://cdn.qwiklabs.com/i3PmHY7jPV1XuAIql%2FDlkUHWFWuPJLcW1VECFP9P%2BuI%3D"></p>
<p>2015 年におけるニューヨーク市内のタクシー賃走回数は、毎月 1,000 万回を超えていることがわかります。これはかなりの量です。</p>

<p><strong>質問: </strong>2015 年のイエロー タクシーの平均速度は？</p>
<p>前のクエリを以下に置き換え、[<strong>実行</strong>] をクリックします。</p>

```sql
#standardSQL
SELECT
  EXTRACT(HOUR
  FROM
    pickup_datetime) hour,
  ROUND(AVG(trip_distance / TIMESTAMP_DIFF(dropoff_datetime,
        pickup_datetime,
        SECOND))*3600, 1) speed
FROM
  `bigquery-public-data.new_york.tlc_yellow_trips_2015`
WHERE
  trip_distance > 0
  AND fare_amount/trip_distance BETWEEN 2
  AND 10
  AND dropoff_datetime > pickup_datetime
GROUP BY
  1
ORDER BY
  1
```

<p>次の結果が表示されます。</p>
<p><img alt="BQML_taxi_hr_speed.png" src="https://cdn.qwiklabs.com/%2BF8Mj8HqPM9b8%2Fna1JYPy66xTKDcUu%2BQs1oh5Gy07A4%3D"></p>
<p>日中の平均速度はおよそ時速 11～12 マイルですが、午前 5 時の平均速度はほぼ倍の時速 21 マイルになっています。午前 5 時は交通量が少ないはずなので、これは直感的に理解できます。</p>

<h2 id="step5">予測対象を特定する</h2>
<p>次に、機械学習モデルを BigQuery に作成し、過去の賃走データセットに基づいてニューヨーク市のタクシー運賃を予測します。乗車前に運賃を予測できれば、乗客とタクシー会社の双方が、より効率的に乗車・配車の計画を立てられるようになります。</p>
<h2 id="step6">特徴を選択し、トレーニング データセットを作成する</h2>
<p>ニューヨーク市のイエロー タクシーのデータセットは、市が提供する<a href="https://cloud.google.com/bigquery/public-data/nyc-tlc-trips">一般公開データセット</a>です。これは BigQuery に読み込まれ、自由に探索できるようになっています。フィールドの全一覧を<a href="https://bigquery.cloud.google.com/table/nyc-tlc:yellow.trips">こちら</a>で確認してから、<a href="https://bigquery.cloud.google.com/table/nyc-tlc:yellow.trips?tab=preview">データセットをプレビュー</a>し、機械学習モデルが過去のタクシー賃走と運賃の関係を理解するのに役立つ特徴を見つけます。</p>
<p>以下のフィールドが運賃予測モデルに適した入力であるかどうかをテストします。</p>
<ul>
<li>通行料</li>
<li>運賃</li>
<li>時間帯</li>
<li>乗車場所</li>
<li>降車場所</li>
<li>乗客の人数</li>
</ul>
<p>クエリを以下に置き換えます。</p>

```sql
#standardSQL
WITH params AS (
    SELECT
    1 AS TRAIN,
    2 AS EVAL
    ),

  daynames AS
    (SELECT ['Sun', 'Mon', 'Tues', 'Wed', 'Thurs', 'Fri', 'Sat'] AS daysofweek),

  taxitrips AS (
  SELECT
    (tolls_amount + fare_amount) AS total_fare,
    daysofweek[ORDINAL(EXTRACT(DAYOFWEEK FROM pickup_datetime))] AS dayofweek,
    EXTRACT(HOUR FROM pickup_datetime) AS hourofday,
    pickup_longitude AS pickuplon,
    pickup_latitude AS pickuplat,
    dropoff_longitude AS dropofflon,
    dropoff_latitude AS dropofflat,
    passenger_count AS passengers
  FROM
    `nyc-tlc.yellow.trips`, daynames, params
  WHERE
    trip_distance > 0 AND fare_amount > 0
    AND MOD(ABS(FARM_FINGERPRINT(CAST(pickup_datetime AS STRING))),1000) = params.TRAIN
  )

  SELECT *
  FROM taxitrips
```

<p>このクエリについて、以下の点に注意します。</p>
<ol>
<li>クエリのメインの部分は一番下の「<code>SELECT * FROM taxitrips</code>」です。</li>
<li>
<code>taxitrips</code> がニューヨーク市のデータセットの抽出の大部分を担い、<code>SELECT</code> にトレーニングの特徴とラベルが含まれます。</li>
<li>
<code>WHERE</code> でトレーニングを行わないデータを取り除きます。</li>
<li>
<code>WHERE</code> には、データの 1/1,000 のみを取得するためのサンプリング句も含まれます。</li>
<li>独立した <code>EVAL</code> セットを簡単に構築できるよう、<code>TRAIN</code> という変数を定義しています。</li>
</ol>
<p>このクエリの目的について理解を深めたところで、[<strong>実行</strong>] をクリックします。</p>
<p>次のような結果が表示されます。</p>
<p><img alt="3784193f53252195.png" src="https://cdn.qwiklabs.com/9%2FORyElhKMLalupP%2FuG%2BZqE%2FTjLX4XYCXnvsEmGLang%3D"></p>
<p>どれがラベル（正解）ですか。</p>
<p><code>total_fare</code> がラベル（今回の予測対象）です。このフィールドは <code>tolls_amount</code> と <code>fare_amount</code> から作成しています。チップは任意であるため、モデルでは無視できます。</p>

<h2 id="step7">モデルを格納する BigQuery データセットを作成する</h2>
<p>このセクションでは、新しい BigQuery データセットを作成します。このデータセットに ML モデルを格納します。</p>
<ol>
<li>
<p>左側の [リソース] パネルで、Qwiklabs の GCP プロジェクト ID を選択します。</p>
</li>
<li>
<p>次に、ページの右側にある [<strong>データセットを作成</strong>] をクリックします。</p>
</li>
<li>
<p>[データセットを作成] ダイアログに、次のように入力します。</p>
</li>
</ol>
<ul>
<li>[<strong>データセット ID</strong>] に「<strong>taxi</strong>」と入力します。</li>
<li>その他の値はデフォルトのままにします。</li>
</ul>
<p><img alt="create-dataset.png" src="https://cdn.qwiklabs.com/QGOFCQMb3UNnOf2dByXcmcH7%2BX6xwnoQFX0Fdo7fRLU%3D"></p>
<ol start="4">
<li>
<p>[<strong>データセットを作成</strong>] をクリックします。</p>
</li>
</ol>

<h2 id="step8">BQML モデルタイプを選択し、オプションを指定する</h2>
<p>最初の特徴を選択したので、最初の ML モデルを BigQuery に作成する準備ができました。</p>
<p>次のモデルタイプから選択できます。</p>
<ul>
<li>
<b>予測</b>: 線形回帰による、翌月の売上などの数値の予測（linear_reg）。</li>
<li>
<b>分類</b>: ロジスティック回帰による、迷惑メールの分類などのバイナリ分類またはマルチクラス分類（logistic_reg）。</li>
<li>
<b>クラスタリング</b>: 原因分析のために教師なし学習を使用する場合に利用できる、K 平均法クラスタリング（kmeans）。</li>
</ul>
<aside>
<strong>注:</strong> 機械学習で使用されているモデルタイプは他にも多数あります（ニューラル ネットワークやディシジョン ツリーなど）。これらは <a href="https://www.tensorflow.org/tutorials/">TensorFlow</a> などのライブラリで利用可能です。BQML は現時点で上記の 3 つをサポートしています。詳細については、<a href="https://cloud.google.com/bigquery/docs/reference/standard-sql/bigqueryml-syntax-create">BQML ロードマップ</a>をご覧ください。

</aside>

<p>次のクエリを入力して、モデルを作成し、モデル オプションを指定します。</p>

```sql
CREATE or REPLACE MODEL taxi.taxifare_model
OPTIONS
  (model_type='linear_reg', labels=['total_fare']) AS

WITH params AS (
    SELECT
    1 AS TRAIN,
    2 AS EVAL
    ),

  daynames AS
    (SELECT ['Sun', 'Mon', 'Tues', 'Wed', 'Thurs', 'Fri', 'Sat'] AS daysofweek),

  taxitrips AS (
  SELECT
    (tolls_amount + fare_amount) AS total_fare,
    daysofweek[ORDINAL(EXTRACT(DAYOFWEEK FROM pickup_datetime))] AS dayofweek,
    EXTRACT(HOUR FROM pickup_datetime) AS hourofday,
    pickup_longitude AS pickuplon,
    pickup_latitude AS pickuplat,
    dropoff_longitude AS dropofflon,
    dropoff_latitude AS dropofflat,
    passenger_count AS passengers
  FROM
    `nyc-tlc.yellow.trips`, daynames, params
  WHERE
    trip_distance > 0 AND fare_amount > 0
    AND MOD(ABS(FARM_FINGERPRINT(CAST(pickup_datetime AS STRING))),1000) = params.TRAIN
  )

  SELECT *
  FROM taxitrips
```

<p>次に、[<strong>実行</strong>] をクリックしてモデルのトレーニングを行います。</p>
<p>モデルのトレーニングが終わるのを待ちます（5～10 分）。</p>
<p>モデルのトレーニングが終わったら、「This was a CREATE operation. Results will not be shown」というメッセージが表示されます。</p>
<p>タクシーのデータセット内に <strong>taxifare_model</strong> があることを確認します。</p>
<p><img alt="taxifare-model.png" src="https://cdn.qwiklabs.com/3e%2B%2Bn9FtlJ4zZDbP1edDClbadjTZmcifikRDzSbd3fA%3D"></p>
<p>次に、未知の評価データに対するモデルの性能を評価します。</p>

<h2 id="step9">分類モデルの性能を評価する</h2>
<h3>性能の評価基準を選択する</h3>
<p>線形回帰モデルには、<a href="https://en.wikipedia.org/wiki/Root-mean-square_deviation">二乗平均平方根誤差（RMSE）</a>などの損失指標を使用します。RMSE が下がるまでトレーニングを続け、モデルを改善していきます。</p>
<p>BQML では、トレーニング済みの ML モデルを評価するときにクエリ可能なフィールドとして <code>mean_squared_error</code> があります。RMSE を取得するには <code>SQRT()</code> を追加します。</p>
<p>トレーニングが完了したので、<code>ML.EVALUATE</code> を使用したクエリでモデルの性能を評価できます。以下をコピーしてクエリエディタに貼り付け、[<strong>実行</strong>] をクリックします。</p>

```sql
#standardSQL
SELECT
  SQRT(mean_squared_error) AS rmse
FROM
  ML.EVALUATE(MODEL taxi.taxifare_model,
  (

  WITH params AS (
    SELECT
    1 AS TRAIN,
    2 AS EVAL
    ),

  daynames AS
    (SELECT ['Sun', 'Mon', 'Tues', 'Wed', 'Thurs', 'Fri', 'Sat'] AS daysofweek),

  taxitrips AS (
  SELECT
    (tolls_amount + fare_amount) AS total_fare,
    daysofweek[ORDINAL(EXTRACT(DAYOFWEEK FROM pickup_datetime))] AS dayofweek,
    EXTRACT(HOUR FROM pickup_datetime) AS hourofday,
    pickup_longitude AS pickuplon,
    pickup_latitude AS pickuplat,
    dropoff_longitude AS dropofflon,
    dropoff_latitude AS dropofflat,
    passenger_count AS passengers
  FROM
    `nyc-tlc.yellow.trips`, daynames, params
  WHERE
    trip_distance > 0 AND fare_amount > 0
    AND MOD(ABS(FARM_FINGERPRINT(CAST(pickup_datetime AS STRING))),1000) = params.EVAL
  )

  SELECT *
  FROM taxitrips

  ))
```

<p><code>params.EVAL</code> フィルタで、異なるタクシー賃走データセットに対してモデルを評価します。</p>
<p>モデルの実行後、モデルの結果を確認します（モデルの実際の RMSE 値はわずかに異なる場合があります）。</p>
<table>
<tr>
<td colspan="1" rowspan="1">
<p><strong>行</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>rmse</strong></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>1</p>
</td>
<td colspan="1" rowspan="1">
<p>9.477056435999074</p>
</td>
</tr>
</table>
<p>モデルを評価したら、<strong>RMSE</strong> が 9.47 になりました。ここで取得したのは二乗平均平方根誤差（RMSE）であるため、誤差 9.47 は total_fare と同じ単位で評価できます。したがって、+-$9.47 になります。</p>
<p>この損失指標で、このモデルを本番環境に使用してもいいかどうかは、モデルのトレーニング開始前に設定するベンチマーク基準によって異なります。ベンチマークとは、許容できる最低レベルのモデルの性能と精度を確立するための基準です。</p>

<h2 id="step10">タクシー運賃を予測する</h2>
<p>次に、作成したモデルを使用して予測を行うためのクエリを作成します。以下をコピーしてクエリエディタに貼り付け、[<strong>実行</strong>] をクリックします。</p>

```sql
#standardSQL
SELECT
*
FROM
  ml.PREDICT(MODEL `taxi.taxifare_model`,
   (

 WITH params AS (
    SELECT
    1 AS TRAIN,
    2 AS EVAL
    ),

  daynames AS
    (SELECT ['Sun', 'Mon', 'Tues', 'Wed', 'Thurs', 'Fri', 'Sat'] AS daysofweek),

  taxitrips AS (
  SELECT
    (tolls_amount + fare_amount) AS total_fare,
    daysofweek[ORDINAL(EXTRACT(DAYOFWEEK FROM pickup_datetime))] AS dayofweek,
    EXTRACT(HOUR FROM pickup_datetime) AS hourofday,
    pickup_longitude AS pickuplon,
    pickup_latitude AS pickuplat,
    dropoff_longitude AS dropofflon,
    dropoff_latitude AS dropofflat,
    passenger_count AS passengers
  FROM
    `nyc-tlc.yellow.trips`, daynames, params
  WHERE
    trip_distance > 0 AND fare_amount > 0
    AND MOD(ABS(FARM_FINGERPRINT(CAST(pickup_datetime AS STRING))),1000) = params.EVAL
  )

  SELECT *
  FROM taxitrips

));
```

<p>タクシー運賃に対するモデルの予測が、その賃走の実際の運賃とその他の特徴とともに表示されます。結果は以下のようになります。</p>
<p><img alt="タクシーに関する予測" src="https://cdn.qwiklabs.com/vOie0YpofU3oI7zMJrG9OJT%2Bom89nokwCtceenccSc0%3D"></p>

<h2 id="step11">特徴量エンジニアリングによるモデルの強化</h2>
<p>機械学習モデルの構築は反復的なプロセスです。初期モデルの性能を評価した後、前に戻って機能や行をプルーニングし、改善の余地がないか確認します。</p>
<h3>トレーニング データセットのフィルタリング</h3>
<p>タクシー運賃の一般的な統計情報を表示してみましょう。以下をコピーしてクエリエディタに貼り付け、[<strong>実行</strong>] をクリックします。</p>

```sql
SELECT
  COUNT(fare_amount) AS num_fares,
  MIN(fare_amount) AS low_fare,
  MAX(fare_amount) AS high_fare,
  AVG(fare_amount) AS avg_fare,
  STDDEV(fare_amount) AS stddev
FROM
`nyc-tlc.yellow.trips`
# 1,108,779,463 件
```

<p>次のような出力が返されます。</p>
<p><img alt="フィルタ" src="https://cdn.qwiklabs.com/uX%2Fr9WrJRi7usPBEaElFe7gFgAMrm0Q3exGSbGCeV1I%3D"></p>
<p>データセットに異常値（負の値の運賃や $50,000 を超える運賃など）があるのがわかります。モデルが不自然な外れ値を学習しないように、タクシー運賃に関する知識を適用しましょう。</p>
<p>$6 から $200 までの間の運賃のみにデータを制限しましょう。以下をコピーしてクエリエディタに貼り付け、[<strong>実行</strong>] をクリックします。</p>

```sql
SELECT
  COUNT(fare_amount) AS num_fares,
  MIN(fare_amount) AS low_fare,
  MAX(fare_amount) AS high_fare,
  AVG(fare_amount) AS avg_fare,
  STDDEV(fare_amount) AS stddev
FROM
`nyc-tlc.yellow.trips`
WHERE trip_distance > 0 AND fare_amount BETWEEN 6 and 200
# 843,834,902 件
```

<p>次のような出力が返されます。</p>
<p><img alt="フィルタ 2" src="https://cdn.qwiklabs.com/%2Fyxad%2F%2BGAAIWNmbuq1vVTypMqkwK4B7Vpe145Xa8glc%3D"></p>
<p>結果は多少絞られました。今回はニューヨーク市にターゲットを絞っているため、ここで移動距離を制限しましょう。</p>
<p>以下をコピーしてクエリエディタに貼り付け、[<strong>実行</strong>] をクリックします。</p>

```sql
SELECT
  COUNT(fare_amount) AS num_fares,
  MIN(fare_amount) AS low_fare,
  MAX(fare_amount) AS high_fare,
  AVG(fare_amount) AS avg_fare,
  STDDEV(fare_amount) AS stddev
FROM
`nyc-tlc.yellow.trips`
WHERE trip_distance > 0 AND fare_amount BETWEEN 6 and 200
    AND pickup_longitude > -75 #タクシーの移動距離の制限
    AND pickup_longitude < -73
    AND dropoff_longitude > -75
    AND dropoff_longitude < -73
    AND pickup_latitude > 40
    AND pickup_latitude < 42
    AND dropoff_latitude > 40
    AND dropoff_latitude < 42
    # 827,365,869 件
```

<p>次のような出力が返されます。</p>
<p><img alt="フィルタ 3" src="https://cdn.qwiklabs.com/wGM1HpCBLaRa7UNNoa0%2BKV0FjVOqyO48cMDBd4wq52k%3D"></p>
<p>この学習モデルには、まだ 8 億を超える乗車件数を含む大きなサイズのトレーニング データセットがあります。以下の新しい制約を使用してモデルを再トレーニングし、それがどの程度良好に機能するかを見てみましょう。</p>
<h3>モデルの再トレーニング</h3>
<p>新しいモデル <code>taxi.taxifare_model_2</code> を呼び出して、合計運賃を予測するために線形回帰モデルの再トレーニングを行います。乗車と降車の間の<a href="https://en.wikipedia.org/wiki/Euclidean_distance">ユークリッド距離</a>（直線）のための計算機能もいくつか追加しました。</p>
<p>以下をコピーしてクエリエディタに貼り付け、[<strong>実行</strong>] をクリックします。</p>

```sql
CREATE OR REPLACE MODEL taxi.taxifare_model_2
OPTIONS
  (model_type='linear_reg', labels=['total_fare']) AS


WITH params AS (
    SELECT
    1 AS TRAIN,
    2 AS EVAL
    ),

  daynames AS
    (SELECT ['Sun', 'Mon', 'Tues', 'Wed', 'Thurs', 'Fri', 'Sat'] AS daysofweek),

  taxitrips AS (
  SELECT
    (tolls_amount + fare_amount) AS total_fare,
    daysofweek[ORDINAL(EXTRACT(DAYOFWEEK FROM pickup_datetime))] AS dayofweek,
    EXTRACT(HOUR FROM pickup_datetime) AS hourofday,
    SQRT(POW((pickup_longitude - dropoff_longitude),2) + POW(( pickup_latitude - dropoff_latitude), 2)) as dist, #乗車と降車の間のユークリッド距離
    SQRT(POW((pickup_longitude - dropoff_longitude),2)) as longitude, #乗車と降車の間のユークリッド距離（経度）
    SQRT(POW((pickup_latitude - dropoff_latitude), 2)) as latitude, #乗車と降車の間のユークリッド距離（緯度）
    passenger_count AS passengers
  FROM
    `nyc-tlc.yellow.trips`, daynames, params
WHERE trip_distance > 0 AND fare_amount BETWEEN 6 and 200
    AND pickup_longitude > -75 #タクシーの移動距離の制限
    AND pickup_longitude < -73
    AND dropoff_longitude > -75
    AND dropoff_longitude < -73
    AND pickup_latitude > 40
    AND pickup_latitude < 42
    AND dropoff_latitude > 40
    AND dropoff_latitude < 42
    AND MOD(ABS(FARM_FINGERPRINT(CAST(pickup_datetime AS STRING))),1000) = params.TRAIN
  )

  SELECT *
  FROM taxitrips
```

<p>モデルの再トレーニングには数分かかります。コンソールに以下のメッセージが表示されたら、次のステップに進むことができます。</p>
<p><img alt="再トレーニングされたモデル" src="https://cdn.qwiklabs.com/WsL7xhxMxQFpf5HkKpC0P3bFAlLxeStYzA3uLFCPa6E%3D"></p>
<h3>新しいモデルの評価</h3>
<p>線形回帰モデルが最適化されたので、それを使ってデータセットを評価し、どの程度機能するかを見てみましょう。以下をコピーしてクエリエディタに貼り付け、[<strong>実行</strong>] をクリックします。</p>

```sql
SELECT
  SQRT(mean_squared_error) AS rmse
FROM
  ML.EVALUATE(MODEL taxi.taxifare_model_2,
  (

  WITH params AS (
    SELECT
    1 AS TRAIN,
    2 AS EVAL
    ),

  daynames AS
    (SELECT ['Sun', 'Mon', 'Tues', 'Wed', 'Thurs', 'Fri', 'Sat'] AS daysofweek),

  taxitrips AS (
  SELECT
    (tolls_amount + fare_amount) AS total_fare,
    daysofweek[ORDINAL(EXTRACT(DAYOFWEEK FROM pickup_datetime))] AS dayofweek,
    EXTRACT(HOUR FROM pickup_datetime) AS hourofday,
    SQRT(POW((pickup_longitude - dropoff_longitude),2) + POW(( pickup_latitude - dropoff_latitude), 2)) as dist, #乗車と降車の間のユークリッド距離
    SQRT(POW((pickup_longitude - dropoff_longitude),2)) as longitude, #乗車と降車の間のユークリッド距離（経度）
    SQRT(POW((pickup_latitude - dropoff_latitude), 2)) as latitude, #乗車と降車の間のユークリッド距離（緯度）
    passenger_count AS passengers
  FROM
    `nyc-tlc.yellow.trips`, daynames, params
WHERE trip_distance > 0 AND fare_amount BETWEEN 6 and 200
    AND pickup_longitude > -75 #タクシーの移動距離の制限
    AND pickup_longitude < -73
    AND dropoff_longitude > -75
    AND dropoff_longitude < -73
    AND pickup_latitude > 40
    AND pickup_latitude < 42
    AND dropoff_latitude > 40
    AND dropoff_latitude < 42
    AND MOD(ABS(FARM_FINGERPRINT(CAST(pickup_datetime AS STRING))),1000) = params.EVAL
  )

  SELECT *
  FROM taxitrips

  ))
```

<p>次のような出力が返されます。</p>
<p><img alt="再トレーニングされたモデル（出力）" src="https://cdn.qwiklabs.com/8oZTc75ZsZIOhOU%2FE24sEIC9zcC6XS0caHnC6DGJ5%2BY%3D"></p>
<p>RMSE は、+-$5.12 に減少しました。これは最初のモデルの +-$9.47 よりも大幅に改善しています。</p>
<p>RSME により予測エラーの標準偏差が定義されるために、再トレーニングされた線形回帰によってモデルの精度が大幅に向上したことがわかります。</p>
<h2 id="step12">理解度を確認する</h2>
<p>今回のラボで学習した内容の理解を深めるため、以下の選択問題を用意しました。正解を目指して頑張ってください。</p>

<h2 id="step13">探索できるその他のデータセット</h2>
<p>シカゴのタクシー運賃を予測する場合など、他のデータセットに対してモデルを作成するには、以下のリンクを使用して <strong>bigquery-public-data</strong> プロジェクトを開始します。</p>

# Dialogflow と BigQuery ML でヘルプデスク チャットボットを実装する

<h2 id="step2">概要</h2>
<p>テクニカル サポートで問題が解決されるまでにかかる時間を正確に予測できたら便利だと思いませんか。このラボでは、BigQuery Machine Learning を使用して、ヘルプデスクのレスポンス時間を予測するシンプルな機械学習モデルをトレーニングします。その後、Dialogflow を使用してシンプルなチャットボットを構築し、そのヘルプデスク チャットボットにトレーニング済みの BigQuery ML モデルを統合します。最終的なソリューションでは、リクエストが行われた時点での推定レスポンス時間をユーザーに返します。</p>
<p>ここでは、一般的なクラウド開発プロセスの手順に沿って以下の順序で演習を進めます。</p>
<ol>
<li>
<p>BigQuery Machine Learning を使用してモデルをトレーニングする</p>
</li>
<li>
<p>シンプルな Dialogflow アプリケーションをデプロイする</p>
</li>
<li>
<p>Dialogflow 内のインライン コードエディタを使用して、BigQuery を統合する Node.js フルフィルメント スクリプトをデプロイする</p>
</li>
<li>
<p>チャットボットをテストする</p>
</li>
</ol>
<h3>ラボの内容</h3>
<ul>
<li>
<p>BigQuery ML を使用して機械学習モデルをトレーニングする方法</p>
</li>
<li>
<p>BigQuery ML を使用して機械学習モデルを評価、改善する方法</p>
</li>
<li>
<p>Dialogflow エージェントにインテントとエンティティをインポートする方法</p>
</li>
<li>
<p>カスタム Node.js フルフィルメント スクリプトを実装する方法</p>
</li>
<li>
<p>BigQuery を Dialogflow に統合する方法</p>
</li>
</ul>
<h3>要件</h3>
<ul>
<li>
<p>Dialogflow の基本コンセプトと構成。<a href="https://dialogflow.com/docs/tutorial-build-an-agent">ここをクリック</a>すると、基本的な会話設計と Webhook を使ったフルフィルメントについて説明する Dialogflow の入門チュートリアルをご覧いただけます。</p>
</li>
<li>
<p>SQL と Node.js（またはその他のコーディング言語）に関する基礎知識。</p>
</li>
</ul>
<h3>ソリューションに関するフィードバック / ラボのヘルプ</h3>
<ul>
<li>
<p>このソリューションについてのお問い合わせやこのラボに関するフィードバックは、<a href="mailto:gcct-ce-amer-specialists-ml@google.com">gcct-ce-amer-specialists-ml@google.com</a> までご連絡ください。</p>
</li>
</ul>

<h2 id="step4">BigQuery Machine Learning を使用してモデルをトレーニングする</h2>
<h3>BigQuery Console を開く</h3>
<p>Google Cloud Console で [<strong>ナビゲーション メニュー</strong>] &gt; [<strong>BigQuery</strong>] を選択します。</p>
<p><img alt="BigQuery_menu.png" src="https://cdn.qwiklabs.com/QwUX6algdXPgIThqBlgB2rnqGkoQUUANkl3xicD06uc%3D"></p>
<p>[<strong>Cloud Console の BigQuery へようこそ</strong>] メッセージボックスが開きます。このメッセージボックスには、クイックスタート ガイドとリリースノートへのリンクが表示されています。</p>
<p>[<strong>完了</strong>] をクリックします。</p>
<p>BigQuery コンソールが開きます。</p>
<p>
</p>
<p><img alt="bq-console.png" src="https://cdn.qwiklabs.com/vd3QNHGB4BAyHjAvOHw9qD0iqCPNaHAzY657z%2FGWLtY%3D"></p>

<p>左側のメニューで対象のプロジェクトをクリックし、[<strong>データセットを作成</strong>] をクリックします。[<strong>データセット ID</strong>] に「<strong>helpdesk</strong>」と入力します。</p>
<p><img alt="f5f8a74c9fa1b3de.png" src="https://cdn.qwiklabs.com/pq%2BfEvhe5Ua7qFtrTYyhGY%2BvsMslCkNrsyBX7dmyc6o%3D"></p>
<p>[データのロケーション] は [<strong>デフォルト</strong>] のままにして、[<strong>データセットを作成</strong>] をクリックします。</p>

<p>左側のメニューで、新たに作成した <strong>helpdesk</strong> データセットをクリックし、[<strong>テーブルを作成</strong>] をクリックします。</p>
<p><img alt="6ce7343f738549e9.png" src="https://cdn.qwiklabs.com/GJiWd%2BpvUVCmnipIbO%2Bdy78vaMMGcYCunpbBddslxZg%3D"></p>
<p>以下のパラメータを使用して新しいテーブルを作成します。その他のフィールドはすべてデフォルトのままにします。</p>
<table>
<tr>
<td colspan="1" rowspan="1">
<p><strong>プロパティ</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>値</strong></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p><strong>テーブルの作成元</strong></p>
</td>
<td colspan="1" rowspan="1">
<p>Google Cloud Storage</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p><strong>GCS バケットからファイルを選択</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><code>gs://solutions-public-assets/smartenup-helpdesk/ml/issues.csv</code></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p><strong>ファイル形式</strong></p>
</td>
<td colspan="1" rowspan="1">
<p>CSV</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p><strong>テーブル名</strong></p>
</td>
<td colspan="1" rowspan="1">
<p>issues</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p><strong>自動検出</strong></p>
</td>
<td colspan="1" rowspan="1">
<p>[<strong>スキーマと入力パラメータ</strong>] チェックボックスをオン</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p><strong>[詳細オプション] &gt; [スキップするヘッダー行]</strong></p>
</td>
<td colspan="1" rowspan="1">
<p>1</p>
</td>
</tr>
</table>
<p>[<strong>テーブルを作成</strong>] をクリックします。</p>
<p>新しい BigQuery テーブルにソースデータを読み込むジョブがトリガーされます。ジョブが完了するまでに 30 秒ほどかかります。左側で [<strong>ジョブ履歴</strong>] を選択するとこのジョブを確認できます。</p>

<p>クエリエディタに以下のクエリを追加します。</p>

```sql
SELECT * FROM `helpdesk.issues` LIMIT 1000
```

<p>[<strong>実行</strong>] をクリックしてデータを確認します。</p>
<p>次のクエリでは、データ フィールドの <code>category</code> と <code>resolutiontime</code> を使用して、問題が解決されるまでにかかる時間を予測する機械学習モデルを構築します。モデルタイプはシンプルな線形回帰で、トレーニング済みモデルの名前は、helpdesk データセットの <strong>predict_eta_v0</strong> です。</p>
<p>このクエリを実行します。完了までに <strong>1</strong> 分ほどかかります。</p>

```sql
CREATE OR REPLACE MODEL `helpdesk.predict_eta_v0`
OPTIONS(model_type='linear_reg') AS
SELECT
 category,
 resolutiontime as label
FROM
  `helpdesk.issues`
```

<p>次のクエリを実行して、作成した機械学習モデルを評価します。このクエリは、モデルの性能に関する指標を生成します。</p>

```sql
WITH eval_table AS (
SELECT
 category,
 resolutiontime as label
FROM
  helpdesk.issues
)
SELECT
  *
FROM
  ML.EVALUATE(MODEL helpdesk.predict_eta_v0,
    TABLE eval_table)
```

<p><img alt="6542e699b12047b9.png" src="https://cdn.qwiklabs.com/tZePoHntLXazkYkhxUGOjzfauZu0kckOpbZuQjj%2BjYI%3D"></p>
<p>この評価指標からは、このモデルの性能があまりよくないことがわかります。r2_score 指標と explained_variance 指標の値が 0 に近い場合、アルゴリズムがデータのシグナルとノイズをうまく区別できていないことになります。トレーニングに使用するフィールドに <strong>seniority</strong>、<strong>experience</strong>、<strong>type</strong><em><em></em><em></em></em> を追加して、改善がみられるかどうか調べてみましょう。この最終的なトレーニング済みモデルの名前は <strong>predict_eta</strong> です。以下のクエリを実行します。</p>

```sql
CREATE OR REPLACE MODEL `helpdesk.predict_eta`
OPTIONS(model_type='linear_reg') AS
SELECT
 seniority,
 experience,
 category,
 type,
 resolutiontime as label
FROM
  `helpdesk.issues`
```

<p>さらに次のクエリを実行して、作成した更新済みの機械学習モデルを評価します。</p>

```sql
WITH eval_table AS (
SELECT
 seniority,
 experience,
 category,
 type,
 resolutiontime as label
FROM
  helpdesk.issues
)
SELECT
  *
FROM
  ML.EVALUATE(MODEL helpdesk.predict_eta,
    TABLE eval_table)
```

<p>トレーニングに使用するフィールドを追加した結果、モデルの性能が改善しました。r2_score 指標と explained_variance 指標の値が 1 に近い場合、モデルが明確な直線関係を見出していることになります。また、*_error 指標の値が先ほどより低くなっていますが、これは、モデルの性能が向上したことを表しています。</p>
<p><img alt="3be7b028fccb0b25.png" src="https://cdn.qwiklabs.com/gNW0QlqiFz8UHs6ZZZE%2FfDcBuj8LSgRQwtlPJEWgW%2Bo%3D"></p>
<p>では、特定のシナリオの解決時間を予測するクエリを実行してみましょう。</p>

```sql
WITH pred_table AS (
SELECT
  5 as seniority,
  '3-Advanced' as experience,
  'Billing' as category,
  'Request' as type
)
SELECT
  *
FROM
  ML.PREDICT(MODEL `helpdesk.predict_eta`,
    TABLE pred_table)
```

<p><img alt="18c7e605fbc7fdc0.png" src="https://cdn.qwiklabs.com/RdCgMQ3wPJe%2BfuTcdVzeCLlTN9wf9Vov2PQRfx9Jmws%3D"></p>
<p>seniority が <strong>5</strong>、experience が <strong>3-Advanced</strong>、category が <strong>Billing</strong>、type が <strong>Request</strong> の場合の平均レスポンス時間は <strong>3.74 日</strong>という結果が返されました。</p>
<h2 id="step5">Dialogflow エージェントを作成する</h2>
<p>新しいブラウザタブを開き、Dialogflow でエージェントを作成します。</p>
<ol>
<li>
<p>この <a href="http://dialogflow.com" target="_blank">Dialogflow.com のリンク</a>をクリックし、右上にある [<strong>Go to console</strong>] をクリックします。</p>
</li>
<li>
<p>[<strong>Sign-in with Google</strong>] をクリックし、このラボにログインしたときの認証情報を使用してログインします。</p>
</li>
<li>
<p>[<strong>許可</strong>] をクリックして、Dialogflow にこのラボの Google アカウントへのアクセスを許可します。</p>
</li>
<li>
<p>[<strong>Email preferences</strong>] チェックボックスをオフ、[<strong>Yes, I have read and accept the agreement</strong>] チェックボックスをオンにし、[<strong>Accept</strong>] をクリックして利用規約に同意します。</p>
</li>
<li>
<p>[<strong>Create Agent</strong>] をクリックします。</p>
</li>
<li>
<p>エージェントの名前を入力し、言語やタイムゾーンなどのその他のプロパティを選択します。</p>
</li>
<li>
<p>[Google Project] を Qwiklabs のプロジェクト ID に設定します。</p>
<p><img alt="3886839d6cf7625a.png" src="https://cdn.qwiklabs.com/%2BbzreNsmm1UmRF5ibKVbIfMDBoDLmJMZ3F6mS4e0uME%3D"></p>
</li>
</ol>
<p>準備ができたら、[<strong>Create</strong>] をクリックします。</p>

<h2 id="step6">シンプルなヘルプデスク エージェントのインテントとエンティティをインポートする</h2>
<p>ここでは、時間を節約するために、エージェントをゼロから構成する代わりに既存のヘルプデスク エージェントのインテントとエンティティをインポートします。</p>
<h2 id="step7">IT ヘルプデスク エージェントをインポートする</h2>
<ul>
<li>設定ファイルを <a href="https://github.com/googlecodelabs/cloud-dialogflow-bqml/raw/master/ml-helpdesk-agent.zip">https://github.com/googlecodelabs/cloud-dialogflow-bqml/raw/master/ml-helpdesk-agent.zip</a> からダウンロードして、お使いのパソコンに保存します。</li>
<li>引き続き Diagflow コンソールで、左上にあるハンバーガー アイコンをクリックして左側のパネルを開きます。</li>
</ul>
<p><img alt="open-left-pane.gif" src="https://cdn.qwiklabs.com/hPz85BXUqClfC0q8pu0kN3DpJzo9pnQ%2Bm%2B4JxWFaR6Y%3D"></p>
<ul>
<li>エージェント名の横にある歯車アイコン <img alt="c999657044af2883.png" src="https://cdn.qwiklabs.com/gI0H9AEqZDJgiHrg1VIHGIWqYmjAwPixA64jt81Lc%2FM%3D"> をクリックします。</li>
</ul>
<p><img alt="d30a42054694019b.png" src="https://cdn.qwiklabs.com/9cCcjx5SASQKhrxkZPUFFKnrBVjcftDURqZOotkYHd8%3D"></p>
<ul>
<li>設定のリストで [<strong>Export and Import</strong>] タブをクリックします。</li>
<li>[<strong>Import from zip</strong>] を選択します。</li>
<li>パソコンに保存した <code>ml-helpdesk-agent.zip</code> ファイルを選択します。</li>
<li>表示されるテキスト ボックスに「<code>IMPORT</code>」と入力し、[<strong>Import</strong>] をクリックします。</li>
<li>[<strong>Done</strong>] をクリックします。</li>
</ul>
<p>インポートが完了したら、左側のパネルを使用して [<strong>Intents</strong>] に移動し、インポートされたインテントを確認します。[<strong>Submit Ticket</strong>] がメイン インテントで、[<strong>Submit Ticket - Email</strong>] と [<strong>Submit Ticket - Issue Category</strong>] というフォローアップ インテントがあります。[<strong>Submit Ticket - Email</strong>] はユーザーのメールアドレスを収集するために使用され、[<strong>Submit Ticket - Issue Category</strong>] は、ユーザーから問題の説明を収集してサポート カテゴリを自動的に推測するために使用されます。</p>
<p>左側のパネルを使用して [<strong>Entities</strong>] に移動し、[<strong>@category</strong>] エンティティを確認します。このエンティティは、ユーザーから提供されたリクエストの説明をサポート カテゴリにマッピングするために使用されます。そのサポート カテゴリを使用してレスポンス時間が予測されます。</p>

<h2 id="step8">BigQuery を統合するフルフィルメントをインライン エディタで作成する</h2>
<aside><strong>注:</strong> このコードではチケットは保存されません。装飾されたメッセージがデモのためにユーザーに返されるだけです。

</aside>
<p>左側のパネルで [<strong>Fulfillment</strong>] をクリックし、[<strong>Inline Editor</strong>] を [<strong>Enabled</strong>] に切り替えます。</p>
<p><img alt="b27336e809799f86.png" src="https://cdn.qwiklabs.com/pQoxEV4MmQAZqaDb3t%2FcIEXpMLDh0SGCgomgFWofXTE%3D"></p>
<aside><strong>注:</strong> 「Your Google Cloud resources are still being provisioned, please refresh the page and try again in a few minutes」というエラーが表示された場合は、しばらく待ってからページを再読み込みしてください。

</aside>
<p>次のコードをコピーして [<strong>index.js</strong>] タブに貼り付けて、既存のコンテンツを置き換えます。</p>

```js
/**
* Copyright 2017 Google Inc. All Rights Reserved.
*
* Licensed under the Apache License, Version 2.0 (the "License");
* you may not use this file except in compliance with the License.
* You may obtain a copy of the License at
*
*    http://www.apache.org/licenses/LICENSE-2.0
*
* Unless required by applicable law or agreed to in writing, software
* distributed under the License is distributed on an "AS IS" BASIS,
* WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
* See the License for the specific language governing permissions and
* limitations under the License.
*/

"use strict";

const functions = require("firebase-functions");
const { WebhookClient } = require("dialogflow-fulfillment");
const { Card } = require("dialogflow-fulfillment");
const BIGQUERY = require("@google-cloud/bigquery");

const BIGQUERY_CLIENT = new BIGQUERY({
projectId: "your-project-id" // ** CHANGE THIS **
});

process.env.DEBUG = "dialogflow:debug";

exports.dialogflowFirebaseFulfillment = functions.https.onRequest(
(request, response) => {
  const agent = new WebhookClient({ request, response });
  console.log(
    "Dialogflow Request headers: " + JSON.stringify(request.headers)
  );
  console.log("Dialogflow Request body: " + JSON.stringify(request.body));

  function welcome(agent) {
    agent.add(`Welcome to my agent!`);
  }

  function fallback(agent) {
    agent.add(`I didn't understand`);
    agent.add(`I'm sorry, can you try again?`);
  }

  function ticketCollection(agent) {
    // Capture Parameters from the Current Dialogflow Context
    console.log('Dialogflow Request headers: ' + JSON.stringify(request.headers));
    console.log('Dialogflow Request body: ' + JSON.stringify(request.body));
    const OUTPUT_CONTEXTS = request.body.queryResult.outputContexts;
    const EMAIL = OUTPUT_CONTEXTS[OUTPUT_CONTEXTS.length - 1].parameters["email.original"];
    const ISSUE_CATEGORY = OUTPUT_CONTEXTS[OUTPUT_CONTEXTS.length - 1].parameters.category;
    const ISSUE_TEXT = request.body.queryResult.queryText;
 

    // The SQL Query to Run
    const SQLQUERY = `WITH pred_table AS (SELECT 5 as seniority, "3-Advanced" as experience,
          @category as category, "Request" as type)
          SELECT cast(predicted_label as INT64) as predicted_label
          FROM ML.PREDICT(MODEL helpdesk.predict_eta,  TABLE pred_table)`;

    const OPTIONS = {
      query: SQLQUERY,
      // Location must match that of the dataset(s) referenced in the query.
      location: "US",
      params: {
        category: ISSUE_CATEGORY
      }
    };
    return BIGQUERY_CLIENT.query(OPTIONS)
      .then(results => {
        //Capture results from the Query
        console.log(JSON.stringify(results[0]));
        const QUERY_RESULT = results[0];
        const ETA_PREDICTION = QUERY_RESULT[0].predicted_label;
    
        //Format the Output Message
        agent.add( EMAIL + ", your ticket has been created. Someone will you contact shortly. " +
            " The estimated response time is " + ETA_PREDICTION  + " days."
        );
        agent.add(
          new Card({
            title:
              "New " + ISSUE_CATEGORY +
              " Request for " + EMAIL +
              " (Estimated Response Time: " + ETA_PREDICTION  +
              " days)",
            imageUrl:
              "https://developers.google.com/actions/images/badges/XPM_BADGING_GoogleAssistant_VER.png",
            text: "Issue description: " + ISSUE_TEXT,
            buttonText: "Go to Ticket Record",
            buttonUrl: "https://assistant.google.com/"
          })
        );
        agent.setContext({
          name: "submitticket-collectname-followup",
          lifespan: 2
        });
      })
      .catch(err => {
        console.error("ERROR:", err);
      });
  }

  // Run the proper function handler based on the matched Dialogflow intent name
  let intentMap = new Map();
  intentMap.set("Default Welcome Intent", welcome);
  intentMap.set("Default Fallback Intent", fallback);
  intentMap.set("Submit Ticket - Issue Category", ticketCollection);
  agent.handleRequest(intentMap);
}
);
```

<p>さらに、このファイルの BIGQUERY_CLIENT 変数を更新して、<code>your-project-id</code> を実際のプロジェクト ID に置き換えます。</p>
<p>このフルフィルメント スクリプトのクエリ ステートメントに <strong>ML.PREDICT</strong> 関数が含まれていることに注目してください。この関数により、レスポンス時間の予測がクライアントに返されます。チケットの説明から自動的にカテゴリが特定されて、レスポンス時間の予測のために <strong>BigQuery ML</strong> に送信されます。</p>
<p><img alt="a2a70551a65fa5ad.png" src="https://cdn.qwiklabs.com/eUeGul0WQvjFRSRVBvbVXytCNW%2BaCfJkUHd%2BL8RLMOk%3D"></p>
<p>次のコードをコピーして [<strong>package.json</strong>] タブに貼り付けて、既存のコンテンツを置き換えます。</p>

```json
{
   "name": "dialogflowFirebaseFulfillment",
   "description": "Dialogflow Fulfillment Library quick start sample",
   "version": "0.0.1",
   "private": true,
   "license": "Apache Version 2.0",
   "author": "Google Inc.",
   "engines": {
     "node": ">=6.0"
   },
   "scripts": {
     "start": "firebase serve --only functions:dialogflowFirebaseFulfillment",
     "deploy": "firebase deploy --only functions:dialogflowFirebaseFulfillment"
   },
   "dependencies": {
     "firebase-admin": "^4.2.1",
     "firebase-functions": "^0.5.7",
     "dialogflow-fulfillment": "0.3.0-beta.3",
     "@google-cloud/bigquery": "^1.3.0"
  
   }
}
```

<p>[<strong>Deploy</strong>] ボタンをクリックします。デプロイが正常に完了したというメッセージが表示されるまで待ちます。これには数分かかることがあります。</p>

<h2 id="step9">フルフィルメントの Webhook を有効にする</h2>
<p>次に、左側のパネルで [<strong>Intents</strong>] に戻ります。[<strong>Submit Ticket</strong>] の横にある下矢印をクリックして、そのフォローアップ インテントを表示します。</p>
<p>最後のフォローアップ インテントである [<strong>Submit Ticket - Issue Category</strong>] をクリックして、編集用に開きます。</p>
<p><img alt="dfbb7bb68466fb24.png" src="https://cdn.qwiklabs.com/c4Alzy3UxdR7WwH%2BFyw%2F2AyEKi%2BkhFQdWaDr0gUE2zQ%3D"></p>
<p>一番下までスクロールして、[<strong>Fulfillment</strong>] セクションで [<strong>Enable Webhook call for this intent</strong>] がオンになっていることを確認します。</p>
<p><img alt="b454a5f35626270d.png" src="https://cdn.qwiklabs.com/PZzr1ElsJNpL9fiPTihtXkpX0pigGPA1%2FlRafJsXLbg%3D"></p>
<h2 id="step10">チャットボットをテストする</h2>
<p>この時点で、Dialogflow が設定されています。</p>
<p>右側の [<strong>Try it now</strong>] パネルに次のように入力してテストします。1 行ずつ入力してください。</p>
<ol>
<li>「Hi」</li>
<li>「I would like to submit a ticket」</li>
<li>「My email is student@qwiklabs.net」</li>
<li>「I can't login」</li>
</ol>
<p><img alt="c321d93bddbc79fa.png" src="https://cdn.qwiklabs.com/EOOK3T26yGHoyzIdW2S%2Futex0FkT%2FB%2FYQ29R5175i9o%3D"></p>
<p>4 に対する出力が次のようになるはずです。</p>
<p><img alt="521ebb632e5d6bb9.png" src="https://cdn.qwiklabs.com/uNfelx%2B29XXqtjseOxi0PcIykMf7s%2FpkAIJEgeXYDgw%3D"></p>
<p>このフルフィルメント スクリプトでは、特定のプラットフォームのエンドユーザーにより充実したコンテンツを提供するために <a href="https://github.com/dialogflow/dialogflow-fulfillment-nodejs">Dialogflow フルフィルメント ライブラリ</a>も活用しています。</p>
<p><img alt="44db09e7e45487b.png" src="https://cdn.qwiklabs.com/50q1AkqtYONB9%2FBA1RQ6h4Op%2FeVcxJ0vQ0ByCLXfdTo%3D"></p>
<aside><strong>注:</strong> 上のような出力が表示されない場合は、エージェントの [Fulfillment] セクションに戻り、インライン エディタにコピーしたコードを再度デプロイしてください。また、コードで指定されている <strong>projectId</strong> を実際のプロジェクトに合わせて更新したことを確認してください。

</aside>
<h2 id="step11">BigQuery の統合について</h2>
<p>すでに見たように、この BigQuery ML モデルで予測を返すには <strong>seniority</strong> フィールドと <strong>experience</strong> フィールドも必要です。実際のアプリでは、ユーザーから提供された名前や会社の ID を使用して、ユーザーの勤続年数（seniority）と経験レベル（experience）を会社のデータベースからプログラムで取得することも可能です。この例では、勤続年数が <strong>5</strong> 年、経験レベルが <strong>3-Advanced</strong> であると仮定しています。</p>
<p><img alt="2767ede0a7bb81b0.png" src="https://cdn.qwiklabs.com/OiHkgapgUjQtbTBelgFxOb459aJVUgQD%2FhbSDon9Amw%3D"></p>
<aside class="special">
<strong>追加の実習:</strong> 勤続年数が 5 年ではなく 2 年のユーザーに対するレスポンス時間を予測するようにフルフィルメント スクリプトを変更して、ボットを再度テストしてみてください。レスポンス時間の予測は変わりましたか。
</aside>
<p>この例をさらに拡張して、<strong>seniority</strong> と <strong>experience</strong> の情報をチャットボットのユーザーから収集することもできます。そのためには、Dialogflow のスロット充填機能を使用します。Dialogflow のスロット充填機能の詳細については、<a href="https://dialogflow.com/docs/concepts/slot-filling">ここをクリック</a>してください。</p>
<h2 id="step12">ワンクリック統合を使用してテストする</h2>
<p>Dialogflow によって、チャットボットでさまざまな統合が可能になります。サンプル ウェブ ユーザー インターフェースを見てみましょう。</p>
<p>Dialogflow の左側のパネルにある [<strong>Integrations</strong>] をクリックします。</p>
<p>[<strong>Web Demo</strong>] のスイッチを切り替えて、ウェブデモ統合を有効にします。  <img alt="e5cc494e391690f3.png" src="https://cdn.qwiklabs.com/Tr48B5FA%2FlER8UljDXbVV5MtDwlpGAsLIZDgcd1JMRo%3D"></p>
<p><strong>URL</strong> リンクをクリックしてウェブデモを開始します。<img alt="73cbcb3d2d0d5bdc.png" src="https://cdn.qwiklabs.com/wA8adqwV21%2BK2sxsHUlzhW%2Fbv8I3R42pAI2Yl7IGMkY%3D"></p>
<p>[Ask something] セクションに入力して、チャット インターフェースの使用を開始します。Chrome ブラウザを使用している場合は、マイクのアイコンをクリックすると、口頭でチャットボットに質問することもできます。次のようにチャットボットとチャットしてみましょう。</p>
<ul>
<li>
<p>「Hi」と入力して Enter キーを押します。チャットボットは前と同じようにレスポンスを返します。</p>
</li>
<li>
<p>次に「Submit ticket」と入力します。</p>
</li>
<li>
<p>メールアドレスを伝えるために「My email is student@qwiklabs.net」と入力します。</p>
</li>
<li>
<p>チケットの詳細として「My printer is broken」と入力します。</p>
</li>
</ul>