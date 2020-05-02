# サーバーサイドプログラミング入門 3
(Source: [サーバサイドプログラミング入門  第 27 章以降〜](https://www.nnn.ed.nico/courses/497/chapters/6890))

## 脆弱性
（intro-to-serverside-programming-2.md からの続き）秘密の掲示板のデザインや機能を開発してきたが、ここからはこの Web サービスが持つ脆弱性に対して対策を施していく。

### そもそも脆弱性とは？
脆弱性（vulnerability）の例

- 個人情報などの秘密が勝手に閲覧できてしまう
- Web サイトの内容が書き換えられてしまう
- サイトを閲覧した人をウイルスに感染させてしまう
- なりすましによる不正な行為ができてしまう

このような脆弱性が自分の Web サービスにある場合に起こりうる問題

- 利用者から損害賠償請求をされる
- 信用回復のために費用がかかってしまう
- 利益を生む Web サービスが停止してしまう
- 信用が失われて今後のサービス運営が難しくなる

また日本においては、個人情報をシステムが持つ場合には、[個人情報保護法](http://elaws.e-gov.go.jp/search/elawsSearch/elaws_search/lsg0500/detail?lawId=415AC0000000057&openerCode=1) により、個人情報取扱業者として安全管理のための適切な措置を行う義務が発生する。たとえそれが、*個人が公開する無料のシステムであっても*である。

個人情報の定義：生存する個人に関する情報であって、 当該情報に含まれる氏名、生年月日その他の記述等により特定の個人を識別できるもの （他の情報と容易に照合することができ、 それにより特定の個人を識別することができることとなるものを含む）

### 代表的な脆弱性

|脆弱性|影響|内容|
|-|-|-|
|OSコマンド・インジェクション|大|任意の OS のコマンドを実行できてしまう|
|SQL インジェクション|大|任意の SQL コマンドを実行できてしまう|
|ディレクトリ・トラバーサル|大|任意のファイルを閲覧、操作できてしまう|
|セッション・ハイジャック|大|利用者のセッションが乗っ取られてしまう|
|クロスサイト・スクリプティング（XSS）|中|スクリプトにより Web サイトの改竄ができてしまう|
|クロスサイト・リクエストフォージェリ|中|利用者の意図しない操作がされてしまう|
|HTTP ヘッダインジェクション|中|偽ページの表示などができてしまう|
|クリックジャッキング|小|利用者の意図しないクリックをしてしまう|

ちなみに、現在の秘密の匿名掲示板には、「セッションハイジャック」「クロスサイト・スクリプティング」「クロスサイトリクエストフォージェリ」の 3 つの脆弱性が存在する。  
上記の脆弱性とは別に、パスワードの保護に関する脆弱性、総当たり攻撃に対する脆弱性もある。

### OS コマンド・インジェクション
ここでは、OS コマンド・インジェクションがどういうものなのか実際に体験してみよう。

`os-command-injection` というプロジェクトを作成して、その中に `index.js` を以下のように実装する。

**index.js**

```js
'use strict';
const http = require('http');
const cp = require('child_process');
const server = http.createServer((req, res) => {
  const path = req.url;
  res.writeHead(200, {
    'Content-Type': 'text/plain; charset=utf-8'
  });
  res.end(cp.execSync('echo ' + path));
});
const port = 8000;
server.listen(port, () => {
  console.info('Listening on ' + port);
});
```

これは HTTP サーバーにリクエストされた path の内容を Linux の `echo` コマンドを使って標準出力し、結果を HTTP のレスポンスとして返すプログラムである。

`child_process` モジュールは子プロセス（プロセスとは、OS が実行する処理の単位）を作るモジュールであり、`child_process` モジュールの `execSync` は同期的に与えられたコマンドを実行し、標準出力を結果として取得するもの。

**なお、このプロセスは、わざと脆弱性を引き起こすためのプログラムにしている。**

以上を終えたら、`node index.js` でプログラムを実行して、http://localhost:8000/hoge という URL にアクセスしてみる。

すると

```
/hoge
```

と表示される。アクセスした URL のパスを表示する HTTP サーバーなので正しい挙動のように思える。  
次に、http://localhost:8000/hoge;date というアドレスにアクセスしてみる。すると、`/hoge;date` という文字が表示されるかと思いきや、

```
/hoge
Fri May  1 13:02:09 JST 2020
```

のように表示されてしまう。もちろんこれは、`;` という文字がコマンドの区切りを表す記号であるため、`echo /hoge` と `date` というコマンドであると解釈されたためであり、これが **OS コマンド・インジェクション脆弱性** である。

OS コマンド・インジェクションを防ぐには、クライアントから受け取ったデータを使ってコマンドを実行しないことが重要。仮に止むを得ず使う場合にも、エスケープ処理など、OS のコマンドが自由に実行されないかを入念にチェックする必要がある。  
今回の場合は、単純に `res.end(path)` とするだけでよい。

## XSS 脆弱性の対策
XSS 脆弱性とは、クロスサイト・スクリプティング脆弱性の略称で、Web サービスの外部からの入力で表示が変化する機能において、意図しない HTML、JavaScript、CSS の変更ができる脆弱性。

### XSS 脆弱性の被害例

- 利用者のブラウザで Cookie を読み取り、外部の別のサイトに送信するなどして利用者の Cookie の値を盗み取られる
- 利用者の権限で、意図しない Web サービスの機能が使われてしまう
- Web サイトの情報を書き換えられ、誤った情報が利用者に伝わってしまう

サイトの機能自体が書き換えられてしまうため、その被害は悪用のされかた次第で甚大なものになってしまう。

### 秘密の匿名掲示板で XSS してみる

今まで作成してきた秘密の匿名掲示板の Web アプリを起動して、http://localhost:8000/posts にアクセス・ログインする。

そして以下の内容を投稿してみる。

```html
こんにちは、クラスの人達だけの匿名掲示板、盛り上がるといいですね〜(๑˃̵ᴗ˂̵)و
<script>window.onload=function(){document.getElementsByTagName('h2')[0].innerHTML='新規投稿<small>(本名を入力しても自動的に除去されます)</small>';}</script>
```

こうすると、「新規投稿」と表示されている見出しの右側に、

```
（本名を入力しても自動的に除去されます）
```

という誤った情報が追加されてしまう。騙された投稿者が自分の本名を投稿してしまうとそれが晒されてしまう。

<img src='https://cdn.fccc.info/DOPZ/soroban/d949e52234db8d75d311dd2b5d38e963/soroban-guide-2835/26948455-private.png?Expires=1588349960&Key-Pair-Id=APKAIXOVMBEKCVHZBGWQ&Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9jZG4uZmNjYy5pbmZvLyovc29yb2Jhbi8qL3Nvcm9iYW4tZ3VpZGUtMjgzNS8qLioiLCJDb25kaXRpb24iOnsiRGF0ZUxlc3NUaGFuIjp7IkFXUzpFcG9jaFRpbWUiOjE1ODgzNDk5NjB9fX1dfQ__&Signature=SOx-yZmdJmzkrzwNkWWz7oUr7uSBzAjhcTIORKumVtjv5s0DRdhd40VwbXG9uAFPT7MzSDkjqbgwd6fiJSuik5JQYKfDlJ0ZjMaAAokDj2doMp2wQ1g4kV7u3SLsvMUKtQXDOLSV-bRXV8EeWXzTqmvv0MAqS3Wb~Ac7vBtqD4bPaB5-ZS2EIMvZhbMCnv5DCIZpPuc7lBOjyn69ku6FZm0Nc0hZiKuSUK~27Jv0fFp49pHH0o3BdJEKvZ8XNfr7FUB94~2FXOAmkyFS-jhc6WA-5NadEEcT90b1Sz1NtVTrpBOuOkaii3BW2JHg39JbX6YzSMvFL~-Zu7a-gIch6w__'>

では、上記 XSS 攻撃の詳細を以下で見ていく。

原因は、以下の JavaScript が埋め込まれたこと（わかりやすいように以下では改行して表示）。

```js
<script>
window.onload = function() {
    document.getElementsByTagName('h2')[0]
    .innerHTML
    = '新規投稿<small>(本名を入力しても自動的に除去されます)</small>';
}
</script>
```

投稿内容の部分であっても、HTML タグが含まれていれば、ブラウザは区別せず処理してしまう。  
ここでは `<script>`  タグが書かれているので、悪意を持った JS コードが実行されてしまう。

<details close>
<summary>
スクリプトの内容*
</summary>

```js
window.onload = function() {};
```

以上の構文は、HTML ドキュメントが全部読み込まれてしまったあとに無名関数のスコープの処理を実行するもの。

```js
document.getElementByTagName('h2')[0]
```

これは、HTML ドキュメント内の `h2` 要素で一番最初に見つかったもの（「新規投稿」と書かれた HTML 要素）を取得している。

```js
.innerHTML = '新規投稿<small>(本名を入力しても自動的に除去されます)</small>';
```

この処理は、取得したその要素の中の HTML を書き換えるものとなっている。
</details>

この脆弱性を許してしまった原因は何か？ → それは、テンプレートエンジンの pug で、`<br />` タグを許容するためにエスケープ処理を取り除いたこと。

### 秘密の匿名掲示板の XSS 脆弱性を防ごう
テンプレートエンジン pug には、XSS を防ぐためのエスケープ処理が実装されているので、`views/posts.pug` に、以下の変更差分の修正を入れて対応可能。

```diff
-  p.card-text!=post.content
+  p.card-text #{post.content}
```

こうすることで、`script` 要素がそのまま文字列として表示されるようになる。

### リグレッション
しかしこの修正によって、改行ができていた機能まで失われてしまった。このように、コードに修正を入れることによって本来あった機能が失われることを **リグレッション（regression）** という。

今回は、改行やタブを表示させるための CSS を利用する。

```diff
-  p.card-text #{post.content}
+  p.card-text(style="white-space:pre; overflow:auto;") #{post.content}
```

HTML 要素に style という属性で CSS を指定すると、そのタグだけを対象にしたスタイルを設定できる。  
`white-space:pre; overflow:auto;` は、CSS の `white-space` というプロパティに `pre` という値を、`overflow` というプロパティに `auto` という値を設定している。　　
これで、改行やスペースなどをそのまま表示させるというスタイルが設定されたうえに、あまりに長い要素があった場合はスクロールバーを表示して枠からはみ出さないようになる。

この変更により、`lib/posts-handler.js` で改行（`\n`）を `<br />` に置換する必要もなくなったので、その部分は削除する。

`lib/posts-handler.js`

```diff
- post.content = post.content.replace(/\n/g, '<br>');
```

これで、改行がちゃんと改行として表示されるようになった。

### リグレッションが起きないように、テストを作成する
脆弱性の対応のリグレッションが起きないことを確認するために、テストに追加しておこう。

`test.js` を以下のように実装する。

```js
'use strict';
const pug = require('pug');
const assert = require('assert');

// pug のテンプレートにおける XSS 脆弱性のテスト
const html = pug.renderFile('./views/posts.pug', {
  posts: [{
    id: 1,
    content: '<script>alert(\'test\');</script>',
    postedBy: 'guest1',
    trackingCookie: 1,
    createdAt: new Date(),
    updatedAt: new Date()
  }],
  user: 'guest1'
});

// スクリプトタグがエスケープされて含まれていることをチェック
assert(html.includes('&lt;script&gt;alert(\'test\');&lt;/script&gt;'));
console.log('テストが正常に完了しました');
```

- 上記コードでは、pug テンプレートに渡す `posts` という名前の配列に投稿を表すオブジェクトを一つだけ入れ、そのオブジェクトの `content` プロパティに script タグを打ち込んでいる。
- `assert(...)` の部分では、html に表示された内容が、`&lt;script&gt;alert(\'test\');&lt;/script&gt;` という、安全にエスケープされた内容になっているかを確認している。
- `include` 関数は、引数で指定した文字列が含まれるかどうかを判定する関数。見つかった場合には true を、そうでない場合は false を返す。

`yarn test` （あるいは `note test.js`）でテストを実行して、「テストが正常に完了しました」と表示されれば XSS 脆弱性がないことがわかる。これで問題なく XSS 脆弱性の対応とそのリグレッションの対応を行うことができた。

## パスワードの脆弱性の対策
### 平文でパスワード管理するということ
パスワードを平文で管理するのは好ましくない状態であると言える。
理由は、**OS コマンド・インジェクション** や **SQL インジェクション** のような攻撃により、すべてのユーザーのパスワードが盗み取られてしまう可能性があるため。

平文管理に代わり、今回は **メッセージダイジェスト** を利用したパスワードの保護を行う。

### メッセージダイジェスト
任意の長さのデータを、そのデータを代表する値に変換する関数のことを **ハッシュ関数** といい、このハッシュ関数をかけた結果のことを **ハッシュ値** という。メッセージダイジェストとは、ハッシュ関数を使って作られたそのパスワードの変換結果である。

### ハッシュ関数の種類
ハッシュ関数には様々なアルゴリズムが存在するが、パスワードで利用できるハッシュ関数のアルゴリズムは、以下 2 つの性質を持つことが求められる。

- **原像計算困難性（Preimage resistance）**: 作られたハッシュ値から元の値を推測することができない、または、計算に莫大な時間がかかる
- **衝突困難性（Collision resistance）**: 同じハッシュ値をもつ 2 つのデータを見つけることが難しい

パスワードに使うハッシュ関数のアルゴリズムで、有名なものとしては

- MD5
- SHA
- bcrypt

というものがある。前者の MD5 ハッシュ関数は従来よく使われてきたが、現在では衝突困難性の面で危険があり、簡易な用途でしか用いられない。  
そのため現在では bcrypt や SHA-256 などが標準的なハッシュ関数のアルゴリズムとして用いられる。

### ハッシュ関数を試す
試しにコマンドラインで MD5 のアルゴリズムのハッシュ関数を実行するコマンド `md5sum` を利用してみよう。

コンソールに以下 2 行を入力してみる。

```bash
echo -n apple | md5sum
echo -n 1234 | md5sum
```

`echo` の `-n` オプションは標準出力に改行を含めないというオプション。`md5sum` は、標準入力の値に MD5 のハッシュ関数を適用するというコマンド。

これらのコマンドをパイプでつなぐことで、指定した文字列の MD5 ハッシュ値を求めている。

それぞれ入力すると、以下のような結果が出力される。

```
1f3870be274f6c49b3e31a0c6728957f  -
81dc9bdb52d04dc20036dbd8313ed055  -
```

この文字列がそれぞれ `apple` という文字列、`1234` という文字列を表す MD5 関数によるメッセージダイジェストとなる。

パスワードを保存する際に、パスワードではなくこのハッシュ値を保存しておけばパスワードが流出する心配はなく、そして、入力されたパスワードをチェックする際は、入力値のハッシュ値と、保存されたハッシュ値を比較すれば認証がでいる。

### 秘密の匿名掲示板のパスワード管理を改善しよう
では、平文の代わりにこのハッシュ値をほぞん　するように秘密の匿名掲示板を酒精していく。

元々使っていた `http-auth` モジュールのパスワードファイルは、ハッシュ値を利用したフォーマットにも対応している。  
その認証方法へ置き換えていく。

`bcrypt` アルゴリズムによるハッシュ値を得るためのモジュールをインストールする。

```bash
yarn global add htpasswd@2.4.0
```

以上を実行して、パスワードファイルを作るためのモジュール [htpasswd](https://www.npmjs.com/package/htpasswd) をグローバルインストールする。

そうしたら、以下のコマンドで、パスワードファイルに記述する `bcrypt` アルゴリズムのダイジェストを作成していく。

```bash
htpasswd -D users.htpasswd admin
htpasswd -D users.htpasswd guest1
htpasswd -D users.htpasswd guest2
htpasswd -bB users.htpasswd admin apple
htpasswd -bB users.htpasswd guest1 1234
htpasswd -bB users.htpasswd guest2 5678
```

以上のコマンドは、`users.htpasswd` ファイルで指定したユーザーを削除し、その後指定したユーザー ID とパスワードを `bcrypt` アルゴリズムで追加するというコマンド。3 つのユーザーを一度削除してから、追加しなおしている。

それぞれの行を実行すると

```
Deleting password for user admin.
Deleting password for user guest1.
Deleting password for user guest2.
Adding password for user admin.
Adding password for user guest1.
Adding password for user guest2.
```

以上のように表示される。ここで、`users.htpasswd` の中身を確認して、

```htpasswd
admin:$2a$05$qPuVblJ77Ql9thZq1ZLRN.kL0WF7lEoTh4i5ROuMYh79zPQpK0sMC
guest1:$2a$05$6uFP5EmVZtlJdZDWFgOZF.IJwqp1TW2zuSUbYahkb5Ox7u6W0N0YG
guest2:$2a$05$x.yGMxxrWxKWn3vt6/MVYOrMqiL47j3fxnFJKe6Ct22oCa3GwvIJO
```

となっていれば成功。なお、このパスワードはランダムに生成されるため、実行するたびに中身が変わっていく。

これでサーバを起動して ID: admin、PW: apple で問題なくログインできればパスワードのメッセージダイジェストによる保護は完了！

なお実際の Web サービスではより強固な管理をするために、ハッシュ関数を複数回にわたって適用する **ストレッチ** という手法や、個人ごとに異なる **ソルト** というランダムな秘密の文字列をパスワードに結合してハッシュ関数を適用するなどの工夫が行われている。

### 総当たり攻撃
これで、パスワードの平文管理に関する対策は実施できたが、まだ問題がある。現在の実装では、パスワードが間違っていても何度でもログイン試行できるため、総当たり攻撃を受けてしまうおそれがある。

対策としては「3 回パスワードを間違うとアカウントロックをかける」などが考えられるが、そうしようとする場合、時間経過でロックが自動解除されたり、または管理者はアカウントロックを解除できたり、本人がメールなどで本人確認をしてアカウントをロック解除できるような機能を設けたりしなくてはならないなど、いろいろと大変。

ここでは、総当たり攻撃に対する別の対策を行う。

まずは、総当たり攻撃にはどのような種類のものがあるかを学ぼう。よく行われるものに

- よく設定されるパスワードで認証を行う辞書攻撃
- ID とパスワードが同じアカウントに対して認証を行うジョーアカウント探索
- 全ての文字種から作った短いパスワードのアカウントに対する組み合わせ攻撃

というものがある。

試しに、この掲示板に対して単純な辞書攻撃をし、アカウントが乗っ取れるかチェックしてみよう。

いったん、別のプロジェクト `password-challenger`（ディレクトリ）を作成し、`yarn add request` の実行および `wget https://raw.githubusercontent.com/progedu/password-challenger/master/password` による password という名前のファイルの取得を行う（→ これは、Openwall という団体が提供している、よく使われるパスワード一覧の一部）。

その後、当該プロジェクト配下に以下の内容で `index.js` を作成する。

<details close>
<summary>password-challenger/index.js</summary>

```js
'use strict';
const request = require('request');
const fs = require('fs');
const readline = require('readline');
const rs = fs.ReadStream('password');
const rl = readline.createInterface({ 'input': rs, 'output': {} });
rl.on('line', (line) => {
  // 不正アクセス禁止法に抵触しないよう自分の開発アプリケーションの検証用途に用いること
  request(`http://admin:${line}@localhost:8000/posts`, (error, response, body) => {
    if (!error && response.statusCode === 200) {
      console.log(`Password is "${line}"`);
      process.exit();
    }
  });
});
rl.on('close', () => {
  console.log('password file was closed.');
});
```

**解説**

- `request` という HTTP のクライアントのモジュール、`fs` というファイルシステムのモジュール、`readline` というファイルを一行ずつ読み込むモジュールを読み込む。
- ```js
  const rs = fs.ReadStream('password');
  const rl = readline.createInterface({ 'input': rs, 'output': {} });
  ```
  これは、`password` というファイルを読み取り、`rl` という一行ずつ読み取るオブジェクトを作成している。
- 
    ```js
    rl.on('line', (line) => {
    // 不正アクセス禁止法に抵触しないよう自分の開発アプリケーションの検証用途に用いること
        request(`http://admin:${line}@localhost:8000/posts`, (error, response, body) => {
            if (!error && response.statusCode === 200) {
            console.log(`Password is "${line}"`);
            process.exit();
            }
        });
    });
    ```
    以上が、ファイルが一行読まれたときの処理。ユーザー名は admin で固定、パスワードは読み込んだ一行の文字列が入った変数 `line` としている。

    二つ目の引数 `(error, response, body) => { ... } ` は、リクエストが終わった際の処理を記述する無名関数。アクセスした結果、エラーがなく、リクエストのステータスコードが 200 であれば、認証を突破したとみなせる。

    その時に、コンソールにそのパスワードを表示して、`process.exit()` でプログラムのプロセス自体を終了している。
- ```js
  rl.on('close', () => {
      console.log('password file was closed');
  });
  ```
  以上は、`rl` オブジェクトがファイルを読み込み終わり、`close` というイベントを発行してファイルを閉じた時にコンソールに「password file was closed.」と表示するもの。
</details>

以上の実装が終わったら、ターミナルの一つのウインドウでは、秘密の掲示板の HTTP サーバーを起動させておき、別のターミナルウインドウでこの辞書攻撃ツールを実行してみる。そうすると、

```
password file was closed.
Password is "apple"
```

のように、簡単にパスワードが破られてしまうことがわかる。

### パスワードジェネレーターでランダムなパスワードを作成しよう
では
このような辞書攻撃に脆弱でないパスワードにするためにはどうすればよいのか。

対策のひとつに、**パスワードジェネレーター** によって大文字小文字数字を含む、ランダムな 12 文字以上のパスワードを生成するという方法がある。

パスワードの長さは 12 文字、英子文字と英大文字、数字の 3 種類を含むとして、パスワードジェネレーターを実装してみる。

<details close>
<summary>
password-challenger/password-generator.js
</summary>

```js
'use strict';
const length = 12;
const charset =
  'abcdefghijklmnopqrstuvwxyz' +
  'ABCDEFGHIJKLMNOPQRSTUVWXYZ' +
  '0123456789';

