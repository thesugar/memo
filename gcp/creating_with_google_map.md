# Creating with Google Maps
In this quest you will use several tools available in Google Cloud to manipulate data and create a Google Map - map location details to find subway stations or a business; use geocoding and Apps Script to send an email of a map; visualize data on a customized map; and build a server-side proxy to create a map on a mobile device.

> このクエストでは Google Cloud で利用可能なツールを使ってデータを操作し Google Map を作成します。具体的には、地下鉄の駅やビジネス上の目的地を見つけるための詳細な地図情報を作成したり、ジオコーディングと Google Apps Script を使ってその地図を E メールにして送信したり、カスタムされた地図上でデータを可視化したり、モバイル端末上で地図を作成するためのサーバサイドのプロキシを構築したりします。

# Google Apps Script: コード 4 行で Google スプレッドシート、Google マップ、Gmail にアクセスする

<h2 id="step2">概要</h2>
<p>このラボでは、ウェブ開発言語の中でも主流の JavaScript でコードを記述して、Google のデベロッパー テクノロジーを簡単に利用する方法について説明します。まず Google Apps Script でコードを記述して、Google スプレッドシート内のセルに保存された住所を抽出し、その住所をもとに Google マップを生成して、その地図へのリンクを Gmail を使って自分や友人宛てに送信します。このアプリの最大のメリットは、わずか ４ 行のコードでこの一連の処理を実行できるところです。</p>
<h2 id="step3">目標</h2>
<ul>
<li>
<p>Apps Script について簡単に学ぶ（当初の知識としては十分な程度）</p>
</li>
<li>
<p>Google スプレッドシートで新しいスプレッドシートを作成する</p>
</li>
<li>
<p>任意のドキュメントでスクリプト エディタを開く</p>
</li>
<li>
<p>Apps Script コードを編集、保存、実行する</p>
</li>
<li>
<p>結果を Gmail で確認する</p>
</li>
</ul>

<h3>G Suite にログインする</h3>
<p>左上のパネルの [<strong>Google Console を開く</strong>] をクリックして、プロンプトが表示されたらユーザー名とパスワードを入力します。</p>
<p><img alt="sign-in.gif" src="https://cdn.qwiklabs.com/VlqoIx0APvIV%2FFKJeweX4URn0rTGCqfXnVfRbwjMYvg%3D"></p>
<ul>
<li>販売パートナーを通じて購入した G Suite に関する契約に<strong>同意</strong>します。</li>
<li>G Suite 無料試用に関する追加の<strong>利用規約に同意します</strong>。</li>
</ul>
<p><img alt="terms-of-service.png" src="https://cdn.qwiklabs.com/AETxipS8KJz0h4LPOx15tg0GcNo4Ynb%2Fw%2BFK0pLA5WM%3D"></p>
<p><strong>管理コンソール</strong>が開きます。</p>
<p><img alt="管理コンソール" src="https://cdn.qwiklabs.com/5LUcVH9C4VVi%2B6mOo5a%2Bg23W7nE579fTjmP0EDgwiJo%3D"></p>
<ql-warningbox>
「現在、リクエストを処理できません。しばらくしてからもう一度お試しください。」というサーバーエラーが表示された場合、管理コンソールはまだ起動中です。ブラウザ ウィンドウを閉じて、[<strong>Google Console を開く</strong>] を再度クリックします。</ql-warningbox>
<p></p>
<ql-infobox>
右上の <strong>Google アプリ</strong>のアイコンをクリックすると、このページからお好きな G Suite アプリケーションに移動できます。<strong>Google アプリ</strong>のアイコンが表示されるようにブラウザ ウィンドウを広げてください。</ql-infobox>
<p><img alt="google-apps-icon.png" src="https://cdn.qwiklabs.com/gudW3U6LSA6btnq8cyCZGUaHhiFCwNxjUQyhK5gxnrw%3D"></p>

