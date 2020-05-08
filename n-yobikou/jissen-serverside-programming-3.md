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
<details close>

**解説**

まず、`userId` が `0` で、 `username` が `testuser` のユーザーを DB 上に作成している。

その後、POST メソッドを使い予定と候補を作成している。

そこからリダイレクトされることを検証し、予定が表示されるページヘのアクセスが 200 のステータスコードであることを検証している。

なお、テストが終わった後に、テストで作成されたユーザー以外のデータを削除する処理が加えられている。
ここで使われている `findByPk` 関数は、 モデルに対応するデータを主キーによって 1 行だけ取得することができる関数。  
また、`Promise.all` 関数は、 配列で渡された全ての Promise が終了した際に結果を返す、Promise オブジェクトを作成する。  
なお、空配列が渡された場合も Promise オブジェクトが作成される。

