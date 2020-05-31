## レコメンデーションシステムについて

物件の推奨システムは どのように動作するのでしょうか？ 最初に すべての物件の評価を 取り込む必要があります 物件の紹介時に ユーザーから受けたものです。 まず 物件データに移動して 評価を取り込む必要があります 明示的に下された評価です。 たとえば ユーザーを物件の内覧に 案内したときに 家屋の細かい部分を確認したユーザーが ４つの星をクリックした評価など あるいは評価が暗黙的に なされることもあります ユーザーがウェブサイトを訪れて 特定の物件のページを 長時間閲覧している場合などです。 

次に 機械学習モデルをトレーニングして 最新の物件データベースにある すべての物件について ユーザーごとの評価を 予測できるようにします。 その後 ユーザーがまだ見ていない物件から 高い評価が予測された上位５件を選びます 推奨エンジンの動作を知らない人は Facebookのフィードに 自動車が表示されたのを見て 自動車に関する記事を 読んだからだと考えます 「あの記事を読んでいたときは フィードに車がずっと表示されていたな」 実はそうではありません。 車に関する記事を読んだことにより すべての車の評価が上昇する一方で それ以外の項目の評価は変動せず 結果として 相対的に下落します。 そして 評価が最も高い車が 上位５件にランクインします。 これらの車は以前から 評価は付いていたのです。が 車の記事を読むまでは 相対的な評価が高くなかったということです。 

さて ユーザーが物件をどう評価するかを 予測するにはどうすればよいか？ 特に物件をまだ見ていない場合は？ この場合のモデルの基盤は２つです。 ユーザーの他の物件への評価と 対象物件への他のユーザーからの評価です。 ごく簡単なモデルでは 特定の物件を評価したユーザーに注目し その中から問題のユーザーに よく似た３人を選びます 同じ国に住んでいるとか 年齢が同じであるとか 同じ大学に通ったとか そういった条件で最も似た３人を選びます この３人のユーザー評価を平均して 問題のユーザーの評価予測とします。 もちろん これは高度なモデルではありません。 シンプルすぎて精度が低いかもしれません。 この３人が何らかの理由で 他の人が関心を 寄せないような物件に評価を付けていたとしたら？ 評価はこの３つしかありません。 最も似た3人のユーザーの評価です。 不人気の物件の評価は 簡単に操作できるのです。 とはいえこの 類似ユーザーによる 特定の物件の評価を利用するという考え方が 推奨モデルの基本的な前提になっています 

では このモデルのどこに機械学習が あるのでしょうか？ 特定のユーザーに最も近いユーザーを どう見つけるか それが このモデルの鍵になります 検討すべきユーザーの人数は？ ３人、５人、７人 何人でしょうか？ 他にも考慮すべき要素はないでしょうか？ 寄せられた評価の件数が多い物件と 少ない物件に差をつけるなど まずは 意図的に評価を控えたかどうかの推測に 役立つパラメータを探してみましょう 数千の物件があり、1件あたりのレビュー数が 2、３個の場合もあります その場合 レビューしたユーザーと 予測したいユーザーに関連性がない可能性があります このような評価行列は極端な疎行列であるため 項目をクラスタ化し ユーザーを集める必要があります 

もっと直感的な例を使って説明しましょう 友人全員が スポーツタイプの四輪駆動車、 SUVを運転していると仮定します。 あなたはポルシェについて 書かれた記事を読んでいます その場合 ポルシェのSUVモデルが フィードに表示される可能性があります たとえ読んでいた記事が ツードアのポルシェに関するもので 友人の全員がトヨタのSUVに 乗っていたとしてもです。 友人が誰一人ポルシェSUVの 所有者ではないにもかかわらず 機械学習モデルはポルシェのSUVに 高い評価を与えたということになります 機械学習モデルは基本的にこう考えます 「このユーザーと同じような人は誰？」 次にこう考えます 「これは人が高い評価を付けたくなる物件だろうか？」 評価の予測は この２つを組み合わせて行われます あらゆるものを考慮した末に 特定のユーザーがある物件に対して下す評価は そのユーザーとよく似たユーザーたちが 過去に下した評価の平均に 物件自体の品質による校正を 加えたものになります これで問題と対応方法がわかりました 

