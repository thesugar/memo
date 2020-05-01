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

