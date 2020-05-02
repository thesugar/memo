# サーバサイドプログラミング入門
Source: [サーバサイドプログラミング入門](https://www.nnn.ed.nico/courses/497/chapters/6890)

## Node.js
この学習資料での Node.js のインストール手順  
- ubuntu の Docker で実施
- nvm という Node.js のバージョンマネージャー導入
    - `curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash`
    - `source ~/.bashrc`
    - `nvm` と打ち込んで、使い方が表示されればインストール完了
- nvm を使って Node.js をインストール
    - `nvm install v10.14.2`
    - `nvm use v10.14.2`
        - `Now using node v10.14.2 (npm v6.4.1)` と表示されればインストール完了
- Node.js を使ってみる
    - `node` とだけ入力すると `>` が表示され、コンソールが入力を受け付ける状態になる。これは **REPL** (レプル；Read-Eval-Print Loop) というもの。
- プロジェクトを作って、そこに JavaScript ファイルを作成すれば `node app.js <some_arg>` というように実行できる。

## アルゴリズムの改善
実行時間を測定するには time コマンドを使う。

```bash
time node app.js
```
これを実行すると、以下のように結果が表示される。

```
real	0m2.029s
user	0m2.040s
sys     0m0.023s
```
これは実行にかかった、実際の時間（real）、今の実行ユーザーとしてかかった時間（user）、システムが別のことに使った時間（sys）を表示している。

フィボナッチ数列（fib(n) = fib(n-1) + fib(n-2) ただし fib(0) = 0, fib(1) = 1）の計算は、以下のように増加していく。
- n が２の時の fib(2) は、fib(1) + fib(0)、つまり、0 + 0 + 1で 1回
- n が３の時の fib(3) は、fib(2) + fib(1)、これは、1 + 0 + 1 で 2回
- n が４の時の fib(4) は、fib(3) + fib(2)、これは、2 + 1 + 1 で 4回
- n が５の時の fib(5) は、fib(4) + fib(3)、これは、4 + 2 + 1 で 7回
- n が６の時の fib(6) は、fib(5) + fib(4)、これは、7 + 4 + 1 で 12回
- n が７の時の fib(7) は、fib(6) + fib(5)、これは、12 + 7 + 1 で 20回
- n が８の時の fib(8) は、fib(7) + fib(6)、これは、20 + 12 + 1 で 33回
- n が50の時の fib(50) は、fib(49) + fib(48)、これは、12,586,269,024 + 7,778,742,048 + 1 で 20,365,011,073回
- n が100の時の fib(100) は、fib(99) + fib(98)、これは、354,224,848,179,261,915,074 + 218,922,995,834,555,169,025 + 1 で 573,147,844,013,817,084,100回

このように、回を重ねると後に倍々に計算回数が増えるような処理の増え方を **指数オーダー** といい、 *O(2^n)* と表す。  
  
### 👍プロファイルツールの使い方
処理に時間がかかっている様子やどれくらいメモリを使っているのかを調べる方法に **プロファイル** と呼ばれる方法がある。

```bash
node --prof app.js
```

以上を実行することで、`isolate-0x395f810-v8.log` というような名前のログファイルにプロファイルの様子が出力される。このログファイルは、次のコマンドでわかりやすく表示させることができる。

```bash
node --prof-process isolate-0x395f810-v8.log 
```

こうすると、プロファイル結果が表示される（かなり長い）。この中の `[Summary]` という項目に、プログラムの処理全体の概要が示されている。

```
 [Summary]:
   ticks  total  nonlib   name
   1575   89.6%   94.1%  JavaScript
      0    0.0%    0.0%  C++
      3    0.2%    0.2%  GC
     84    4.8%          Shared libraries
     98    5.6%          Unaccounted
```

また、`[Bottom up (heavy) profile]` という項目に、時間のかかった処理が並んでいる。

```
 [Bottom up (heavy) profile]:
  Note: percentage shows a share of a particular caller in the total
  amount of its parent calls.
  Callers occupying less than 1.0% are not shown.

   ticks parent  name
   1573   89.5%  LazyCompile: *fib /workspace/fibonacci/app.js:3:13
   1573  100.0%    LazyCompile: *fib /workspace/fibonacci/app.js:3:13
   1573  100.0%      LazyCompile: *fib /workspace/fibonacci/app.js:3:13
   1573  100.0%        LazyCompile: *fib /workspace/fibonacci/app.js:3:13
   1573  100.0%          LazyCompile: *fib /workspace/fibonacci/app.js:3:13
   1573  100.0%            LazyCompile: *fib /workspace/fibonacci/app.js:3:13
```

一番左の ticks という列は、各処理にかかったイベントループ数を示している。これらをみると、fib という関数の呼び出しにほとんどの時間がかかっていることがわかる。

### メモ化を使ったアルゴリズムの改善
Map を用いて以下のようにフィボナッチ関数を実装する。

```js
'use strict';

const memo = new Map();
memo.set(0, 0);
memo.set(1, 1);

const fib = n => {
  if (memo.has(n)) {
    return memo.get(n);
  } 
  const value = fib(n-1) + fib(n-2);
  memo.set(n, value);
  return value;
}

const length = 40;
for (let i=0; i <= length; i++) {
  console.log(fib(i));
} 
```

`time node app.js` を実行すると、

```
real	0m0.079s
user	0m0.058s
sys     0m0.021s
```

このように大幅に速くなったことがわかる。プロファイルも実行してみる。（`node --prof app.js`→`node --prof-process isolate-xxx.log`）

出力されたプロファイルの `[Summary]` を見てみると以下のようになり、total の数値をみると、JavaScript の処理にかかる時間が大幅に減ったことがわかる。

```
 [Summary]:
   ticks  total  nonlib   name
      2    2.2%  100.0%  JavaScript
      0    0.0%    0.0%  C++
      4    4.5%  200.0%  GC
     87   97.8%          Shared libraries
```

`[Bottom up (heavy) profile]` の項目をみると、最も時間のかかっている処理がもはや fib 関数の実行ではないことがわかる。

```
 [Bottom up (heavy) profile]:
  Note: percentage shows a share of a particular caller in the total
  amount of its parent calls.
  Callers occupying less than 1.0% are not shown.

   ticks parent  name
     36   40.4%  /root/.nvm/versions/node/v10.14.2/bin/node
     34   94.4%    /root/.nvm/versions/node/v10.14.2/bin/node
     20   58.8%      LazyCompile: ~NativeModule.compile internal/bootstrap/loaders.js:302:44
     20  100.0%        LazyCompile: ~NativeModule.require internal/bootstrap/loaders.js:148:34
      4   20.0%          Script: ~<anonymous> util.js:1:11
```

このメモ化を使ったアルゴリズムの改善がされたこちらの fib 関数のオーダーは、*O(n)* となる。n に対して n 倍の時間をかければ問題を解くことができるというこのようなオーダーを線形オーダーという。

## 集計処理を行うプログラム

**ここで用いる csv ファイルは [こちら](https://github.com/progedu/adding-up/blob/master/popu-pref.csv)**

```js
const fs = require('fs');
const readline = require('readline')
```

この部分は、Node.js に用意された **モジュール** を呼び出している。

`fs` は FileSystem の略で、ファイルを扱うためのモジュール。
`readline` は、ファイルを一行ずつ読み込むためのモジュー売る。

```js
const rs = fs.createReadStream('./popu-pref.csv');
const rl = readline.createInterface({ 'input': rs, 'output' : {} });
```

以上の部分は、csv ファイルかrあ、ファイルの読み込みを行う Stream を作成し、さらにそれを `readline` オブジェクトの input として設定し、`rl` オブジェクトを作成している。

さて、ここで突然出てきた Stream とはなんなのか？

### Stream
Node.js では、入出力が発生する処理をほとんど **Stream** という形で扱う。Stream とは、非同期で情報を取り扱うための概念で、情報自体ではなく情報の流れに注目する。

Node.js で Stream を扱う際は、Stream に対してイベントを監視し、イベントが発生したときに呼び出される関数を設定することによって、情報を利用する。

このように、あらかじめイベントが発生したときに実行される関数を設定しておいて、起こったイベントに応じて処理を行うことを **イベント駆動型プログラミング** と呼ぶ。

```js
const rl = readline.createInterface({ 'input': rs, 'output': {} })
```

ここで作成された `rl` というオブジェクトも Stream のインターフェースを持っている。利用する際には

```js
rl.on('line', (lineString) => {
  console.log(lineString);
});
```

のようなコードになる。このコードは、`rl` オブジェクトで `line` というイベントが発生したらこの無名関数を呼んでください、という意味。なお、`lineString` には、読み込んだ 1 行の文字列が入っている。

### ファイルからデータを抜き出す
2010 年と 2015 年のデータから「集計年」「都道府県」「15〜19歳の人口」を抜き出す、という処理を実装したい。

```js
rl.on('line', (lineString) => {
  const columns = lineString.split(',');
  const year = parseInt(columns[0]);
  const prefecture = columns[1];
  const popu = parseInt(columns[3]);

  if (year === 2010 | year === 2015) {
    console.log(year);
    console.log(prefecture);
    console.log(popu);
  }
});
```

### グルーピングを行う

```js
'use strict';

const fs = require('fs');
const readline = require('readline');
const rs = fs.createReadStream('./popu-pref.csv');
const rl = readline.createInterface({ 'input' : rs, 'output': {} });
const prefectureDataMap = new Map(); // key: 都道府県 value: 集計データのオブジェクト
rl.on('line', (lineString) => {
  const columns = lineString.split(',');
  const year = parseInt(columns[0]);
  const prefecture = columns[1];
  const popu = parseInt(columns[3]);
  if (year === 2010 | year === 2015) {
    // 連想配列 `prefectureDataMap` からデータを取得する。
    // value の値が falsy の場合に、value に初期値となるオブジェクトを代入する。その県のデータを処理するのがはじめてであれば、value の値は undefined となるのでこの条件を満たし、value に値が代入される
    let value = prefectureDataMap.get(prefecture);
    if (!value) {
      value = {
        popu10: 0,
        popu15: 0,
        change: null,
      };
    }
    
    if (year === 2010) {
      value.popu10 = popu;
    }
    
    if (value === 2015) {
      value.popu15 = popu;
    }
    
    prefectureDataMap.set(prefecture, value);
  }
});

// close イベントは、全ての行を読み込み終わったあとに呼び出される。
// また、人口の変化率（change）の計算は、その県のデータが揃ったあとでしか正しく行えないので、close イベントの中へ実装する
rl.on('close', () => {
  for (let [key, value] of prefectureDataMap) {
    value.change = value.popu15 / value.popu10;
  }
  console.log(prefectureDataMap);
});
```

**`for-of` 構文について**
Map や Array の中身を `of` の前に与えられた変数に代入して for ループと同じことができる。配列に含まれる要素を使いたいだけで、添字は不要な場合に便利。

また、Map に for-of を使うと、キーと値で要素が 2 つある配列が前に与えられた変数に代入される。ここでは **分割代入** という方法を使っている。

### データの並び替え
得られた結果を、変化率ごとに並べ替えてみる。

```js
rl.on('close', () => {
  for (let [key, value] of prefectureDataMap) { 
    value.change = value.popu15 / value.popu10;
  }
  const rankingArray = Array.from(prefectureDataMap).sort((pair1, pair2) => {
    return pair2[1].change - pair1[1].change;
  });
  console.log(rankingArray);
});
```

まず、`Array.from(prefectureDataMap)` の部分で、連想配列を普通の配列に変換する処理を行っている。さらに、Array の `sort` 関数を呼んで、無名関数を渡している。`sort` に対して渡すこの関数は *比較関数* と言い、これによって並び替えのルールを決めることができる。

比較関数は 2 つの引数を受け取って、前者の引数を pair1 を後者の引数 pair2 より前にしたいときは負の整数を、pair2 を pair1 より前にしたいときは正の整数を、pair1 と pair2 の並びをそのままにしたいときは 0 を返す必要がある。

ここでは変化率の降順に並び替え行いたいので、pair2 が pair1 より大きかった場合、pair2 を pair1 より前にする必要がある。

つまり、pair2 が pair1 より大きいときに正の整数を返すような処理を書けばよいので、ここでは pair2 の変化率のプロパティから pair1 の変化率のプロパティを引き算した値を返している。

詳細は [MDN の sort 関数のドキュメント](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)。

（ちなみに、ここでの pair1, pair2 は（`Array.from(prefectureDataMap)` を行ったものであり、） `['hoge 県', { popu10: ..., popu15: ..., change: ...}]` という形をしているので、`pair1[1].change`  などと指定している）

> `Array.from()` について：Array.from() メソッドを用いれば、配列に似た形のもの（ここでは Map）を普通の配列に変換することができる。ここでは、`prefectureDataMap` は、各都道府県名が key で、各集計データオブジェクトが value の Map であったから、`Array.from(prefectureDataMap)` とすると `[['hoge 県', { popu10: ..., popu15: ..., change: ...}],[...],...,[...]]`という形の配列になる。

最後に、`map` 関数を使って、きれいに整形して出力してみる。

```js
rl.on('close', () => {
  for (let [key, value] of prefectureDataMap) {
    value.change = value.popu15 / value.popu10;
  }
  const rankingArray = Array.from(prefectureDataMap).sort((pair1, pair2) => {
    return pair2[1].change - pair1[1].change;
  });
  const rankingStrings = rankingArray.map(([key, value]) => {
    return key + ': ' + value.popu10 + '=>' + value.popu15 + ' 変化率： ' + value.change;
  });
  console.log(rankingStrings);
});
```

## ライブラリ
### 環境変数
環境変数は特別なシェル変数。通常のシェル変数は現在実行しているシェルからしか参照できないが、環境変数はシェルで実行したプロセスからも参照できる。Ubuntu では、デフォルトでユーザー名が代入されている `USER` や、ユーザーのホームディレクトリの場所が代入されている `HOME` などがある。

Ubuntu で、現在のシェルで定義されている環境変数一覧をみるには `printenv` コマンドを使う。

個々の環境変数の値を表示させるときは `echo` コマンドを使う。

```bash
$ echo $HOME
/root
```

`PATH` という環境変数は、実行したいプログラムがある場所を列挙しておくもの。`PATH` に指定された場所にあるプログラムは、そのファイル名を入力するだけで実行できるようになり、絶対パスや相対パスで直接ファイルを指定する必要はない。

インストールが完了したら、

```bash
source ~/.bashrc
```

と入力することで、設定を再読込できる。

### npm パッケージを作ってみる
たとえば、整数を足しあわせたりする sum というライブラリを作成してみる。

```bash
cd ~/workspace
mkdir sum
cd sum
yarn init
```

`yarn init` というコマンドで、npm パッケージを作成するためのチュートリアルを起動する。インストラクションにしたがって Enter を押していく。`entry point` というのは、ライブラリとして読み込まれる JavaScript ファイル名のこと（`index.js` でよい）。

`ISC` ライセンスとは、ソフトウェアを使用、コピー、改変、配布する許可を与えるライセンス。
`private` の設定をすると、公開はしないことになる。

この状態で、index.js を作成する。

```js
`use strict`
function add(numbers) {
  let result = 0;
  for (let num of numbers) {
    result = result + num;
  }
  return result;
}
module.exports = {
  add // add: add と書くのと同じ
};
```

**`module.exports = { ... }` という記法について**  
特定の関数をモジュールとして公開する場合に、`module.exports` オブジェクトのプロパティとして関数を登録する。こうすることで、この場合ではこの sum パッケージに add メソッドが追加される。

### 作成したパッケージを使用する
今回は npm に公開はせず、ローカルの別の場所でこのパッケージを使う。まずは、このパッケージを使うアプリケーションを作成してみる。

```bash
cd ~/workspace
mkdir sum-app
cd sum-app
```

先ほどと同様に `yarn init` と入力してパッケージ作成のチュートリアルを行う。そして、先ほど開発した sum パッケージをインストールする。

```bash
yarn add ../sum
```

次に、sum-app フォルダを開いて、app.js を以下のように編集する。

```js
'use strict'
const s = require('sum');
console.log(s.add([1, 2, 3, 4]));
```

## モジュール化された処理
CRUD を考え、以下の機能を持つモジュールを作成する。

- todo: タスクの作成
- done: タスクの状態の更新（「完了」状態にする）
- list: タスクの状態が未完了のものの読み込み
- donelist: タスクの状態が完了のものの読み込み
- del: タスクの削除

今回はテストコードも書く。`package.json` に以下を追記。

```json
"scripts": {
  "test": "node test.js"
}, ...
```

今回のすべてのデータは、連想配列 Map の key にタスクの文字列を、value に完了しているかどうかの真偽値を入れることによって表現する。

index.js
```js
'use strict';

// key: タスクの文字列 value: 完了しているかどうかの真偽値
const tasks = new Map();

/**
 * TODO を追加する
 * @param {string} task
 */
const todo = task => {
    tasks.set(task, false);
}

/**
 * タスクと完了したかどうかが含まれる配列を受け取り、完了したかを返す
 * @param {array} taskAndIsDonePair
 * @return {boolean} 完了したかどうか
 */
const isDone = taskAndIsDonePair => {
    return taskAndIsDonePair[1];
}

/**
 * タスクと完了したかどうかが含まれる配列を受け取り、完了していないか返す
 * @param {array} taskAndIsDonePair
 * @return {boolean} 完了していないかどうか
 */
const isNotDone = taskAndIsDonePair => {
    return !isDone(taskAndIsDonePair);
}

/**
 * TODO の一覧の配列を取得する
 * @return {array}
 */
const list = () => {
    return Array.from(tasks) // Map を、キーと値で構成される要素 2 つの配列に変換する
        .filter(isNotDone)
        .map(t => t[0]);
}

/**
 * TODO を完了状態にする
 * @param {string} task
 */
const done = task => {
    if (tasks.has(task)) {
        tasks.set(task, true);
    }
}

/**
 * 完了済みのタスクの一覧の配列を取得する
 * @return {array}
 */
const donelist = () => {
    return Array.from(tasks)
                .filter(isDone)
                .map(t => t[0]);
}

/**
 * 項目を削除する
 * @param {string} task
 */
const del = task => {
    tasks.delete(task);
}

// todo 関数 と list 関数をモジュールの関数として追加する
module.exports = { todo, list, done, donelist, del };
```

上のコードにおいて、`Array.from` 関数は、連想配列の Map をキーと値で構成される要素 2 つの配列に変換する。`[['鉛筆を買う', false], ['りんごを買う', true], ...]` のような感じになる。

`list` 関数内部で行われていることを例示すると以下のようなイメージ。

```js
Array.from(Map { '鉛筆を買う' => false, 'ノートを買う' => false, '勉強をする' => true })
// [ [ '鉛筆を買う', false ], [ 'ノートを買う', false ], [ '勉強をする', true ] ]
[ [ '鉛筆を買う', false ], [ 'ノートを買う', false ], [ '勉強をする', true ] ].filter(isNotDone)
// [ [ '鉛筆を買う', false ], [ 'ノートを買う', false ] ]
[ [ '鉛筆を買う', false ], [ 'ノートを買う', false ] ].map(t => t[0])
// [ '鉛筆を買う', 'ノートを買う' ]
```

テストコードは以下のようになる。

```js
'use strict';

const todo = require('./index.js');
const assert = require('assert');

// todo と list のテスト
todo.todo('ノートを買う');
todo.todo('鉛筆を買う');
assert.deepEqual(todo.list(), ['ノートを買う', '鉛筆を買う']);

// done と donelist のテスト
todo.done('鉛筆を買う');
assert.deepEqual(todo.list(), ['ノートを買う']);
assert.deepEqual(todo.donelist(), ['鉛筆を買う']);

// del のテスト
todo.del('ノートを買う');
todo.del('鉛筆を買う');
assert.deepEqual(todo.list(), []);
assert.deepEqual(todo.donelist(), []);

console.log('🎉テストが正常に完了しました😙');
```

これで `yarn test` を実行して、「テストが正常に完了しました」と表示されれば成功。

```js
const assert = require('assert');
```
上記は、Node.js でテストをするためのモジュール `assert` を呼び出している。

```js
assert.deepEqual(todo.list(), ['ノートを買う', '鉛筆を買う']);
```

`assert.deepEqual` は、与えられたオブジェクトや配列の中身まで比較してくれる `equal` 関数。もう少し詳しくは以下を参照。

```js
assert.equal(1, 1); // ✅
assert.equal([1], [1]) // 🛑 AssertionError
// ↑は、JavaScript では配列やオブジェクトを == 演算子で比較した場合、
// 同じオブジェクト自身でないと false になるため

assert.deepEqual([1], [1]); // ✅
assert.deepEqual({p: 1}, {p: 1}); // ✅
```

## ボットインタフェースとの連携
Hubot + CoffeeScript で Slack bot を作るレッスンは省略。
その中で出てきた一部を以下で学習。

### 文字列操作
#### 正規表現
```js
/todo (.+)/i
```

この表現は、todo で始まり、その後に 1 つスペースを置いてなんらかの文字列が記述されているという正規表現。最後の / の後の i は、大文字でも小文字でもマッチするというオプション。`.+` の部分は、`.` が改行文字以外のどの 1 文字にもマッチする文字であり、`+` は直前の文字の 1 回以上の繰り返しにマッチするという意味。任意の文字の繰り返しなので、`aaa` や `あいうえお` もマッチする。

`(.+)` と、正規表現を `()` で囲んだものは、**グループ**といって正規表現のかたまりを表す表現。この `()` の中でマッチした内容を後でプログラムから取得することができる。→ `hoge.match[1]` とする。このとき、添字の 0 にはマッチした文字列全体が、添字の 1 には 1 番目の () でマッチした文字列が含まれる。

上記の `/todo (.+)/i` は、実用においては「todo 鉛筆を買う」のような使われ方を想定している。

詳細は [MDN の正規表現の資料](https://developer.mozilla.org/ja/docs/Web/JavaScript/Guide/Regular_Expressions)。

#### 文字列のメソッド
`trim` 関数は、文字列の関数。文字列の両端の空白を取り除いた文字列を取得する関数。

`join` 関数は、配列のすべての要素を与えられた文字列でつないで 1 つの文字列にする関数。`\n` という改行を表すエスケープシーケンスで結合するなど。

## 同期 I/O と非同期 I/O
今回は、`fs` モジュールを使用してファイルへデータを保存することで情報の永続化を行う。そのためにまず、Node.js がファイルを読み込んだり、書き込んだり、または通信をしたりする際の仕組みについて知る必要がある。

Node.js は他のプログラミング言語と少し違うところがある。それは**非同期 I/O** を使ったプログラミングがやりやすいという側面。

### 非同期 I/O
I/O とは入出力処理のこと。多くのプログラミング言語では I/O 処理の間、例えばディスクに書き込む待ち時間や通信の待ち時間は、プログラムは停止してその I/O 処理を待つ。このような I/O 処理のことを**同期 I/O** と呼ぶ。

一方、**非同期 I/O** は同期 I/O　とは異なり、I/O の開始処理をしても、その終了は待たない。I/O の待ち時間中にも別の処理を実施し、コンピューターのリソースをうまく活用する。

なお、この *I/O 処理の間プログラムが停止すること*を **ブロッキング** と呼ぶ。

ブロッキングの間にも CPU を効率よく利用するために、マルチプロセスにしたり、より軽量なプロセスであるスレッドを使いマルチスレッドにして解決することもある。

Node.js では、マルチプロセスやマルチスレッドではなく、*シングルスレッドでブロッキングしない非同期 I/O を利用* することで効率化を図っている。

### 非同期 I/O による実装

**app.js**
```js
'use strict';
const fs = require('fs');
const fileName = './test.txt';
for (let count = 0; count < 500; count++) {
  fs.appendFile(fileName, 'あ', 'utf8', () => {});
  fs.appendFile(fileName, 'い', 'utf8', () => {});
  fs.appendFile(fileName, 'う', 'utf8', () => {});
  fs.appendFile(fileName, 'え', 'utf8', () => {});
  fs.appendFile(fileName, 'お', 'utf8', () => {});
  fs.appendFile(fileName, '\n', 'utf8', () => {});
}
```

これは、500 回にわたり、`test.txt` というファイルに「あいうえお」と改行コードを書き込むということを行うコード。`appendFile` 関数は、ファイルに対して書き込みを行う関数で、ここでは utf8 の文字コードで書き込んでいる。`() => {}` の部分は、`appendFile` を行ったあとの処理をコールバック関数で指定している（非同期 I/O を利用する関数では、このコールバック関数を指定しないとエラーになる）が、今回は「何もしない」という意味の `() => {}` を渡している。

これを実行して `test.txt` を確認すると次のような感じになる。

```txt
えお
あいうえいお
あいえうお
あいうおえ
あいあうえ
おいうお
あええうお
あいうえお
あいうえお
あいうえお
いあいうえあお
いうえ
```

以上のように順番が乱れて表示される。*これが非同期 I/O である*。非同期 I/O は「I/O の処理がひとつ終わってから、次の I/O の処理を行う」ことを保証していない。そのため、このように順不同になる性質がある。その反面、CPU を効率よく利用することができる。

では、これを直すためにはどうすればよいか？

### 同期 I/O を使った実装
Node.js では、デフォルトでは非同期 I/O の関数が呼ばれるが、Sync という修飾子がついた同期 I/O の関数も提供されている。

**app.js**
```js
'use strict';
const fs = require('fs');
const fileName = './test.txt';
for (let count = 0; count < 500; count++) {
  fs.appendFileSync(fileName, 'あ', 'utf8');
  fs.appendFileSync(fileName, 'い', 'utf8');
  fs.appendFileSync(fileName, 'う', 'utf8');
  fs.appendFileSync(fileName, 'え', 'utf8');
  fs.appendFileSync(fileName, 'お', 'utf8');
  fs.appendFileSync(fileName, '\n', 'utf8');
}
```

すべての書き込み関数を `appendFileSync` に変更した。これで実行して test.txt を確認すると、今度は正しく以下のように表示される。

```txt
あいうえお
あいうえお
あいうえお
・・・
```

## 永続化処理と例外処理

[モジュール化された処理](##モジュール化された処理) の部分で実装した todo パッケージに、永続化処理と例外処理を追加する。

```js
'use strict';

// key: タスクの文字列 value: 完了しているかどうかの真偽値
// ここは、try 文の中で tasks に再代入する部分があるので const でなく let にしている
let tasks = new Map();

const fs = require('fs');
const fileName = './tasks.json';

// 同期的にファイルから復元
try {
    const data = fs.readFileSync(fileName, 'utf8');
    tasks = new Map(JSON.parse(data));
} catch (ignore) {
    console.log(fileName + 'から復元できませんでした');
}


/**
 * タスクをファイルに保存する
 */
const saveTasks = () => {
    fs.writeFileSync(fileName, JSON.stringify(Array.from(tasks)), 'utf8');
}

/**
 * TODO を追加する
 * @param {string} task
 */
const todo = task => {
    tasks.set(task, false);
    saveTasks();
}

/**
 * タスクと完了したかどうかが含まれる配列を受け取り、完了したかを返す
 * @param {array} taskAndIsDonePair
 * @return {boolean} 完了したかどうか
 */
const isDone = taskAndIsDonePair => {
    return taskAndIsDonePair[1];
}

/**
 * タスクと完了したかどうかが含まれる配列を受け取り、完了していないか返す
 * @param {array} taskAndIsDonePair
 * @return {boolean} 完了していないかどうか
 */
const isNotDone = taskAndIsDonePair => {
    return !isDone(taskAndIsDonePair);
}

/**
 * TODO の一覧の配列を取得する
 * @return {array}
 */
const list = () => {
    return Array.from(tasks) // Map を、キーと値で構成される要素 2 つの配列に変換する
        .filter(isNotDone)
        .map(t => t[0]);
}

/**
 * TODO を完了状態にする
 * @param {string} task
 */
const done = task => {
    if (tasks.has(task)) {
        tasks.set(task, true);
        saveTasks();
    }
}

/**
 * 完了済みのタスクの一覧の配列を取得する
 * @return {array}
 */
const donelist = () => {
    return Array.from(tasks)
                .filter(isDone)
                .map(t => t[0]);
}

/**
 * 項目を削除する
 * @param {string} task
 */
const del = task => {
    tasks.delete(task);
    saveTasks();
}

// todo 関数 と list 関数をモジュールの関数として追加する
module.exports = { todo, list, done, donelist, del };
```

上記のコードで、

```js
/**
 * タスクをファイルに保存する
 */
const saveTasks = () => {
    fs.writeFileSync(fileName, JSON.stringify(Array.from(tasks)), 'utf8');
}
```

`saveTasks` 関数はタスクをファイルに保存する関数であり、`tasks` という連想配列（Map）を `Array.from` で配列に変換したあと、さらに、`JSON.stringify` という関数で JSON の文字列に変換し、さらに同期的に（Sync）ファイルに書き出している。

この `saveTasks` 関数を、タスクを追加する todo 関数、タスクを完了にする done 関数、タスクを削除する del 関数、以上のそれぞれの処理のあとで呼び出している。

さて、以下の例外処理の部分が例外処理。`catch(ignore)` の ignore には、発生したエラーが渡される（ここでは後続の処理に使わないので ignore という変数名をあてている）

```js
// 同期的にファイルから復元
try {
    const data = fs.readFileSync(fileName, 'utf8');
    tasks = new Map(JSON.parse(data));
} catch (ignore) {
    console.log(fileName + 'から復元できませんでした');
}
```

また、処理としては、`fileName` という名前のファイルを読み込み、その中身を `JSON.parse` という関数で解釈（パース）して、連想配列の Map オブジェクトを作るときに渡している。

連想配列の Map は、引数にキーと値の 2 つの要素で与えられる配列の配列を渡すことで、その値が入った連想配列を作成できるため、ここではその機能を利用している（data には `[['鉛筆を買う', false], ['みかんを買う', true], ...]` のような内容が入っている）。

なお、テストコードにおいて、テストの際に存在する `tasks.json` を読み込んでしまいテストが失敗することがある（タスクを追加して、予想通りタスクが追加されているか deepEqual で比較する部分で、元々手動で追加していた他のタスクが存在するせいで不一致となり失敗）。
ファイルを削除してからテストを実行するようにするには、以下のように書く。

```js
const fs = require('fs');
fs.unlink('./tasks.json', (err) => {
  // テスト処理
});
```

上記のように `unlink` 関数に渡す無名関数（コールバック関数）の中で処理を行うことで、非同期処理でも順序を制御し、`tasks.json` ファイルが削除されたあとテストを実行することができる。

## HTTP サーバー
ここからは、Web サービスを作ることをテーマに学習する。まずは、HTTP サーバーを利用したプログラミングから。

### Web
Web の主な用途は
- Web サイト（例：Yahoo! のようなポータルサイトや Amazon のようなショッピングサイトなど）
- ユーザーインターフェースとしての Web（例：HTML で作られた、通信をしないヘルプなど）
- プログラム用 API としての Web（例：扱いやすいデータフォーマットで提供される HTTP の API など）

であると紹介されている。

現在では、Web サイト自体よりも、Web ビューなどのユーザーインターフェース技術や、スマートフォンなどのアプリで用いられる Web API の技術の存在感も大きくなっている。

### Web サービス
**Web サービス** ：HTTP などのインターネット関連技術を利用して通信を行うサービスのこと。  
HTML をクライアントとして提供するものや、HTTP による API を提供することで様々なデバイスから利用するものがある。  
つまり、Web サービスを作るためには HTTP でサービスを提供する必要があり、 HTTP サーバーが必要になる。

### Node.js で HTTP サーバーを作ってみる
プロジェクトを新たに作成して、以下の index.js を作成。

```js
'use strict';
const http = require('http'); // http モジュールを読み込む

/*
http モジュールの機能で、サーバーを作成している。
サーバーには、リクエストオブジェクト req とレスポンスオブジェクト res　を受け取る無名関数を渡す。
この無名関数は、サーバーにリクエストがあったときに呼び出される。
*/
const server = http.createServer((req, res) => {
    // リクエストがきたとき、200 という成功を示すステータスコードとともに、レスポンスヘッダを書き込んでいる。
    // `Content-Type`（内容の形式）が`text/plain`（通常のテキスト）であるという情報
    // `charset` が `utf-8` であるという情報、を書き出している。
    res.writeHead(200, {
        'Content-Type': 'text/plain; charset=utf-8'
    });

    // write 関数は HTTP のレスポンスの内容を書き出している。
    // ここでは、リクエストヘッダの UA の中身をレスポンスの内容として書き出している
    res.write(req.headers['user-agent']+'\n');

    // end メソッドを呼び出してレスポンスの書き出しを終了
    res.end();
});
const port = 8000; // この HTTP が起動するポートを宣言

// listen 関数でサーバーを起動し、起動した際に実行する関数を渡している。
// サーバーが立ち上がると、特定のポートからリクエストがないかをずっと聞き耳を立てるため listen という
server.listen(port, () => {
    console.log('Listening on ' + port);
});
```

以上を記述したら、`node index.js` を実行して HTTP サーバーを起動する。`Listening on 8000` とコンソール に表示されればサーバの起動は成功。

tmux で 2 つのウインドウを使って（もしくは VS Code でもう一つターミナルを追加して）curl でアクセスしてみる。

tmux の場合は `tmux` コマンドで tmux を実行したら `Ctrl + b → c` でウインドウを新たに追加する。

`curl http://localhost:8000` を実行すると、`curl/7.58.0` とコンソールに表示され、UA が curl であることがわかる。

この user-agent には、Web サービスを利用する際のソフトウェアおよびハードウェアを記述することになっているが、クライアントによる事故申告制のため偽装が可能。例えばコンソールで `curl -A "dummy" http://localhost:8000` と入力するとコンソールには `dummy` と表示される（`-A` は `user-agent` を指定するオプション）

（※もしくは、`curl localhost:8000 -H "User-Agent: dummy" `としてヘッダ（中の UA）を指定することもできる）

## ログ
今回は、サーバー側でなにかあった際にそれを記録する方法を学ぶ。そのような記録のことを **ログ** といい、ログをとることを **ロギング** という。

ログを出力するときは、`console.log` や `console.info`、`console.error` などを使う。処理としてはどれも `console.log` と同じだが、ログの意味づけ（**ログレベル**）が異なる。


|関数|ログの内容|出力|
|---|---|---|
|info|情報。普段から残しておきたい情報に使う|標準出力
|warn|警告。問題となる可能性がある情報に使う|エラー標準出力
|error|エラー。直ちに対応が必要な情報に使う|エラー標準出力

実際のコードは以下のようになる（上記（### Node.js で HTTP サーバーを作ってみる） のコードに追加するかたちになる）。

```js
'use strict';
const http = require('http');
const server = http.createServer((req, res) => {
  // 👀アクセスログを出力
  console.info('[' + new Date() + '] Requested by ' + req.connection.remoteAddress);
  res.writeHead(200, {
    'Content-Type': 'text/plain; charset=utf-8'
  });
  res.write(req.headers['user-agent']);
  res.end();
  // 😊エラーログを出力
}).on('error', (e) => {
  console.error('[' + new Date() + '] Server Error', e);
}).on('clientError', (e) => {
  console.error('[' + new Date() + '] Client Error', e);
});
const port = 8000;
server.listen(port, () => {
  console.log('Listening on ' + port);
});
```

**上記コードの解説**

```js
console.info('[' + new Date() + '] Requested by ' + req.connection.remoteAddress);
```

`req.connection.remoteAddress` は、リクエストが送られた IP 情報を出力している。

```js
}).on('error', (e) => {
  console.error('[' + new Date() + '] Server Error', e);
}).on('clientError', (e) => {
  console.error('[' + new Date() + '] Client Error', e);
```

これがエラーハンドリング部分。以前出てきた Stream と同様、HTTP サーバーがイベントを発行する存在として作られているためこのような記述になる。  

ここでは、'error' という文字列のイベントが発生した際には、サーバーエラーが発生したこととしてエラーログを表示し、'clientError' という文字列のイベントが発生した際には、クライアントエラーが発生したこととしてエラーログに出力する。

#### アクセスログについて解説
以上の記述が終わったら、`node index.js` として実際に動かしてみる。起動すると `Listening on 8000` と表示されるので、ポートフォワードをしている場合はブラウザでアクセス、していない場合は curl で叩いてみる。するとコンソールに `[Sun Apr 26 2020 08:45:00 GMT+0000 (Coordinated Universal Time)] Requested by ::ffff:127.0.0.1` のように表示される（この表示は curl でアクセスしたときのもの）。

（curl でなくて）ブラウザ（Chrome）からのアクセスの場合は、このようなログが 2 行出力される。なぜなら、Chrome が URL にアクセスすると、`/favicon.ico` というパスに、アイコン画像がないか確認しにいくためである。そのため、ログは 2 回記録される。

*以下は仮想マシン（VirtualBox）を使っている前提での解説* : また、（curl ではなく）ブラウザからアクセスすると IP アドレスは `::ffff:10.0.2.2` のように表示される。これは、VirtualBox（仮想化のアプリケーション）の仕組みのため。仮想的なネットワークを使ってホストとなる OS とゲストとなる Ubuntu を別の麻疹と認識し、それぞれ IP を割り当てているため。つまり、Mac のホスト OS から、VirtualBox を経由してゲスト OS の HTTP サーバーにアクセスするので `10.0.2.2` のようになる。（*コンテナを使った場合は、ホスト OS = ゲストOS なので、ブラウザからアクセスしても 127.0.0.1 になる？試していないからわからない*）

半面、Curl を用いた場合は、HTTP サーバを立てているゲスト OS 自身からのリクエストになるので `127.0.0.1` となる。

#### エラーログについて解説
エラーを起こすために、試しに同じポートで 2 つのサーバを立てようとしてみる。
tmux か　VS Code のターミナルなどで、`node index.js` を 2 つのターミナルで実行する（2 つめに実行しようとするとエラーが出る）。

これを、上記コードのようにハンドリング処理を書いていればエラーログとして出力できる。

なお、これらのログをファイルにも残したい場合は、以下のコマンドで起動する。

```shell
node index.js 2>&1 | tee -a application.log
```

このコマンドで起動することで、標準出力と標準エラー出力をコンソールとファイルの両方に出力する。`tee` コマンドは、コンソールとファイルの両方に出力するコマンド。

ひとつめのウインドウで `node index.js 2>&1 | tee -a application.log` としてサーバーを起動し、もう一つのウインドウでも `node index.js 2>&1 | tee -a application.log` と実行してみる。その後、application.log の内容を確認すると、

```txt
Listening on 8000
[Sun Apr 26 2020 08:44:35 GMT+0000 (Coordinated Universal Time)] Server Error { Error: listen EADDRINUSE: address already in use :::8000
    at Server.setupListenHandle [as _listen2] (net.js:1290:14)
    at listenInCluster (net.js:1338:12)
    at Server.listen (net.js:1425:7)
    at Object.<anonymous> (/workspace/node-js-http/index.js:33:8)
    at Module._compile (internal/modules/cjs/loader.js:689:30)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:700:10)
    at Module.load (internal/modules/cjs/loader.js:599:32)
    at tryModuleLoad (internal/modules/cjs/loader.js:538:12)
    at Function.Module._load (internal/modules/cjs/loader.js:530:3)
    at Function.Module.runMain (internal/modules/cjs/loader.js:742:12)
  code: 'EADDRINUSE',
  errno: 'EADDRINUSE',
  syscall: 'listen',
  address: '::',
  port: 8000 }
```

のようにログがファイルに出力されている。

## HTTP のメソッド
**HTTP メソッド** とは、HTTP のリクエストの種類。HTTP 1.1 のリクエストは 8 種類ある。

|メソッド|意味|CRUD|例（動画サイトの場合）|
|---|---|---|---|
|GET|取得|Read|動画を視聴する|
|POST|追加|Create|動画やコメントを投稿する|
|PUT|更新/追加|Update / Create|動画の情報を変更する|
|DELETE|削除|Delete|動画を削除する|

この他にも、HEAD と OPTIONS と TRACE と CONNECT というメソッドがある。

### 実装

```js
'use strict';
const http = require('http');

const server = http.createServer((req, res) => {
    const now = new Date();
    console.info('[' + now + '] Requested by '+ req.connection.remoteAddress);

    res.writeHead(200, {
        'Content-Type': 'text/plain; charset=utf-8'
    });

    switch (req.method) {
        case 'GET':
            res.write('GET ' + req.url);
            break;
        
        case 'POST':
            res.write('POST ' + req.url);
            let rawData = '';
            req.on('data', (chunk) => {
                rawData += chunk;
            }).on('end', () => {
                console.info('[' + now + '] Data posted: ' + rawData);
            });
            break;
        
        case 'DELETE':
            res.write('DELETE ' + req.url);

        default:
            break;
    }    
    res.end();
}).on('error', error => {
    console.error('[' + new Date() + '] Server Error', error);
}).on('clientError', error => {
    console.error('[' + new Date() + '] Client Error', error);
});
const port = 8000;

server.listen(port, () => {
    console.log('Listening on ' + port);
});
```

**コードの説明**：

`req` オブジェクトから、HTTP メソッドの文字列 `req.method` を得て、処理を分岐させている。`GET` メソッドならば、その URL をコンテンツとしてレスポンスに返して終わり。`POST` メソッドでもまずは URL をコンテンツとしてレスポンスに返している。

ただし、POST の際には、追加して送られてくるデータがあるので、その処理を喜寿tうしている。

`req` と書かれているリクエストオブジェクトも、Stream と同様にイベントを発行するオブジェクトである。そのため、データを受け取った際には `data` というイベントが発生する。

データは細切れな状態で chunk 変数に入れて受け取り、それを元の `rawData` という文字列につなげる。すべて受信し終わって `end` というイベントを受け取ったら、文字列データを info ログとして出力する。


**コードの実行のしかた**：

tmux あるいは　VS Code のコンソールで 2 つウインドウを立ち上げて、一方では `node index.js` でサーバを立ち上げ、もう一つのウインドウからリクエストする。

- GET メソッド
  - `curl http://localhost:8000/messages` のように、URL のパスに `/messages` などをつけてアクセスしてみる。
    - `GET /messages` と、クライアント側のコンソールに表示される
- POST メソッド
  - `curl -d 'message=HELLO!' http://localhost:8000/messages` のように、`-d` オプションをつけることで、データを POST メソッドを使って送信する。
    - 実行後、クライアント側では `POST /messages` と表示され、サーバ側のコンソール では `Data posted: message=HELLO!` のように POST されたデータが表示される。
- DELETE メソッド
  - `curl -X DELETE http://localhost:8000/messages` のように、`-X` オプションを使って DELETE メソッドを送信できる。

> ※ **Curl コマンドのオプションについて補足**　
> - POST/PUT/DELETE は `-X` オプションの後にメソッド名を書く
>   - `curl -X {POST, PUT, DELETE} [URL]`)。
>- POST でデータを送信するには `-d` オプションのあとにデータ内容を書く。ファイルデータの場合はファイル名の前に `@` をつける
>    - `curl -d "hoge=0&fuga=1" http://www.example.com`
>   - `curl -d @hoge.json http://www.example.com`

## HTML のフォーム
今回は、POST メソッドを利用してアンケートフォームを作る。

### `form` タグ
`form` タグとは、input タグ等を利用した部品により情報を送信するためのタグ。ラジオボタン、テキストエリア、チェックボックス、セレクトボックスなどの部品を利用することができる。

|要素|HTML タグ|
|---|---|
|<input type="radio">ラジオボタン</input>|`<input type="radio">`|
|<input type="checkbox">チェックボックス</input>|`<input type="checkbox">`|
|<button>ボタン</button>|`<button>`|
|<input type="text" value="テキスト"></input>|`<input type="text">`|
|<textarea>テキストエリア</textarea>|`<textrea>`|
|<select>|`<select>`|
|<select size="3">aaa</select>|`<select size="3">`|
|<progress value=0.3>プログレスバー|`<progress>`|

```html
<form method="post" action="/enquetes/yaki-shabu">
```

以上のタグでは、POST メソッドを利用して `/enquetes/yaki-shabu` というパスに対して情報を投稿するように設定している。

`method` 属性には HTTP のメソッド、`action` 属性には URL のパスを設定する。

```html
<input type="radio" name="yaki-shabu" value="焼き肉" /> 焼き肉
<input type="radio" name="yaki-shabu" value="しゃぶしゃぶ" /> しゃぶしゃぶ
```

上記においては、`input` タグを使って部品を配置する。`type` 属性で部品のタイプを指定可能だが、ここでは `radio` という値を使ってラジオボタンを配置している。

`name` 属性は、POST されるデータのキーとなり、`value` 属性は値となる（key-value 構造になる）。

```html
<button type="submit">投稿</button>
```

これは、`form` タグで情報を送信する際に利用するボタンとなる `button` タグ。`type` 属性に `submit` と指定する必要がある。

### 作成する HTML

form.html

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>アンケート</title>
</head>

<body>
<h1>どちらが食べたいですか？</h1>
<form method="post" action="/enquetes/yaki-shabu">
  <input type="radio" name="yaki-shabu" value="焼き肉" /> 焼き肉
  <input type="radio" name="yaki-shabu" value="しゃぶしゃぶ" /> しゃぶしゃぶ
  <button type="submit">投稿</button>
</form>
</body>

</html>
```

### サーバーの実装
次に、index.js を編集して、HTTP サーバーに GET でアクセスした際に、このアンケートフォームを表示するように実装する。

今までの実装に手を加える部分は以下のとおり。

- 'Content-Type'（コンテンツのタイプ）を plain text から html に変更する。
- GET リクエストがあった場合に、先ほど作成した `form.html` の内容を読み込んで、読み込んだ `form.html` をそのままレスポンスとしてクライアントに返す（この読み込んだものをそのまま返すことを「パイプ」という）。
- フォームに投稿された情報をログとして出力し、投稿内容を HTML として表示する。

具体的には以下のようなコードになる。

index.js

```js
'use strict';
const http = require('http');
const server = http.createServer((req, res) => {
  const now = new Date();
  console.info('[' + now + '] Requested by ' + req.connection.remoteAddress);
  res.writeHead(200, {
    'Content-Type': 'text/html; charset=utf-8'
  });

  switch (req.method) {
    case 'GET':
      const fs = require('fs');
      const rs = fs.createReadStream('./form.html');
      rs.pipe(res);
      break;
    case 'POST':
      let rawData = '';
      req.on('data', (chunk) => {
        rawData = rawData + chunk;
      }).on('end', () => {
        const decoded = decodeURIComponent(rawData);
        console.info('[' + now + '] 投稿: ' + decoded);
        res.write('<!DOCTYPE html><html lang="ja"><body><h1>' +
          decoded + 'が投稿されました</h1></body></html>');
        res.end();
      });
      break;
    default:
      break;
  }
}).on('error', (e) => {
  console.error('[' + new Date() + '] Server Error', e);
}).on('clientError', (e) => {
  console.error('[' + new Date() + '] Client Error', e);
});
const port = 8000;
server.listen(port, () => {
  console.info('[' + new Date() + '] Listening on ' + port);
});
```

**コード詳細**

```js
const fs = require('fs');
const rs = fs.createReadStream('./form.html');
rs.pipe(res);
```

以上の部分は、`fs` モジュールの `createReadStream` でファイルの読み込みストリームを作成した後、レスポンスのオブジェクト `res` に対して `pipe` 関数で **パイプ** している。

> **パイプについて**  
> Node.js では Stream 形式のデータは、読み込み用の Stream と書き込み用の Stream をつないでそのままデータを受け渡すことができる。その関数が `pipe` という関数の機能になる。
>
> つまりここでは、HTTP のレスポンスのコンテンツとしてファイルの内容をそのまま返すことができるようになる。
>
> また、`pipe` 関数を利用した場合は `res.end` 関数を呼ぶ必要がないため、POST メソッドを使った場合のみ `res.end` を行えばよい。

```js
const decoded = decodeURIComponent(rawData);
console.info('[' + now + '] 投稿: ' + decoded);
res.write('<!DOCTYPE html><html lang="ja"><body><h1>' +
  decoded + 'が投稿されました</h1></body></html>');
```

上記の部分は、まず最初に JavaScript の decodeURIComponent 関数を利用して、URL エンコードされた値をデコードして元に戻している。

`console.info(...)` の部分は、「yaki-syabu=焼き肉」か「yaki-syabu=しゃぶしゃぶ」のどちらに投稿されたのかをログに残す処理になる。  
（POST されたデータは、（デコードしないかぎりは）URL エンコードされたうえで、`=` でキーと値を結合された形で送信される（`yaki-shabu=%E7%84%BC%E3%81%8D%E8%82%89` のようになる）。

`res.write(...)` の部分は、投稿内容を HTML の見出しとしてレスポンスとして返している。

## テンプレートエンジン
今回は、動的にさまざまなアンケートを表jいさせてみる。前回は、焼肉としゃぶしゃぶのアンケートを提供していたが、今回はごはんとパンのアンケートを作っていく。

また、URL のパスを

- enquetes/yaki-shabu にアクセスしたときは焼肉としゃぶしゃぶ
- enquetes/rice-bread にアクセスしたときはごはんとパン

のようにそれぞれ異なるアンケートを出せるようにする。

### テンプレートエンジンとは
テンプレートエンジンとは、テンプレートと文字列とプログラムを組み合わせることで、静的なユーザーインターフェースのデータである HTML を動的に出力することができるライブラリ。

今回は pug というものを利用する。pug は、閉じタグが不要なタグの宣言や、入れ子構造に沿った適切なインデントを使うことで、HTML を簡潔
に表現できる。

まずは pug をインストールする。

```bash
yarn add pug@2.0.0-rc.4
yarn global add pug-cli
echo "node_modules" >> .gitignore
git add .gitignore
```

ターミナルで `echo "h1 Pug! | pug` と実行し、`<h1>Pug!</h1>` のように表示されれば問題なくインストールされている。

具体的に、pug の書き方は次のようになる。

```pug
doctype html
html(lang="ja")
  head
    meta(charset="UTF-8")
    title アンケート
  body
    h1 どちらが食べたいですか？
    form(method="post" action="/enquetes/yaki-shabu")
      span 名前:
      input(type="text" name="name")
      input(type="radio" name="yaki-shabu" value="焼き肉")
      span 焼き肉
      input(type="radio" name="yaki-shabu" value="しゃぶしゃぶ")
      span しゃぶしゃぶ
      button(type="submit") 投稿
```

基本的に要素名、その要素の値、と記述するが、属性は要素名の後に `()` を書き、その中に記述する。

この pug を使って、リクエストのあった path に応じて、複数のアンケートを動的に表示させるようにする。

まず、pug ファイルはこのように記述する。

**form.pug**

```pug
doctype html
html(lang="ja")
  head
    meta(charset="UTF-8")
    title アンケート
  body
    h1 どちらが食べたいですか？
    form(method="post" action=path)
      span 名前:
      input(type="text" name="name")
      input(type="radio" name="favorite" value=firstItem)
      span #{firstItem}
      input(type="radio" name="favorite" value=secondItem)
      span #{secondItem}
      button(type="submit") 投稿
```

JS は次のような実装になる。

**index.js**

```js
'use strict';
const http = require('http');
// pug モジュールを読み込む
const pug = require('pug');
const server = http.createServer((req, res) => {
  const now = new Date();
  console.info('[' + now + '] Requested by ' + req.connection.remoteAddress);
  res.writeHead(200, {
    'Content-Type': 'text/html; charset=utf-8'
  });

  switch (req.method) {
    case 'GET':
      // リクエストの URL に応じて出しわける
      if (req.url === '/enquetes/yaki-shabu') {
        // pug モジュールを利用して `form.pug` を描画し、レスポンスに書き出す。また、描画の関数 renderFile() に、埋め込みたい変数をプロパティとして持っているオブジェクトを渡してあげる
        res.write(pug.renderFile('./form.pug', {
          path: req.url,
          firstItem: '焼き肉',
          secondItem: 'しゃぶしゃぶ'
        }));
      } else if (req.url === '/enquetes/rice-bread') {
        res.write(pug.renderFile('./form.pug', {
          path: req.url,
          firstItem: 'ごはん',
          secondItem: 'パン'
        }));
      }
      res.end();
      break;
    case 'POST':
      let rawData = '';
      req.on('data', (chunk) => {
        rawData = rawData + chunk;
      }).on('end', () => {
        const decoded = decodeURIComponent(rawData);
        console.info('[' + now + '] 投稿: ' + decoded);
        res.write('<!DOCTYPE html><html lang="ja"><body><h1>' +
          decoded + 'が投稿されました</h1></body></html>');
        res.end();
      });
      break;
    default:
      break;
  }
}).on('error', (e) => {
  console.error('[' + new Date() + '] Server Error', e);
}).on('clientError', (e) => {
  console.error('[' + new Date() + '] Client Error', e);
});
const port = 8000;
server.listen(port, () => {
  console.info('[' + new Date() + '] Listening on ' + port);
});
```

このように実装すると、`http://localhost:8000/enquetes/yaki-shabu`、`http://localhost:8000/enquetes/rice-bread` それぞれにアクセスすると 2 つのアンケートが出し分けられる。

ごはんとパンのアンケートに答えると、`name=thesugar&favorite=パンが投稿されました` のようにブラウザには表示され、コンソールにも `[Mon Apr 27 2020 16:56:22 GMT+0000 (UTC)] 投稿: name=thesugar&favorite=パン` のように表示される（複数の POST されたデータは & で連結される）。

## Heroku で Web サービスを公開
### 準備手順
- Heroku アカウント作成
- Heroku CLI のインストール
  - どのようにやってもいいんだろうけど、教材では `sudo snap install --classic heroku` とやっている（Ubuntu環境）。`snap` コマンドは apt と同じようなパッケージマネージャである Snappy を操作するコマンド。Snappy は既存のアプリケーションやシステムの影響を受けないという特徴を持つ。
  - `heroku help` とすればコマンドの使い方が表示される。
  - `heroku login -i` とするとログインできる
- プロジェクトディレクトリで `Procfile` を作成する
  - ```bash
    echo "web: node index.js" > Procfile
    ```
    として作成する。
  - `Procfile` は、Heroku がアプリを動作させる際に「どのようなコマンドを実行すべきなのか」を認識するための設定ファイル。
- 次に `app.json` というファイルを作成し、以下のように書く。これは、Heroku で動かすアプリの説明ファイル。

  ```json
  {
    "name": "Enquetes",
    "description": "Enquetes for favorite foods.",
    "repository": "https://github.com/progedu/node-js-http-3016",
    "logo": "",
    "keywords": ["node"],
    "image": "heroku/nodejs"
  }
  ```
- そうしたら次に `package.json` に engines を追記。

  ```json
  ...}, "engines": {
    "node" : "~10"
  }
  ```
- 最後に、`index.js` の `const port = 8000` の部分を以下のように書き換える。

  ```js
  const port = process.env.PORT || 8000;
  ```

ここまで完了したら、いったんローカルで動作確認をして問題ないか試す。

```bash
# 必要なら
# yarn install
node index.js
```

問題ないようであれば、Git のリポジトリにコミットする。

```bash
git add .
git commit -am "Heroku で起動できるように変更"
```

Heroku ではサーバー上へソースコードを配置する（デプロイする）ために Git を利用しているため、Heroku での実行前にはこのように Git のリポジトリにコミットすることが必要。

### デプロイ手順
まずはサーバの用意をする。

```bash
heroku create
```

`heroku create` は、ランダムな名前のサーバ名で Heroku にサーバを用意し、そのサーバの Git のリポジトリを heroku という名前のリモートリポ一人として登録する。

なお Heroku ではサーバーは仮想化されており、この 1 サーバーを Dyno（ダイノ）という単位で呼ぶ。

*⚠️*　ここで注意点！同一ディレクトリ内で 2 回以上 `heroku create` すると設定がややこしくなり、以降の作業に支障をきたす。`heroku create` は必ず 1 回だけ実行すること。

```bash
Creating app... done, xxx-xxx-XXXX
https://xxx-xxx-XXXX.herokuapp.com/ | https://git.heroku.com/xxx-xxx-XXXX.git
```

以上のように表示される。ここの `...herokuapp.com` の URL がサーバーの URL になる。

> Tips  
2 回以上 heroku create してしまったとき
同一ディレクトリ内で 2 回以上 heroku create してしまった場合は、heroku info で 今使用している Heroku サーバーの情報を確認しましょう。
ここで表示される URL が デプロイ先のサーバーです。デプロイ後はこの URL にパスを追加して確認しましょう。
また heroku apps で、あなたの Heroku にあるすべてのサーバー一覧を確認できます。
heroku destroy --app サーバー名 --confirm サーバー名 で Heroku サーバーを削除できるので、不要なものは削除しましょう。

次にデプロイを行う。

```bash
git push heroku master-2019:master
```
↑の Git コマンドは、 heroku（リモート）の master にローカルの （たとえば）master-2019（の名前のリポジトリ） を push するという意味。

デプロイ実行後、最終的に

```
remote: Verifying deploy.... done.
```

と表示されれば成功。そうすれば、さきほどの URL に path を追加してアクセスすれば動作を確認できる。

投稿した内容がログに表示されているかを確認するには、

```bash
heroku logs
```

を実行すると、ログが表示される。Heroku のログ機能自体に、日付を出力する機能があるため、アプリ側で日時を書き出す処理を書いていると、日時が二重で表示される。

なお、`heroku logs` コマンドを入力しても何も表示されない場合は

```bash
heroku logs -a xxx-xxx-XXXX
```

を試してみること。`xxx-xxx-XXXX` は先ほど表示されたサーバーのホスト名。忘れた場合は `heroku apps` で確認できる。