では最後の問題に取り組みましょう インフラの問題です。 評価予測のコンピューティングを どれぐらいの頻度でどこで実行するのか？ その評価のコンピューティングの完了後 結果をどこに保存するのか？ 評価予測のコンピューティングは どれぐらいの頻度でどこで実行するでしょうか どの物件を推奨するかは ユーザーが物件に評価を付けるたびに 更新すべきものではありません。 新しい評価がシステムに追加されるたびに 推奨物件を見直す必要はありません。 １日に一度更新すれば十分でしょう 週に一度でもいいぐらいです。 つまり この処理を ストリーミングにする必要はありません。 バッチで対応できます。 一方 逆のケースもあります 物件が数千あり ユーザーが数百万人いるような場合は ユーザーが物件に評価を追加するたびに 評価を算出する方がよいでしょう これをスケーラブルな方法で行います 単独のマシンで実行するのではなく 大規模なデータセットに拡張できる フォールトトレラントな手法で行います そのように実行しなくてはならない場合に 一般的なソリューションは Apache Hadoopのような ビッグデータプラットフォームです。 

最後に 算出した評価は どこに保存します。か？ そもそも保存するのはなぜでしょうか？ 推奨物件はウェブアプリに追加する情報です。が ユーザーがウェブページを見るたびに 計算処理することは避けたいです。ね そのため事前にバッチジョブで 算出しておきます １日か１週間に一度 評価を事前に算出しておいて ユーザーのログインがあったら 事前に評価を算出していた物件を推奨します。 そのため 予測された推奨物件を トランザクションで保存しておく必要があります なぜトランザクション？ ユーザーが推奨物件に目を通している間に 予測テーブルを更新できるからです。 ユーザーごとに５件の推奨物件があり ユーザーが合計100万人いるとすると テーブルの大きさは500万行になります とはいえこれは一般的に データをMySQLのような リレーショナル データベース マネジメント システム（RDBMS）に保存する場合 十分に小さくてコンパクトなテーブルであり 一般的なソリューションと言えます。

### オンプレミスからクラウドへの移行

物件推奨システムを作成するために Hadoopなどのビッグデータ プラットフォームと MySQLなどのRDBMSを 使うことに決めました どちらもオープンソースのテクノロジーです。 すでにHadoopクラスタと MySQLデータベースを オンプレミスでお使いかもしれません。 ここでは 推奨システムが オンプレミスで運用中と仮定し それをGCPに移行する方法を 説明することにします。 もちろん それによって価値が増す場合にのみ 移行を行いますが その価値についても説明します。 

物件推奨モデルについて たとえば データサイエンスチームがすでに オンプレミスのモデルを稼働させており SparkMLジョブをHadoopクラスタで 扱っているとします。 チームはGCPのスケーリングと 柔軟性に魅力を感じ 既存のSparkMLジョブを オンプレミスから同等のGCPに移行したいと考え これをパイロットプロジェクトとして 実行するつもりです。 前のレッスンで説明したとおり 評価予測の算出は 1日に一度で十分です。 リアルタイムで行う必要はありません。 Hadoopバッチ処理で十分です。 SparkMLを使いますが そのジョブをオンプレミスで実行するのではなく 機械学習ジョブをCloud Dataprocで実行し 評価をCloud SQLのRDBMSに保存します。 ユーザー１人に５件の推奨物件という 小さなデータセットだからです。 これは最初のラボで行う実習です。 余談です。が これを知ると Gmailのスマートリプライのすごさがわかります この物件推奨システムとは違って Sparkの返信に提供される推奨は 新着メールが受信トレイに届いたときに リアルタイムで作成されます 言うまでもないことです。が Gmailはここで説明したものより はるかに高度なことをしています。 

Cloud SQLを使うことにした理由を 説明しましたが 評価の保存に使用できる ビッグデータプロダクトは他にもあります アクセスパターンに基づいて ストレージを選ぶには この表が参考になります このコースでは他のシナリオでも ソリューションについて説明します。が 手短に言うと **Cloud Storage** を グローバルファイルシステムとして使い Cloud SQLを SQLを通じてアクセスする トランザクションリレーショナルデータの RDBMSとして使います さらに **Datastore** を トランザクション用の 非SQLオブジェクト指向データベースとして使い Bigtableを高スループットの 非SQL追記専用データの保存に使います トランザクションではなく追記専用データです。 Bigtableは一般に 接続されたデバイスの センサーデータなどに使います BigQueryを SQLデータウェアハウスとして使って すべての分析ニーズに対応します。 

