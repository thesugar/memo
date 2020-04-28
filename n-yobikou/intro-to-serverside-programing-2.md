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