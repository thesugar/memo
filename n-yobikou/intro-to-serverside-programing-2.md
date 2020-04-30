# サーバサイドプログラミング入門（後編）
Source: [サーバサイドプログラミング入門](https://www.nnn.ed.nico/courses/497/chapters/6890)

----

上記教材では仮想環境（VirtualBox/ Vagrant）を使っているが、自分は Docker を使用。

- `docker commit` でコミット
    - `docker commit [docker名] [dockerhubのリポジトリ名]`（`docker commit modest_shannon thesugar/hogerepo:fugatag`）
- `docker push` で DockerHub に push
    - `docker push thesugar/hogerepo:fugatag` のようなかたち
- `docker run -d -p <host_port>:<container_port> -v <host_dir>:<container_dir> --name <container_name> <image> <exec_cmd>`
    - `-d`: バックグラウンド実行
    - `-p <host_port>:<container_port>`
        - ホストのポートへのアクセスを、コンテナのポートに転送する
    - `-v <hsot_dir>:<container_dir>`
        - ホストのディレクトリを、コンテナのディレクトリとしてマウントする
    - `--name <container_name> <image> <exec_cmd>`
        - コンテナ名を付けて、イメージからコンテナを生成する。
        - コンテナ生成後に実行するコマンドを記載する(任意)
    - 例（というかメモ）：`docker run -d -p 8000:8000 -it ubuntu:n-yobikou-until-half-of-serversideprogintro bash` でバックグラウンドで実行しつつポートフォワーディング。その後 VS Code でアタッチして中でコード書いたりできる。（バックグラウンド実行とはしなくてもよかったかも）

----

## 認証で利用者を制限する
ここまでで、アンケートを行う Web サービスを Heroku 上で動かせるようになった。しかし、現状のままでは誰でも URL を知っていればアクセスできる状態になっている。今度はこの Web サービスに認証をかけてみる。

### 認証
認証（authentication）とは、コンピューターの利用者の正当性を検証すること。

> 二段階認証  
現在では、ユーザー名とパスワードだけでは必要なセキュリティを確保できないことから、専用のトークンを発行するハードウェアを利用したり携帯電話を利用する二段階認証が用いられることもある。

今回は、認証方法の中でも Basic 認証を行う。

### Basic 認証
Basic 認証とは、HTTP のプロトコルで定義されている、ヘッダの Authorization というヘッダの値に、エンコードされた ID とパスワードを含めて通信することで認証する方式である。このエンコードは **Base64** という暗号化されない方式でなされるため、ただの HTTP で Basic 認証を使用すると盗聴や改竄をされる恐れがある。  
そのため Basic 認証を使用する際は、通信内容が暗号化される HTTPS を介する必要がある。

なお、Basic 認証以外の認証方法でメジャーな方法としては、Cookie というヘッダを利用した認証や、**OAuth 認証** と呼ばれる方式がある。

#### 実装
ここでは簡単のため、HTTP を介する Basic 認証を実装する。
まずは Basic 認証をするためのライブラリをインストールする。

```bash
yarn add http-auth@3.2.3
```

そして、Basic 認証の実装部分は以下のようになる。

**index.js**

```js
// 前略...
const pug = require('pug')
const auth = require('http-auth');
const basic = auth.basic(
    { realm: 'Enquetes Area.' },
    (username, password, callback) => {
        callback(username === 'guest' && password === 'abcdef');
    }
);
const server = http.createServer(basic, (req, res) => {
    // 以下は今まで書いていたのと同じ
```

`auth.basic` 関数にはオブジェクトと無名関数を渡している。オブジェクトの `realm` プロパティは、Basic 認証時に保護する領域を規定する文字列となる。無名関数では、指定された呼び出し方でユーザー名とパスワードを設定している。

また、`http.createServer` 関数の第一引数に basic オブジェクトを渡して Basic 認証に対応させて `server` のオブジェクトを作成している。

これで実行してみる。

```bash
node index.js
```

以上で起動して http://localhost:8000/enquetes/yaki-shabu に Chrome でアクセスすると Basic 認証のダイアログが表示される。

curl の場合は `curl -u guest:abcdef http://localhost:8000/enquetes/yaki-shabu` とすればよい。

Chrome の場合、開発者ツールを開いて、Network タブを開いたうえでアンケートのページを再読み込みし、その後、Network タブに表示された yaki-shabu という項目をクリックして、右側の Headers タブから Request Headers という項目を確認してみる。すると、

```txt
Authorization:Basic Z3Vlc3Q6YWJjZGVm
```

のように先ほど入力された ID とパスワードがエンコードされた情報を確認できる。一見、元のユーザー名とパスワードが推測できないようになっているようにも見えるが、たとえば Chrome の開発者ツールの Console タブで `atob("Z3Vlc3Q6YWJjZGVm")` と入力して実行すると "guest:abcdef" という文字列が表示される。このように、Base64 エンコードはすぐに解読できてしまう（ちなみに、平文 -> Base64 は `btoa('guest:abcdef')` というふうにすればすぐできる。

次に、ログアウト用の URL を作成してみる。Basic 認証では、特定の URL にアクセスした際に、ステータスコード **401 - Unauthorized** （アクセス権不正）を返すことによってログアウトされる。

index.js において、`res.writeHead(200, ...)` という部分の上に以下を追加する。

```js
if (req.url === '/logout') {
    res.writeHead(401, {
        'Content-Type': 'text/plain; charset=utf-8'
    });
    res.end('ログアウトしました');
    return;
}
```

path が `/logout` であるときには、ステータスコード 401 を返す。メッセージ「ログアウトしました」をコンテンツとして書き出して、`res.end` でレスポンスを終了している。また、return 文でこの関数の実行も終えている。

実際に、http://localhost:8000/logout にアクセスする（か curl で叩く）と、「ログアウトしました」と表示される。すると、ブラウザの場合でも、http://localhost:8000/enquetes/yaki-shabu にアクセスすると、また Basic 認証のダイアログが表示されるようになる。

## Cookie を使った秘密の匿名掲示板
ここからは、セキュリティに問題のない秘密の匿名掲示板の Web サービスを作っていく。

### 秘密の匿名掲示板の要件
要件は以下のとおりとする。

- 認証ができる
    - 配布された ID とパスワードによる認証を必須とする
- 認証した人だけが投稿内容を閲覧できる
- 認証した人だけが投稿できる
- 自身の投稿内容を削除できる
- 管理人機能
    - 管理人の投稿だとわかる
    - 管理人は全ての投稿を削除できる
    - 管理人はどのアカウントの投稿かわかる
- 匿名であるけれども同じユーザーであることを認識でき、一人で複数人が書き込んでいるような振る舞いが簡単にはできないようにできる

認証ができる、認証した人だけが投稿内容を閲覧できる、投稿できる、削除できる、は Basic 認証やメソッドなどの HTTP の技術を使えば実現できそう。

最後の要件である「匿名であるけれども〜」は、アカウントの ID を暗号化してしまえばよいかもしれないが、それだと個人が特定されすぎてしまう（匿名掲示板なのに発言同士が紐付きすぎて実質固定ハンドルのようになる）。できれば 1 日の間だけ当人であることを特定できるような仕組みを導入したい。 -> Cookie を使う。

### Cookie
Cookie とは、Web ブラウザに情報を記録するための仕組み。また、有効期限を設定することができる。

内部的には HTTP のリクエストヘッダの `Cookie` という項目と、レスポンスヘッダの `Set-Cookie` という項目を利用することで実現されている。

この Cookie は、Web サービスにおいてセッションの管理を行ったり、広告を出す目的でユーザーの行動履歴を記録するためによく使われている。

---

```bash
cd ~/workspace
mkdir cookie-study
cd cookie-study
touch index.js
```

これから、最後にアクセスした時間を Cookie に記録する実装を行う。HTTP サーバーの記述は今まで同様で、その中で、`res.setHeader` を使って Cookie を設定。

```js
'use strict';
const http = require('http');
const server = http.createServer((req, res) => {
    const now = Date.now();
    res.setHeader('Set-Cookie', 'last_access=' + now + ';')
    const last_access_time = req.headers.cookie ? parseInt(req.headers.cookie.split('last_access=')[1]) : now;
    res.end(new Date(last_access_time).toString());
});
const port = 8000;
server.listen(port, () => {
    console.info('Listening on ' + port);
});
```

**コードの解説**

- Cookie に情報を入れるためには、レスポンスヘッダの `Set-Cookie` ヘッダに、`キー名=値; expire=日付;` の形で値を書き込む必要がある。
また、`expires` は有効期限を設定するためのもの（省略可）。
- 上記コードでは、now という変数に現在時刻のミリ秒の数値を代入して、Cookie として last_access というキー名でヘッダにセットしている。
- `req.headers.cookie` でブラウザから送られた Cookie の内容を参照することができる。
- 上記コードの実装では、ページを開くたびに、前回このページへアクセスした時刻を表す数字が表示される。
- `res.end(new Date(last_access_time).toString())` の部分は、ミリ秒表記を `new Date` に渡し、文字列に変換している。

## UI、URI、モジュールの設計
秘密の匿名掲示板の要件が定義できたところで、今度は大まかな実装方針を決めていく。

今回行う設計は UI設計、URI 設計、モジュール設計の 3 つ。

- UI 設計
    - どのような UI にするのか決める。
    <img src="https://cdn.fccc.info/a91N/soroban/05600edd4f284edc1fc7c58aafcda5b2/soroban-guide-2826/706c904c-private.png?Expires=1588092008&Key-Pair-Id=APKAIXOVMBEKCVHZBGWQ&Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9jZG4uZmNjYy5pbmZvLyovc29yb2Jhbi8qL3Nvcm9iYW4tZ3VpZGUtMjgyNi8qLioiLCJDb25kaXRpb24iOnsiRGF0ZUxlc3NUaGFuIjp7IkFXUzpFcG9jaFRpbWUiOjE1ODgwOTIwMDh9fX1dfQ__&Signature=nefqy7k-C51xND-7VJGxJAVJSicY45YdVkZctCqSAz2mykfmKUVI-sXY~jsvTu9b9l6OIT5fd3-YBixH7aBsRXIWOy1bpp1FnQCrNJ4~kQh4h2rvjljlvDvSLa28XS5-kMAkPwoxLOAhFl4rqoCqp4eoTJsg3TuJdPCXHI-NnCpMSj9LRYMilpasBlnalnUSs3hr3LFU8xSes73FtMt0Ly6wXqEaej~N5wI-i7fDoHnWHNcQDxhgr5wo1i~iB0OJ7MsvS3LJDwxMEaF29SsLTeQmVZFTlfwagxRgqiF2-yFPFAkVhURCFNrkTRD3aTYChO5p86VNYDlpnODkhrxN5Q__">

- URI 設計
    - この UI を構成するために必要な URI を設計する。

    |メソッド|パスとクエリ|処理|
    |---|---|---|
    |GET|/posts|投稿一覧|
    |POST|/posts|投稿とリダイレクト|
    |POST|/posts?delete=1|削除|
    |GET|/logout|ログアウト|
    |GET|/favicon.ico|ファビコン|

    - 投稿一覧の表示には /posts というパスを利用する。また、投稿一覧には投稿フォームも設置する。
    - 投稿には POST メソッドを用いる。また、投稿処理が完了したあと投稿一覧への**リダイレクト**処理を行う。
    - 削除に関しては、DELETE のメソッドを利用したいところだが、実は HTML の `<form>` タグの機能だけでは GET と POST しか使用できない。そのため、ここでは `/posts?delete=1` のように `?delete=1` というクエリをつけて POST を利用する。
    - ログアウト処理は、Basic 認証の Authorization ヘッダを引き継がない 401 のステータスコードを返す URL である。このパスにアクセスすると Basic 認証が解除されログアウトする。
    - favicon も設置する。

- モジュール設計
    - 以下のようなモジュールを構成する。

    |ファイルパス|モジュールの役割|
    |-|-|
    |index.js|HTTP サーバーを起動する|
    |lib/router.js|リクエストを、処理を行うハンドラに振り分ける|
    |lib/posts-handler.js|/posts のリクエストを処理する|
    |lib/handler-utils.js|その他のリクエストを処理する|
    |lib/post.js|投稿を追加、取得、削除する|

    - `index.js` はいつもどおり、JavaScript を実行すると最初に実行されるコード。HTTP サーバを起動したり、Basic 認証の設定などを行う。
    - `lib/router.js` は、リクエストを具体的な処理に振り分けしていくモジュールとする。  
    このように、*リクエストを具体的な処理に振り分ける* ことを **ルーティング** という。
    また、リクエストに対して具体的な処理をする関数を **リクエストハンドラ** といい、ここでは略してハンドラと呼んでいる。
    - `posts-handler.js` と `handler-utils.js` の 2 つのモジュールは、ルーターにより振り分けられたリクエストに対して、具体的な処理を実行するモジュールとなる。/posts に来たリクエストと、そうではないリクエストでモジュールを分けている。
    - `lib/post.js` では、投稿の永続化に関わる具体的な処理である、追加と取得と削除の機能を実現できるようにしていく。

#### リダイレクトについて
上記 URI 設計の部分で出てきた「**リダイレクト**」について。

ためしにコードを書いてみると次のようになる。

index.js

```js
'use strict';
const http = require('http');
const server = http.createServer((req, res) => {
    res.writeHead(302, {
        'Location': 'https://google.co.jp'
    });
    res.end();
});
const port = 8000;
server.listen(port, () => {
    console.info('Listening on ' + port);
})
```

以上がリダイレクトを実行する処理である。ステータスコードの 302 - Found は「`Location` という項目のヘッダで指定されたサイトに内容を発見した」ということを表すステータスコードになる。

以上の内容で HTTP サーバを起動（`node index.js`）して、Chrome で localhost:8000 にアクセスして、Google に転送されれば成功。

## フォームによる投稿機能の実装
要件も設計も固まったので、これから実装を行っていく。
[決めた要件](###秘密の匿名掲示板の要件) の中の「投稿できる」という部分だけを実装していく。

### プロジェクトの作成

```bash
cd ~/workspace
mkdir secret-board
cd secret-board
touch index.js test.js
echo "node_modules/" > .gitignore
yarn init
```

`yarn init` で聞かれる name や version などは基本的に
そのままで Enter を押し続ければよい（description は「秘密の匿名掲示板」に、license は「ISC」としておく）。

package.json に test のスクリプトを加える。

```json
"scripts": {
    "test": "node test.js"
}, ...
```

### 実装

まずは、簡単な HTTP サーバーを実装していく。

<details close>
<summary> index.js </summary>

```js
'use strict';
const http = require('http');

const server = http.createServer((req, res) => {
  res.end('hi');
}).on('error', (e) => {
  console.error('Server Error', e);
}).on('clientError', (e) => {
  console.error('Client Error', e);
});

const port = 8000;
server.listen(port, () => {
  console.info('Listening on ' + port);
});
```

</details>


次に URI 設計を見てここからの作り方を確認してみる。

**（再掲）**

|メソッド|パスとクエリ|処理|
|--|--|--|
|GET|/posts|投稿一覧|
|POST|/posts|投稿とリダイレクト|

投稿一覧の表示と投稿するためのページは、以上のように、`/posts` というパスにアクセスされたときに実行される。

これを踏まえて、

**（再掲）**
|ファイルパス|モジュールの役割|
|--|--|
|lib/router.js|リクエストの処理を行うハンドラに振り分ける|
|lib/posts-handler.js|/posts のリクエストを処理する|

まずは以上ふたつのモジュールが必要になる。それぞれを実装していく。

ただしまずはその前に、モジュールが完成しているものと想定して、`index.js` を以下のように実装する。

<details close>
<summary>
index.js
</summary>

```js
'use strict';
const http = require('http');
const router = require('./lib/router');

const server = http.createServer((req, res) => {
    router.route(req, res);
}).on('error', error => {
    console.error(`Server Error ${error}`);
}).on('clientError', error => {
    console.error(`Client Error ${error}`);
});

const port = 8000;
server.listen(port, () => {
    console.info(`Listening on ${port}`);
})
```
</details>

`router` というモジュールの `route` という関数を呼べば、必要なリクエストの振り分け処理を行ってくれる、という予定で以上のように書く。

なお、モジュールを `require` で読み込む際は、`.js` は省略することができる。

そして、次に `lib/router.js` を以下のように実装する。

**lib/router.js**

```js
'use strict';
const postsHandler = require('./posts-handler');

function route(req, res) {
  switch (req.url) {
    case '/posts':
      postsHandler.handle(req, res);
      break;
    case '/logout':
      // TODO ログアウト処理
      break;
    default:
      break;
  }
}

module.exports = {
  route
};
```

ここでは、`/posts` のパスにアクセスがあったとき、その内容を `posts-handler` モジュールの `handle` 関数に行ってもらうものとする。

ちなみに、`require` でのパスの指定においては `require` を書いたモジュール自身を基準として相対パスを指定するが、`fs.readFile()` など Node.js でファイル操作をするための関数では、変わらず作業ディレクトリ（node コマンドを実行したときに、シェルがいたディレクトリ）を基準として相対パスを指定することに注意。

最後に、`lib/posts-handler.js` を以下のように実装する。

```js
'use strict';
function handle(req, res) {
  switch (req.method) {
    case 'GET':
      res.end('hi');
      break;
    case 'POST':
      // TODO POSTの処理
      break;
    default:
      break;
  }
}

module.exports = {
  handle
};
```

今度は、GET メソッドにアクセスがあった際には HTML の投稿フォームが表示されるようにしてみる。

まずは、テンプレートエンジンの pug をインストールする。

```bash
yarn add pug@2.0.0-rc.4
```

```bash
mkdir views
touch views/posts.pug
```

以上のように投稿フォームを作るテンプレートファイルを作成する。

`views/posts.pug` を以下のように実装する。

```pug
doctype html
html(lang="ja")
  head
    meta(charset="UTF-8")
    title 秘密の匿名掲示板
  body
    h1 秘密の匿名掲示板
    h2 新規投稿
    form(method="post" action="/posts")
      div
        textarea(name="content" cols=40 rows=4)
      div
        button(type="submit") 投稿
```

そうしたら、GET リクエストが来たら `views/posts.pug` を表示するように `lib/posts-handler.js` を変更する。

```js
case 'GET':
    res.writeHead(200, {
        'Content-Type': 'text/html; charset=utf-8'
    });
    res.end(pug.renderFile('./views/posts.pug'));
    break;
```

### 投稿された内容をログに書き出してみよう

今度は投稿を実現するために、投稿された情報をログに書き出して、そのあと投稿フォームにリダイレクトしてみよう。

`lib/posts-hander.js` の `case 'POST':` の部分を実装していく。

最終的には、以下のようになる。

<details close>
<summary>
lib/posts-handler.js
</summary>

```js
'use strict';
const pug = require('pug');
const contents = [];

const handle = (req, res) => {
    switch (req.method) {
        case 'GET':
            res.writeHead(200, {
                'Content-Type' : 'text/html; charset=utf-8'
            });
            res.end(pug.renderFile('./views/posts.pug'));
            break;
        case 'POST':
            let body = [];
            req.on('data', chunk => {
                body.push(chunk);
            }).on('end', () => {
                body = Buffer.concat(body).toString();
                const decoded = decodeURIComponent(body);
                const content = decoded.split('content=')[1];
                console.info('投稿されました: ' + content);
                contents.push(content);
                console.info('投稿された全内容: ' + contents)
                handleRedirectPosts(req, res);
            });
            break;
        default:
            break;
    }
}

const handleRedirectPosts = (req, res) => {
    res.writeHead(303, {
        'Location': '/posts'
    });
    res.end();
}

module.exports = {
    handle
};
```

</details>

```js
  let body = [];
  req.on('data', (chunk) => {
    body.push(chunk);
  }).on('end', () => {
    body = Buffer.concat(body).toString();
    const decoded = decodeURIComponent(body);
    const content = decoded.split('content=')[1];
    console.info('投稿されました: ' + content);
    handleRedirectPosts(req, res);
  });
```

以上は、リクエストで `data` と `end` という名前のイベントが生じたときの処理の実装。


```js
const content = decoded.split('content=')[1];
```

↑では、内容が `key=value` の形式で渡されるため、フォームで設定した `content` というキー名の値の部分を取得している。

```js
console.info('投稿されました: ' + content);
handleRedirectPosts(req, res);
```

↑では、ログを出力し、`handleRedirectPosts` という関数を呼び出している。これは `/posts` へのリダイレクトをハンドリングする関数。

ハンドリングは以下のとおりの実装。

```js
function handleRedirectPosts(req, res) {
  res.writeHead(303, {
    'Location': '/posts'
  });
  res.end();
}
```

ここでは、`/posts` へのリダイレクトを実装している。

今回はステータスコードとして以前使用した 302 - Found ではなく、303 - See Other を利用している。303 は POST でアクセスした際に、その処理の終了後、*GET でも同じパスにアクセスしなおしてほしいとき* に利用するステータスコード。

以上の実装が終わったら、`node index.js` で起動して、Chrome で http://localhost:8000/posts にアクセスしてみる。そうして、投稿フォームに「こんにちは」などと入力して投稿してみるとログに

```
投稿されました: こんにちは
```

と表示される。

## 認証された投稿の一覧表示機能
投稿機能ができあがったところで、今度は、認証されたユーザーにのみ投稿一覧が表示される機能を作っていく。

認証を実装するために、まずは前にも利用した [http-auth](https://www.npmjs.com/package/http-auth) というパッケージをインストールする。

```bash
yarn add http-auth@3.2.3
```

[http-auth](https://www.npmjs.com/package/http-auth) の使い方を調べると、ファイルを利用してユーザーを登録する方法がある。

今回はその方法を利用して、`index.js` に実装していく。

まずはユーザーファイル一覧を作る。

**`users.htpasswd` ファイル**

```
admin:apple
guest1:1234
guest2:5678
```

ここでは、`admin`、`guest1`、`guest2` というユーザーを追加し、それぞれのユーザー名の `:` の後にパスワードを記述して設定している（*⚠️* ここでは平文で管理してしまっていることに注意。機能開発を優先するためいったん平文管理のまま実装するが、最後にセキュリティ問題を解決する実装を学ぶ）。

次に、`index.js` に Basic 認証を実装する。

<details close>
<summary>index.js 変更差分</summary>

```diff
 'use strict';
 const http = require('http');
+const auth = require('http-auth');
 const router = require('./lib/router');

+const basic = auth.basic({
+  realm: 'Enter username and password.',
+  file: './users.htpasswd'
+});
+
-const server = http.createServer((req, res) => {
+const server = http.createServer(basic, (req, res) => {
   router.route(req, res);
 }).on('error', (e) => {
   console.error('Server Error', e);
```
</details>

### ログアウト機能を作ろう
ログアウトは、`/logout` というパスにアクセスして、ステータスコードの 401 - Unauthorized を返す必要がある。

これに伴い、 `/posts` 以外へのアクセスをハンドリングする必要があるので `handler-util` という `/posts` 以外のリクエストをハンドリングするモジュールも作成しよう。

と、その前に、`lib/router.js` を以下のように変更（追記）する。

<details close>
<summary>lib/router.js</summary>

```js
'use strict';
const postsHandler = require('./posts-handler');
const util = require('./handler-util');

const route = (req, res) => {
    switch (req.url) {
        case '/posts':
            postsHandler.handle(req, res);
            break;
        case '/logout':
            util.handleLogout(req, res);
            break;
        default:
            break;
    }
}

module.exports = {
    route
};
```

</details>

次に、`lib/handler-util.js` を以下のように実装する。

<details close>
<summary>lib/handler-util.js</summary>

```js
'use strict';

const handleLogout = (req, res) => {
    res.writeHead(401, {
        'Content-Type': 'text/plain; charset=utf-8'
    });
    res.end('ログアウトしました');
}

module.exports = {
    handleLogout
};
```
</details>

これは、ステータスコード 401 - Unauthorized を返し、その後「ログアウトしました」というテキストをレスポンスに書き込むモジュールである。

最後に、テンプレート `views/posts.pug` にログアウト用のリンクを加える。

```diff
     h1 秘密の匿名掲示板
+    a(href="/logout") ログアウト
     h2 新規投稿
```

### 実装されていない URL には 404 - Not Found を返すようにする
現在は `/posts` と `/logout` 以外のパスへアクセスした際は、サーバーがレスポンスを返さないため、ずっとブラウザ上の表示が終わらない（読み込み中のまま、どうにもならない）。

ここで、現在処理が実装されていない URL にアクセスした際には 404 - Not Found を返すようにしたい。

まず、`lib/router.js` の `switch　(req.url)` で振り分けている部分の `default:` に以下を追加する。

```diff
     default:
+      util.handleNotFound(req, res);
       break;
```

ここでは、*`handleNotFound` という関数がある前提で実装している。*

次に、`lib/handler-util.js` にも以下を追記する。

```js
// ...略
const handleNotFound = (req, res) => {
    res.writeHead(404, {
        'Content-Type': 'text/plain; charset=utf-8'
    });
    res.end('ページが見つかりません。');
}

module.exports = {
    handleLogout,
    handleNotFound
};
```

### 投稿内容一覧ページを作ろう
今度は投稿内容一覧を作っていく。
投稿した内容は、アプリケーション内の配列 `contents` に格納sあれているので、それを表示するようにすればよい。

複数の項目をテンプレートで機能させるには、テンプレートエンジン [pug](https://pugjs.org/language/iteration.html) の iteration の機能を用いる。

投稿内容の配列 `contents` がそのまま、テンプレート内でも利用できると仮定し、以下のように `posts.pug` に追記する。

```pug
// 投稿ボタンの下に以下を追記
h2 投稿一覧
each content in contents
    p #{content}
    hr
```

そうしたら、`lib/posts-handler.js` を以下のように変更して、テンプレートファイルに contents を渡すようにする。。

```js
res.end(pug.renderFile('./views/posts.pug', { contents: contents })); // ※ここは {contents: contents} じゃなくて {contents} としてよい。
```

### 未対応のメソッドのリクエストの処理
`lib/posts-handler.js` において、対応されていないメソッドのリクエストが来た際に、400 - Bad Request というステータスコードを返し、「未対応のメソッドです」というテキストを返すように実装する。

<details close>
<summary>lib/posts-handler.js</summary>

まずは `handler-util` をインポートする。

```js
const util = require('./handler-util');
```

その後、`switch (req.method)` でリクエストメソッドによって振り分けている部分の default 句で未対応のリクエストへのハンドラを呼ぶ（`handleBadRequest` が実装済みという想定で）。

```js
default:
    util.handleBadRequest(req, res);
    break;
```
</details>


<details close>
<summary>lib/handler-util.js</summary>

以下の関数を追加する（もちろん `module.exports` にも `handleBadRequest` を追加すること）。

```js
const handleBadRequest = (req, res) => {
  res.writeHead(400, {
    'Content-Type': 'text/plain; charset=utf-8'
  });
  res.end('未対応のメソッドです');
}
```
</details>

## データベースへの保存機能の実装
ここまで認証機能と投稿、投稿の一覧機能などを作ってきた。

ただし、投稿内容をアプリケーション上の配列に保存しているため、Node.js のサーバーを再起動すると投稿された内容が消えてしまう。

そこで今回は [PostgreSQL](https://www.postgresql.org/) （ポストグレスキューエル）という DB を利用して、投稿内容をファイル上に保存できるようにする（本セクションでは、詳細までは立ち入らず、とりあえず使えるレベルまで）。

### PostgreSQL をインストールする
PostgreSQL のインストールから（以下は Ubuntu 前提）。

```bash
sudo apt update
sudo apt install -y postfresql-10
```

次に、以下のコマンドを実行する。

```bash
sudo su - postgres
```

これは、`postgres` という名前の Linux ユーザーでログインするというコマンド。次に、

```bash
psql
```

以上で、PostgreSQL のターミナル型フロントエンドを起動する。

> この部分で 
> ```
> psql: could not connect to server: No such file or > directory
>         Is the server running locally and accepting
>         connections on Unix domain socket "/var/run/postgresql/.s.PGSQL.5432"?
> ```
> というエラーが出たら、`service postgresql start` とすることで PostgreSQL のサーバーを起動すれば解決するはず）


以下のように表示されればターミナル型フロントエンドの起動は成功。

```bash
$ psql
psql (10.12 (Ubuntu 10.12-0ubuntu0.18.04.1))
Type "help" for help.
```

### パスワードを設定する
そうしたら、以下のコマンドを実行する。

```sql
alter role postgres with password 'postgres';
```

これは `postgres` という PostgreSQL 内のユーザーのパスワードを `postgres` というパスワードに変更するもの。この DB は開発専用であるため簡単なパスワードを設定している。

```
ALTER ROLE
```

と表示されれば成功。

### データベースを作成する
`psql` のターミナルに今度は

```sql
create database secret_board;
```

以上を入力する。これで `secret_board` という名前のデータベースが作成される。

```
CREATE DATABASE
```

と表示されればデータベースの作成は成功。

最後に、`\q` というコマンドを打って psql のターミナルを終了し、さらに `exit` と打って Linux 上の postgres ユーザーからログアウトし、元のユーザーに戻す。

### ソースコードからデータベースを扱ってみよう
では、ここから、ソースコードにデータベースを使う実装を入れていく。

#### sequelize をインストールする
Node.js には DB を利用するためのライブラリがたくさんあるが、ここではデータベース特有のコマンドを知らなくても済むパッケージの sequelize を利用していく。

```bash
yarn add sequelize@4.33.4
yarn add pg@7.4.1
yarn add pg-hstore@2.3.2
```

（*⚠️*: sequelize は古いバージョンでは正常に動作しないおそれがあるため、必ず sequelize@4.33.4 以降をインストールすること）

無事すべてエラーが出ずにインストールできれば、sequelize というモジュールで PostgreSQL を利用できる。

#### データベースの構成について考える
次に、DB にどのような情報を入れたいのか考えてみる。DB には、特定の形式でデータを格納する必要がある。このように、*データの形式を定めること* を **データモデリング** という。

この秘密の匿名掲示板では「投稿」という情報を DB に登録していく必要がある。

この投稿の情報はどのような要素を持っているか、ということを、後で追加する機能まで考えていくと

- 投稿内容
- 投稿したユーザー名
- トラッキング Cookie の内容
- 作成日時

これらが必要になる。これら 4 つの情報を保存できるように sequelize の実装をしていく。

まずは、データの保存などを行うモジュール `lib/post.js` を作成する。

`lib/post.js`

```js
'use strict';
const Sequelize = require('sequelize');
const sequelize = new Sequelize(
  'postgres://postgres:postgres@localhost/secret_board',
  {
    logging: false,
    operatorsAliases: false 
  });
const Post = sequelize.define('Post', {
  id: {
    type: Sequelize.INTEGER,
    autoIncrement: true,
    primaryKey: true
  },
  content: {
    type: Sequelize.TEXT
  },
  postedBy: {
    type: Sequelize.STRING
  },
  trackingCookie: {
    type: Sequelize.STRING
  }
}, {
  freezeTableName: true,
  timestamps: true
});

Post.sync();
module.exports = Post;
```

**解説**
- `const Sequelize = require('sequelize');` でまず sequelize をモジュールとして読み込んでいる
- `const sequelize = new Sequelize(...)` の部分は、DB の設定を渡したうえで、秘密の匿名掲示板を表すデータベースのインスタンスを生成している。
- また、ここでは sequelize が出すログの設定をオフにしており、かつ sequelize の演算子エイリアス（Operators Aliases）も使わないためオフにしている（[このドキュメント](https://sequelize.org/v4/manual/tutorial/querying.html#operators-aliases)が参考になる。SQL インジェクションを防ぐためにユーザーからの入力をサニタイズするようなときに設定するっぽい？）。
- `const Post = sequelize.define('Post', {id: {}, ...}, {freezeTableName: true, timestamps: true});` の部分は、先ほど定義したデータモデルを sequelize の形式にしたがって記述したもの。投稿を `Post` という名前のオブジェクトとして定義している。
- 以下は、DB にデータを入れる際に自動的に増加する整数値を ID として保存してくれる機能をつける設定。入力する情報が一意に特定できるような固有の ID を付加し、それが本当に固有であるのかどうかをチェックしてくれる **primaryKey** という設定をしている。
    ```js
    id: {
    type: Sequelize.INTEGER,
    autoIncrement: true,
    primaryKey: true
    },
    ```
    > **主キー**  
    > DB に情報を格納する際には、あとでその情報を参照するためにキーとなる情報を一緒に付加して保存する。この、情報を一つに特定するための固有な（一意の）データのことを **主キー**（Primary Key）という。  
    この主キーは、DB に格納したデータを後で高速に参照したい場合、設定する必要がある。今後実装する、投稿の削除機能を実現する際に必要となる。（とはいえ RDB では主キーはほぼ必須で設定するイメージだけど（個人的な感想））
- ```js
  content: {
    type: Sequelize.TEXT
  },
  ```
  `content` は、投稿内容を表す。制限のない大きな長い文字列を保存できる設定にしている。
- ```js
  postedBy: {
    type: Sequelize.STRING
  },
  ```
  `postedBy` は、投稿したユーザーの名前。255 文字までの長さを保存できる設定にしている。
- ```js
  trackingCookie: {
    type: Sequelize.STRING
  }
  ```
  `trackingCookie` は、後ほどユーザーごとに付与する Cookie に保存する値。これも、255 文字までの長さを保存できる設定にしている。
- ```js
    {
    freezeTableName: true,
    timestamps: true
    }
  ```
  この `freezeTableName` は、テーブル名を `Post` という名前に固定するという設定。  
  `timestamps: true` は、`createdAt` という作成日時と `updatedAt` という更新日時を*自動的に*追加してくれる設定。
- `Post.sync();` は、定義をした Post というオブジェクト自体をデータベースに適用して同期を取っている。
- `module.exports = Post;` は、このオブジェクト自体をモジュールとしてエクスポート（公開）している。

`lib.post.js` モジュールの実装はこれで完了。

---
#### データベースに保存する
次に、`lib/posts-handler` でデータベースに保存するように実装していく。

まず、以下のとおり、上記で作成した post モジュールをインポートする。

```js
const Post = require('./post');
```

その後、以下のような処理を追記する。

```js
case 'POST':
    let body = [];
    req.on('data', chunk => {
        body.push(chunk);
    }).on('end', () => {
        body = Buffer.concat(body).toString();
        const decoded = decodeURIComponent(body);
        const content = decoded.split('content=')[1];
        console.info('投稿されました: ' + content);
        contents.push(content);
        console.info('投稿された全内容: ' + contents);
        // 新しく追記するのは ↓ ここから！
        Post.create({
            content: content, // ここは content, で OK
            trackingCookie: null,
            postedBy: req.user
        }).then(() => {
            handleRedirectPosts(req, res);
        });
    });
    break;
```

以上の、`Post.create({...})` という部分は、sequelize のデータベース上にデータを作る方法である。投稿内容は content をそのまま使う。Cookie は未実装なのでいったん null にし、投稿したユーザー名は `req.user` で取得する。

これでさっそく起動してみる。http://localhost:8000/posts にアクセスして投稿してみたら、以下のコマンドで DB の内容を表示してみる。

```sql
sudo su -postgres
psql
\c secret_board // `secret_board` テーブルに接続（connect）
select * from "Post";
```

こうすると、DB の内容が表示されるはず。

```
 id |      content       | postedBy | trackingCookie |         createdAt          |         updatedAt          
----+--------------------+----------+----------------+----------------------------+----------------------------
 10 | あいうえお | admin    |                | 2020-04-29 20:34:49.996+00 | 2019-04-29 20:34:49.996+00
(1 row)
```

ここまで確認できたら、q を押してデータベースの表示を終了し、

```
\q
exit
```

と入力して、postgres ユーザーのセッションからログアウトする。

#### データベースから読み出す
次は DB から読み出す実装をする。
`lib/posts-handler.js` は今まで、そのモジュールの中で配列（`contents`）を持って、そこに投稿内容を push していって、その配列を表示する、という格好になっていたが、DB を導入したので、DB から読み出すようにする。

```diff
 const pug = require('pug');
 const util = require('./handler-util');
 const Post = require('./post');
-const contents = [];
```

```diff
     'Content-Type': 'text/html; charset=utf-8'
   });
-  res.end(pug.renderFile('./views/posts.pug', { contents: contents }));
+  Post.findAll().then((posts) => {
+    res.end(pug.renderFile('./views/posts.pug', {
+      posts: posts
+    }));
+  });
   break;
 case 'POST':
   let body = [];
   req.on('data', (chunk) => {
     body.push(chunk);
   }).on('end', () => {
     body = Buffer.concat(body).toString();
     const decoded = decodeURIComponent(body);
     const content = decoded.split('content=')[1];
     console.info('投稿されました: ' + content);
-    contents.push(content);
-    console.info('投稿された全内容: ' + contents);
     Post.create({
       content: content,
       trackingCookie: null,
```

`Post.findAll().then(...)` のところは、sequelize にて投稿内容を取得する実装。自前で実装しなくても findAll() みたいなビルトインのメソッドが使える！なお、findAll() の引数に `{order: [['id', 'DESC']]}` を与えれば、*連番で作成した ID の**降順**にソートされた情報が取得できる（つまり、後で投稿されたものほど先（上）に表示されるようになる）*。

また、今までは `contents` という配列を pug テンプレートに渡していたが、今回の変更で、データベースから取得したデータである `posts` を渡すように変更している。

#### データベースから取得したデータを表示しよう
今度は、`contents` を `posts` に変更したことを受け、pug テンプレートも修正する。

**views/posts.pug**
```diff
       div
         button(type="submit") 投稿
     h2 投稿一覧
-    each content in contents
-      p #{content}
+    each post in posts
+      p #{post.content}
+      p 投稿日時: #{post.createdAt}
       hr
```

投稿内容に加えて、作成日時を表示するようにテンプレートを変更している。
これで今一度 http;//localhost:8000/posts にアクセスしてみると、ログイン後、投稿内容と投稿日時が表示される！

> Tips: データベースの初期化  
  ```bash
  sudo su - postgres
  psql
  ```
> まずは以上のようにして PostgreSQL の対話型フロントエンドを起動する。そこへ  

```sql
drop database secret_board;
create database secret_board;
```

> と入力すれば、データベースを削除し再作成することができる。

## トラッキング Cookie の実装
ここまでの実装で、秘密の匿名掲示板の基本的な機能が利用できるようになった。ここからは基本機能をより良いものにし、運用できるようにするための実装に入る。

残された要件は

- 自身の投稿内容を削除できる
- 管理人機能
- 匿名だけど同じユーザーであることを認識でき、自作自演を防止できる

この 3 つだが、このセクションでは 3 番目の自作自演防止機能を実装していく。ここでは、一日で有効期限が切れるように設定した Cookie を利用してみる。

では、Cookie を付与して、その日の間だけは同じ人の投稿だとわかるようにしよう。仕組みとしては、`/posts` にアクセスがあった際、

- Cookie がついていなければ追跡するための情報を付与
- Cookie がついていれば何もしない
- 投稿時に Cookie から追跡するための情報を取得してそれを投稿の情報に含める

このような要件として実装していく。なお、このようにユーザーの行動を追跡するために付与される Cookie のことを **トラッキング Cookie** と呼ぶ。

具体的な Cookie としては `tracking_id` という名前で、値には**トラッキング ID** をランダムな正の整数値で保存するように実装してみる。

なお、この仕様では `tracking_id` が偽装されたときに、サーバーで生成されたことを検証することができない、というセキュリティの問題がある（つまり偽装し放題）。この問題については別のセクションで修正するものとし、今回はそのまま実装する。

### cookies モジュールをインストールする
今回は、Cookie を利用するために [cookies](https://github.com/pillarjs/cookies) というモジュールを利用する。このモジュールは、Cookie をヘッダに書き込む際に簡単な API で Cookie を利用することができるライブラリである。（<font size=0.1>2020/4/29 時点で GitHub 1000 スター程度だけど実際の使用に耐えるのか？</font>）

```bash
yarn add cookies@0.5.1
```

それでは、

- Cookie がついていなければ追跡するための情報を付与
- Cookie がついていれば何もしない

以上の要件を実装していく。

`lib/posts-handler.js` を以下の差分のように編集する。

```diff
 'use strict';
 const pug = require('pug');
+const Cookies = require('cookies');
 const util = require('./handler-util');
 const Post = require('./post');

+const trackingIdKey = 'tracking_id';
```

ここでは、cookies をモジュールとして読み込み、あとで使う Cookie の名前として `trackingIdKey` という定数を定義している。

```diff
 function handle(req, res) {
+  const cookies = new Cookies(req, res);
+  addTrackingCookie(cookies);
+
  switch (req.method) {
    case 'GET':
      res.writeHead(200, {
```

`handle` 関数の処理の中で、cookies の API に合わせて、`cookies` オブジェクトの作成を行い、`addTrackingCookie` という **関数がある前提** で関数を呼び出している。

```js
function addTrackingCookie(cookies) {
  if (!cookies.get(trackingIdKey)) {
    const trackingId = Math.floor(Math.random() * Number.MAX_SAFE_INTEGER);
    const tomorrow = new Date(Date.now() + (1000 * 60 * 60 * 24));
    cookies.set(trackingIdKey, trackingId, { expires: tomorrow });
  }
}

function handleRedirectPosts(req, res) {
    // ...
}
```

`addTrackingCookie` は以上のように実装する。引数に `cookies` オブジェクトを受け取る。

```js
if(!cookies.get(trackingIdKey)) {
```

この `!cookies.get(trackingIdKey)` は、cookies の API に合わせて、`trackingIdKey` で指定された名前の Cookie の値を取得し、否定演算子 `!` を前に置くことで、`get` 関数で取得した値が undefined や null などの falsy な値のときに true となる式。

つまりこの if 文は、値がないときに処理を実行する。

```js
const trackingId = Math.floor(Math.random() * Number.MAX_SAFE_INTEGER);
```

↑では、ランダムな整数値を生成してトラッキング ID としている。`Math.random` 関数は 0 以上 1 未満のランダムな小数を返すので、それに整数の最大値をかけ、Math.floor 関数で小数点以下を切り捨てている。

> `Math.random()`は（もちろん？）擬似乱数（本当の乱数ではなくアルゴリズムによって擬似的に生成されたもの）。

```js
const tomorrow = new Date(Date.now() + (1000 * 60 * 60 * 24));
```

Date クラスにミリ秒を渡して new でオブジェクトを生成すると、そのミリ秒が指し示す時刻の Date オブジェクトが生成できる。`tommorow` は 1 日後の Date オブジェクトになる。

```js
cookies.set(trackingIdKey, trackingId, { expires: tomorrow });
```

上記コードは、定数 `trackingIdKey` で指定された名前でトラッキング ID の Cookie を設定し、オプションのオブジェうととして、24 時間後の明日に有効期限が切れるように、先ほど作った Date オブジェクト `tommorow` を渡している。

ここまで完了したら、`node index.js` で起動して、http://localhost:8000/posts にアクセスしてログインする。  
Chrome のデベロッパーツールの Application タブから Cookies の項目の localhost を選択し、`tracking_id` の値が含まれていることを確認する。  
また、Expires の項目が 24 時間後に設定されていることも確認する。  
さらに、再読み込みをしても `tracking_id` の値が変化しなければ、実装がうまくいっている。  
なお、Cookie を削除して（左側のバーで右クリック -> Clear で localhost:8000 用の Cookie 全削除あるいは個別のクッキーの上で右クリック -> Delete で個別削除）、またページを再読み込みすると、`tracking_id` には新たな値が設定される。

### トラッキング ID を保存し、表示する
今度は、トラッキング ID をデータベースへの書き込み時に保存し、それを投稿一覧で表示させてみる。

`lib/posts-handler.js` の実装において、今まで `trackingCookie: null` としていた部分は以下の変更差分のように編集する。

```diff
 console.info('投稿されました: ' + content);
 Post.create({
   content: content,
-  trackingCookie: null,
+  trackingCookie: cookies.get(trackingIdKey),
   postedBy: req.user
 }).then(() => {
   handleRedirectPosts(req, res);
```

`cookies` の `get` 関数でクッキーの値を取得してデータベースに保存する（この時点では、先ほどの実装によりユーザーには必ずクッキーが付与されている）。

そして、`views/posts.pug` に以下を追加する。

```diff
       h2 投稿一覧
       each post in posts
+        h3 #{post.id} : ID:#{post.trackingCookie}
         p #{post.content}
         p 投稿日時: #{post.createdAt}
         hr
```

投稿自体の ID（主キー） と投稿者のトラッキング ID をそれぞれ表示するようにした。

さらに、*閲覧情報をサーバーのログに残す*ようにしてみる。

<details close>
<summary> lib/posts-handler.js </summary>

`switch (req.method)` で、リクエストのメソッドごとに処理を振り分けている部分で、GET 要求に対しての処理を書く部分。以下を追記する。

```diff
     res.end(pug.renderFile('./views/posts.pug', {
       posts: posts
     }));
+    console.info(
+      `閲覧されました: user: ${req.user}, ` +
+      `trackingId: ${cookies.get(trackingIdKey) },` +
+      `remoteAddress: ${req.connection.remoteAddress} ` +
+      `userAgent: ${req.headers['user-agent']}`
+    );
   });
   break;
 case 'POST':
```

これで、ユーザー名（`users.htpasswd` に登録したもの＝Basic 認証のユーザー名）とトラッキング ID とリモートアドレス（クライアントの IP アドレス）とユーザーエージェントがログに出力される。
</details>

ここまで完了したら、サーバを起動して、http://localhost:8000/posts にアクセスして確認してみる。

ちなみに、やはりこの実装だと、開発者ツールで Cookie を手打ちで変更するだけで、いくらでも偽装できてしまう（掲示板に投稿者の ID として表示される ID 情報も変更できる）。

## 削除機能の実装
1 日で有効期限を迎えるトラッキング Cookie を利用する実装を行ったので、残る要件は

- 自身の投稿内容を削除できる
- 管理人機能

の 2 つになる。今回実装するのは、「自身の投稿内容を削除できる機能」。

この削除機能において、DB が活躍する。
仮にファイルで全ての投稿を管理していた場合、目的の投稿をファイルの中から探して削除する機能を自分で実装しなければならないが、DB を利用している場合は、DB の削除機能を使えば簡単に実現できる。

### 削除 UI の作成
まずは削除を実行するための UI を作成する。
`views/posts.pug` を以下のように編集する。

```diff
       h3 #{post.id} : ID:#{post.trackingCookie}
       p #{post.content}
       p 投稿日時: #{post.createdAt}
+      - var isDeletable = (user === post.postedBy)
+      if isDeletable
+        form(method="post" action="/posts?delete=1")
+          input(type="hidden" name="id" value=post.id)
+          button(type="submit") 削除
       hr
```

**解説**

以下のコードは、pug のテンプレート内に JavaScript のコードを記述する方法である。各投稿を行ったユーザーのユーザー名 `post.postedBy` が、アクセスしているユーザー名 `user` と同じ場合に、削除可能であるという変数が true になる。（`const` でも OK）

```pug
- var isDeletable = (user === post.postedBy)
```

次に、 `if isDeletable` は pug のテンプレートにおいて条件分岐をする方法。  
if ブロック配下（`form` 以下の部分）は削除を行うフォームとそのボタン。`/posts?delete=1` というパスに POST メソッドでリクエストを送る。

`input` 要素の `type` 属性として `hidden` を指定すると、画面には非表示の部品になる。この非表示の部品で、投稿につけられた ID を送るようにしている。  
この ID を用いて削除処理をサーバー上で行う。

次に、サーバー側で、リクエストをしたユーザー名を pug テンプレートから `user` という名前で使えるように（＝上の `(user === post.postedBy)` の `user` にちゃんとユーザーが入るように）する。

`lib/posts-handler.js` を以下のように実装する（`pug.renderFile('pugのpath', { pugファイルに渡すオブジェクト })` の部分で、ユーザーを受け渡せばいいだけ）。なお、`req.user` には Basic 認証で認証したユーザー名（admin やら guest1 やら）が入っている。

```diff
 });
 Post.findAll({ order: [['id', 'DESC']] }).then((posts) => {
   res.end(pug.renderFile('./views/posts.pug', {
-    posts: posts
+    posts: posts,
+    user: req.user
   }));
   console.info(
   `閲覧されました: user: ${req.user}, ` +
```

ここまで実装すると、各投稿の投稿者のみに投稿削除ボタンが表示されるようになる。現時点では、削除を行うサーバー側の実装がまだないので、削除ボタンをクリックしても「ページが見つかりません」と表示される。

### 削除処理を作ろう
削除処理をサーバー上で実装していく。

#### ルーティングの設定
まずは `lib/router.js` に新しく case 文を追記する。`postsHandker` モジュールと、そのモジュールの `handleDelete` 関数は（後から作るが）実装済みの想定で書いてしまうこと。

```diff
 case '/posts':
   postsHandler.handle(req, res);
   break;
+case '/posts?delete=1':
+  postsHandler.handleDelete(req, res);
+  break;
 case '/logout':
   util.handleLogout(req, res);
   break;
```

#### 削除処理の実装
次に、`lib/posts-hander.js` で具体的な削除処理を実装する。

```js
function handleDelete(req, res) {
  switch (req.method) {
    case 'POST':
      let body = [];
      req.on('data', (chunk) => {
        body.push(chunk);
      }).on('end', () => {
        body = Buffer.concat(body).toString();
        const decoded = decodeURIComponent(body);
        const id = decoded.split('id=')[1];
        Post.findByPk(id).then((post) => {
          if (req.user === post.postedBy) {
            post.destroy().then(() => {
              handleRedirectPosts(req, res);
            });
          }
        });
      });
      break;
    default:
      util.handleBadRequest(req, res);
      break;
  }
}
```

（もちろん、`module.exports = { handle, handleDelete }` とするのも忘れずに） 

**解説**

- `switch (req.method)` の部分は、POST メソッドのときだけ後続の処理が呼ばれるように実装している。
- `let body = [];` 以降の部分は、`req.on('data', chunk => { body.push(chunk); })` では POST のデータ（＝削除処理の場合は、hidden 指定したフォームからの情報「`id=（投稿のID）`」が来る）を受け取っていて、`on('end', () => {...})` の部分で URI エンコードをデコードして、投稿の ID を取得している。
- `Post.findByPk(id).then(...)` は sequelize における削除の実装。
    - まず、`Post.findByPk(id)` の部分で、ID を使って投稿を取得している（Post テーブルの Pk（プライマリーキー）は投稿の ID そのものだから）。
    - 取得ができたら、`then(...)` の中に渡された無名関数が呼ばれる。その時、取得した投稿内容が post に入る。
    - `if (req.user === post.postedBy)` で、投稿を行ったユーザー本人が削除を実行しようとしていることをチェック。
    - 問題なければ、sequelize にビルトインの `destroy` 関数を使って削除し、リダイレクトを行う。

> **Tips: 認証と認可**  
> &emsp;この実装では、そもそも投稿した本人にしか削除ボタンは表示されないのに、サーバーサイドでも改めて投稿した本人かどうかをチェックしている。
> &emsp;これは、今の実装のままでは、サーバーに来た HTTP のリクエストが、どのようなクライアントからのリクエストであるか完全に保証することはできないためである。  
> &emsp;例えば、他の認証ユーザーが悪意を持って、curl などのコマンドラインツールから他人の投稿の削除を試みる可能性がある。
> &emsp;そのようなリクエストが来た場合に削除が行われないよう、（UI（クライアントサイドに削除ボタンを表示するしないのみならず、）サーバー上でもしっかりチェックを行う必要がある。  
> &emsp;このように、特定の機能を利用する権限があるかどうかを、認証されたユーザーに対して確認し、資格に応じて許可することを **認可** という。  
> &emsp;認証と認可は、セキュリティ上きわめて重要な機能となっており、かならずサーバー上でチェックする必要がある。

👉　具体的には、上記のサーバサイドでの if 文による認可処理を書かないと、`curl -X POST http://guest1:1234@localhost:8000/posts?delete=1 -d 'id=10'` のように curl コマンドを叩くことで、guest1 として認証済みでありさえすれば他人（guest1 以外）の投稿も削除できてしまう。  
認可処理を書けば、この curl コマンドを叩くと if 文で遮断できる。

---

これで `node index.js` としてサーバーを再起動して、自分の投稿を削除できれば削除の実装が完成！

> *✍️ MEMO:* ここで、sequelize@4.33.4 という今までのバージョンだと「`Post.findByPk()` という関数なんてないよ」というエラーが出た。調べたところ、ちょっと前までは `findByPk` のかわりに `findById` という関数が使われていて、今は `findByPk` に置き換わったらしい。  
ということで、このタイミングで `yarn upgrade sequelize@5` としてバージョンを引き上げた。  
それに付随して `DeprecationWarning: A boolean value was passed to options.operatorsAliases. This is a no-op with v5 and should be removed.` という warning が出た。演算子エイリアスで false を指定した部分ももういらないっぽい。

> **Tips: 論理削除**  
> 今回実装した `handleDelete` のように、データベースから実際にデータを削除することを **物理削除** という。  
> 一方で、実際にデータを削除するかわりに、「削除された」ことを表すフラグを立ててデータが削除されたとみなすことを **論理削除** という。論理削除を使えば、削除したデータを復元することも可能であるため、実用上はこちらを使うことも多い。

## 管理者機能の実装
削除機能の実装も終わったので、残る機能は管理人機能だけとなった。
管理人機能として要件に挙がっていたものは、

- 管理人の投稿だとわかる
- 管理人はすべての投稿を削除できる
- 管理人はどのアカウントの投稿かわかる

以上の 3 つ。

### 管理人の投稿機能を実装する
以下では、簡易r人ユーザーの名前を `admin` であるとして実装していく。
「管理人の投稿だとわかる」機能は、テンプレートを編集するだけで実装できる。

`views/posts.pug` を以下のように編集する。

```diff
     h2 投稿一覧
     each post in posts
-      h3 #{post.id} : ID:#{post.trackingCookie}
+      if (post.postedBy === 'admin')
+        h3 #{post.id} : 管理人 ✅
+      else
+        h3 #{post.id} : ID:#{post.trackingCookie}
       p #{post.content}
       p 投稿日時: #{post.createdAt}
       - var isDeletable = (user === post.postedBy)
```

<details close>
<summary>ログアウト画面の Modify</summary>
今後、管理人とそうでないユーザーでログインし直すことが多くなるため、ログアウトページに `/posts` へのリンクを作ってしまう。

`lib/handler-utils.js` の `handleLogout` 関数を以下のように編集する。

```diff
 function handleLogout(req, res) {
   res.writeHead(401, {
-    'Content-Type': 'text/plain; charset=utf-8',
+    'Content-Type': 'text/html; charset=utf-8'
   });
-  res.end('ログアウトしました');
+  res.end('<!DOCTYPE html><html lang="ja"><body>' +
+    '<h1>ログアウトしました</h1>' +
+    '<a href="/posts">ログイン</a>' +
+    '</body></html>'
+  );
 }
```

Chrome の一部バージョンでは、真っ白な画面になることがある。その場合は、handleLogout 関数のステータスコードを 401 から 302（一時的にページを転送する）に変更してみる。

</details>

### 管理人の削除機能を実装する
これは、削除を認可していた実装を修正すればよい。
テンプレートにおける表示の部分と、実際の削除処理（サーバーサイド）の 2 箇所を修正する。

- `views/posts.pug` は以下のとおり変更

    ```diff
    -      - const isDeletable = (user === post.postedBy)
    +      - const isDeletable = (user === post.postedBy || user === 'admin')
    ```

    削除可能フラグを true にする条件を、本人または管理人のとき、となるように変更した。

- `lib/posts-hander.js` の `handleDelete` 関数を以下のように編集

    ```diff
    -    if (req.user === post.postedBy) {
    +    if (req.user === post.postedBy || req.user === 'admin') {
    ```

    これも同様の変更。

### 投稿したアカウントがわかる機能の実装
「管理人はどのアカウントによって投稿されたかわかる」機能。これは表示機能なので、テンプレートだけで実装できる。

`views/posts.pug` を以下のように編集する。

```pug
p 投稿日時:...

// 以下 2 行を追加
if (user === 'admin')
    p 投稿者 #{post.postedBy}

const isDeletable = ...
```

こうすることで、admin でログインしているときは全ての投稿について「投稿者: guest1」「投稿者: admin」などと投稿者が表示され、admin 以外でログインした場合にはその表示はされない。  
（この機能、若干ピンと来づらいが、それはこれが匿名掲示板だから。近年の SNS 的なものであれば誰にでも投稿ユーザーがわかるようなものが多いはず）

### 微修正（投稿中の改行、スペースの処理）

ここまでの実装だと、投稿に改行を含めても、投稿一覧の表示では改行として表示されない。改行が表示されるように、改行を HTML タグの `<br />` に置換し、テンプレート上で HTML の表示を許可するようにしてみる。

DB から取得した投稿内容の改行は、以下の処理で `<br />` に置換できる。

**lib/posts-handler.js**

```js
Post.findAll({order: [['id', 'DESC']]}).then(posts => {
    // 以下の forEach を新たに追加
    // 改行を表すエスケープシーケンスを br タグに置換
    posts.forEach(post => {
        post.content = post.content.replace(/\n/g, '<br />');
    });
```

また、pug のテンプレートにおいては `p! = post.content` という記述で、タグをタグと認識し HTML 内に投稿内容を表示できる。こうしないと `<br />` がそのまま文字列として表示されてしまう。

```diff
-      p #{post.content}
+      p!= post.content
       p 投稿日時: #{post.createdAt}
```

また、ここまでの実装では、投稿に半角スペースを加えると `+`（プラスマーク）に変換されていた。 `decodeURIComponent` はプラス文字をデコードしないようだ。デコードする文字列 body について、プラス文字を %20 に置換してからデコードするようにすると解決した。

```js
const decoded = decodeURIComponent(body.replace(/\+/g, '%20'));
```

## デザインの改善
すでに要件としてあがっていた機能を実装し終えた。この回ではデザイン、特に見た目を改善していく。

### favicon を表示させる
まずはファビコンを表示させてみる。実際のファビコン用の画像としては、`https://raw.githubusercontent.com/progedu/secret-board-3026/master/favicon.ico` を使う（wget で取得する）

<img src='https://raw.githubusercontent.com/progedu/secret-board-3026/master/favicon.ico'>

まずは、 `/favicon.ico` へのリクエストをルーティングするために、`lib/router.js` の `route` 関数を以下のように実装する。

```js
case '/logout':
    //...
// ↓ これを追加
case '/favicon.ico':
    util.handleFavicon(req, res);
    break;
```

次に、`lib/handler-util.js` を以下のように実装する。

まずは `const fs = require('fs');` として、FileSystem のモジュール `fs` を読み込むこと。

**lib/handler-util.js**

```js
const handleFavicon = (req, res) => {
    res.writeHead(200, {
        'Content-Type': 'image/vnd.microsoft.icon'
    });
    const favicon = fs.readFileSync('./favicon.ico');
    res.end(favicon);
}
```

あとは、この `handleFavicon` 関数を `module.exports` に含めること。

**解説**
- `'Content-Type': 'image/vnd.microsoft.icon'` の部分は、ファビコンのコンテンツタイプをレスポンスヘッダに書き出している。
-   ```js
    const favicon = fs.readFileSync('./favicon.ico');
    res.end(favicon);
    ```
    これは、ファビコンのファイルを Stream として同期的に読み出し、それをそのままレスポンスの内容として書き出している。    
    ちなみに、`fs` を使う場合は、path は、`fs` を使っているファイル（ここでは `lib/handler-util.js`）からの相対パスではなく、プロジェクトのルートからのパスになる（favicon.ico はプロジェクトのルートディレクトリに配置されている）。
    > 参考：　`require()` は require を書いているファイルからの相対パスで指定する。

これを実装すると、`/favicon.ico` にアクセスするとファビコンが表示されるし、`/posts` にアクセスすると、タブにファビコンが表示される（されない場合は Command + Shift + R）。

### Bootstrap を使って見た目を改善する
#### Bootstrap とは？
Bootstrap は Twitter 社が提供している、どのようなデバイスでページを見ても適切なデザインとなるような HTML、CSS、JavaScript
を提供する Web ページ作りのための部品集。  
様々な部品や見せ方に対する HTML の要素の class が用意されている。

このように単一の HTML で PC やタブレット、スマートフォンなど様々なデバイスに合わせた表示ができるデザインのことを **レスポンシブデザイン** という。

今回変更するのは `views/posts.pug` だけとなる。

### Bootstrap の読み込み
まずは、この Bootstrap の CSS と JavaScript をテンプレートに読み込んでみる。

**`views/posts.pug`**

```diff
 html(lang="ja")
   head
     meta(charset="UTF-8")
+    link(rel="stylesheet",
+    href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css",
+    integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm",
+    crossorigin="anonymous")
```

これは CSS ファイルを読み込む設定をしている。

次に、body 要素の最後に以下を加える。

```diff
         form(method="post" action="/posts?delete=1")
           input(type="hidden" name="id" value="#{post.id}")
           button(type="submit") 削除
       hr
+
+    script(src="https://code.jquery.com/jquery-3.4.1.slim.min.js")
+    script(src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/js/bootstrap.min.js",
+    integrity="sha384-JZR6Spejh4U02d8jOt6vLEHfe/JQGiRRSQQxSfFWpi1MquVdAyjUar5+76PVCmYl",
+    crossorigin="anonymous")
```

ここでは Bootstrap に必要となる JavaScript のソースコードを読み込んでいる。

なお、これらの CSS ファイルや JavaScript ファイルは、Bootstarp がインターネット上で提供しているものを利用している。また、Bootstrap に利用されている jQuery も同じくインターネット上で提供されているものを利用して読み込む。

### デザインの適用
#### body の左右にマージンを与える
まず、最初のデザインの適用として、`body` の左右に適切なマージンを与える class 属性を `body` に設定してみる。

```diff
- body
+ body.container
```

Pug では、`タグ名.クラス名` と続けることで、タグに class 属性を付与することができる。プレーンな HTML でいえば `<body class="container">` に相当。Pug で `body(class="container")` と書いても同様の変換結果になる。

次は、最初の見出しとログアウトリンクの配置を整える。

```diff
-    h1 秘密の匿名掲示板
-    a(href="/logout") ログアウト
+    div.my-3
+      h1 秘密の匿名掲示板
+      a(href="/logout") ログアウト
```
`div` 要素を加えた。`div` 要素の `my-3` というクラスは、余白設定用のクラス。`my-3` の場合、縦方向（`y` 軸方向）への余白（`margin`）を、5 段階用意されているうちの 3 段階目の値に指定するもの。

#### ログアウトリンクをボタンにして右側に配置
今度は、ログアウトリンクをボタンに変更し、右側に配置する。

```diff
   body.container
     div.my-3
+      a(href="/logout").btn.btn-info.float-right ログアウト
       h1 秘密の匿名掲示板
-      a(href="/logout") ログアウト
```

`a` タグにボタンのデザインを適用するクラス `btn` `btn-info`、さらに右側に配置するクラス `float-right` を設定し、`h1` 要素の上に配置する。

Pug では `タグ名.クラス名1.クラス名2.クラス名3` のように書くことで複数のクラスを設定できる。プレーンな HTML で今回の a タグを書くと `<a href="/logout" class="btn btn-info float-right">ログアウト</a>` になる。

#### 投稿フォームのデザインを変更

```diff
     h2 新規投稿
     form(method="post" action="/posts")
-      div
-        textarea(name="content" cols=40 rows=4)
-      div
-        button(type="submit") 投稿
+      div.form-group
+        textarea(name="content" rows="4").form-control
+      div.form-group
+        button(type="submit").btn.btn-primary 投稿
```

#### 投稿一覧の見た目を変える
今度は、投稿一覧を [Card](https://getbootstrap.com/docs/4.0/components/card/) という部品を使って表示する。

<detals close>
<summary>views/posts.pug</summary>

```diff
     h2 投稿一覧
     each post in posts
-      - var isPostedByAdmin = (post.postedBy === 'admin')
-      if isPostedByAdmin
-        h3 #{post.id} : 管理人 ★
-      else
-        h3 #{post.id} : ID:#{post.trackingCookie}
-      p!= post.content
-      p 投稿日時: #{post.createdAt}
-      - var isAdmin = (user === 'admin')
-      if isAdmin
-        p 投稿者: #{post.postedBy}
-      - var isDeletable = (user === post.postedBy || isAdmin)
-      if isDeletable
-        form(method="post" action="/posts?delete=1")
-          input(type="hidden" name="id" value="#{post.id}")
-          button(type="submit") 削除
-      hr
+      div.card.my-3
+        div.card-header
+          - var isPostedByAdmin = (post.postedBy === 'admin')
+          if isPostedByAdmin
+            span #{post.id} : 管理人 ★
+          else
+            span #{post.id} : ID:#{post.trackingCookie}
+        div.card-body
+          p.card-text!=post.content
+        div.card-footer
+          div 投稿日時: #{post.createdAt}
+          - var isAdmin = (user === 'admin')
+          if isAdmin
+            div 投稿者: #{post.postedBy}
+          - var isDeletable = (user === post.postedBy || isAdmin)
+          if isDeletable
+            form(method="post" action="/posts?delete=1")
+              input(type="hidden" name="id" value=post.id)
+              button(type="submit").btn.btn-danger.float-right 削除

      script(src="https://code.jquery.com/jquery-3.4.1.slim.min.js")
      script(src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/js/bootstrap.min.js",
      integrity="sha384-JZR6Spejh4U02d8jOt6vLEHfe/JQGiRRSQQxSfFWpi1MquVdAyjUar5+76PVCmYl",
      crossorigin="anonymous")
```

**解説**
- `div.card.my-3` : Card の要素
- `div.card-header` : ここからが Card の小要素となる。これは Card のヘッダ（`card-header`）。`div` 要素を二階層（`div.card` と `div.card-header`）追加し、管理人または ID が表示されるエリアを `h3` 要素から `span` 要素に変更した（こういう細かいところを落とすとコンソールにごちゃごちゃ警告が出たりする）。
- `div.card-body` が Card の中身。  
`p.card-text!=post.content` と書くことで、`post.content` というコードを `card-text` というクラスが設定された `p` 要素の中に入れる。
- 無論だが、ここで登場する `post.id` や `post.content` はデータベースから取得したデータを示しており、`タグ名.クラス名` とはまったくの別物（それはそう）。
- `div.card-footer` はカードのフッタになる部分。投稿日時と投稿者を `p` 要素から `div` 要素に変更し、`button` に対して `btn-danger` という危険なボタン（削除）であることを示すデザインの class を適用して右に配置。

</detail>

ここまでで機能、デザインがすべて完成した。ただし、まだこの Web サービスにはセキュリティ上の問題が多い。  
次のセクションからは、セキュリティ対策を施していく。

### 時刻表示について
現時点では、投稿日時は `投稿日時: Thu Apr 30 2020 15:16:06 GMT+0900 (Japan Standard Time)` というような形で表示されている。

`yarn add moment-timezone@0.5.0` して Moment Timezone というライブラリをインストールする。

そして、このライブラリを使用するために以下のようにする。

**lib/posts-handler.js**
```js
const moment = require('moment-timezone');
```

```diff
 Post.findAll({ order: [['id', 'DESC']] }).then((posts) => {
   posts.forEach((post) => {
     post.content = post.content.replace(/\n/g, '<br>');
+    post.formattedCreatedAt = moment(post.createdAt).tz('Asia/Tokyo').format('YYYY年MM月DD日 HH時mm分ss秒');
   });
```

この実装で、投稿のオブジェクトに `formattedCreatedAt` というフォーマットされた投稿日の属性を用意することができる。

あとは pug テンプレートから、この `formattedCreatedAt` を利用すれば表示に反映できる。

```diff
- div 投稿日時: #{post.createdAt}
+ div 投稿日時: #{post.formattedCreatedAt}
```
