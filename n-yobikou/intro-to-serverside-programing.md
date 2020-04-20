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