function passwordGenerator() {

  let password = '';
  for (let i = 0; i < length; i++) {
    password += charset[Math.floor(Math.random() * charset.length)];
  }
  const includeAllTypes = /[a-z]/.test(password) && /[A-Z]/.test(password) && /[0-9]/.test(password);
  return includeAllTypes ? password : passwordGenerator();
}

console.log(passwordGenerator());
```

**解説**
- `length`: パスワードの長さ
- `charset`: パスワードに使う文字の候補を格納。記号などを含めたい場合はここに追加する。
- ```js
  const includeAllTypes = /[a-z]/.test(password) && /[A-Z]/.test(password) && /[0-9]/.test(password);
  ```
  以上の部分は、正規表現を使ってパスワードに 3 種類の文字が使われていることを確認している。`/正規表現/.test('文字列')` メソッドは、正規表現と指定された文字列の一致を調べるための検索を実行し、true または false を返す。
- `return includeAllTypes ? password : passwordGenerator()` の部分では、3 種類の文字種が使われていない場合はもう一度 passwordGenerator を返し、パスワードを作り直している。
</details>

以上の実装が終わったら、コンソールからこのスクリプトを実行すればパスワードを生成できる。

そうしたら、秘密の掲示板のプロジェクト（ディレクトリ、リポジトリ）`secret-board` で、

```
htpasswd -D users.htpasswd admin
htpasswd -bB users.htpasswd admin 生成されたパスワード
```

を実行してパスワードを更新。これでサーバーを再起動する。

起動後、`password-challenger` ディレクトリでもう一度辞書攻撃をするスクリプトを試してみる。終了を待っても、突破されたパスワードは表示されないはず。  
これで、簡単な辞書攻撃に対する対策は完了！

> **Tips**  
> ところで、パスワードの管理について、サーバー上ではメッセージダイジェストで管理されますが、 クライアント側となる個人では、どのように管理するのが良いでしょうか。
>
> おすすめの方法は、ブラウザに記憶させて OS の認証機能を用いることです。
Chrome でパスワードを入力する時に、ブラウザに記憶をさせるかダイアログが表示されることがあると思いますが、それをそのまま利用するというものです。
>
> そうした場合、あなたのコンピューターに直接ログインしたりされない限りは、そのパスワードを利用することができません。
またブラウザ内ではパスワードの情報は暗号化されて保管されるというメリットがあります。
>
> 似たような方法に、パスワードを管理してくれるアプリを利用する、という手もあります。
>
> なお古典的な方法ではありますが、このようにジェネレーターで生成したパスワードを紙の手帳などに記入して、それらを大切に管理する方法も比較的安全な方法だと言えます。
>
> ただし、どのような管理方法をするにせよ、
>
> - Web サービスごとに異なるパスワードを使用する
> - 最低でも大文字小文字数字を混ぜた 12 文字以上のパスワードとする
> - 誕生日などの個人情報から推測されないパスワードにする
>
>これらの項目は最低限の対策として、気をつけたほうが良いでしょう。
>
>また非常に重要なアカウントでは、携帯電話や専用トークンを発行する機器を利用した二段階認証（Two-factor Authentication、2FA）を導入すると良いでしょう。
なぜなら、XSS などの脆弱性で偽物のサイトを表示された場合、そこに入力したパスワードを盗まれるなどの事故が起こる可能性があるためです。

## セッション固定化攻撃脆弱性の対策
### セッション
**セッション**とは、システムにログインまたは接続してから、ログアウトまたは切断するまでの一連の操作や通信のことを言う。

秘密の匿名掲示板では、トラッキング Cookie に仕込まれたトラッキング ID で識別できるやりとりがセッションに該当する。

### セッション固定化攻撃
**セッション固定化攻撃**とは、セッションの乗っ取りを行うための攻撃手法の一つ。セッション乗っ取りのことを**セッションハイジャック**という。  
このセッション固定化攻撃では、外部から指定可能なセッションの識別子を利用してセッションハイジャックを行う。

### セッション固定化攻撃を試す
秘密の匿名掲示板において、どのようにセッションが乗っ取られるかを試してみよう。

まず、`guest1` でログインして、普通に書き込む。そうすると例えば ID として「8417464962306809」のような値が表示される（これは掲示板の機能でつけたもの。一日かぎりの ID）。

次に、Chrome のシークレットウィンドウを開く。シークレットウインドウは、Cookie の履歴やブラウザの履歴、検索履歴などを残さないモード。そのため、新たに立ち上げると、まったく Cookie のない状態でサイトにアクセスできる。

今度は、シークレットウインドウで `guest2` でログインしてみる。デベロッパーツールで Application の Cookies の http://localhost:8000 を確認すると、先ほど `guest1` でアクセスしたときとは異なる tracking_id が設定されていることがわかる

ここで、デベロッパーツールの左側に表示されている http://localhost:8000 という文字の上で右クリックし、Clear を選択する。

その後、Console タブを開いて、`document.cookie = 'tracking_id=8417464962306809';` を実行する。

このコードは、tracking_id を新たに設定するもの。ここまできたら、`guest2` でログインしているシークレットウインドウで書き込むと、先ほど `guest1` で書き込んだときと同じ ID が表示され、なりすまし投稿ができてしまう。

これが攻撃者の指定したセッション ID が設定できる攻撃、セッション固定化攻撃である。

### セッション固定化攻撃の対策
簡単な対策方法に「セッションの識別子であるトラッキング ID を表示しない」という方法があるが、これはそもそも、匿名ユーザーをゆるく識別したいという要件上、変更することができない。

ということで、ハッシュ関数を使ってトラッキング ID のなりすましを回避することを考える。

ユーザー ID を使って、トラッキング ID をそのユーザーであることを検証できる形式にする、と言う方法を使う。

トラッキング ID を新たに以下の形式にしよう。

```
(元々のトラッキング ID)_(元々のトラッキング ID とユーザー名を結合したもののハッシュ値)
```

こうすることで、リクエストが来るたびに、トラッキング ID が本当にその人のものであるかをチェックして、違っていた場合は新しいトラッキング ID を振る、ということが可能になる。

具体的にどのようにチェックするかというと、以下のようにする。

1. リクエストで送られた Cookie の値の「元々のトラッキング ID 」と 「元々のトラッキング ID とユーザー名を結合したもののハッシュ値」を分離する
1. 「元々のトラッキング ID 」と「ユーザー名」を結合した文字列をつくる
1. 結合した文字列をハッシュ関数にかけて、ハッシュ値を得る
1. 「送られてきたハッシュ値」と「サーバー上で生成したハッシュ値」が同じであるかを検証する
1. もし偽装されたものであれば、ハッシュ値が異なったり、または付与されていないものとなる

各ユーザーは他のユーザーのユーザー名（`guest1` や `guest2`）を知らないので、自分で他人のハッシュ値は計算できないはず。それゆえ、この形式を使って検証すsることでなりすましを防止できる。

では、この形式を実際に利用してみよう。トラッキング ID をつねに検証し、問題がない場合には利用し、問題がある場合にはトラッキング ID を再度振り直すように実装していく。

<details close>
<summary>lib/posts-handler.js の変更差分</summary>

```diff
 'use strict';
