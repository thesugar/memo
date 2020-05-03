# 実践サーバーサイドプログラミング
[Source](https://www.nnn.ed.nico/courses/497/chapters/6891)

## §01. Web フレームワーク
### Web フレームワークとは
Web フレームワークは、正確には Web Application Framework（WAF）と呼ばれる。動的な Web サイトや　Web API を提供するたえのフレームワークで、Web 開発でよく行われる作業や実装を効率化するための様々な仕組みを備えている。

例えば、以下のような機能がある。

- ルーティング機能
- プロジェクトの雛形の作成機能
- セキュリティ対策機能
- ログ出力を補助する機能
- 認証認可機能
- フレームワーク自体を拡張する機能
- テンプレートエンジン

Node.js には数多くの Web フレームワークがあるが、ここでは Express という Web フレームワークを使っていく。

### Express
Express とは OpenJS 財団により保守されている MIT ライセンスの Web フレームワークである。Ruby の Web フレームワークである Sinatra に影響を受けて作られている。

この Express は特徴として

- 堅牢生の高いルーティング
- 高いパフォーマンス

に重点が置かれている。そのため、安全で高速な Web サービスを作るのが得意である。半面、機能はシンプルで、高機能で複雑なアプリケーションを作るためには工夫を要する。

主な機能として

- ルーティング機能
- プロジェクトの雛形の作成機能
- ログ出力を補助する機能
- フレームワーク自体を拡張する機能
- テンプレートエンジン

これらの機能が標準で用意されている。なお、この Express は、セキュリティに関して、[Security Policies and Procedures](https://github.com/expressjs/express/blob/master/Security.md) という文章を公開している。ここには、「セキュリティに関するバグがあったときにはメールを送ってほしいこと」「しかるべきプロセスで修正情報が公開されるということ」が書かれている。

### Express を利用して Web アプリケーションを作る
それでは、この Express を利用して Web アプリケーションを作ってみよう。

コンソールを起動したら、Express application generator をインストールして、アプリケーションを作成する。これはジェネレーターといい、プロジェクトの雛形を作るツールである。

```bash
yarn global add express-generator@4.16.0
cd ~/workspace
express --view=pug express-study
```

以上のコマンドでインストールと作成を実行すると

```
   create : express-study/
   create : express-study/public/
   create : express-study/public/javascripts/

    ... 略

   run the app:
     $ DEBUG=express-study:* npm start
```

上記のように表示され、Express を使ったアプリケーションに必要なファイルが一通り左k酢英される。
なお、`express --view=pug express-study` のコマンドの部分は `express-study` という名前のプロジェクトを作成し、テンプレートエンジンに `pug` を使用するコマンドである。

次に、プロジェクトディレクトリに移動し、必要な npm モジュールをインストールして起動してみる。

```bash
cd express-study
yarn install
```

下記のコマンドで起動することができる。

```bash
DEBUG=experess-study:* PORT=8000 yarn start
```

このコマンドは、`DEBUG` という環境変数には `express-study:*` という値を設定し、`PORT` という環境変数には `8000` という値を設定した状態で `yarn start` コマンドを実行するという意味になる。

なお、`DEBUG=express-study:*` は「アプリケーション固有のデバッグログを表示させる」ために設定している。  
`DEBUG=express-study:*` という書き方で、「`express-study:` で始まる文字列が設定されたログをすべて出力する」設定になる。この `*` はワイルドカードである。

`PORT=8000` は、「このサーバーを 8000 番ポートで起動する」目的で設定している。

yarn +（スクリプト名）というコマンドで package.json であらかじめ設定されたコマンドが実行されるため、yarn start はその仕組みを使って、Express フレームワークで作られた Node.js サーバーを起動させる。

実行後、

```bash
yarn run v1.13.0
$ node ./bin/www
  express-study:server Listening on port 8000 +0ms
```

以上のように表示されれば起動は成功。  
http://localhost:8000/ にアクセスしてみると、Express のジェネレーターで作られた雛形のページが表示される。

<img src="https://cdn.fccc.info/LMwq/soroban/2c0af42e5ff52a1d3f4e43c257e1763e/soroban-guide-2841/e09c5cff-private.png?Expires=1588522755&Key-Pair-Id=APKAIXOVMBEKCVHZBGWQ&Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9jZG4uZmNjYy5pbmZvLyovc29yb2Jhbi8qL3Nvcm9iYW4tZ3VpZGUtMjg0MS8qLioiLCJDb25kaXRpb24iOnsiRGF0ZUxlc3NUaGFuIjp7IkFXUzpFcG9jaFRpbWUiOjE1ODg1MjI3NTV9fX1dfQ__&Signature=BlkupVdUAYDP4fS23ViHqsO-zN6E0~1QHAIkZDdfc0uEvmvvof0O9NasiPuX-WlBKG0-~9NmGLFn81cdBxnU17Ip9rloyhLSvw2wznhTWFCAAoi6X8U3nXa6YOqnkU-syRfrUeCO78Zfoq0ON5EqovryWyYMrxSa7K-dHttC0YaHRSXAcm3Ni2xt5gbAi3KlG1QWHdtu-QQw3FlzdUUC2cfGywpgiY9Apir~WG6db2LwGl9MYfHaeI~I8tOlDClguabdDyDslaB9tUBykhpCgbKkuocQbGFtFgZn00g5aqy~iBLykCDc0lX2t8qo0h5eoljZ1nMxC97cbFeId8QLxw__">

この時に作られるファイル構成は以下のようになっている。

```
.
├── app.js
├── bin
│   └── www
├── package.json
├── public
│   ├── images
│   ├── javascripts
│   └── stylesheets
│       └── style.css
├── routes
│   ├── index.js
│   └── users.js
└── views
    ├── error.pug
    ├── index.pug
    └── layout.pug
```

これらは　Express のジェネレータによって生成されるファイルである。

**説明**
- `app.js`: Web アプリケーション自体を表すモジュールが記述された JavaScript のファイル。アプリケーションの動きを変更する際の実装は、ここを起点に実装されている。
- `bin/www`: 起動を行うための Node.js のスクリプト。このスクリプトにおいて `app` モジュールが読み込まれる。
- `package.json`: おなじみ、npm パッケージの様々な設定が書いてある JSON ファイル。
- `public` ディレクトリ: 画像や、クライアント側で用いる JavaScript、CSS などの静的ファイルが格納される。デフォルトでは `style.css` のみが格納されている。
- `routes` ディレクトリ: この Express でルーティングを担うオブジェクトである Router オブジェクトの実装がされたモジュールのファイルが置かれるディレクトリ。
- `views` ディレクトリ: pug（テンプレートエンジン）のテンプレートファイルが置かれている。

次に、`package.json` の中身を確認してみよう。

```json
{
  "name": "express-study",
  "version": "0.0.0",
  "private": true,
  "scripts": {
    "start": "node ./bin/www"
  },
  "dependencies": {
    "cookie-parser": "~1.4.3",
    "debug": "~2.6.9",
    "express": "~4.16.0",
    "http-errors": "~1.6.2",
    "morgan": "~1.9.0",
    "pug": "2.0.0-beta11"
  }
}
```

以上のように書かれていて、Express がどのようなモジュールに依存しているのかがわかる。

- cookie-parser: Cookie を解釈するモジュール
- debug: デバッグ用のログを表示するモジュール
- express: Express の本体
- http-errors: HTTP のエラーを作成するモジュール
- morgan: コンソールにログを整形して出力するモジュール
- pug: テンプレートエンジン

のようになっている。

試しに、debug というモジュールの機能を使ってみよう。このモジュールは、基本的なログを出力する機能を持つモジュールではあるのだが、`*` で表されるワイルドカードを利用して、出力させたいログを起動時に設定することができる。

では今、下記の差分のように `app.js` の最初の部分を編集してみよう。

<details close>
<summary>app.js</summary>

```diff
+'use strict';
+
+const debug = require('debug');
+const debugInfo = debug('module:info');
+setInterval(() => {
+  debugInfo('some information.');
+}, 1000);
+const debugError = debug('module:error');
+setInterval(() => {
+  debugError('some error.');
+}, 1000);
+
 var createError = require('http-errors');
 var express = require('express');
 var path = require('path');
```

**解説**

```js
const debugInfo = debug('module:info');
setInterval(() => {
  debugInfo('some information.');
}, 1000);
```

これは、`module:info` という設定のロガーを `debugInfo` という変数に用意し、`setInterval` 関数で 1000 ミリ秒（すなわち 1 秒）ごとに `some information.` という文字列をログに出力している。

`const debugError = debug('module:error')` の部分も、`module:error` という設定情報が異なるだけで他は同様。

</details>

ここまで出来たら、実行してみよう。

```
DEBUG=express-study:*,module:* PORT=8000 yarn start
```

以上のコマンドで実行される。`express-study:*` と `module:*` を設定してログを出力させている。このように、複数のログの出力の設定をするときには `,` で続けて書くことができる。

実行すると、1 秒ごとに 2 行ずつカラフルなログが出力され続ける。  
また、すべてのログには、前のログからどれだけの時間が経過したかが `+1s` や `+1ms` という形式で表示される。

このデバッグログを活用することで、処理にどれだけの時間がかかっているかをプロファイルすることもでき、パフォーマンスの問題に対処する際のデバッグが可能。

Express では、以上の debug モジュールを使って見やすいログ出力が行われるように作られている。
このように、フレームワークから様々な便利機能が用意されており、それらを組み合わせ、効率的に開発ができる仕組みが準備されている。

## §2. Express の API
ここまでで Web フレームワークが Web サービスを開発・運営していくために便利な機能を提供してくれることを学んだ。この§では、Express の使い方の中でも以下のことを学んでいく。

- セキュリティに関する設定
- API の使い方

### Express のセキュリティに関する設定
まず最初に、セキュリティに関する設定について。Express を利用したセキュリティに関するベストプラクティスは [Production Best Practices](http://expressjs.com/en/advanced/best-practice-security.html) にまとまっている。

ここでは、

- 脆弱性のあるバージョンを使わない
- HTTPS で TLS（Transport Layer Security: SSL の暗号化通信をより標準化したもの）という暗号化通信を使う
- helmet というモジュール（Express の機能を拡張し、HTTP において脆弱性となるヘッダを取り除くなどし、安全に使えるようにしてくれる）を利用する
- Cookie を安全に使う（デフォルトのキー名を使わない等）
- 依存ライブラリの安全性を確かめる
- 既知の脆弱性に気を付ける

ということが紹介されている。

今回は、実際に helmet の組み込みを行って効果を確認してみよう。

```bash
cd ~/workspace
express --view=pug express-api
cd express-api
yarn install
DEBUG=express-api:* PORT=8000 yarn start
```

以上でサーバーを起動して Chrome で http://localhost:8000/ にアクセスしよう。その後、デベロッパーツールを開く。

Network タブを開き、再読み込みすると、localhost へのアクセスが表示されるので、表示されたリストから localhost と書かれた行を選択する。

詳細が表示されたら、そこから Headers タブを表示しよう。
すると Express が返したレスポンスヘッダの値が表示される。その中に、`X-Powered-By:Express` と書かれたヘッダがある。

<img src="https://cdn.fccc.info/caLV/soroban/3dcad0de40b5f9695d4170eaf1cc5b6b/soroban-guide-2842/f8f89970-private.png?Expires=1588526762&Key-Pair-Id=APKAIXOVMBEKCVHZBGWQ&Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9jZG4uZmNjYy5pbmZvLyovc29yb2Jhbi8qL3Nvcm9iYW4tZ3VpZGUtMjg0Mi8qLioiLCJDb25kaXRpb24iOnsiRGF0ZUxlc3NUaGFuIjp7IkFXUzpFcG9jaFRpbWUiOjE1ODg1MjY3NjJ9fX1dfQ__&Signature=sNsWbVBpDHVQ0eSW2X-9gZDT8xAtBfDkiWb~r5rKCUBuryLghUhdoDVsB5x2QKqBU~HTSq4EIqczSf6HRoYfvY-v5kea3pqIKwmZXfXZES~ckiIptFilXm54npaHxHh9O2PZCZF2wjrWB6nbjwuGUl5l~moOr8vbIOm0ktA9xVVI71ZsCnILJfJ4OYGhUSn2dxYpLSu8pZqhi1RSGlFN-iWB5vft~s9fBU~kqtFiE3E1ZNGNo8nWI94vfMANfypoQPzW7auCpm8TAl2kKZ8Z7hMp0~BBYlSGyScpGlgc9hwjOF79e8D6G68wbdy-Clqru1qQKWsVu-0RBJ1UhByeyA__">

`X-Powered-By` ヘッダは、この Web サービスがどのようなフレームワークやアプリケーションにより作られているかを表示するヘッダ。ここでは値が `Express` となっているため、Express によって開発されていることがわかる。

このヘッダによって、アクセスしたサイトがどのようなフレームワークによって作られているかを知ることができるが、これは危険なことでもある。たとえば、Express に脆弱性があることがわかったときに攻撃対象にされる危険性がある。

このような問題を回避するためにも、`X-Powered-By` というヘッダは送信しないほうが安全なのである。

先ほど説明した helmet というモジュールは、このような Express フレームワークのデフォルトでは危険な挙動を、セキュリティ上問題ないものに変更してくれる。

では helmet をインストールしてみよう。

```bash
yarn add helmet@1.1.0
```

さらに `app.js` を以下の変更差分のように変更する。

<details close>
<summary>app.js</summary>

```diff
 var express = require('express');
 var path = require('path');
 var cookieParser = require('cookie-parser');
 var logger = require('morgan');
+var helmet = require('helmet');

 ...
 var app = express();
+app.use(helmet());
```

`helmet` モジュールを読み込み、`app` というオブジェクト（express のインスタンス）の `use` 関数を使って `helmet` を使うように登録している。

</details>

以上の編集ができたら、再度、サーバーを起動してみる。

```bash
DEBUG=express-api:* PORT=8000 yarn start
```

以上のコマンドで起動させ、先ほど同様に localhost へのアクセスのリクエストヘッダを確認、レスポンスヘッダの中に `X-Powered-By:Express` がなければ対応は成功（残っている場合はスーパーリロード）。

この helmet というモジュールは他にも

- XSS 脆弱性に対処するための Content Security Policy の設定機能
- クリックジャッキングに対処するための frameguard 機能

など、さまざまな脆弱性に対する機能を備えている。

### Express の API

実際にコードを見ながら進めていく。

**app.js**

<details close>
<summary>コード</summary>

```js
var indexRouter = require('./routes/index');
var usersRouter = require('./routes/users');
```
</details>

これは、routes ディレクトリの中にある Router オブジェクトのモジュールをそれぞれ読み込んでいる。では、それらの中身を見ていこう。

**routes/index.js**
<details close>
<summary>コード</summary>

```js
var express = require('express');
var router = express.Router();

/* GET home page. */
router.get('/', function(req, res, next) {
  res.render('index', { title: 'Express' });
});

module.exports = router;
```

</details>

これが、Express における [Router オブジェクト](http://expressjs.com/en/4x/api.html#router) の基本的な使い方になる。
流れとしては、Router を設定して、あとで登場する Application オブジェクトにセットすることによって、特定の path への HTTP へのアクセスを処理することができる。

上記コードでは、GET メソッドで `/` というルートのパスにアクセスがあったときに、`views/index.pug` のテンプレートを利用して、[render 関数](http://expressjs.com/en/4x/api.html#res.render) を呼び、HTML 形式の文字列を作って、レスポンスとして返す、という処理になっている。 

似たような処理を以前、フレームワークを使わずに書いたが、それに比べると、switch 文でメソッドやパスを分岐させたりしなくてよく、シンプルな書き方が可能になっている。

**routes/users.js**

<details close>
<summary>コード</summary>

```js
var express = require('express');
var router = express.Router();

/* GET users listing. */
router.get('/', function(req, res, next) {
  res.send('respond with a resource');
});

module.exports = router;
```

</details>

ほぼ `routes/index.js` と同じ内容だが、`res` というオブジェクトを使うところで render 関数ではなく [send 関数](http://expressjs.com/en/4x/api.html#res.send)が呼ばれている。

`send` 関数は、文字列が渡されると、自動的にその内容をレスポンスのボディの値としてくれる。

それでは、`app.js` に戻ろう。

**app.js**

<details close>
<summary>コード</summary>

```js
var app = express();
app.use(helmet());

// view engine setup
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'pug');
```

</details>

ここで、[Application オブジェクト](http://expressjs.com/en/4x/api.html#app) を express のモジュールを利用して作成し、`app` という変数に格納している。

Application オブジェクトには、use 関数という、Middleware や Router オブジェクトを登録するための関数がある。  
Middleware とは、helmet のような、Express の機能を拡張するモジュールのことである。

`use` 関数を使ったあとは、Application オブジェクトの [set 関数](http://expressjs.com/en/4x/api.html#app.set)を利用して Application の瀬亭を行っている。ここでは、テンプレートのファイルが `views` ディレクトリにあることと、テンプレートエンジンが `pug` であることを設定している。

**app.js 続き**

<details close>
<summary>コード</summary>

```js
app.use(logger('dev'));
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'public')));
```

</details>

以上のコードはそれぞれ

- ログを出すための `logger` を使う設定
- JSON 形式を解釈したり作成するための `json` を使う設定
- URL をエンコードしたりデコードするための `urlencoded` を使う設定
- Cookie を解釈したり作成するための `cookieParser` を使う設定
- 静的なファイルを `public` というディレクトリにするという設定

であり、それぞれ `use` 関数を使って行われている。

<details close>
<summary>コード</summary>

```js
app.use('/', indexRouter);
app.use('/users', usersRouter);
```

</details>

ここでは、`/` というパスにアクセスされたときは `routes/index.js` で記述している Router オブジェクトを、`/users` というパスにアクセスされたときは `routes/users.js` で記述した Router オブジェクトを利用するように記述している。

なお、Express は、このように該当するパスや条件に関して、*Router オブジェクトのような処理を実行するオブジェクトを複数登録して、逐次処理していく仕組み* で作られている。

このときに逐次処理を行っていく関数のことを **ハンドラ** という。

ハンドラには、リクエストとレスポンス、そして `next` という名前の関数が引数として渡される。そして、`next` を実行すると次のハンドラが実行されるようになっている。

<details close>
<summary>コード</summary>

```js
// catch 404 and forward to error handler
app.use(function (req, res, next) {
  next(createError(404));
});
```

</details>

以上のコードは、存在しないパスへのアクセスがあったときの処理が記述されている。エラーを発生させて、そのエラーのプロパティの `status` に 404 というステータスコードを設定し、next 関数を呼び出している。  
この next 関数の呼び出しにより、もし次のリクエストのハンドラが登録されていればそのハンドラが引き続き呼び出される。

<details close>
<summary>コード</summary>

```js
// error handler
app.use(function (err, req, res, next) {
  // set locals, only providing error in development
  res.locals.message = err.message;
  res.locals.error = req.app.get('env') === 'development' ? err : {};

  // render the error page
  res.status(err.status || 500);
  res.render('error');
});
```

</details>

以上のコードは、エラー処理である。`views/error.pug` というテンプレートを使ってエラーを表示させるという記述。
Application オブジェクトに登録された `env` という値が `development` である場合つまり開発環境の場合は、エラーのオブジェクトをテンプレートに渡し、そうでない本番環境におけるエラーのときは、スタックトレースを表示させるエラーオブジェクトはテンプレートに渡さない実装になっている。

では今度は、自分で新たな Router オブジェクトを登録してみよう。  
`router/photos.js` ファイルを作成し、以下のように記述する。

<details close>
<summary>コード</summary>

```js
'use strict';
const express = require('express');
const router = express.Router();

router.get('/', (req, res, next) => {
  res.send('Some photos');
});

router.get('/:title', (req, res, next) => {
    res.send(req.params.title);
})

module.exports = router;
```

</details>

`router.get` 関数の第一引数は、`app.use`　関数で登録される際のパス（下記参照）以下のパスを記述すればよく、ここでは（`/photos/` などとするのでなく）`/` で問題ない。

また、`router.get('/:title', (req, res, next) => {...});` の部分は、Router オブジェクトに渡す path を "/:hoge" と指定することで、パスのパラメーターの hoge という項目を `req.params.hoge` で受け取ることができるという機能を利用しているもの。これにより、動的なルーティングが可能になる。

なお、このままこの動的ルートの機能を使うと XSS 脆弱性がある（path にスクリプトを埋め込めるため）ので、実用の際には、パスで指定されたパラメータが実際に存在するものかどうか確認して意図しない挙動を防ぐ必要がある。

さて、次に、app.js でこの Router オブジェクトを登録する。以下の変更差分のように修正を行おう。

<details close>
<summary>差分</summary>

```diff
 var indexRouter = require('./routes/index');
 var usersRouter = require('./routes/users');
+var photosRouter = require('./routes/photos');
```

```diff
 app.use('/', indexRouter);
 app.use('/users', usersRouter);
+app.use('/photos', photosRouter);
```
</details>

以上完了したら、`DEBUG=express-api:* PORT=8000 yarn start` で再起動を行って、http://localhost:8000/photos にアクセスすると「Some photos」という文字列が表示される。  
また、たとえば http://localhost:8000/photos/vacation にアクセスすると「vacation」という文字列が表示される。

## §3. GitHub を使った外部認証
この回では、フレームワークを使うと便利な、外部サービスを利用したユーザーの外部認証機構を実装してみよう。この回を終えると、Express を利用して GitHub のアカウントでログインできるサイトを作ることができるようになる。

今回、この GitHub のアカウントを利用した外部認証を実装するために **OAuth 2.0** という仕組みを利用する。

### OAuth 2.0
**OAuth 2.0** とは、アプリケーションの利用者が、第三者のアプリケーションに権限を与えることで、そのアプリケーションの機能や認証を第三者のアプリケーションでも利用できるようにする仕組みである。

ここでは、GitHub 上のログインの認証機能を、自分が開発するアプリケーションにおいて利用できるようにしていく。

大まかには以下の流れで認証機能を利用できる。

1. GitHub 上で、自分が開発しているアプリケーションを登録する
1. GitHub への登録時に作ったトークンをアプリケーションに設定する
1. 利用者は、ログインしようとすると、アプリケーションが GitHub のユーザー認証情報を利用してログインしてよいか認可が求められるようになる
1. 利用者が許可すると、アプリケーションは GitHub からその利用者のユーザー情報を受け取れる
1. GitHub から受け取った情報を使って利用者を認証する

<img src="https://cdn.fccc.info/4dt2/soroban/71a364848576b1c1845695d8c9f95f70/soroban-guide-2843/9e51e4b6-private.png?Expires=1588542268&Key-Pair-Id=APKAIXOVMBEKCVHZBGWQ&Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9jZG4uZmNjYy5pbmZvLyovc29yb2Jhbi8qL3Nvcm9iYW4tZ3VpZGUtMjg0My8qLioiLCJDb25kaXRpb24iOnsiRGF0ZUxlc3NUaGFuIjp7IkFXUzpFcG9jaFRpbWUiOjE1ODg1NDIyNjh9fX1dfQ__&Signature=RrCo9DWjTX5mDtllWedcE5xJ50ufWeBOgYTVOTPEVfUetlgFTZeLY5e5XrFLJR2C-fs3DUWFDkVqqjAx3SVcV~RzDAJ970iZ92nu9KMn7it0RzojaQ2LIuSLmHsoIZJpZztQOCyxCTOlVU0F4Vzma1~kOuhYvyMY9UbKrjrAZ4EhVhZPh94WPWPiU0UU9vgnvDRLVrCM~3y47NIX8ehU6VZBIP05AAFXtvTjCBJFJtaxTuFaurfHoNYbZeK~Ym1~6Htbd29EJ6GpBgreWCiZikdVRwCWeQu04QD5P3IyEISRQdGxZMFS8PvB4urF3BSY3-bnLQloBICUDaQtmZpVng__">

仕組みとしては HTTP の Authorization ヘッダを利用するが、認証寺には暗号化されている HTTPS 通信を利用するようにしよう。

----
では以下で実装していく。

まずは今回必要になる npm モジュールをインストールしておく。

```bash
yarn add passport@0.3.2
yarn add passport-github2@0.1.9
yarn add express-session@1.15.6
```

今回利用するモジュールは [passport](http://www.passportjs.org/) というモジュール。このモジュールは、様々な Web サービスとの外部認証を組み込むためのプラットフォームとなるライブラリ。

[passport-github2](https://github.com/cfsghost/passport-github) は、passport が GitHub の OAuth 2.0 認証を利用するためのモジュール。このようなモジュールを Strategy モジュールと呼ぶ。

最後の [express-session](https://github.com/expressjs/session) は、Express でセッションを利用できるようにするためのモジュール。認証した結果をセッション情報として維持するためにこのモジュールが必要になる。

モジュールのインストールが完了したら、今度は、GitHub 上でこれから自分が開発するアプリケーションの登録を行う。

https://github.com/settings/applications/new

以上の URL にアクセスする。このページでは、OAuth 2.0 で連携するためのアプリケーションを登録することができる。

<img src="https://cdn.fccc.info/4c2l/soroban/3cfb734416b36597a96fbf90ea0b189c/soroban-guide-2843/d75d422e-private.png?Expires=1588542268&Key-Pair-Id=APKAIXOVMBEKCVHZBGWQ&Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9jZG4uZmNjYy5pbmZvLyovc29yb2Jhbi8qL3Nvcm9iYW4tZ3VpZGUtMjg0My8qLioiLCJDb25kaXRpb24iOnsiRGF0ZUxlc3NUaGFuIjp7IkFXUzpFcG9jaFRpbWUiOjE1ODg1NDIyNjh9fX1dfQ__&Signature=RrCo9DWjTX5mDtllWedcE5xJ50ufWeBOgYTVOTPEVfUetlgFTZeLY5e5XrFLJR2C-fs3DUWFDkVqqjAx3SVcV~RzDAJ970iZ92nu9KMn7it0RzojaQ2LIuSLmHsoIZJpZztQOCyxCTOlVU0F4Vzma1~kOuhYvyMY9UbKrjrAZ4EhVhZPh94WPWPiU0UU9vgnvDRLVrCM~3y47NIX8ehU6VZBIP05AAFXtvTjCBJFJtaxTuFaurfHoNYbZeK~Ym1~6Htbd29EJ6GpBgreWCiZikdVRwCWeQu04QD5P3IyEISRQdGxZMFS8PvB4urF3BSY3-bnLQloBICUDaQtmZpVng__">

- Application name は、 OAuth test
- Homepage URL は、 http://localhost:8000/
- Application description は、 Test application for GitHub OAuth
- Authorization callback URL は、 http://localhost:8000/auth/github/callback

以上を入力後、「Register application」ボタンをクリックする。これでアプリケーションの登録完了した画面が表示される。

<img src="https://cdn.fccc.info/ZRWw/soroban/37fa9733167c1cae83bb7c4f01c19d32/soroban-guide-2843/9147f88d-private.png?Expires=1588542268&Key-Pair-Id=APKAIXOVMBEKCVHZBGWQ&Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9jZG4uZmNjYy5pbmZvLyovc29yb2Jhbi8qL3Nvcm9iYW4tZ3VpZGUtMjg0My8qLioiLCJDb25kaXRpb24iOnsiRGF0ZUxlc3NUaGFuIjp7IkFXUzpFcG9jaFRpbWUiOjE1ODg1NDIyNjh9fX1dfQ__&Signature=RrCo9DWjTX5mDtllWedcE5xJ50ufWeBOgYTVOTPEVfUetlgFTZeLY5e5XrFLJR2C-fs3DUWFDkVqqjAx3SVcV~RzDAJ970iZ92nu9KMn7it0RzojaQ2LIuSLmHsoIZJpZztQOCyxCTOlVU0F4Vzma1~kOuhYvyMY9UbKrjrAZ4EhVhZPh94WPWPiU0UU9vgnvDRLVrCM~3y47NIX8ehU6VZBIP05AAFXtvTjCBJFJtaxTuFaurfHoNYbZeK~Ym1~6Htbd29EJ6GpBgreWCiZikdVRwCWeQu04QD5P3IyEISRQdGxZMFS8PvB4urF3BSY3-bnLQloBICUDaQtmZpVng__">

この画面に表示されている Client ID と Client Secret をメモしておく。

次に `app.js` を以下のように実装していく。

<details close>
<summary>コード</summary>

```diff
 var cookieParser = require('cookie-parser');
 var logger = require('morgan');
 var helmet = require('helmet');
+var session = require('express-session');
+var passport = require('passport');
+var GitHubStrategy = require('passport-github2').Strategy;
+
+var GITHUB_CLIENT_ID = 'f756acb8748f85e2014b';
+var GITHUB_CLIENT_SECRET = '0fc57f6660bd5da78873eeacda8c131859b64f30';
+
+passport.serializeUser(function (user, done) {
+  done(null, user);
+});
+
+passport.deserializeUser(function (obj, done) {
+  done(null, obj);
+});
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

**解説**

```js
var session = require('express-session');
var passport = require('passport');
var GitHubStrategy = require('passport-github2').Strategy;

var GITHUB_CLIENT_ID = 'f756aXXXXXXXXXX2014b';
var GITHUB_CLIENT_SECRET = '0fc57f666XXXXXXXXXXXXXXXX8c131859b64f30';
```

上記は必要モジュールの読み込みと、先ほどメモした Client ID と Client Secret を設定している。  
実際に公開するアプリケーションで運用する場合にはこれらのトークンは秘匿する必要があるため、環境変数から読み出したり、.gitignore にsエッテイされたバージョン管理しないファイルから読み込んだりすること。

`passport-github2` モジュールからは、Strategy オブジェクトを取得している。

```js
passport.serializeUser(function (user, done) {
  done(null, user);
});

passport.deserializeUser(function (obj, done) {
  done(null, obj);
});
```

上記のコードは、認証されたユーザー情報をどのようにセッションに保存し、どのようにセッションから読み出すかという処理を記述している。
**serializeUser** には、ユーザーの情報をデータとして保存する処理を記述すrす。**deserializeUser** は、保存されたデータをユーザーの情報として読み出す際の処理を設定する。

シリアライズ、デシリアライズとは、メモリ上に参照として飛び散ったデータを 0 と 1 で表せるバイナリのデータとして保存できる形式に変換したり、元に戻したりすることをいう。

上記の実装は、ユーザー情報のすべてをそのままオブジェクトとしてセッションに保存し、そのまますべてを読み出す記述となっている。

また、ここに出てくる done 関数は、第一引数にはエラーを、第二引数には結果をそれぞれ含めて実行する必要がある。

```js
passport.use(new GitHubStrategy({
  clientID: GITHUB_CLIENT_ID,
  clientSecret: GITHUB_CLIENT_SECRET,
  callbackURL: 'http://localhost:8000/auth/github/callback'
},
  function (accessToken, refreshToken, profile, done) {
    process.nextTick(function () {
      return done(null, profile);
    });
  }
));
```

以上のコードは、`passport` モジュールに、GitHub を利用した認証の戦略オブジェクト（`GitHubStrategy(...)`）を設定している。また、認証後に実行する処理を、`process.nextTick` 関数を利用して設定している。

ここも先ほどのシリアライズ処理と同様で、処理が完了した後、done 関数を呼び出す必要がある。

なお、`process.nextTick` 関数を利用せずにここに処理を書いた場合
外部認証を使ったログインが多発した際に、すべての通常の Web サービスが全く動かなくなってしまうという問題が発生してしまう。

いったいなぜか？  
`process.nextTick` 関数は、シングルスレッドで動く Node.js のイベントループを処理する仕組みの中で非常に重要な関数の一つである。  
この関数に、処理したい関数をコールバック関数として渡して実行することで、すぐには実行を行わず、現在の処理が終わった後のタイミングでコールバック関数を実行することができる。

</details>

### Node.js のイベントループの仕組み
これを理解するために、簡単に Node.js のイベントループの仕組みを説明する。

Node.js では非同期 IO を利用している間に別の処理を行ったり、非常に時間がかかる処理のせいで他の処理が滞ってしまわないようにするために、多くの処理がイベントループを利用して記述される。

イベントループは簡略化すると以下のようになっている。

1. setTimeout 関数に登録されたコールバック関数の実行
1. process.nextTick 関数に登録されているコールバック関数の実行（メインモジュールの実行）
1. IO イベントの発生
1. IO イベントのコールバック関数の実行
1. process.nextTick 関数に登録されているコールバック関数の実行

<img src='https://cdn.fccc.info/4Ubm/soroban/7bf0d769be1a56001de2d8000af4f819/soroban-guide-2843/2768c2d7-private.png?Expires=1588542268&Key-Pair-Id=APKAIXOVMBEKCVHZBGWQ&Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9jZG4uZmNjYy5pbmZvLyovc29yb2Jhbi8qL3Nvcm9iYW4tZ3VpZGUtMjg0My8qLioiLCJDb25kaXRpb24iOnsiRGF0ZUxlc3NUaGFuIjp7IkFXUzpFcG9jaFRpbWUiOjE1ODg1NDIyNjh9fX1dfQ__&Signature=RrCo9DWjTX5mDtllWedcE5xJ50ufWeBOgYTVOTPEVfUetlgFTZeLY5e5XrFLJR2C-fs3DUWFDkVqqjAx3SVcV~RzDAJ970iZ92nu9KMn7it0RzojaQ2LIuSLmHsoIZJpZztQOCyxCTOlVU0F4Vzma1~kOuhYvyMY9UbKrjrAZ4EhVhZPh94WPWPiU0UU9vgnvDRLVrCM~3y47NIX8ehU6VZBIP05AAFXtvTjCBJFJtaxTuFaurfHoNYbZeK~Ym1~6Htbd29EJ6GpBgreWCiZikdVRwCWeQu04QD5P3IyEISRQdGxZMFS8PvB4urF3BSY3-bnLQloBICUDaQtmZpVng__'>

このように非同期の IO イベントがいつやってくるかわからないので、IO イベントのコールバック関数が、process.nextTick に登録された関数の呼び出しに挟まる形でイベントループが設計されているのである。

このイベントループに合わせて、nextTick 関数のコールバック関数で処理を細切れにすることによって、他の時間のかかる処理が IO イベントから発生するイベントの処理を待たせる要因にならないようにしている。

なお今回の passport の実装の例における `process.nextTick` で登録する関数の処理には、処理に時間のかかるデータベースへの保存を行う処理が記述される。

では、実装を終えたところで `PORT=8000 yarn start` によりサーバを起動して http://localhost:8000/ にアクセスしてみよう。  
エラーが起きず、"Welcome to Express" と書かれたページが表示されれば問題ない。

なおここでは、ただ設定を行っただけとなる。  
次に、実際に GitHub を利用した認証を行うための処理を記述する。

`app.js` を以下のように記述しよう。

<details close>
<summary>コード</summary>

```diff
+app.use(session({ secret: '417cce55dcfcfaeb', resave: false, saveUninitialized: false }));
+app.use(passport.initialize());
+app.use(passport.session());
+
app.use('/', indexRouter);
app.use('/users', usersRouter);
app.use('/photos', photosRouter);

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
+
 // catch 404 and forward to error handler
 app.use(function (req, res, next) {
   next(createError(404));
 });
```

**解説**

```js
app.use(session({ secret: '417cce55dcfcfaeb', resave: false, saveUninitialized: false }));
app.use(passport.initialize());
app.use(passport.session());
```

以上は、`express-session`（＝`express-session` のインスタンスは `session` 変数に格納されている） と `passport` でセッションを利用するという設定。

`express-session` には、セッション ID が作成されるときに利用される秘密鍵の文字列と、セッションを必ずストアに保存しない設定、セッションが初期化されていなくてもストアに保存しないという設定をそれぞれ付してある。これはどちらも、セキュリティ強化のための設定。

なお、秘密鍵、シークレットは

```bash
node -e "console.log(require('crypto').randomBytes(8).toString('hex'));"
```

以上のコマンドで出力できるランダムな文字列を自分用に設定しておくと安全。

次は、パスに対する HTTP リクエストのハンドラの登録。

```js
app.get('/auth/github',
 passport.authenticate('github', { scope: ['user:email'] }),
 function (req, res) {
});
```

これは、GitHub への認証を行うための処理を、GET で `/auth/github` にアクセスした際に行うというものである。  
またリクエストが行われた際の処理も何もしない関数として登録してある。認証実行寺にログを出力する必要がある場合にはこの関数に記述すればよい。
上記の `passport.authenticate('github', { scope: ['user:email']})` は GitHub に対して、スコープを `user:email` として、認証を行うように設定している。スコープというのは、GitHub の OAuth 2.0 で認可される範囲のことを指す。

[GitHub の OAuth 2.0 のスコープ](https://developer.github.com/v3/oauth/#scopes) には、リポジトリのアクセスやユーザー同士のフォローに関してなど、様々なスコープが存在している。

```js
app.get('/auth/github/callback',
 passport.authenticate('github', { failureRedirect: '/login' }),
 function (req, res) {
  res.redirect('/');
});
```

ここでは OAuth 2.0 の仕組みの中で用いられる、GitHub が利用者の許可に対する問い合わせの結果を送るパスの `/auth/github/callback` のハンドラを登録している。

`passport.authentiate('github', { failureRedirect: '/login' })` で、認証が失敗した際には、再度ログインを促す `/login` にリダイレクトする。

`/login` のルーティングはまだ実装していないため、これからまた実装する。なお、認証に成功していたバイアは、`/` というドキュメントルートにリダイレクトするように実装している。
</details>

ここまで実装できたところで、サーバを再起動して、http://localhost:8000/auth/github にアクセスしてみよう。GitHub にリダイレクトされて、"Authorize application" のページが表示されれば成功。  
ここで Authorize application ボタンをクリックして、このアプリケーションにアクセスする認可を与え、その後、http://localhost:8000/ が表示されれば、外部認証の実装ができたことになる。

ただしこのままだとログアウトできなかったりログインしていることがわからないため、ここからさらに実装を加えていく。

次に、`app.js` に以下のように追記する。

<details close>
<summary> コード </summary>

```diff
   res.redirect('/');
 });

+app.get('/login', function (req, res) {
+  res.render('login');
+});
+
+app.get('/logout', function (req, res) {
+  req.logout();
+  res.redirect('/');
+});
+
 // catch 404 and forward to error handler
 app.use(function (req, res, next) {
   next(createError(404));
 });
```

**解説**

```js
app.get('/login', function (req, res) {
 res.render('login');
});
```

以上は、`/login` に GET でアクセスがあったときに、`login.pug` というテンプレートがあることを前提にログインページを描画するコード。

```js
app.get('/logout', function (req, res) {
 req.logout();
 res.redirect('/');
});
```

以上のコードは、`/logout` に GET でアクセスがあったときにログアウトを行い、`/` のドキュメントルートにリダイレクトさせるもの。

</details>

次に、`views/login.pug` を実装する。views フォルダの中に `login.pug` ファイルを作成し、以下のように実装する。

<details close>
<summary>コード</summary>

```pug
extends layout

block content
  a(href="/auth/github") Login with GitHub
```

pug のテンプレートは **継承** が可能である。継承をすることで、`views/layout.pug` というテンプレートの `content` というブロックだけを書き換えることができる。

なお、`views/layout.pug` は以下のようになっている。

```pug
doctype html
html
  head
    title= title
    link(rel='stylesheet', href='/stylesheets/style.css')
  body
    block content
```

ここの例では、`block content` と記述された部分が、`views/login.pug` において、

```pug
a(href="/auth/github") Login with GitHub
```

と入れ替わる。この仕組みによって、テンプレートを使い回すことができる。
</details>

これで `/login` と `/logout` を実装できたが、くわえて、ルート `/` にアクセスした際に、ログインしていることをわかりやすくしてみよう。

`index.pug` を以下のように実装する。

```pug
 extends layout
 block content
   h1= title
   p Welcome to #{title}
+  if user
+    p Hello, #{user.username}
+    a(href="/logout") Logout
+  else
+    a(href="/login") Login
```

ここでは、テンプレートに `user` というオブジェクトが渡されることを前提に実装し、`user` オブジェクトがある際には `Hello, #{user.username}` と表示し、ログアウトのリンクを表示する。

`user` が存在しない際には、`/login` へのリンクを表示する。

また、テンプレートに合わせて `routes/index.js` を以下のように編集する。

```diff
 /* GET home page. */
 router.get('/', function(req, res, next) {
-  res.render('index', { title: 'Express' });
+  res.render('index', { title: 'Express', user: req.user });
 });
```

`req.user` オブジェクトにユーザ情報が含まれているので、それをそのままテンプレートの `user` というプロパティに含めるように変更することで、テンプレートにオブジェクトを渡せる。

ここまで終わったらサーバを再起動して http://localhost:8000/ にアクセスしてみよう。

うまくいっていれば、Login のリンクが表示され、それをクリックすると Login with GitHub のリンクが表示され、さらにそれをクリックするとログインできて、「Hello （ユーザー名）」という文言とログアウトのリンクが表示されるはず。

この外部認証を利用することで、Web サービスは、ユーザーのパスワードやパスワードのハッシュ値などの情報を管理しなくて済む。

これは予想外の攻撃から Web サービスを守るためにも非常に有益。

なお、ここまで実装したら、`/users` は認証が完了していないと見られないようにしたほうがよいだろう。認証されていない場合には `/login` にリダイレクトするようにする。

そのためには、`app.js` を以下のように書き換えるとよい。

<details close>
<summary>app.js</summary>

```diff
 app.use(passport.session());

 app.use('/', index);
-app.use('/users', users);
+app.use('/users', ensureAuthenticated, usersRouter);
 app.use('/photos', photos);

(中略)

 res.redirect('/');
 });

+function ensureAuthenticated(req, res, next) {
+  if (req.isAuthenticated()) { return next(); }
+  res.redirect('/login');
+}
+
 // catch 404 and forward to error handler
 app.use(function(req, res, next) {
 var err = new Error('Not Found');
```

Router オブジェクトを登録する `app.use` 関数の第一引数にはパス、第二引数に `ensureAuthenticated` 関数、第三引数に Router オブジェクトを渡して呼び出すことで、そのパスへのアクセスに認証が必要となる。

http://localhost:8000/users にはログインしないとアクセスできないことを確かめられれば成功。
</details>