ここで必要なのは トランザクションデータベースであり データ量は数ギガバイト以下と見込まれるため Cloud SQLを選びます 図があると理解しやすいでしょう GCPでデータがどこに保存されるのかを 可視化したものです。 画像や音声などの非構造化データには **Cloud Storage** を使います 構造化データを使用し トランザクションが必要な場合は Cloud SQLまたは Cloud Datastoreを使います どちらを使うかは アクセスパターンを SQLにするか、非SQLにするかで決まります 非SQLはKey-Valueペアを意味します。 別の言い方をすると １つの鍵でデータを検索するなら **Datastore** を使い SQLを使ってデータを検索するなら **Cloud SQL** を使います。

一般に **Cloud SQL** は数ギガバイトで 頭打ちになります 水平にスケーリングすることで 数ギガバイトを超えるデータにも対応できる トランザクションデータベースが必要な場合や データをグローバルに分散するために 複数のデータベースが必要な場合は Cloud Spannerが最適です。 １つのデータベースで十分な場合も Cloud SQLを使います データが多い、複数の大陸で トランザクションを行う必要があるといった理由で 複数のデータベースが必要な場合は Cloud Spannerを使います 構造化データを分析したい場合は BigtableまたはBigQueryの使用をご検討ください Bigtableを使うのは リアルタイムの高スループットの アプリケーションが必要な場合です。 BigQueryを使うのはペタバイト規模の データセットで分析を行う場合です。 今回の物件推奨ユースケースでは 評価と予測をどこかに保存する必要があります これらはユーザーの評価と 物件の構造化データです。 トランザクションワークロードで扱える 読み書き可能なデータであり １つのデータベースで十分な 小さなデータセットなのでCloud SQLを使います  Cloud SQLとは？ Googleがホストするリレーショナル データベースで クラウド上に存在します。 オープンソースデータベースの MySQLとPostgresなど 複数のデータベースをサポートします。 今回はMySQLを使います Cloud SQLの長所は すぐに馴染めることです。 ほとんどのMySQL文と関数をサポートし ストアドプロシージャやトリガー、 ビューも使えます Cloud SQLはクラウドの経済性を 柔軟な料金制度という形で提供します。 使用した分だけ料金を支払えば済みます GCPがお客様に代わって MySQLを管理します。 バックアップやレプリケーションなどが 含まれます クラウド上にあるのでどこからでも接続でき 静的IPアドレスの割り当ても可能 一般のSQLコネクタライブラリを使えます Googleファイアウォールの 内側にあるため高速です。 Cloud SQLインスタンスを App EngineやCompute Engineと 同じリージョンに配置すれば 広い帯域幅で利用でき また Googleセキュリティで保護されます Cloud SQLは安全なGoogle データセンターにあるので 推奨データを保存する場所は安全です。 

ところで 推奨データの計算処理の 初回はどのように実行すればよいでしょうか？ ここでビッグデータコンピューティングの歴史を 振り返ってみましょう 2006年より前は ビッグデータとは文字通り 大きなデータベースを意味していました データベースの設計は ストレージが比較的安価で 処理能力が高価な時代に始まったため データを保存場所からプロセッサに コピーして処理する方法が合理的でした 処理の後、結果がストレージに書き戻されていました 2006年頃にビッグデータの分散処理が Hadoopによって現実的なものになります Hadoopの基本的な考えは コンピュータのクラスタを作成し 分散処理を活用するというものです。 Hadoop分散ファイルシステムによって データがクラスタ内のマシンに保存され MapReduceによって このデータの分散処理が行われます Hadoop関連ソフトウェアの エコシステムがHadoopを中心に成立し （Hive、Pig、Sparkもその一部です。） 2010年頃にBigQueryがリリースされました BigQueryを皮切りにGoogleは 多数のビッグデータサービスを開発します。 2015年頃には Cloud Dataprocをリリースしました Cloud Dataprocは HadoopとSparkクラスタを作成して データ処理ワークロードを管理する マネージドサービスです。 推奨システムのもう１つの要素は Hadoop上で動くソフトウェアです。 今回は このソフトウェアが 物件の推奨を作成できるように 機械学習モデルをトレーニングします。 今回は ApacheSparkに含まれる SparkMLを使います ApacheSparkは オープンソースのソフトウェアプロジェクトで バッチデータやストリーミングデータを 高速処理する分析エンジンを提供します。 Sparkはインメモリ処理を行うため 同じ処理を実行するHadoopジョブより 最大で100倍高速です。 Sparkにはデータ処理を抽象化する 耐障害性分散データセット（RDD）と DataFramesという２つの機能もあります 物件の推奨には Sparkジョブを作成して使用します。 一方でコンピューティングは Cloud Dataprocで行います

