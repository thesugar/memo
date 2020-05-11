# 実践サーバーサイドプログラミング（3）
[Source](https://www.nnn.ed.nico/courses/497/chapters/6891)

## § 15. 「予定調整くん」の設計
ここまで、Web プログラミングに関する様々な知識を身につけてきた。ここからは入門コースの集大成となる Web サービスを開発していく。

入門コースの仕上げとして、スケジュール調整サービスを作る。  
スケジュール調整サービスの基本的な要求は一つ。

- できるだけ多くの人が参加できる適切な予定日時を決定したい

以上である。  
具体的なユースケースとしては、

- GitHub アカウントを知っている仲間内で、オフ会の日程を決定する

など。**ユースケース** とは、実際にシステムがユーザーに利用される際のやりとりのことを言う。

まず最初にこのサービスの名前とプロジェクト名を決める。  
サービスの名前は「予定調整くん」で、プロジェクト名は `scheduke-arranger` とする。

### 「予定調整くん」の要件定義
予定調整くんに必要な機能は以下のとおりとする。

- 予定を作れる
- 予定に候補（候補日）が作れる
- 予定の候補に対して出欠を編集できる
- 予定に対してコメントが編集できる
- 予定を削除できる
- 予定を編集できる

以上のような要件は **機能要件** とも呼ばれ、要求を満たすための機能があることを定義したものとなっている。

機能要件でないものを **非機能要件** という。非機能要件には、機能に付随する性能に対する要件や、セキュリティに関する要件などがある。

### 「予定調整くん」で出てくる用語を定義する
それでは、要件にあがった用語を細かく定義していく。

|用語|英語表記|意味|
|--|--|--|
|ユーザー|user|予定調整くんの利用者|
|予定|schedule|複数のユーザーの出欠が必要なイベント|
|候補|candidate|予定の開催の日時および期間の候補、つまり候補日程|
|出欠|availability|予定の候補に対する出欠意思で、「欠席・わからない・出席」が選択できる|
|コメント|comment|予定に対してユーザーがつけるコメント|

以上のように、この「予定調整くん」野中に登場する言葉とその意味を定義する。

このように、システムの中に登場する用語と意味をしっかり定義するということは、ソフトウェアづくりの中で非常に重要！

### データモデリング
今度はそれぞれのデータモデリングを行い、データモデルを作成しよう。

先ほど紹介した用語はすべて、データベース上におけるエンティティとみなしてよいだろう。

エンティティ同士の関係を表すと以下の ER 図のようになる。

<img src="./images/schedule-arranger-er.png">

主キーや外部キーなどの属性はすべて省いて、エンティティ名だけで書いている。  
一番中心となるエンティティはユーザー。  
ユーザーは、予定、出欠、コメントとそれぞれ「1 対 多」の関連を持っている。

- ユーザーは、予定を作成できる
- ユーザーは、出欠を編集できる
- ユーザーは、コメントを編集できる

以上の要件があるため、この関係は妥当だと言える。  
またこの関係から、予定、出欠、コメントはそれぞれ、ユーザーに従属している。

そのためこれらの関係を表す線には、 `1` と 0 から N を表す `0..n` が記載されている。この関係は「1 対 多」の関連における従属を表す ER 図の線の書き方である。

次に予定は、候補とコメントとそれぞれ「1 対 多」の関連を持っている。これも、

- 予定には、候補がある
- 予定には、ユーザーによってコメントが編集できる

これらの要件を満たすために必要な要件であると言える。同様に、候補とコメントは予定に従属している。

そして候補は、出欠と「1 対 多」の関連を持っている。

- 候補に対して、ユーザーは出欠を編集できる

という要件があるためこれも必要だろう。  
同様に出欠は候補が存在しないと記入できないため、これも出欠が候補の従属エンティティだということができるだろう。

さて、このようにデータモデルができたところで、先ほどの基本的な要件（「予定を作れる」など）が満たされるかを確認すると、どうやらこのデータモデルでよさそうだとわかる。

なお、このようにデータモデルなどの設計を行っているが、実際に実装で都合が悪いところが出てきたら、それにあわせてデータモデルはへんこう　していく。

このような設計は、設計どおりにしっかりと作ることが重要なのではなく、設計をすることによって、要件に漏れがないかといったことや、根本的な仕組みに問題がないかをチェックすることが目的である。

したがって、設計をしたら感あらずこの通りに作らなくてはいけない、というわけではない。

### URL 設計
次に要件を満たすための URL 設計を行おう。  
まずは内容を表示するページ構成を考える。

- トップページ
- 自分が作った予定の一覧表示ページ
- 予定表示ページ
- 予定作成ページ
- 予定編集ページ
- 出欠表ページ
- コメントページ

思いつくままに挙げてみた。ログインとログアウトのページは省いている。

ただし、機能的に一緒であっても問題なさそうなページがありそうなので、まとめてみる。

- トップ/自分が作った予定の一覧表示ページ
- 予定表示/出欠表/コメントページ
- 予定作成ページ
- 予定編集ページ

このようにまとめられそうである。  
トップページでは、自分が作った予定の一覧をログイン時とログアウト時で出し分けるように実装すればよさそう。

予定の内容、出欠表、コメントは同じページ内にまとめて出欠やコメントの編集を Ajax を利用して編集するとより利便性が上がる。

予定の作成と編集ページは、普通の HTML のフォームdえ実現できそう。

これらを踏まえて、メソッドと URL のパス、内容をまとめると以下の表のようになる。

#### ページの URL 一覧

|パス|メソッド|ページ内容|
|--|--|--|
|/|GET|トップ/自分が作った予定の一覧表示ページ|
|/schedules/new|GET|予定作成ページ|
|/schedules/:scheduleId|GET|予定表示/出欠表/コメントページ|
|/schedules/:scheduleId/edit|GET|予定編集ページ|
|/login|GET|ログイン|
|/logout|GET|ログアウト|

以上のようなページの URL で表現できそう。  
表のパスに出てきている `:scheduleId` は、予定エンティティの主キーとする。

続けて、ページではなく、フォームの投稿先や Ajax で利用する Web API の URL も設計してしまおう。

#### Web API の URL 一覧

|処理内容|パス|メソッド|利用方法|
|--|--|--|--|--|
|予定作成|/schedules|POST|フォーム|
|予定編集|/schedules/:scheduleId?edit=1|POST|フォーム|
|予定削除|/schedules/:scheduleId?delete=1|POST|フォーム|
|出欠編集|/schedules/:scheduleId/users/:userId/candidates/:candidateId|POST|Ajax|
|コメント編集|/schedules/:scheduleId/users/:userId/comments|POST|Ajax|

以上のようにまとめられる。メソッドはすべて POST とした。

なおここに登場する `:userId` はユーザーエンティティの主キー、`:candidateId` は候補エンティティの主キーとする。

このようにデータモデルと URL 設計をすることで、Web サービスの全貌が見えてくる。

ただし、特に URL 設計に関しては、適切な URL 設計が実装前にはできないことも多い。セキュリティの要件やパフォーマンス上の要件などの技術的制約により URL 設計が影響をうけることがあり、それらは詳細な実装の前には想定しづらいためである。

そのため、場当たり的にどんどん作っていきたいという考えもあるが、とはいえ何も考えなしに作成していくと全体の URL 設計の統一がなされず非常にわかりづらい URL 構成になってしまいがち。

以上のように大枠の URL 設計を経ることによって、全体として統制が取れ、わかりやすい URL を設計することができる。

なお Ajax などで利用する Web API の設計には様々な設計思想やツールがあるが、ここでは紹介を割愛する。

### モジュール設計
ここまではインターフェースの設計であった。  
ここからはモジュールの設計も行っていく。

ただし Web フレームワークの Express を利用する前提に立つことでずいぶんとモジュール設計コストを下げることができる。

URL 設計にそって、`/routes` ディレクトリ以下の Router モジュールを以下のように用意すればよい。

### Router モジュール一覧
|ファイル名|責務|
|--|--|
|routes/login.js|ログイン処理|
|routes/logout.js|ログアウト処理|
|routes/schedules.js|予定に関連する処理|
|routes/availabilities.js|出欠の更新に関する処理|
|routes/comments.js|コメントの更新に関する処理|

このように責務を分割しよう。

それぞれの責務は独立性と凝集性が高く、分離しやすいため、上記のようなモジュール構成で作ることにより、効率的に開発することができる。

このように Web フレームワークを用いることで、お決まりのモジュール構成の形式にしたがって設計を考えられるため、設計の段階でもずいぶん楽をすることができる。

なお、データモデルやデータストアへの永続化に関しては、`/models` というディレクトリを作成し、それぞれのエンティティごとにファイルを定義する。

### データモデル一覧

|ファイル名|責務|
|--|--|
|models/user.js|ユーザーの定義と永続化|
|models/schedule.js|予定の定義と永続化|
|models/candidate.js|候補の定義と永続化|
|models/availability.js|出欠の定義と永続化|
|models/comment.js|コメントの定義と永続化|

今回は永続化には sequelize を用いる。  
上記の表では、データモデルで決定したエンティティの設計に沿ってモジュール分割を行っている。

これで JavaScript で利用するモジュールは `/routes` と `/models` ディレクトリ以下にそれぞれモジュール分割することができた。

なお、このように、リクエストのルーティングを行い処理をコントロールする部分と、エンティティのモデリングや振る舞い、永続化などの処理をする部分と、表示内容の形式を定義する部分を分割する構造のことを MVC という。

### MVC
<img src="./images/mvc.png">

M は Model、すなわちデータモデルであり、ここでいう `/models` ディレクトリ以下のモジュールである。  
V は View、ここでは `/views` ディレクトリ以下のビューという見た目を司る pug テンプレートとなる。  
C は Controller、Express では `/routes` ディレクトリ以下の Router モジュールを表す。

なお世の中には、以上のような MVC の構成をデフォルトで提供している Web フレームワークも存在する。

ただし、MVC は、いくつか派生したアイデアがさまざまな書籍で紹介されていることもあり、 M, V, C それぞれの責務や依存関係のあり方については多くの解釈がある。

そのため共通項としては、M, V, C のそれぞれのモジュールに分割された構造を持つ設計だという解釈でよかろう。

なお、このように、モデル、コントローラー、ビューをそれぞれモジュールとして分割して、 インタフェースを制限することで、 それぞれのモジュールの交換性能を高め、変更時の影響を少なくできるというメリットがある。

半面、モジュール分割しない場合よりも、モジュール間のインタフェースの定義の実装量は増えてしまいがち。

なお今回の構成では、ビューはモデルに依存していますが、モデルはビューに依存しないため、 テンプレートエンジンを仮に変えたとしても、`/models` 以下のモジュールに変更を加えることなく利用し続けることができるというメリットがある。

以上で、予定調整くんの設計は完了となる。

💡 ここで、GitHub 認証を使うためのアプリケーションを作成しておこう。

GitHub へのログイン後 `https://github.com/settings/applications/new` にアクセスし、

- Application name を、予定調整くん（開発用）
- Homepage URL を、`http://localhost:8000/`
- Application description を、GitHub 認証を利用して予定を調整してくれるアプリケーション
- Authorization callback URL を、 `http://localhost:8000/auth/github/callback`

以上に設定してアプリケーション登録をする。

## § 16. 認証の実装とテスト
ここまでで「予定調整くん」の設計ができた。  
続けて実装をしていく。この回ではまずプロジェクトを作成し、認証と必要な router モジュールを用意していく。

### 開発の準備

```bash
cd ~/workspace
express --view=pug schedule-arranger
cd schedule-arranger
yarn install
```

上記コマンドを実行することで express の雛形が作成され、必要な npm パッケージがインストールされる。そうしたら早速起動して、雛形が問題なく利用できることを確かめよう。

```bash
PORT=8000 yarn start
```

以上でサーバーを起動してから `http://localhost:8000` にアクセスし、Welcome to Express という文言が表示されれば問題ない。

次にこのディレクトリを Git 管理できるようにしておく。

```bash
echo "node_modules/" > .gitignore
git init
git add .
git commit -m "first commit"
```

以上を入力して、いったんここまでの実装をコミットしておこう。

### X-Powered-By ヘッダの除去
まずは、セキュリティ対策のために helmet モジュールをインストールしておく。

```bash
yarn add helmet@3.8.2
```

インストールしたら、以下の変更を `app.js` に加える。

<details close>
<summary>app.js</summary>

```diff
 var cookieParser = require('cookie-parser');
 var logger = require('morgan');
+var helmet = require('helmet');

 var indexRouter = require('./routes/index');
 var usersRouter = require('./routes/users');

 var app = express();
+app.use(helmet());

 // view engine setup
 app.set('views', path.join(__dirname, 'views'));
 ```

</details>

実装が終わったら、再度アクセスして Chrome のデベロッパーツールの Network タブを開いて再読込を行い、レスポンスヘッダを確認して、`X-Powered-By` が送られてきていないことを確認しよう。

### Router モジュールのファイルの作成
次に、作成することがわかっている Router モジュールの JavaScript ファイルを作成する。

```bash
touch routes/login.js
touch routes/logout.js
touch routes/schedules.js
touch routes/availabilities.js
touch routes/comments.js
```

以上のコマンドを実行して、`routes/` 以下にファイルだけを作成してしまおう。

### GitHub 認証の実装
引き続き GitHub 認証の実装を行っていく。まずは必要なモジュールをインストール。

```bash
yarn add passport@0.3.2
yarn add passport-github2@0.1.9
yarn add express-session@1.13.0
```

以上がインストールできたら、前のセクションの最後の GitHub のアプリ登録画面のところに表示されている Client ID と Client Secret を用意し、`app.js` を以下のように実装していく。

<details close>
<summary>app.js</summary>

```diff
 var cookieParser = require('cookie-parser');
 var logger = require('morgan');
 var helmet = require('helmet');
+var session = require('express-session');
+var passport = require('passport');
+
+var GitHubStrategy = require('passport-github2').Strategy;
+var GITHUB_CLIENT_ID = '2f831cb3d4aaXXXXXXXXX'; // 自分の値を使う
+var GITHUB_CLIENT_SECRET = '9fbc340ac0175123695d2dedfbdf5aXXXXXXXXX'; // 自分の値を使う
+
+passport.serializeUser(function (user, done) {
+  done(null, user);
+});
+
+passport.deserializeUser(function (obj, done) {
+  done(null, obj);
+});
+
+
+passport.use(new GitHubStrategy({
+  clientID: GITHUB_CLIENT_ID,
+  clientSecret: GITHUB_CLIENT_SECRET,
+  callbackURL: 'http://localhost:8000/auth/github/callback'
+},
+  function (accessToken, refreshToken, profile, done) {
+    process.nextTick(function () {
+      return done(null, profile);
+    });
+  }
+));
 
 var indexRouter = require('./routes/index');
 var usersRouter = require('./routes/users');
```
</details>

`app.js` をさらに以下のように編集する。

<details close>
<summary>app.js</summary>

```diff
 app.use(cookieParser());
 app.use(express.static(path.join(__dirname, 'public')));
 
+app.use(session({ secret: 'e55be81b307c1c09', resave: false, saveUninitialized: false }));
+app.use(passport.initialize());
+app.use(passport.session());
+
 app.use('/', indexRouter);
 app.use('/users', usersRouter);
```
</details>

この変更における `session` 関数に渡すオブジェクトの `secret` の値は、`node -e "console.log(require('crypto').randomBytes(8).toString('hex'));` コマンドで自分用に新たに生成された値を利用すること。

以上で passport の設定は完了。

次に GitHub 認証に必要な Router オブジェクトなどを作成していく。  
まず、`app.js` に以下の変更を加える。

<details close>
<summary>app.js</summary>

```diff
 var indexRouter = require('./routes/index');
-var usersRouter = require('./routes/users');
+var loginRouter = require('./routes/login');
+var logoutRouter = require('./routes/logout');
 
 var app = express();
 app.use(helmet());
```
</details>

まず、不要な `users` ルーターモジュールの読み込みを除去して、`login` と `logout` のルーターを読み込むようにしている。

`app.js` をさらに以下のように編集する。

<details close>
<summary>app.js</summary>

```diff
 app.use(passport.session());
 
 app.use('/', indexRouter);
-app.use('/users', usersRouter);
+app.use('/login', loginRouter);
+app.use('/logout', logoutRouter);
+
+app.get('/auth/github',
+  passport.authenticate('github', { scope: ['user:email'] }),
+  function (req, res) {
+});
+
+app.get('/auth/github/callback',
+  passport.authenticate('github', { failureRedirect: '/login' }),
+  function (req, res) {
+    res.redirect('/');
+});
 
 // catch 404 and forward to error handler
 app.use(function (req, res, next) {
```
</details>

加えて、`routes/login.js` を以下のように実装し、

<details close>
<summary>login.js</summary>

```js
'use strict';
const express = require('express');
const router = express.Router();

router.get('/', (req, res, next) => {
  res.render('login', { user: req.user });
});

module.exports = router;
```
</details>

`routes/logout.js` を以下のように実装する。

<details close>
<summary>logout.js</summary>

```js
'use strict';
const express = require('express');
const router = express.Router();

router.get('/', (req, res, next) => {
  req.logout();
  res.redirect('/');
});

module.exports = router;
```
</details>

また、views フォルダの中に `login.pug` ファイルを以下の内容で作成する。

<details close>
<summary>views/login.pug</summary>

```pug
extends layout

block content
  a(href="/auth/github") GitHubでログイン
  if user
     p 現在 #{user.username} でログイン中

```
</details>

なお、もう不要となる `routes/users.js` は削除して構わない。

ここまできたら、サーバーを再起動し、`http://localhost:8000/login` にアクセスして、ログインできるか試してみよ。

また、同じ URL に再度アクセスして、ユーザー名が表示されていることも確認すること。それもうまくいったら、`http://localhost:8000/logout` にアクセスしてログアウトし、 `/login` に再度アクセスしてユーザー名が表示されなくなることも試してみよう。

### Router オブジェクトをテストする
ここまd実装できたところでこの内容をテストするテストコードをじっそう　していく。

まずはテストに必要なモジュールをインストールする。

```bash
yarn add mocha@4.0.1 --dev
yarn add supertest@3.1.0 --dev
yarn add passport-stub@1.1.1 --dev
```

`supertest` は、Express の Router オブジェクトをテストするモジュール。この supertest は、テスト内で Express のサーバーを起動することなく、Router の挙動をテストできる。

また、テストする際に、passport の認証が邪魔になってしまう場合がある。  
その passport の挙動をコントロールするために、passport-stub というモジュールもここでは利用する。  
この passport-stub は、GitHub 認証のログイン・ログアウト処理をテスト内で模倣することができる。

仕上げに package.json を以下のように実装して、`yarn test` コマンドが利用できるようにしよう。

<details close>
<summary>package.json</summary>

```diff
   "version": "0.0.0",
   "private": true,
   "scripts": {
-    "start": "node ./bin/www"
+    "start": "node ./bin/www",
+    "test": "node_modules/mocha/bin/mocha --timeout 10000"
   },
   "dependencies": {
     "cookie-parser": "~1.4.3",
```

環境によってはテストに時間がかかり、タイムアウトしてしまい正常にテストできないことを考慮し、`--timeout 10000` を追加することでタイムアウトまでの時間をデフォルトの 2000 ミリ秒から 10000 ミリ秒（10 秒）に延ばしている。
</details>

引き続き mocha のためのテストコードを実装する。

```bash
mkdir test
touch test/test.js
```

### リクエストのテスト
テストコードを記述するファイルができたら、`test/test.js` を以下のように実装してみよう。

<details close>
<summary>test.js</summary>

```js
'use strict';
const request = require('supertest');
const app = require('../app');

describe('/login', () => {

  it('ログインのためのリンクが含まれる', (done) => {
    request(app)
      .get('/login')
      .expect('Content-Type', 'text/html; charset=utf-8')
      .expect(/<a href="\/auth\/github"/)
      .expect(200, done);
  });

});
```
</details>

以上の記法が、supertest のテストの記法になる。

`request(app).get('/login')` で、 `/login` への GET リクエストを作成する。

その後、 `expect` 関数に、文字列を 2 つ引数として渡し、ヘッダにその値が存在するかをテストしている。

`expect` 関数に、正規表現を一つ渡すと、 HTML の body 内にその正規表現が含まれるかをテストする。

そしてテストを終了する際には、 `expect` 関数に、期待されるステータスコードの整数と、テスト自体の引数に渡される `done` 関数を渡す。

なお以上のテストは、`/login` にアクセスした際に、

- レスポンスヘッダの 'Content-Type' が `text/html; charset=utf-8` であること
- `<a href="/auth/github"` が HTML に含まれること
- ステータスコードが 200 OK で返ること

以上の 3 つの仕様をテストしている。実装ができたら、`yarn test` でテストを実行。「1 passing」という結果が表示されれば成功。

### 認証時のテスト
次に、ログインしたときには `/login` にユーザー名が表示されることをテストしてみる。

passport-stub を利用して、`test/test.js` を以下のように実装する。

<details close>
<summary>test.js</summary>

```diff
 const app = require('../app');
+const passportStub = require('passport-stub');
 
 describe('/login', () => {
+  before(() => {
+    passportStub.install(app);
+    passportStub.login({ username: 'testuser' });
+  });
+
+  after(() => {
+    passportStub.logout();
+    passportStub.uninstall(app);
+  });
```
</details>

あともう一つ、

<details close>
<summary>app.js</summary>

```diff
       .expect(200, done);
   });
 
+  it('ログイン時はユーザー名が表示される', (done) => {
+    request(app)
+      .get('/login')
+      .expect(/testuser/)
+      .expect(200, done);
+  });
 });
```
</details>

このように実装する。

```js
  before(() => {
    passportStub.install(app);
    passportStub.login({ username: 'testuser' });
  });

  after(() => {
    passportStub.logout();
    passportStub.uninstall(app);
  });
```

上記の実装は、`describe` 以下のテストを実行する前と後に実行したい処理を記述している。

`before` 関数で記述された処理は `describe` 内のテスト前に、`after` 関数で記述された処理は `describe` 内のテスト後に実行される。  
これは mocha の機能で、他にも各 `it` 内のテストのぜんご　に実行される処理も記述できる。

ちなみにここでは、テストの前に `passportStub` を `app` オブジェクトにインストールし、`testUser` というユーザー名のユーザーでログインしている。  
テストの後は、ログアウトして、アンインストールする処理を実行している。

```js
  it('ログイン時はユーザー名が表示される', (done) => {
    request(app)
      .get('/login')
      .expect(/testuser/)
      .expect(200, done);
  });
```

以上のコードでは、`/login` にアクセスした後、その HTML の body 内に `testuser` という文字列が含まれることをテストしている。

加えて、「`/logout` にアクセスした際に、 `/` にリダイレクトされる」という仕様もテストする。  
`/` へのリダイレクトは `.expect('Location', '').expect(302, done)` という処理を記述することでテストできる。

<details close>
<summary>test.js</summary>

```diff
   });
 });

+describe('/logout', () => {
+  it('/ にリダイレクトされる', (done) => {
+    request(app)
+      .get('/logout')
+      .expect('Location', '/')
+      .expect(302, done);
+  });
+});
```
</details>

ここまで実装ができたら `yarn test` でテストを実行し、ログインした時の挙動がちゃんとテストされていることを確認する。

このように Express の Router モジュールもテスト可能であり、GitHub の外部認証を用いなくても認証が関連するテストを実行することができる。  
これからは実装していくさまざまな機能を、このテストの機能で検証しながら実装を行っていく。

## § 17. ユーザーの保存
前回までで GitHub 認証と Router オブジェクトのテストができるようになった。  
次はモデルの実装を行って、それを使ってデータの保存を行ってみよう。

モデルの実装には、データベースとして PostgreSQL を、データベースを扱うためのライブラリとして Node.js の sequelize モジュールを利用する。

実装するモデルは以下の 5 つ。以下 5 つの詳細なデータモデリングと実装をしていこう。

- user
- schedule
- candidate
- availability
- comment

### ユーザー（user）のデータモデリング
まずは user。このデータモデルの役割は、ユーザー名を永続化することである。  
現在セッション内にユーザーのデータは保存されているが、サーバーを再起動したりした際にはそのデータは消えてしまう。

そのためユーザーの存在確認やデータ表示時にユーザー名を利用する場合には、この Web サービスのデータベースに保存する必要がある。

GitHub の認証の後、`req.user` オブジェクトにはどのようなプロパティがあるのだろうか。

[passport-github](https://github.com/cfsghost/passport-github) の[ソースコード](https://github.com/cfsghost/passport-github/blob/10d2ed58a6513fa8f27c3a230914cf578a4a831e/lib/strategy.js#L78) を確認すると、

```js
/**
 * Retrieve user profile from GitHub.
 *
 * This function constructs a normalized profile, with the following properties:
 *
 *   - `provider`         always set to `github`
 *   - `id`               the user's GitHub ID
 *   - `username`         the user's GitHub username
 *   - `displayName`      the user's full name
 *   - `profileUrl`       the URL of the profile for the user on GitHub
 *   - `emails`           the user's email addresses
 *
 * @param {String} accessToken
 * @param {Function} done
 * @api protected
 */
```

コメントにより以上のようなプロパティが `req.user` オブジェクトに存在していることがわかる。

ここで必要になるのは、

- GitHub のユーザー ID である `id`
- GitHub のユーザー名である `username`

だけとなりそうである。

|user の属性名|形式|内容|
|--|--|--|
|userId|数値|GitHub のユーザー ID、主キー|
|username|文字列|GitHub のユーザー名|

*（ここで言う数値型の GitHub のユーザー ID とはなんのことか正直わからないが、実際にこのセクションを終えた時点で、データベースに保存されているユーザー情報を確認するとたしかに「userid: 53966025, username: thesugar」っていう感じで数値で保存されている。。）*

### ユーザー（user）の実装
実装にあたって、まずは sequelize と PostgreSQL 関連モジュールをインストールする。

```bash
yarn add sequelize@5 # 教材では @4.33.4 としているが古いので 5 をインストールする
yarn add pg@7.4.1
yarn add pg-hstore@2.3.2
```

無事インストールできたら、PostgreSQL のデータベースも作る。

```sql
sudo su - postgres
psql

CREATE DATABASE schedule_arrranger;
```

以上を入力して `CREATE DATABASE` と表示されれば成功。

`\q` -> `exit` で psql を終了させたのち psotgres ユーザーのセッションを終了させたら、次は user のモデルを実装するためのファイルを作成する。

```bash
mkdir models
touch models/sequelize-loader.js
touch models/user.js
```

複数のモデルをそれぞれ別のファイルに記述したいので、`models/sequelize-loader.js` という sequelize の読み込みの定義を書く部分を別ファイルとした。

まずは `models/sequelize-loader.js` を以下のように実装しよう。

<details close>
<summary>sequelize-loader.js</summary>

```js
'use strict';
const Sequelize = require('sequelize');
const sequelize = new Sequelize(
  'postgres://postgres:postgres@localhost/schedule_arranger',
  // ↓ この `{...}` は sequelize ver.5 ではもはや記述自体不要
  {
    operatorsAliases: false
  });

module.exports = {
  database: sequelize,
  Sequelize: Sequelize
};
```
</details>

今回は、sequelize が出力する SQL などもログで見たいので、ログが出力される設定にしてある（ログを出力しない場合は、`new Sequelize('postgres://...', { /* ここ */ })` の第二引数（`/* ここ */`）に `logging:false` を設定する）。  
DB の URL は先ほど作成した DB のもの。

次に先ほどのデータモデル通りに、sequelize の記法の定義に沿って `models/user.js` を以下のように実装しよう。

なお、それぞれのデータモデルとテーブルの対応は、以下のように実装する。

|データモデルの実装|テーブル名|
|--|--|
|models/user.js|users|
|models/schedule.js|schedules|
|models/candidate.js|candidates|
|models/availability.js|availabilities|
|models/comment.js|comments|

これにより、ユーザーのデータモデルは以下のようになる。

<details close>
<summary>models/user.js</summary>

```js
'use strict';
const loader = require('./sequelize-loader');
const Sequelize = loader.Sequelize;

const User = loader.database.define('users', {
  userId: {
    type: Sequelize.INTEGER,
    primaryKey: true,
    allowNull: false
  },
  username: {
    type: Sequelize.STRING,
    allowNull: false
  }
}, {
    freezeTableName: true,
    timestamps: false
  });

module.exports = User;
```

userId は Sequelize.INTEGER というデータ型、username は Sequelize.STRING というデータ型にし、 どちらも、 null 値を許可しない設定にした。   
なおデータモデルの `sync` 関数が呼ばれた際に、これらの設定にもとづいて SQL の CREATE TABLE が実行され、データベースとの対応が取れるようになる。

</details>

これで user の実装は完了。

### 予定（schedule）のデータモデリング
次に schedule の属性を考える。まず、主キーにする ID が必要となりそう。また、予定名は必ず必要であろう。  
他には、予定に関して何かしらの状態をコメントするために、メモを付けられるようにしておく。  
あとは、自分が作成した予定の一覧を作るために、その予定を誰が作ったのかという情報も必要。

以上をまとめると、以下のようになる。

|schedule の属性名|形式|内容|
|--|--|--|
|scheduleId|数値|予定 ID、主キー、連番で付けられる|
|scheduleName|文字列|予定名|
|memo|文字列|メモ|
|createdBy|数値|作成者、ユーザー ID|

しかし、実はこのデータモデルにはセキュリティ上の問題がある。予定を表示する URL は、

```
/schedules/:scheduleId
```

となるが、予定の URL は、予定の関係者以外には秘密にしたいことがあるかもしれない。  
連番の予定 ID を用いた場合、簡単に予定の存在や URL を推測されてしまう。これはセキュリティ上の問題になりうる。

つまり、秘密にしたい ID は推測されづらいものである必要がある。

このようなときに便利な ID がある。**UUID** という。

### UUID
UUID とは、Universally Unique Identifier の略称で、全世界で同じ値を持つことがない一意の識別子のことである。  
UUID の生成の実装は、様々なプログラミング言語で提供されており、Node.js でも uuid というモジュールが利用可能。

好きなタイミング・好きな場所でこの UUID は発行できる。  
また、十分に衝突しないだけのデータ領域が確保されているため、かぶることは滅多にない。

なお、実際に生成される UUID は、文字列で表した場合は `fd70bde2-504f-4ae9-95fd-46727b7d224b` のようなランダムな 16 進数の文字列になる。

そして PostgreSQL などのデータベースではこの UUID を 128 ビットの値として格納するための UUID 型というデータ型が提供されている。

この UUID を scheduleId に用いることによって、スケジュールの存在が URL アクセスによって推測されないような工夫をしてみよう。

ただし、この UUID はランダムな値になるため、自分が作成した予定一覧を ID を使って並べ替えると、ランダムな順番になってしまう。  
更新日時の降順で予定が並ぶように、更新日時も属性として加えよう。

以上を加味すると、schedule は以下のようなデータモデルがよさそう。

|schedule の属性名|形式|内容|
|--|--|--|
|scheduleId|**UUID**|予定 ID、主キー、<s>連番で付けられる</s>|
|scheduleName|文字列|予定名|
|memo|文字列|メモ|
|createdBy|数値|作成者、ユーザー ID|
|updatedAt|日時|更新日時|

### 予定（schedule）の実装
このデータモデルを実際に実装してみる。

`models/schedule.js` を以下のように実装する。

<details close>
<summary>models/schedule.js</summary>

```js
'use strict';
const loader = require('./sequelize-loader');
const Sequelize = loader.Sequelize;

const Schedule = loader.database.define('schedules', {
  scheduleId: {
    type: Sequelize.UUID,
    primaryKey: true,
    allowNull: false
  },
  scheduleName: {
    type: Sequelize.STRING,
    allowNull: false
  },
  memo: {
    type: Sequelize.TEXT,
    allowNull: false
  },
  createdBy: {
    type: Sequelize.INTEGER,
    allowNull: false
  },
  updatedAt: {
    type: Sequelize.DATE,
    allowNull: false
  }
}, {
    freezeTableName: true,
    timestamps: false,
    indexes: [
      {
        fields: ['createdBy']
      }
    ]
  });

module.exports = Schedule;
```

memo は長さに制限のない文字列として設定してある。  
また、全ての値で null を許容しない設定とした。

</details>

### 候補日程（candidate）のデータモデリング
次は、candidate の属性を考える。  
候補日程は、連番の候補日程 ID を振ってしまっても問題なさそう。  
また、候補日程名を設定できるようにしよう。そして、データ取得時に利用できるように、予定 ID も属性として必要である。

|candidate の属性名|形式|内容|
|--|--|--|
|candidateId|数値|候補日程 ID、主キー、連番で付けられる|
|candidateName|文字列|候補日程名|
|scheduleId|UUID|関連する予定 ID|

候補日程名には、`12/24 12:00〜` のような値も、`1/3 ~ 1/5 のどこか` のような値も `来年` のような値も受け入れられるように、文字列としてある。

### 候補日程（candidate）の実装

<details close>
<summary>models/candidate.js</summary>

```js
'use strict';
const loader = require('./sequelize-loader');
const Sequelize = loader.Sequelize;

const Candidate = loader.database.define('candidates', {
  candidateId: {
    type: Sequelize.INTEGER,
    primaryKey: true,
    autoIncrement: true,
    allowNull: false
  },
  candidateName: {
    type: Sequelize.STRING,
    allowNull: false
  },
  scheduleId: {
    type: Sequelize.UUID,
    allowNull: false
  }
}, {
    freezeTableName: true,
    timestamps: false,
    indexes: [
      {
        fields: ['scheduleId']
      }
    ]
  });

module.exports = Candidate;
```

今回は、予定 ID （`scheduleId`）で大量のデータから検索されることが想定されるので、予定 ID（`scheduleId`）にはインデックスを貼ってある。

</details>

### 出欠（availability）のデータモデリング
出欠のデータは

- 候補日程 ID
- ユーザー ID

この 2 つによって一意になるはず。あとは「欠席」「わからない」「出席」で表される出欠内容、そして、検索性を高めるために予定 ID を一緒に入れておけばよさそう。

|availability の属性名|形式|内容|
|--|--|--|
|candidateId|数値|候補日程 ID、主キー|
|userId|数値|GitHub のユーザー ID、主キー|
|availability|数値|出欠の種類|
|scheduleId|UUID|関連する予定 ID|

上の表には、主キーとして candidateId と userId の 2 つを使っている。  
実は主キーは複数の属性で設定することもできる。  
このように 2 つの属性で主キーを京成したものを **複合主キー** という。

複合種キーにするべき属性の数が増えた際には、代わりの主キーとなる大力ーを設けることがあるが、ここでは、複合主キーのまま利用してみよう。

### 出欠（availability）の実装

<details close>
<summary>models/availability.js</summary>

```js
'use strict';
const loader = require('./sequelize-loader');
const Sequelize = loader.Sequelize;

const Availability = loader.database.define('availabilities', {
  candidateId: {
    type: Sequelize.INTEGER,
    primaryKey: true,
    allowNull: false
  },
  userId: {
    type: Sequelize.INTEGER,
    primaryKey: true,
    allowNull: false
  },
  availability: {
    type: Sequelize.INTEGER,
    allowNull: false,
    defaultValue: 0
  },
  scheduleId: {
    type: Sequelize.UUID,
    allowNull: false
  }
}, {
    freezeTableName: true,
    timestamps: false,
    indexes: [
      {
        fields: ['scheduleId']
      }
    ]
  });

module.exports = Availability;
```

ここでは、 candidateId と userId がそれぞれ主キーとして設定してある。  
また、検索のために、 scheduleId でインデックスが利用できるようにしてある。

</details>

### コメント（comment）のデータモデリング
コメントは予定に対して付けられるコメントであるため、

- 予定 ID
- ユーザー ID

以上の二つで複合主キーを設定すればよさそう。また、コメントの内容も属性として必要になる。

これらを考慮すると、以下のようになる。

|comment の属性名|形式|内容|
|--|--|--|
|scheduleId|UUID|関連する予定 ID、主キー|
|userid|数値|GitHub のユーザー ID、主キー|
|comment|文字列|コメント|

### コメント（comment）の実装

<details close>
<summary>models/comment.js</summary>

```js
'use strict';
const loader = require('./sequelize-loader');
const Sequelize = loader.Sequelize;

const Comment = loader.database.define('comments', {
  scheduleId: {
    type: Sequelize.UUID,
    primaryKey: true,
    allowNull: false
  },
  userId: {
    type: Sequelize.INTEGER,
    primaryKey: true,
    allowNull: false
  },
  comment: {
    type: Sequelize.STRING,
    allowNull: false
  }
}, {
    freezeTableName: true,
    timestamps: false
  });

module.exports = Comment;
```
</details>

コメントにおいても、scheduleId で大量のデータの中から検索するため、scheduleId のインデックスを作成する必要があるはず。

しかし、ここではインデックスを作成する必要がない。
なぜなら scheduleId と userId で複合主キーを作成しており、その主キーの作成順番が、scheduleId > userId という順番となっているためである。

RDB では主キーには自動的にインデックスが構築される。  
複合主キーで作成された主キーのインデックスは、途中までデータを検索する順番が 一緒であればそれをインデックスとして使うことができます。

そのため上の例では、 scheduleId のインデックスは別途作成しなくても 主キーのインデックスを代わりに用いることができるのである。  
（ちなみに、candidate や availability では、scheduleId は複合主キーを京成するものではなかった）

以上でモデルの実装は終了となる。

実際にこのアプリケーションに読み込んでテーブルが作成できるかどうかを試してみよう。

### リレーションの設定
なお、この sequelize では、モデルを使ってエンティティ同士の関係を定義しておくことで、後で自動的に RDB 上でテーブルの結合をしてデータを取得することができる。

その機能を使えるようにデータを読み込んでみよう。
`app.js` を以下の変更のように実装してみる。

<details close>
<summary>app.js</summary>

```diff
 var session = require('express-session');
 var passport = require('passport');
 
+// モデルの読み込み
+var User = require('./models/user');
+var Schedule = require('./models/schedule');
+var Availability = require('./models/availability');
+var Candidate = require('./models/candidate');
+var Comment = require('./models/comment');
+User.sync().then(() => {
+  Schedule.belongsTo(User, {foreignKey: 'createdBy'});
+  Schedule.sync();
+  Comment.belongsTo(User, {foreignKey: 'userId'});
+  Comment.sync();
+  Availability.belongsTo(User, {foreignKey: 'userId'});
+  Candidate.sync().then(() => {
+    Availability.belongsTo(Candidate, {foreignKey: 'candidateId'});
+    Availability.sync();
+  });
+});
+
 var GitHubStrategy = require('passport-github2').Strategy;
 var GITHUB_CLIENT_ID = '2f831cb3d4aac02393aa';
 var GITHUB_CLIENT_SECRET = '9fbc340ac0175123695d2dedfbdf5a78df3b8067';
```
</details>

**解説**

以上の実装を解説する。

```js
User.sync().then(() => {
```

ここではまず sync 関数という、モデルに合わせてデータベースのテーブルを作成する関数を呼び出している。 そして User に対応するテーブルの作成が終わった後に実行したい処理を、無名関数で記述している。

```js
  Schedule.belongsTo(User, {foreignKey: 'createdBy'});
  Schedule.sync();
```

これは、 予定がユーザーの従属エンティティであることを定義している。  
また、 Schedule における createdBy が User の外部キーとなることを設定し、 Schedule に対応するテーブルを作成している。

```js
  Comment.belongsTo(User, {foreignKey: 'userId'});
  Comment.sync();
```

以上も同様。コメントがユーザーに従属していることを定義しており、sync 関数を呼び出している。

```js
  Availability.belongsTo(User, {foreignKey: 'userId'});
```

これは、出欠がユーザーに従属していることを定義している。

```js
  Candidate.sync().then(() => {
    Availability.belongsTo(Candidate, {foreignKey: 'candidateId'});
    Availability.sync();
  });
```

ここでは、候補日程に対応するテーブルを sync 関数で作成し、その後、 出欠が候補日程に従属していることを定義して、Availability の sync 関数を呼び出している。

これで、リレーションに関連した結合が自動的に行われるように設定されて、テーブルが作成される。

なお、リレーションをみてみると、

- 予定と候補日程
- 予定とコメント

これらのリレーションは定義していない。  
これは、候補日程もコメントも予定 ID のインデックスを利用してそれぞれ取得し、 特にテーブルの結合を用いては取得しないため、ここでは設定しなかったものである。

では、サーバーを起動してテーブルを作成してみよう。

```bash
PORT=8000 yarn start
```

実行すると、

```
Executing (default): CREATE TABLE IF NOT EXISTS "users" ("userId" INTEGER NOT NULL , "username" VARCHAR(255) NOT NULL, PRIMARY KEY ("userId"));
Executing (default): SELECT i.relname AS name, ix.indisprimary AS primary, ix.indisunique AS unique, ix.indkey AS indkey, array_agg(a.attnum) as column_indexes, array_agg(a.attname) AS column_names, pg_get_indexdef(ix.indexrelid) AS definition FROM pg_class t, pg_class i, pg_index ix, pg_attribute a WHERE t.oid = ix.indrelid AND i.oid = ix.indexrelid AND a.attrelid = t.oid AND t.relkind = 'r' and t.relname = 'users' GROUP BY i.relname, ix.indexrelid, ix.indisprimary, ix.indisunique, ix.indkey ORDER BY i.relname;
Executing (default): CREATE TABLE IF NOT EXISTS "schedules" ("scheduleId" UUID NOT NULL , "scheduleName" VARCHAR(255) NOT NULL, "memo" TEXT NOT NULL, "createdBy" INTEGER NOT NULL REFERENCES "users" ("userId") ON DELETE NO ACTION ON UPDATE CASCADE, "updatedAt" TIMESTAMP WITH TIME ZONE NOT NULL, PRIMARY KEY ("scheduleId"));
Executing (default): CREATE TABLE IF NOT EXISTS "comments" ("scheduleId" UUID NOT NULL , "userId" INTEGER NOT NULL  REFERENCES "users" ("userId") ON DELETE NO ACTION ON UPDATE CASCADE, "comment" VARCHAR(255) NOT NULL, PRIMARY KEY ("scheduleId","userId"));
Executing (default): CREATE TABLE IF NOT EXISTS "candidates" ("candidateId"   SERIAL , "candidateName" VARCHAR(255) NOT NULL, "scheduleId" UUID NOT NULL, PRIMARY KEY ("candidateId"));
Executing (default): SELECT i.relname AS name, ix.indisprimary AS primary, ix.indisunique AS unique, ix.indkey AS indkey, array_agg(a.attnum) as column_indexes, array_agg(a.attname) AS column_names, pg_get_indexdef(ix.indexrelid) AS definition FROM pg_class t, pg_class i, pg_index ix, pg_attribute a WHERE t.oid = ix.indrelid AND i.oid = ix.indexrelid AND a.attrelid = t.oid AND t.relkind = 'r' and t.relname = 'schedules' GROUP BY i.relname, ix.indexrelid, ix.indisprimary, ix.indisunique, ix.indkey ORDER BY i.relname;
Executing (default): SELECT i.relname AS name, ix.indisprimary AS primary, ix.indisunique AS unique, ix.indkey AS indkey, array_agg(a.attnum) as column_indexes, array_agg(a.attname) AS column_names, pg_get_indexdef(ix.indexrelid) AS definition FROM pg_class t, pg_class i, pg_index ix, pg_attribute a WHERE t.oid = ix.indrelid AND i.oid = ix.indexrelid AND a.attrelid = t.oid AND t.relkind = 'r' and t.relname = 'comments' GROUP BY i.relname, ix.indexrelid, ix.indisprimary, ix.indisunique, ix.indkey ORDER BY i.relname;
Executing (default): SELECT i.relname AS name, ix.indisprimary AS primary, ix.indisunique AS unique, ix.indkey AS indkey, array_agg(a.attnum) as column_indexes, array_agg(a.attname) AS column_names, pg_get_indexdef(ix.indexrelid) AS definition FROM pg_class t, pg_class i, pg_index ix, pg_attribute a WHERE t.oid = ix.indrelid AND i.oid = ix.indexrelid AND a.attrelid = t.oid AND t.relkind = 'r' and t.relname = 'candidates' GROUP BY i.relname, ix.indexrelid, ix.indisprimary, ix.indisunique, ix.indkey ORDER BY i.relname;
Executing (default): CREATE INDEX "schedules_created_by" ON "schedules" ("createdBy")
Executing (default): CREATE INDEX "candidates_schedule_id" ON "candidates" ("scheduleId")
Executing (default): CREATE TABLE IF NOT EXISTS "availabilities" ("candidateId" INTEGER NOT NULL  REFERENCES "candidates" ("candidateId") ON DELETE NO ACTION ON UPDATE CASCADE, "userId" INTEGER NOT NULL  REFERENCES "users" ("userId") ON DELETE NO ACTION ON UPDATE CASCADE, "availability" INTEGER NOT NULL DEFAULT 0, "scheduleId" UUID NOT NULL, PRIMARY KEY ("candidateId","userId"));
Executing (default): SELECT i.relname AS name, ix.indisprimary AS primary, ix.indisunique AS unique, ix.indkey AS indkey, array_agg(a.attnum) as column_indexes, array_agg(a.attname) AS column_names, pg_get_indexdef(ix.indexrelid) AS definition FROM pg_class t, pg_class i, pg_index ix, pg_attribute a WHERE t.oid = ix.indrelid AND i.oid = ix.indexrelid AND a.attrelid = t.oid AND t.relkind = 'r' and t.relname = 'availabilities' GROUP BY i.relname, ix.indexrelid, ix.indisprimary, ix.indisunique, ix.indkey ORDER BY i.relname;
Executing (default): CREATE INDEX "availabilities_schedule_id" ON "availabilities" ("scheduleId")
```

以上のように、大量の SQL が実行される。

> **NOTE:**   
> 途中でエラーが沖田場合には、以下のコマンドで DB を作り直してエラーの対応を実施する。
> `drop database schedule_arranger; create database schedule_arranger;`

### ユーザーの保存
無事ここまでモデルを実装できたら、実際に DB にユーザー情報を保存させてみよう。  
`app.js` を以下の変更差分のように実装する。

<details close>
<summary>app.js</summary>

```diff
     process.nextTick(function () {
-      return done(null, profile);
+      User.upsert({
+        userId: profile.id,
+        username: profile.username
+      }).then(() => {
+        done(null, profile);
+      });
     });
   }
   ));
```
</details>

これは、 GitHub 認証が実行された際に呼び出される処理。

```js
      User.upsert({
        userId: profile.id,
        username: profile.username
```

ここでは、 User モデルに対して、取得されたユーザー ID とユーザー名を User のテーブルに保存している。

upsert 関数は、 INSERT または UPDATE を行うという意味の造語 = UPSERT を行う関数。  
主キーで識別されるデータがない場合にはデータを挿入し、ある場合には渡されたデータを元に更新を行ってくれる。

では、サーバーを起動し、`http://localhost:8000/login` にアクセスしてログインしてみよう。

```
Executing (default): CREATE OR REPLACE FUNCTION pg_temp.sequelize_upsert() RETURNS integer AS $func$ BEGIN INSERT INTO "users" ("userId","username") VALUES ('15885373','Soichiro-Yoshimura'); RETURN 1; EXCEPTION WHEN unique_violation THEN UPDATE "users" SET "userId"='15885373',"username"='Soichiro-Yoshimura' WHERE ("userId" = '15885373'); RETURN 2; END; $func$ LANGUAGE plpgsql; SELECT * FROM pg_temp.sequelize_upsert();
```

以上のような SQL が実行されて、エラーが発生していなければ成功。

なお、この `CREATE OR REPLACE FUNCTION` という SQL 文は、 データベースの内部に関数を作成する PostgreSQL の機能。  
UPSERT の機能を実現するためにここでは利用されている。

内部的には INSERT 文を実行し、もし失敗する場合は引き続いて UPDATE 文を実行するという関数が PostgreSQL 内で定義されたあと、実行されている。

以上でこのモデルの実装とユーザーの保存の実装は完了。

## § 18. 予定の一覧の表示
ここまででデータモデルの作成とユーザーの保存ができるようになった。

ここからはこの「予定調整くん」の中心的な機能となる、予定の作成と表示を実装していく。

予定の表示を行う前にまず、予定の作成を行えるようにする必要がある。まずはフォームを使って予定を作れるようにするところから始める。

1. トップ画面にログインへのリンクを作成
1. ログイン時にしか表示されない予定作成フォームを作成
1. 予定作成フォームから送られた情報を保存
1. トップ画面に自分が作った予定一覧を作成
1. 予定と出欠表の表示画面を作成

本セクションでは、この流れで実装を行っていく。

### トップ画面にログインへのリンクを作成

<details close>
<summary>views/index.pug</summary>

```diff
 block content
   h1= title
   p Welcome to #{title}
+  div
+    if user
+      a(href="/logout") #{user.username} をログアウト
+    else
+      a(href="/login") ログイン
```
</details>

このテンプレートには user は割り当てられていないので、`routes/index.js` を以下のように変更して、このテンプレートに user オブジェクト渡す。

<details close>
<summary>routes/index.js</summary>

```diff
 /* GET home page. */
 router.get('/', function (req, res, next) {
-  res.render('index', { title: 'Express' });
+  res.render('index', { title: 'Express', user: req.user });
 });
```
</details>

### ログイン時にしか表示されない予定作成フォームを作成
予定作成フォームに必要な要素は

- 予定名
- メモ
- 作成者

であった。ここで作成者が必要になるため、この予定作成フォームは、必ずログイン時にしか表示されないようにしなくてはならない。

また、

- 候補日程

も同様にこの予定作成のときに作ってしまおう。  
候補日程はテキストエリアのフォームに改行を入れて複数入力する形式にしよう。

`views/new.pug` を新たに作成して、以下のように実装する。

<details close>
<summary>views/new.pug</summary>

```pug
extends layout

block content
  form(method="post", action="/schedules")
    div
      h5 予定名
      input(type="text" name="scheduleName")
    div
      h5 メモ
      textarea(name="memo")
    div
      h5 候補日程 (改行して複数入力してください)
      textarea(name="candidates")
    button(type="submit") 予定をつくる
```

`/schedules` というパスに POST メソッドを送信する普通のフォーム。

</details>

今度はこの予定作成フォームを表示させ、POST を受け取る Router オブジェクトを実装しよう。

`routes/schedules.js` を以下のように実装する。

<details close>
<summary>routes/schedules.js</summary>

```js
'use strict';
const express = require('express');
const router = express.Router();
const authenticationEnsurer = require('./authentication-ensurer');

router.get('/new', authenticationEnsurer, (req, res, next) => {
  res.render('new', { user: req.user });
});

router.post('/', authenticationEnsurer, (req, res, next) => {
  console.log(req.body); // TODO 予定と候補を保存する実装をする
  res.redirect('/');
});

module.exports = router;
```
</details>

このように実装する。まずは POST で受けとった情報を `console.log` 関数を使って標準出力に表示するだけにとどめている。なお、ここでは `const authenticationEnsurer = require('./authentication-ensurer');` という認証を確かめるハンドラ関数 `authentication-ensurer.js` がある前提で実装した。この、認証をしているかチェックする関数はさまざまな Router オブジェクトから利用したいためである。

ではその `routes/authentication-ensurer.js` を作成して以下のように実装しよう。

<details close>
<summary>routes/authentication-ensurer.js</summary>

```js
'use strict';

function ensure(req, res, next) {
  if (req.isAuthenticated()) { return next(); }
  res.redirect('/login');
}

module.exports = ensure;
```
</details>

認証をチェックして、認証されていない場合は `/login` にリダイレクトを行う関数がモジュールとして用意できた。  
なお、`req.isAuthenticated()` は passport により提供されるものであり、`res.redirect()` は Express の機能。

そして先ほど作成した `routes/schedules.js` をルーターとして Application オブジェクトに登録しておく。`app.js` を以下のように実装する。

<details close>
<summary>app.js</summary>

```diff
 var indexRouter = require('./routes/index');
 var loginRouter = require('./routes/login');
 var logoutRouter = require('./routes/logout');
+var schedulesRouter = require('./routes/schedules');
 
 var app = express();
 app.use(helmet());
```

以上でモジュールを読み込み、

```diff
 app.use('/', indexRouter);
 app.use('/login', loginRouter);
 app.use('/logout', logoutRouter);
+app.use('/schedules', schedulesRouter);
 
 app.get('/auth/github',
   passport.authenticate('github', { scope: ['user:email'] }),
```

この実装で `/schedules` のパスに登録する。これで完成。
</details>

ここまで行ったら、サーバーを再起動して `http://localhost:8000/schedules/new` にアクセスし、ログインが求められるかをチェック。また、ログインをした後に同じ URL に再度アクセスして予定作成フォームが表示されれば成功。

無事に予定作成フォームが表示されたら、フォームに適当な文字を入力して、「予定をつくる」ボタンを押してみよう。

```
{ scheduleName: 'ラーメンを食べに行く',
  memo: 'とんこつラーメンか\r\n醤油とんこつラーメンで',
  candidates: '12/5の昼食\r\n12/6の昼食' }
```

以上のように表示されればフォームとして機能していることがわかる。  
今度は、この情報をデータベースに保存する処理を書いてみよう。

### 予定作成フォームから送られた情報を保存
ここから受け取った情報を保存していくが、その前に一つしなくてはならないことがある。

それは、前回紹介した UUID を生成するためのライブラリのインストールである。

UUID は被らないランダム ID で、ライブラリを使っていくらでも生成することができる。

```bash
yarn add uuid@3.3.2
```

以上のコマンドで uuid をインストールする。使い方は簡単で、`uuid.v4()` を呼び出すと、UUID の文字列が取得できる。

では、次に `routes/schedules.js` を実装していく。

<details close>
<summary>routes/schedules.js</summary>

```diff
 const express = require('express');
 const router = express.Router();
 const authenticationEnsurer = require('./authentication-ensurer');
+const uuid = require('uuid');
+const Schedule = require('../models/schedule');
+const Candidate = require('../models/candidate');
 
 router.get('/new', authenticationEnsurer, (req, res, next) => {
   res.render('new', { user: req.user });
 });
 
 router.post('/', authenticationEnsurer, (req, res, next) => {
-  console.log(req.body); // TODO 予定と候補を保存する実装をする
-  res.redirect('/');
+  const scheduleId = uuid.v4();
+  const updatedAt = new Date();
+  Schedule.create({
+    scheduleId: scheduleId,
+    scheduleName: req.body.scheduleName.slice(0, 255) || '（名称未設定）',
+    memo: req.body.memo,
+    createdBy: req.user.id,
+    updatedAt
+  }).then((schedule) => {
+    const candidateNames = req.body.candidates.trim().split('\n').map((s) => s.trim()).filter((s) => s !== "");
+    const candidates = candidateNames.map((c) => {
+      return {
+        candidateName: c,
+        scheduleId: schedule.scheduleId
+      };
+    });
+    Candidate.bulkCreate(candidates).then(() => {
+      res.redirect('/schedules/' + schedule.scheduleId);
+    });
+  });
 });
```
</details>

**解説**

```js
  Schedule.create({
    scheduleId: scheduleId,
    scheduleName: req.body.scheduleName.slice(0, 255) || '（名称未設定）',
    memo: req.body.memo,
    createdBy: req.user.id,
    updatedAt
  }).then((schedule) => {
```
これは、予定をデータベース内に保存しているコード。

予定名だけは、データベース上長さの制限があるため、`.slice(0, 255)` という関数呼び出しによって、 255 文字以内の文字の長さになるようにしてある。  
さらに、空の文字列を入力した場合に限って、予定名を `（名称未設定）` として保存させるようにする。

```js
  }).then((schedule) => {
    const candidateNames = req.body.candidates.trim().split('\n').map((s) => s.trim()).filter((s) => s !== "");
    const candidates = candidateNames.map((c) => {
      return {
        candidateName: c,
        scheduleId: schedule.scheduleId
      };
    });
    Candidate.bulkCreate(candidates).then(() => {
      res.redirect('/schedules/' + schedule.scheduleId);
    });
```

以上は、無事予定を保存し終わったら実行される関数の中で、 まずはリクエストのボディの中から、候補日程の配列を取得している。  
それを使って今度は保存すべき候補の candidate オブジェクトを作成している。

最後に sequelize の複数のオブジェクトを保存する関数、 `bulkCreate` 関数を利用して保存し、その処理が終わったあとに、 `/schedules/:scheduleId` にリダイレクトされる処理を記述している。

なお、sequelize のデータベースの処理は基本的に全て、非同期 IO で実行される。そのため、*前のデータベースの処理が前提となって次の処理を行いたい場合* は、実行して得られる結果オブジェクトに対して、`then` 関数を呼び、そこに次に行いたい処理を記述する。

今まで説明を行ってきませんでしたがこの結果として得られるオブジェクトは、 ES6 の機能である、 **Promise** というオブジェクトとなる。

この Promise とはなんなのか？

### Promise
Promise とは、非同期に実行される未来の結果を表すオブジェクト。Promise から値を取得する際には `then` 関数を呼び、そこに実行完了時に呼び出されてほしい関数を与えることによって未来の結果を利用する。

Promise オブジェクトは自分で作ることもできる。
また、複数の Promise オブジェクトをまとめた Promise を作ることもできるし、例外を含む結果を扱うこともできる機能も持ち合わせている。  
そのため、Promise は非同期処理の結果を簡単に取り扱うことができる非常に便利な仕組みとなっている。

----
ではさっそく実装したものを動かして予定を保存してみよう。  
以上で起動して `http://localhost:8000/schedules/new` から適当な内容で登録してみる。

表示内容は Not Found 404 となるが、これはまだ予定表示ページを作っていないため仕方ない。

今のところは、`http://localhost:8000/schedules/:scheduleId` のような URL にリダイレクトされれば問題ない（`:scheduleId` の部分は（UUIDゆえ）ランダムな値になる）。

サーバー側のコンソールに SQL の実行結果が表示されていて、エラーが起こっていなければ問題ない。

続けて、この内容を表示するための画像を作っていく。

### トップ画面に自分が作った予定一覧を作成
まずは、トップ画面に自分が作成した予定の一覧を表示させてみる。
`views/index.pug` を以下のように変更する。

<details close>
<summary>views/index.pug</summary>

```diff
   p Welcome to #{title}
   div
     if user
-      a(href="/logout") #{user.username} をログアウト
+      div
+        a(href="/logout") #{user.username} をログアウト
+      div
+        a(href="/schedules/new") 予定を作る
+      - var hasSchedule = schedules.length > 0
+      if hasSchedule
+        h3 あなたの作った予定一覧
+        table
+          tr
+            th 予定名
+            th 更新日時
+          each schedule in schedules
+            tr
+              td
+                a(href=`/schedules/${schedule.scheduleId}`) #{schedule.scheduleName}
+              td #{schedule.updatedAt}
     else
-      a(href="/login") ログイン
+      div
+        a(href="/login") ログイン
```
</details>

このテンプレートでは `schedules` という自分自身が作成した予定の配列が渡されていることを前提として実装している。

リンクと予定一覧を分けるために `div` 要素でログイン、ログアウトのリンクを囲ってある。

また `hasSchedule` という、予定を持っているかどうかのフラグを作成し、`schedules` という配列をループさせてテーブルの列を作成している。

ではテンプレートに合うように、`routes/index.js` において、自分の作った予定の配列を `schedules` プロパティに割り当てるようにしよう。

<details close>
<summary>routes/index.js</summary>

```js
'use strict';
const express = require('express');
const router = express.Router();
const Schedule = require('../models/schedule');

/* GET home page. */
router.get('/', (req, res, next) => {
  const title = '予定調整くん';
  if (req.user) {
    Schedule.findAll({
      where: {
        createdBy: req.user.id
      },
      order: [['"updatedAt"', 'DESC']]
    }).then((schedules) => {
      res.render('index', {
        title: title,
        user: req.user,
        schedules
      });
    });
  } else {
    res.render('index', { title: title　});
  }
});

module.exports = router;
```

この際なので、変数 `title` も `予定調整くん` に変更し、ES6 の形ですべて書き換えてしまおう。
</details>

```js
    Schedule.findAll({
      where: {
        createdBy: req.user.id
      },
      order: [['"updatedAt"', 'DESC']]
    }).then((schedules) => {
```

以上の実装で、自分で作成した予定を絞り込み、作成日時順にソートして取得している。  
また、認証済みかどうか（`req.user` オブジェクトがあるかどうか）で処理全体を振り分けている。

[`findAll` 関数](http://docs.sequelizejs.com/class/lib/model.js~Model.html#findall) は、条件にあったデータモデルに対応するレコードをすべて取得する関数。

ここでは、データベースの取得に作成者のユーザー ID を利用する。  
なお、このデータモデルでは、`createdBy` に保存されているユーザー ID にはインデックスを作成しているが、`updatedAt` にはインデックスを作成しなかった。

これは、一人が左k酢英する予定の数はそんなに多くならないため、ソートのコストはそんなに高くならないと考えられるためである。  
もし必要となったら、その際にデータモデルを変更してしまえばよいだろう。

では、サーバーを再起動して `http://localhost:8000` でログインしよう。  
そして、自分が作った予定一覧に、先ほど適当に登録した予定が表示されるか確認してみよう。

一覧として更新日時と一緒に表示されれば成功となる。  
なお、予定の内容を表示させられるリンクについてはまだ 404 Not Found となるはず。

### 予定と出欠表の表示画面を作成
ここからは今回作る最後の画面、予定の表示画面の実装になる。

- 予定の内容
- 出欠表

以上の 2 つを表示するように実装する。  
出欠表の構成は、行を候補日、列をユーザー名、のように作成し、出欠自体を表示する部分は、ボタンを仮置きして後ほど作っていく。

<details close>
<summary>views/schedule.pug</summary>

```pug
extends layout

block content
  h4 #{schedule.scheduleName}
  p(style="white-space:pre;") #{schedule.memo}
  p 作成者: #{schedule.user.username}
  h3 出欠表
  table
    tr
      th 予定
      each user in users
        th #{user.username}
    each candidate in candidates
      tr
        th #{candidate.candidateName}
        each user in users
          td
            button 欠席
```

</details>

ここでは

- `schedule` を URL で指定された予定
- `schedule.user` を予定の作成者のユーザー
- `users` を表示者と出欠情報を持つ前ユーザー
- `candidates` を全候補

としてこのテンプレートへ変数が渡されたものとして実装している。

なお、出欠の更新は最終的に Ajax で行いたいため、HTML のボタン要素を置くだけにとどめている。

次に、`routes/schedules.js` を以下のように実装しよう。

<details close>
<summary>routes/schedules.js</summary>

```diff
 const uuid = require('uuid');
 const Schedule = require('../models/schedule');
 const Candidate = require('../models/candidate');
+const User = require('../models/user');
 
 router.get('/new', authenticationEnsurer, (req, res, next) => {
   res.render('new', { user: req.user });
```

以上の部分で、必要となるユーザーのデータモデルを読み込んでいる。そして以下が Router オブジェクトの実際の処理である。

```diff
   });
 });
 
+router.get('/:scheduleId', authenticationEnsurer, (req, res, next) => {
+  Schedule.findOne({
+    include: [
+      {
+        model: User,
+        attributes: ['userId', 'username']
+      }],
+    where: {
+      scheduleId: req.params.scheduleId
+    },
+    order: [['"updatedAt"', 'DESC']]
+  }).then((schedule) => {
+    if (schedule) {
+      Candidate.findAll({
+        where: { scheduleId: schedule.scheduleId },
+        order: [['"candidateId"', 'ASC']]
+      }).then((candidates) => {
+         res.render('schedule', {
+              user: req.user,
+              schedule: schedule,
+              candidates: candidates,
+              users: [req.user]
+            });
+      });
+    } else {
+      const err = new Error('指定された予定は見つかりません');
+      err.status = 404;
+      next(err);
+    }
+  });
+});
 module.exports = router;
```
</details>

スケジュールを取得しtえ、その候補を取得する実装となる。

**解説**

```js
  Schedule.findOne({
    include: [
      {
        model: User,
        attributes: ['userId', 'username']
      }],
    where: {
      scheduleId: req.params.scheduleId
    },
    order: [['"updatedAt"', 'DESC']]
  }).then((schedule) => {
```

以上の実装は、sequelize を利用して０部流を結合してユーザーを取得する書き方。  
`schedule.user` というプロパティに、ユーザー情報が設定される。

なお、findOne 関数は、そのデータモデルに対応するデータを 1 行だけ取得する関数。

またユーザーの属性としては、ユーザー ID とユーザー名が取得されている。  
取得自体は、予定の更新日時の降順で取得されるようになっている。

```js
    if (schedule) {
      Candidate.findAll({
        where: { scheduleId: schedule.scheduleId },
        order: [['"candidateId"', 'ASC']]
      }).then((candidates) => {
         res.render('schedule', {
              user: req.user,
              schedule: schedule,
              candidates: candidates,
              users: [req.user]
            });
      });
    } else {
      const err = new Error('指定された予定は見つかりません');
      err.status = 404;
      next(err);
    }
```

以上で、予定が見つかった場合に、その候補一覧を取得している。  
並びは、候補 ID の昇順、つまり作られた順番に並ぶ。

無事取得できたら、先ほどのテンプレートに必要な変数を設定して、テンプレートを描画している。

なお、予定が見つからなかった場合には 404 Not Found を表示するようにしている。

ここまで実装できたら、サーバーを再起動して `http://localhost:8000/` でログインし、自分が作った予定一覧にある（自分が作った）予定のリンクをクリックして、その予定のメモ、候補、出欠表が表示されるかチェックしてみよう。

出欠表の列は自分の名前、行はそれぞれの候補になっていれば実装は成功。

### 予定が作成でき、表示されることのテスト
予定が作成でき、表示されることのテストを書く。

<details close>
<summary>test/test.js</summary>

まずは、各モデルを読み込む。

```js
const User = require('../models/user');
const Candidate = require('../models/candidate');
const Schedule = require('../models/schedule');
```

次に、既存のテストコードの下に以下を追記する。

```js
describe('/schedules', () => {
  before(() => {
    passportStub.install(app);
    passportStub.login({ id: 0, username: 'testuser' });
  });

  after(() => {
    passportStub.logout();
    passportStub.uninstall(app);
  });

  it('予定が作成でき、表示される', (done) => {
    User.upsert({ userId: 0, username: 'testuser' }).then(() => {
      request(app)
        .post('/schedules')
        .send({ scheduleName: 'テスト予定1', memo: 'テストメモ1\r\nテストメモ2', candidates: 'テスト候補1\r\nテスト候補2\r\nテスト候補3' })
        .expect('Location', /schedules/)
        .expect(302)
        .end((err, res) => {
          const createdSchedulePath = res.headers.location;
          request(app)
            .get(createdSchedulePath)
            // 以下で、作成された予定と候補が表示されていることをテストする
            .expect(/テスト予定1/)
            .expect(/テストメモ1/)
            .expect(/テストメモ2/)
            .expect(/テスト候補1/)
            .expect(/テスト候補2/)
            .expect(/テスト候補3/)
            .expect(200)
            .end((err, res) => {
              if (err) return done(err);
              // テストで作成したデータを削除
              const scheduleId = createdSchedulePath.split('/schedules/')[1];
              Candidate.findAll({
                where: { scheduleId: scheduleId }
              }).then((candidates) => {
                const promises = candidates.map((c) => { return c.destroy(); });
                Promise.all(promises).then(() => {
                  Schedule.findByPk(scheduleId).then((s) => { 
                    s.destroy().then(() => { 
                      if (err) return done(err);
                      done(); 
                    });
                  });
                });
              });
            });
        });
    });
  });

});
```
</details>

**解説**

まず、`userId` が `0` で、 `username` が `testuser` のユーザーを DB 上に作成している。

その後、POST メソッドを使い予定と候補を作成している。

そこからリダイレクトされることを検証し、予定が表示されるページヘのアクセスが 200 のステータスコードであることを検証している。

なお、テストが終わった後に、テストで作成されたユーザー以外のデータを削除する処理が加えられている。
ここで使われている `findByPk` 関数は、 モデルに対応するデータを主キーによって 1 行だけ取得することができる関数。  
また、`Promise.all` 関数は、 配列で渡された全ての Promise が終了した際に結果を返す、Promise オブジェクトを作成する。  
なお、空配列が渡された場合も Promise オブジェクトが作成される。

## § 19. 出欠の表示と更新
ここまでで、予定と出欠表を表示するページが作成された。  
今度は出欠表を使って自分の出欠を更新できるようにしていこう。

### 出欠表と表示アルゴリズム
前回までの実装と、HTML へ表示された出欠表は

- 行を、候補
- 列を、ユーザー名

で構成していた。このとき、出欠のデータをどのように用意すれば、表の内容を表示できるのだろうか。

そもそも、出欠の情報はまだ DB に追加していない。そのため、出欠表の閲覧ユーザーはまずは自分はすべて「欠席」であるという状態で、この出欠表が表示される必要がある。

また、他のユーザーが更新した出欠のデータでも、そのユーザーは特定の候補にだけ出欠を入力する場合が考えられる。  
入力が行われた一部候補の出欠に関してだけ、情報が保管されている状態である。

以上を考えると、つまり DB 上に情報がなくとも、表示上、出欠データがあるように見せる工夫が必要なのである。

また、テンプレート内で、候補で列をループし、ユーザー名で行をループしている。そこに出欠情報を表示させるにはどのようにすればいいだろうか。

こういう 2 つのキーで参照する情報には、入れ子になった連想配列 Map を利用する。

- 出欠 MapMap（Map の Map）
    - キーは、ユーザー ID
    - 値は、出欠 Map
- 出欠 Map
    - キーは、候補 ID
    - 値は、出欠内容

MapMap が用意されていれば ユーザー ID と候補 ID を使って、出欠内容を取得することがでい、テンプレートの候補とユーザーのループの中で値を取得することができる。

では、この出欠データを作る手順をまとめていく。

1. データベースからその予定のすべての出欠を取得する
1. 出欠 MapMap（キー: ユーザー ID、値：出欠 Map（キー：候補 ID、値：出欠））を作成する
1. 閲覧ユーザーと出欠に紐づくユーザーからユーザー Map（キー：ユーザー ID、値：ユーザー）を作る
1. 全ユーザー・全候補で二重ループして、それぞれの朱毛kつの値がない場合には「欠席」を設定する

### 出欠情報の作成
では実際に `routes/schedules.js` を以下のように実装する。

<details close>
<summary>routes/schedules.js</summary>

```diff
 const Schedule = require('../models/schedule');
 const Candidate = require('../models/candidate');
 const User = require('../models/user');
+const Availability = require('../models/availability');
 
 router.get('/new', authenticationEnsurer, (req, res, next) => {
   res.render('new', { user: req.user });
```

```diff
         where: { scheduleId: schedule.scheduleId },
         order: [['"candidateId"', 'ASC']]
       }).then((candidates) => {
-        res.render('schedule', {
-          user: req.user,
-          schedule: schedule,
-          candidates: candidates,
-          users: [req.user]
+        // データベースからその予定の全ての出欠を取得する
+        Availability.findAll({
+          include: [
+            {
+              model: User,
+              attributes: ['userId', 'username']
+            }
+          ],
+          where: { scheduleId: schedule.scheduleId },
+          order: [[User, 'username', 'ASC'], ['"candidateId"', 'ASC']]
+        }).then((availabilities) => {
+          // 出欠 MapMap(キー:ユーザー ID, 値:出欠Map(キー:候補 ID, 値:出欠)) を作成する
+          const availabilityMapMap = new Map(); // key: userId, value: Map(key: candidateId, availability)
+          availabilities.forEach((a) => {
+            const map = availabilityMapMap.get(a.user.userId) || new Map();
+            map.set(a.candidateId, a.availability);
+            availabilityMapMap.set(a.user.userId, map);
+          });
+
+          console.log(availabilityMapMap); // TODO 除去する
+
+          res.render('schedule', {
+            user: req.user,
+            schedule: schedule,
+            candidates: candidates,
+            users: [req.user],
+            availabilityMapMap: availabilityMapMap
+          });
         });
       });
     } else {
```
</details>

**解説**

```js
// データベースからその予定の全ての出欠を取得する
Availability.findAll({
  include: [
    {
      model: User,
      attributes: ['userId', 'username']
    }
  ],
  where: { scheduleId: schedule.scheduleId },
  order: [[User, 'username', 'ASC'], ['"candidateId"', 'ASC']]
}).then((availabilities) => {
```

この部分は、予定 ID でしぼりこんだ出欠の取得である（しぼりこみは `where` で行う）。  
後にユーザー情報を利用するため、ユーザー名もテーブルの結合をして取得してある（`include` の部分）。また、取得順番は、ユーザー名の昇順、候補 ID の昇順で並べている。

```js
// 出欠 MapMap(キー:ユーザー ID, 値:出欠Map(キー:候補 ID, 値:出欠)) を作成する
const availabilityMapMap = new Map(); // key: userId, value: Map(key: candidateId, availability)
availabilities.forEach((a) => {
  const map = availabilityMapMap.get(a.user.userId) || new Map();
  map.set(a.candidateId, a.availability);
  availabilityMapMap.set(a.user.userId, map);
});
```

ここで出欠 MapMap を作っている。ここでは、出欠のデータがあったものだけをデータとして入れ子の連想配列の中に保存している。

ここまで実装できたら、サーバーを再起動して `http:/localhost:8000` にアクセス・ログインの後、何かしらの予定を作成し、その予定の内容を表示するページにアクセスしてみよう。

すると、`Map {}` と表示されるはず。これはまだ出欠のレコードが一つもできていないためである。

ここまでうまくいったら次を実装していく。

<details close>
<summary>routes/schedule.js</summary>

```diff
         where: { scheduleId: schedule.scheduleId },
         order: [['"candidateId"', 'ASC']]
       }).then((candidates) => {
         // データベースからその予定の全ての出欠を取得する
         Availability.findAll({
           include: [
             {
               model: User,
               attributes: ['userId', 'username']
             }
           ],
           where: { scheduleId: schedule.scheduleId },
           order: [[User, 'username', 'ASC'], ['"candidateId"', 'ASC']]
         }).then((availabilities) => {
           // 出欠 MapMap(キー:ユーザー ID, 値:出欠Map(キー:候補 ID, 値:出欠)) を作成する
           const availabilityMapMap = new Map(); // key: userId, value: Map(key: candidateId, availability)
           availabilities.forEach((a) => {
             const map = availabilityMapMap.get(a.user.userId) || new Map();
             map.set(a.candidateId, a.availability);
             availabilityMapMap.set(a.user.userId, map);
           });

+          // 閲覧ユーザーと出欠に紐づくユーザーからユーザー Map (キー:ユーザー ID, 値:ユーザー) を作る
+          const userMap = new Map(); // key: userId, value: User
+          userMap.set(parseInt(req.user.id), {
+              isSelf: true,
+              userId: parseInt(req.user.id),
+              username: req.user.username
+          });
+          availabilities.forEach((a) => {
+            userMap.set(a.user.userId, {
+              isSelf: parseInt(req.user.id) === a.user.userId, // 閲覧ユーザー自身であるかを含める
+              userId: a.user.userId,
+              username: a.user.username
+            });
+          });
+
+          // 全ユーザー、全候補で二重ループしてそれぞれの出欠の値がない場合には、「欠席」を設定する
+          const users = Array.from(userMap).map((keyValue) => keyValue[1]);
+          users.forEach((u) => {
+            candidates.forEach((c) => {
+              const map = availabilityMapMap.get(u.userId) || new Map();
+              const a = map.get(c.candidateId) || 0; // デフォルト値は 0 を利用
+              map.set(c.candidateId, a);
+              availabilityMapMap.set(u.userId, map);
+            });
+          });
+
           console.log(availabilityMapMap); // TODO 除去する

           res.render('schedule', {
             user: req.user,
             schedule: schedule,
             candidates: candidates,
-            users: [req.user],             
+            users: users,
             availabilityMapMap: availabilityMapMap
           });
         });
       });
     } else {
```
</details>

**解説**

```js
// 閲覧ユーザーと出欠に紐づくユーザーからユーザー Map (キー:ユーザー ID, 値:ユーザー) を作る
const userMap = new Map(); // key: userId, value: User
userMap.set(parseInt(req.user.id), {
    isSelf: true,
    userId: parseInt(req.user.id),
    username: req.user.username
});
availabilities.forEach((a) => {
  userMap.set(a.user.userId, {
    isSelf: parseInt(req.user.id) === a.user.userId, // 閲覧ユーザー自身であるかを含める
    userId: a.user.userId,
    username: a.user.username
  });
});
```

ここでは、ユーザー Map を作成し、まずはユーザー自身を加えている。  
さらにその後、出欠のデータを一つでも持っていたユーザーをユーザー Map に含めている。

なおここでは、閲覧ユーザーであるかどうかを示す `isSelf` というフラグも作成している。  
このフラグは今後、出欠情報を更新する UI を表示させていいかどうかの判断に利用する。

```js
// 全ユーザー、全候補で二重ループしてそれぞれの出欠の値がない場合には、「欠席」を設定する
const users = Array.from(userMap).map((keyValue) => keyValue[1]);
users.forEach((u) => {
  candidates.forEach((c) => {
    const map = availabilityMapMap.get(u.userId) || new Map();
    const a = map.get(c.candidateId) || 0; // デフォルト値は 0 を利用
    map.set(c.candidateId, a);
    availabilityMapMap.set(u.userId, map);
  });
});
```

出欠情報を持つ全ユーザー、全候補で二重ループを実行し、出欠データを更新していく。  
出欠情報が存在しない場合は、デフォルト値の 0 を利用する。  
この 0 が欠席を表すことになる。

ここまで実装できたら、サーバーを再起動して、何かしら予定を作成し、その予定の内容を表示するページにアクセスしてみよう。すると、

```
Map { '15885373' => Map { 13 => 0, 14 => 0, 15 => 0 } }
```

以上のように出欠 MapMap の内容が標準出力に表示されるはず。

### テンプレートに出欠情報を対応させる
ここまでで出欠情報が用意できたので、次はそれをテンプレートで利用しよう。  
`views/schedule.pug` を以下のように修正する。

<details close>
<summary>views/schedule.pug</summary>

```diff
       tr
         th #{candidate.candidateName}
         each user in users
+          - var availability = availabilityMapMap.get(user.userId).get(candidate.candidateId)
+          - var availabilityLabels = ['欠', '？', '出'];
           td
-            button 欠席
+            if user.isSelf
+              button #{availabilityLabels[availability]}
+            else
+              p #{availabilityLabels[availability]}
```
</details>

出欠の状況に応じて「欠」「？」「出」のいずれかが表示されるようにした。  
また、閲覧ユーザーの列である場合のみ、button 要素を利用するようにしている。

<details close>
<summary>routes/schedules.js</summary>

```diff
             });
           });
 
-          console.log(availabilityMapMap); // TODO 除去する
-
           res.render('schedule', {
             user: req.user,
             schedule: schedule,
```

`routes/schedules.js` から `console.log` は取り除く。
</details>

### 出欠更新の Web API の実装
次は、この出欠の更新を行うための Web API を実装する。  
出欠表から Ajax でこの出欠表の更新が行えるようにするためである。

出欠の更新は Ajax にて以下のパスで行われる設計になっていた。

```
/schedules/:scheduleId/users/:userId/candidates/:candidateId
```

これに沿って実装していく。`routes/availabilities.js` を以下のように実装する。

<details close>
<summary>routes/availabilities.js</summary>

```js
'use strict';
const express = require('express');
const router = express.Router();
const authenticationEnsurer = require('./authentication-ensurer');
const Availability = require('../models/availability');

router.post('/:scheduleId/users/:userId/candidates/:candidateId', authenticationEnsurer, (req, res, next) => {
  const scheduleId = req.params.scheduleId;
  const userId = req.params.userId;
  const candidateId = req.params.candidateId;
  let availability = req.body.availability;
  availability = availability ? parseInt(availability) : 0;

  Availability.upsert({
    scheduleId: scheduleId,
    userId: userId,
    candidateId: candidateId,
    availability: availability
  }).then(() => {
    res.json({ status: 'OK', availability: availability });
  });
});

module.exports = router;
```
</details>

以上では、パスから予定 ID、ユーザー ID、候補 ID を受け取り、POST のリクエストに含まれる `availaility` というプロパティで、DB を更新する実装になっている。

もちろん API の利用には認証を求めるようにしておく（`authenticationEnsurer`）。

また、更新後は JSON で `{status: 'OK', availability: ${availability} }` という値が戻るようにしてある。

以上で実装した Router オブジェクトを、Application オブジェクトに設定しておこう。  
`app.js` を以下のように実装する。

<details close>
<summary>app.js</summary>

```diff
 var loginRouter = require('./routes/login');
 var logoutRouter = require('./routes/logout');
 var schedulesRouter = require('./routes/schedules');
+var availabilitiesRouter = require('./routes/availabilities');
 
 var app = express();
```

```diff
 app.use('/logout', logoutRouter);
 app.use('/schedules', schedulesRouter));
+app.use('/schedules', availabilitiesRouter);
 
 app.get('/auth/github',
   passport.authenticate('github', { scope: ['user:email'] }),
```
</details>

これで Router オブジェクトが使われるように設定できた。

### 出欠更新のテストの実装
先ほど実装した Web API がちゃんと実行されるか、テストを実装して確かめる。  
`test/test.js` を以下のように実装していく。

<details close>
<summary>test/test.js</summary>

```diff
 const User = require('../models/user');
 const Schedule = require('../models/schedule');
 const Candidate = require('../models/candidate');
+const Availability = require('../models/availability');
 
 describe('/login', () => {
   before(() => {
```

```diff
             .expect(/テスト候補2/)
             .expect(/テスト候補3/)
             .expect(200)
-            .end((err, res) => {
-              if (err) return done(err);
-              // テストで作成したデータを削除
-              const scheduleId = createdSchedulePath.split('/schedules/')[1];
-              Candidate.findAll({
-                where: { scheduleId: scheduleId }
-              }).then((candidates) => {
-                const promises = candidates.map((c) => { return c.destroy(); });
-                Promise.all(promises).then(() => {
-                  Schedule.findByPk(scheduleId).then((s) => { 
-                    s.destroy().then(() => { 
-                      done(); 
-                    });
-                  });
-                });
-              });
-            });
+            .end((err, res) => { deleteScheduleAggregate(createdSchedulePath.split('/schedules/')[1], done, err);});
         });
     });
   });
});
+
+describe('/schedules/:scheduleId/users/:userId/candidates/:candidateId', () => {
+  before(() => {
+    passportStub.install(app);
+    passportStub.login({ id: 0, username: 'testuser' });
+  });
+
+  after(() => {
+    passportStub.logout();
+    passportStub.uninstall(app);
+  });
+
+  it('出欠が更新できる', (done) => {
+    User.upsert({ userId: 0, username: 'testuser' }).then(() => {
+      request(app)
+        .post('/schedules')
+        .send({ scheduleName: 'テスト出欠更新予定1', memo: 'テスト出欠更新メモ1', candidates: 'テスト出欠更新候補1' })
+        .end((err, res) => {
+          const createdSchedulePath = res.headers.location;
+          const scheduleId = createdSchedulePath.split('/schedules/')[1];
+          Candidate.findOne({
+            where: { scheduleId: scheduleId }
+          }).then((candidate) => {
+            // 更新がされることをテスト
+            const userId = 0;
+            request(app)
+              .post(`/schedules/${scheduleId}/users/${userId}/candidates/${candidate.candidateId}`)
+              .send({ availability: 2 }) // 出席に更新
+              .expect('{"status":"OK","availability":2}')
+              .end((err, res) => { deleteScheduleAggregate(scheduleId, done, err); });
+          });
+        });
+    });
+  });
+});

+function deleteScheduleAggregate(scheduleId, done, err) {
+  Availability.findAll({
+    where: { scheduleId: scheduleId }
+  }).then((availabilities) => {
+    const promises = availabilities.map((a) => { return a.destroy(); });
+    Promise.all(promises).then(() => {
+      Candidate.findAll({
+        where: { scheduleId: scheduleId }
+      }).then((candidates) => {
+        const promises = candidates.map((c) => { return c.destroy(); });
+        Promise.all(promises).then(() => {
+          Schedule.findByPk(scheduleId).then((s) => { 
+            s.destroy().then(() => { 
+              if (err) return done(err);
+              done(); 
+            });
+          });
+        });
+      });
+    });
+  });
+}
```
</details>

今回新たに `deleteScheduleAggregate` という関数を用意して、予定、そこに紐づく出欠・候補を削除するためのメソッドを切り出して、「予定が作成でき、表示される」と「出欠が更新できる」の両方でテストの最後に実行できるように実装した。

Aggregate は集約という意味の英単語であるが、特定の親のデータモデルが他のデータモデルを所有するという関係を持っていることを表したもの。

```js
.end((err, res) => { deleteScheduleAggregate(createdSchedulePath.split('/schedules/')[1], done);});
```

以上の実装で、テストで作成した予定と、そこに紐づく情報を削除するメソッドを呼び出している。引数としては `scheduleId` と `done` の 2 つが渡されている。

追加された `deleteScheduleAggregate` 関数を見てみよう。

```js
function deleteScheduleAggregate(scheduleId, done, err) {
  Availability.findAll({
    where: { scheduleId: scheduleId }
  }).then((availabilities) => {
    const promises = availabilities.map((a) => { return a.destroy(); });
    Promise.all(promises).then(() => {
      Candidate.findAll({
        where: { scheduleId: scheduleId }
      }).then((candidates) => {
        const promises = candidates.map((c) => { return c.destroy(); });
        Promise.all(promises).then(() => {
          Schedule.findByPk(scheduleId).then((s) => { 
            s.destroy().then(() => { 
              if (err) return done(err);
              done(); 
            });
          });
        });
      });
    });
  });
}
```

この関数では、まず最初に `scheduleId` を元にすべての出欠を取得している。  
その後、そのすべてを `destroy` 関数で削除し、その結果の `Promise` オブジェクトの配列を取得する。

その後は候補を削除し、同様にそのすべての候補が削除された結果として渡される Promise オブジェクトの配列を同様に一つの Promise にし、候補が全て削除されたら、さらに予定を削除するように実装している。

> 削除の順番まとめ：availabilities（出欠）-> candidates（候補）-> schedules（予定）

なお、このような親子関係のあるデータの削除方法にはコツがある。  
基本的には子ども側のデータから消していくことで、DB の処理が仮に途中で止まってしまっても、データが不正な状況を防ぐことができるのである。

そこで、ここでも最も子供のデータである「出欠」を消し、次にその親のデータの「候補」を消し、最後に「予定」を消す、とすることで、データが不正になる問題にたいしょ　している。

なおここでは `const` で `promises` という変数を二度宣言している。  
`const` による変数宣言では `{` と `}` で囲まれた **スコープ** と呼ばれる範囲を考え、同名の変数がある場合、変数の利用と同じスコープにある宣言が優先して利用されるようになっている。

そして、次は「出欠が更新できる」のテスト。

```js
describe('/schedules/:scheduleId/users/:userId/candidates/:candidateId', () => {
  before(() => {
    passportStub.install(app);
    passportStub.login({ id: 0, username: 'testuser' });
  });

  after(() => {
    passportStub.logout();
    passportStub.uninstall(app);
  });
```

以上の実装は、利用するパスの記述と、テストの実施の際にログインとログアウトを行う実装。

```js
  it('出欠が更新できる', (done) => {
    User.upsert({ userId: 0, username: 'testuser' }).then(() => {
      request(app)
        .post('/schedules')
        .send({ scheduleName: 'テスト出欠更新予定1', memo: 'テスト出欠更新メモ1', candidates: 'テスト出欠更新候補1' })
        .end((err, res) => {
          const createdSchedulePath = res.headers.location;
          const scheduleId = createdSchedulePath.split('/schedules/')[1];
          Candidate.findOne({
            where: { scheduleId: scheduleId }
          }).then((candidate) => {
            // 更新がされることをテスト
            const userId = 0;
            request(app)
              .post(`/schedules/${scheduleId}/users/${userId}/candidates/${candidate.candidateId}`)
              .send({ availability: 2 }) // 出席に更新
              .expect('{"status":"OK","availability":2}')
              .end((err, res) => { deleteScheduleAggregate(scheduleId, done, err); });
          });
        });
    });
  });
```

上記では、まずは `/schedules` に POST を行い「予定」と「候補」を作成する。その後、

```js
Candidate.findOne({
  where: { scheduleId: scheduleId }
}).then((candidate) => {
```

以上の部分で「予定」に関連する候補を取得し、その「候補」に対して、POST で Web API に対して欠席を出席に更新している。

そしてそのリクエストのレスポンスに `'{"status":"OK","availability":2}'` が含まれるかどうかをテストしている。

以上のテストが実装できたら、`yarn test` でテストを実行する。テストがすべて通ればこの Web API の実装は問題なく行えていることがわかる。

----
ここまでの「出欠が変更できる」のテストは、Web API のインターフェースが返した結果をそのまま信用して結果を判断している。  
本当に DB に保存されたかどうかはテストされていない。  
本当に DB に存在するかどうかのテストも追加しよう。

<details close>
<summary>test/test.js</summary>

```diff
'use strict';
 const request = require('supertest');
+const assert = require('assert');
 const app = require('../app');
 const passportStub = require('passport-stub');
 const User = require('../models/user');
```

```diff
               .post(`/schedules/${scheduleId}/users/${userId}/candidates/${candidate.candidateId}`)
               .send({ availability: 2 }) // 出席に更新
               .expect('{"status":"OK","availability":2}')
-              .end((err, res) => { deleteScheduleAggregate(scheduleId, done, err); });
+              .end((err, res) => {
+                Availability.findAll({
+                  where: { scheduleId: scheduleId }
+                }).then((availabilities) => {
+                  assert.equal(availabilities.length, 1);
+                  assert.equal(availabilities[0].availability, 2);
+                  deleteScheduleAggregate(scheduleId, done, err);
+                });
+              });
           });
         });
     });
```
</details>

以上のように記述することで、出欠更新のリクエストを実行した後、 データベースの値を取得して、その内容をテストすることができる。  
予定に関連する出欠情報がひとつあることと、その内容が更新された `2` であることをテストしている。

`Availability.findAll` 関数はデータベースから where で条件を指定した全ての出欠を取得する。  
また結果オブジェクトの `then` 関数を呼び出すことで、そこで渡す無名関数の引数 `availabilities` には、 出欠のモデルである `models/availability.js` で定義したモデルの配列が渡される。

よってここでは、 `availabilities` の配列の長さを検証し、その内容も検証する実装を `assert` モジュールを利用して書いている。

## § 20. コメントの表示と更新
前回までで Web API を利用した出欠の更新ができるようになった。  
今度はこれを Ajax でクライアントの JavaScript から出欠の更新を利用できるようにしていく。

すでに習ったクライアントサイドのフレームワークである webpack と DOM 操作と Ajax のライブラリである jQuery を 利用して実装していく。

また、出欠の更新が実装できたら、同じ Ajax の仕組みを利用して、 予定に対するコメントの更新も行えるようにしていく。

### Ajax による出欠の更新
すでに、出欠表には「欠」というラベルのボタンが表示されるようになっているため、そのボタンを押した時に、

- 「欠」ならば「？」に更新
- 「？」ならば「出」に更新
- 「出」ならば「欠」に更新

という挙動となるように実装していく。

まずは webpack と babel、そして jQuery をインストールする。

```bash
yarn add webpack@4.26.1 webpack-cli@3.1.2 @babel/core@7.1.6 @babel/preset-env@7.1.6 babel-loader@8.0.4 --dev
yarn add jquery@3.4.1
```

これでインストールを行い、以下のコマンドで必要なディレクトリとファイルを作成する。

```bash
touch webpack.config.js
mkdir app
echo "'use strict';" > app/entry.js
mkdir public/javascripts
```

また、`webpack.config.js` を以下のように設定しよう。

<details close>
<summary>webpack.config.js</summary>

```js
module.exports = {
  context: __dirname + '/app',
  mode: 'none',
  entry: './entry',
  output: {
    path: __dirname + '/public/javascripts',
    filename: 'bundle.js'
  },
  module: {
    rules: [{
      test: /\.js$/,
      exclude: /node_modules/,
      use: {
        loader: 'babel-loader',
        options: {
          presets: ['@babel/preset-env']
        }
      }
    }]
  }
};
```

</details>

以上を実装して準備ができたら以下を実行して、クライアント用の `public/javascripts/bundle.js` ファイルを作成する。

```bash
node_modules/.bin/webpack
# 上記でエラーが出た時は代わりに以下
# node node_modules/webpack/bin/webpack.js
```

次に、HTML からこの JavaScript を利用できるようにする。  
すべての HTML で共通して利用できるように `views/layout.pug` で以下のように実装してしまおう。

```diff
     link(rel='stylesheet', href='/stylesheets/style.css')
   body
     block content
+    script(src="/javascripts/bundle.js")
```
body 要素の最後に script 要素を追記している。 これで `public/javascripts/bundle.js` が全てのページで読み込まれるようになる。

そして、今度は出欠ボタンのオブジェクトを取得できるように、 出欠表を表示しているテンプレート `views/schedule.pug` にも以下のように変更をいれる。

<details close>
<summary>views/schedule.pug</summary>

```diff
           - var availabilityLabels = ['欠', '？', '出'];
           td
             if user.isSelf
-              button #{availabilityLabels[availability]}
+              button(
+                data-schedule-id=schedule.scheduleId
+                data-user-id=user.userId
+                data-candidate-id=candidate.candidateId
+                data-availability=availability).availability-toggle-button #{availabilityLabels[availability]}
             else
               p #{availabilityLabels[availability]}
```
</details>

`button` 要素に `class` を設定した。  
また、`data-` で始まる名前の属性を設定して、Ajax での通信に利用できるようにしている。

この `data-*` 属性は、HTML の要素に独自のデータを保有させたい場合に利用する属性となっている。  
ここでは、

- 予定 ID
- ユーザー ID
- 候補 ID
- 出欠

これらの情報を button 要素の中に含まれるようにしている。

ではこの button 要素をクリックした際に Web API にアクセスして、出欠を更新するように実装しよう。

`app/entry.js` を以下のように実装する。

<details close>
<summary>app/entry.js</summary>

```js
'use strict';
import $ from 'jquery';

$('.availability-toggle-button').each((i, e) => {
  const button = $(e);
  button.click(() => {
    const scheduleId = button.data('schedule-id');
    const userId = button.data('user-id');
    const candidateId = button.data('candidate-id');
    const availability = parseInt(button.data('availability'));
    const nextAvailability = (availability + 1) % 3;
    $.post(`/schedules/${scheduleId}/users/${userId}/candidates/${candidateId}`,
      { availability: nextAvailability },
      (data) => {
        button.data('availability', data.availability);
        const availabilityLabels = ['欠', '？', '出'];
        button.text(availabilityLabels[data.availability]);
      });
  });
});
```
</details>

すべての出欠ボタンに対して、クリックすることで出欠状態の更新が Web API を通じて実行されるように実装してある。

```js
$('.availability-toggle-button').each((i, e) => {
```

この実装は、jQuery を利用して、 `availability-toggle-button` の `class` が 設定されている要素をセレクタで取得している。  
そして `each` 関数で、各要素に対して、 引数 `i` は順番、 引数 `e` は HTML 要素が渡される関数を実行している。

```js
const button = $(e);
button.click(() => {
```

`$(e)` で、 ボタン要素の jQuery オブジェクトを取得し、そのボタンがクリックした際の関数を記述している。

```js
const scheduleId = button.data('schedule-id');
const userId = button.data('user-id');
const candidateId = button.data('candidate-id');
const availability = parseInt(button.data('availability'));
const nextAvailability = (availability + 1) % 3;
```

以上で、 jQuery の `data` 関数を使用して `data-*` 属性を取得することで、

- 予定 ID
- ユーザー ID
- 候補 ID
- 出欠

を取得している。また、

- 次の出欠の数値

も計算している。 0 → 1 → 2 → 0 → 1 → 2 と循環させたいため、 ここでは 1 を足して 3 の剰余を次の出欠の数値としている。

なお、 `availability` はここで数値の計算をしたいため、 `parseInt` 関数を利用して文字列から数値に変換している。

```js
$.post(`/schedules/${scheduleId}/users/${userId}/candidates/${candidateId}`,
  { availability: nextAvailability },
  (data) => {
    button.data('availability', data.availability);
    const availabilityLabels = ['欠', '？', '出'];
    button.text(availabilityLabels[data.availability]);
  });
```

以上は出欠更新の Web API の呼び出しと、実行結果を受け取って `button` 要素の、 `data-*` 属性を更新し、ボタンのラベルを更新している。

これで実装できたので、実際に動かしてみよう。

```bash
node_modules/.bin/webpack
PORT=8000 yarn start
```

予定のページに進んで、出欠ボタンをクリックするたびにボタンの文字が変わり、サーバー側のコンソールでは UPSERT の SQL が発行されれば成功。

これで出欠表の部分の機能は完成。

### コメントの表示の機能
同様に、予定に対してユーザーが行う湖面tのに対しても、表示と更新の機能を実装していこう。

さっそく出欠表の最後の行に、それぞれのユーザーのコメントが表示できるように実装してみよう。

まずはテンプレートにコメントを表示できるようにする。  
テンプレートに、キーをユーザー ID、値をコメントとした連想配列を渡せばよい。

`routes/schedules.js` を以下のように実装する。

<details close>
<summary>routes.schedules.js</summary>


```diff
 const Candidate = require('../models/candidate');
 const User = require('../models/user');
 const Availability = require('../models/availability');
+const Comment = require('../models/comment');
 
 router.get('/new', authenticationEnsurer, (req, res, next) => {
   res.render('new', { user: req.user });
```

```diff
             });
           });
 
-          res.render('schedule', {
-            user: req.user,
-            schedule: schedule,
-            candidates: candidates,
-            users: users,
-            availabilityMapMap: availabilityMapMap
+          // コメント取得
+          Comment.findAll({
+            where: { scheduleId: schedule.scheduleId }
+          }).then((comments) => {
+            const commentMap = new Map();  // key: userId, value: comment
+            comments.forEach((comment) => {
+              commentMap.set(comment.userId, comment.comment);
+            });
+            res.render('schedule', {
+              user: req.user,
+              schedule: schedule,
+              candidates: candidates,
+              users: users,
+              availabilityMapMap: availabilityMapMap,
+              commentMap: commentMap
+            });
           });
         });
       });
```

以上で、 予定 ID でしぼりこんで取得される予定に関する、全てのコメントを取得している。  
その後、連想配列 `commentMap` に格納し、テンプレートに `commentMap` というプロパティ名で割り当ててテンプレートを描画している。
</details>

ここまできたら、サーバーを再起動してアクセス・ログインし、何かしらの予定の出欠表を表示させてみよう。

まだコメントのデータはないので、ここはとりあえず以下のように、コメントを取得するための SQL が実行できていることだけを確認しよう。

```sql
Executing (default): SELECT "scheduleId", "userId", "comment" FROM "comments" AS "comments" WHERE "comments"."scheduleId" = '8c321079-bc36-4e43-b151-167f6162fb7b';
```

### コメントの更新の Web API の実装
テンプレートに受け渡す部分は、完全には確認できないので途中までの実装として、ここからは、 コメントを更新するための Web API を作ってみよう。

ほとんど出欠の更新の時に書いた実装と同じ。最初に決めた設計では、

```
/schedules/:scheduleId/users/:userId/comments
```

以上のパスでこの Web API が使える設計になっていた。  
では、 `routes/comments.js` を以下のように実装する。

<details close>
<summary>routes/comments.js</summary>

```js
'use strict';
const express = require('express');
const router = express.Router();
const authenticationEnsurer = require('./authentication-ensurer');
const Comment = require('../models/comment');

router.post('/:scheduleId/users/:userId/comments', authenticationEnsurer, (req, res, next) => {
  const scheduleId = req.params.scheduleId;
  const userId = req.params.userId;
  const comment = req.body.comment;

  Comment.upsert({
    scheduleId: scheduleId,
    userId: userId,
    comment: comment.slice(0, 255)
  }).then(() => {
    res.json({ status: 'OK', comment: comment });
  });
});

module.exports = router;
```
</details>

予定 ID、ユーザー ID、コメントを body から `comment` というプロパティ名で取得し、 Comment のデータモデルを利用して、データベースに UPSERT を実施している。  
また、コメントの内容だけは、 255 文字以内になるように `.slice()` で切り取っている。

次にこの Router オブジェクトを Application オブジェクトに登録する。
`app.js` を以下のように実装します。

<details close>
<summary>app.js</summary>

```diff
 var logoutRouter = require('./routes/logout');
 var schedulesRouter = require('./routes/schedules');
 var availabilitiesRouter = require('./routes/availabilities');
+var commentsRouter = require('./routes/comments');
 
 var app = express();
 app.use(helmet());
```

```diff
 app.use('/logout', logoutRouter);
 app.use('/schedules', schedulesRouter);
 app.use('/schedules', availabilitiesRouter);
+app.use('/schedules', commentsRouter);
 
 app.get('/auth/github',
   passport.authenticate('github', { scope: ['user:email'] }),
```
</details>

そして、このコメントが更新できるかどうかのテストコードを書いてみよう。  
`test/test.js` に以下のテストを追記する。  
「出欠が更新できる」の下にこのテストを書き足してしまおう。

<details close>
<summary>test/test.js</summary>

```diff
 const Schedule = require('../models/schedule');
 const Candidate = require('../models/candidate');
 const Availability = require('../models/availability');
+const Comment = require('../models/comment');
 
 describe('/login', () => {
   before(() => {
```

```diff
   });
 });
 
+describe('/schedules/:scheduleId/users/:userId/comments', () => {
+  before(() => {
+    passportStub.install(app);
+    passportStub.login({ id: 0, username: 'testuser' });
+  });
+
+  after(() => {
+    passportStub.logout();
+    passportStub.uninstall(app);
+  });
+
+  it('コメントが更新できる', (done) => {
+    User.upsert({ userId: 0, username: 'testuser' }).then(() => {
+      request(app)
+        .post('/schedules')
+        .send({ scheduleName: 'テストコメント更新予定1', memo: 'テストコメント更新メモ1', candidates: 'テストコメント更新候補1' })
+        .end((err, res) => {
+          const createdSchedulePath = res.headers.location;
+          const scheduleId = createdSchedulePath.split('/schedules/')[1];
+          // 更新がされることをテスト
+          const userId = 0;
+          request(app)
+            .post(`/schedules/${scheduleId}/users/${userId}/comments`)
+            .send({ comment: 'testcomment' })
+            .expect('{"status":"OK","comment":"testcomment"}')
+            .end((err, res) => {
+              Comment.findAll({
+                where: { scheduleId: scheduleId }
+              }).then((comments) => {
+                assert.equal(comments.length, 1);
+                assert.equal(comments[0].comment, 'testcomment');
+                deleteScheduleAggregate(scheduleId, done, err);
+              });
+            });
+        });
+    });
+  });
+});
+
 function deleteScheduleAggregate(scheduleId, done, err) {
+  const promiseCommentDestroy = Comment.findAll({
+    where: { scheduleId: scheduleId }
+  }).then((comments) => { comments.map((c) => { return c.destroy(); });});
+
   Availability.findAll({
     where: { scheduleId: scheduleId }
   }).then((availabilities) => {
```
</details>

以上で、コメントを Web API を利用して予定のコメントを更新し、 結果として更新されたコメントが取得されることと、データベースに実際に保存されていることをテストしている。  
コメントの内容が、 `testcomment` であることには注意が必要だが、 処理や構造は、出欠更新のテストとほとんど同じ内容となっている。

```js
Comment.findAll({
  where: { scheduleId: scheduleId }
}).then((comments) => { comments.map((c) => { return c.destroy(); });});
```

以上は、 `deleteScheduleAggregate` 関数に新たに追加された、予定に関連するコメントの削除処理である。  
厳密には、このコメントを削除し、さらに出欠、候補が全て削除された後に 予定を削除するべきだが、ここではコードを簡単に実装するために、コメントだけを独立して削除している。  
そのためテストの実行状況によっては、予定が存在しないコメントが残ってしまう可能性がある。これはあとで修正を行う。

ではテストを実行しよう。`yarn test` でテストの通過を確認でき、SQL も

```
Executing (default): CREATE OR REPLACE FUNCTION pg_temp.sequelize_upsert() RETURNS integer AS $func$ BEGIN INSERT INTO "comments" ("scheduleId","userId","comment") VALUES ('59e32c9f-87cf-4a9c-a07a-1aea2ccfdbb8','0','testcomment'); RETURN 1; EXCEPTION WHEN unique_violation THEN UPDATE "comments" SET "scheduleId"='59e32c9f-87cf-4a9c-a07a-1aea2ccfdbb8',"userId"='0',"comment"='testcomment' WHERE (("scheduleId" = '59e32c9f-87cf-4a9c-a07a-1aea2ccfdbb8' AND "userId" = '0')); RETURN 2; END; $func$ LANGUAGE plpgsql; SELECT * FROM pg_temp.sequelize_upsert();
```

```
Executing (default): DELETE FROM "comments" WHERE ("scheduleId","userId") IN (SELECT "scheduleId","userId" FROM "comments" WHERE "scheduleId" = '59e32c9f-87cf-4a9c-a07a-1aea2ccfdbb8' AND "userId" = 0 LIMIT 1)
```

以上の二つ（それぞれコメントの追加更新にかかわる SQL とコメントの削除に関わる SQL）が表示されていれば問題ない。  
これで、コメント更新の Web API が完成。

### Ajax によるコメントの更新
ではこの回の最後に、コメントの更新の UI も加えてしまおう。  
まずは `views/schedule.pug` を以下のように編集する。

<details close>
<summary>views/schedule.pug</summary>

```diff
 extends layout
 
 block content
   h4 #{schedule.scheduleName}
   p(style="white-space:pre;") #{schedule.memo}
   p 作成者: #{schedule.user.username}
   h3 出欠表
   table
     tr
       th 予定
       each user in users
         th #{user.username}
     each candidate in candidates
       tr
         th #{candidate.candidateName}
         each user in users
           - var availability = availabilityMapMap.get(user.userId).get(candidate.candidateId)
           - var availabilityLabels = ['欠', '？', '出'];
           td
             if user.isSelf
               button
                 data-schedule-id="#{schedule.scheduleId}"
                 data-user-id=user.userId
                 data-candidate-id=candidate.candidateId
                 data-availability=availability).availability-toggle-button #{availabilityLabels[availability]}
             else
               p #{availabilityLabels[availability]}
+    tr
+      th コメント
+      each user in users
+        if user.isSelf
+          td
+            p#self-comment #{commentMap.get(user.userId)}
+            button(
+              data-schedule-id=schedule.scheduleId
+              data-user-id=user.userId)#self-comment-button 編集
+        else
+          td
+            p #{commentMap.get(user.userId)}
```
</details>

コメント行を出欠表の末尾につけて、閲覧者自体のコメントの下には編集ボタンを配置した。  
`button` 要素には、 `data-*` 属性のほか、 id として、 `self-comment-button` をつけてある。

加えてコメント自体を表示する段落の p 要素にも、`self-comment` という id をつけた。  
class でなく id を利用したのは、基本的にこのページには自身のコメントは一つしかないため。

次にこの HTML に合わせて、 `app/entry.js` を編集する。

<details close>
<summary>app/entry.js</summary>

```diff
         button.text(availabilityLabels[data.availability]);
       });
   });
 });
+
+const buttonSelfComment = $('#self-comment-button');
+buttonSelfComment.click(() => {
+  const scheduleId = buttonSelfComment.data('schedule-id');
+  const userId = buttonSelfComment.data('user-id');
+  const comment = prompt('コメントを255文字以内で入力してください。');
+  if (comment) {
+    $.post(`/schedules/${scheduleId}/users/${userId}/comments`,
+      { comment: comment },
+      (data) => {
+        $('#self-comment').text(data.comment);
+      });
+  }
+});
```
</details>

基本的に出欠更新の処理と同じだが、 `prompt` 関数を利用してコメントを入力できるようにしている。

実際に動かしてみよう。

```bash
node_modules/.bin/webpack
PORT=8000 yarn start
```

以上のコマンドを実行して JavaScript を出力してからサーバーを起動する。  
予定に対してコメントが付けられるかどうかを試してみよう。編集ボタンを押して、ダイアログに入力したコメントが反映され、 再読み込みをしてもそのコメントが表示されれば、全ての実装がうまくいっている。

### Promise チェーンを利用した書き直し
すでに Promise の then 関数について学んだが、ここまでは非同期の処理の結果を引数に持つ関数を then に渡して結果を利用することしか紹介していなかった。

しかし、この then 関数の引数に渡す関数では、return で値を返すことができる。  
また仮にその return で返す値が Promise であっても問題なく処理を行ってくれるという特徴を持っている。

ここで簡単な実装例を紹介しよう。

```js
new Promise((resolve) => {
  resolve(2);
}).then((v1) => { // v1 は 2
  new Promise((resolve) => {
    resolve(v1 * 3);
  }).then((v2) => { // v2 は 6
    new Promise((resolve) => {
      resolve(v2 * 4);
    }).then((v3) => { // v3 は 24
      console.log(v3); // 24 が出力される
    });
  });
});
```

Promise オブジェクトは、`new` で新しくオブジェクトを作成し、その際の引数に resolve 関数という非同期の結果を確定させる関数を呼び出すことで作成できる。

```js
new Promise((resolve) => {
  resolve(2);
})
```

以上の実装は、結果が 2 となる Promise オブジェクトを表している。

今までの書き方では `then` をつなげて書いていたため、深い入れ子構造の then 関数の呼び出しとなっていた。そのため、

```js
}).then((v1) => { // v1 は 2
  new Promise((resolve) => {
    resolve(v1 * 3);
  }).then((v2) => { // v2 は 6
    new Promise((resolve) => {
      resolve(v2 * 4);
    }).then((v3) => { // v3 は 24
      console.log(v3); // 24 が出力される
    });
```

このような深い `then` の入れ子構造になる。

この例では、Promise オブジェクトが 3 回発生するため、3 つの入れ子構造になった結果、24 が標準出力に出力される。

これを then 関数に渡す関数内で return を利用すると、以下のように書き直せる。

```js
new Promise((resolve) => {
  resolve(2);
}).then((v1) => { // v1 は 2
  return new Promise((resolve) => {
    resolve(v1 * 3);
  });
}).then((v2) => { // v2 は 6
  return new Promise((resolve) => {
    resolve(v2 * 4);
  });
}).then((v3) => { // v3 は 24
  console.log(v3); // 24 が出力される
});
```

以上はまったく同じ処理だが、return で Promise オブジェクトを返すことで、次の then 関数の引数では、その結果をそのまま受け取ることができる。

ではこのテクニックを使っていくつか、3 階層以上の深い入れ子になってしまった Promise の then 関数を書き直してみよう。

最初に、`test/test.js` の deleteScheduleAggregate 関数を書き直す。

<details close>
<summary>test/test.js</summary>

```js
function deleteScheduleAggregate(scheduleId, done, err) {
  const promiseCommentDestroy = Comment.findAll({
    where: { scheduleId: scheduleId }
  }).then((comments) => {
    return Promise.all(comments.map((c) => { return c.destroy(); }));
  });

  Availability.findAll({
    where: { scheduleId: scheduleId }
  }).then((availabilities) => {
    const promises = availabilities.map((a) => { return a.destroy(); });
    return Promise.all(promises);
  }).then(() => {
    return Candidate.findAll({
      where: { scheduleId: scheduleId }
    });
  }).then((candidates) => {
    const promises = candidates.map((c) => { return c.destroy(); });
    promises.push(promiseCommentDestroy);
    return Promise.all(promises);
  }).then(() => {
    return Schedule.findByPk(scheduleId).then((s) => { return s.destroy(); });
  }).then(() => {
    if (err) return done(err);
    done();
  });
}
```
</details>

深かった入れ子が解消された。
まず最初に、

```js
const promiseCommentDestroy = Comment.findAll({
  where: { scheduleId: scheduleId }
}).then((comments) => {
  return Promise.all(comments.map((c) => { return c.destroy(); }));
});
```

ここでコメントが削除された Promise オブジェクトを作成している。
その後、

```js
const promises = availabilities.map((a) => { return a.destroy(); });
return Promise.all(promises);
```

then 関数の中で、全ての出欠の削除の結果をあらわす Promise オブジェクトを return で返している。

その後は、

```js
return Candidate.findAll({
  where: { scheduleId: scheduleId }
});
```

全ての候補が取得できたことの Promise オブジェクトを返し、

```js
const promises = candidates.map((c) => { return c.destroy(); });
promises.push(promiseCommentDestroy);
return Promise.all(promises);
```

ここで、全ての候補が削除され、かつ、全てのコメントが削除されたことを示す Promise オブジェクトを作成して return 句で返している。

このようにして入れ子の形をチェインの形に書き直すことができる。

なお、先ほどの実装ではコメントが削除されずに残ってしまう可能性があったが、 今回の実装ではそのようなことがないように改善されている。

このように、処理結果は変えずに、 処理内容を改善したりコードを扱いやすくすることを **リファクタリング** という。

ではテストを実行してみよう。

`yarn test` でテストを実行して、`6 passing` と表示されてテストが無事成功し、

```sql
DELETE FROM "availabilities"
DELETE FROM "comments"
DELETE FROM "candidates"
DELETE FROM "schedules"
```

これらで始まる削除 SQL が全て実行されていればリファクタリングは成功。

次は routes/schedules.js の /:scheduleId の GET のハンドラをリファクタリングしてみよう。
以下のように書き換えることができる。

<details close>
<summary>routes/schedules.js</summary>

```js
router.get('/:scheduleId', authenticationEnsurer, (req, res, next) => {
  let storedSchedule = null;
  let storedCandidates = null;
  Schedule.findOne({
    include: [
      {
        model: User,
        attributes: ['userId', 'username']
      }],
    where: {
      scheduleId: req.params.scheduleId
    },
    order: [['"updatedAt"', 'DESC']]
  }).then((schedule) => {
    if (schedule) {
      storedSchedule = schedule;
      return Candidate.findAll({
        where: { scheduleId: schedule.scheduleId },
        order: [['"candidateId"', 'ASC']]
      });
    } else {
      const err = new Error('指定された予定は見つかりません');
      err.status = 404;
      next(err);
    }
  }).then((candidates) => {
    // データベースからその予定の全ての出欠を取得する
    storedCandidates = candidates;
    return Availability.findAll({
      include: [
        {
          model: User,
          attributes: ['userId', 'username']
        }
      ],
      where: { scheduleId: storedSchedule.scheduleId },
      order: [[User, '"username"', 'ASC'], ['"candidateId"', 'ASC']]
    });
  }).then((availabilities) => {
    // 出欠 MapMap(キー:ユーザー ID, 値:出欠Map(キー:候補 ID, 値:出欠)) を作成する
    const availabilityMapMap = new Map(); // key: userId, value: Map(key: candidateId, availability)
    availabilities.forEach((a) => {
      const map = availabilityMapMap.get(a.user.userId) || new Map();
      map.set(a.candidateId, a.availability);
      availabilityMapMap.set(a.user.userId, map);
    });

    // 閲覧ユーザーと出欠に紐づくユーザーからユーザー Map (キー:ユーザー ID, 値:ユーザー) を作る
    const userMap = new Map(); // key: userId, value: User
    userMap.set(parseInt(req.user.id), {
      isSelf: true,
      userId: parseInt(req.user.id),
      username: req.user.username
    });
    availabilities.forEach((a) => {
      userMap.set(a.user.userId, {
        isSelf: parseInt(req.user.id) === a.user.userId, // 閲覧ユーザー自身であるかを含める
        userId: a.user.userId,
        username: a.user.username
      });
    });

    // 全ユーザー、全候補で二重ループしてそれぞれの出欠の値がない場合には、「欠席」を設定する
    const users = Array.from(userMap).map((keyValue) => keyValue[1]);
    users.forEach((u) => {
      storedCandidates.forEach((c) => {
        const map = availabilityMapMap.get(u.userId) || new Map();
        const a = map.get(c.candidateId) || 0; // デフォルト値は 0 を利用
        map.set(c.candidateId, a);
        availabilityMapMap.set(u.userId, map);
      });
    });

    // コメント取得
    return Comment.findAll({
      where: { scheduleId: storedSchedule.scheduleId }
    }).then((comments) => {
      const commentMap = new Map();  // key: userId, value: comment
      comments.forEach((comment) => {
        commentMap.set(comment.userId, comment.comment);
      });
      res.render('schedule', {
        user: req.user,
        schedule: storedSchedule,
        candidates: storedCandidates,
        users: users,
        availabilityMapMap: availabilityMapMap,
        commentMap: commentMap
      });
    });
  });
});
```

</details>

Candidate.findAll の結果と Availability.findAll の結果をそれぞれ then 関数の return で返すようにした。

他の Promise オブジェクトへの処理をまたいで利用したい値である

- データベースに保存されている予定
- データベースに保存されている候補

以上 2 つのデータは、

```js
let storedSchedule = null;
let storedCandidates = null;
```

以上のように then に渡す関数のスコープの外側に変数宣言をすることで、 ほかの then に渡している関数からも利用できるようにしている。

これもまた、深い入れ子構造になっていた関数を浅くすることができた。

`yarn test` で、全てのテストが問題なく実行され、`6 passing` というように、エラーが生じずに全てのテストが実行できれば、このリファクタリングは完了。

残る機能要件は、予定の編集と削除だけとなった。

----
**練習**

1 秒後に整数 11 を返す関数 getA と、
1 秒後に整数 13 を返す関数 getB と、
1 秒後に整数 17 を返す関数 getC の 3 つの関数があります。

<details close>
<summary>コード</summary>

```js
function getA() {
  return new Promise((resolve) => {
    setTimeout(() => { resolve(11); }, 1000);
  });
}

function getB() {
  return new Promise((resolve) => {
    setTimeout(() => { resolve(13); }, 1000);
  });
}

function getC() {
  return new Promise((resolve) => {
    setTimeout(() => { resolve(17); }, 1000);
  });
}
```
</details>

それぞれ答えとなる整数値を Promise オブジェクトとして返す。

この 3 つの関数の結果をかけあわせた数値を、 Promise チェインを一度は利用して標準出力に出力してみよ。
2431 が答えとして表示されるようにすること。

なお、getA 関数と getB 関数の答えを返す Promise オブジェクトは、

```js
getA().then((a) => {
  return getB().then((b) => { return a * b; });
})
```

以上の記述で得られる。
標準出力には、 console.log 関数を利用すること。

**答え**

index.js

```js
getA().then((a) => {
  return getB().then((b) => { return a * b; });
}).then((result) => {
  getC().then((c) => { console.log(result * c); });
});
```

以上の実装で、 node index.js の結果、 2431 が得られる。

getA と getB をかけた答えから作られる Promise オブジェクトの then 関数を呼び出し、 その結果と getC 関数で得られる答えをかけあわせたものを console.log で出力している。

ちなみに、前回学習した `Promise.all` 関数を使うと、例えば次のように書ける。

```js
Promise.all([getA(), getB(), getC()]).then((results) => {
    console.log(results[0] * results[1] * results[2]);
});
```

こちらの実装では、getA, getB, getC の Promise すべてが同時に開始されるため、上の実装例とは違い約 1 秒で終了する。

また、さらに進んだ内容だが、async/await という構文を使うと、次のように簡潔に書くことができる。

```js
getA().then(async(a) => {
  const b = await getB();
  const c = await getC();
  console.log(a * b * c);
});
```

この実装は、匿名関数を `async` 関数とし、 `await` がついている式の Promise を順番に待つ。getA, getB, getC が順番に実行されるため、終了まで 3 秒かかる。

## § 21. 予定の編集と削除
ここまでで、出欠表が持つ機能の実装が完了した。  
今回は、予定の内容の編集と削除を実装していく。  
これが終われば、機能要件はすべて実装完了になる。

### 予定の編集で更新できる属性
ひとことに予定の編集と言っても、編集できる属性の項目は決めなくてはいけない。  
編集したい内容を考えると、

- 予定名
- メモ

以上は編集したいだろう。加えて編集したいものといえば、

- 予定の候補

がある。

ただし、ここでよく考える必要がある。  
すでに予定の候補の編集という機能は、他の要件と衝突してしまっているのである。

例えば、A さんが作った予定に「 1 月 1 日」という名前の予定の候補があったとする。  
その後、A さん、 B さん、C さんが、「出席」という出欠情報を入力したとしよう。

その後、Aさんが「 1 月 1 日」を「 1 月 2 日」に編集した場合を考える。
その場合も、B さん、 C さんの出欠情報は出欠のまま出欠表に入力されたままとなっている。
これは運用上のトラブルを発生させかねない。

結果として B さん、C さんは、 1 月 2 日に予定かぶりを発生させてしまうことになるだろう。

この問題からわかるのは、予定の候補の編集は、ただ編集だけできるだけでは成立しないということ。

候補の編集機能に加えて、予定を欠席の状態に戻す機能や、 候補に変更があったことを通知したりするなどの別な機能がないと、 運用上の問題が起こってしまうのである。

そのため、ここの編集では、

- 新たな予定の候補を追加できる

という要件として実装していこう。

また、その際には既存の候補も一緒に予定編集フォームに表示してあげることで、 編集の利便性を高めることができそう。

予定の編集の要件が固まったところで、実際に予定編集フォームを作成していく。

### 予定編集フォームの実装
まずはテンプレートファイル `views/edit.pug` を作成する。

このテンプレートには、

- `schedule` というプロパティ名の予定のモデル
- `candidates` というプロパティ名で候補のモデルの配列

が割り当てられる前提で実装をしていく。

<details close>
<summary>views/edit.pug</summary>

```pug
extends layout

block content
  h3 予定の編集
  form(method="post", action=`/schedules/${schedule.scheduleId}?edit=1`)
    div
      h5 予定名
      input(type="text" name="scheduleName" value=schedule.scheduleName)
    div
      h5 メモ
      textarea(name="memo") #{schedule.memo}
    div
      label 既存の候補日程
      ul
        each candidate in candidates
          li #{candidate.candidateName}
      p 候補日程の追加 (改行して複数入力してください)
      textarea(name="candidates")
    div
      button(type="submit") 以上の内容で予定を編集する
```

ほとんど予定作成フォームと同じ構造になっているが、既存の候補日程も表示できるようになっている。  
また、フォームの投稿を受け付ける URL のパスは

```
/schedules/:scheduleId?edit=1
```
となっているため、そのように実装した。

</details>

では、これに合わせたハンドラを Router オブジェクトに登録してしまおう。
`routes/schedules.js` を以下のように編集する。

<details close>
<summary>routes/schedules.js</summary>

```diff
     });
   });
 });
+
+router.get('/:scheduleId/edit', authenticationEnsurer, (req, res, next) => {
+  Schedule.findOne({
+    where: {
+      scheduleId: req.params.scheduleId
+    }
+  }).then((schedule) => {
+    if (isMine(req, schedule)) { // 作成者のみが編集フォームを開ける
+      Candidate.findAll({
+        where: { scheduleId: schedule.scheduleId },
+        order: [['"candidateId"', 'ASC']]
+      }).then((candidates) => {
+        res.render('edit', {
+          user: req.user,
+          schedule: schedule,
+          candidates: candidates
+        });
+      });
+    } else {
+      const err = new Error('指定された予定がない、または、予定する権限がありません');
+      err.status = 404;
+      next(err);
+    }
+  });
+});
+
+function isMine(req, schedule) {
+  return schedule && parseInt(schedule.createdBy) === parseInt(req.user.id);
+}
+
 module.exports = router;
```
</details>

**解説**

```js
router.get('/:scheduleId/edit', authenticationEnsurer, (req, res, next) => {
```

まず URL は、予定表示のページの末尾に `/edit` を加えたものとしている。

```js
Schedule.findOne({
  where: {
    scheduleId: req.params.scheduleId
  }
}).then((schedule) => {
```

ここで、指定された予定 ID の予定を取得。

```js
if (isMine(req, schedule)) { // 作成者のみが編集フォームを開ける
```

isMine という関数を別途用意して、自身の予定であればその後の処理を行っている。

```js
Candidate.findAll({
  where: { scheduleId: schedule.scheduleId },
  order: [['"candidateId"', 'ASC']]
}).then((candidates) => {
  res.render('edit', {
    user: req.user,
    schedule: schedule,
    candidates: candidates
  });
});
```

以上の実装で候補を取得し、テンプレート edit を描画している。  
なお、作成順に並ぶように candidateId の昇順で並び替える。

```js
} else {
  const err = new Error('指定された予定がない、または、予定する権限がありません');
  err.status = 404;
  next(err);
```

この実装は、予定が自分が作ったものでなかったり、そもそも存在しなかったときに使われる。
`404 Not Found` のステータスを返すように実装しています。

```js
function isMine(req, schedule) {
  return schedule && parseInt(schedule.createdBy) === parseInt(req.user.id);
}
```

この関数は、リクエストと予定のオブジェクトを受け取り、その予定が自分のものであるかの真偽値を返す関数。

では実際に動かしてみよう。サーバーを再起動して、自分が作った予定の表示画面にアクセスしたら、末尾に `/edit` をつけてアクセスしてみよう。ここで、予定名、メモ、既存の候補日程、これらが表示されていれば、とりあえずフォームに最初に表示させる情報としては問題ない。

### 予定編集を反映させる実装

では、`/schedules/:scheduleId?edit=1` に POST メソッドで送られる情報を使って、データベースを更新してみよう。実装の流れとしては、

1. 予定 ID で指定された予定を取得
1. 予定名とメモを更新
1. 新しく追加された候補をデータベース上に作成
1. 予定表示ページにリダイレクト

という実装になる。

また、予定作成と同じ実装になる部分は、いくつか関数に切り出してしまおう。  
`routes/schedules.js` を以下のように編集する。

<details close>
<summary>routes/schedules.js</summary>

```diff
     createdBy: req.user.id,
     updatedAt: updatedAt
   }).then((schedule) => {
-    const candidateNames = req.body.candidates.trim().split('\n').map((s) => s.trim()).filter((s) => s !== "");
-    const candidates = candidateNames.map((c) => { return {
-      candidateName: c,
-      scheduleId: schedule.scheduleId
-    };});
-    Candidate.bulkCreate(candidates).then(() => {
-      res.redirect('/schedules/' + schedule.scheduleId);
-    });
+    createCandidatesAndRedirect(parseCandidateNames(req), scheduleId, res);
   });
 });
```

`createCandidatesAndRedirect` という関数で、候補の作成とリダイレクトを行うようにまとめた。  
この処理は、編集でも全く同じとなるためである。

```diff
    return schedule && parseInt(schedule.createdBy) === parseInt(req.user.id);
 }

+router.post('/:scheduleId', authenticationEnsurer, (req, res, next) => {
+  Schedule.findOne({
+    where: {
+      scheduleId: req.params.scheduleId
+    }
+  }).then((schedule) => {
+    if (isMine(req, schedule)) {
+      if (parseInt(req.query.edit) === 1) {
+        const updatedAt = new Date();
+        schedule.update({
+          scheduleId: schedule.scheduleId,
+          scheduleName: req.body.scheduleName.slice(0, 255) || '（名称未設定）',
+          memo: req.body.memo,
+          createdBy: req.user.id,
+          updatedAt: updatedAt
+        }).then((schedule) => {
+          // 追加されているかチェック
+          const candidateNames = parseCandidateNames(req);
+          if (candidateNames) {
+            createCandidatesAndRedirect(candidateNames, schedule.scheduleId, res);
+          } else {
+            res.redirect('/schedules/' + schedule.scheduleId);
+          }
+        });
+      } else {
+        const err = new Error('不正なリクエストです');
+        err.status = 400;
+        next(err);
+      }
+    } else {
+      const err = new Error('指定された予定がない、または、編集する権限がありません');
+      err.status = 404;
+      next(err);
+    }
+  });
+});
+
+function createCandidatesAndRedirect(candidateNames, scheduleId, res) {
+  const candidates = candidateNames.map((c) => {
+    return {
+      candidateName: c,
+      scheduleId: scheduleId
+    };
+  });
+  Candidate.bulkCreate(candidates).then(() => {
+    res.redirect('/schedules/' + scheduleId);
+  });
+}
+
+function parseCandidateNames(req) {
+  return req.body.candidates.trim().split('\n').map((s) => s.trim()).filter((s) => s !== "");
+}
+
 module.exports = router;
```

以上が具体的な予定更新の処理となる。

</details>

**解説**

```js
  Schedule.findOne({
    where: {
      scheduleId: req.params.scheduleId
    }
  }).then((schedule) => {
```

まず予定 ID で予定を取得している。

```js
if (isMine(req, schedule)) {
  if (parseInt(req.query.edit) === 1) {
    const updatedAt = new Date();
  schedule.update({
    scheduleId: schedule.scheduleId,
    scheduleName: req.body.scheduleName.slice(0, 255) || '（名称未設定）',
    memo: req.body.memo,
    createdBy: req.user.id,
    updatedAt: updatedAt
  }).then((schedule) => {
```

リクエストの送信者が作成者であるかをチェックし、 `edit=1` のクエリがあるときのみ更新を行う。 更新は予定名、メモ、作成者、更新日時について。  
なおここで実行されている、 `schedule` の `update` 関数は SQL における UPDATE 文に対応しており、予定の更新を行うもの。

```js
// 追加されているかチェック
const candidateNames = parseCandidateNames(req);
if (candidateNames) {
  createCandidatesAndRedirect(candidateNames, schedule.scheduleId, res);
} else {
  res.redirect('/schedules/' + schedule.scheduleId);
}
```

リクエストから候補日程の配列をパースする関数 `parseCandidateNames` を呼び出す。  
パースとは、文字列を文法にそって正しく分解して解釈すること。

ここでは、`parseCandidateNames` 関数の中でリクエスト内に含まれる改行で区切られた候補日程を分割し、 `trim` 関数で前後の空白を除去するなどして、候補日程の配列が得られるようにしている。

そして、その追加候補があるかどうかによって、 `createCandidatesAndRedirect` 関数を呼んで、 候補を追加してリダイレクトするか、そのままリダイレクトするかを if 文で分岐している。

```js
} else {
  const err = new Error('不正なリクエストです');
  err.status = 400;
  next(err);
}
```

この実装は、`edit=1` 以外のクエリが渡された際に 400 Bad Request のステータスコードを返す処理となる。

```js
} else {
  const err = new Error('指定された予定がない、または、編集する権限がありません');
  err.status = 404;
  next(err);
```
以上の実装で、予定が見つからない場合や自分自身の予定ではない場合に、 404 Not Found のステータスコードを返すように実装している。

```js
function createCandidatesAndRedirect(candidateNames, scheduleId, res) {
  const candidates = candidateNames.map((c) => { 
    return {
      candidateName: c,
      scheduleId: scheduleId
    };
  });
  Candidate.bulkCreate(candidates).then(() => {
    res.redirect('/schedules/' + scheduleId);
  });
}
```

`createCandidatesAndRedirect` 関数は、すでに予定作成にあった実装の切り出しを行った関数である。
候補日程の配列、予定 ID、レスポンスオブジェクトを受け取り、 候補の作成とリダイレクトを行っている。

```js
function parseCandidateNames(req) {
  return req.body.candidates.trim().split('\n').map((s) => s.trim()).filter((s) => s !== "");
}
```

これもすでに存在したリクエストから予定名の配列をパースする処理を、 `parseCandidateNames` という関数名で切り出したものである。

関数は切り出すことによって、他の場所で再利用することができるようになる。

あとは、編集ページヘのリンクが無くて不便なので、`views/schedule.pug` も以下のように編集して、リンクを作成してしまおう。

```diff
   h4 #{schedule.scheduleName}
   p(style="white-space:pre;") #{schedule.memo}
   p 作成者: #{schedule.user.username}
+  - var isMine = parseInt(user.id) === schedule.user.userId
+  if isMine
+    div
+      a(href=`/schedules/${schedule.scheduleId}/edit`) この予定を編集する
   h3 出欠表
   table
     tr
```

ここまで完了したらサーバーを再起動して `http://localhost:8000/` にアクセスしてログインし、予定表示ページを表示して編集ページへのリンクから編集を行ってみよう。

- 予定名が編集できる
- メモが編集できる
- 新しい予定の候補が追加できる

以上 3 つを試してみよう。問題なく編集できれば編集機能は実装できたということである。

### 予定が編集できることのテスト
「予定が更新でき、候補が追加できる」というテストを追加する。  
`test/test.js` を以下のように実装する。

<details close>
<summary>test/test.js</summary>

```diff
   });
 });
 
+describe('/schedules/:scheduleId?edit=1', () => {
+  before(() => {
+    passportStub.install(app);
+    passportStub.login({ id: 0, username: 'testuser' });
+  });
+
+  after(() => {
+    passportStub.logout();
+    passportStub.uninstall(app);
+  });
+
+  it('予定が更新でき、候補が追加できる', (done) => {
+    User.upsert({ userId: 0, username: 'testuser' }).then(() => {
+      request(app)
+        .post('/schedules')
+        .send({ scheduleName: 'テスト更新予定1', memo: 'テスト更新メモ1', candidates: 'テスト更新候補1' })
+        .end((err, res) => {
+          const createdSchedulePath = res.headers.location;
+          const scheduleId = createdSchedulePath.split('/schedules/')[1];
+          // 更新がされることをテスト
+          request(app)
+            .post(`/schedules/${scheduleId}?edit=1`)
+            .send({ scheduleName: 'テスト更新予定2', memo: 'テスト更新メモ2', candidates: 'テスト更新候補2' })
+            .end((err, res) => {
+              Schedule.findByPk(scheduleId).then((s) => {
+                assert.equal(s.scheduleName, 'テスト更新予定2');
+                assert.equal(s.memo, 'テスト更新メモ2');
+              });
+              Candidate.findAll({
+                where: { scheduleId: scheduleId },
+                order: [['"candidateId"', 'ASC']]
+              }).then((candidates) => {
+                assert.equal(candidates.length, 2);
+                assert.equal(candidates[0].candidateName, 'テスト更新候補1');
+                assert.equal(candidates[1].candidateName, 'テスト更新候補2');
+                deleteScheduleAggregate(scheduleId, done, err);
+              });
+            });
+        });
+    });
+  });
+});
+
 function deleteScheduleAggregate(scheduleId, done, err) {
   const promiseCommentDestroy = Comment.findAll({
     where: { scheduleId: scheduleId }
```
</details>

### 削除機能の実装
削除機能は編集ページにつけてしまう予定であった。削除ボタンを実装しよう。

`views/edit.pug` を以下のように実装して、削除ボタンを作る。

<details close>
<summary>views/edit.pug</summary>

```diff
       p 候補日程の追加 (改行して複数入力してください)
       textarea(name="candidates")
     div
       button(type="submit") 以上の内容で予定を編集する
+  h3 危険な変更
+  form(method="post", action=`/schedules/${schedule.scheduleId}?delete=1`)
+    button(type="submit") この予定を削除する
```
</details>

そのまま、このボタンを押した時に実行される削除処理の実装もしてしまおう。

`routes/schedules.js` を以下のように実装する。  
すでに `test/test.js` において `deleteScheduleAggregate` 関数という、予定に関連する情報を削除する関数を実装しているので、それをそのまま利用しよう。

<details close>
<summary>routes/schedules.js</summary>

```diff
       });
     });
+  } else if (parseInt(req.query.delete) === 1) {
+    deleteScheduleAggregate(req.params.scheduleId, () => {
+      res.redirect('/');
+    });
   } else {
    const err = new Error('不正なリクエストです');
```

`parseInt(req.query.delete) === 1` とすることで、delete=1 というクエリが渡された時の処理を記述している。  
`deleteScheduleAggregate` 関数をそのまま利用し、削除実行後、 `/` というルートパスにリダイレクトを行っている。

```diff
   }
 });
 
+function deleteScheduleAggregate(scheduleId, done, err) {
+  const promiseCommentDestroy = Comment.findAll({
+    where: { scheduleId: scheduleId }
+  }).then((comments) => {
+    return Promise.all(comments.map((c) => { return c.destroy(); }));
+  });
+
+  Availability.findAll({
+    where: { scheduleId: scheduleId }
+  }).then((availabilities) => {
+    const promises = availabilities.map((a) => { return a.destroy(); });
+    return Promise.all(promises);
+  }).then(() => {
+    return Candidate.findAll({
+      where: { scheduleId: scheduleId }
+    });
+  }).then((candidates) => {
+    const promises = candidates.map((c) => { return c.destroy(); });
+    promises.push(promiseCommentDestroy);
+    return Promise.all(promises);
+  }).then(() => {
+    return Schedule.findByPk(scheduleId).then((s) => { return s.destroy(); });
+  }).then(() => {
+    if (err) return done(err);
+    done();
+  });
+}
+
+router.deleteScheduleAggregate = deleteScheduleAggregate;
+
 function createCandidatesAndRedirect(candidateNames, scheduleId, res) {
     const candidates = candidateNames.map((c) => { return {
       candidateName: c,
```

以上は `test/test.js` の deleteScheduleAggregate をほとんどそのままコピーしたもの。
また、`router.deleteScheduleAggregate = deleteScheduleAggregate` の部分で、test/test.js 内でもこの関数を利用できるように この Router オブジェクトの公開関数としてある。

</details>

サーバーを再起動して、再度アクセス・ログインして、実際に予定が削除できるのかを試してみよう。

削除をしてみて、その結果、自分が作った予定の一覧が出てこなくなれば成功。

また、テストのほうで、先ほど Router オブジェクトに移動させた `deleteScheduleAggregate` 関数を利用するように修正する。

`test/test.js` を以下のように実装しよう。

<details close>
<summary>test/test.js</summary>

```diff
 const Availability = require('../models/availability');
 const Comment = require('../models/comment');
+const deleteScheduleAggregate = require('../routes/schedules').deleteScheduleAggregate;
 
 describe('/login', () => {
```

以上で関数をモジュールから呼び出し、

```diff
     });
   });
 });
-
-function deleteScheduleAggregate(scheduleId, done, err) {
-  const promiseCommentDestroy = Comment.findAll({
-    where: { scheduleId: scheduleId }
-  }).then((comments) => {
-    return Promise.all(comments.map((c) => { return c.destroy(); }));
-  });
-
-  Availability.findAll({
-    where: { scheduleId: scheduleId }
-  }).then((availabilities) => {
-    const promises = availabilities.map((a) => { return a.destroy(); });
-    return Promise.all(promises);
-  }).then(() => {
-    return Candidate.findAll({
-      where: { scheduleId: scheduleId }
-    });
-  }).then((candidates) => {
-    const promises = candidates.map((c) => { return c.destroy(); });
-    promises.push(promiseCommentDestroy);
-    return Promise.all(promises);
-  }).then(() => {
-    return Schedule.findByPk(scheduleId).then((s) => { return s.destroy(); });
-  }).then(() => {
-    if (err) return done(err);
-    done();
-  });
-}
```

以上のように、`deleteScheduleAggregate` 関数を除去する。

</details>

また、上記のテストでは「予定に関連するすべての情報が削除できる」というテストが書かれていない。  
コメント、出欠、候補、予定、のすべてを DB 上から削除できることをテストできるようにする。

<details close>
<summary>test/test.js</summary>

```js
describe('/schedules/:scheduleId?delete=1', () => {
  before(() => {
    passportStub.install(app);
    passportStub.login({ id: 0, username: 'testuser' });
  });

  after(() => {
    passportStub.logout();
    passportStub.uninstall(app);
  });

  it('予定に関連する全ての情報が削除できる', (done) => {
    User.upsert({ userId: 0, username: 'testuser' }).then(() => {
      request(app)
        .post('/schedules')
        .send({ scheduleName: 'テスト更新予定1', memo: 'テスト更新メモ1', candidates: 'テスト更新候補1' })
        .end((err, res) => {
          const createdSchedulePath = res.headers.location;
          const scheduleId = createdSchedulePath.split('/schedules/')[1];

          // 出欠作成
          const promiseAvailability = Candidate.findOne({
            where: { scheduleId: scheduleId }
          }).then((candidate) => {
            return new Promise((resolve) => {
              const userId = 0;
              request(app)
                .post(`/schedules/${scheduleId}/users/${userId}/candidates/${candidate.candidateId}`)
                .send({ availability: 2 }) // 出席に更新
                .end((err, res) => {
                  if (err) done(err);
                  resolve();
                });
            });
          });

          // コメント作成
          const promiseComment = new Promise((resolve) => {
            const userId = 0;
            request(app)
              .post(`/schedules/${scheduleId}/users/${userId}/comments`)
              .send({ comment: 'testcomment' })
              .expect('{"status":"OK","comment":"testcomment"}')
              .end((err, res) => {
                if (err) done(err);
                resolve();
              });
          });

          // 削除
          const promiseDeleted = Promise.all([promiseAvailability, promiseComment]).then(() => {
            return new Promise((resolve) => {
              request(app)
                .post(`/schedules/${scheduleId}?delete=1`)
                .end((err, res) => {
                  if (err) done(err);
                  resolve();
                });
            });
          });

          // テスト
          promiseDeleted.then(() => {
            const p1 = Comment.findAll({
              where: { scheduleId: scheduleId }
            }).then((comments) => {
              assert.equal(comments.length, 0);
            });
            const p2 = Availability.findAll({
              where: { scheduleId: scheduleId }
            }).then((availabilities) => {
              assert.equal(availabilities.length, 0);
            });
            const p3 = Candidate.findAll({
              where: { scheduleId: scheduleId }
            }).then((candidates) => {
              assert.equal(candidates.length, 0);
            });
            const p4 = Schedule.findByPk(scheduleId).then((schedule) => {
              assert.equal(!schedule, true);
            });
            Promise.all([p1, p2, p3, p4]).then(() => {
              if (err) return done(err);
              done();
            });
          });
        });
    });
  });
});
```

予定作成、出欠作成、コメント作成、削除、の様々なフェーズで Promise オブジェクトを作成している。  
なお `findAll` 関数は、戻ってきた Promise オブジェクトの then に渡す関数の引数として、 データモデルのオブジェクトの配列を渡す。 `findByPk` 関数は、then に渡す関数の引数として、 データモデルのオブジェクトを渡し、存在しない場合は null になる。

ちなみに、変数 `a` が `null` であることをテストしたい場合は `assert.equal(!a, true);` でテストできる（上記コードでは schedule 変数について行っている）。

</details>

`yarn test` して、すべてのテストが通り、削除の SQL が実行されていることも確認すること。

以上で、すべての機能要件を実装しきったことになる。

## § 22. デザインの改善
すでにすべての機能要件を実装することができた。  
今回はデザイン（見た目）を改善していく。

### JavaScript の this
今回、実装を始める前に `this` についての解説をはさむ。

基本的には、 **`this` が書かれた処理の関数が所属するオブジェクト** が `this` という変数の中に入っている。

そのため、関数が特に何にも所属していない場合には、グローバルオブジェクトという一番上位のスコープを表すオブジェクトが入っている。

さらに状況によっては、`this` の値は他の値に変わることがある。  
たとえば関数が new というキーワードで呼び出された際には、新しく作られた関数オブジェクト自体を表したり、`apply` という関数を使った際には、`this` には好きな値を入れることもできたりする。

しかし、基本的には、`this` は **`this` が書かれた関数が所属するオブジェクト** というふうに捉えればよい。

実際に動きを確かめてみよう。  
Chrome のデベロッパーツールの Console を開いて

```js
const a = {title: 'タイトル', print: function(){ console.log(this.title); }};
a.print();
```

以上の二行を実行すると、「タイトル」と表示される。つまり `this` は オブジェクト `a` を表していることがわかる。

次に、

```js
function printGlobal() { console.log(this); }
printGlobal();
```

と入力すると、`Window { external: Object, chrome: Object, document: document, ...} ` というように Window オブジェクトが Console に表示される。  
ブラウザの JavaScript においては、Window オブジェクトがグローバルオブジェクトになっているからである。

このように、`this` 変数は、呼び出される場所によって異なる挙動を示す。

*👀* **function 宣言による記法とアロー関数の違いもある。**

```js
const hoge = {
    print: () => { console.log(this); }
}

const moge = {
    print: function() {
        console.log(this);
    }
}
```

以上の二つを考える場合、`hoge.print()` の結果は Window オブジェクトだが、`moge.print()` は `{print: f}` というふうに、呼び出し元の moge オブジェクト自体を指す。

----

### Bootstrap のインストール

```bash
yarn add bootstrap@4.0.0
yarn add popper.js@1.14.0
```

以上のコマンドで Bootstrap と Boostrap が依存している依存モジュールのインストールを行う。

インストールできたら、組み込みを行う。  
まず、 `app/entry.js` を以下のように実装する。

<details close>
<summary>app/entry.js</summary>

```diff
 'use strict';
 import $ from 'jquery';
+const global = Function('return this;')();
+global.jQuery = $;
+import bootstrap from 'bootstrap';
 
 $('.availability-toggle-button').each((i, e) => {
```

</details>

この処理では、グローバルオブジェクトの jQuery というプロパティに jQuery を代入している。  
これを行わないと Bootstrap が jQuery を利用できないため。

```js
const global = Function('return this;')();
```

`Function` 関数は、引数で受け取った文字列をもとに関数を生成する。  
そして、ここでは作った関数をその場で呼び出し、グローバルオブジェクトを取得している。  
先ほど学んだ this を、グローバルオブジェクトを取得する手段として使っている。

モジュールを用いた JavaScript の開発では、グローバルオブジェクトを取得するのにこのような工夫をしてやる必要がある。

次に、`views/layout.pug` を以下のように編集して Bootstrap に対応させる。

<details close>
<summary>views/layout.pug</summary>

```diff
 doctype html
 html
   head
-    title= title
-    link(rel='stylesheet', href='/stylesheets/style.css')
+    title 予定調整くん
+    meta(charset="utf-8")
+    meta(name="viewport" content="width=device-width, initial-scale=1")
+    link(rel="stylesheet",
+    href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css",
+    integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm",
+    crossorigin="anonymous")
   body
-    block content
+    div.container
+      block content
     script(src="/javascripts/bundle.js")

```

</details>

過去 Bootstrap を利用したときと同じ設定で Bootstrap の CSS を読み込んでいる。  
また title もここでは「予定調整くん」で固定してしまっている。

なお、block content で継承元で呼び出されるてぬレートを `div.container` の中に含めるようにしている。  
これは Bootstrap のための設定のひとつとなる。

### NavBar の設置 layout.pug
すべてのページでログイン、ログアウトのバーを表示させよう。  
`views/layout.pug` を以下のように変更する。

<details close>
<summary>views/layout.png</summary>

```diff
    link(href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css", rel="stylesheet", integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm", crossorigin="anonymous")
   body
+    nav.navbar.navbar-light.bg-light
+      div.navbar-header
+        a(href="/").navbar-brand.nav-link 予定調整くん
+      ul.navbar-nav
+        if user
+          li.nav-item
+            a(href="/logout").nav-link #{user.username} をログアウト
+        else
+          li.nav-item
+            a(href="/login").nav-link ログイン
     div.container
       block content
     script(src="/javascripts/bundle.js")
```
</details>

[NavBar](https://getbootstrap.com/docs/4.0/components/navbar/) という部品をそのまま利用した。
また、左側に「予定調整くん」というラベルの書かれたトップに戻るリンクを配置し、ログアウトのリンクに、ログインしているユーザー名も含めてみた。

### トップページ index.pug
トップページには、[Jumbotron](https://getbootstrap.com/docs/4.0/components/jumbotron/) というタイトルと説明を表示させる部品を利用してみる。  
また、class を割り当てて表やボタンの見栄えをよくする。

<details close>
<summary>views/index.pug</summary>

```diff
 extends layout
 
 block content
-  h1= title
-  p Welcome to #{title}
-  div
-    if user
-      div
-        a(href="/logout") #{user.username} をログアウト
-      div
-        a(href="/schedules/new") 予定を作る
-      - var hasSchedule = schedules.length > 0
-      if hasSchedule
-        h3 あなたの作った予定一覧
-        table.table
+  div.jumbotron.my-3
+    h1.display-4 予定調整くん
+    p.lead 予定調整くんは、GitHubで認証でき、予定を作って出欠が取れるサービスです
+  if user
+    div
+      a(href="/schedules/new").btn.btn-info 予定を作る
+    - var hasSchedule = schedules.length > 0
+    if hasSchedule
+      h3.my-3 あなたの作った予定一覧
+      table.table
+        tr
+          th 予定名
+          th 更新日時
+        each schedule in schedules
           tr
-            th 予定名
-            th 更新日時
-          each schedule in schedules
-            tr
-              td
-                a(href=`/schedules/${schedule.scheduleId}`) #{schedule.scheduleName}
-              td #{schedule.updatedAt}
-    else
-      div
-        a(href="/login") ログイン
+          td
+            a(href=`/schedules/${schedule.scheduleId}`) #{schedule.scheduleName}
+          td #{schedule.updatedAt}
```
</details>

### ログインページ login.pug
ログインページでは、ログイン中のラベルを消して、ボタンのスタイルを適用させる。  
`views/login.pug` を以下のように編集する。

<details close>
<summary>views/login.pug</summary>

```diff
 extends layout
 
 block content
-  a(href="/auth/github") GitHubでログイン
-  if user
-    p 現在 #{user.username} でログイン中
+  a(href="/auth/github").btn.btn-info.my-3 GitHubでログイン
```
</details>

ボタンにクラスを割り当てたため、`test.js` を以下のように編集する。

<details close>
<summary>test.js</summary>

```diff
   it('ログインのためのリンクが含まれる', (done) => {
     request(app)
       .get('/login')
       .expect('Content-Type', 'text/html; charset=utf-8')
-      .expect(/<a href="\/auth\/github"/)
+      .expect(/<a class="btn btn-info my-3" href="\/auth\/github"/)
       .expect(200, done);
  });
```
</details>

### 予定作成ページ new.pug
Bootstrap で [フォーム](http://getbootstrap.com/css/#forms) にデザインを適用する場合には、`label` 要素を用いたり、`label` 要素と `input` 要素を対応づける必要があった。

以下のように `views/new.pug` を変更する。

<details close>
<summary>views/new.pug</summary>

```diff
 block content
-  form(method="post", action="/schedules")
-    div
-      h5 予定名
-      input(type="text" name="scheduleName")
-    div
-      h5 メモ
-      textarea(name="memo")
-    div
-      h5 候補日程 (改行して複数入力してください)
-      textarea(name="candidates")
-  button(type="submit") 予定をつくる
+  form(method="post", action="/schedules").my-3
+    div.form-group
+      label(for="scheduleName") 予定名
+      input(type="text" name="scheduleName")#scheduleName.form-control
+    div.form-group
+      label(for="memo") メモ
+      textarea(name="memo")#memo.form-control
+    div.form-group
+      label(for="candidates") 候補日程 (改行して複数入力してください)
+      textarea(name="candidates" rows="4")#candidates.form-control
+    button(type="submit").btn.btn-info 予定をつくる
```

</details>

### 予定編集ページ
編集ページでも似たように Bootstrap のフォームのデザインの適用を行う。  
`views/edit.pug` を以下のように変更する。

<details close>
<summary>views/edit.pug</summary>

```diff
 block content
   h3.my-3 予定の編集
   form(method="post", action=`/schedules/${schedule.scheduleId}?edit=1`)
-    div
-      h5 予定名
-      input(type="text" name="scheduleName" value=schedule.scheduleName)
-    div
-      h5 メモ
-      textarea(name="memo") #{schedule.memo}
-    div
-      label 既存の候補日程
-      ul
+    div.form-group
+      label(for="scheduleName") 予定名
+      input(type="text" name="scheduleName" value=schedule.scheduleName)#scheduleName.form-control
+    div.form-group
+      label(for="memo") メモ
+      textarea(name="memo")#memo.form-control #{schedule.memo}
+    div.form-group
+      label 既存の日程候補
+      ul.list-group
         each candidate in candidates
-          li #{candidate.candidateName}
-      p 候補日程の追加 (改行して複数入力してください)
-      textarea(name="candidates")
+          li.list-group-item #{candidate.candidateName}
+      label(for="candidates").my-2 候補日程の追加 (改行して複数入力してください)
+      textarea(name="candidates")#candidates.form-control
     div
-      button(type="submit") 以上の内容で予定を編集する
-  h3 危険な変更
+      button(type="submit").btn.btn-info 以上の内容で予定を編集する
+  h3.my-3 危険な変更
   form(method="post", action="/schedules/#{schedule.scheduleId}?delete=1")
-    button(type="submit") この予定を削除する
+    button(type="submit").btn.btn-danger この予定を削除する
```

予定を消す際のボタンは、`btn-danger` という class を適用することで、注意を喚起している。

</details>

### 予定表示ページ schedule.pug
最後にこの「予定調整くん」の中心的な機能である、予定表示ページの出欠表のデザインの見た目を改善sる。

`views/schedule.pug` を以下のように修正していく。

<details close>
<summary>views/schedule.pug</summary>

```diff
 extends layout
 
 block content
-  h4 #{schedule.scheduleName}
-  p(style="white-space:pre;") #{schedule.memo}
-  p 作成者: #{schedule.user.username}
+  div.card.my-3
+    div.card-header
+      h4 #{schedule.scheduleName}
+    div.card-body
+      p(style="white-space:pre;") #{schedule.memo}
+    div.card-footer
+      p 作成者: #{schedule.user.username}
   - var isMine = parseInt(user.id) === schedule.user.userId
   if isMine
     div
-      a(href=`/schedules/${schedule.scheduleId}/edit`) この予定を編集する
-  h3 出欠表
-  table
+      a(href=`/schedules/${schedule.scheduleId}/edit`).btn.btn-info この予定を編集する
+  h3.my-3 出欠表
+  table.table.table-bordered
     tr
       th 予定
       each user in users
@@ -20,24 +24,29 @@ block content
         each user in users
           - var availability = availabilityMapMap.get(user.userId).get(candidate.candidateId)
           - var availabilityLabels = ['欠', '？', '出'];
+          - var buttonStyles = ['btn-danger', 'btn-secondary', 'btn-success'];
           td
             if user.isSelf
-              button(
+              button(class=`availability-toggle-button btn-lg ${buttonStyles[availability]}`
                 data-schedule-id=schedule.scheduleId
                 data-user-id=user.userId
                 data-candidate-id=candidate.candidateId
-                data-availability=availability).availability-toggle-button #{availabilityLabels[availability]}
+                data-availability=availability) #{availabilityLabels[availability]}
             else
-              p #{availabilityLabels[availability]}
+              h3 #{availabilityLabels[availability]}
     tr
       th コメント
       each user in users
         if user.isSelf
           td
-            p#self-comment #{commentMap.get(user.userId)}
+            p
+              small#self-comment #{commentMap.get(user.userId)}
             button(
               data-schedule-id=schedule.scheduleId
-              data-user-id=user.userId)#self-comment-button 編集
+              data-user-id=user.userId)#self-comment-button.btn-sm.btn-info 編集
         else
           td
-            p #{commentMap.get(user.userId)}
+            p
+              small #{commentMap.get(user.userId)}
```

</details>

上記では、予定の内容の表示に [Card](https://getbootstrap.com/docs/4.0/components/card/) という部品を使った。  
また、['欠', '？', '出'] だけのラベルではなく、ボタンの色も変えるようにしている。

なお、コメントは小さいボタンのデザインを表示したほか、コメント自体も小さく表示されるようにした。

再読み込みをして確認してみよう。  
ただ、出欠ボタンを実際にクリックすると気づくと思いますが、 クリックで変わったあとの出欠に応じて、ボタンの色も追従して変更させなくてはいけない。

`app/entry.js` を以下のように編集しよう。

```diff
         button.data('availability', data.availability);
         const availabilityLabels = ['欠', '？', '出'];
         button.text(availabilityLabels[data.availability]);
+
+        const buttonStyles = ['btn-danger', 'btn-secondary', 'btn-success'];
+        button.removeClass('btn-danger btn-secondary btn-success');
+        button.addClass(buttonStyles[data.availability]);
       });
   });
 });
```

以上の実装では、ボタンとセルのスタイルをいったんすべて取り除き、新たに追加するという実装にしている。

### 時刻の表示
トップページにログインした際の「あなたのつくった予定一覧」での日付が、現在は UTC での表示になっている。  
これを JST 日本標準時に変換した後、 `YYYY/MM/DD HH:mm` 形式で、「2020/05/10 12:00」のように表示されるように修正しよう。

以前使った [Moment Timezone](https://momentjs.com/timezone/) というモジュールを使う。

```js
yarn add moment-timezone@0.5.0
```

<details close>
<summary>routes/index.js</summary>

```diff
 const express = require('express');
 const router = express.Router();
 const Schedule = require('../models/schedule');
+const moment = require('moment-timezone');
 
 /* GET home page. */

(中略)

       },
       order: [['"updatedAt"', 'DESC']]
     }).then((schedules) => {
+      schedules.forEach((schedule) => {
+        schedule.formattedUpdatedAt = moment(schedule.updatedAt).tz('Asia/Tokyo').format('YYYY/MM/DD HH:mm');
+      });
       res.render('index', {
         title: title,
         user: req.user,
```

上記では、YYYY/MM/DD HH:mm 形式にフォーマットされた日付を `formattedUpdatedAt` というプロパティ名で利用できるようにしている。  
あとはこの `formattedUpdatedAt` をテンプレートで使用すればよい。

</details>

<details close>
<summary></summary>

```diff
           tr
             td
               a(href=`/schedules/${schedule.scheduleId}`) #{schedule.scheduleName}
-            td #{schedule.updatedAt}
+            td #{schedule.formattedUpdatedAt}
```

</details>

以上完了したら、一度サーバーを止め、

```bash
node_modules/.bin/webpack
PORT=8000 yarn start
```

として JavaScript を出力してからサーバーを起動する。以上でデザインの改善は終了。

使わなくなった css ファイルは削除してしまおう。

```bash
rm public/stylesheets/style.css
```

## § 23. セキュリティ対策と公開
### 行うべきセキュリティ対策
今回はどのようなセキュリティ対策を行うべきなのか。過去紹介した脆弱性一覧を見ながら対策を考えよう。

|脆弱性|影響|内容|
|--|--|--|
|OS コマンド・インジェクション|大|任意の OS のコマンドを実行できてしまう|
|SQL インジェクション|大|任意の SQL コマンドを実行できてしまう|
|ディレクトリ・トラバーサル|大|任意のファイルを閲覧、操作できてしまう|
|セッションハイジャック|大|利用者のセッションが乗っ取られてしまう|
|クロスサイト・スクリプティング（XSS）|中|スクリプトにより Web サイトの改竄ができてしまう|
|クロスサイト・リクエストフォージェリ（CSRF）|中|利用者の意図しない操作がされてしまう|
|HTTP ヘッダインジェクション|中|偽ページの表示などができてしまう|
|クリックジャッキング|小|利用者の意図しないクリックをしてしまう|

今回は GitHub 認証を利用しているため、ユーザーのアカウント管理は考える必要はない。

OS コマンドインジェクションは、OS コマンドを実行している部分がないため大丈夫であろう。

SQL インジェクションも、今回は　SQL をプログラム内の文字列として直接実行することはなく、必ず sequelize を利用して DB 操作を行っているため関係ない。

ディレクトリ・トラバーサルは、Express の Router オブジェクトを介してルーティングをしているため問題ない。

セッション・ハイジャックに関して、今回は Express の session の仕組みをそのまま利用して実装している。したがって、これらの脆弱性にはじゅうぶん気をつける必要があるが、いま対応が必要なものはない。

クロスサイト・スクリプティングだが、今回は pug テンプレートを使用して HTML を描画している。そして、エスケープを無視して表示を行っている部分はないはずであるため OK。

クロスサイト・リクエストフォージェリは、現時点の実装では脆弱性として存在している。

- 予定作成フォーム
- 予定編集フォーム
- 予定削除フォーム

これらの実装で使用しているフォームで、突かれてしまうつくりになっているため、これらは **対策しなくてはいけない**。

HTTP  ヘッダインジェクションは、Cookie の内容を直接使用したり、自身で URL 全体を受け付けるリダイレクト機能を用意していないので大丈夫だろう。

クリックジャッキングは、実はすでにここまでの実装で対策がなされている。Express の標準の設定で、レスポンスヘッダに `X-Frame-Options:SAMEORIGIN` という内容が含まれている。  
このヘッダがあることで HTML の iframe 要素の機能により外部シアとからアクセスできないよう対策されている。

なお、レスポンスヘッダの `X-Frame-Options` は、HTML における ”フレーム” という HTML 内に HTML を読み込む機能においてどのようなサイトからの読み込みを許可するかという設定である。 

このヘッダに対して、クリックジャッキングを防ぐために、同じサイト内からの読み込みだけしか許可しない `SAMEORIGIN` を設定しているのである。

以上を踏まえると、今回は CSRF 脆弱性対策だけを行う必要がありそうとわかる。

### CSRF 脆弱性対策
Express では簡単に CSRF 対策を行うことができる。  
今回は csurf というミドルウェアをインストールして対策する。

```bash
yarn add csurf@1.8.3
```

あとは csurf の README.md に記載されている内容を元に、`routes/schedules.js` を以下のように実装する。

<details close>
<summary>routes/schedules.js</summary>

```diff
 const User = require('../models/user');
 const Availability = require('../models/availability');
 const Comment = require('../models/comment');
+const csrf = require('csurf');
+const csrfProtection = csrf({ cookie: true });

-router.get('/new', authenticationEnsurer, (req, res, next) => {
-  res.render('new', { user: req.user });
+router.get('/new', authenticationEnsurer, csrfProtection, (req, res, next) => {
+  res.render('new', { user: req.user, csrfToken: req.csrfToken() });
 });

-router.post('/', authenticationEnsurer, (req, res, next) => {
+router.post('/', authenticationEnsurer, csrfProtection, (req, res, next) => {
   const scheduleId = uuid.v4();
   const updatedAt = new Date();
   Schedule.create({
```
</details>

ここでは、`csurf` モジュールを読み込み、CSRF 対策として利用できるトークンを `csrfToken` というプロパティ名でテンプレートに渡している。

また、POST リクエスト時には、`csrfProtection` 関数を Router オブジェクトに渡すことで、チェックが行われるようにしている。

<details close>
<summary>routes/schedules.js</summary>

```diff
});

-router.get('/:scheduleId/edit', authenticationEnsurer, (req, res, next) => {
+router.get('/:scheduleId/edit', authenticationEnsurer, csrfProtection, (req, res, next) => {
   Schedule.findOne({
     where: {
       scheduleId: req.params.scheduleId
     }
   }).then((schedule) => {
     if (isMine(req, schedule)) { // 作成者のみが編集フォームを開ける
       Candidate.findAll({
         where: { scheduleId: schedule.scheduleId },
         order: [['"candidateId"', 'ASC']]
       }).then((candidates) => {
         res.render('edit', {
           user: req.user,
           schedule: schedule,
-          candidates: candidates
+          candidates: candidates,
+          csrfToken: req.csrfToken()
         });
       });
     } else {
```

```diff
   return schedule && parseInt(schedule.createdBy) === parseInt(req.user.id);
 }

-router.post('/:scheduleId', authenticationEnsurer, (req, res, next) => {
+router.post('/:scheduleId', authenticationEnsurer, csrfProtection, (req, res, next) => {
   Schedule.findOne({
     where: {
       scheduleId: req.params.scheduleId
```

</details>

以上の部分にも同様の実装をしている。  
編集と削除時のフォームの表示のときには、CSRF 対策のトークンを渡し、POST リクエスト時にチェックするようにしている。

今の状態では、CSRF 対策のトークンが生成され、チェックされるようになっている。  
しかしテンプレート側で、一緒にトークンを POST で送るようにはなっていないので、チェックで弾かれて、投稿ができないようになっているはずである。

これを確認してみよう。実装ができたら `PORT=8000 yarn start` としてサーバーを起動して再度アクセス・ログインのうえ、予定の作成・編集・削除いずれもがまったくできなくなっていることを確認しよう。

では今度は、作成フォームと編集フォームと削除フォームのそれぞれにトークンを埋め込んで、ちゃんと POST リクエストが成功するようにしてみよう。

`views/new.pug` を以下のように編集する。

<details close>
<summary>views/new.pug</summary>

```diff
 block content
   form(method="post", action="/schedules").my-3
+    input(type="hidden" name="_csrf" value!=csrfToken)
     div.form-group
       label(for="scheduleName") 予定名
       input(type="text" name="scheduleName")#scheduleName.form-control
```

</details>

`views/edit.pug` にも以下の行を追加する。

<details close>
<summary>views/edit.pug</summary>

```diff
 block content
   h3.my-3 予定の編集
   form(method="post", action=`/schedules/${schedule.scheduleId}?edit=1`)
+    input(type="hidden" name="_csrf" value!=csrfToken)
     div.form-group
       label(for="scheduleName") 予定名
       input(type="text" name="scheduleName" value=schedule.scheduleName)#scheduleName.form-control
```

```diff
       button(type="submit").btn.btn-info 以上の内容で予定を編集する
   h3.my-3 危険な変更
   form(method="post", action=`/schedules/${schedule.scheduleId}?delete=1`)
+    input(type="hidden" name="_csrf" value!=csrfToken)
     button(type="submit").btn.btn-danger この予定を削除する
```

それぞれ `_csrf` というキーで一緒に POST されるようにしている。  
これが csurf モジュールの使い方である。

</details>

実装ができたら再度アクセスして、予定の作成・編集・削除が問題なくできることを確認しよう。  
問題なければ　CSRF 脆弱性の対策は完了である。

## ログインできなかった際のリダイレクト機能
現在の実装で、`http://localhost:8000/schedules/29e8ff4e-6a44-4274-807f-e2999d9c8b72` このような出欠表の URL にアクセスして、GitHub 認証がされていなかった場合を考える。  
未ログインなので `/login` のログインページにリダイレクトされる。  
そしてその後ログインすると `http://localhost:8000/` に飛ばされてしまい、元々アクセスしたかった URL 情報は失われてしまう。

これを修正し、ログインの直後にもともとの URL に戻れるように実装しよう。

実装方法としては、ログインページのクエリにリダイレクトしたいパスを含められるようにし、いったん Cookie にログイン後の戻り先を覚えさせて対応する。

まず、`routes/authentication-ensurer.js` を以下のように実装する。

<details close>
<summary>route/authentication-ensurer.js</summary>

```diff
 function ensure(req, res, next) {
   if (req.isAuthenticated()) { return next(); }
-  res.redirect('/login');
+  res.redirect('/login?from=' + req.originalUrl);
 }
```

認証がうまくいかなかった際に、どこにアクセスしようとしたかを、`/login` のクエリに含めた形でリダイレクトしている。

</detais>

次に、`routes/login.js` を以下のように実装する。

<details close>
<summary>routes/login.js</summary>

```diff
 const router = express.Router();

 router.get('/', (req, res, next) => {
-  res.render('login', { user: req.user });
+  const from = req.query.from;
+  if (from) {
+    res.cookie('loginFrom', from, { expires: new Date(Date.now() + 600000)});
+  }
+  res.render('login');
 });
```

ログインページ表示時に、どこからログインしようとしたかを、保存期間を 10 分として Cookie に保存してから、ログインページを描画するようにした。
</details>

そして最後、`app.js` を以下のように実装する。

<details close>
<summary>app.js</summary>

```diff
 app.get('/auth/github/callback',
   passport.authenticate('github', { failureRedirect: '/login' }),
   function (req, res) {
-    res.redirect('/');
+    var loginFrom = req.cookies.loginFrom;
+    // オープンリダイレクタ脆弱性対策
+    if (loginFrom &&
+      !loginFrom.includes('http://') &&
+      !loginFrom.includes('https://')) {
+      res.clearCookie('loginFrom');
+      res.redirect(loginFrom);
+    } else {
+      res.redirect('/');
+    }
   });
```
</details>

認証が終わったらこれまでは `/` のルーとパスにリダイレクトしていたが、それを Cookie で保存された URL のパスにリダイレクトするようにする。

なお Cookie の値はユーザーが自由に編集できるのだった。自由に入力ができる部分には脆弱性が生まれがちで、実際にこのような状況で発生する **オープンリダイレクタ脆弱性** というものがある。

そこでここではオープンリダイレクタ脆弱性に対処すべく、`http://` か `https://` ではじまる URL でしかリダイレクトを実施しないように対策している。（教材原文ママ。実際は、`loginFrom` は `%2Fschedules%2Fd9296911-8ac4-4ab9-ad68-8c93038ec7c8` のような値がセットされるので、`http(s)://` を含んでいたら弾く、という作りだと思う）

外部サイトに勝手にリダイレクトさせ、信用あるページであるように見せるような脆弱性が、これで未然に防止される。

以上でサーバを再起動し、 `http://localhost:8000/` にアクセス、ログインしてみよう。

何かの予定表示の URL をメモしてから、ログアウトし、 メモした URL に直接アクセスしてみよう。

ログアウト中なのでログイン画面になる。 そこからログインすると、ログインの後で、予定表示のページが表示されるはず。

以上で問題なければ、予定表示のページの URL を誰かに渡して、 出欠を取る際の利便性が大きくあがる。

これでもう「予定調整くん」は問題なく利用できそうである。

### Heroku への公開
まずは `app.json` を作成する。

<details close>
<summary>app.json</summary>

```json
{
  "name": "schedule-arranger",
  "description": "予定調整くん",
  "logo": "",
  "keywords": ["node"],
  "image": "heroku/nodejs"
}
```
</details>

次に `package.json` を編集し、Heroku で使う Node.js のバージョンを記述する。

<details close>
<summary>package.json</summary>

```diff
     "supertest": "^3.1.0",
     "webpack": "^4.26.1",
     "webpack-cli": "^3.1.2"
-  }
+  },
+  "engines": {
+    "node": "~10"
+   }
 }
```

</details>

`models/sequelize-loader.js` を以下のように実装して、 Heroku のデータベースを使えるようにする。

<details close>
<summary>models/sequelize-loader.js</summary>

```diff
 'use strict';
 const Sequelize = require('sequelize');
 const sequelize = new Sequelize(
-  'postgres://postgres:postgres@localhost/schedule_arranger',
+  process.env.DATABASE_URL || 'postgres://postgres:postgres@localhost/schedule_arranger',
   {
     operatorsAliases: false
   });

 module.exports = {
```
</details>

一旦この時点で Heroku にアプリケーションを作り、アップロードをしてしまおう。  
まずは Git のリポジトリにコミットしよう。

```bash
git add .
git commit -am "Heroku対応"
```

次に、Heroku コマンドでサーバーやデータベースの作成を行う。

```bash
heroku login -i
```

以上のコマンドの後にメールアドレスとパスワードを入力して Heroku にログイン。

```
Logged in as hoge@fuga.com
```

のように表示されればログイン成功。  
次は、`heroku create` と入力すれば、 Heroku のサーバーが作成され、作成されたサーバーの URL がターミナルに表示される。

次は Heroku で利用する PostgreSQL の DB を作成する。

```bash
heroku addons:create heroku-postgresql:hobby-dev
```

以上のコマンドで開発用の簡易的な　PostgreSQL のデータベースを作成できる。

そうしたら git push コマンドで Heroku にデプロイしてみよう。

```bash
# master-2019 はローカルのブランチ名
git push heroku master-2019:master
```

デプロイが終わったら、先ほど heroku create して作成したサーバーの URL にアクセスして、トップページが表示できれば問題ない。

そして次は、GitHub 認証のために、本番の環境の GitHub のアプリケーh村として登録する。  
GitHub のアプリケーションの登録は、コールバックするための URL やサイトの説明の都合上、開発用のものと別なものを利用する必要がある。

GitHub にログイン後、`https://github.com/settings/applications/new` にアクセスし、

- Application name を、予定調整くん
- Homepage URL を、`https://xxx-xxxxxxx-XXXXXX.herokuapp.com/`（先ほど Heroku で作成した自分のアプリケーションのもの）
- Application description を、GitHub認証を利用して予定を調整してくれるアプリケーション
- Authorization callback URL を、 `https://xxx-xxxxxxx-XXXXXX.herokuapp.com/auth/github/callback`

以上の通りに設定し、アプリケーション登録をする。登録後の画面の Client ID と Client Secret は以下で使用する。

次に、 本番環境の URL と GitHub のアプリケーションの Client ID と Client Secret を Heroku の環境変数として登録していく。

```bash
heroku config:set HEROKU_URL='https://xxx-xxxxxxx-XXXXXX.herokuapp.com/'
heroku config:set GITHUB_CLIENT_ID='000000000000000000'
heroku config:set GITHUB_CLIENT_SECRET='000000000000000000000000000000000'
```

以上のようにコンソールに入力して登録する。
そして、 `app.js` を編集し、この環境変数を利用できるように修正する。

<details close>
<summary>app.js</summary>

```diff
 });

 var GitHubStrategy = require('passport-github2').Strategy;
-var GITHUB_CLIENT_ID = '2f831cb3d4aac02393aa';
-var GITHUB_CLIENT_SECRET = '9fbc340ac0175123695d2dedfbdf5a78df3b8067';
+var GITHUB_CLIENT_ID = process.env.GITHUB_CLIENT_ID || '2f831cb3d4aac02393aa';
+var GITHUB_CLIENT_SECRET = process.env.GITHUB_CLIENT_SECRET || '9fbc340ac0175123695d2dedfbdf5a78df3b8067';

 passport.serializeUser(function (user, done) {
   done(null, user);
```

</details>

以上で、 Client ID と Client Secret として環境変数で指定されたものが使われるようになった。

<details close>
<summary>app.js</summary>

```diff
 passport.use(new GitHubStrategy({
   clientID: GITHUB_CLIENT_ID,
   clientSecret: GITHUB_CLIENT_SECRET,
-  callbackURL: 'http://localhost:8000/auth/github/callback'
+  callbackURL: process.env.HEROKU_URL ? process.env.HEROKU_URL + 'auth/github/callback' : 'http://localhost:8000/auth/github/callback'
 },
```
</details>

以上が、コールバックして GitHub から情報を戻す URL となる。

では以上の内容を反映するため、再度 Heroku にデプロイを行おう。

```bash
git commit -am "Herokuの環境変数を利用できるように修正"
git push heroku master-2019:master
```

以上の git コマンドを使い、`remote: Verifying deploy... done.` と表示されたら終了です。
Heroku の URL に再度アクセスして、この「予定調整くん」の要件である、

- 予定を作れる
- 予定に候補が作れる
- 予定の候補に対して出欠を編集できる
- 予定に対してコメントが編集できる
- 予定を削除できる
- 予定を編集できる

これらがちゃんと動作するかを試してみよう。
この検証が終われば「予定調整くん」は完成。

これで長かった「予定調整くん」の開発はすべて終わり！
また、入門コースの開発もこれで全て終わり！！！