<h2 id="step5">Google Apps Script とは</h2>
<p><a href="http://developers.google.com/apps-script">Google Apps Script</a> が提供する開発環境は、これまでに使い慣れたものではないかもしれませんが、次のことが可能になります。</p>
<ul>
<li>ブラウザベースのコードエディタでの開発に加え、Apps Script 用のコマンドライン デプロイツールである <a href="http://developers.google.com/apps-script/guides/clasp">clasp</a> を使用すれば、ローカルでの開発も可能です。</li>
<li>G Suite やその他の Google または Google 外部のサービス（URLfetch、JDBC など）にアクセスできるようカスタマイズされた特殊な JavaScript でコーディングを行うことができます。</li>
<li>認証コードの記述を行う必要はありません。Apps Script で自動的に処理されます。</li>
<li>アプリをホストする必要はありません。クラウドの Google サーバーから実行されます。</li>
</ul>
<p><strong>注</strong>: Apps Script の詳細は、このラボでは扱いません。さまざまなオンライン リソースがありますので、そちらでご確認ください。公式ドキュメントには、<a href="http://developers.google.com/apps-script/overview">概要とクイックスタート</a>、<a href="http://developers.google.com/apps-script/articles/tutorials">チュートリアル</a>、<a href="https://developers.google.com/apps-script/guides/videos">動画</a>が用意されています。このラボでは、皆さんがコードの作成をスムーズに行い、その環境でどのようなアプリケーションを作成するかを検討できるよう、Apps Script の開発環境について説明していきます。</p>
<p>Apps Script アプリケーションには、次の 2 つがあります。</p>
<ol>
<li>
<a href="http://developers.google.com/apps-script/guides/bound">コンテナ バインド</a> — 1 つの Google ドキュメント（ドキュメント、スプレッドシート、スライド、サイト、フォーム）のみに永続的に関連付けられたものです。</li>
<li>
<a href="https://developers.google.com/apps-script/guides/standalone">スタンドアロン</a> — 特定の G Suite ドキュメントに関連付けられない独立したスクリプトです。</li>
</ol>
<p>コンテナ バインドのアプリもスタンドアロンのアプリも、公開することで幅広く利用できるようになります。</p>
<ul>
<li>非公開 — プロジェクトの所有者だけがアクセスできます</li>
<li>
<a href="https://developers.google.com/apps-script/add-ons/publish">アドオンとして公開</a> — アプリがアドオンストアで公開され、誰でもインストールできます</li>
<li>
<a href="https://developers.google.com/apps-script/execution_web_apps">ウェブアプリとして公開</a> — ウェブ UI コンポーネントを備え、HTTP リクエストが処理される形式のアプリが公開されます</li>
<li>
<a href="https://developers.google.com/apps-script/guides/web#embedding_your_web_app_in_google_sites">Google サイトに埋め込む</a> — 公開されたウェブアプリを、新しいサイトまたは従来のサイトのページに埋め込むことができます</li>
<li>
<a href="http://developers.google.com/apps-script/guides/rest/api">Execution API 対応形式で公開</a> — Execution API からアクセスできる形式のアプリが公開されます</li>
<li>上記の組み合わせ（有効な場合）</li>
</ul>
<p>ここでは、Google スプレッドシートの<strong>コンテナ バインド型</strong> Apps Script アプリを作成していきます。まずは新しいスプレッドシートを作成しましょう。</p>
<h2 id="step6">G Suite 管理 コンソールを設定する</h2>
<ql-warningbox>これらの設定手順中に「We are unable to process your request at this time, please try again later」というサーバーエラーが発生した場合、管理コンソールはまだ起動しています。ブラウザ ウィンドウを閉じ、もう一度 [Google コンソールを開く] をクリックします。</ql-warningbox>
<ol>
<li>
<p>管理コンソールで [<strong>Click here</strong>] ボタンをクリックして、管理コンソールの設定を完了します。</p>
<img alt="click-here.png" src="https://cdn.qwiklabs.com/pd261qmDaTpi9sA5UteqGB4Z6tnmqmrCBr6Bz%2FJZEnE%3D">
</li>
</ol>
<ol start="2">
<li>
<p>[<strong>Next</strong>] をクリックします。このプロンプトが表示されたら、[<strong>No, it's just me </strong>] を選択して [<strong>Next</strong>] をクリックします。</p>
<img alt="wizard3.png" src="https://cdn.qwiklabs.com/IJ%2FMiMKrReYKWRvnxnQA9EGn%2B4K%2F21Z%2F1eyFWIesRRk%3D">
</li>
</ol>
<ol start="3">
<li><p>[Setup your apps] ページにアクセスしたら、[<strong>Next</strong>] をクリックします。次に、[<strong>Next</strong>] &gt; [<strong>Next</strong>] &gt; [<strong>Next</strong>] &gt; [<strong>Do this later</strong>] &gt; [<strong>Next</strong>] の順にクリックします。</p></li>
<li><p>[Google Calendar] ページにアクセスしたら、[<strong>Do this later</strong>] をクリックします。</p></li>
<li>
<p>[Mobile management] ステップに到達したら、「<strong>I want to keep my current configuration</strong>」を選択して [<strong>Next</strong>] をクリックします。</p>
<img alt="wizard.png" src="https://cdn.qwiklabs.com/KdqV%2FhaWbnpMpgcSeUFeiJediloY0wwFMMlDuOmK9j8%3D">
</li>
</ol>
<ol start="6">
<li><p>[<strong>Next</strong>] をクリックします。「Extend and customize your Apps」 ページが表示されたら、[<strong>Next</strong>] をクリックします（表示されない場合もあります）。</p></li>
<li><p>[Recommended for G Suite] ページが表示されたら、[<strong>Next</strong>] をクリックします。</p></li>
<li><p>[Use the Chrome browser] ページが表示されたら、[<strong>Next</strong>] をクリックします。</p></li>
<li><p>[Training and support] ページが表示されたら、[<strong>Do this later</strong>] をクリックします。</p></li>
<li>
<p>これで G Suite 管理ページに戻るはずです。</p>
<img alt="console-ui.png" src="https://cdn.qwiklabs.com/A1rkGjFUltYM4Q%2F8X%2FSkNTXXpypGQ4%2Bg%2B%2BYL5oxmrS4%3D">
</li>
</ol>
<h2 id="step7">新しい Google スプレッドシートを作成して住所を入力する</h2>
<p>次の手順を実行して、新しい Google スプレッドシートに住所を入力します。</p>
<ol>
<li>このリンクから<a href="http://sheets.google.com/create">新しい Google スプレッドシートを作成</a>します。</li>
<li>空白のスプレッドシートの、左上の最初のセル（A1）をクリックします。これが A 列の 1 行目になります。このセルに住所を入力します。郵便番号や都道府県/州、市区町村など、国内/海外のいずれかの地点の有効な住所を入力します。こちらはニューヨーク市の住所を入力した場合の例です。
<img alt="5f8bd97d9aec9778.png" src="https://cdn.qwiklabs.com/GV%2BI9lwkJX%2By32WVk5irwoR0iE9Is2XxAFyogkvMEaU%3D">
</li>
</ol>
<p>このシートで必要な作業はこれだけです。それでは、エディタを開いてコードを記述していきます。</p>
<h2 id="step8">Apps Script のコードを編集する</h2>
<p>Google スプレッドシートを作成したら、コンテナ バインドのスクリプトを編集します。</p>
<ol>
<li>上部のメニューバーで [<strong>ツール</strong>] を選択し、[<strong>スクリプト エディタ</strong>] をクリックしてスクリプト エディタを開きます。</li>
</ol>
<p><img alt="db26cb0d3a66452a.png" src="https://cdn.qwiklabs.com/cNJ5Y%2BTn2Qnmm2LlQo3rMHWgUPpaHdyI0ykZvIIaxps%3D"></p>
<ol start="2">
<li>ブラウザに、コンテナ バインド スクリプト用のコード エディタが表示されます。</li>
</ol>
<p><img alt="9225a55eeb832d54.png" src="https://cdn.qwiklabs.com/BmHmLedm6rKuYI7eR4HMQ6F7xU%2BAd2kVqmn6pl0hIO4%3D"></p>
<p><code>myFunction()</code> というデフォルトの関数が自動的に作成されて、エディタに表示されます。これで、アプリケーションのコードを記述する準備ができました。</p>
<h2 id="step9">（テンプレート）コードを編集する</h2>
<ol>
<li>
<p>表示されているテンプレートのコードは空で、何かを実行することはありません。以下のコードをコピーして、エディタ ウィンドウのテンプレート コードと置き換えます。<code>&lt;YOUR_EMAIL&gt;</code> はご自身のメールアドレスに変更してください。</p>
</li>
</ol>

```js
function sendMap() {
    var sheet = SpreadsheetApp.getActiveSheet();
    var address = sheet.getRange("A1").getValue();
    var map = Maps.newStaticMap().addMarker(address);
    GmailApp.sendEmail("<YOUR_EMAIL>", "Map", 'See below.', {attachments:[map]});
}
```

<ol start="2">
<li>
<p>このアプリを制限して（ユーザーのすべてのシートではなく）対象のシートのみにアクセスするように、次のアノテーションをファイルレベルのコメントとして追加しておきます。これでユーザーが安心して使用できるようになります。</p>
</li>
</ol>
<pre><code>/**&#x000A;* @OnlyCurrentDoc&#x000A;*/&#x000A;</code></pre>
<p><code>sendMap()</code> の最大のメリットは、この<a href="https://developers.google.com/apps-script/guides/services/authorization#manual_authorization_scopes_for_sheets_docs_and_forms">オプションのアノテーション</a>を除けば、4 行でアプリ全体が構成されているところです。</p>
<p>当然ながら、仮のメールアドレス <code>（</code>*<code>friend@example.com</code>*<code>）</code> を、ラボ中に利用可能なアドレスに置き換える必要があります。エディタのコードを置き換えると、ファイル名の左に赤いアスタリスクが表示されます。</p>
<p><img alt="7608884d754bb144.png" src="https://cdn.qwiklabs.com/VLLHtyfuZsOgvx53vP9xxkYNDq3dqNJKiEx%2Bydp0uoY%3D"></p>
<p>これは、ファイルに編集が加えられたため、保存が必要であることを示しています。編集して保存されていない内容があると必ず表示されます。</p>
<ol start="3">
<li>保存して、プロジェクトに名前を付けます（「Hello Maps!」など、名前は何でもかまいません）。ファイルを保存するには、メニューバーの [<strong>ファイル</strong>] &gt; [<strong>保存</strong>] を選択します。</li>
</ol>
<p><img alt="20ff3503c76bc1c7.png" src="https://cdn.qwiklabs.com/Ors27y1WEOLUSMGYK14WDdGxJLJ3OsGsKCgUmjC0JGY%3D"></p>
<p>小さいディスクのアイコン <img alt="c167b6792ab6f136.png" src="https://cdn.qwiklabs.com/OBR%2FXYGaMeHsVJKqP7F42xVzYrcojqvV7jnLYMbae2c%3D"> をクリックするか、Ctrl+S キー（PCs, Linux）または Command+S キー（Mac）をクリックすることでも保存できます。まだプロジェクトに名前を付けていない場合は、次に進む前に名前を付けてください。</p>
<h2 id="step10">Google スプレッドシート、マップ、Gmail アプリを実行する</h2>
<ol>
<li>アプリを実行してみましょう。いくつかの方法があります。関数の名前は <code>sendMap()</code> に変更したので、[<strong>実行</strong>] &gt; [<strong>関数を実行</strong>] &gt; [<strong>sendMap</strong>] の順に選択します。</li>
</ol>
<p><img alt="dab876859f4662b5.png" src="https://cdn.qwiklabs.com/ItIeY5J8SQJ4pZQWJVT2uv33ZF%2FxfYutNcxJJyv3rec%3D"></p>
<p>三角形の実行アイコン <img alt="174f9137757af868.png" src="https://cdn.qwiklabs.com/cnTgkKExc%2FwKVnIQLI3a1a5pGlP0DYFUCgW67umhpV0%3D"> をクリックし、実行される関数が <code>sendMap()</code> であることを確認することもできます。</p>
<ol start="2">
<li>デベロッパーにとって便利な Apps Script の機能の一つに、認証コードを記述する必要がないことがあります。Apps Script は自動的に認証を管理しますが、このスクリプトが（ユーザーに代わって）スプレッドシートにアクセスし、Gmail でメールを送信できるよう、ユーザー自身がアクセスを許可する必要があります。最初の認証ダイアログは、次のようになります。</li>
</ol>
<p><img alt="7dc7ef0d4188a808.png" src="https://cdn.qwiklabs.com/7lFVEp3rh7x%2B4JYyG5jh8Eqe5FwYDAcg%2FosHVYKT8WQ%3D"></p>
<ol start="3">
<li>[<strong>許可を確認</strong>] をクリックします。</li>
<li>メッセージが表示されたら、アカウントを選択します（<strong>ユーザー名</strong>はラボの [<strong>接続の詳細</strong>] セクションで確認できます）。</li>
</ol>
<p><img alt="8aa63b55cc9e5ee0.png" src="https://cdn.qwiklabs.com/jsmb9iXwHnrEKiB1%2BhpOEopqXpk%2FQ6zRYPqNS%2BXjtx8%3D"></p>
<ol start="5">
<li>これで、本当の<em></em> OAuth2 ダイアログ ウィンドウが表示され、スプレッドシートへのアクセスとメールの送信のためのアクセスを許可するかどうかが確認されます。</li>
</ol>
<p><img alt="134f0d05f1d739a2.png" src="https://cdn.qwiklabs.com/ij6811wllArzyd%2B%2BED8JONGaIz4rBcbGm%2BBLPMcvNeU%3D"></p>
<ol start="6">
<li>
<p>アクセスを許可すると、スクリプトが最後まで実行されます。</p>
</li>
<li>
<p>実行を確認するには、[<strong>表示</strong>] &gt; [<strong>実行数</strong>] を選択して、<code>sendMap</code> の [ステータス] が「<code>完了</code>」になっていることを確認します。プロンプトされたら、[<strong>View Dashboard</strong>] をクリックします。</p>
</li>
</ol>
<p><img alt="8d9e8665dee3f5ef.png" src="https://cdn.qwiklabs.com/HX5WAZOcbu%2FRZsIyMMWOJ2UvYU6hC4qWFHxQVx4N63U%3D"></p>
<ol start="8">
<li>メッセージを送信したメール アカウントを確認します。件名が「Map」で本文が次のようなメッセージが届きます。</li>
</ol>
<p><img alt="fc76285462b62185.png" src="https://cdn.qwiklabs.com/uxFur3XbqfWnVHoVqH8fTtB3wyG%2B%2F0G3APBL0DmAZRI%3D"></p>
<p>いかがですか？たった 4 行のコードで、Google の 3 つのサービスにアクセスして意味のある処理を実行できました。これだけではアプリケーションとして完成されていませんが、すばらしい結果です。JavaScript や Apps Script に慣れていなくても、コードの意味や大まかな仕組み、Apps Script で何ができるかを理解していただけたら幸いです。</p>
<h2 id="step11">アプリケーションの詳しい説明</h2>
<p>ここでは、コードを詳しく確認していきます。</p>
<p>このアプリケーションのコードは短いため、全体的な構造について特に説明することはありません。特に Google の 3 種類のサービスを利用する箇所を、1 行ずつ詳しく確認していきます。</p>
<ol>
<li>
<p>次の行は、通常の JavaScript の関数宣言で、<code>sendMap()</code> という関数を定義しています。</p>
</li>
</ol>
<pre><code class="language-bash prettyprint">function sendMap() {&#x000A;</code></pre>
<ol start="2">
<li>
<p>コードの 1 行目では、<a href="https://developers.google.com/apps-script/reference/spreadsheet/">スプレッドシート サービス</a>を呼び出しています。<a href="https://developers.google.com/apps-script/reference/spreadsheet/spreadsheet-app">SpreadsheetApp オブジェクト</a>を使用することにより、Apps Script からサービスにアクセスできます。返されたシートを、sheet という名前の変数に代入しています。<a href="https://developers.google.com/apps-script/reference/spreadsheet/spreadsheet-app#getActiveSheet()">getActiveSheet()</a> メソッドでは名前のとおりの処理が実行されます。ユーザー インターフェース（UI）で現在アクティブなシートへの「ハンドル」が返されます。</p>
</li>
</ol>

```js
var sheet = SpreadsheetApp.getActiveSheet();
```

<ol start="3">
<li>
<p>取得した <code>sheet</code> オブジェクトを使用して、<a href="https://developers.google.com/apps-script/reference/spreadsheet/sheet#getRange(String)">getRange()</a> で <a href="https://developers.google.com/sheets/api/guides/concepts#a1_notation">A1 表記</a>のセル範囲（ここでは 1 つのセル）を参照しています。「セル範囲」とはセルのグループを指しますが、今回のように 1 つのセルの場合もあります。セル <code>A1</code> は、住所を入力したセルです。<a href="https://developers.google.com/apps-script/reference/spreadsheet/range#getValue()">getValue()</a> でセル範囲の内容<em></em>を取得し、戻り値を address 変数に代入しています。その他の住所を追加して、他のセルからも読み出してみましょう。</p>
</li>
</ol>

```js
var address = sheet.getRange("A1").getValue();
```

<ol start="4">
<li>
<p>3 行目では、<a href="https://developers.google.com/apps-script/reference/maps/maps">Maps オブジェクト</a>を使って Google <a href="https://developers.google.com/apps-script/reference/maps/">マップサービス</a>にアクセスしています。マップサービスへのアクセスが成功したら、<a href="https://developers.google.com/apps-script/reference/maps/maps#newStaticMap()">newStaticMap()</a> で新しい静的マップの作成がリクエストされます。また <a href="https://developers.google.com/apps-script/reference/maps/static-map#addMarker(String)">addMarker()</a> メソッドで、シートから取得した住所に「ピン」がドロップされます。</p>
</li>
</ol>

```js
var map = Maps.newStaticMap().addMarker(address);
```

<ol start="5">
<li>
<p>最後の行では、<a href="https://developers.google.com/apps-script/reference/mail/">メールサービス</a>を使用しています。<a href="https://developers.google.com/apps-script/reference/gmail/gmail-app">GmailApp オブジェクト</a>の <a href="https://developers.google.com/apps-script/reference/mail/mail-app#sendEmail(Object)">sendEmail()</a> メソッドを呼び出してメールを送信しています。メールには「See below.」というテキストと地図画像の添付ファイルが含まれます。</p>
</li>
</ol>

```js
GmailApp.sendEmail("friend@example.com", "Map", 'See below.', {attachments:[map]});
}
```

<h2 id="step12">お疲れさまでした</h2>
<p>ここでは、Google Apps Script を使用してコードを記述しました。Google のデベロッパー テクノロジーを利用して Google スプレッドシート内の住所を抽出し、その住所をもとに Google マップを生成し、その地図をメールで送信するコードです。</p>

<p>このセルフペース ラボは、Qwiklabs の「<a href="https://google.qwiklabs.com/quests/51?locale=ja">G Suite: 統合</a>」クエストの一部です。クエストとは学習パスを構成する一連のラボのことで、完了すると成果が認められて上のようなバッジが贈られます。バッジは公開して、オンライン レジュメやソーシャル メディア アカウントにリンクさせることができます。このラボの修了後、<a href="https://google.qwiklabs.com/quests/103?locale=ja/enroll">次のクエストに登録</a>すれば、すぐにクレジットを受け取ることができます。<a href="https://google.qwiklabs.com/catalog?locale=ja">受講可能なその他の Qwiklabs のクエストもご確認ください</a>。</p>
<h3>次のラボを受講する</h3>
<p>「<a href="https://google.qwiklabs.com/focuses/2200?locale=ja&amp;parent=catalog">G Suite API にアクセス可能なアプリの作成</a>」に進んでクエストを続けるか、その他のおすすめのラボをご確認ください。</p>
<ul>
<li>
<p><a href="https://google.qwiklabs.com/focuses/4823?locale=ja&amp;parent=catalog">Google Maps Platform で周辺検索サービスを作成する</a></p>
</li>
<li>
<p><a href="https://google.qwiklabs.com/focuses/1784?locale=ja&amp;parent=catalog">Apps Script CLI - clasp</a></p>
</li>
</ul>
<h3>次のステップと詳細情報</h3>
<ul>
<li>
<p>Google Sheets API の<a href="https://developers.google.com/sheets/">デベロッパー向けドキュメント</a>を読む。</p>
</li>

</ul>
<h4>補足資料</h4>
<p>このラボで紹介しているコードは、GitHub リポジトリ（<a href="http://github.com/googlecodelabs/apps-script-intro">github.com/googlecodelabs/apps-script-intro</a>）でも入手できます（このラボの内容は、リポジトリのコードと同じ内容になるようにしています）。以下に、このラボの内容をさらに掘り下げたり、Google デベロッパー ツールにプログラムでアクセスしたりするために役立つ資料を示します。</p>
<h4>ドキュメント</h4>
<ul>
<li>
<p>Google <a href="http://developers.google.com/apps-script">Apps Script</a> ドキュメント サイト</p>
</li>
<li>
<p>Apps Script <a href="https://developers.google.com/apps-script/reference/gmail/">Gmail サービス</a></p>
</li>
<li>
<p>Apps Script <a href="https://developers.google.com/apps-script/guides/sheets">スプレッドシート サービス</a></p>
</li>
<li>
<p>Apps Script <a href="https://developers.google.com/apps-script/reference/maps/">マップ サービス</a></p>
</li>
</ul>
<h4>関連動画と一般動画</h4>
<ul>
<li>
<p>Google Apps Script <a href="https://developers.google.com/apps-script/guides/videos">動画ライブラリ</a>のその他の動画</p>
</li>
<li>
<p>The G Suite Developer Show の<a href="http://goo.gl/JpBQ40">動画シリーズ</a></p>
</li>
</ul>
<h4>ニュースと更新</h4>
<ul>
<li>G Suite <a href="http://gsuite-developers.googleblog.com">デベロッパー ブログ</a>
</li>
<li>G Suite デベロッパー <a href="https://twitter.com/GSuiteDevs">Twitter</a>（@GSuiteDevs）</li>
<li>G Suite <a href="https://developers.google.com/google-apps/newsletters">デベロッパー マンスリー ニュースレター</a>
</li>
</ul>

# Use Google Maps API to Visualize BigQuery Geospatial Data

<h2 id="step2">Overview</h2>
<p>In this lab you will use the <a href="https://cloud.google.com/maps-platform/">Google Maps Platform</a> to visualize taxi drop off locations in 2016 from journeys that started from the block around the Empire State building.  This data is stored in a Google BigQuery public dataset.</p>
<p>You will slowly build up the necessary code until you can draw an area (either a rectangle or a circle) that has the heatmap of the drop off data displayed.  The location data is fetched from BigQuery using a different method based on the type of query. Each query will use the BigQuery GIS functions.</p>
<p><img alt="Main Image" src="https://cdn.qwiklabs.com/7S4MzDweab1RXn%2BdxpcsNveusw%2F65jokS%2BT5NI%2FlPEY%3D"></p>
<ul>
<li>
<b>Rectangle queries:</b> specify the rectangle using ST_MAKEPOLYGON with ST_GEOPOINT to make the rectangle and then test if the point is within using ST_CONTAINS.</li>
<li>
<b>Radius queries:</b> specify if the point is within the radius using BigQuery GIS functions ST_DISTANCE and ST_GEOPOINT.</li>
</ul>
<h3>Standard SQL vs. Legacy SQL</h3>
<p>BigQuery supports two versions of SQL: Legacy SQL and Standard SQL. The latter is the 2011 ANSI standard. Standard SQL is used in this lab because it has better standards compliance and it's required for the GIS functions.</p>
<h3>Rectangle Queries</h3>
<p>Rectangle queries are straightforward to construct in BigQuery. You create a polygon using the points of the rectangle and then use ST_CONTAINS to decide if you include it in the matches.</p>
<h3>Radius Queries</h3>
<p>Radius queries are easy to construct, since you know the circle center and the radius. You use ST_DISTANCE to find the difference of the pickup point and the circle center, and check that it's inside the radius to decide if it should be included in the matches.</p>
<h3>Summary</h3>
<p>You have now reviewed two types of spatial query using BigQuery. As you have seen, location makes a big difference to the result data for the queries against this dataset, but unless you guess where to run your queries, it is hard to discover spatial patterns ad-hoc using just SQL queries.</p>
<p>If only you could visualise the data on a map, and explore the data by defining arbitrary areas of interest! Using the Google Maps APIs you can do just that. You will enable the Maps API, set up a simple web page, and start using the BigQuery API to send queries from your web page.</p>
<h3>Lab Objectives</h3>
<table>

<tr>
<th></th>
<th>Learning Objective</th>
</tr>


<tr>
<td>1</td>
<td>How to query petabyte-scale location datasets in seconds with BigQuery, using SQL queries, User Defined Functions and the BigQuery API</td>
</tr>
<tr>
<td>2</td>
<td>How to use the Google Maps Platform to add a Google Map to a web page and enable users to draw shapes on it</td>
</tr>
<tr>
<td>3</td>
<td>How to visualize queries against large datasets on a Google Map like in the example image below, which shows the density of taxi drop off locations in 2016 from journeys that started from the block around the Empire State Building</td>
</tr>

</table>
<h2 id="step3">Setup</h2>

<p>You need to setup and configure the Apache Web Server so you can view your page, and you need a work area so you can edit the files using the Cloud Shell editor.</p>
<p>Click the  <strong>Open in new window</strong> button in Cloud Shell.</p>
<p><img alt="open-in-new-window.png" src="https://cdn.qwiklabs.com/yzsfRePr4HK2V8wlCvHj9HVccsENK0YwZ3yJKxZ6Zoo%3D"></p>
<p>In the terminal type <code>edit .</code></p>
<p>The edit command will open the Cloud Shell editor. You will use this editor for this lab.</p>
<h3>Create your work area</h3>
<ol start="5">
<li>
<p>In Cloud Shell, make a directory for your website.</p>
</li>
</ol>
<pre><code>mkdir ~/website&#x000A;</code></pre>
<ol start="6">
<li>
<p>Run the following to .ake a default web page:</p>
</li>
</ol>
<pre><code>echo "&lt;b&gt;Hello World&lt;/b&gt;" &gt; ~/website/index.html&#x000A;</code></pre>
<ol start="7">
<li>
<p>Now create a simple web server:</p>
</li>
</ol>
<pre><code>cd ~/website&#x000A;python -m SimpleHTTPServer 8080&#x000A;</code></pre>
<p>You are using the Python SimpleHTTPSever to serve an HTML page.</p>
<h3>Check the site is viewable</h3>
<ol start="8">
<li>Click the preview button in the Cloud Shell ribbon and select <strong>Preview on port 8080</strong>.</li>
</ol>
<p><img alt="preview8080.png" src="https://cdn.qwiklabs.com/qxHcHwmQz31xTMvZKI2EGNJritL2bYbrI2cGnG2DguM%3D"></p>
<p>You should see <strong>Hello World</strong> displayed in the new tab.</p>
<h2 id="step4">Initial page (with map)</h2>
<p>With your web server up and running you can start the software development.  You will create an index.html file in your webiste directory, then create an API key and add it to your page so you can view it.</p>
<p>Start with a simple HTML page that loads a Google Map using the Maps JavaScript API and a few lines of Javascript. The code from the Google Maps Platform's <a href="https://goo.gle/gmp-simple-map">Simple Map Sample</a> is a great place to start. It's reproduced here for you to use in this lab.</p>
<ol>
<li>
<p>In Code Editor, from the <strong>website</strong> directory, open the <strong>index.html</strong> file.</p>
</li>
<li>
<p>Replace the existing text with the HTML code below:</p>
</li>
</ol>

```html
<!DOCTYPE html>
<html>
<!--
Copyright 2016 Google Inc.
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at
    http://www.apache.org/licenses/LICENSE-2.0
Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->
  <head>
    <title>Simple Map</title>
    <meta name="viewport" content="initial-scale=1.0">
    <meta charset="utf-8">
    <style>
      html, body {
        height: 100%;
        margin: 0;
        padding: 0;
      }
      #map {
        height: 100%;
      }
    </style>
    <script>

    // general vars
    let map;


    // ClientID var


    // BigQuery vars


    // initMap
    function initMap() {
        map = new google.maps.Map(document.getElementById('map'), {
          center: {lat: -43.525650, lng: 172.639847}, //where is this?
        zoom: 13
        });
        // call setupDrawingTools

    }


    // setUpDrawingTools


    // authorize


    // handleAuthResult


    // loadApi


    // sendQuery


    // checkJobStatus


    // getQueryResults


    // doHeatMap


    // rectangleQuery


    // rectangleSQL


    // circleQuery


    // circleSQL



</script>
  </head>
  <body>
    <div id="map"></div>
    <!-- Javascript Libraries -->
    <script src="https://maps.googleapis.com/maps/api/js?key=API_KEY&callback=initMap" async defer></script>

    <!-- BigQuery API -->


    <!-- Authorize Code -->


  </body>
</html>
```

<ol start="3">
<li>
<p>Click <strong>File</strong> &gt; <strong>Save</strong>.</p>
</li>
</ol>
<h3>Enable the Maps Javscript API</h3>
<p>Before you can use the Google Maps API you need to enable the service in your lab-provided account.</p>
<ol start="4">
<li>
<p>Go to the the GCP Console tab, and from the <strong>Navigation menu</strong>, select <strong>APIs &amp; Services</strong> &gt; <strong>Library</strong>.</p>
</li>
<li>
<p>Search for "maps javascript api".</p>
</li>
<li>
<p>Click <strong>Maps Javascript API</strong>.</p>
</li>
<li>
<p>Click <strong>ENABLE</strong>.</p>
</li>
</ol>
<p>Wait for the API to enable.</p>
<h3>Generate an API key</h3>
<p>To view the web page you need to generate an API key, which will be used in the index.html file.</p>
<ol start="8">
<li>
<p>In the Google Cloud Console, from the <strong>Navigation menu</strong>, select <strong>APIs &amp; Services</strong> &gt; <strong>Credentials</strong>.</p>
</li>
<li>
<p>Click <strong>Create credentials</strong> &gt; <strong>API key</strong>.</p>
</li>
<li>
<p>Copy the API key to the clipboard.</p>
</li>
<li>
<p>Click <strong>CLOSE</strong>.</p>
</li>
</ol>
<p>Click <em>Check my progress</em> to verify the objective.
<ql-activity-tracking step="1">
Enable the Maps Javascript API and generate an API key
</ql-activity-tracking></p>
<h3>Add the API key to index.html</h3>
<p>Take care not to leave any extra spaces when editing the URL.</p>
<ol start="12">
<li>
<p>In the Code Editor edit line 94 of the index.html file and replace API_KEY with your newly created API key.</p>
</li>
<li>
<p>Click <strong>File</strong> &gt; <strong>Save</strong>.</p>
</li>
</ol>
<h3>Test the page</h3>
<p>Time to check your work.</p>
<ol start="14">
<li>Go to the web preview page and refresh the page.  A map of Christchurch, New Zealand should appear.</li>
</ol>
<ql-infobox>
You can troubleshoot any issues by opening the developer console.
</ql-infobox>
<h3>Set the map to default to New York City</h3>
<ol start="15">
<li>
<p>Switch to the Code Editor and change line 44 to the following:</p>
</li>
</ol>

```js
center: {lat: 40.744593, lng: -73.990370}, // Manhattan, New York.
```

<ol start="16">
<li>Change line 45 from <code>zoom: 13</code> to <code>zoom: 15</code>.</li>
</ol>
<p>The <code>zoom: 15</code> setting starts you close to the streets when the map loads.</p>
<ol start="17">
<li>Switch to the web preview and refresh the page.</li>
</ol>
<p>The map will display Manhattan, New York.</p>
<h2 id="step5">Enable the features to draw and visualise the data</h2>
<p>You will next make edits to the index.html file to enable the ability to draw the areas and visualize the heat map onto the map.</p>
<h3>Load the drawing and visualization libraries</h3>
<p>To add drawing capabilities to your map, change the script that loads the Maps JavaScript API by adding an optional parameter that tells Google Maps Platform to enable the drawing library.</p>
<p>This lab also uses the <code>HeatmapLayer</code>, so you will also update the script to request the visualization library. To do this, you will add the libraries parameter, and specify the visualization and drawing libraries as comma-separated values, e.g. libraries=visualization,drawing</p>
<ol>
<li>
<p>Edit index.html and change line 94 to the line below.  Make sure you add back your API_KEY:</p>
</li>
</ol>

```js
<script src='https://maps.googleapis.com/maps/api/js?libraries=visualization,drawing&key=API_KEY&callback=initMap' async defer></script>
```

<h3>Add the DrawingManager</h3>
<p>To use user-drawn shapes as input to a query, add the <code>DrawingManager</code> to your map, with the Circle and Rectangle tools enabled.</p>
<ol start="2">
<li>
<p>Add this code to the <code>// general vars</code> after <code>let map;</code>:</p>
</li>
</ol>

```js
let drawingManager;
```

<ol start="3">
<li>
<p>It is a good idea to put all of the <code>drawingManager</code> set up code into a new function. Add the following code under <code>//setUpDrawingTools</code>:</p>
</li>
</ol>

```js
    function setUpDrawingTools(){
      // Initialize drawing manager.
      drawingManager = new google.maps.drawing.DrawingManager({
        drawingMode: google.maps.drawing.OverlayType.CIRCLE,
        drawingControl: true,
        drawingControlOptions: {
          position: google.maps.ControlPosition.TOP_LEFT,
          drawingModes: [
            google.maps.drawing.OverlayType.CIRCLE,
            google.maps.drawing.OverlayType.RECTANGLE
          ]
        },
        circleOptions: {
          fillOpacity: 0
        },
        rectangleOptions: {
          fillOpacity: 0
        }
      });
      drawingManager.setMap(map);
      // each one of these handlers do nothing currently, we will replace them later
      // with working code

      // rectangle search handler
      drawingManager.addListener('rectanglecomplete', rectangle => {});

      // circle search handler
      drawingManager.addListener('circlecomplete', circle => {});

    }
```

<p>This code sets up a palette of draw options on the map.  You cannot see anything on the page yet - you need to set up the initialization to the map initialization.</p>
<ol start="4">
<li>
<p>Add a call to <code>setUpDrawingTools()</code> in initMap() under <code>// call setupDrawingTools</code>.</p>
</li>
</ol>
<pre><code class="language-javascript prettyprint">        setUpDrawingTools();&#x000A;</code></pre>
<ol start="5">
<li>Save the file and reload the review page.</li>
</ol>
<p>Now you have a new tool panel with the three types of shape you can draw.  You can use the draw tools, but you need to set up event handlers to perform actions.</p>
<p><img alt="Drawing Tool Palette" src="https://cdn.qwiklabs.com/Bnh81tEijSzUzRHynw9eUWJ1ETqBtUy4SE%2Bd6wb3IzE%3D"></p>
<h2 id="step6">Using the BigQuery Client API</h2>
<p>The Google BigQuery Client API will help you avoid writing lots of boilerplate code needed to build the requests, parse responses, and handle authentication. This lab uses the BigQuery API via Google APIs Client Library for JavaScript since you will be developing a browser-based application.</p>
<h3>Add the Google Client API for JavaScript</h3>
<ol>
<li>
<p>Add the code below to the line after the line <code>&lt;!-- BigQuery API --&gt;</code> (this should be line 127):</p>
</li>
</ol>
<pre><code class="language-html prettyprint">    &lt;script src='https://apis.google.com/js/client.js'&gt;&lt;/script&gt;&#x000A;</code></pre>
<ol start="2">
<li>
<p>Save the file.</p>
</li>
</ol>
<h3>Get the preview site domain name</h3>
<ol start="3">
<li>Switch to the web preview window and copy the domain name to the clipboard.  You will refer to this value as the <strong>PREVIEW_DOMAIN_NAME</strong>.</li>
</ol>
<p><img alt="domain name" src="https://cdn.qwiklabs.com/Kd3%2BplYCUuoUJOjE9Vlqo7e6f0fdGIOeVN%2Boe081lZM%3D"></p>
<p>You will use <strong>PREVIEW_DOMAIN_NAME</strong> shortly.</p>
<h3>Create OAuth 2.0 Credentials</h3>
<p>After loading the Google Client API, authorize the user to access the data in BigQuery with OAuth 2.0. First, you need to set up some credentials in your Google Cloud Console Project.</p>
<ol start="4">
<li>In the Google Cloud Console, from the Navigation menu, select <strong>APIs &amp; Services &gt; Credentials</strong>.</li>
</ol>
<p>Before you can set up your credentials, you need to add some configuration for the Authorization screen that an end user of your application will see when they authorize your app to access BigQuery data on their behalf.</p>
<ol start="5">
<li>
<p>Click <strong>Create credentials</strong> and in the dropdown menu, click <strong>OAuth Client ID</strong>.</p>
</li>
<li>
<p>Click <strong>Configure consent screen</strong> (right side of the screen).</p>
</li>
</ol>
<p>Add the Big Query API to the scopes for this token.</p>
（internal, external の選択が出るはず。external だと審査が必要らしい。このチュートリアルだと internal で問題なく進行したが実際のアプリだと external にしないといけなかったりする？）

<img src="./images/external.png" >

<ol start="7">
<li>
<p>In the <strong>Application name</strong> field type in "bigquery-maps".</p>
</li>
<li>
<p>Click the <strong>Add Scope</strong> button in the Scopes for Google APIs section.</p>
</li>
<li>
<p>From the list, check the box next to the Big Query API entry with the <code>../auth/bigquery</code> scope.</p>
</li>
<li>
<p>Click <strong>Add</strong>.</p>
</li>
<li>
<p>In Authorized domains, paste <strong>PREVIEW_DOMAIN_NAME</strong> and press enter.</p>
</li>
</ol>
<p>Remove the <code>https//</code> and the final <code>/</code> from the URL.</p>
<ol start="12">
<li>
<p>Click <strong>Save</strong>.</p>
</li>
<li>
<p>From the Navigation menu navigate to <strong>APIs &amp; Services &gt; Credentials</strong> and select <strong>Web application</strong>.</p>
（Credential > 「+ CREATE CREDENTIALS > OAUTH client ID > Application types の選択画面が出るので、そこで Web Application を選択」
</li>
<li>
<p>In the <strong>Name</strong> field enter "bigquery-maps".</p>
</li>
<li>
<p>Under Restrictions, in the Authorized JavaScript Origins field, enter the URL of https://PREVIEW_DOMAIN_NAME</p>
</li>
</ol>
<p>Replace <strong>PREVIEW_DOMAIN_NAME</strong> with the actual value you copied to your clipboard in step 3.</p>
<ol start="16">
<li>Press <strong>Enter</strong>.</li>
</ol>
<p>You must press the Enter key to add the domain name to the list before you save.</p>
<ol start="17">
<li>Click <strong>Create</strong>.</li>
</ol>
<p>A pop up shows you the client ID and client secret. You need the client ID to perform authentication against BigQuery. Leave the popup showing, you come back to this tab shortly.</p>
<ol start="18">
<li>
<p>Edit index.html and add the following line to after the line <code>// ClientID var</code> :</p>
</li>
</ol>
<pre><code class="language-javascript prettyprint">    let clientId = 'CLIENT_ID';&#x000A;</code></pre>
<p>Replace <strong>CLIENT_ID</strong> with the client ID value displayed in the popup page.</p>
<p><strong>Save</strong> the file.</p>
<p>Click <em>Check my progress</em> to verify the objective.
<ql-activity-tracking step="2">
Create OAuth 2.0 Credentials
</ql-activity-tracking></p>
<h2 id="step7">Authorization and Initialization of the BigQuery access</h2>
<p>Your web page will need to authorize the user to access BigQuery before initializing the map. In this example you use OAuth 2.0 as described in the authorization section of the <a href="https://developers.google.com/api-client-library/javascript/features/authentication">JavaScript Client API documentation</a>. You need to use the OAuth client ID and your Project ID to send queries.</p>
<p>When the Google Client API is loaded in the web page you need to perform the following steps:</p>
<ul>
<li>Authorize the user.</li>
<li>If authorized, load the BigQuery API.</li>
<li>Load and initialize the map.</li>
</ul>
<h3>Authorize the user</h3>
<p>The end user of the application needs to authorize the application to access data in BigQuery on their behalf. The Google Client API for JavaScript handles the OAuth logic to do this.</p>
<p>In a real-world application you have many choices about how to integrate the authorization step.</p>
<p>For example, you could call <code>authorize()</code> from a UI element like a button, or do it when the page has loaded. Here you have chosen to authorize the user after the Google Client API for JavaScript has been loaded, by using a callback function in the <code>gapi.load()method</code>.</p>
<p>Next, write some code immediately after the <code>&lt;script&gt;</code> tag that loads the Google Client API for Javascript to load both the client library and the auth module so that you can authenticate the user straight away.</p>
<ol>
<li>
<p>Add this code to just after the comment <code>&lt;!-- Authorize Code --&gt;</code>:</p>
</li>
</ol>
<pre><code class="language-html prettyprint">    &lt;script type='text/javascript'&gt;&#x000A;    gapi.load('client:auth', authorize);&#x000A;    &lt;/script&gt;&#x000A;&#x000A;</code></pre>
<ql-infobox>
<b>Note:</b> The authorization process uses a pop-up window, so check that you do not block this when you run the code the first time.
</ql-infobox>
<ol start="2">
<li>
<p>Add this code just under the line starting <strong>let clientId =</strong> which is under <code>// ClientID var</code>:</p>
</li>
</ol>
<pre><code class="language-javascript prettyprint">    let scopes = 'https://www.googleapis.com/auth/bigquery';&#x000A;</code></pre>
<ol start="3">
<li>
<p>Add this code under <code>// authorize</code></p>
</li>
</ol>
<pre><code class="language-javascript prettyprint">    function authorize(event) {&#x000A;      gapi.auth.authorize({client_id: clientId, scope: scopes, immediate: false}, handleAuthResult);&#x000A;      return false;&#x000A;    }&#x000A;&#x000A;</code></pre>
<p>This code calls <code>gapi.auth.authorize</code> and the function <code>handleAuthResult</code> below will perform the API loading on successful authentication.</p>
<h3>On authorization, load the BigQuery API</h3>
<ol start="4">
<li>
<p>Add this code under <code>// handleAuthResult</code>:</p>
</li>
</ol>
<pre><code class="language-javascript prettyprint">    function handleAuthResult(authResult) {&#x000A;      if (authResult &amp;&amp; !authResult.error) {&#x000A;        loadApi();&#x000A;        return;&#x000A;      }&#x000A;      console.error('Not authorized.')  &#x000A;    }&#x000A;&#x000A;</code></pre>
<p>If the authorization was successful, you can load the API, otherwise post to the console.</p>
<h3>Load the map</h3>
<ol start="5">
<li>
<p>Alter the code under <code>// loadApi</code>:</p>
</li>
</ol>
<pre><code class="language-javascript prettyprint">    function loadApi(){&#x000A;      gapi.client.load('bigquery', 'v2').then(&#x000A;       () =&gt; initMap()&#x000A;      );&#x000A;    }&#x000A;&#x000A;</code></pre>
<p>This code loads the BigQuery library and then calls the <code>initMap</code> function.  You are changing the map initialization point from the library load to here, which is a more logical place to do it.  The next step removes the call from the library load.</p>
<ol start="6">
<li>
<p>Edit the line of code under <code>&lt;!-- Javascript Libraries --&gt;</code>.  Delete <code>&amp;callback=initMap</code> as this is no longer needed because <code>loadApi()</code> calls <code>initMap()</code>.</p>
</li>
</ol>
<h2 id="step8">Implement BigQuery API functions</h2>
<p>BigQuery API calls usually execute in seconds, but may not return a response immediately. You need some logic to poll BigQuery to find out the status of long-running jobs, and only fetch the results when the job is complete.</p>
<h3>Add needed variables</h3>
<ol>
<li>
<p>Add this code under the comment <code>// general vars</code>:</p>
</li>
</ol>
<pre><code class="language-javascript prettyprint">    let recordLimit = 10000;&#x000A;    let jobCheckTimer;&#x000A;    let heatmap;&#x000A;&#x000A;</code></pre>
<p>Here is what these commands do:</p>
<table>

<tr>
<th>var</th>
<th>purpose</th>
</tr>


<tr>
<td>recordLimit</td>
<td>controls the results returned from the query for this lab to a managable size</td>
</tr>
<tr>
<td>jobCheckTimer</td>
<td>timer used when checking for the query completion</td>
</tr>
<tr>
<td>heatmap</td>
<td>used to track the mapped locations on the map</td>
</tr>

</table>
<h3>Specifying the BigQuery project, dataset and table</h3>
<ol start="2">
<li>
<p>Add this code under the comment <code>// BigQuery vars</code>:</p>
</li>
</ol>
<pre><code class="language-javascript prettyprint">    let gcpProjectId = 'YOUR_PROJECT_ID';&#x000A;    let bigQueryProjectId = 'bigquery-public-data';&#x000A;    let datasetId = 'new_york';&#x000A;    let tableName = 'tlc_yellow_trips_2016';&#x000A;&#x000A;</code></pre>
<p>Replace <strong>YOUR_PROJECT_ID</strong> with your project ID.  You can find your project ID on the home page of the console.</p>
<p>Here is what this code does:</p>
<table>

<tr>
<th>var</th>
<th>purpose</th>
</tr>


<tr>
<td>gcpProjectId</td>
<td>is the billing account, needed for any charges</td>
</tr>
<tr>
<td>bigQueryProjectId</td>
<td>is where the dataset resides</td>
</tr>
<tr>
<td>datasetId</td>
<td>the name of the public dataset</td>
</tr>
<tr>
<td>tableName</td>
<td>the table inside the dataset</td>
</tr>

</table>
<h3>Sending a request</h3>
<ol start="3">
<li>
<p>Add this code under the comment <code>// sendQuery</code>.</p>
</li>
</ol>

```js
    function sendQuery(queryString){
      let request = gapi.client.bigquery.jobs.query({
          'query': queryString,
          'timeoutMs': 30000,
          'datasetId': datasetId,
          'projectId': gcpProjectId,
          'useLegacySql':false
      });
      request.execute(response => checkJobStatus(response.jobReference.jobId));
    }
```

<p>This code wraps the BigQuery request and passes the necessary variables.  It also sets up a check of the job status (the <code>checkJobStatus</code> is defined next).</p>
<h3>Check the status of a job</h3>
<ol start="4">
<li>
<p>Add this code under the comment <code>// checkJobStatus</code>.</p>
</li>
</ol>

```js
    function checkJobStatus(jobId){
      let request = gapi.client.bigquery.jobs.get({
        'projectId': gcpProjectId,
        'jobId': jobId
      });
      request.execute(response => {
        if (response.status.errorResult){
          // Handle any errors.
          console.log(response.status.error);
          return;
        }
        if (response.status.state == 'DONE'){
          // Get the results.
          clearTimeout(jobCheckTimer);
          getQueryResults(jobId);
          return;
        }
        // Not finished, check again in a moment.
        jobCheckTimer = setTimeout(checkJobStatus, 500, [jobId]);       
      });
    }
```

<p>This function checks the status of a job periodically, using the get API method and the jobId returned by the original query request. In this code the check runs every 500 milliseconds until the job is complete.</p>
<h3>Getting the results of a query</h3>
<ol start="5">
<li>
<p>Add this code under the comment <code>// getQueryResults</code>.</p>
</li>
</ol>

```js
    function getQueryResults(jobId){
      let request = gapi.client.bigquery.jobs.getQueryResults({
        'projectId': gcpProjectId,
        'jobId': jobId
      });
      request.execute(response => doHeatMap(response.result.rows))
    }
```

<p>In <code>checkJobStatus</code> you call <code>getQueryResults</code> once you see the query completes.  This code takes the query results and passes them to <code>doHeatMap</code> to draw.</p>
<h3>Rectangle queries</h3>
<ol start="6">
<li>
<p>Replace the line of code under  <code>// rectangle search handler</code> with this code.</p>
</li>
</ol>
<pre><code class="language-javascript prettyprint">      drawingManager.addListener('rectanglecomplete', rectangle =&gt; rectangleQuery(rectangle.getBounds()));&#x000A;</code></pre>
<p>This code passes the rectangle bounds to <code>rectangleQuery</code>.  <code>rectangleQuery</code> is defined shortly.</p>
<h3>Query BigQuery for the data for rectangular search</h3>
<ql-infobox>
A rectangular shape on a Google Map, or any map in the Web Mercator projection, does not represent a rectangular area on the ground. This is because lines of longitude converge towards the poles where they meet (Google Maps doesn't show areas above about 85 degrees north or south of the equator). The larger the ground area covered, the bigger the distortion of the true shape and area on the ground will be, but the effect should be negligible over short distances of up to a few kilometers.
</ql-infobox>
<ol start="7">
<li>
<p>Add this code under the comment <code>// rectangleQuery</code>:</p>
</li>
</ol>
<pre><code class="language-javascript prettyprint">    function rectangleQuery(latLngBounds){&#x000A;      let queryString = rectangleSQL(latLngBounds.getNorthEast(), latLngBounds.getSouthWest());&#x000A;      sendQuery(queryString);&#x000A;    }&#x000A;&#x000A;</code></pre>
<p>This code unpacks the coordinates and passes them to <code>rectangleSQL</code> to create the query string, then performs the query.</p>
<ol start="8">
<li>
<p>Add this code under the comment <code>// rectangleSQL</code>.</p>
</li>
</ol>

```js
    function rectangleSQL(ne, sw){
      var queryString = 'WITH SHAPE AS (select ST_MAKEPOLYGON(ST_MakeLine(['
      queryString += 'ST_GEOGPOINT(' + ne.lng() + ',' + ne.lat() +'), '
      queryString += 'ST_GEOGPOINT(' + sw.lng() + ',' + ne.lat() +'), '
      queryString += 'ST_GEOGPOINT(' + sw.lng() + ',' + sw.lat() +'), '
      queryString += 'ST_GEOGPOINT(' + ne.lng() + ',' + sw.lat() +')])) '
      queryString += 'AS POLY) '
      queryString += 'SELECT pickup_latitude, pickup_longitude '
      queryString +=  'FROM `' + bigQueryProjectId +'.' + datasetId + '.' + tableName + '`,SHAPE '
      queryString += 'WHERE ST_CONTAINS(POLY, ST_GEOGPOINT(pickup_longitude,pickup_latitude)) is TRUE'
      queryString += ' LIMIT ' + recordLimit;
      return queryString;
    }
```

<p>This code generates the query string for the passed coordinates.</p>
<h3>Visualizing the response</h3>
<p>You will visualize the density of locations using a heatmap. The Maps Javascript API has a <code>HeatmapLayer</code> class for this purpose. The <code>HeatmapLayer</code> takes an array of latitude and longitude coordinates, so it is quite easy to convert the rows returned from the query into a heatmap.</p>
<ol start="9">
<li>
<p>Add this code under the comment <code>// doHeatMap</code>.</p>
</li>
</ol>

```js
    function doHeatMap(rows){
      let heatmapData = [];
      if (heatmap!=null){
        heatmap.setMap(null);
      }
      for (let i = 0; i < rows.length; i++) {
          let f = rows[i].f;
          let coords = { lat: parseFloat(f[0].v), lng: parseFloat(f[1].v) };
          let latLng = new google.maps.LatLng(coords);
          heatmapData.push(latLng);
      }
      heatmap = new google.maps.visualization.HeatmapLayer({
          data: heatmapData
      });
      heatmap.setMap(map);
    }
```

<p>This code is called from <code>getQueryResults</code>.  <code>getQueryResults</code> passes a <code>response.result.rows</code> array which contains the latitude and longitude values of the taxi pickups.</p>
<p>Each row will have a property called <code>f</code> which is an array of columns. Each column will have a v property containing the value. Your code needs to loop through the columns in each row and extract the values.</p>
<p>Once the points are added to the heatmap, <code>setMap</code> is called to make them visible.</p>
<h3>Testing the page works now</h3>
<ol start="10">
<li>
<p>Save index.html.</p>
</li>
<li>
<p>Switch to the preview page and refresh the page.</p>
</li>
</ol>
<ql-warningbox>
You should see a popup, if not then check and unblock it.

<img src="./images/popup.png" width=400>
</ql-warningbox>
<ol>
<li>
<p>In the pop-up, click on your lab account.</p>
</li>
<li>
<p>Click <strong>Allow</strong>.</p>
<img src="./images/allow.png" width=400>
</li>
<li>
<p>Select the rectangle tool from the palette.</p>
</li>
<li>
<p>Zoom into Manhattan, then draw an area with the rectangle.</p>
</li>
<li>
<p>The request will take a few seconds to complete (depending on the size of your drawn box), so be patient. You will see the heat map displayed within the rectangle you drew.</p>
</li>
</ol>
<p><img alt="Rectangle Heat Map" src="https://cdn.qwiklabs.com/GEvBUlcz6mV1bi3VMdmdJttdX0%2FBdaY4TcUbuN8f%2FoY%3D"></p>
<p>Click <em>Check my progress</em> to verify the objective.
<ql-activity-tracking step="3">
Implement BigQuery API functions
</ql-activity-tracking></p>
<h2 id="step9">Adding circles functionality</h2>
<p>Now you will add the other palette option to the code, circles.</p>
<h3>Querying by radius around a point</h3>
<ol>
<li>
<p>Go back to the Code Editor and add this code under the comment <code>// circleQuery</code>.</p>
</li>
</ol>
<pre><code class="language-javascript prettyprint">    function circleQuery(circle){&#x000A;      let queryString = circleSQL(circle.getCenter(), circle.radius);&#x000A;      sendQuery(queryString);&#x000A;    }&#x000A;&#x000A;</code></pre>
<p>Using the same technique for rectangles, you can handle an OverlayComplete event to get the center and radius of a user-drawn circle, and build up the SQL for the query in the same way.</p>
<ol start="2">
<li>
<p>Add this code under the comment <code>// circleSQL</code>.</p>
</li>
</ol>

```js
	function circleSQL(center, radius){
        var queryString;
        var centerLat = center.lat();
        var centerLng = center.lng();
        var kmPerDegree = 111.045;
        queryString = 'SELECT pickup_latitude, pickup_longitude '
        queryString += 'FROM `' + bigQueryProjectId +'.' + datasetId + '.' + tableName + '` ';
        queryString += 'WHERE ST_DISTANCE(ST_GEOGPOINT(pickup_longitude,pickup_latitude),'
        queryString += 'ST_GEOGPOINT('+ centerLng + ', ' + centerLat + ')) <'+ radius;
        queryString +=' AND dropoff_latitude <> 0.0'
        queryString +=' LIMIT ' + recordLimit;
        return queryString;
    }
```

<ol start="3">
<li>
<p>Replace the line of code under  <code>// circle search handler</code> with this code.</p>
</li>
</ol>
<pre><code class="language-javascript prettyprint">      drawingManager.addListener('circlecomplete', circle =&gt; circleQuery(circle));&#x000A;</code></pre>
<p>The code adds a listener to fire the circleQuery code everytime you draw a circle.</p>
<ol start="4">
<li>
<p>Save index.html.</p>
</li>
<li>
<p>Switch to the web preview page and refresh the page.</p>
</li>
<li>
<p>Select the circle tool from the palette.</p>
</li>
<li>
<p>Zoom into Manhattan, then draw an area with the circle.</p>
</li>
<li>
<p>The request will take a few seconds to complete, so be patient. You will see the heat map displayed within the circle you drew.</p>
</li>
</ol>
<p><img alt="Circle Heat Map" src="https://cdn.qwiklabs.com/T2oEEF78ohcs%2B7pupUrDjV1%2F8nOHGQLTWiKsRezVg2s%3D"></p>
<p>Click <em>Check my progress</em> to verify the objective.
<ql-activity-tracking step="4">
Add circles functionality
</ql-activity-tracking></p>
<h2 id="step10">Congratulations!</h2>
<p>You have created a web page that will query a BigQuery table and use the geospatial data to draw a heat map, all using the Google APIs.</p>

<details close>
<summary>今回作った index.html の全体</summary>

```html
<!DOCTYPE html>
<html>
<!--
Copyright 2016 Google Inc.
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at
    http://www.apache.org/licenses/LICENSE-2.0
Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->
  <head>
    <title>Simple Map</title>
    <meta name="viewport" content="initial-scale=1.0">
    <meta charset="utf-8">
    <style>
      html, body {
        height: 100%;
        margin: 0;
        padding: 0;
      }
      #map {
        height: 100%;
      }
    </style>
    <script>

    // general vars
    let map;
    let drawingManager;
    let recordLimit = 10000;
    let jobCheckTimer;
    let heatmap;

    // ClientID var
    let clientId = '147276041032-7f58edf99vg8ppcuu3nkbhbcb26olt79.apps.googleusercontent.com'
    let scopes = 'https://www.googleapis.com/auth/bigquery';

    // BigQuery vars
    let gcpProjectId = 'qwiklabs-gcp-03-9fe7090980c5';
    let bigQueryProjectId = 'bigquery-public-data';
    let datasetId = 'new_york';
    let tableName = 'tlc_yellow_trips_2016';

    // initMap
    function initMap() {
        map = new google.maps.Map(document.getElementById('map'), {
          center: {lat: 40.744593, lng: -73.990370}, // Manhattan, New York.
        zoom: 15
        });
        // call setupDrawingTools
        setUpDrawingTools();

    }

    // setUpDrawingTools
    function setUpDrawingTools(){
      // Initialize drawing manager.
      drawingManager = new google.maps.drawing.DrawingManager({
        drawingMode: google.maps.drawing.OverlayType.CIRCLE,
        drawingControl: true,
        drawingControlOptions: {
          position: google.maps.ControlPosition.TOP_LEFT,
          drawingModes: [
            google.maps.drawing.OverlayType.CIRCLE,
            google.maps.drawing.OverlayType.RECTANGLE
          ]
        },
        circleOptions: {
          fillOpacity: 0
        },
        rectangleOptions: {
          fillOpacity: 0
        }
      });
      drawingManager.setMap(map);
      // each one of these handlers do nothing currently, we will replace them later
      // with working code

      // rectangle search handler
      drawingManager.addListener('rectanglecomplete', rectangle => rectangleQuery(rectangle.getBounds()));

      // circle search handler
      drawingManager.addListener('circlecomplete', circle => circleQuery(circle));

    }

    // authorize
    function authorize(event) {
      gapi.auth.authorize({client_id: clientId, scope: scopes, immediate: false}, handleAuthResult);
      return false;
    }

    // handleAuthResult
    function handleAuthResult(authResult) {
      if (authResult && !authResult.error) {
        loadApi();
        return;
      }
      console.error('Not authorized.')  
    }

    // loadApi
    function loadApi(){
      gapi.client.load('bigquery', 'v2').then(
       () => initMap()
      );
    }

    // sendQuery
    function sendQuery(queryString){
      let request = gapi.client.bigquery.jobs.query({
          'query': queryString,
          'timeoutMs': 30000,
          'datasetId': datasetId,
          'projectId': gcpProjectId,
          'useLegacySql':false
      });
      request.execute(response => checkJobStatus(response.jobReference.jobId));
    }

    // checkJobStatus
    function checkJobStatus(jobId){
      let request = gapi.client.bigquery.jobs.get({
        'projectId': gcpProjectId,
        'jobId': jobId
      });
      request.execute(response => {
        if (response.status.errorResult){
          // Handle any errors.
          console.log(response.status.error);
          return;
        }
        if (response.status.state == 'DONE'){
          // Get the results.
          clearTimeout(jobCheckTimer);
          getQueryResults(jobId);
          return;
        }
        // Not finished, check again in a moment.
        jobCheckTimer = setTimeout(checkJobStatus, 500, [jobId]);       
      });
    }

    // getQueryResults
    function getQueryResults(jobId){
      let request = gapi.client.bigquery.jobs.getQueryResults({
        'projectId': gcpProjectId,
        'jobId': jobId
      });
      request.execute(response => doHeatMap(response.result.rows))
    }

    // doHeatMap
    function doHeatMap(rows){
      let heatmapData = [];
      if (heatmap!=null){
        heatmap.setMap(null);
      }
      for (let i = 0; i < rows.length; i++) {
          let f = rows[i].f;
          let coords = { lat: parseFloat(f[0].v), lng: parseFloat(f[1].v) };
          let latLng = new google.maps.LatLng(coords);
          heatmapData.push(latLng);
      }
      heatmap = new google.maps.visualization.HeatmapLayer({
          data: heatmapData
      });
      heatmap.setMap(map);
    }

    // rectangleQuery
    function rectangleQuery(latLngBounds){
      let queryString = rectangleSQL(latLngBounds.getNorthEast(), latLngBounds.getSouthWest());
      sendQuery(queryString);
    }

    // rectangleSQL
    function rectangleSQL(ne, sw){
      var queryString = 'WITH SHAPE AS (select ST_MAKEPOLYGON(ST_MakeLine(['
      queryString += 'ST_GEOGPOINT(' + ne.lng() + ',' + ne.lat() +'), '
      queryString += 'ST_GEOGPOINT(' + sw.lng() + ',' + ne.lat() +'), '
      queryString += 'ST_GEOGPOINT(' + sw.lng() + ',' + sw.lat() +'), '
      queryString += 'ST_GEOGPOINT(' + ne.lng() + ',' + sw.lat() +')])) '
      queryString += 'AS POLY) '
      queryString += 'SELECT pickup_latitude, pickup_longitude '
      queryString +=  'FROM `' + bigQueryProjectId +'.' + datasetId + '.' + tableName + '`,SHAPE '
      queryString += 'WHERE ST_CONTAINS(POLY, ST_GEOGPOINT(pickup_longitude,pickup_latitude)) is TRUE'
      queryString += ' LIMIT ' + recordLimit;
      return queryString;
    }

    // circleQuery
    function circleQuery(circle){
      let queryString = circleSQL(circle.getCenter(), circle.radius);
      sendQuery(queryString);
    }

    // circleSQL
	function circleSQL(center, radius){
        var queryString;
        var centerLat = center.lat();
        var centerLng = center.lng();
        var kmPerDegree = 111.045;
        queryString = 'SELECT pickup_latitude, pickup_longitude '
        queryString += 'FROM `' + bigQueryProjectId +'.' + datasetId + '.' + tableName + '` ';
        queryString += 'WHERE ST_DISTANCE(ST_GEOGPOINT(pickup_longitude,pickup_latitude),'
        queryString += 'ST_GEOGPOINT('+ centerLng + ', ' + centerLat + ')) <'+ radius;
        queryString +=' AND dropoff_latitude <> 0.0'
        queryString +=' LIMIT ' + recordLimit;
        return queryString;
}

</script>
  </head>
  <body>
    <div id="map"></div>
    <!-- Javascript Libraries -->
    <script src="https://maps.googleapis.com/maps/api/js?libraries=visualization,drawing&key=AIzaSyBy-dzHswjb9i9IpL-FatVlrlPZcDUqiFs" async defer></script>

    <!-- BigQuery API -->
    <script src='https://apis.google.com/js/client.js'></script>

    <!-- Authorize Code -->
    <script type='text/javascript'>
    gapi.load('client:auth', authorize);
    </script>


  </body>
</html>
```

</details>

<h3>Finish Your Quest</h3>
<p>This self-paced lab is part of the <a href="https://google.qwiklabs.com/quests/103">Creating with Google Maps</a> Quest. A Quest is a series of related labs that form a learning path. Completing this Quest earns you the badge above, to recognize your achievement. You can make your badge public and link to them in your online resume or social media account. <a href="https://google.qwiklabs.com/quests/103/enroll">Enroll in this Quest</a> and get immediate completion credit if you've taken this lab.  <a href="https://google.qwiklabs.com/catalog">See other available Qwiklabs Quests</a>.</p>
<h3>Take your next lab</h3>
<p>Continue your Quest with <a href="https://google.qwiklabs.com/catalog_lab/1716">Build a Nearby Business Search Service with Google Maps Platform</a>, or check out these suggestions:</p>
<ul>
<li>
<p><a href="https://google.qwiklabs.com/catalog_lab/1118">Google Maps Web Services Proxy for Mobile Applications</a></p>
</li>
<li>
<p><a href="https://google.qwiklabs.com/catalog_lab/366">Mapping the NYC Subway</a></p>
</li>
</ul>
<h3>Next Steps / Learn More</h3>
<p>This lab is based on <a href="https://codelabs.developers.google.com/codelabs/bigquery-maps-api/index.html?index=..%2F..index#0">Querying and Visualising Location Data in BigQuery Using Google Maps API</a>, with updates to the Geographic Information Systems <a href="https://cloud.google.com/bigquery/docs/reference/standard-sql/geography_functions">functions</a> added to BigQuery.</p>

# Build a Nearby Business Search Service with Google Maps Platform（Google Maps Platform を使って、近くのお店を探すサービスをつくる）

<h2 id="step2">Overview</h2>
<p>Learn to use Google Maps Platform's Maps and Places APIs to build a local business search, which geolocates the user and shows interesting places around them. The app integrates location, place details, place photos, and more.</p>
<h3><strong>What is Google Maps Platform?</strong></h3>
<p>Google Maps Platform brings the wealth of Google's location-based information to your app. Millions of developers use Google Maps Platform to help their users navigate the world around them, on the web and on mobile devices. The Google Maps Platform offers three core products:</p>
<ul>
<li>
<p><strong>Maps</strong> enables you to build customized, agile experiences that bring the real world to your users with static and dynamic maps, and 360° Street View imagery.</p>
</li>
<li>
<p><strong>Places</strong> helps your users discover the world with rich location data for over 150 million places by using phone numbers, addresses, and real-time signals.</p>
</li>
<li>
<p><strong>Routes</strong> gives your users the best way to get from A to Z with high-quality directions that factor in real-time traffic updates. Determine the route a vehicle travels to create more precise itineraries.</p>
</li>
</ul>
<h3><strong>What you'll build</strong></h3>
<p>In this lab you're going to build a webpage that displays a Google map centered on the user's location, finds nearby places, and displays the places as clickable markers to show more details about each place. <img alt="ae1caf211daa484d.png" src="https://cdn.qwiklabs.com/qcCgkKIomS2ed3o15kq7KDSeBrCv%2BfkHRaK%2B5A%2Bg%2B5k%3D"></p>
<h3><strong>What you'll learn</strong></h3>
<ul>
<li>
<p>How to create a customizable map</p>
</li>
<li>
<p>How to geolocate the user</p>
</li>
<li>
<p>How to search for nearby places and display the results</p>
</li>
<li>
<p>How to fetch and display details about a place</p>
</li>
<li>
<p>How to deploy a web app using Google App Engine</p>
</li>
</ul>

<h3><strong>Start App Engine</strong></h3>
<p>In the Google Cloud Platform Console, from the Navigation menu, select __App Engine. __This brings up the "Welcome to App Engine" screen. Click <strong>Create Application</strong>.</p>
<p><img alt="ddb57cee088543ca.png" src="https://cdn.qwiklabs.com/jcE4G6XAGa4Eh8nLmVMM%2BO2rOK87MT4y7Dm5Et1xWaM%3D"></p>
<p>Accept the default region (or select a new one) and click <strong>Create app.</strong></p>
<p><img alt="e8329cc3051c3315.png" src="https://cdn.qwiklabs.com/gJ%2B21ZYl%2Bfz2VqQ%2F8t8WxDVGOm5lP6xV12ESxdn5Psk%3D"></p>
<p>Accept the standard settings (us-central, Python, Standard) then <strong>Next</strong> to continue.</p>
<p>Once you get a notification that "Your App Engine app has successfully been created", open the Cloud Shell by clicking the <strong>Activate Cloud Shell</strong> button in the upper right corner of the console.</p>
<p><img alt="dd53bbfdfebf62a8.png" src="https://cdn.qwiklabs.com/RDVn0eDCfRC1WL%2FRN4xNExzCcI0rGz%2BsL9%2BvRdVtKXI%3D"></p>
<p>Then click <strong>Start Cloudshell</strong>.</p>
<p>In the Cloud Shell, download all the sample code needed for this lab by running this command:</p>
<pre><code>git clone https://github.com/googlecodelabs/google-maps-nearby-search-js/&#x000A;</code></pre>
<p>Move into the repo you just cloned.</p>
<pre><code>cd google-maps-nearby-search-js&#x000A;</code></pre>
<p>Copy the <code>index.html</code> file from the repo into the <code>work/</code> folder, then move into your <code>work/</code> folder. Throughout the rest of the lab, make your edits in the version in the <code>work/</code> folder.</p>
<pre><code>cp index.html work/&#x000A;cd work&#x000A;</code></pre>
<aside class="warning"><p><strong>Note: </strong>If for any reason you are  unable to access the github repository, you can still complete this lab - just create two files. </p>
<p>Make a new directory called <code>work</code>, and in that directory make a blank file called <code>index.html</code> and a blank file called <code>app.yaml</code>.  These are the commands to run in Cloud Shell to accomplish the above:</p>
<p><code>mkdir work</code></p>
<p><code>cd work</code></p>
<p><code>touch index.html</code></p>
<p><code>touch app.yaml</code></p>
<p>You can then copy the content for <code>app.yaml</code> below, then run</p>
<p><code>edit app.yaml</code></p>
<p>in the Cloud Shell to paste the code into <code>app.yaml</code>.</p>
<p><strong>Note:</strong> When you open the Cloud Shell Editor, it may close your current Shell session and instantiates a new one below the new Editor window. The new Shell session starts in your Home directory so you will need to manually move into your project directory again in the new window before you can deploy your app.<code>cd google-maps-nearby-search-js</code></p>
</aside>
<p>The <code>work</code> folder contains an <code>app.yaml</code> file. This is the file that configures your App Engine app's settings, such as specifying how URL paths correspond to request handlers and static files.</p>
<p>For this lab you do not need to modify this file. This is what the <code>app.yaml</code> looks like:</p>
<pre><code>runtime: python37&#x000A;&#x000A;handlers:&#x000A;- url: /&#x000A;  static_files: index.html&#x000A;  upload: index.html&#x000A;&#x000A;- url: /index\.html&#x000A;  static_files: index.html&#x000A;  upload: index.html&#x000A;&#x000A;- url: /.*&#x000A;  secure: always&#x000A;  script: auto&#x000A;</code></pre>
<p>Now you are ready to deploy your app to App Engine and host it on Google Cloud Platform.</p>
<p>Use this command, which looks for the <code>app.yaml</code> file in the current directory and deploys the app according to the settings specified within:</p>
<pre><code>gcloud app deploy&#x000A;</code></pre>
<p>Type "y" when asked if you want to continue.</p>
<p>To view your new app, get the URL for your web app with this command:</p>
<pre><code>gcloud app browse&#x000A;</code></pre>
<p>This returns a URL that you can paste into your web browser. In the case of this lab, it follows the format  <a href="https://your-project-id.appspot.com">https://your-project-id.appspot.com</a><code></code></p>
<p>Right now there's nothing to look at. Each time you want to view changes to your app based on edits in your code, run <code>gcloud app deploy</code> <code>app.yaml</code> to deploy the changes, and reload the page in your browser.</p>
<p>Click <em>Check my progress</em> to verify the objective.
<ql-activity-tracking step="1">
Start App Engine
</ql-activity-tracking></p>
<h3>Enable Google Maps Platform APIs and get an API key</h3>
<p>This section explains how to authenticate your app to the Maps JavaScript API and Places API using your own API key.</p>
<p>Follow these steps to enable the APIs used in this lab and get an API key:</p>
<ol>
<li>In the Google Cloud Platform Console, from the Navigation menu, select <strong>APIs &amp; Services <strong>&gt;</strong> Library</strong>.</li>
<li>Click the <strong>Maps JavaScript API</strong> tile and click the <strong>Enable</strong> button.</li>
<li>Repeat steps 1 and 2 to enable the <strong>Places API</strong>.</li>
<li>From the <strong>Navigation menu</strong>, select <strong>APIs &amp; Services <strong>&gt;</strong> Credentials</strong>.</li>
<li>Click <strong>Create Credentials</strong> and choose <strong>API key</strong>.</li>
<li>You will see an "API key created" confirmation screen and your new key is listed on the Credentials page. You will need this key in the next step to add a map to your webpage.</li>
</ol>
<aside class="special"><p><strong>Tip:</strong> To prevent quota theft, secure your API key. See <a href="https://developers.google.com/maps/api-key-best-practices#best_practice_list" target="_blank">Protecting API keys</a> for best practices and <a href="https://developers.google.com/maps/api-key-best-practices#restrict_apikey" target="_blank">Restrict an API key</a> for specific settings to edit in your Google Cloud Platform Console.</p>
</aside>
<p>Click <em>Check my progress</em> to verify the objective.
<ql-activity-tracking step="2">
Enable Google Maps Platform APIs and get an API key
</ql-activity-tracking></p>
<h2 id="step3">Step 1: Create a Map with a Default Center</h2>
<p>There are three steps to creating a Google map on your web page:</p>
<ul>
<li>
<p>Create an HTML page</p>
</li>
<li>
<p>Get an API key</p>
</li>
<li>
<p>Add a map</p>
</li>
</ul>
<h3><strong>Create an HTML page</strong></h3>
<p>Below is the map you'll create for this step. The map is centered on the Sydney Opera House in Sydney, Australia. If the user denies permission to get their location, the map defaults to this location and still be able to provide interesting search results.</p>
<p><img alt="569b9781658fec74.png" src="https://cdn.qwiklabs.com/UmJIccnYt8WNdpnk7xerCIn8VpA%2BbszqjGD8f2NZR2M%3D"></p>
<p>The code for a basic HTML web page is provided in the <code>index.html</code> file, which is in the repo you downloaded. You'll be making modifications to this file, and can use the Cloud Shell Editor as a code editor to modify the code in the <code>work</code>/<code>index.html</code> file, or your favorite text editor. At the end of each step in this lab, there is a link to a sample <code>index.html</code> that you can compare your own code against.</p>
<p>There are two ways to open the Cloud Shell Editor:</p>
<ul>
<li>From the Cloud Shell command line, type <code>edit index.html</code> and confirm that you want to open this file in the Cloud Shell Editor.</li>
<li>In the upper right of the Cloud Shell, click the "pencil" icon to open the Cloud Shell Editor.</li>
</ul>
<p><img alt="ba731110a97f468f.png" src="https://cdn.qwiklabs.com/O1pSCpaSe6p5nxOMmjQg3Vsf0YwHaqW2bT56hM6Iym0%3D"></p>
<aside class="warning"><p><strong>Note: </strong>When you open the Cloud Shell Editor, it may close your current Shell session and instantiate a new one below the new Editor window. The new Shell session starts in your Home directory, so you will need to manually move into your project directory again in the new window.<code>cd google-maps-nearby-search-js</code></p>
</aside>
<p>For your reference, the current contents of <code>index.html</code> are provided below:</p>

```html
<!DOCTYPE html>
<html>

<head>
  <title>Google Maps Place Picker</title>
  <meta name="viewport" content="initial-scale=1.0, user-scalable=no">
  <meta charset="utf-8">
  <style>
    /* Always set the map height explicitly to define the size of the div
     * element that contains the map. */
    #map {
      height: 100%;
      background-color: grey;
    }

    /* Optional: Makes the sample page fill the window. */
    html,
    body {
      height: 100%;
      margin: 0;
      padding: 0;
    }
  </style>
</head>

<body>
  <!-- Map appears here -->
  <div id="map"></div>
</body>

</html>
```

<p>Note that this is a very basic page with a single <code>div</code> element and a <code>style</code> element. You can add any content you like to the web page. You should see a large div with a grey background spanning the full visible window.</p>
<aside class="special"><p>From this point forward, all testing/verification (e.g. the<strong> Test It Out</strong> sections in subsequent steps) will involve you typing the <code>gcloud app deploy</code> command in the <code>/work</code> directory in your Cloud Shell and reloading the web page in your browser to see the changes you've made.</p>
</aside>
<h3>Add a map</h3>
<p>This section shows you how to load the Maps JavaScript API into your web page, and how to write your own JavaScript that uses the API to add a map to the webpage.</p>
<p>Add this <code>script</code> code after the <code>map</code> &lt;div&gt; and before the close <code>&lt;/body&gt;</code> tag.</p>
<pre><code> &lt;script&gt;&#x000A;    /* Note: This example requires that you consent to location sharing when&#x000A;     * prompted by your browser. If you see the error "Geolocation permission&#x000A;     * denied.", it means you probably did not give permission for the browser * to locate you. */&#x000A;    let pos;&#x000A;    let map;&#x000A;    function initMap() {&#x000A;      // Set the default location and initialize all variables&#x000A;      pos = { lat: -33.857, lng: 151.213 };&#x000A;      map = new google.maps.Map(document.getElementById('map'), {&#x000A;        center: pos,&#x000A;        zoom: 15&#x000A;      });&#x000A;    }&#x000A;  &lt;/script&gt;&#x000A;  &lt;script async defer src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&amp;callback=initMap"&gt;&#x000A;  &lt;/script&gt;&#x000A;</code></pre>
<h3><strong>Retrieve your API key</strong></h3>
<ol>
<li>
<p>In the Console you should still see the <strong>APIs &amp; Services <strong>&gt;</strong> Credentials</strong>.</p>
</li>
<li>
<p>Copy the value of the key parameter and replace "<code>YOUR_API_KEY</code>" in the script source URL with the API key you just created:</p>
</li>
</ol>
<pre><code>    &lt;script async defer&#x000A;    src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&amp;callback=initMap"&gt;&#x000A;    &lt;/script&gt;&#x000A;</code></pre>
<p><strong>Save</strong> the file.</p>
<h3><strong>Test It Out</strong></h3>
<p>In your Cloud Shell, ensure you are in the <code>work/</code> directory. Run the following to deploy your updated code:</p>
<pre><code>gcloud app deploy&#x000A;</code></pre>
<p>Reload the browser tab that contains your web app. You should see a map appear now where the grey rectangle was before. If you see an error message instead, make sure you have replaced "<code>YOUR_API_KEY</code>" in the final <code>&lt;script&gt;</code> tag with your own API key. See above for how to get an API key if you don't already have one.</p>
<h4><strong>Full Sample Code</strong></h4>
<p>The full code for this project through Step 1 is  <a href="https://github.com/googlecodelabs/google-maps-nearby-search-js/blob/master/step1/index.html">available on Github</a>.</p>
<h2 id="step4">Step 2: Geolocate Your User</h2>
<p>Next, you'll want to display the geographic location of the user or device on a Google map, using your browser's HTML5 Geolocation feature along with the Maps JavaScript API.</p>
<p>If you were standing in San Jose, CA, you'd see a map that displays your geographic location:</p>
<p><img alt="1dbb3fec117cd895.png" src="https://cdn.qwiklabs.com/XpIzmY56aQMarVhcWC0Aso7JHWn4cU5Kv4WupcWZVXQ%3D"></p>
<h3><strong>What is Geolocation?</strong></h3>
<p>Geolocation refers to the identification of the geographic location of a user or computing device via a variety of data collection mechanisms. Typically, most geolocation services use network routing addresses or internal GPS devices to determine this location. This application uses the web browser's W3C Geolocation standard navigator.geolocation property to determine the user's location.</p>
<h3><strong>Try it yourself</strong></h3>
<p>Here's the entire code you need to request permission to get the user's location and re-center the map on that location. Modify the code inside the main <code>&lt;script&gt;</code> tag:</p>

```html
 <script>
    /* Note: This example requires that you consent to location sharing when
     * prompted by your browser. If you see the error "Geolocation permission
     * denied.", it means you probably did not give permission for the browser
     * to locate you. */
    let pos;
    let map;
    let bounds;
    let infoWindow;
    let currentInfoWindow;
    let service;
    let infoPane;
    function initMap() {
      // Initialize variables
      bounds = new google.maps.LatLngBounds();
      infoWindow = new google.maps.InfoWindow;
      currentInfoWindow = infoWindow;

      // Try HTML5 geolocation
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(function (position) {
          pos = {
            lat: position.coords.latitude,
            lng: position.coords.longitude
          };
          map = new google.maps.Map(document.getElementById('map'), {
            center: pos,
            zoom: 15
          });
          bounds.extend(pos);

          infoWindow.setPosition(pos);
          infoWindow.setContent('Location found.');
          infoWindow.open(map);
          map.setCenter(pos);
        }, function () {
          handleLocationError(true, infoWindow);
        });
      } else {
        // Browser doesn't support geolocation
        handleLocationError(false, infoWindow);
      }
    }

    // Handle a geolocation error
    function handleLocationError(browserHasGeolocation, infoWindow) {
      // Set default location to Sydney, Australia
      pos = { lat: -33.856, lng: 151.215 };
      map = new google.maps.Map(document.getElementById('map'), {
        center: pos,
        zoom: 15
      });

      // Display an InfoWindow at the map center
      infoWindow.setPosition(pos);
      infoWindow.setContent(browserHasGeolocation ?
        'Geolocation permissions denied. Using default location.' :
        'Error: Your browser doesn\'t support geolocation.');
      infoWindow.open(map);
      currentInfoWindow = infoWindow;
    }
  </script>
```

<h3>Test it out</h3>
<p>Deploy your changes:</p>
<pre><code>gcloud app deploy&#x000A;</code></pre>
<p>Reload your page, your browser should now ask you for permission to share your location with the app.</p>
<ul>
<li>Try clicking "Block" one time to see if it handles the error gracefully and remains centered on Sydney.</li>
<li>Reload again and try clicking "Allow" to see if the geolocation works and moves the map to your current location.</li>
</ul>
<p><strong>Note:</strong> Your browser might save the last setting of permissions you granted to this page. In order to choose a different permission (allow when you previously blocked, or block when you previously allowed), you may need to clear your browser settings. In Chrome, there is a location icon at the right side of the address bar. Clicking the icon brings up a window that describes your current setting and allows you to clear that setting for future visits. Click <strong>Clear these settings for future visits</strong> any time you want to test handling of different block/allow permissions of geolocation.</p>
<p><img alt="99b0008cb56dcfc6.png" src="https://cdn.qwiklabs.com/WZjCseTkA5TlWnmzkuEsN5OJixEtgLjg5YZgF2xaQ%2B0%3D"></p>
<h4><strong>Full Sample Code</strong></h4>
<p>The full code for this project through Step 2 is  <a href="https://github.com/googlecodelabs/google-maps-nearby-search-js/blob/master/step2/index.html">available on Github</a>.</p>
<h2 id="step5">Step 3: Search for Nearby Places</h2>
<p>A Nearby Search lets you search for places within a specified area by keyword or type. A Nearby Search must always include a location, which can be specified in one of two ways:</p>
<ul>
<li>a <code>LatLngBounds</code> object defining a rectangular search area.</li>
<li>a circular area defined as the combination of the <code>location</code> property — specifying the center of the circle as a <code>LatLng</code> object — and a radius, measured in meters.</li>
</ul>
<p>Initiate a Nearby Search with a call to the <code>PlacesService</code>'s <code>nearbySearch()</code> method, which returns an array of <code>PlaceResult</code> objects.</p>
<h3><strong>Load the Places Library</strong></h3>
<p>First, to access the Places Library services, update the <code>&lt;script&gt;</code> tag where you included the maps javascript api  to introduce the <code>libraries</code> parameter and add <code>places</code> as a value:</p>
<pre><code>&lt;script async defer&#x000A;    src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&amp;libraries=places&amp;callback=initMap"&gt;&#x000A;</code></pre>
<p>If you are loading multiple libraries, you would separate them with a comma (no spaces). For example:</p>
<pre><code>&lt;script async defer&#x000A;    src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&amp;libraries=places,geometry&amp;callback=initMap"&gt;&#x000A;</code></pre>
<h3><strong>Call the Places Nearby search request and handle the response</strong></h3>
<p>Next, form a PlaceSearch Request. The minimum required fields are:</p>
<ul>
<li>Either of:</li>
<li>
<code>bounds</code>, which must be a <code>google.maps.LatLngBounds</code> object defining the rectangular search area; or</li>
<li>a <code>location</code> and a <code>radius</code>; the former takes a <code>google.maps.LatLng</code> object, and the latter takes a simple integer, representing the circle's radius in meters. The maximum allowed radius is 50 000 meters. Note that when rankBy is set to DISTANCE, you must specify a location but you cannot specify a radius or bounds.</li>
<li>Either of:</li>
<li>A <code>keyword</code> - A term to be matched against all available fields, including but not limited to name, type, and address, as well as customer reviews and other third-party content.</li>
<li>A <code>type</code> - Restricts the results to places matching the specified type. Only one type may be specified (if more than one type is provided, all types following the first entry are ignored). See the  <a href="https://developers.google.com/places/supported_types">list of supported types</a>.</li>
</ul>
<p>For this lab you will use the user's current position as the location for the search and rank results by distance.</p>
<p>Add the following to the main <code>script</code> to write two functions to call the search and handle the response. The keyword <code>sushi</code> is used as the search term, but you can change it to any keyword you like.</p>

```js
   // Perform a Places Nearby Search Request
    function getNearbyPlaces(position) {
      let request = {
        location: position,
        rankBy: google.maps.places.RankBy.DISTANCE,
        keyword: 'sushi'
      };

      service = new google.maps.places.PlacesService(map);
      service.nearbySearch(request, nearbyCallback);
    }

    // Handle the results (up to 20) of the Nearby Search
    function nearbyCallback(results, status) {
      if (status == google.maps.places.PlacesServiceStatus.OK) {
        createMarkers(results);
      }
    }
```

<p>The code to define the <code>createMarkers</code> function is provided in the next section, titled "Generate markers for search results".</p>
<p>Now you can call <code>getNearbyPlaces</code> after attempting geolocation.</p>
<p>Add this line in two places:</p>
<ol>
<li>
<p>To the end of the <code>if (navigator.geolocation)</code> block, above <code>//Browser doesn't support geolocation</code> block, beneath the call <code>map.setCenter(pos);:</code></p>
</li>
</ol>
<pre><code>         // Call Places Nearby Search on user's location&#x000A;          getNearbyPlaces(pos);&#x000A;</code></pre>
<ol start="2">
<li>
<p>At the end of the <code>handleLocationError</code> function:</p>
</li>
</ol>
<pre><code>     // Call Places Nearby Search on the default location&#x000A;      getNearbyPlaces(pos);&#x000A;</code></pre>
<h3><strong>Generate markers for search results</strong></h3>
<p>A marker identifies a location on a map. By default, a marker uses a standard image. See the  <a href="https://developers.google.com/maps/documentation/javascript/markers">Markers guide</a> for information about customizing marker images.</p>
<p>The <code>google.maps.Marker</code> constructor takes a single <code>Marker options</code> object literal, specifying the initial properties of the marker.</p>
<p>The following fields are particularly important and commonly set when constructing a marker:</p>
<ul>
<li>
<code>position</code> (required) specifies a <code>LatLng</code> identifying the initial location of the marker.</li>
<li>
<code>map</code> (optional) specifies the Map on which to place the marker. If you do not specify the map on construction of the marker, the marker is created but is not attached to (or displayed on) the map. You may add the marker later by calling the marker's <code>setMap()</code> method.</li>
</ul>
<p>Add the code below to set the position, map, and title for one marker per place returned in the response. You also use the <code>extend</code> method of our <code>bounds</code> variable to ensure that our center and all the markers are visible on the map.</p>

```js
   // Set markers at the location of each place result
    function createMarkers(places) {
      places.forEach(place => {
        let marker = new google.maps.Marker({
          position: place.geometry.location,
          map: map,
          title: place.name
        });

        // Adjust the map bounds to include the location of this marker
        bounds.extend(place.geometry.location);
      });
      /* Once all the markers have been placed, adjust the bounds of the map to
       * show all the markers within the visible area. */
      map.fitBounds(bounds);
    }
```

<h3><strong>Test it out</strong></h3>
<ul>
<li>
<p>Run <code>gcloud app deploy</code> in your Cloud Shell.</p>
</li>
<li>
<p>Reload the page and click "Allow" to give geolocation permissions.</p>
</li>
<li>
<p>You should see up to twenty red markers around the center location of the map.</p>
</li>
<li>
<p>Reload again and block geolocation permissions this time. Do you still get results at the default center of your map (in our sample, the default is in Sydney, Australia)?</p>
</li>
</ul>
<h4><strong>Full sample code</strong></h4>
<p>The full code for this project through Step 3 is  <a href="https://github.com/googlecodelabs/google-maps-nearby-search-js/blob/master/step3/index.html">available on Github</a>.</p>
<h2 id="step6">Step 4: Show Place Details on Demand</h2>
<p>Once you have a place's Place ID (delivered as one of the fields in the results of your nearby search), you can request additional details about the place, such as its complete address, phone number, user rating and reviews, etc. In this lab you'll make a sidebar to display place detail information and make the markers interactive so the user can select places to view details.</p>
<h3><strong>Make a generic sidebar</strong></h3>
<p>Here's some simple code for a sidebar that you can use to slide out and display the place details when the user clicks on a marker.</p>
<p>Add these to the end of the <code>style</code> tag:</p>

```css
   /* Styling for an info pane that slides out from the left.
     * Hidden by default. */
    #panel {
      height: 100%;
      width: null;
      background-color: white;
      position: fixed;
      z-index: 1;
      overflow-x: hidden;
      transition: all .2s ease-out;
    }

    .open {
      width: 250px;
    }

    /* Styling for place details */
    .hero {
      width: 100%;
      height: auto;
      max-height: 166px;
      display: block;
    }

    .place,
    p {
      font-family: 'open sans', arial, sans-serif;
      padding-left: 18px;
      padding-right: 18px;
    }

    .details {
      color: darkslategrey;
    }

    a {
      text-decoration: none;
      color: cadetblue;
    }
```

<h3><strong>Add click listeners to the markers</strong></h3>
<p>In the <code>createMarkers</code> function, add a <code>click listener</code> to each marker as you create them. The <code>click listener</code> fetches details about the place associated with that marker and calls the function to display the details.</p>
<p>Add the following at the end of the <code>forEach</code> loop inside <code>createMarkers</code>. The <code>showDetails</code> method will be implemented in the next section.</p>

```js
       // Add click listener to each marker
        google.maps.event.addListener(marker, 'click', () => {
          let request = {
            placeId: place.place_id,
            fields: ['name', 'formatted_address', 'geometry', 'rating',
              'website', 'photos']
          };

          /* Only fetch the details of a place when the user clicks on a marker.
           * If we fetch the details for all place results as soon as we get
           * the search response, we will hit API rate limits. */
          service.getDetails(request, (placeResult, status) => {
            showDetails(placeResult, marker, status)
          });
        });
```

<p>In the <code>getDetail</code>s request above, the <code>placeId</code> property specifies a single place for the details request, and the <code>fields</code> property is an array of field names for information you want returned about the place. A full list of fields you can request is in the  <a href="https://developers.google.com/maps/documentation/javascript/reference/places-service#PlaceResult">PlaceResult interface documentation</a>.</p>
<h3><strong>Show place details in an info window</strong></h3>
<p>An info window displays content (usually text or images) in a pop-up window above the map, at a given location. The info window has a content area and a tapered stem. The tip of the stem is attached to a specified location on the map.</p>
<p>Add the following below the "Adjust the map bounds..." section to attach an info window to a marker (you could also attach an info window to a specific latitude/longitude).</p>

```js
   // Builds an InfoWindow to display details above the marker
    function showDetails(placeResult, marker, status) {
      if (status == google.maps.places.PlacesServiceStatus.OK) {
        let placeInfowindow = new google.maps.InfoWindow();
        placeInfowindow.setContent('<div><strong>' + placeResult.name +
          '</strong><br>' + 'Rating: ' + placeResult.rating + '</div>');
        placeInfowindow.open(marker.map, marker);
        currentInfoWindow.close();
        currentInfoWindow = placeInfowindow;
        showPanel(placeResult);
      } else {
        console.log('showDetails failed: ' + status);
      }
    }
```

<p>Here, create an <code>InfoWindow</code> that displays the name and rating of the business, and attaches that window to the marker. You will define <code>showPanel</code> in the next section for displaying details in a sidebar.</p>
<h3><strong>Load place details in a sidebar</strong></h3>
<p>Use the same details returned in the <code>PlaceResult</code> object to populate another div. In this sample, use <code>infoPane</code> which is an arbitrary variable name for the div with the ID "<code>panel</code>". Each time the user clicks on a new marker, this code closes the sidebar if it was already open, erases the old details, adds the new details, and opens the sidebar.</p>
<p>Add the following below the <code>InfoWindow</code> that you just added:</p>

```js
   // Displays place details in a sidebar
    function showPanel(placeResult) {
      // If infoPane is already open, close it
      if (infoPane.classList.contains("open")) {
        infoPane.classList.remove("open");
      }

      // Clear the previous details
      while (infoPane.lastChild) {
        infoPane.removeChild(infoPane.lastChild);
      }

      // Add place details with text formatting
      let name = document.createElement('h1');
      name.classList.add('place');
      name.textContent = placeResult.name;
      infoPane.appendChild(name);
      if (placeResult.rating != null) {
        let rating = document.createElement('p');
        rating.classList.add('details');
        rating.textContent = `Rating: ${placeResult.rating} \u272e`;
        infoPane.appendChild(rating);
      }
      let address = document.createElement('p');
      address.classList.add('details');
      address.textContent = placeResult.formatted_address;
      infoPane.appendChild(address);
      if (placeResult.website) {
        let websitePara = document.createElement('p');
        let websiteLink = document.createElement('a');
        let websiteUrl = document.createTextNode(placeResult.website);
        websiteLink.appendChild(websiteUrl);
        websiteLink.title = placeResult.website;
        websiteLink.href = placeResult.website;
        websitePara.appendChild(websiteLink);
        infoPane.appendChild(websitePara);
      }

      // Open the infoPane
      infoPane.classList.add("open");
    }
```

<h3><strong>Display a Place Photo with the Place Details</strong></h3>
<p>The <code>getDetails</code> result returns an array of up to 10 photos associated with the <code>placeId</code>.</p>
<p>Here, we display the first photo above the place name in the sidebar.</p>
<p>Place this code above creation of the "name" element if you want the photo to appear at the top of the sidebar.</p>

```js
     // Add the primary photo, if there is one
      if (placeResult.photos != null) {
        let firstPhoto = placeResult.photos[0];
        let photo = document.createElement('img');
        photo.classList.add('hero');
        photo.src = firstPhoto.getUrl();
        infoPane.appendChild(photo);
      }
```

<h3>Test it out</h3>
<ul>
<li>Run <code>gcloud app deploy</code> in your Cloud Shell.</li>
<li>Reload the page in your browser and allow geolocation permissions.</li>
<li>Click on a marker to see the InfoWindow pop up from the marker displaying a few details, and the sidebar slide out from the left to display more details.</li>
<li>Test whether the search also works if you reload and deny geolocation permissions.</li>
<li>Edit your search keyword for a different query and explore the result returned for that search.</li>
</ul>
<p>Click <em>Check my progress</em> to verify the objective.
<ql-activity-tracking step="3">
Deploy the updated app engine application
</ql-activity-tracking></p>
<h4>Full sample code</h4>
<p>The full code for this project through Step 4 is  <a href="https://github.com/googlecodelabs/google-maps-nearby-search-js/blob/master/step4/index.html">available on Github</a>.</p>
<h2 id="step7">Congratulations</h2>
<p>Congratulations! You've used many features of the Google Maps JavaScript API, including the Places Library. Here are some of the API features you learned about:</p>
<ul>
<li>Creating a map with the  <a href="https://developers.google.com/maps/documentation/javascript/reference/map">google.maps.Map</a> class</li>
<li>Using the user's browser for  <a href="https://developers.google.com/maps/documentation/javascript/geolocation">geolocation</a> and displaying the results on a map</li>
<li>Adding  <a href="https://developers.google.com/maps/documentation/javascript/markers">markers</a> to your map and  <a href="https://developers.google.com/maps/documentation/javascript/events">responding to user click events</a> on them</li>
<li>Adding  <a href="https://developers.google.com/maps/documentation/javascript/infowindows">info windows</a> to display more information when a user clicks a marker</li>
<li>Loading the  <a href="https://developers.google.com/maps/documentation/javascript/places">Places Library</a> and performing a  <a href="https://developers.google.com/maps/documentation/javascript/places#place_search_requests">Nearby Search</a>
</li>
<li>Fetching and displaying  <a href="https://developers.google.com/maps/documentation/javascript/places#place_details">place details</a> and  <a href="https://developers.google.com/maps/documentation/javascript/places#places_photos">place photos</a>
</li>
</ul>

<p>This self-paced lab is part of the <a href="https://google.qwiklabs.com/quests/103">Creating with Google Maps</a> Quest. A Quest is a series of related labs that form a learning path. Completing this Quest earns you the badge above, to recognize your achievement. You can make your badge public and link to it in your online resume or social media account. <a href="https://google.qwiklabs.com/quests/103/enroll">Enroll in this Quest</a> and get immediate completion credit if you've taken this lab.  <a href="https://google.qwiklabs.com/catalog">See other available Qwiklabs Quests</a>.</p>
<h3>Take your next lab</h3>
<p>Continue your Quest with <a href="https://google.qwiklabs.com/catalog_lab/1118">Google Maps Web Services Proxy for Mobile Applications</a>, or check out these suggestions:</p>
<ul>
<li>
<p><a href="https://google.qwiklabs.com/catalog_lab/366">Mapping the NYC Subway</a></p>
</li>
<li>
<p><a href="https://google.qwiklabs.com/catalog_lab/1478">Google Apps Script: Access Google Sheets, Maps &amp; Gmail in 4 Lines of Code</a></p>
</li>
</ul>
<h3>Next steps</h3>
<p>To do even more with maps, explore the  <a href="https://developers.google.com/maps/documentation/javascript/tutorial">Maps JavaScript API documentation</a> and  <a href="https://developers.google.com/maps/documentation/javascript/places">Places Library documentation</a> which contain guides, tutorials, the API reference, more code samples, and support channels. Some popular features are  <a href="https://developers.google.com/maps/documentation/javascript/importing_data">Importing Data into Maps</a>,  <a href="https://developers.google.com/maps/documentation/javascript/styling">Styling Your Map</a>, and adding the  <a href="https://developers.google.com/maps/documentation/javascript/streetview">Street View Service</a>.</p>
<h3>Resources</h3>
<ul>
<li><a href="https://cloud.google.com/monitoring/kubernetes-engine/customizing">Customizing Kubernetes Monitoring</a></li>
<li><a href="https://cloud.google.com/kubernetes-engine/docs/tutorials/installing-istio">Installing Istio on GKE</a></li>
<li><a href="https://cloud.google.com/monitoring/workspaces/">Stackdriver Workspaces</a></li>
<li><a href="https://cloud.google.com/kubernetes-monitoring/">Kubernetes Monitoring in Stackdriver</a></li>
<li><a href="https://landing.google.com/sre/sre-book/toc/index.html">The Google SRE book</a></li>
<li><a href="https://landing.google.com/sre/sre-book/chapters/service-level-objectives/">Service Level Objectives</a></li>
</ul>

# Google Maps Web Services Proxy for Mobile Applications

<h2 id="step2">Introduction</h2>
<p>Suppose you want to create an augmented reality style mobile game, where users must visit real-world locations to progress through the game. Given that a mobile device has access to its current location, the game can randomly generate each location the user needs to visit, but how do you know these locations are accessible? A randomly generated location could very well be in the ocean, or some other inaccessible area.</p>
<p>What you need is a way to identify real-world locations that your game can randomly offer as destinations for the game. The  <a href="https://developers.google.com/places/web-service/">Google Places API</a> is a perfect fit for this, as it allows you to search for places within a particular radius at a given location. Given that the user's mobile device knows its current location, you can use the Google Places API Web Service to search for nearby places and offer these as destinations the player must reach.</p>
<p>Using the Google Places API directly from a mobile device presents some interesting problems in terms of ensuring API key security, and optimising network performance. You will address these issues by building a server-side proxy using  <a href="https://golang.org/">Golang</a> and  <a href="https://cloud.google.com/appengine/">Google App Engine</a>. The proxy will take requests from the mobile devices, and make requests to the Google Places API on its behalf.</p>
<h3><strong>What you'll build</strong></h3>
<p>In this lab you'll build a Google App Engine proxy for the Google Places API web service, which will:</p>
<ul>
<li>
<p>Protect your Google Maps API key from being directly exposed to a mobile application.</p>
</li>
<li>
<p>Reduce network traffic by removing extraneous data provided by the Google Places API before returning it to the mobile application.</p>
</li>
<li>
<p>Cache data provided by the Google Places API to improve network performance, while still complying with the  <a href="https://developers.google.com/places/web-service/policies#pre-fetching_caching_or_storage_of_content">Google Places API Policies</a>.</p>
</li>
</ul>
<h3><strong>What you'll learn</strong></h3>
<ul>
<li>
<p>How to use  <a href="https://cloud.google.com/">Google Cloud</a> services like  <a href="https://cloud.google.com/shell/">Google Cloud Shell</a> and  <a href="https://cloud.google.com/appengine/">Google App Engine</a>.</p>
</li>
<li>
<p>How to use  <a href="https://developers.google.com/maps/web-services/">Google Maps APIs Web Services</a> safely.</p>
</li>
</ul>
<h3><strong>What you'll need</strong></h3>
<ul>
<li>
<p>Basic knowledge of  <a href="https://golang.org/">Golang</a>.</p>
</li>
</ul>
<h2 id="step3">Setup</h2>

<h3>Launch Google Cloud Shell Code Editor</h3>
<p>Use the Google Cloud Shell Code Editor to easily create and edit directories and files in the Cloud Shell instance.</p>
<p>Click the <strong>Launch the code editor</strong> button (looks like a pencil) to launch the the Cloud Shell Code Editor.</p>
<p><img alt="17dd18646253c6db.png" src="https://cdn.qwiklabs.com/6mF0wyFlQaN%2BAwf6dcG3kHvircvLg%2Bvdto%2ByvHso9B0%3D"></p>
<h2 id="step4">Hello World, in Go</h2>
<h3><strong>Start developing with Hello World</strong></h3>
<p>Create a Go App Engine app to serve as the basis for the rest of the lab.</p>
<p>In the Cloud Shell Code Editor, create a new <code>places-proxy</code> directory for your application by clicking <strong>File</strong> &gt; <strong>New Folder</strong>.</p>
<p><img alt="a837fcd55e1b586e.png" src="https://cdn.qwiklabs.com/Jq2nKl219L4x9loF3RV56AGUgO8yOO6S9wgviict5Nc%3D"></p>
<p>Name the new folder <code>places-proxy</code>.</p>
<p>Next, create a small Go App Engine app to make sure everything's working.</p>
<p>In the <code>places-proxy</code> directory create a new file by selecting <strong>File</strong> &gt; <strong>New File</strong>. Name the new file <code>app.yaml</code>. Put the following content in the <code>app.yaml</code> file:</p>

```yaml
runtime: go
api_version: go1

handlers:
- url: /.*
  script: _go_app
  login: admin
```

<p><strong>Save</strong> the file.</p>
<p>This configuration file configures your App Engine app to use the Golang Standard run time. For background information on the meaning of the configuration items in this file, see the  <a href="https://cloud.google.com/appengine/docs/go/">Google App Engine Go Standard Environment Documentation</a>.</p>
<aside class="warning"><p><strong>Important:</strong> In the above app.yaml file, you'll see we include the line <code>login: admin</code> - this ensures that only you will be able to access the application we're about to build. In a real-world scenario however, your application would require an authentication implementation suitable for clients (other than yourself) to consume, such as <a href="https://developers.google.com/identity/protocols/OAuth2" target="blank">OAuth</a>. </p>
</aside>
<p>In the <code>places-proxy</code> directory create a new file and name the new file <code>proxy.go</code>. Put the following content in the file:</p>

```go
package proxy

import (
    "fmt"
    "net/http"
)

func init() {
    http.HandleFunc("/", handler)
}

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "Hello, world!")
}
```

<p>At a high level, here is what this code does. You have defined a package <code>proxy</code> that registers a handler function for HTTP requests matching the path  <code>"/"</code> in the <code>init</code> function, which is called by the Golang App Engine runtime at startup.</p>
<p>The handler function, handily called <code>handler</code>, simply writes out the text string <code>"Hello, world!"</code> when called. This text will be relayed back to your browser, where you will be able to read it. In future steps you'll extend this further into your Places API proxy.</p>
<p>To test this application, run the App Engine development server inside the Cloud Shell instance. Go to the Cloud Shell command line, and type the following:</p>
<pre><code class="language-bash prettyprint">cd places-proxy&#x000A;</code></pre>
<pre><code class="language-bash prettyprint">goapp serve&#x000A;</code></pre>
<p>You will see some lines of log output showing that you are indeed running the development server on the Cloud Shell instance, with the hello world web app listening on localhost port 8080.</p>
<p>Click on <strong>Preview on port 8080</strong> (on the right, above the code editor) to open a new tab in your web browser.</p>
<p><img alt="a808a16a82ea7537.png" src="https://cdn.qwiklabs.com/oPkku4jlT%2Bt9JHo%2F2uudEu4fERQUM8KynW6Q5UR9B1w%3D"></p>
<p>Log in as Administrator with the username assigned to you at the start of this lab.</p>
<p><img alt="dcbb75c1d92a15aa.png" src="https://cdn.qwiklabs.com/Z1CT0sUVYh1Jc1fyco2XTnv5uoyfznUD3PIC2x5nlO0%3D"></p>
<p>After you are validated, the  the App Engine development server serves you the "Hello, world!" message.</p>
<p><img alt="d4d2b6564329d560.png" src="https://cdn.qwiklabs.com/3hiQAkcxLWi2OU70fcdmVepZCpO%2FRlooWWAXek5yIyI%3D"></p>
<aside class="special"><p><strong>Tip:</strong> Leave this browser window open, as you'll keep returning to it while you develop the proxy.</p>
</aside>
<p>In the next step you'll extend this code to make requests to the Google Places API and return the results back to the browser, forming the basis of your proxy.</p>
<h2 id="step5">A minimal proxy</h2>
<p>Your next step is to create a basic working proxy that makes requests to the Google Places API and returns the data to the browser (or mobile application) that made the request to the proxy. With this in place, you will have implicitly achieved one of the goals of this lab, which is to prevent the Google Maps API key from being exposed directly to the mobile application.</p>
<p>API keys for Google Maps Web Services are intended for use only in server-side applications, and are not restricted to specific websites or mobile applications. Embedding these keys directly into a mobile application could give a malicious user the opportunity to extract the API key and abuse it. When you communicate with the Google Maps Web Services through a proxy, you can protect the API key by embedding it within the proxy server.</p>
<p>To do this, you'll edit <code>proxy.go</code> again to do the following:</p>
<ul>
<li>Define a constant, <code>placesAPIKey</code>, to contain your Google Maps API key.</li>
<li>Define another constant, <code>placesURL</code> , to contain the URL for the Google Places API Web Service, with placeholders for each of the variable parts of it, namely the API key, and the location and radius we want to use to search for nearby places.</li>
<li>Define a function called <code>fetchPlaces</code> which is responsible for making the request to the Google Places API.</li>
<li>Modify the <code>handler</code> function to call the <code>fetchPlaces</code> function and return the results to the browser.</li>
</ul>
<p>Open <code>proxy.go</code> and delete the contents of the file. Add the edited commands below to the <code>proxy.go</code> file:</p>

```go
package proxy

import (
    "appengine"
    "appengine/urlfetch"
    "fmt"
    "io/ioutil"
    "net/http"
)

// Define constants for the API key and the web service URL.
const placesAPIKey = "YOUR_API_KEY"
const placesURL = "https://maps.googleapis.com/maps/api/place/nearbysearch/json?key=%s&location=%s&radius=%s"

// Makes the request to the Places API.
func fetchPlaces(ctx appengine.Context, location, radius string) ([]byte, error) {
    client := urlfetch.Client(ctx)
    resp, err := client.Get(fmt.Sprintf(placesURL, placesAPIKey, location, radius))
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    return ioutil.ReadAll(resp.Body)
}

// Calls the fetchPlaces function and returns the results to the browser.
func handler(w http.ResponseWriter, r *http.Request) {
    ctx := appengine.NewContext(r)
    places, err := fetchPlaces(ctx, r.FormValue("location"), r.FormValue("radius"))
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    w.Header().Add("Content-Type", "application/json; charset=utf-8")
    w.Write(places)
}

func init() {
    http.HandleFunc("/", handler)
}
```

<p>You need to make one quick diversion to replace the <code>YOUR_API_KEY</code> in the above source with an actual Google Maps API key. Follow these steps to get an API key:</p>
<ol>
<li>
<p>Open the  <a href="https://console.developers.google.com/flows/enableapi?apiid=places_backend&amp;reusekey=true">Google API Console</a> in a new browser. Use this link, because it has some smarts to enable the correct APIs for you.</p>
<p><img alt="b114bdd202f2ea7f.png" src="https://cdn.qwiklabs.com/H7uXiFz9GUc8WVlSesUr%2BXLk4RK3ECcuZgqLi2Vof%2FY%3D"></p>
</li>
<li>
<p>Use the dropdown to select GCP Project ID for this lab and then click <strong>Continue</strong> to enable the API and any related services.</p>
<p><img alt="3bbe6d20163a8312.png" src="https://cdn.qwiklabs.com/0WTTNcfroPdB9tlzDk4Ti%2FU4by8f09WTQqHle%2B2xoKk%3D"></p>
</li>
<li>
<p>On the Credentials page, click the  <strong>API key</strong> link. Don't worry about adding restrictions to this API Key for the purposes of this lab.</p>
<p><img alt="288ed2487a1f54b7.png" src="https://cdn.qwiklabs.com/Kd7%2B5PwVIVh64TptKEvHozrcs%2BdlBEkFy%2BUYHqIU7w4%3D"></p>
</li>
<li>
<p>Name the API key then click <strong>Create</strong>.</p>
<p><img alt="30325badead27efc.png" src="https://cdn.qwiklabs.com/TOkPA3BM4oWtnLTEMJb0CeUahnPNuGkp4qf%2FVVsrN%2BM%3D"></p>
</li>
<li>
<p>An API key created window displays. Copy the API key.</p>
</li>
<li>
<p>In the <code>proxy.go</code> file replace <code>YOUR_API_KEY</code> with it (leaving the quotes around it).</p>
</li>
<li>
<p><strong>Save</strong> the file.</p>
</li>
</ol>
<aside><b>Note:</b> Close the API key created window but keep the Credentials window open in the browser as you'll need to copy and paste your API key later in this lab.
</aside>
<p>Back in Cloud Shell, restart your application by pressing <strong>Ctrl</strong>+<strong>C</strong> to stop it, then run:</p>
<pre><code>goapp serve&#x000A;</code></pre>
<p>You'll need to restart the application each time you make a change to the source code.</p>
<aside class="special"><p><strong>Tip: </strong>You can actually press the up arrow to go back through the shell's history, rather than typing a command again!</p>
</aside>
<p>Once the application has restarted, go back to the browser window that originally displayed the "Hello, World!" message (or use the web preview again to open a new browser window), and reload the page. You should see the following:</p>
<p><img alt="b30f48872199997.png" src="https://cdn.qwiklabs.com/rOq4N5dsdoXG9BwcGtibyOVN1H1qFV9xwZNfc2SAQKs%3D"></p>
<p>What's happening here? The good news is you're displaying exactly the response the Google Places API is sending you - proxy success! The bad news is that the response is an error. You haven't specified a location and radius to use in the search. If you study the <code>handler</code> function, you'll see it expects the location and radius to be specified as part of the URL used to request the proxy, and it then passes those on to the Places API as arguments to the <code>fetchPlaces</code> function.</p>
<p>Go back to the browser window showing the error, and append a location and radius to it. Use your current location if you know it! If you don't, just copy everything after <code>appspot.com</code> and append your own URL:</p>
<pre><code>https://8080-dot-9570539-dot-devshell.appspot.com/?authuser=0&amp;environment_name=default&amp;location=-33.8594663,151.1927602&amp;radius=500&#x000A;</code></pre>
<aside class="special"><p><strong>Note: </strong>The appended part in the above URL is <strong><code>&amp;location=-33.8594663,151.1927602&amp;radius=500</code></strong>. Your URL will be slightly different, so don't just copy the one above entirely, use it as a guide.</p>
</aside>
<p>Now you should see lots of data for nearby places, returned by the Google Places API:</p>
<p><img alt="24471171252a75a8.png" src="https://cdn.qwiklabs.com/ySoRkydNmp4Pva07aMa0DgwpBdxzU0FvJoQvPhOEqSU%3D"></p>
<h2 id="step6">Stripping down the data</h2>
<p>As you can see from the output of your proxy, the Google Places API returns an extensive amount of data for each place in the response, including photos, opening hours, classifications, and more. Our supposed augmented reality game doesn't need any of this information, as it's only interested in the latitude and longitude coordinates of the locations. Your next step will be to enhance the proxy, by having it strip out the extraneous data provided by the Google Places API, so that it returns just the location coordinates to the mobile device. Reducing the amount of data being sent to the mobile application is an important consideration, given the inconsistent capacity of mobile networks.</p>
<p>Edit <code>proxy.go</code> again. The code remains mostly intact, with some additions:</p>
<ul>
<li>Define a type called <code>placeResults</code> which defines the structure of the JSON returned from the Google Places API using types provide by Golang.</li>
<li>Define a function called <code>formatPlaces</code> which takes the JSON response from the Google Places API,  and converts it into a variable of type <code>placeResult.</code>
</li>
<li>In the original <code>handler</code> function, change the last line to call <code>formatPlaces</code> and return its value, rather than directly returning the response from the Google Places API.</li>
</ul>
<p>Edit <code>proxy.go</code> and delete the contents, then copy the edited commands below into the file. Remember to replace <code>YOUR_API_KEY</code> with your own API key.</p>

```go
package proxy

import (
    "appengine"
    "appengine/urlfetch"
    "encoding/json"
    "fmt"
    "net/http"
    "io"
)

const placesAPIKey = "YOUR_API_KEY"
const placesURL = "https://maps.googleapis.com/maps/api/place/nearbysearch/json?key=%s&location=%s&radius=%s"

// Represents the structure of the Places API JSON response.
type placeResults struct {
    Results []struct {
        Geometry struct {
            Location struct {
                Lat float64 `json:"lat"`
                Lng float64 `json:"lng"`
            } `json:"location"`
        } `json:"geometry"`
    } `json:"results"`
}

// Takes the JSON response from the Google Places API, and converts it into a variable of type placeResult
func formatPlaces(body io.Reader)([]byte, error) {
    var places placeResults
    if err := json.NewDecoder(body).Decode(&places); err != nil {
        return nil, err
    }
    return json.Marshal(places)
}

func fetchPlaces(ctx appengine.Context, location, radius string) ([]byte, error) {
    client := urlfetch.Client(ctx)
    resp, err := client.Get(fmt.Sprintf(placesURL, placesAPIKey, location, radius))
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    // Change the return value to use the new formatPlaces function.
    return formatPlaces(resp.Body)
}

func handler(w http.ResponseWriter, r *http.Request) {
    ctx := appengine.NewContext(r)
    places, err := fetchPlaces(ctx, r.FormValue("location"), r.FormValue("radius"))
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    w.Header().Add("Content-Type", "application/json; charset=utf-8")
    w.Write(places)
}

func init() {
    http.HandleFunc("/", handler)
}
```

<p>In Cloud Shell, press <strong>Ctrl</strong>+<strong>C</strong> to stop the application, then restart by entering:</p>
<pre><code>goapp serve&#x000A;</code></pre>
<p>Reload the browser window with the URL that contains the location and radius parameters. You should see a much lighter version of the data returned from the Google Places API:</p>
<p><img alt="a5e873ba8a9a071c.png" src="https://cdn.qwiklabs.com/QhWlAJJp41lwhw9rI9L01L3o4X7b2VjkafMD8XRjA7Q%3D"></p>
<p>Magic! But how did it happen? You took advantage of the way Golang converts strings of JSON into typed data. If you define only the fields in the JSON that you're interested in, such as  <code>lat</code> and <code>lng,</code> Golang conveniently ignores all the other data. The <code>formatPlaces</code> function simply converts a JSON string into a typed variable, and back into a JSON string again, all for the sake of stripping down the data.</p>
<h2 id="step7">Caching responses</h2>
<p>Suppose several people near each other are playing your augmented reality game. The data returned from the Google Places API will be very similar, if not identical, for each player. It therefore makes sense for your proxy to store a copy of the places returned for a given location in a cache that can be used to retrieve places for subsequent requests for the same (or a very nearby) location. This significantly speeds up your proxy server.</p>
<p>Google App Engine gives you access to Memcache, a fast caching service and purposely suited to this type of task. Memcache allows you to store values by a given key, and then retrieve them again using that same key. For caching places by location, you'll use the location as the key for storing the response from the Google Places API. This presents a slight problem though, as most locations will be unique, since an increasing number of decimal places in a latitude and longitude increases precision. For example, a latitude or longitude with 9 decimal places indicates a degree of accuracy down to 110 microns, which is about the width of two human hairs, while 4 decimal places indicates a lesser degree of accuracy of around 11 metres. If each request to your proxy used a latitude and longitude with 9 decimal places, every location would be entirely unique, and looking up your cached responses would never retrieve a value. To address this problem, round off the latitude and longitude in each request's location to 4 decimal places, and use the rounded location as your cache key.</p>
<p>Now you'll edit <code>proxy.go</code> to do the following:</p>
<ul>
<li>Define a couple of new constants:  <code>locationPrecision,</code> which defines the rounding precision for locations by as described above, and <code>cacheExpiry</code> which defines how long to store results in the cache. You can tune each of these to increase the frequency with which the cache is used.</li>
<li>Define a function called <code>normalizeLocation.</code> This performs the rounding of the latitude and longitude given for each request.</li>
<li>Modify the <code>fetchPlaces</code> function to store places in the cache after they are retrieved from the Google Places API.</li>
<li>Modify the <code>handler</code> function to first check for places in the cache before requesting them via <code>fetchPlaces</code>. If places are found in the cache, we will also send a background request to <code>fetchPlaces,</code> which ensures that both the cache remains fresh, and that we also remain in compliance with the  <a href="https://developers.google.com/places/web-service/policies#pre-fetching_caching_or_storage_of_content">Google Places API Policies</a>.</li>
</ul>
<p>Edit <code>proxy.go</code> by deleting the contents then adding the edited commands below. Remember to replace <code>YOUR_API_KEY</code> with your own API key.</p>

```go
package proxy

import (
    "appengine"
    "appengine/memcache"
    "appengine/urlfetch"
    "encoding/json"
    "errors"
    "fmt"
    "net/http"
    "strconv"
    "strings"
    "time"
    "io"
)

// New constants for location rounding and cache expiry.
const locationPrecision = 4  // Up to 11 metres
const cacheExpiry = time.Second * 600  // 5 minutes
const placesAPIKey = "YOUR_API_KEY"
const placesURL = "https://maps.googleapis.com/maps/api/place/nearbysearch/json?key=%s&location=%s&radius=%s"

type placeResults struct {
    Results []struct {
        Geometry struct {
            Location struct {
                Lat float64 `json:"lat"`
                Lng float64 `json:"lng"`
            } `json:"location"`
        } `json:"geometry"`
    } `json:"results"`
}

// Rounds off the latitude and longitude of a location.
func normalizeLocation(location string) (string, error) {
    var lat, lng float64
    var err error
    latLng := strings.Split(location, ",")
    if len(latLng) != 2 {
        return "", errors.New("Invalid location")
    }
    if lat, err = strconv.ParseFloat(latLng[0], locationPrecision); err != nil {
        return "", errors.New("Invalid location")
    }
    if lng, err = strconv.ParseFloat(latLng[1], locationPrecision); err != nil {
        return "", errors.New("Invalid location")
    }
    return fmt.Sprintf("%.2f,%.2f", lat, lng), nil
}

func formatPlaces(body io.Reader)([]byte, error) {
    var places placeResults
    if err := json.NewDecoder(body).Decode(&places); err != nil {
        return nil, err
    }
    return json.Marshal(places)
}

// fetchPlaces now stores results in the cache.
func fetchPlaces(ctx appengine.Context, location, radius string) ([]byte, error) {
    client := urlfetch.Client(ctx)
    resp, err := client.Get(fmt.Sprintf(placesURL, placesAPIKey, location, radius))
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    places, err := formatPlaces(resp.Body)
    if (err == nil) {
        memcache.Set(ctx, &memcache.Item{
            Key:        location,
            Value:      places,
            Expiration: cacheExpiry,
        })
    }
    return places, err
}

// handler now retrieves results from the cache if they exist.
func handler(w http.ResponseWriter, r *http.Request) {
    radius := r.FormValue("radius")
    location, err := normalizeLocation(r.FormValue("location"))
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    ctx := appengine.NewContext(r)
    var places []byte
    if cached, err := memcache.Get(ctx, location); err == nil {
        places = cached.Value
        // We use Golang's goroutines here to call fetchPlaces in the background,
        // without having to wait for the result it returns. This ensures that
        // both the cache remains fresh, and that we also remain in compliance
        // with the Google Places API Policies.
        go fetchPlaces(ctx, location, radius)
    } else if places, err = fetchPlaces(ctx, location, radius); err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    w.Header().Add("Content-Type", "application/json; charset=utf-8")
    w.Write(places)
}

func init() {
    http.HandleFunc("/", handler)
}
```

<p>In the Cloud Shell command line, restart the application, and then reload the browser window with the URL that includes the location and radius parameters. You should see exactly the same results when you change the digits in the 5th or 6th decimal places of the latitude and longitude in the URL.</p>


<h2 id="step8">Congratulations!</h2>
<p>Congratulations, you have successfully completed this lab!</p>

<p>A Quest is a series of related labs that form a learning path. This self-paced lab is part of the Qwiklabs <a href="https://google.qwiklabs.com/quests/39">Website and Web Application</a> and <a href="https://google.qwiklabs.com/quests/103">Creating with Google Maps</a> Quest. Completing this Quest earns you the badge above, to recognize your achievement. You can make your badges public and link to them in your online resume or social media account. <a href="https://google.qwiklabs.com/quests/39">Enroll in this Quest</a> and get immediate completion credit if you've taken this lab.  <a href="https://google.qwiklabs.com/catalog">See other available Qwiklabs Quests</a>.</p>
<h3>Take your next lab</h3>
<p>Continue your Quest with <a href="https://google.qwiklabs.com/catalog_lab/366">Mapping the NYC Subway</a>, or try one of these suggestions:</p>
<ul>
<li>
<p><a href="https://google.qwiklabs.com/catalog_lab/1117">Using Ruby on Rails with Cloud SQL for PostgreSQL</a></p>
</li>
<li>
<p><a href="https://google.qwiklabs.com/catalog_lab/359">Speaking with a Webpage - Streaming Speech Transcripts</a></p>
</li>
</ul>
<h3>Next steps/learn more</h3>
<p>If you enjoyed this lab, but would like to dive into the code some more, have a look at the source code repository at  <a href="https://github.com/googlecodelabs/google-maps-web-services-proxy">https://github.com/googlelabs/google-maps-web-services-proxy</a>.</p>
<p>Or, if you're into maps, check out the <a href="https://cloud.google.com/customers/map-of-life/">Map Of Life</a>.</p>

# Mapping the NYC Subway

<h2 id="step2">Overview</h2>
<p>Store locators are an often required component of any commercial website that wants to encourage users to visit brick and mortar stores. In this lab you'll build a website that serves location data of subway stations and scales automatically, thanks to the power of Google App Engine. You will learn how to expose an easily navigable visualization of the NYC subway using Google Maps JavaScript API.</p>
<p>NYC Open Data makes the wealth of public data generated by various New York City agencies and other City organizations available for public use. In this lab, you'll visualize the Subway Station and Subway Line data sets.</p>
<h3><strong>What you'll learn</strong></h3>
<ul>
<li>
<p>How to use  <a href="https://cloud.google.com/">Google Cloud</a> services like  <a href="https://cloud.google.com/shell/">Google Cloud Shell</a> and  <a href="https://cloud.google.com/appengine/">Google App Engine</a></p>
</li>
<li>
<p>How to serve geographic information system (<a href="https://en.wikipedia.org/wiki/Geographic_information_system">GIS data</a> on the web in an effortlessly scalable fashion</p>
</li>
<li>
<p>How to style a Google map in your app</p>
</li>
</ul>
<h3><strong>What you'll build</strong></h3>
<p>In this lab, you'll build a visualization for the NYC Subway Station dataset that is:</p>
<table>
<tr>
<td colspan="1" rowspan="1">
<ul>
<li>
<strong>Scalable</strong> - you'll use Google App Engine to automatically scale your serving capability to match request load.</li>
<li>
<strong>Maintainable</strong> - you'll use the Go programming language to make the back end code simple, reliable, and efficient.</li>
<li>
<strong>Customizable</strong> - you'll style the map using the <a href="https://developers.google.com/maps/documentation/javascript/styling">styling capability</a> of the Google Maps JavaScript API, with the help of <a href="https://mapstyle.withgoogle.com">Google Maps Style Wizard</a>.</li>
</ul>
</td>
<td colspan="1" rowspan="1">
<img alt="map_preview" src="https://cdn.qwiklabs.com/HwLleeFCdVoaK%2B5gp6Odlm2gaArTFbUToNRclSg62vU%3D">
</td>
</tr>

<h2 id="step4">Enable the Google Maps JavaScript API</h2>
<p>Click on the menu icon in the top left of the screen.</p>
<p><img alt="burger_menu.png" src="https://cdn.qwiklabs.com/DuEdLz3bS6dSpKkxwiK7pbISMK%2B3inPOStiVq8oKRig%3D"></p>
<p>Select <strong>APIs &amp; services</strong> from the drop down and click on "Dashboard".</p>
<p><img alt="api_nav.png" src="https://cdn.qwiklabs.com/WQJmwscI63oS33XFMAfnfi8KoO8ewLpDH4A8ZsLKufM%3D"></p>
<p>Click on <strong>Enable APIs and services</strong>.</p>
<p><img alt="api_dashboard.png" src="https://cdn.qwiklabs.com/Yo%2BXa10XwmyZ1SXn%2FsTup71Oor3HahGd9UnbdYcAdec%3D"></p>
<p>Then, search for <code>Maps JavaScript API</code> in the search box.</p>
<p>Click on <strong>Maps JavaScript API</strong>, then click <strong>Enable</strong>.</p>

<p>Click <em>Check my progress</em> to verify the objective.
<ql-activity-tracking step="1">
Enable the Google Maps JavaScript API
</ql-activity-tracking></p>
<h3><strong>Code Editor</strong></h3>
<p>In this lab you will add and edit files. You can use your choice of editor, (<code>vim</code>, <code>nano</code>, <code>emacs</code>) or the Cloud Shell Code Editor. Instructions in this lab use the Code Editor.</p>
<p>Open the Code Editor by clicking the pencil icon.</p>
<p><img alt="pencil.png" src="https://cdn.qwiklabs.com/VOe8Zmi4cS8ZES%2Bm7bwAQ7uax81a0mprJeuoqfzTK1g%3D"></p>
<h2 id="step5">Hello World, in Go</h2>
<p>You will start by creating a Go App Engine app that will serve as the basis for the rest of the lab.</p>
<p>In the Code Editor, click <strong>File</strong> &gt; <strong>New Folder</strong>.</p>
<p><img alt="cloud_shell_folder.png" src="https://cdn.qwiklabs.com/a60DtqZvnhFUY14BBYJbPKMSMWnWMv5lEU7u2On%2FfiI%3D"></p>
<p>Name the new folder <code>nyc-subway</code>.</p>
<p>Create a file in the <code>nyc-subway</code> directory named <code>app.yaml</code> and enter the following content:</p>
<pre><code>runtime: go&#x000A;api_version: go1&#x000A;&#x000A;handlers:&#x000A;- url: /.*&#x000A;  script: _go_app&#x000A;</code></pre>
<p>This configuration file tells your App Engine app to use the Go standard runtime. For background information on the configuration items in this file, see the  <a href="https://cloud.google.com/appengine/docs/go/">Google App Engine Go Standard Environment Documentation</a>.</p>
<p>Save the file.</p>
<p>Next, create a <code>hello.go</code> file in the <code>nyc-subway</code> directory, and add the following to the file:</p>

```go
package hello

import (
    "fmt"
    "net/http"
)

func init() {
    http.HandleFunc("/", handler)
}

func handler(w http.ResponseWriter, r *http.Request) {
        // Writes Hello, World! to the user's web browser via `w`
        fmt.Fprint(w, "Hello, world!")
}
```

<p>It is worth pausing a moment here to understand what this code does, at least at a high level. You have defined a package <code>hello</code> that registers a handler function for HTTP requests matching the path  <code>"/"</code> in the <code>init</code> function, which is called by the Go App Engine runtime at startup.</p>
<p>The handler function, handily called <code>handler</code>, simply writes out the text string <code>"Hello, world!"</code>. This text will be relayed back to your browser, where you'll be able to read it. In future sections you'll make handlers that respond with GeoJSON data instead of simple hard coded strings.</p>
<p>After carrying out these steps, you should now have an editor that looks like this:</p>
<p><img alt="hello_file_contents.png" src="https://cdn.qwiklabs.com/6RSlZQjX7S8%2Fx%2BzEQs8ISV5eqVMc7ob0gOFsAYsodm8%3D"></p>
<p>To test this application, run the App Engine development server inside the Cloud Shell instance.</p>
<p>In Cloud Shell, run the following:</p>
<pre><code>gcloud config set project &lt;PROJECT_ID&gt;&#x000A;</code></pre>
<pre><code>cd nyc-subway &amp;&amp; goapp serve&#x000A;</code></pre>
<p>You will see some lines of log output showing you that you are indeed running the development server on the Cloud Shell instance, with the hello world web app listening on localhost port 8080.</p>
<p>Open a web browser tab on this app by clicking the <strong>Web preview</strong> icon and selecting <strong>Preview on port 8080</strong>.</p>
<p><img alt="web_prev_menu.png" src="https://cdn.qwiklabs.com/Fjpmg7Zc4eTx3AhgePysAOr00y8DapxewhqCXVEucPc%3D"></p>
<p>A new tab opens in your web browser, which displays content served from the App Engine development server.</p>
<p>Shut down the server by typing <code>[CTRL] + [C]</code>,</p>
<p>In the next section you'll add the NYC Subway data to this app, and start visualizing it.</p>
<h2 id="step6">NYC Subway data</h2>
<h3>GeoJSON, the <em>lingua franca</em> of the GIS world</h3>
<p>As mentioned in the previous section, you'll make handlers in your Go code that render GeoJSON data to the web browser. But what is GeoJSON?</p>
<p>In the  <a href="https://en.wikipedia.org/wiki/Geographic_information_system">Geographic Information System</a> (GIS) world, you need to communicate knowledge about geographic entities between computer systems. Maps are great for humans to read, but computers typically prefer their data in more easily digested formats.</p>
<p><a href="http://geojson.org/">GeoJSON</a> is a format for encoding geographic data structures, like the Subway Stations in New York City. GeoJSON has been standardized in an  <a href="http://ietf.org/">Internet Engineering Task Force</a> standard called  <a href="https://tools.ietf.org/html/rfc7946">RFC7946</a>. GeoJSON is defined in terms of  <a href="http://www.json.org/">JSON</a>, JavaScript Object Notation, which itself was standardised in  <a href="http://www.ecma-international.org/publications/files/ECMA-ST/ECMA-404.pdf">ECMA-404</a>, by the same organization that standardized JavaScript,  <a href="http://www.ecma-international.org/">Ecma International</a>.</p>
<p>The important thing is that GeoJSON is a widely supported wire format for communicating geographic knowledge. This lab uses GeoJSON in the following ways:</p>
<ul>
<li>Use Go packages to parse the NYC data into an internal GIS specific data structure that you'll use to filter requested data.</li>
<li>Serialize the requested data for transit between the web server and web browser.</li>
<li>Use a JavaScript library to convert the response into markers and polylines on a map.</li>
</ul>
<p>This will save you a significant amount of typing code, because you don't need to write parsers and generators to convert the on-the-wire datastream into in-memory representations.</p>
<h3><strong>Reviewing the data</strong></h3>
<p><a href="https://data.cityofnewyork.us/">NYC Open Data</a> makes the wealth of public data generated by various New York City agencies and other City organizations available for public use. In this lab, you'll visualize the  <a href="https://data.cityofnewyork.us/Transportation/Subway-Stations/arq3-7z49">Subway Station</a> and  <a href="https://data.cityofnewyork.us/Transportation/Subway-Lines/3qz8-muuu">Subway Line</a> data sets.</p>
<aside class="warning">
<b>Note:</b> Please review the <a href="http://www1.nyc.gov/home/terms-of-use.page" target="_blank">NYC Open Data Terms of Use</a> to understand the restrictions under which you can use this data set.
</aside>
<p>You'll visualize the data with markers on the map, rendered using  <a href="https://developers.google.com/maps/documentation/javascript/datalayer">Google Maps JavaScript API's Data layer</a>.</p>
<p>Start by downloading the GeoJSON data from the NYC Open Data website into your app.</p>
<p>In your Cloud Shell instance's command line window make a <code>data</code> directory inside the <code>nyc-subway</code> directory, <code>cd</code> to that directory:</p>
<pre><code>mkdir data &amp;&amp; cd data&#x000A;</code></pre>
<p>Now  use <code>curl</code> to retrieve subway stations:</p>
<pre><code>curl "https://data.cityofnewyork.us/api/geospatial/arq3-7z49?method=export&amp;format=GeoJSON" -o subway-stations.geojson&#x000A;</code></pre>
<p>And again to retrieve the subway lines:</p>
<pre><code>curl "https://data.cityofnewyork.us/api/geospatial/3qz8-muuu?method=export&amp;format=GeoJSON" -o subway-lines.geojson&#x000A;</code></pre>
<p>Finally, back out of the data directory:</p>
<pre><code>cd ..&#x000A;</code></pre>
<p>Next, update the <code>app.yaml</code> file the the following code to reflect a more complex App Engine application:</p>

```yaml
runtime: go
api_version: go1

handlers:
- url: /
  static_files: static/index.html
  upload: static/index.html
- url: /(.*\.(js|html|css))$
  static_files: static/\1
  upload: static/.*\.(js|html|css)$
- url: /.*
  script: auto
```

<p>This <code>app.yaml</code> configuration directs requests for <code>/</code>, <code>/*.js</code>, <code>/*.css</code> and <code>/*.html</code> to a set of static files. This means that the static HTML component of your app will be served directly by App Engine file serving infrastructure, and not your Go app. This reduces server load, and increases serving speed.</p>
<p>Now, create a directory called <code>static</code> under the main directory of the app (this means creating the directory <code>nyc-subway/static</code>), and add the following three files into this new directory:</p>
<h4>index.html</h4>

```html
<html>
  <head>
    <title>NYC Subway Stations</title>
    <link rel="stylesheet" type="text/css" href="style.css">
  </head>
  <body>
    <div id="map" class="map"></div>

    <script src="app.js"></script>
    <!-- TODO: Replace YOUR_API_KEY in the following line -->
    <script async defer src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&callback=initMap">
    </script>
  </body>
</html>
```

<p>The <code>index.html</code> is the file initially served when the browser requests the root of the web application. This file sets up a <code>&lt;div&gt;</code> to contain the map, loads some CSS to style the map, pulls in a JavaScript file that will configure the map, and finally loads the  <a href="https://developers.google.com/maps/documentation/javascript/">Google Maps JavaScript API</a>.</p>
<p>There is one quick diversion you need to make here, to replace the <code>YOUR_API_KEY</code> in the above HTML source with an actual Google Maps API key. Follow these steps to get an API key:</p>
<ol>
<li>Go to the <a href="https://console.developers.google.com/flows/enableapi?apiid=maps_backend&amp;reusekey=true" target="_blank">Google API Console</a>. (Use this link, because it has some smarts to enable the correct APIs for you.)</li>
<li>From the <strong>Create a project</strong> dropdown, select the Project ID for this lab).</li>
<li>Click <strong>Continue</strong> to enable the API and any related services.</li>
<li>On the Credentials page, click <strong>What credentials do I need</strong> to get an API key.</li>
<li>Copy the API key</li>
<li>In <code>index.html</code>, replace <code>YOUR_API_KEY</code> with the copied API key.</li>
</ol>
<p>Save the file.</p>
<p>The script resource in <code>index.html</code> looks similar to this once you replace <code>YOUR_API_KEY</code>:</p>
<pre><code>&lt;script async defer&#x000A;    src="https://maps.googleapis.com/maps/api/js?key=AIza12345678ABC_DEFGHIJ&amp;callback=initMap"&gt;&#x000A;</code></pre>
<p>Click <em>Check my progress</em> to verify the objective.
<ql-activity-tracking step="2">
Generate API key
</ql-activity-tracking></p>
<p>Create a new file <code>static/style.css</code>. This  is a CSS style file that makes the map take up the whole page, without any margin or padding.</p>
<h4>style.css</h4>

```css
.map {
    height: 100%;
}

html, body {
    height: 100%;
    margin: 0;
    padding: 0;
}
```

<p>Create a new file <code>static/app.js</code>.</p>
<h4>app.js</h4>

```js
// Eslint configuration for editor.
/* globals google */
/* eslint-env browser */
/* eslint quotes: ["warn", "single"]*/

function initMap() {
  const map = new google.maps.Map(document.querySelector('#map'), {
    zoom: 11,
    center: {
      // New York City
      lat: 40.7305,
      lng: -73.9091
    }
  });
  map.data.loadGeoJson('/data/subway-stations');
  map.data.loadGeoJson('/data/subway-lines');
}
```

<p>This is a JavaScript file that configures your map to be centered on New York City, at zoom level 11. It then loads both the Subway Stations and Subway Lines GeoJSON data that you retrieved earlier.</p>
<p>The eagle eyed amongst you may have noticed a disconnect - you have the GeoJSON data inside your application folder, but it isn't exposed in the <code>app.yaml</code> file. You're going to serve the GeoJSON data from the Go app, so you can do some smart things with the data in future sections.</p>
<p>Next, update the Go source code.</p>
<p>Rename the <code>hello.go</code> file to <code>main.go</code> to better reflect what you're doing in this app (Right-click on the file &gt; <strong>Rename</strong>) and update the file with the following:</p>
<h4>main.go</h4>

```go
package main

import (
        "io/ioutil"
        "log"
        "net/http"
        "path/filepath"
)

// GeoJSON is a cache of the NYC Subway Station and Line data.
var GeoJSON = make(map[string][]byte)

// cacheGeoJSON loads files under data into `GeoJSON`.
func cacheGeoJSON() {
        filenames, err := filepath.Glob("data/*")
        if err != nil {
                log.Fatal(err)
        }
        for _, f := range filenames {
                name := filepath.Base(f)
                dat, err := ioutil.ReadFile(f)
                if err != nil {
                        log.Fatal(err)
                }
                GeoJSON[name] = dat
        }
}

// init is called from the App Engine runtime to initialize the app.
func init() {
        cacheGeoJSON()
        http.HandleFunc("/data/subway-stations", subwayStationsHandler)
        http.HandleFunc("/data/subway-lines", subwayLinesHandler)
}

func subwayStationsHandler(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Content-type", "application/json")
        w.Write(GeoJSON["subway-stations.geojson"])
}

func subwayLinesHandler(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Content-type", "application/json")
        w.Write(GeoJSON["subway-lines.geojson"])
}
```

<p>This code does a couple of things. On startup it loads all the files under <code>data/</code> into memory, then it configures itself to serve requests for files under <code>/data/</code>. This is the missing link mentioned previously. In future sections you'll do more intelligent things with the GeoJSON data than just serving it wholesale.</p>
<p>Now you're ready to re-run the web application locally. It's also a good idea to use the <code>go fmt</code> command on a regular basis, to correct any formatting mistakes that crept into your code.</p>
<p>In Cloud Shell, in the <code>nyc-subway</code> directory run the following:</p>
<pre><code>goapp serve&#x000A;</code></pre>
<p>Refresh the web preview tab, and you should see a view like the following.</p>
<p><img alt="map.png" src="https://cdn.qwiklabs.com/ftFF1SMktcA07CaesRRK%2F%2BNH5SFVlVHwF5YM8XM8Z8I%3D"></p>
<p>You have a Go server serving content to a browser that contains all of the subway stations in NYC, along with the train lines. But it doesn't really look like a map of a subway, does it?</p>
<p>In the next section you will re-style this map to make it look like a subway map.</p>
<h2 id="step7">Styling the map</h2>
<p>One of the most important requirements of a map is that it communicates effectively. It would be good if the subway lines don't compete with all the information on the base map tiles. In this section you'll use a map style to hide a lot of information on the map and drop the colors to shades of gray. Then you'll add JavaScript code to style the subway lines based on the  <a href="https://en.wikipedia.org/wiki/New_York_City_Subway_nomenclature">New York City Subway nomenclature</a>.</p>
<h3><strong>Styling the map</strong></h3>
<p>Open <a href="https://mapstyle.withgoogle.com/" target="_blank">mapstyle.withgoogle.com</a> in a new tab and click the <strong>Create a Style</strong> button.</p>
<p><img alt="styling_map.png" src="https://cdn.qwiklabs.com/7Pl%2FpWjcVRRmOrs%2BZw9IJd7YpipTV2JW8FWbZY7iAbQ%3D"></p>
<p>This puts you in a screen with map style attributes on the left, and an unstyled map on the right. The screenshot below shows a style that turns down the density of <strong>Landmarks</strong> and <strong>Labels,</strong> and uses the <strong>Silver</strong> theme to drop the amount of color on the base map. Try different themes, or select the <strong>More Options</strong> button at the bottom of the screen and explore all the available styling capabilities.</p>
<p>When you are ready, click the <strong>Finish</strong> button.</p>
<p><img alt="map_style.png" src="https://cdn.qwiklabs.com/79xtMipvTkBPjHiY%2Baaxe42%2BEZrmTSLTf1s8UZwEP2g%3D"></p>
<p>Extract the JSON description of the map style you have configured so you can put the styling JSON into your code in the next step by clicking the <strong>Copy JSON</strong> button.</p>
<p><img alt="export_style.png" src="https://cdn.qwiklabs.com/iFSA%2BKswWki41bY%2FbAON2wji%2FThl0Bg0%2F6u9zPOtq%2B4%3D"></p>
<p>Go back to the <code>app.js</code> file. You are going to add a bunch of code to apply the map style you just designed to the map, along with some code to style the stations and the lines.</p>
<p>The following code includes style code copied for the example in the previous step. To use the style you created, replace the the code between <code>const mapStyle =</code> and the semi-colon (;) above the <code>//Colors from</code> line. You can also opt not to use the map you styled and just use the lab example.</p>
<h4>app.js</h4>

```js
// Eslint configuration for editor.
/* globals google */
/* eslint-env browser */
/* eslint quotes: ["warn", "single"]*/

// Replace the following with the JSON from the Styling Wizard
const mapStyle = [
  {
    elementType: 'geometry',
    stylers: [
      {
        color: '#eceff1'
      }
    ]
  },
  {
    elementType: 'labels',
    stylers: [
      {
        visibility: 'off'
      }
    ]
  },
  {
    featureType: 'administrative',
    elementType: 'labels',
    stylers: [
      {
        visibility: 'off'
      }
    ]
  },
  {
    featureType: 'road',
    elementType: 'geometry',
    stylers: [
      {
        color: '#cfd8dc'
      }
    ]
  },
  {
    featureType: 'road',
    elementType: 'geometry.stroke',
    stylers: [
      {
        visibility: 'off'
      }
    ]
  },
  {
    featureType: 'road.local',
    stylers: [
      {
        visibility: 'off'
      }
    ]
  },
  {
    featureType: 'water',
    stylers: [
      {
        color: '#b0bec5'
      }
    ]
  }
];

// Colors from https://en.wikipedia.org/wiki/New_York_City_Subway_nomenclature#Colors_and_trunk_lines
const routeColors = {
  // IND Eighth Avenue Line
  A: '#2850ad',
  C: '#2850ad',
  E: '#2850ad',

  // IND Sixth Avenue Line
  B: '#ff6319',
  D: '#ff6319',
  F: '#ff6319',
  M: '#ff6319',

  // IND Crosstown Line
  G: '#6cbe45',

  // BMT Canarsie Line
  L: '#a7a9ac',

  // BMT Nassau Street Line
  J: '#996633',
  Z: '#996633',

  // BMT Broadway Line
  N: '#fccc0a',
  Q: '#fccc0a',
  R: '#fccc0a',
  W: '#fccc0a',

  // IRT Broadway – Seventh Avenue Line
  '1': '#ee352e',
  '2': '#ee352e',
  '3': '#ee352e',

  // IRT Lexington Avenue Line
  '4': '#00933c',
  '5': '#00933c',
  '6': '#00933c',

  // IRT Flushing Line
  '7': '#b933ad',

  // Shuttles
  S: '#808183'
};

// initMap is called from the Google Maps JS library after the library has initialised itself.
function initMap() {
  const map = new google.maps.Map(document.querySelector('#map'), {
    zoom: 12,
    center: {
      // New York City
      lat: 40.7305,
      lng: -73.9091
    },
    styles: mapStyle
  });
  const infowindow = new google.maps.InfoWindow();

  // Load GeoJSON data from the back end
  map.data.loadGeoJson('/data/subway-stations');
  map.data.loadGeoJson('/data/subway-lines');

  // Style the GeoJSON features (stations & lines)
  map.data.setStyle(feature => {
    const line = feature.getProperty('line');
    // Stations have line property, while lines do not.
    if (line) {
      // Icon path from: https://material.io/icons/#ic_train
      return {
        icon: {
          fillColor: '#00b0ff',
          strokeColor: '#3c8cb8',
          fillOpacity: 1.0,
          scale: 1.2,
          path: 'M12 2c-4 0-8 .5-8 4v9.5C4 17.43 5.57 19 7.5 19L6 20.5v.5h2.2' +
            '3l2-2H14l2 2h2v-.5L16.5 19c1.93 0 3.5-1.57 3.5-3.5V6c0-3.5-3.58-' +
            '4-8-4zM7.5 17c-.83 0-1.5-.67-1.5-1.5S6.67 14 7.5 14s1.5.67 1.5 1' +
            '.5S8.33 17 7.5 17zm3.5-7H6V6h5v4zm2 0V6h5v4h-5zm3.5 7c-.83 0-1.5' +
            '-.67-1.5-1.5s.67-1.5 1.5-1.5 1.5.67 1.5 1.5-.67 1.5-1.5 1.5z'
        }
      };
    }

    // if type is not a station, it's a subway line
    const routeSymbol = feature.getProperty('rt_symbol');

    return {
      strokeColor: routeColors[routeSymbol]
    };
  });

  map.data.addListener('click', ev => {
    const f = ev.feature;
    const stationName = f.getProperty('name');
    let line = f.getProperty('line');
    // Stations have line property, while lines do not.
    if (!line) {
      return;
    }
    if (line.includes('-')) {
      line += ' lines';
    } else {
      line += ' line';
    }
    infowindow.setContent(`<b>${stationName} Station</b><br/>Serves ${line}`);
    // Hat tip geocodezip: http://stackoverflow.com/questions/23814197
    infowindow.setPosition(f.getGeometry().get());
    infowindow.setOptions({
      pixelOffset: new google.maps.Size(0, -30)
    });
    infowindow.open(map);
  });
}
```

<p>Save the file.</p>
<p>Look through the code and see what's here.</p>
<p>The first constant defined is <code>mapStyle</code>. You should replace this block with the JSON content you built with  <a href="https://mapstyle.withgoogle.com/">mapstyle.withgoogle.com</a>. This changes the style of the basemap tiles. The style in the above snippet is specifically designed to drop the map to the background of the user's attention, and allow the subway lines and stations to dominate.</p>
<p>The next constant defined is a hashmap named <code>routeColors</code>. This constant is used further down in the code to style the subway lines depending on the  <a href="https://en.wikipedia.org/wiki/New_York_City_Subway_nomenclature#Colors_and_trunk_lines">NYC Subway Nomenclature</a>. Then we have the <code>initMap</code> function that the Google Maps JavaScript API calls to initialize the content of the map.</p>
<p>The first half dozen lines of this function should be familiar from the previous section. We initialize the map, and load the GeoJSON for both the stations and the lines. There are two additions here, the first is the addition of the <code>styles</code> attribute to the Google Maps constructor to link in the <code>mapStyle</code>, and the second is a constant definition for an <code>InfoWindow</code>.</p>
<p>An  <a href="https://developers.google.com/maps/documentation/javascript/infowindows">InfoWindow</a> is a pop up window  that displays when someone clicks on a subway station. It is used in the click listener that is added to the Data layer. There is a block of javascript that is responsible for constructing a useful text description of the station a user clicks on, and opens the InfoWindow.</p>
<p>The last piece of new code is the style for the content of the Google Maps Data layer. This inline function is called by the Google Maps Data layer for each item in the data layer. It returns a javascript object that defines how it should be styled on screen. See  <a href="https://developers.google.com/maps/documentation/javascript/datalayer#style_geojson_data">Styling GeoJSON Data</a> for information on how to use this capability.</p>
<p>If you refresh the preview in your web browser, you should see something like this:</p>
<p><img alt="map_prev.png" src="https://cdn.qwiklabs.com/HwLleeFCdVoaK%2B5gp6Odlm2gaArTFbUToNRclSg62vU%3D"></p>
<p>In the next section you will look into reducing the amount of GeoJSON data the browser has to both download and parse.</p>
<h2 id="step8">Filtering the data</h2>
<p>In this section you'll add code to the Go back-end that narrows the returned stations to just the visible area of the map. This reduces the amount of data sent to the user's browser, making the site faster.</p>
<p>There's an API endpoint that the front end uses to get the stations, but currently the client does not supply the visible area that the user is viewing. So, first add the visible map area to the requested URL as parameters. You do this by delaying loading of the GeoJSON for subway stations until an idle callback fires. This means you have access to the map viewport.</p>
<h4>app.js</h4>

```js
// Eslint configuration for editor.
/* globals google */
/* eslint-env browser */
/* eslint quotes: ["warn", "single"]*/

// Map Style for the basemap tiles.
const mapStyle = [
  {
    elementType: 'geometry',
    stylers: [
      {
        color: '#eceff1'
      }
    ]
  },
  {
    elementType: 'labels',
    stylers: [
      {
        visibility: 'off'
      }
    ]
  },
  {
    featureType: 'administrative',
    elementType: 'labels',
    stylers: [
      {
        visibility: 'off'
      }
    ]
  },
  {
    featureType: 'road',
    elementType: 'geometry',
    stylers: [
      {
        color: '#cfd8dc'
      }
    ]
  },
  {
    featureType: 'road',
    elementType: 'geometry.stroke',
    stylers: [
      {
        visibility: 'off'
      }
    ]
  },
  {
    featureType: 'road.local',
    stylers: [
      {
        visibility: 'off'
      }
    ]
  },
  {
    featureType: 'water',
    stylers: [
      {
        color: '#b0bec5'
      }
    ]
  }
];

// Colors from https://en.wikipedia.org/wiki/New_York_City_Subway_nomenclature#Colors_and_trunk_lines
const routeColors = {
  // IND Eighth Avenue Line
  A: '#2850ad',
  C: '#2850ad',
  E: '#2850ad',

  // IND Sixth Avenue Line
  B: '#ff6319',
  D: '#ff6319',
  F: '#ff6319',
  M: '#ff6319',

  // IND Crosstown Line
  G: '#6cbe45',

  // BMT Canarsie Line
  L: '#a7a9ac',

  // BMT Nassau Street Line
  J: '#996633',
  Z: '#996633',

  // BMT Broadway Line
  N: '#fccc0a',
  Q: '#fccc0a',
  R: '#fccc0a',
  W: '#fccc0a',

  // IRT Broadway – Seventh Avenue Line
  '1': '#ee352e',
  '2': '#ee352e',
  '3': '#ee352e',

  // IRT Lexington Avenue Line
  '4': '#00933c',
  '5': '#00933c',
  '6': '#00933c',

  // IRT Flushing Line
  '7': '#b933ad',

  // Shuttles
  S: '#808183'
};

// initMap is called from the Google Maps JS library after the library has initialised itself.
function initMap() {
  const map = new google.maps.Map(document.querySelector('#map'), {
    zoom: 12,
    center: {
      // New York City
      lat: 40.7305,
      lng: -73.9091
    },
    styles: mapStyle
  });
  const infowindow = new google.maps.InfoWindow();
  let stationDataFeatures = [];

  // Load GeoJSON for subway lines. Stations are loaded in the idle callback.
  map.data.loadGeoJson('/data/subway-lines');

  // Style the GeoJSON features (stations & lines)
  map.data.setStyle(feature => {
    const line = feature.getProperty('line');
    // Stations have line property, while lines do not.
    if (line) {
      // Icon path from: https://material.io/icons/#ic_train
      return {
        icon: {
          fillColor: '#00b0ff',
          strokeColor: '#3c8cb8',
          fillOpacity: 1.0,
          scale: 1.2,
          path: 'M12 2c-4 0-8 .5-8 4v9.5C4 17.43 5.57 19 7.5 19L6 20.5v.5h2.2' +
            '3l2-2H14l2 2h2v-.5L16.5 19c1.93 0 3.5-1.57 3.5-3.5V6c0-3.5-3.58-' +
            '4-8-4zM7.5 17c-.83 0-1.5-.67-1.5-1.5S6.67 14 7.5 14s1.5.67 1.5 1' +
            '.5S8.33 17 7.5 17zm3.5-7H6V6h5v4zm2 0V6h5v4h-5zm3.5 7c-.83 0-1.5' +
            '-.67-1.5-1.5s.67-1.5 1.5-1.5 1.5.67 1.5 1.5-.67 1.5-1.5 1.5z'
        }
      };
    }

    // if type is not a station, it's a subway line
    const routeSymbol = feature.getProperty('rt_symbol');
    return {
      strokeColor: routeColors[routeSymbol]
    };
  });

  map.data.addListener('click', ev => {
    const f = ev.feature;
    const stationName = f.getProperty('name');
    let line = f.getProperty('line');
    // Stations have line property, while lines do not.
    if (!line) {
      return;
    }
    if (line.includes('-')) {
      line += ' lines';
    } else {
      line += ' line';
    }
    infowindow.setContent(`<b>${stationName} Station</b><br/>Serves ${line}`);
    // Hat tip geocodezip: http://stackoverflow.com/questions/23814197
    infowindow.setPosition(f.getGeometry().get());
    infowindow.setOptions({
      pixelOffset: new google.maps.Size(0, -30)
    });
    infowindow.open(map);
  });

  // The idle callback is called every time the map has finished
  // moving, zooming,  panning or animating. We use it to load
  // Geojson for the new viewport.
  google.maps.event.addListener(map, 'idle', () => {
    const sw = map.getBounds().getSouthWest();
    const ne = map.getBounds().getNorthEast();
    map.data.loadGeoJson(
      `/data/subway-stations?viewport=${sw.lat()},${sw.lng()}|${ne.lat()},${ne.lng()}`,
      null,
      features => {
        stationDataFeatures.forEach(dataFeature => {
          map.data.remove(dataFeature);
        });
        stationDataFeatures = features;
      }
    );
  });
}

  // The idle callback is called every time the map has finished
  // moving, zooming,  panning or animating. We use it to load
  // Geojson for the new viewport.
  google.maps.event.addListener(map, 'idle', () => {
    const sw = map.getBounds().getSouthWest();
    const ne = map.getBounds().getNorthEast();
    map.data.loadGeoJson(
      `/data/subway-stations?viewport=${sw.lat()},${sw.lng()}|${ne.lat()},${ne.lng()}`,
      null,
      features => {
        stationDataFeatures.forEach(dataFeature => {
          map.data.remove(dataFeature);
        });
        stationDataFeatures = features;
      }
    );
  });
```

<p>This updated <code>app.js</code> introduces a couple of  <a href="https://en.wikipedia.org/wiki/ECMAScript#6th_Edition_-_ECMAScript_2015">ES6</a> features to make the JavaScript easier to read. This will  <a href="http://caniuse.com/#feat=es6-class">work fine in almost all current web browsers</a>. You will need to transpile this JavaScript to ES5 before using this in a production web site, but for our purposes in this tutorial, ES6 is fine. We use ES6 style  <a href="https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Template_literals">Template literals</a> and Fat Arrow syntax to cut down on boilerplate code.</p>
<p>This code contains a couple of additions over the previous section. An idle callback asks the map for viewport bounds, and adds these to the URL used to request subway station GeoJSON data. The backend to will be updated to use these parameters shortly. Also added is a state to allow you to drop station features once we have loaded the new stations.</p>
<p>Now add the ability to filter the stations returned based on the viewport reported from the front end. You need to install a couple of Go packages for your server first.</p>
<p>Shut down the server with a <code>[CTRL] + [c]</code> in the Cloud Shell command line, then run the following commands:</p>
<pre><code>go get github.com/paulmach/go.geojson&#x000A;</code></pre>
<pre><code>go get github.com/dhconnelly/rtreego&#x000A;</code></pre>
<p>These packages provide the following capabilities:</p>
<ul>
<li>Parse GeoJSON with  <a href="https://github.com/paulmach/go.geojson">github.com/paulmach/go.geojson</a>
</li>
<li>Run viewbox bounded queries efficiently with  <a href="https://github.com/dhconnelly/rtreego">github.com/dhconnelly/rtreego</a>
</li>
</ul>
<p>Next you will create a new <code>go</code> file responsible for parsing the content from the <code>subway-stations.geojson</code> file, storing the resultant stations in an  <a href="https://en.wikipedia.org/wiki/R-tree">R-tree data structure</a>, and responding to the <code>/subway-stations</code> request.</p>
<p>In the code editor, create a new file in the <code>nyc-subway</code> directory named <code>stations.go</code>.</p>
<h4>stations.go</h4>

```go
package main

import (
        "encoding/json"
        "fmt"
        "log"
        "math"
        "net/http"
        "strconv"
        "strings"

        rtree "github.com/dhconnelly/rtreego"
        geojson "github.com/paulmach/go.geojson"
)

// Stations is an RTree housing the stations
var Stations = rtree.NewTree(2, 25, 50)

// Station is a wrapper for `*geojson.Feature` so that we can implement
// `rtree.Spatial` interface type.
type Station struct {
        feature *geojson.Feature
}

// Bounds implements `rtree.Spatial` so we can load
// stations into an `rtree.Rtree`.
func (s *Station) Bounds() *rtree.Rect {
        return rtree.Point{
                s.feature.Geometry.Point[0],
                s.feature.Geometry.Point[1],
        }.ToRect(1e-6)
}

// loadStations loads the geojson features from
// `subway-stations.geojson` into the `Stations` rtree.
func loadStations() {
        stationsGeojson := GeoJSON["subway-stations.geojson"]
        fc, err := geojson.UnmarshalFeatureCollection(stationsGeojson)
        if err != nil {
                // Note: this will take down the GAE instance by exiting this process.
                log.Fatal(err)
        }
        for _, f := range fc.Features {
                Stations.Insert(&Station{f})
        }
}

// subwayStationsHandler reads r for a "viewport" query parameter
// and writes a GeoJSON response of the features contained in
// that viewport into w.
func subwayStationsHandler(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Content-type", "application/json")
        vp := r.FormValue("viewport")
        rect, err := newRect(vp)
        if err != nil {
                str := fmt.Sprintf("Couldn't parse viewport: %s", err)
                http.Error(w, str, 400)
                return
        }
        s := Stations.SearchIntersect(rect)
        fc := geojson.NewFeatureCollection()
        for _, station := range s {
                fc.AddFeature(station.(*Station).feature)
        }
        err = json.NewEncoder(w).Encode(fc)
        if err != nil {
                str := fmt.Sprintf("Couldn't encode results: %s", err)
                http.Error(w, str, 500)
                return
        }
}

// newRect constructs a `*rtree.Rect` for a viewport.
func newRect(vp string) (*rtree.Rect, error) {
        ss := strings.Split(vp, "|")
        sw := strings.Split(ss[0], ",")
        swLat, err := strconv.ParseFloat(sw[0], 64)
        if err != nil {
                return nil, err
        }
        swLng, err := strconv.ParseFloat(sw[1], 64)
        if err != nil {
                return nil, err
        }
        ne := strings.Split(ss[1], ",")
        neLat, err := strconv.ParseFloat(ne[0], 64)
        if err != nil {
                return nil, err
        }
        neLng, err := strconv.ParseFloat(ne[1], 64)
        if err != nil {
                return nil, err
        }
        minLat := math.Min(swLat, neLat)
        minLng := math.Min(swLng, neLng)
        distLat := math.Max(swLat, neLat) - minLat
        distLng := math.Max(swLng, neLng) - minLng

        r, err := rtree.NewRect(
                rtree.Point{
                        minLng,
                        minLat,
                },
                []float64{
                        distLng,
                        distLat,
                })
        if err != nil {
                return nil, err
        }
        return r, nil
}
```

<p>There is a fair amount going on in this file. The <code>package</code> line and the <code>import</code> stanza place this file in the same package as declared in <code>main.go</code>, and import the required dependencies.</p>
<p>The <code>stations</code> variable declares an R-tree where you'll store all the stations for the NYC subway. An R-tree data structure is specifically designed for geographic queries like "Show me all the points within a specified bounding box." This is useful for finding stations within a viewport.</p>
<p>Next, define a <code>Station</code> structure. This is a way of being able to import the parsed GeoJSON data into the <code>Stations</code> R-tree. We wrap the <code>*geojson.Feature</code> and implement <code>Bounds()</code> as a function on <code>*Station</code>. This way, Station implements the  <a href="https://godoc.org/github.com/dhconnelly/rtreego#Spatial">rtree.Spatial</a> interface, as required by  <a href="https://godoc.org/github.com/dhconnelly/rtreego#Rtree.Insert">rtree.Insert</a>.</p>
<p>Then a <code>loadStations()</code> function is defined, which is called when this module is initialized, which you'll see when you update <code>main.go</code> below. Finally, we define <code>subwayStationsHandler()</code> function, along with its <code>newRect(string)</code> helper function. While <code>newRect</code> looks long, it is merely converting the <code>viewport</code> parameter that we constructed above in <code>app.js</code> into a search query that the <code>Stations</code> R-tree can understand.</p>
<p>The <code>subwayStationsHandler</code> will be invoked when the front end makes a request to the <code>/data/subway-stations</code> endpoint. At the core, this function retrieves the <code>viewport</code> parameter, converts that to a search query using the previously mentioned <code>newRect</code> helper function, executes a query over the <code>Stations</code> R-tree, and converts the returned stations into JSON, which is then written out to the browser.</p>
<p>Next you will make a few small changes to the <code>main.go</code> file, because the <code>subwayStationsHandler</code> function has moved to <code>stations.go</code>, and you need to add a call to <code>loadStations()</code> in the <code>init()</code> function.</p>
<h4>main.go</h4>

```go
package main

import (
        "io/ioutil"
        "log"
        "net/http"
        "path/filepath"
)

// GeoJSON is a cache of the NYC Subway Station and Line data.
var GeoJSON = make(map[string][]byte)

// cacheGeoJSON loads files under data into `GeoJSON`.
func cacheGeoJSON() {
        filenames, err := filepath.Glob("data/*")
        if err != nil {
                // Note: this will take down the GAE instance by exiting this process.
                log.Fatal(err)
        }
        for _, f := range filenames {
                name := filepath.Base(f)
                dat, err := ioutil.ReadFile(f)
                if err != nil {
                        log.Fatal(err)
                }
                GeoJSON[name] = dat
        }
}

// init is called from the App Engine runtime to initialize the app.
func init() {
        cacheGeoJSON()
        loadStations()
        http.HandleFunc("/data/subway-stations", subwayStationsHandler)
        http.HandleFunc("/data/subway-lines", subwayLinesHandler)
}

func subwayLinesHandler(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Content-type", "application/json")
        w.Write(GeoJSON["subway-lines.geojson"])
}
```

<p>Restart the server:</p>
<pre><code>go fmt *.go &amp;&amp; goapp serve&#x000A;</code></pre>
<p>You should now be able to view the updated content in your web browser.</p>
<p>As you zoom in and out, or pan the map, you'll notice that stations disappear and reappear in odd ways near the edge of the viewport. This is because in this version you've tightly cropped the returned stations to the current browser viewport. You can work around this by expanding the viewport used to query the <code>Stations</code> R-tree. We can also look at reducing the number of stations that are displayed when the map is first displayed, by clustering stations. This is the focus of the next section!</p>
<h2 id="step9">Clustering on the server</h2>
<p>In this section you'll cluster the approximately four hundred stations in the NYC subway so that the user of the map isn't overwhelmed with stations when viewing the map. This requires changes to both the front end and the back end code. The front end requires change because points are no longer just stations. Each point can be either a station, or a cluster of stations. On the back end, you need to add code that groups the stations into clusters and generates useful information for the info windows (pop ups) in a way that works for both clusters and individual stations.</p>
<p>In the <code>app.js</code> file, you need to make three major changes:</p>
<ul>
<li>
<p>Add the map's current zoom level to the <code>/data/subway-stations</code> AJAX request, for use by the clusterer implementation.</p>
</li>
<li>
<p>Add dynamic styling for the markers that will visually differentiate between subway stations and clusters of stations.</p>
</li>
<li>
<p>Simplify the click handler's info window construction code, because the GeoJSON returned by the back-end is now responsible for populating content for the title and description.</p>
</li>
</ul>
<h4>app.js</h4>

```js
// Eslint configuration for editor.
/* globals google */
/* eslint-env browser */
/* eslint quotes: ["warn", "single"]*/

// Map Style for the basemap tiles.
const mapStyle = [
  {
    elementType: 'geometry',
    stylers: [
      {
        color: '#eceff1'
      }
    ]
  },
  {
    elementType: 'labels',
    stylers: [
      {
        visibility: 'off'
      }
    ]
  },
  {
    featureType: 'administrative',
    elementType: 'labels',
    stylers: [
      {
        visibility: 'off'
      }
    ]
  },
  {
    featureType: 'road',
    elementType: 'geometry',
    stylers: [
      {
        color: '#cfd8dc'
      }
    ]
  },
  {
    featureType: 'road',
    elementType: 'geometry.stroke',
    stylers: [
      {
        visibility: 'off'
      }
    ]
  },
  {
    featureType: 'road.local',
    stylers: [
      {
        visibility: 'off'
      }
    ]
  },
  {
    featureType: 'water',
    stylers: [
      {
        color: '#b0bec5'
      }
    ]
  }
];

// Colors from https://en.wikipedia.org/wiki/New_York_City_Subway_nomenclature#Colors_and_trunk_lines
const routeColors = {
  // IND Eighth Avenue Line
  A: '#2850ad',
  C: '#2850ad',
  E: '#2850ad',

  // IND Sixth Avenue Line
  B: '#ff6319',
  D: '#ff6319',
  F: '#ff6319',
  M: '#ff6319',

  // IND Crosstown Line
  G: '#6cbe45',

  // BMT Canarsie Line
  L: '#a7a9ac',

  // BMT Nassau Street Line
  J: '#996633',
  Z: '#996633',

  // BMT Broadway Line
  N: '#fccc0a',
  Q: '#fccc0a',
  R: '#fccc0a',
  W: '#fccc0a',

  // IRT Broadway – Seventh Avenue Line
  '1': '#ee352e',
  '2': '#ee352e',
  '3': '#ee352e',

  // IRT Lexington Avenue Line
  '4': '#00933c',
  '5': '#00933c',
  '6': '#00933c',

  // IRT Flushing Line
  '7': '#b933ad',

  // Shuttles
  S: '#808183'
};

// initMap is called from the Google Maps JS library after the library has initialised itself.
function initMap() {
  const map = new google.maps.Map(document.querySelector('#map'), {
    zoom: 12,
    center: {
      // New York City
      lat: 40.7305,
      lng: -73.9091
    },
    styles: mapStyle
  });
  const infowindow = new google.maps.InfoWindow();
  let stationDataFeatures = [];

  // Load GeoJSON for subway lines. Stations are loaded in the idle callback.
  map.data.loadGeoJson('/data/subway-lines');

  // Style the GeoJSON features (stations & lines)
  map.data.setStyle(feature => {
    const type = feature.getProperty('type');
    if (type === 'cluster') {
      // Icon path from: https://material.io/icons/#ic_add_circle_outline
      return {
        icon: {
          fillColor: '#00b0ff',
          strokeColor: '#3c8cb8',
          fillOpacity: 1.0,
          scale: 1.2,
          path: 'M13 7h-2v4H7v2h4v4h2v-4h4v-2h-4V7zm-1-5C6.48 2 2 6.48 2 12s4' +
            '.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 18c-4.41 0-8-3.59-8-8' +
            's3.59-8 8-8 8 3.59 8 8-3.59 8-8 8z'
        }
      };
    } else if (type === 'station') {
      // Icon path from: https://material.io/icons/#ic_train
      return {
        icon: {
          fillColor: '#00b0ff',
          strokeColor: '#3c8cb8',
          fillOpacity: 1.0,
          scale: 1.2,
          path: 'M12 2c-4 0-8 .5-8 4v9.5C4 17.43 5.57 19 7.5 19L6 20.5v.5h2.2' +
            '3l2-2H14l2 2h2v-.5L16.5 19c1.93 0 3.5-1.57 3.5-3.5V6c0-3.5-3.58-' +
            '4-8-4zM7.5 17c-.83 0-1.5-.67-1.5-1.5S6.67 14 7.5 14s1.5.67 1.5 1' +
            '.5S8.33 17 7.5 17zm3.5-7H6V6h5v4zm2 0V6h5v4h-5zm3.5 7c-.83 0-1.5' +
            '-.67-1.5-1.5s.67-1.5 1.5-1.5 1.5.67 1.5 1.5-.67 1.5-1.5 1.5z'
        }
      };
    }

    // if type is not a station or a cluster, it's a subway line
    const routeSymbol = feature.getProperty('rt_symbol');
    return {
      strokeColor: routeColors[routeSymbol]
    };
  });

  map.data.addListener('click', ev => {
    const f = ev.feature;
    const title = f.getProperty('title');
    const description = f.getProperty('description');

    if (!description) {
      return;
    }

    infowindow.setContent(`<b>${title}</b><br/> ${description}`);
    // Hat tip geocodezip: http://stackoverflow.com/questions/23814197
    infowindow.setPosition(f.getGeometry().get());
    infowindow.setOptions({
      pixelOffset: new google.maps.Size(0, -30)
    });
    infowindow.open(map);
  });

  // The idle callback is called every time the map has finished
  // moving, zooming,  panning or animating. We use it to load
  // Geojson for the new viewport.
  google.maps.event.addListener(map, 'idle', () => {
    const sw = map.getBounds().getSouthWest();
    const ne = map.getBounds().getNorthEast();
    const zm = map.getZoom();
    map.data.loadGeoJson(
      `/data/subway-stations?viewport=${sw.lat()},${sw.lng()}|${ne.lat()},${ne.lng()}&zoom=${zm}`,
      null,
      features => {
        stationDataFeatures.forEach(dataFeature => {
          map.data.remove(dataFeature);
        });
        stationDataFeatures = features;
      }
    );
  });
}
```

<p>Next, modify the back end code. The <code>main.go</code> file is unchanged from the previous step, but you need to make two changes to <code>stations.go</code> before adding <code>clusterer.go</code> for the station clustering code.</p>
<ul>
<li>
<p>Call <code>clusterStations</code> to cluster the stations retrieved from the R-tree store. The <code>clusterStations</code> function relies on the new parameter zoom passed from the front end, to cluster appropriately for a given zoom level.</p>
</li>
<li>
<p>Increase the size of the R-tree query towards the end of newRect, to prevent the odd behavior of markers mentioned at the end of the previous step.</p>
</li>
</ul>
<h4>stations.go</h4>

```go
package main

import (
        "encoding/json"
        "fmt"
        "log"
        "math"
        "net/http"
        "strconv"
        "strings"

        rtree "github.com/dhconnelly/rtreego"
        geojson "github.com/paulmach/go.geojson"
)

// Stations is an RTree housing the stations
var Stations = rtree.NewTree(2, 25, 50)

// Station is a wrapper for `*geojson.Feature` so that we can implement
// `rtree.Spatial` interface type.
type Station struct {
        feature *geojson.Feature
}

// Bounds implements `rtree.Spatial` so we can load
// stations into an `rtree.Rtree`.
func (s *Station) Bounds() *rtree.Rect {
        return rtree.Point{
                s.feature.Geometry.Point[0],
                s.feature.Geometry.Point[1],
        }.ToRect(1e-6)
}

// loadStations loads the geojson features from
// `subway-stations.geojson` into the `Stations` rtree.
func loadStations() {
        stationsGeojson := GeoJSON["subway-stations.geojson"]
        fc, err := geojson.UnmarshalFeatureCollection(stationsGeojson)
        if err != nil {
                // Note: this will take down the GAE instance by exiting this process.
                log.Fatal(err)
        }
        for _, f := range fc.Features {
                Stations.Insert(&Station{f})
        }
}

// subwayStationsHandler reads r for a "viewport" query parameter
// and writes a GeoJSON response of the features contained in
// that viewport into w.
func subwayStationsHandler(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Content-type", "application/json")
        vp := r.FormValue("viewport")
        rect, err := newRect(vp)
        if err != nil {
                str := fmt.Sprintf("Couldn't parse viewport: %s", err)
                http.Error(w, str, 400)
                return
        }
        zm, err := strconv.ParseInt(r.FormValue("zoom"), 10, 0)
        if err != nil {
                str := fmt.Sprintf("Couldn't parse zoom: %s", err)
                http.Error(w, str, 400)
                return
        }
        s := Stations.SearchIntersect(rect)
        fc, err := clusterStations(s, int(zm))
        if err != nil {
                str := fmt.Sprintf("Couldn't cluster results: %s", err)
                http.Error(w, str, 500)
                return
        }
        err = json.NewEncoder(w).Encode(fc)
        if err != nil {
                str := fmt.Sprintf("Couldn't encode results: %s", err)
                http.Error(w, str, 500)
                return
        }
}

// newRect constructs a `*rtree.Rect` for a viewport.
func newRect(vp string) (*rtree.Rect, error) {
        ss := strings.Split(vp, "|")
        sw := strings.Split(ss[0], ",")
        swLat, err := strconv.ParseFloat(sw[0], 64)
        if err != nil {
                return nil, err
        }
        swLng, err := strconv.ParseFloat(sw[1], 64)
        if err != nil {
                return nil, err
        }
        ne := strings.Split(ss[1], ",")
        neLat, err := strconv.ParseFloat(ne[0], 64)
        if err != nil {
                return nil, err
        }
        neLng, err := strconv.ParseFloat(ne[1], 64)
        if err != nil {
                return nil, err
        }
        minLat := math.Min(swLat, neLat)
        minLng := math.Min(swLng, neLng)
        distLat := math.Max(swLat, neLat) - minLat
        distLng := math.Max(swLng, neLng) - minLng

        // Grow the rect to ameliorate issues with stations
        // disappearing on Zoom in, and being slow to appear
        // on Pan or Zoom out.
        r, err := rtree.NewRect(
                rtree.Point{
                        minLng - distLng/10,
                        minLat - distLat/10,
                },
                []float64{
                        distLng * 1.2,
                        distLat * 1.2,
                })
        if err != nil {
                return nil, err
        }
        return r, nil
}
```

<p>Before getting to the new <code>clusterer.go</code> file, you need to install one more Go package.</p>
<p>Stop the server in the Cloud Shell command line with a <code>[CTRL] + [c]</code>, then run the following command in the Cloud Shell command line:</p>
<pre><code>go get github.com/smira/go-point-clustering&#x000A;</code></pre>
<p>Next, add the code for clustering the stations, by creating a <code>clusterer.go</code> file alongside the other two <code>go</code> files you have created so far. This code is reasonably complicated, as it does some mathematical calculations to figure out appropriate cluster sizes. The code is commented, but don't feel you have to understand exactly what is going on here. The art and science of projecting the map of the surface of an  <a href="http://mathworld.wolfram.com/OblateSpheroid.html">oblate spheroid</a> onto a flat planar projection is covered in detail on  <a href="https://en.wikipedia.org/wiki/Map_projection">Wikipedia</a>.</p>
<h4>clusterer.go</h4>

```go
package main

import (
        "fmt"
        "math"

        rtree "github.com/dhconnelly/rtreego"
        geojson "github.com/paulmach/go.geojson"
        cluster "github.com/smira/go-point-clustering"
)

// The zoom level to stop clustering at
const minZoomLevelToShowUngroupedStations = 14

// Latitude of NYC, used to guestimate the size of a pixel at a specific
// zoom level.
const nycLatitude float64 = 40.7128

// Station marker image width.
const stationMarkerWidth float64 = 28

// EarthRadius is a rough estimate of earth's radius in km at latitude 0
// if earth was a perfect sphere.
const EarthRadius = 6378.137

// Point enables clustering over `Station`s.
func (s *Station) Point() cluster.Point {
        var p cluster.Point
        p[0] = s.feature.Geometry.Point[0]
        p[1] = s.feature.Geometry.Point[1]
        return p
}

func clusterStations(spatials []rtree.Spatial, zoom int) (*geojson.FeatureCollection, error) {
        var pl cluster.PointList

        for _, spatial := range spatials {
                station := spatial.(*Station)
                pl = append(pl, station.Point())
        }
        clusteringRadius, minClusterSize := getClusteringRadiusAndMinClusterSize(zoom)
        // The following operation groups stations determined to be nearby into elements of
        // "clusters". Some stations may end up not part of any cluster ("noise") - we
        // present these as individual stations on the map.
        clusters, noise := cluster.DBScan(pl, clusteringRadius, minClusterSize)
        fc := geojson.NewFeatureCollection()
        for _, id := range noise {
                f := spatials[id].(*Station).feature
                name, err := f.PropertyString("name")
                if err != nil {
                        return nil, err
                }
                notes, err := f.PropertyString("notes")
                if err != nil {
                        return nil, err
                }
                f.SetProperty("title", fmt.Sprintf("%v Station", name))
                f.SetProperty("description", notes)
                f.SetProperty("type", "station")
                fc.AddFeature(f)
        }
        for _, clstr := range clusters {
                ctr, _, _ := clstr.CentroidAndBounds(pl)
                f := geojson.NewPointFeature([]float64{ctr[0], ctr[1]})
                n := len(clstr.Points)
                f.SetProperty("title", fmt.Sprintf("Station Cluster #%v", clstr.C+1))
                f.SetProperty("description", fmt.Sprintf("Contains %v stations", n))
                f.SetProperty("type", "cluster")
                fc.AddFeature(f)
        }
        return fc, nil
}

func getClusteringRadiusAndMinClusterSize(zoom int) (float64, int) {
        // For highest zoom levels, consider stations 10 meters apart as
        // the same.  Allow for groups of size 2.
        if zoom >= minZoomLevelToShowUngroupedStations {
                return 0.01, 2
        }
        groundResolution := groundResolutionByLatAndZoom(nycLatitude, zoom)
        // Multiply ground resolution per pixel by the width (in pixels).. +
        // "manually adjust".
        clusteringRadius := groundResolution * stationMarkerWidth
        // Set min group size to 3
        return clusteringRadius, 3
}

// groundResolution indicates the distance in km on the ground that is
// represented by a single pixel in the map.
func groundResolutionByLatAndZoom(lat float64, zoom int) float64 {
        // number of pixels for the width of the (square) world map in web
        // mercator.  i.e. for zoom level 0, this would give 256 pixels.
        numPixels := math.Pow(2, float64(8+zoom))
        // We return earth's circumference (at given latitude) divided by
        // number of pixels for the map's width.  Note: EarthRadius is given in
        // km.
        return cos(lat) * 2 * math.Pi * EarthRadius / numPixels
}

// cos returns the cosine function (like math.cos) but accepts degrees as input.
func cos(degree float64) float64 {
        return math.Cos(degree * math.Pi / 180)
}
```

<p>Notice the constants at the top of the file, after the usual <code>package</code> line and <code>import</code> stanza. The clusterer uses these constants in various ways.</p>
<ul>
<li>
<code>minZoomLevelToShowUngroupedStations</code> configures the zoom level at which we effectively stop clustering.</li>
<li>
<code>nycLatitude</code> and <code>earthRadius</code> are used in <code>getClusteringRadiusAndMinClusterSize</code> and its helper function <code>groundResolutionByLatAndZoom</code> to approximate the real world size represented by a map pixel, in kilometers.</li>
<li>
<code>stationMarkerWidth</code> is used to figure out how large an area to cluster over.</li>
</ul>
<p>Functions are defined next. We define a <code>Point</code> method over <code>*Station</code> so we can easily cluster stations in <code>clusterStations</code>. The main entry point for this file, <code>clusterStations</code>, is called from <code>stations.go</code> to cluster the stations found in the R-tree. The majority of the code in this function is at the tail end, where we iterate through the generated <code>clusters</code> and individual unclustered stations, labelled as <code>noise</code>. The calling convention of the clusterer package is a little obtuse, returned a list of integers acting as indexes into the original list of points handed in. From these two lists, <code>clusters</code> and <code>noise</code>, we generate a list of <code>geojson.PointFeature</code> that are rendered by the front end.</p>
<p>The three helper functions <code>getClusteringRadiusAndMinClusterSize</code>, <code>groundResolutionByLatAndZoom</code> and <code>cos</code> are taken, with only light edits, from the  <a href="https://github.com/google/election-au-2016">election-au-2016</a> project, an open source project hosted on GitHub that shows the back end that was used to drive  <a href="https://ausvotes.withgoogle.com/">ausvotes.withgoogle.com</a> during the 2016 Australian Federal Election. These helper functions are used to configure the clustering algorithm, called in the middle of <code>clusterStations</code>.</p>
<p>Restart your server:</p>
<pre><code>go fmt *.go &amp;&amp; goapp serve&#x000A;</code></pre>
<p>Refresh the map preview tab again. Stations are elided dynamically by clustering them, depending on zoom level.</p>
<p><img alt="final_map_prev.png" src="https://cdn.qwiklabs.com/YwPZtSc%2BJbRFhckqmYLqBpHDt3htJcXVKOFqRo9DnFw%3D"></p>
<p>You now have a working map that clusters stations and looks good. Wouldn't it be great to show it to your friends? In the next step, publish the application to Google App Engine infrastructure so anyone can see your creation!</p>
<h2 id="step10">Deploying to production</h2>
<p>To show your app off to the world you need to automatically scale the back end, based on the number of concurrent incoming requests. Thankfully, Google App Engine handles this effortlessly. All you need to do is deploy the app.</p>
<p>Shut down the server with a <code>[CTRL] + [c]</code> in the Cloud Shell command line.</p>
<p>Update the <code>main.go</code> file with the following code:</p>
<h4>main.go</h4>

```go
package main

import (
	"io/ioutil"
	"log"
	"net/http"
    "path/filepath"
    "os"
)

// GeoJSON is a cache of the NYC Subway Station and Line data.
var GeoJSON = make(map[string][]byte)

// cacheGeoJSON loads files under data into `GeoJSON`.
func cacheGeoJSON() {
	filenames, err := filepath.Glob("data/*")
	if err != nil {
		// Note: this will take down the GAE instance by exiting this process.
		log.Fatal(err)
	}
	for _, f := range filenames {
		name := filepath.Base(f)
		dat, err := ioutil.ReadFile(f)
		if err != nil {
			log.Fatal(err)
		}
		GeoJSON[name] = dat
	}
}

func main() {
    cacheGeoJSON()
	loadStations()
	http.HandleFunc("/data/subway-stations", subwayStationsHandler)
    http.HandleFunc("/data/subway-lines", subwayLinesHandler)
    http.HandleFunc("/", indexHandler)

    port := os.Getenv("PORT")
    if port == "" {
            port = "8080"
            log.Printf("Defaulting to port %s", port)
    }

    log.Printf("Listening on port %s", port)
    if err := http.ListenAndServe(":"+port, nil); err != nil {
            log.Fatal(err)
    }
}

func subwayLinesHandler(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-type", "application/json")
	w.Write(GeoJSON["subway-lines.geojson"])
}

func indexHandler(w http.ResponseWriter, r *http.Request) {
    if r.URL.Path != "/" {
            http.NotFound(w, r)
            return
    }
}
```

<p>Here we are adding indexHandler responds to requests. The http.ResponseWriter object assembles the HTTP server response; by writing to it, you send data to the browser. The http.Request object is a data structure that represents the incoming HTTP request.
The main function is the entry point of your executable program, so it starts the application.</p>
<p>Update the app.yaml with the following code:</p>

```yaml
runtime: go113

handlers:
- url: /
  static_files: static/index.html
  upload: static/index.html
- url: /(.*\.(js|html|css))$
  static_files: static/\1
  upload: static/.*\.(js|html|css)$
- url: /.*
  script: auto
```

<p>Now, to deploy the app you need to move the application to the GOPATH.
For this, execute the following commands:</p>
<pre><code>cd&#x000A;mv nyc-subway/ gopath/src/&#x000A;cd gopath/src/nyc-subway/&#x000A;</code></pre>
<pre><code>gcloud app deploy&#x000A;</code></pre>
<p>When asked which region to create the App Engine app in, select <code>us-central</code>.</p>
<p><img alt="app_regions.png" src="https://cdn.qwiklabs.com/ivO5PnfjRRGqBKuapFCYPRnD5VnF9shAxS6GuNVrUWU%3D"></p>
<p>Enter "Y" to confirm services to deploy.</p>
<p><img alt="app_confirm.png" src="https://cdn.qwiklabs.com/JZaSOrCDNUtYned%2B1Rsz9Z%2FoF4CSOOvnpkAaKWPWX9w%3D"></p>
<p>View your deployed service at https://[PROJECT_ID].appspot.com . You can also run this command to get the URL:</p>
<pre><code>gcloud app browse&#x000A;</code></pre>
<p>Click the URL to view your deployed service on a new browser tab.</p>
<p><img alt="app_url.png" src="https://cdn.qwiklabs.com/Dfe17B2BSgHgkTS4H36zddzzLl8uWIY7MfSf0QJt48s%3D"></p>
<aside class="warning">
<b>Note:</b> When publishing an application using Google Maps JavaScript API to production for real, please enforce a referer restriction in the API Console for the API key. See the [Get An API Key](https://developers.google.com/maps/documentation/javascript/get-api-key) documentation for more detail.
</aside>
<p>Click <em>Check my progress</em> to verify the objective.
<ql-activity-tracking step="3">
Deploying the app to production
</ql-activity-tracking></p>
<h2 id="step11">Congratulations!</h2>
<p>Your application is deployed and you can view a working map of NY subway stations in your browser.</p>

<p>This self-paced lab is part of the Qwiklabs <a href="https://google.qwiklabs.com/quests/39">Website and Web Application</a> and <a href="https://google.qwiklabs.com/quests/103">Creating with Google Maps</a> Quest. A Quest is a series of related labs that form a learning path. Completing this Quest earns you the badges above, to recognize your achievement. You can make your badge (or badges) public and link to them in your online resume or social media account. Enroll in a Quest and get immediate completion credit if you've taken this lab. See <a href="https://google.qwiklabs.com/catalog">other available Qwiklabs Quests</a>.</p>
<h3>Take your next lab</h3>
<p>Continue your Quest with  <a href="https://google.qwiklabs.com/catalog_lab/1117">Using Ruby on Rails with Cloud SQL for PostgreSQL</a>, or check out these suggestions:</p>
<ul>
<li>
<p><a href="https://google.qwiklabs.com/catalog_lab/2253">Use Google Maps API to Visualize BigQuery Geospacial Data</a></p>
</li>
<li>
<p><a href="https://google.qwiklabs.com/catalog_lab/1118">Google Maps Web Services Proxy for Mobile Applications</a></p>
</li>
</ul>