+const crypto = require('crypto');
 const pug = require('pug');
```

```diff
 const trackingIdKey = 'tracking_id';

 function handle(req, res) {
   const cookies = new Cookies(req, res);
-  addTrackingCookie(cookies);
+  const trackingId = addTrackingCookie(cookies, req.user);
   switch (req.method) {
     case 'GET':
       res.writeHead(200, {
```

```diff
 console.info(
   `閲覧されました: user: ${req.user}, ` +
-  `trackingId: ${cookies.get(trackingIdKey) },` +
+  `trackingId: ${trackingId},` +
   `remoteAddress: ${req.connection.remoteAddress}, ` +
   `userAgent: ${req.headers['user-agent']} `
 );
```

```diff
 console.info('投稿されました: ' + content);
 Post.create({
   content: content,
-  trackingCookie: cookies.get(trackingIdKey),
+  trackingCookie: trackingId,
   postedBy: req.user
 }).then(() => {
   handleRedirectPosts(req, res);
```

```diff
   function handleDelete(req, res) {
     …
   }
 }

-function addTrackingCookie(cookies) {
-  if (!cookies.get(trackingIdKey)) {
-    const trackingId = Math.floor(Math.random() * Number.MAX_SAFE_INTEGER);
-    const tomorrow = new Date(Date.now() + (1000 * 60 * 60 * 24));
-    cookies.set(trackingIdKey, trackingId, { expires: tomorrow });
-  }
-}
+/**
+ * Cookieに含まれているトラッキングIDに異常がなければその値を返し、
+ * 存在しない場合や異常なものである場合には、再度作成しCookieに付与してその値を返す
+ * @param {Cookies} cookies
+ * @param {String} userName
+ * @return {String} トラッキングID
+ */
+function addTrackingCookie(cookies, userName) {
+  const requestedTrackingId = cookies.get(trackingIdKey);
+  if (isValidTrackingId(requestedTrackingId, userName)) {
+    return requestedTrackingId;
+  } else {
+    const originalId = Math.floor(Math.random() * Number.MAX_SAFE_INTEGER);
+    const tomorrow = new Date(Date.now() + (1000 * 60 * 60 * 24));
+    const trackingId = originalId + '_' + createValidHash(originalId, userName);
+    cookies.set(trackingIdKey, trackingId, { expires: tomorrow });
+    return trackingId;
+  }
+}
+
+function isValidTrackingId(trackingId, userName) {
+  if (!trackingId) {
+    return false;
+  }
+  const [originalId, requestedHash] = trackingId.split('_'); // 分割代入
+  return createValidHash(originalId, userName) === requestedHash;
+}
+
+function createValidHash(originalId, userName) {
+  const sha1sum = crypto.createHash('sha1');
+  sha1sum.update(originalId + userName);
+  return sha1sum.digest('hex'); // メッセージダイジェストを 16 進数の文字列として取得
+}

 function handleRedirectPosts(req, res) {
```

</details>

**解説**

```js
const crypto = require('crypto');
```

これは、Node.js に組み込まれている `crypto` モジュールを読み込んでいる。この `crypto` モジュールには、ハッシュ関数の他にも様々な暗号化のための関数が実装されている。

```js
const trackingId = addTrackingCookie(cookies, req.user);
```

この部分で、`addTrackingCookie` の戻り値として、有効な ID を得る。  
なお、`addTrackingCookie` はトラッキング ID を検証し、有効ならその ID を返すし、無効であれば新しいトラッキング ID を振り直す（という処理内容に鑑みると `addTrackingCookie` という命名が適切かどうかは疑問）。

そうしたら、`console.info(...)` の中で trackingId を表示している部分や、`Post.create({content:content, trackingCookie: ..., postedBy: req.users})` など、今まで Cookie から直接取得した値を使っていた部分で、上記の `trackingId`（有効だと確認できた ID）を使うようにする。

`addTrackingCookie` 関数は大幅に変更が加えられ、Cookie に含まれているトラッキング ID に異常がなければその値を返し、異常なものである場合は、再度作成し Cookie に付与してその値を返すようにしている。

```js
const requestedTrackingId = cookies.get(trackingIdKey);
```

↑この部分は、Cookie からリクエストに含まれたトラッキング ID を取得している。

```js
function createValidHash(originalId, userName) {
    const sha1sum = crypto.createHash('sha1');
    sha1sum.update(originalId + userName);
    return sha1sum.digest('hex');
}
```

↑以上は、`createValidHash` 関数を実装。これは SHA-1 アルゴリズムを利用して、元々のトラッキング ID とユーザー名を結合した文字列に対してメッセージダイジェストを作成しており、最終的にそのメッセージダイジェストを 16 進数（hex）の文字列として取得している。

これでサーバーを再起動して、先ほどと同じように Cookie を偽装して他人になりすませるか試してみると、偽装できないようになっているはず。  
tracking_id を除去して `document.cookie = 'tracking_id=***_****'` と他人のトラッキング ID を使おうとしても、書き込んだら別の Cookie が発行され、投稿一覧の ID にも、なりすまそうと意図した ID ではなく新しい ID が表示される。

また、ここで、`views/posts.pug` にて、

```diff
-      span #{post.id} : ID:#{post.trackingCookie}
+      span #{post.id} : ID: #{post.trackingCookie.split('_')[0]}
```

とすることで、投稿一覧の ID にはハッシュ値は表示せず元々の ID のみを表示することができる。デベロッパーツールの Cookie を見るともちろん `tracking_id =******_*******` という形でハッシュ値も表示されているとはいえ、生成方法を推測されにくくする一定の効果はあるだろう。

以上がセッション固定化攻撃に対する対策となる。

しかしながら、この方法ではまだ

- ユーザー名を元のトラッキング ID の右側に結合して SHA-1 アルゴリズムのハッシュ関数を適用する

というハッシュ値の作成方法がバレたり推測されたりした場合に、なりすましを許す可能性がある。

次節では、より安全性の高いトラッキング ID を生成する方法を実装していく。

> 所見：以上の対策の肝は、ユーザー名（guest1とか）を組み合わせたハッシュ値を利用する（ユーザー名を組み合わせたハッシュ値を利用しているということは知られないようにしながら）、ということにある（はず）。今回のやり方もおもしろいけど、ユーザー名を利用するのであれば、ユーザー名と ID の組み合わせを DB に持っておいてそこで付き合わせて認可するのも手？（ただこの掲示板の場合毎日 ID が変わる仕様だし、負荷大きくなるか。。）

## より堅牢なセッション管理
ここまでで、とりあえずのセッション固定化攻撃脆弱性の対応を行った。

具体的なセッションを表す識別子であるトラッキング ID を、「元々の ID」と「元々の ID とユーザー名を結合した文字列から作成したハッシュ値」の 2 つで構成することで、トラッキング ID をクライアント側で生成（偽装）して乗っ取りを行うことを防ぐ、という方法を取った。

しかしこの方法では、文字列の結合のしかたやハッシュアルゴリズムが推測されてしまうことで、ハッシュ値を計算可能なのである。

ハッシュ値が計算可能ということは、各自でトラッキング ID を生成してセッションの乗っ取りができてしまう。そこで、今回はより堅牢なトラッキング ID を生成する方法を実装する。

### tracking_id のハッシュ値を推測してなりすましてみよう
まずは、秘密の匿名掲示板に guest1 でログインして適当に投稿する。  
すると、ID として `ID: 5059256563950591` のように番号が表示される。

では、ここで、Node.js の REPL を起動して、

```js
const crypto = require('crypto');
const sha1sum = crypto.createHash('sha1');
sha1sum.update('5059256563950591' + 'guest2'); // ここは guest1 でなく guest2 で計算する
sha1sum.digest('hex');
```

以上のように実行してハッシュ値を求めてみる。これは、「SHA1 アルゴリズムを利用し、ユーザー ID を後ろに結合してハッシュ値を求める」という方法が試行錯誤のすえに推測された、というイメージでこの計算をしている。

こうすると、`cceae5036ac397d48240b8e434935defd09980bc` のようなハッシュ値が表示されるので、先ほど表示された ID と今計算したハッシュ値を `_` で結合して `1784335553602311_cceae5036ac397d48240b8e434935defd09980bc` のような文字列を用意する。

用意できたら、この値をトラッキング ID として　Cookie にセットし、送ってみる。
`guest2` でログインして、デベロッパーツールの Application から Cookies の http://localhost:8000 を Clear して、Console タブで `document.cookie ='tracking_id=1784335553602311_cceae5036ac397d48240b8e434935defd09980bc';` のようにセットする。

そうして書き込むと、「ID:1784335553602311」というように表示され、`guest2` でログインしているにもかかわらず、`guest1` ユーザーが書き込んだときの ID に偽装して書きこめてしまう。

上記方法は、『<「表示されているID + （ログインしている）ユーザー名」をハッシュ化した値>を認可に使っている』という情報がバレればできてしまう方法である。ログインしているユーザー名をくっつければいいので、guest2 が guest1 になりすますときも、guest1 というユーザー名自体は知る必要がなく、<「（guest1 が行った書き込みに表示されている ID）+（ログインしているユーザー名=）guest2」をハッシュ化した値>を渡せば認可が通ってしまう。  
（それゆえ、サーバーの管理者には、なりすましが行われていることは調査でわかる）

### セッションの乗っ取りを防ぐ
この脆弱性を回避するには、ゆーざーめい　に加えて秘密の鍵となる文字列を結合するという方法がある。この方法を使うと、秘密の鍵の文字列がばれないかぎりは、ハッシュ値を再現することが難しくなる。

`lib/posts-handler.js` を以下のように実装する。

<details close>
<summary>lib/posts-handler.js</summary>

```diff
+const secretKey =
+  '5a69bb55532235125986a0df24aca759f69bae045c7a66d6e2bc4652e3efb43da4' +
+  'd1256ca5ac705b9cf0eb2c6abb4adb78cba82f20596985c5216647ec218e84905a' +
+  '9f668a6d3090653b3be84d46a7a4578194764d8306541c0411cb23fbdbd611b5e0' +
+  'cd8fca86980a91d68dc05a3ac5fb52f16b33a6f3260c5a5eb88ffaee07774fe2c0' +
+  '825c42fbba7c909e937a9f947d90ded280bb18f5b43659d6fa0521dbc72ecc9b4b' +
+  'a7d958360c810dbd94bbfcfd80d0966e90906df302a870cdbffe655145cc4155a2' +
+  '0d0d019b67899a912e0892630c0386829aa2c1f1237bf4f63d73711117410c2fc5' +
+  '0c1472e87ecd6844d0805cd97c0ea8bbfbda507293beebc5d9';
+
 function createValidHash(originalId, userName) {
   const sha1sum = crypto.createHash('sha1');
-  sha1sum.update(originalId + userName);
+  sha1sum.update(originalId + userName + secretKey);
   return sha1sum.digest('hex');
 }
```

- `const secretKey=` の部分は、秘密鍵となる文字列を宣言している。ここで宣言した文字列は推測の難しい長い文字列であるため、この文字列が外部に漏れないかぎりは簡単にはハッシュ値を計算することはできない。
- この文字列のデータは

  ```js
  require('crypto').randomBytes(256).toString('hex');
  ```
  を実行して得られたもの。`crypto` モジュールの randomBytes 関数は、推測されづらいランダムなバイト列を生成することができる。  
  `toString` 関数を引数 `hex` という文字列で実行することで、16　進数で表されたランダムなバイト列を文字列として得ることができる。
- `sha1sum.update(originalId + userName + secretKey)` では、SHA1 のハッシュ値を計算する際に、この秘密鍵を最後に結合して計算するようにしている。
</details close>

これでずいぶん安全な方法となり、先ほどと同じ方法ではセッション乗っ取りができなくなった。

上記では秘密の文字列をファイルに直書きしたが、この文字列はバレるといけないので、実際には環境変数に入れるなどして、厳重に管理すべき。

このようなセッションの脆弱性に対応するための一番簡単な方法は、安全なセッション管理を提供している Web のフレームワークを利用するというものがある。そうすることで、このような工夫を自前で行わなくても安全にセッションを利用できる。

しかし、要件によってはセッションの仕組みを自分でいじる必要があることもあり、そうした場合には

1. セッションの識別子を推測されにくいものにする
    - 今回の対応はすべてこれ
1. セッションの識別子を URL のパラメーターに入れない
    - ブラウザには、Referer というヘッダの情報に、どの URL からリンクをたどってきたかを通知する機能があるが、そこから他人に自分のセッション識別子が流出してしまうことを防ぐのに重要。
1. HTTPS 通信で利用する Cookie には secure 属性を設定する
    - Cookie の利用を HTTPS 通信の時のみに限定するもの。通信の経路の盗聴によってセッションの識別子が盗まれることを防ぐことができる。
1. ログインが成功したら新しいセッションの識別子を発行する
    - 今回の掲示板では、1 日の間であれば、ログアウトして再ログインした場合でもユーザーを識別できるようにするという要件があるため対策を行わなかったが、この実装をすることで、セッションの固定化攻撃に対して抜本的な対策を施すことができる。

以上の 4 つが重要。

> おまけ:  
> 現在の実装では、元々のトラッキング ID が推測可能なランダム関数を利用して作られている。`Math.random` 関数は、アルゴリズムとして推測が行いやすいランダム値である。
>
> これを以下のように変更する。
> ```diff
> -    const originalId = Math.floor(Math.random() * Number.MAX_SAFE_INTEGER);
> +    const originalId = parseInt(crypto.randomBytes(8).toString('hex'), 16);
> ```
> こうすることで、推測されにくい 8 バイトの整数値を得ることができる。  
> ※ただし JavaScript の整数値の最大値は、2 の 53 乗 - 1 であり、保持できる精度にも限りがあるためここで得られる数値は 4738441943151179000 のように下三桁から四桁が 000 や 0000 という値に丸められる。

## CSRF 脆弱性の対策
### CSRF 脆弱性とは
CSRF 脆弱性とは、クロスサイト・リクエストフォージェリ脆弱性の略称であり、「シーサーフ」脆弱性と読む。  
ログイン機能の中には、認証の状態を持ち続けたり、セッションを Cookie の内容で持ち続けたりするものがある。  
CSRF 脆弱性はその特性を利用して、偽サイトを通じて外部のサイトからリクエストを送ることで、重要な処理を誤って起こさせる脆弱性である。

### CSRF 脆弱性を攻撃してみよう
秘密の掲示板の CSRF 脆弱性を突くための外部サイトを作成してみる。ここでは、簡単にただの HTML フォームとする。

```bash
cd ~/workspace
mkdir csrf-study; cd csrf-study; touch index.html
```

次に、`csrf-study` フォルダに以下の `index.html` を作成する。

<details close>
<summary> index.html </summary>

```html
<!DOCTYPE html>
<html lang="ja">

<head>
  <meta charset="UTF-8">
  <title>偽物の投稿フォーム</title>
</head>

<body>
<h1>ブラウザの挙動テスト</h1>
<form method="post" action="http://localhost:8000/posts">
  <input type="hidden" name="content" value="まんまと罠にかかりましたね">
  <button type="submit">ボタンを押してみてください</button>
</form>

</body>

</html>
```

</details>

ここまでできたら、CSRF 攻撃を体験してみる。

まずは、秘密の匿名掲示板の HTTP サーバーを起動して、`guest1` でログインしておく。

次に、いま作った外部サイトの `index.html` を Chrome で開いてみる。知り合いから送られてきた URL を開いたらこのページが表示されたという想定にしよう。

> 秘密の匿名掲示板の開発などを docker で行っている場合などでも、ここで作る index.html はホスト OS に作ってしまってよい（docker で、秘密の掲示板とは別のディレクトリに index.html を作るとポートフォワードとかが面倒と思われる）


<img src="https://cdn.fccc.info/oWxa/soroban/d42deb7eecbd96e3e07e35ae314cce83/soroban-guide-2839/5f3bbe6f-private.png?Expires=1588428124&Key-Pair-Id=APKAIXOVMBEKCVHZBGWQ&Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9jZG4uZmNjYy5pbmZvLyovc29yb2Jhbi8qL3Nvcm9iYW4tZ3VpZGUtMjgzOS8qLioiLCJDb25kaXRpb24iOnsiRGF0ZUxlc3NUaGFuIjp7IkFXUzpFcG9jaFRpbWUiOjE1ODg0MjgxMjR9fX1dfQ__&Signature=eZp1UjIl1crJc4pGotjaBnxfa9qM5wgqtTCxvTEYx3hs6-XX3SR5pmHgJcQ9nSDXzjw7Gm147qSr7Ed06fIM3Q7H9x1Qu8Iy5ILjVzs0CK2zUo8qeGvtmBFCt~KQyOTPqf2Oz4fGcgMfSoy3EklpBH-hT6b4Sl5FRiGn7bB0-mB7Hb7RKAjbUTMP~vWik6J-CX32CU9ykAze8L1YL~c1VM-GqjSs-tXojhwRT2SwyglhPg-MOuUuzW7rWp-jqxyYc~oaIsJYJ1-xr0xqK~kK~aTEKDP1vlAE6YQ7~JV4nWjiUz3fXsTrXy8ZpjkfWs75~XN4iwAmg96IVXSmgg0q8Q__">

「ボタンを押してみてください」と書いてあるのでこのボタンを押してみる。そうすると、投稿したつもりはないのに、勝手に「まんまと罠にかかりましたね」という内容で秘密の匿名掲示板に投稿されてしまう。

<img src="https://cdn.fccc.info/P9XS/soroban/4cb9fc0e08e5c7aabd3e95c00f864457/soroban-guide-2839/e9203742-private.png?Expires=1588428124&Key-Pair-Id=APKAIXOVMBEKCVHZBGWQ&Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9jZG4uZmNjYy5pbmZvLyovc29yb2Jhbi8qL3Nvcm9iYW4tZ3VpZGUtMjgzOS8qLioiLCJDb25kaXRpb24iOnsiRGF0ZUxlc3NUaGFuIjp7IkFXUzpFcG9jaFRpbWUiOjE1ODg0MjgxMjR9fX1dfQ__&Signature=eZp1UjIl1crJc4pGotjaBnxfa9qM5wgqtTCxvTEYx3hs6-XX3SR5pmHgJcQ9nSDXzjw7Gm147qSr7Ed06fIM3Q7H9x1Qu8Iy5ILjVzs0CK2zUo8qeGvtmBFCt~KQyOTPqf2Oz4fGcgMfSoy3EklpBH-hT6b4Sl5FRiGn7bB0-mB7Hb7RKAjbUTMP~vWik6J-CX32CU9ykAze8L1YL~c1VM-GqjSs-tXojhwRT2SwyglhPg-MOuUuzW7rWp-jqxyYc~oaIsJYJ1-xr0xqK~kK~aTEKDP1vlAE6YQ7~JV4nWjiUz3fXsTrXy8ZpjkfWs75~XN4iwAmg96IVXSmgg0q8Q__">

上記で作成した HTML は、

```html
<h1>ブラウザの挙動テスト</h1>
<form method="post" action="http://localhost:8000/posts">
  <input type="hidden" name="content" value="まんまと罠にかかりましたね">
  <button type="submit">ボタンを押してみてください</button>
</form>
```

ここの部分でフォームが作られており、http://localhost:8000/posts に POST メソッドで input タグの hidden を利用して投稿内容「まんまと罠にかかりましたね」が値として埋め込まれ、ボタンのクリックによってその内容が送られるようになっている。  
なお、掲示板には別ウインドウでログイン済みであるため、この偽サイトや偽フォームで暗証を突破する必要はない。

この CSRF 攻撃の具体的な被害としては

- 利用者のアカウントで指定しない物品の購入
- 退会処理
- 掲示板への書き込み
- パスワードや登録メールアドレスの変更

これらが勝手に処理されて、莫大な被害が生じるおそれがある。

----
この CSRF 脆弱性への対処方法としては

1. 一度しか利用できないトークンを投稿フォームに埋め込み、そのトークンを消費する形で投稿させる
1. 処理を実施する前にさらに ID とパスワードで認証させる
1. リクエストヘッダのリファラ（Referer）を確認してチェックする

以上 3 種類の方法がある。

Web サービスにおいては、一番目の方式による対応が一般的。このトークンのことを、**ワンタイムトークン** や **CSRF トークン** と呼ぶこともある。  
なぜ一度しか利用できないトークンを利用する？　→　同じトークンの文字列が何度でもリクエストに使える場合、そのトークンが流出した場合に CSRF 攻撃に使われる場合があるから。

ワンタイムトークンという一度しか利用できないトークンであれば、「トークンが発行されてから使用するまで」の間に外部にばれないかぎりは、勝手にリクエストが処理されることがなくなる。

なお、二番目と三番目の方法には以下のようなデメリットがあるため、あまり一般的に用いられない。

- 二番目の ID とパスワードを再度入力させる方法対策では、利用者の利便性を損なうおそれがある
- 三番目のリファラを確認する方法は、XSS により同一サイト内で HTML が改竄された場合には攻撃に対して無防備になる

### ワンタイムトークンを実装してみる
<details close>
<summary>lib/posts-handler.js を以下の変更差分のように編集する</summary>

```diff
 const Post = require('./post');

 const trackingIdKey = 'tracking_id';

+const oneTimeTokenMap = new Map(); // キーをユーザー名、値をトークンとする連想配列
+
 function handle(req, res) {
   const cookies = new Cookies(req, res);
   const trackingId = addTrackingCookie(cookies, req.user);
```

これは、ワンタイムトークンをサーバ上に保存しておくための連想配列の宣言。

```diff
       post.content = post.content.replace(/\+/g, ' ');
       post.formattedCreatedAt = moment(post.createdAt).tz('Asia/Tokyo').format('YYYY年MM月DD日 HH時mm分ss秒');
     });
+    const oneTimeToken = crypto.randomBytes(8).toString('hex');
+    oneTimeTokenMap.set(req.user, oneTimeToken);
     res.end(pug.renderFile('./views/posts.pug', {
       posts: posts,
-      user: req.user
+      user: req.user,
+      oneTimeToken: oneTimeToken
     }));
     console.info(
       `閲覧されました: user: ${req.user}, ` +
```

↑ は、HTTP の GET メソッドで、フォームをテンプレートで表示させる部分に記述する（`case 'GET':` の中）。  
`const oneTimeToken = crypto.randomBytes(8).toString('hex');` は、`crypto` モジュールの `randomBytes` を利用して、推測されにくいランダムなバイト列を作成し、それを 16 進数にした文字列を取得している。これがトークンになる。

その後、ユーザー名をキーとして連想配列（Map)にこのトークンを保存（set）する。
さらにテンプレート（pug）に `oneTimeToken` というプロパティ名で、トークン文字列を利用できるように渡している。

```diff
   case 'POST':
     let body = [];
   　req.on('data', (chunk) => {
       body.push(chunk);
     }).on('end', () => {
       body = Buffer.concat(body).toString();
       const decoded = decodeURIComponent(body);
-      const content = decoded.split('content=')[1];
-      console.info('投稿されました: ' + content);
-      Post.create({
-        content: content,
-        trackingCookie: trackingId,
-        postedBy: req.user
-      }).then(() => {
-        handleRedirectPosts(req, res);
-      });
+      const dataArray = decoded.split('&');
+      const content = dataArray[0] ? dataArray[0].split('content=')[1] : '';
+      const requestedOneTimeToken = dataArray[1] ? dataArray[1].split('oneTimeToken=')[1] : '';
+      if (oneTimeTokenMap.get(req.user) === requestedOneTimeToken) {
+        console.info('投稿されました: ' + content);
+        Post.create({
+          content: content,
+          trackingCookie: trackingId,
+          postedBy: req.user
+        }).then(() => {
+          oneTimeTokenMap.delete(req.user);
+          handleRedirectPosts(req, res);
+        });
+      } else {
+        util.handleBadRequest(req, res);
+      }
     });
     break;
   default:
```

以上は POST メソッドで投稿を受け付ける実装。  
decoded の中身は `content=投稿内容&oneTimeToken=トークン本体` のようになっている。  
受け取ったデータを `&` で分割した後、投稿内容とリクエストで送られたワンタイムトークンを取得している。また、投稿内容がない場合やワンタイムトークンがない場合には、空文字列を利用するようにしている。

`if (oneTimeTOkenMap.get(req.user) === requestedOneTimeToken)` の部分は、連想配列に格納されているワンタイムトークンと、クライアントからのリクエストに含まれるワンタイムトークンを照合して、それらが一致する場合のみ投稿を DB に保存する処理を行っている。

投稿が成功した後に、連想配列 `oneTimeTokenMap` の `delete` 関数を利用して、利用済みのトークンを連想配列から除去する。

また、トークンが正しくない場合には 400 - Bad Request を返す。
</details>

次に、このワンタイムトークンをフォームの中に hidden タイプの input 要素として埋め込む。

<details close>
<summary>views/posts.pug を以下のように変更する。</summary>

```diff
     form(method="post" action="/posts")
       div.form-group
         textarea(name="content" rows="4").form-control
+        input(type="hidden" name="oneTimeToken" value=oneTimeToken)
       div.form-group
         button(type="submit").btn.btn-info.float-right 投稿
       div.row
```
</details>

これでワンタイムトークンが埋め込まれる。

また、`lib/hander-util.js` の `handleBadRequest` 関数内で、クライアントにレスポンスする文言も以下のとおり修正する。

```diff
-   res.end('未対応のメソッドです');
+   res.end('未対応のリクエストです');
```

以上で、CSRF 脆弱性の対策は完了。  
`node index.js` で再度、秘密の匿名掲示板のサーバを起動してログイン。

そして、Chrome で新規投稿の下にあるテキストエリアで右クリックし「検証」メニューを選択すると

```html
<textarea name="content" rows="4" class="form-control"></textarea>
<input type="hidden" name="oneTimeToken" value="4e48f39ffea3b1ef">
```

のように書かれた要素がデベロッパーツールの Elements タブに表示されるはず。ちゃんとワンタイムトークンが input 要素に含まれていれば成功。

実際に投稿できることを確かめておこう。そして、投稿が終わるとワンタイムトークンの値も変わっているはず（Elements タブを再度確認するとわかる）。  
GET リクエストがあったらワンタイムトークンを生成するロジックなので、投稿しなくてもブラウザをリロードすればそのたびにワンタイムトークンは新しくなる。  
また、当然ながら、デベロッパーツールでワンタイムトークンの値を手打ちで別の値に書き換えたりするともちろん投稿できない（「未対応のリクエストです」と表示される）。

ここで、最初に作った `csrf-study` フォルダの `index.html` を再度 Chrome で開き、「ボタンを押してみてください」ボタンを押してみる。そうすると「未対応のリクエストです」と表示されて、「まんまと罠にかかりましたね」という内容が投稿されなければ成功である。


#### おまけ
現在の実装では、新規投稿の CSRF 対策はできたが、削除処理の CSRF 対応ができていないのでそこも潰しておく。

以下のようにすればよい。

<details close>
<summary>
lib/posts-handler.js
</summary>

```diff
 function handleDelete(req, res) {
   case 'POST':
     let body = [];
     req.on('data', (chunk) => {
       body.push(chunk);
     }).on('end', () => {
       body = Buffer.concat(body).toString();
       const decoded = decodeURIComponent(body);
-      const id = decoded.split('id=')[1];
-      Post.findByPk(id).then((post) => {
-        if (req.user === post.postedBy || req.user === 'admin') {
-          post.destroy().then(() => {
-            console.info(
-              `削除されました: user: ${req.user}, ` +
-              `remoteAddress: ${req.connection.remoteAddress}, ` +
-              `userAgent: ${req.headers['user-agent']} `
-            );
-            handleRedirectPosts(req, res);
-          });
-        }
-      });
+      const dataArray = decoded.split('&');
+      const id = dataArray[0] ? dataArray[0].split('id=')[1] : '';
+      const requestedOneTimeToken = dataArray[1] ? dataArray[1].split('oneTimeToken=')[1] : '';
+      if (oneTimeTokenMap.get(req.user) === requestedOneTimeToken) {
+        Post.findByPk(id).then((post) => {
+          if (req.user === post.postedBy || req.user === 'admin') {
+            post.destroy().then(() => {
+              console.info(
+                `削除されました: user: ${req.user}, ` +
+                `remoteAddress: ${req.connection.remoteAddress}, ` +
+                `userAgent: ${req.headers['user-agent']} `
+              );
+              oneTimeTokenMap.delete(req.user);
+              handleRedirectPosts(req, res);
+            });
+          } 
+        });
+      } else {
+        util.handleBadRequest(req, res);
+      }
     });
     break;
   default:
```

ワンタイムトークンは、投稿用に使ったトークンを使いまわしている（投稿用と削除用のワンタイムトークンが同じだからといって、（分けたところでトークン生成ロジックは同じようなものを使うんだし）それで弱くなることはない）。

ページに GET 要求があった時点でトークンが発行され、連想配列（`oneTimeTokenMap`）への set もなされているのだから、削除の認可においては、（新たにユーザーごとにトークンを生成して連想配列に登録するみたいなことはせずに）ただちに `if (oneTimeTokenMap.get(req.user) === requestedOneTimeToken) ` で条件判定してよい。
</details>

<details close>
<summary>views/posts.pug</summary>

posts.pug に以下のように追記して、削除フォームにもワンタイムトークンを含めるようにする。

```diff
 if isDeletable
   form(method="post" action="/posts?delete=1")
     input(type="hidden" name="id" value=post.id)
+    input(type="hidden" name="oneTimeToken" value=oneTimeToken)
     button(type="submit").btn.btn-danger.float-right 削除
```
</details>

なお、攻撃をテストするページは、index.html を以下のように編集する。

<details close>
<summary>
index.html
</summary>

```html
<!DOCTYPE html>
<html lang="ja">

<head>
  <meta charset="UTF-8">
  <title>偽物の削除フォーム</title>
</head>

<body>
<h1>ブラウザの挙動テスト</h1>
<form method="post" action="http://localhost:8000/posts?delete=1">
  <input type="hidden" name="id" value="1"></input>
  <button type="submit">ボタンを押してみてください</button>
</form>

</body>

</html>
```

削除に使う投稿 ID は、自分で行った投稿の ID を入力するようにする。
上のコードでは、`value="1"` と、 1 が指定されている。
</details>

以上で、大きな脆弱性への対応はすべて完了。
やっとこれで Web サービスをインターネット上に公開することができる。

## Heroku への安全な公開
ここまでで、秘密の匿名掲示板で対策した脆弱性は以下の 4 つ。

- XSS 脆弱性
- パスワード管理の脆弱性
- セッション固定化攻撃脆弱性
- CSRF 脆弱性

現状の対策でも十分に安全性があると言えるが、とはいえ、今回まったく対応しなかったものもある。例えば以下のようなものがある。

|脆弱性|影響|内容|
|--|--|--|
|DoS 攻撃脆弱性|大|特定のコンピュータからの大量の接続要求で停止してしまう|
|ディレクトリ・とらバー申脆弱性|大|任意のファイルを閲覧、操作できてしまう|
|ヘッダインジェクション脆弱性|大|偽ページの表示などができてしまう|
|クリックジャッキング脆弱性|小|利用者の意図しないクリックを誘発する|

ここまでで紹介しなかった部分にも様々な脆弱性が存在する可能性があり、また、ライブラリやプログラミング言語、プラットフォームなどに脆弱性が発見されることもある。Web サービスを公開するということは、日々変化するセキュリティ事情につねに対応していくものなのである。

### 秘密の匿名掲示板を Heroku に公開しよう
今まで書いてきたコードを、Heroku で動かせるように修正していく。  
まずは、Heroku で動かすために必要なファイル `Procfile` を作成する。

```bash
echo "web: node index.js" > Procfile
```

こうして作った Procfile は、Web アプリケーションの起動コマンドが記述してあるファイルである。

次に `app.json` という、Heroku 上でのアプリケーションの情報を記述した設定ファイルを作成する。以下の内容を記述すること。

**app.json**

```json
{
  "name": "secret-board",
  "description": "秘密の匿名掲示板",
  "repository": "https://github.com/progedu/secret-board-3033",
  "logo": "",
  "keywords": ["node"],
  "image": "heroku/nodejs"
}
```

次に、`package.json` を以下のように編集する（これはただ、Node.js のバージョンを指定しているだけ）。

```diff
     "pg": "^7.4.1",
     "pg-hstore": "^2.3.2",
     "sequelize": "^4.33.4"
+  },
+  "engines": {
+    "node": "~10"
   }
 }
```

他に設定すべきことは、WEb サーバーの起動時のポートとデータベースの URL。

まずはポートの設定。`index.js` を以下の変更差分のように編集する。

```diff
-const port = 8000;
+const port = process.env.PORT || 8000;
 server.listen(port, () => {
   console.info('Listening on ' + port);
 });
```

開発時は 8000 番ポートだったが、Heroku で起動する際は Heroku の環境変数が指定するポートを利用する。

次にデータベースの URL。  
`lib/post.js` を以下のように編集する。

```diff
 const Sequelize = require('sequelize');
 const sequelize = new Sequelize(
-  'postgres://postgres:postgres@localhost/secret_board',
+  process.env.DATABASE_URL || 'postgres://postgres:postgres@localhost/secret_board',
  {
      logging: false,
      operatorsAliases: false
  });
```

これも、データベースを heroku が用意した（= Herokuの環境変数が指定する）URL を利用する設定としている。

ここまで完了したら Heroku にデプロイするために Git リポジトリにコミットする。

```bash
git add .
# 以下で、「インデックスに含まれ、これからコミットされる予定のデータ」と「現在の repo にコミットされているデータ」との差分を表示できる
# git diff --chached
git commit -m "Herokuで実行させるための対応"
```

ここからは、Heroku コマンドでサーバやデータベースの作成を行う。

```bash
heroku login -i
```

以上のコマンドの後にメールアドレスとパスワードを入力して Heroku にログインする。

```bash
heroku create
```

次は以上のコマンドで、Heroku のサーバを作成する。  

```
Creating app... done, xxxxx-xxxxx-XXXXX
https://xxxxx-xxxxx-XXXXX.herokuapp.com/ | https://git.heroku.com/xxxxx-xxxxx-XXXXX.git
```

作成が完了すると、最後にこのように作成したサーバーの URL が表示されるのでそれを控えておくこと。

次は、Heroku で利用する PostgreSQL のデータベースを作成する。

```bash
heroku addons:create heroku-postgresql:hobby-dev
```

以上のコマンドで、Heroku 上に開発用の簡易的な PostgreSQL のデータベースを作成できる。

```
Creating heroku-postgresql:hobby-dev on xxxxx-xxxxx-XXXXX... free
Database has been created and is available
 ! This database is empty. If upgrading, you can transfer
 ! data from another database with pg:copy
Created postgresql-xxxxx-XXXXX as DATABASE_URL
Use heroku addons:docs heroku-postgresql to view documentation
```

このように表示されれば DB の作成は完了となる。

それでは、git push コマンドで Heroku にデプロイしてみよう。

```bash
# master-2019 がローカルの branch 名（という想定）で、master がリモート（Heroku 側）の branch 名
git push heroku master-2019:master
```

以上を実行して、最後の行で、

```
remote: Verifying deploy... done.
```

と表示されればデプロイ成功！

最後に一つだけ対策を追加する。  
Heroku を使うと、Node.js の Web アプリケーションで HTTPS の暗号化通信も行えるのだが、今の実装だとただの HTTP の、平文での通信も行えてしまう。

先ほどメモした URL に `/posts` を足し、`https` を `http` にしたもの、つまり以下のような URL にアクセスし、`http` でもアクセスできてしまうことを確かめよう。

```
http://xxxxx-xxxxx-XXXXX.herokuapp.com/posts
```

この秘密の掲示板で利用している Basic 認証には、暗号化されていない HTTP 通信では、通信経路からユーザー名とパスワードが流出してしまうおそれがあるのだった。

そのようなトラブルを起こさぬよう、HTTP でのアクセスを禁止してしまおう。

`lib/router.js` を以下の変更差分のように変更する。

<details close>
<summary>lib/router.js</summary>

```diff
 const postsHandler = require('./posts-handler');
 const util = require('./handler-util');

 function route(req, res) {
+  if (process.env.DATABASE_URL
+    && req.headers['x-forwarded-proto'] === 'http') {
+    util.handleNotFound(req, res);
+  }
   switch (req.url) {
     case '/posts':
       postsHandler.handle(req, res);
```

追加した if 文の条件は、`process.env.DATABASE_URL` という環境変数がある場合、つまり本番の DB が存在している Heroku 環境であり、かつ、`x-forwarderd-proto` というヘッダの値が `http` であるときのみ true となる条件になっている。

`x-forwarded-proto` ヘッダには、Heroku が Node.js のアプリケーションに対して内部的にリクエストを受け渡す際にアクセスで利用されたプロトコルが含まれている。

この値を使うことで、HTTP なのか HTTPS なのかを判定できるのである。

この条件判定が true の場合（つまり、本番環境で HTTP によりアクセスしてきた場合）はページが見つからないという 404 を返すようにしている。

</details>

無事コードが変更できたら、

```bash
git commit -am "HTTP のアクセスを禁止にする"
```

以上で、インデックスに加えると同時にコミットしてしまおう。  
コミットが完了したら、Heroku に再度デプロイを行う。

```bash
# master-2019 はローカル側のブランチ（という想定）、master はリモート（Heroku側）のブランチ
git push heroku master-2019:master
```

そうして再び `http://〜〜〜.herokuapp.com/posts` （https ではなく http）にアクセスして、「ページが見つかりません」と返ってくれば HTTP でのアクセスを禁止できたとわかる。

なお、この実装ではページは表示できないが、Basic 認証自体は実行できてしまう。アクセスできないため HTTP の URL が使い続けられることはないと思われるが、とはいえ最初から HTTPS のプロトコルの URL だけを配布する必要はあるので注意されたい。

以上をもって、秘密の匿名掲示板の開発、および脆弱性対策が完了し、インターネットに公開できた！