## はじめに
このラボでは、レコメンデーション エンジンで使用される賃貸物件データを Cloud SQL に入力します。レコメンデーション エンジン自体は Dataproc で Spark ML を使用して実行されます。

### Cloud SQL インスタンスを作成する
GCP Console で、[ストレージ] にある [SQL] をクリックします。

[インスタンスを作成] をクリックします。

[MySQL] を選択して、必要な場合は [次へ] をクリックします。

[インスタンス ID] に「rentals」と入力します。

下にスクロールして root パスワードを指定します。忘れないように、このパスワードをメモしておきます。

[作成] をクリックしてインスタンスを作成します。約 1 分で Cloud SQL インスタンスがプロビジョニングされます。

テーブルを作成する
インスタンスが作成されるまでの間に、以下の mySQL スクリプトを読み、疑問があれば解決しておきます。

```sql
CREATE DATABASE IF NOT EXISTS recommendation_spark;

USE recommendation_spark;

DROP TABLE IF EXISTS Recommendation;
DROP TABLE IF EXISTS Rating;
DROP TABLE IF EXISTS Accommodation;

CREATE TABLE IF NOT EXISTS Accommodation
(
  id varchar(255),
  title varchar(255),
  location varchar(255),
  price int,
  rooms int,
  rating float,
  type varchar(255),
  PRIMARY KEY (ID)
);

CREATE TABLE  IF NOT EXISTS Rating
(
  userId varchar(255),
  accoId varchar(255),
  rating int,
  PRIMARY KEY(accoId, userId),
  FOREIGN KEY (accoId)
    REFERENCES Accommodation(id)
);

CREATE TABLE  IF NOT EXISTS Recommendation
(
  userId varchar(255),
  accoId varchar(255),
  prediction float,
  PRIMARY KEY(userId, accoId),
  FOREIGN KEY (accoId)
    REFERENCES Accommodation(id)
);

SHOW DATABASES;
```

このスクリプトはいくつのテーブルを作成します。か。

✅　3

ユーザーが家を評価すると（例えば、4 つ星を付ける）、エントリが_______テーブルに追加されます。

✅　Rating

部屋の数や平均評価など、家に関する一般的な情報は_________テーブルに保存されます。

✅　Accommodation

レコメンデーション エンジンの仕事は、各ユーザーと家の___________テーブルに入力することです。これは、そのユーザーによるその家の予測評価です

✅　Recommendation

Cloud SQL で rentals をクリックし、インスタンス情報を表示します。

データベースに接続する
このページの [このインスタンスに接続] ボックスで、[Cloud Shell を使用して接続] をクリックします。
注: 専用の Cloud Compute Engine VM からインスタンスに接続することもできます。が、今回は Cloud Shell によってマイクロ VM が作成され、そこから操作します。。

Cloud Shell が読み込まれるのを待ちます。

Cloud Shell が読み込まれると、以下のコマンドが入力されています。

```bash
gcloud sql connect rentals --user=root --quiet
```

Enter キーを押します。

IP アドレスがホワイトリストに登録されるのを待ちます。

Whitelisting your IP for incoming connection for 5 minutes...⠹

プロンプトが表示されたら、パスワードを入力して Enter キーを押します。（注: 入力したパスワードも **** などのマスクされた文字も表示されません。）。
これでデータベースに対してコマンドを実行できるようになりました。 mysql-connection.png

次のコマンドを実行します。

```sql
show databases;
```

デフォルトのシステム データベースが表示されます。

```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

注: mySQL コマンドの末尾には必ずセミコロン「;」を付けてください。

先ほど分析した以下の SQL ステートメントをコピーしてコマンドラインに貼り付けます。

```sql
CREATE DATABASE IF NOT EXISTS recommendation_spark;

USE recommendation_spark;

DROP TABLE IF EXISTS Recommendation;
DROP TABLE IF EXISTS Rating;
DROP TABLE IF EXISTS Accommodation;

CREATE TABLE IF NOT EXISTS Accommodation
(
  id varchar(255),
  title varchar(255),
  location varchar(255),
  price int,
  rooms int,
  rating float,
  type varchar(255),
  PRIMARY KEY (ID)
);

CREATE TABLE  IF NOT EXISTS Rating
(
  userId varchar(255),
  accoId varchar(255),
  rating int,
  PRIMARY KEY(accoId, userId),
  FOREIGN KEY (accoId)
    REFERENCES Accommodation(id)
);

CREATE TABLE  IF NOT EXISTS Recommendation
(
  userId varchar(255),
  accoId varchar(255),
  prediction float,
  PRIMARY KEY(userId, accoId),
  FOREIGN KEY (accoId)
    REFERENCES Accommodation(id)
);

SHOW DATABASES;
```

Enter キーを押します。

データベースとして recommendation_spark が表示されていることを確認してください。

```
+----------------------+
| Database             |
+----------------------+
| information_schema   |
| mysql                |
| performance_schema   |
| recommendation_spark |
| sys                  |
+----------------------+
```

次のコマンドを実行して、テーブルを表示します。
USE recommendation_spark;

SHOW TABLES;
Enter キーを押します。

テーブルが 3 つあることを確認してください。

```
+--------------------------------+
| Tables_in_recommendation_spark |
+--------------------------------+
| Accommodation                  |
| Rating                         |
| Recommendation                 |
+--------------------------------+
```

次のクエリを実行します。

```sql
SELECT * FROM Accommodation;
```

宿泊施設の表には何行ありますか。  
✅ Empty セット (0)

### Google Cloud Storage のステージデータ
#### オプション 1: コマンドラインを使用する
新しい Cloud Shell タブを開きます（既存の mySQL Cloud Shell タブを使用しないでください）。

次のコマンドを貼り付けます。

```bash
echo "Creating bucket: gs://$DEVSHELL_PROJECT_ID"
gsutil mb gs://$DEVSHELL_PROJECT_ID

echo "Copying data to our storage from public dataset"
gsutil cp gs://cloud-training/bdml/v2.0/data/accommodation.csv gs://$DEVSHELL_PROJECT_ID
gsutil cp gs://cloud-training/bdml/v2.0/data/rating.csv gs://$DEVSHELL_PROJECT_ID

echo "Show the files in our bucket"
gsutil ls gs://$DEVSHELL_PROJECT_ID

echo "View some sample data"
gsutil cat gs://$DEVSHELL_PROJECT_ID/accommodation.csv
```

Enter キーを押します。

#### オプション 2: コンソール UI を使用する

**コマンドラインを使用してデータをすでに読み込んでいる場合は、以下の手順をスキップしてください。**

[ストレージ] に移動して [Storage] > [ブラウザ] の順に選択します。

[バケットを作成] をクリックします。（バケットがまだ存在しない場合）。

プロジェクト名をバケット名として指定します。

[作成] をクリックします。

次のファイルをローカルにダウンロードしてから、新しいバケット内にアップロードします。

- accommodation.csv
- rating.csv

Google Cloud Storage から Cloud SQL テーブルにデータを読み込む
[SQL] に戻ります。

rentals をクリックします。

宿泊施設のデータをインポートする
[インポート]（トップメニュー）をクリックします。

以下を指定します。

- Cloud Storage ファイル: ブラウズして accommodation.csv を選択する
- インポート形式: CSV
- データベース: プルダウンから recommendation_spark を選択する
- テーブル: コピーして貼り付ける: Accommodation

[インポート] をクリックします。

import-acc-dataset

概要ページにリダイレクトされます。データが読み込まれるまで 1 分ほど待ちます。

ユーザーの評価データをインポートする
[インポート]（トップメニュー）をクリックします。

以下を指定します。

- Cloud Storage ファイル: ブラウズして rating.csv を選択する
- インポート形式: CSV
- データベース: プルダウンから recommendation_spark を選択する
- テーブル: コピーして貼り付ける: Rating

[インポート] をクリックします。

概要ページにリダイレクトされます。データが読み込まれるまで 1 分ほど待ちます。

### Cloud SQL データを探索する
mySQL への Cloud Shell 接続を閉じた場合は、[このインスタンスに接続] で [Cloud Shell を使用して接続] をクリックし、もう一度接続します。

ログインするよう求めるメッセージが表示されたら Enter キーを押します。

パスワードを入力して Enter キーを押します。

評価データに対して以下のクエリを実行します。

```sql
USE recommendation_spark;

SELECT * FROM Rating
LIMIT 15;
SQL 集計関数を使用してテーブル内の行数をカウントします。
SELECT COUNT(*) AS num_ratings
FROM Rating;
```

宿泊施設のレビューの平均値を求めます。

```sql
SELECT
    COUNT(userId) AS num_ratings,
    COUNT(DISTINCT userId) AS distinct_user_ratings,
    MIN(rating) AS worst_rating,
    MAX(rating) AS best_rating,
    AVG(rating) AS avg_rating
FROM Rating;
```

Distinct_user_ratings の 25 はどういう意味ですか。  

👉評価を提供した 25 人の独自ユーザーがいます、の意味。

機械学習では、モデルによる学習用にユーザーが設定した豊富な履歴が必要になります。以下のクエリを実行して、最も多くの評価を行っているユーザーを確認します。

```sql
SELECT
    userId,
    COUNT(rating) AS num_ratings
FROM Rating
GROUP BY userId
ORDER BY num_ratings DESC;
```

「exit」と入力すると mysql プロンプトを終了できます。
Cloud Dataproc を使用して機械学習による住宅のレコメンデーションを生成する
このラボでは、Dataproc を使用して、レコメンデーション（おすすめ）を行う機械学習モデルを構築します。

### 学習内容

このラボでは、次の作業を行います。

- Dataproc を起動する
- Dataproc を使用して SparkML ジョブを実行する

### はじめに
このラボでは、Dataproc を使用して、ユーザーの過去の評価に基づいてレコメンデーションを行う機械学習（ML）モデルのトレーニングを行います。その後トレーニングしたモデルを使用して、データベースに存在するすべてのユーザーに対するレコメンデーションのリストを作成します。

このラボでは、次の作業を行います。

#### Dataproc を起動する

PySpark で作成した ML モデルのトレーニングと適用を行い、商品のおすすめを作成する

Cloud SQL で加えられた行について詳しく確認する

#### Dataproc を起動する

Dataproc を起動して、クラスタ内の各マシンが Cloud SQL にアクセスできるように構成する手順は次の通りです。

GCP Console のナビゲーション メニュー（ナビゲーション メニュー）で [SQL] をクリックし、Cloud SQL インスタンスのリージョンをメモします。


上の画面では、リージョンが us-central1 になっています。

GCP Console のナビゲーション メニュー（ナビゲーション メニュー）で [Dataproc] をクリックします。プロンプトが表示されたら、[API を有効にする] をクリックします。

有効にしたら [クラスタを作成] をクリックして、クラスタに「rentals」と名前を付けます。

[リージョン] に「global」を選択し、[ゾーン] を（Cloud SQL インスタンスと同じゾーンの）「us-central1-a」に変更します。これでクラスタとデータベース間のネットワーク レイテンシを最小限に抑えることができます。

[マスターノード] の [マシンタイプ] には vCPU x 2（n1-standard-2） を選択します。

[ワーカーノード] の [マシンタイプ] には vCPU x 2（n1-standard-2） を選択します。

他の値はすべてデフォルトのままにし、[作成] をクリックします。1～2 分でクラスタのプロビジョニングが完了します。

クラスタの名前、ゾーン、ワーカーノード総数をメモします。

以下の bash スクリプトをコピーして Cloud Shell に貼り付けてください（必要に応じて実行前に CLUSTER、ZONE、NWORKERS を変更してください）。

```bash
echo "Authorizing Cloud Dataproc to connect with Cloud SQL"
CLUSTER=rentals
CLOUDSQL=rentals
ZONE=us-central1-a
NWORKERS=2

machines="$CLUSTER-m"
for w in `seq 0 $(($NWORKERS - 1))`; do
   machines="$machines $CLUSTER-w-$w"
done

echo "Machines to authorize: $machines in $ZONE ... finding their IP addresses"
ips=""
for machine in $machines; do
    IP_ADDRESS=$(gcloud compute instances describe $machine --zone=$ZONE --format='value(networkInterfaces.accessConfigs[].natIP)' | sed "s/\['//g" | sed "s/'\]//g" )/32
    echo "IP address of $machine is $IP_ADDRESS"
    if [ -z  $ips ]; then
       ips=$IP_ADDRESS
    else
       ips="$ips,$IP_ADDRESS"
    fi
done

echo "Authorizing [$ips] to access cloudsql=$CLOUDSQL"
gcloud sql instances patch $CLOUDSQL --authorized-networks $ips
```

Enter キーを押して、プロンプトが表示されたら「Y」と入力し、もう一度 Enter キーを押して続行します。

パッチの適用が完了するまで待ちます。完了したら以下のように表示されます。

Patching Cloud SQL instance...done.

最後に Cloud SQL のメインページの [このインスタンスに接続] の下に表示されているパブリック IP アドレスをクリップボードにコピーするか書き留めます。これは後で使用します。

#### ML モデルを実行する
トレーニングを行ったモデルを作成し、システム内のすべてのユーザーに適用します。

データ サイエンス チームにより、Apache Spark を使って Python で作成されたレコメンデーション モデルが作成されています。そのモデルをステージング バケットにコピーしましょう。

Cloud Shell で以下のコマンドを実行してモデルコードをコピーします。
gsutil cp gs://cloud-training/bdml/v2.0/model/train_and_apply.py train_and_apply.py
cloudshell edit train_and_apply.py
プロンプトが表示されたら、[エディタで開く] を選択します。

エディタ UI が読み込まれるまで待ちます。

train_and_apply.py

```python
#!/usr/bin/env python
"""
Copyright Google Inc. 2016
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
"""


import os
import sys
import pickle
import itertools
from math import sqrt
from operator import add
from os.path import join, isfile, dirname
from pyspark import SparkContext, SparkConf, SQLContext
from pyspark.mllib.recommendation import ALS, MatrixFactorizationModel, Rating
from pyspark.sql.types import StructType, StructField, StringType, FloatType

# MAKE EDITS HERE
CLOUDSQL_INSTANCE_IP = '35.224.50.128'   # <---- CHANGE (database server IP)
CLOUDSQL_DB_NAME = 'recommendation_spark' # <--- leave as-is
CLOUDSQL_USER = 'root'  # <--- leave as-is
CLOUDSQL_PWD  = 'root'  # <---- CHANGE

# DO NOT MAKE EDITS BELOW
conf = SparkConf().setAppName("train_model")
sc = SparkContext(conf=conf)
sqlContext = SQLContext(sc)

jdbcDriver = 'com.mysql.jdbc.Driver'
jdbcUrl    = 'jdbc:mysql://%s:3306/%s?user=%s&password=%s' % (CLOUDSQL_INSTANCE_IP, CLOUDSQL_DB_NAME, CLOUDSQL_USER, CLOUDSQL_PWD)

# checkpointing helps prevent stack overflow errors
sc.setCheckpointDir('checkpoint/')

# Read the ratings and accommodations data from Cloud SQL
dfRates = sqlContext.read.format('jdbc').options(driver=jdbcDriver, url=jdbcUrl, dbtable='Rating', useSSL='false').load()
dfAccos = sqlContext.read.format('jdbc').options(driver=jdbcDriver, url=jdbcUrl, dbtable='Accommodation', useSSL='false').load()
print("read ...")

# train the model
model = ALS.train(dfRates.rdd, 20, 20) # you could tune these numbers, but these are reasonable choices
print("trained ...")

# use this model to predict what the user would rate accommodations that she has not rated
allPredictions = None
for USER_ID in range(0, 100):
  dfUserRatings = dfRates.filter(dfRates.userId == USER_ID).rdd.map(lambda r: r.accoId).collect()
  rddPotential  = dfAccos.rdd.filter(lambda x: x[0] not in dfUserRatings)
  pairsPotential = rddPotential.map(lambda x: (USER_ID, x[0]))
  predictions = model.predictAll(pairsPotential).map(lambda p: (str(p[0]), str(p[1]), float(p[2])))
  predictions = predictions.takeOrdered(5, key=lambda x: -x[2]) # top 5
  print("predicted for user={0}".format(USER_ID))
  if (allPredictions == None):
    allPredictions = predictions
  else:
    allPredictions.extend(predictions)

# write them
schema = StructType([StructField("userId", StringType(), True), StructField("accoId", StringType(), True), StructField("prediction", FloatType(), True)])
dfToSave = sqlContext.createDataFrame(allPredictions, schema)
dfToSave.write.jdbc(url=jdbcUrl, table='Recommendation', mode='overwrite')
```

train_and_apply.py で、30 行目の CLOUDSQL_INSTANCE_IP を見つけ、先ほどコピーした Cloud SQL IP アドレスを貼り付けます。

```py
# MAKE EDITS HERE
CLOUDSQL_INSTANCE_IP = '<paste-your-cloud-sql-ip-here>'   # <---- 変更（データベース サーバー IP）
CLOUDSQL_DB_NAME = 'recommendation_spark' # <--- そのままにする
CLOUDSQL_USER = 'root'  # <--- そのままにする
CLOUDSQL_PWD  = '<type-your-cloud-sql-password-here>'  # <---- 変更
```

33 行目の CLOUDSQL_PWD に Cloud SQL のパスワードを入力します。

エディタには自動保存機能がありますが、確実に保存するために [ファイル] > [保存] を選択してください。

Cloud Shell リボンから [ターミナルを開く] をクリックし、次の Cloud Shell コマンドを使用してこのファイルを Cloud Storage バケットにコピーします。

gsutil cp train_and_apply.py gs://$DEVSHELL_PROJECT_ID
Dataproc で ML ジョブを実行する
Dataproc コンソールで [ジョブ] をクリックします。

8508ce301ff584c3.png

[ジョブを送信] をクリックします。

[ジョブタイプ] に [PySpark] を選択し、[メインの Python ファイル] では、バケットにアップロードした Python ファイルのロケーションを指定します。上部ナビゲーション メニューのプロジェクト ID のプルダウンをクリックして検索できる場合、<bucket-name> がプロジェクト ID の可能性があります。

gs://<bucket-name>/train_and_apply.py

[送信] をクリックします。

注: ジョブが [実行中] から [完了] に変わるまでに最大で 5 分かかります。ジョブの実行中に、結果に対してクエリを実行する次のセクションに進むことができます。。

ジョブが [失敗] した場合は、ログを使用してトラブルシューティングし、エラーを修正してください。変更した Python ファイルを Cloud Storage に再度アップロードし、失敗したジョブのクローンを作成して再度送信する必要があることもあります。

SQL で加えられた行について詳しく確認する
新しいブラウザタブで、[ストレージ] にある [SQL] を開きます。

rentals をクリックし、Cloud SQL インスタンスに関連する詳細情報を表示します。

[このインスタンスに接続] セクションで [Cloud Shell を使用して接続] をクリックすると、新しい Cloudshell タブが開きます。このタブで Enter キーを押します。

受信接続用の IP がホワイトリストに登録されるまでには、数分かかることがあります。

プロンプトが表示されたら、構成した root パスワードを入力して Enter キーを押します。

mysql プロンプトで、次のコマンドを入力します。

USE recommendation_spark;

SELECT COUNT(*) AS count FROM Recommendation;
Empty Set (0) が表示された場合は Dataproc ジョブが完了するまで待ちます。5 分以上経過した場合はジョブが失敗した可能性があるため、トラブルシューティングが必要になります。

ヒント: 前のコマンド（この場合はクエリ）に戻るには Cloud Shell で上矢印を使用します。

モデルはいくつの推奨を提供しましたか。

100
check
125

50

ユーザー向けのレコメンデーションを探します。

```sql
SELECT
    r.userid,
    r.accoid,
    r.prediction,
    a.title,
    a.location,
    a.price,
    a.rooms,
    a.rating,
    a.type
FROM Recommendation as r
JOIN Accommodation as a
ON r.accoid = a.id
WHERE r.userid = 10;
```

以下の結果を確認してください。

```
+--------+--------+------------+-----------------------------+...
| userid | accoid | prediction | title                       |...
+--------+--------+------------+-----------------------------+...
| 10     | 40     |  1.9717555 | Colossal Private Castle     |...
| 10     | 46     |  1.7060381 | Colossal Private Castle     |...
| 10     | 74     |  1.4713808 | Giant Calm Fort             |...
| 10     | 77     |  1.4085547 | Great Private Country House |...
| 10     | 43     |  1.3759944 | Nice Private Hut            |...
+--------+--------+------------+-----------------------------+...
```

このユーザーにおすすめの宿泊施設は上記の 5 つです。今回はデータセットが非常に小さいため、レコメンデーションの質があまり良くありません。（予測評価があまり高くありません。）が、商品のレコメンデーションを作成するプロセスは以上で学習できました。

お疲れさまでした
内容のまとめ:

- 賃貸物件用のフルマネージド Cloud SQL インスタンスを作成しました
- テーブルを作成し、SQL を使ってスキーマを調べました
- CSV からデータを取り込みました
- Cloud Dataproc で Spark ML ジョブを編集、実行しました
- 予測結果を表示しました

## リソース

</p><ul><li><a href="https://cloud.google.com/solutions/migration/hadoop/hadoop-gcp-migration-overview?hl=ja" target="_blank" rel="noopener nofollow"><u>Hadoop から Google Cloud Platform への移行</u></a></li></ul><ul><li><a href="https://cloud.google.com/sql/?hl=ja" target="_blank" rel="noopener nofollow"><u>Cloud SQL のドキュメント</u></a> + <a href="https://cloud.google.com/blog/products/databases/" target="_blank" rel="noopener nofollow" title="https://cloud.google.com/blog/products/databases/" aria-label="https://cloud.google.com/blog/products/databases/">リリースブログ</a></li></ul><ul><li><a href="https://cloud.google.com/dataproc/?hl=ja" target="_blank" rel="noopener nofollow"><u>Cloud Dataproc のドキュメント</u></a> +  <a href="https://cloud.google.com/blog/products/dataproc" target="_blank" rel="noopener nofollow"><u>リリースブログ</u></a></li></ul>