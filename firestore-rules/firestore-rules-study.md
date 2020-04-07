# Firestore セキュリティルール勉強
## [Cloud Firestore セキュリティルールを使ってみる](https://firebase.google.com/docs/firestore/security/get-started?hl=ja)
### ルールの記述
セキュリティルールの基本は、データベース内のドキュメントを識別する `match` ステートメントと、それらのドキュメントへのアクセスを制御する `allow` 式で構成される。

```
service cloud.firestore {
    match /databases/{database}/documents {
        match /<some_path>/ {
            allow read, write: if <some_condition>;
        }
    }
}
```

すべてのデータベースリクエストは、データの読み取り・書き込みの前に、セキュリティルールと照合して評価される。
指定されたいずれかのドキュメントパスへのアクセスがルールによって拒否されると、リクエスト **全体** が失敗する。  
  
基本的なルールセットの例（本番環境アプリケーションでの使用は非推奨）

```
// アプリにサインインしているユーザーにすべてのドキュメントへの read/write アクセスを許可する
service cloud.firestore {
    match /databases/{database}/documents {
        match /{document=**} {
            allow read, write: if request.auth.uid != null;
        }
    }
}
```

上記の `{document=**}` パスは、 **データベース全体の任意のドキュメントに一致** する。

### ルールのテスト
Cloud Firestore にはルールシミュレータが用意されている。認証済みリクエストをシミュレートする場合は、さまざまなプロバイダから認証トークンを作成してプレビューできる。

### ルールのデブロイ
ルールのデブロイには、Firebase コンソール または Firebase CLI を使用できる。
Firebase CLI を使用してセキュリティルールをデブロイすると、Firebase コンソール内の既存のルールが、プロジェクトディレクトリで定義したルールで上書きされる。
CLI でのデプロイは以下のコマンド。

```zsh
firebase deploy --only firestore:rules
```

## [Cloud Firestore セキュリティ ルールを構造化する](https://firebase.google.com/docs/firestore/security/rules-structure?hl=ja)
### サービスとデータベースの宣言
セキュリティルールは、つねに次の宣言で始まる。

```
service cloud.firestore {
    match /databases/{database}/documents {
        // ...
    }
}
```

`service cloud.firestore` という宣言では、ルールのスコープを Firestore とし、Firestore ルールと、Cloud Storage など他のプロダクトのルールとの間の競合を防いでいる。  
`match /databases/{database}/documents` 宣言は、ルールがプロジェクト内の Firestore データベースと一致sるうように指定する。

### 基本的な読み書きのルール
先述のとおり、基本的なルールは、ドキュメントパスを指定する `match` ステートメントと、指定されたデータの読み込みを許可するときに詳細を指定する `allow` 式で構成される。

```
service cloud.firestore {
    match /databases/{databses}/documents {
        match /cities/{city} {
            allow read: if <condition>;
            allow write: if <condition>;
        }
    }
}
```

すべての match ステートメントは、 **コレクションではなくドキュメントを指す必要** がある。（復習：フォルダ的なものがコレクション、フォルダの中の各ファイルのようなものがドキュメント。新しく Firestore DB を作ろうとしたときはまず **コレクション** を作ることになる）  
match ステートメントでは `match /cities/Tokyo` のように特定のドキュメントを指すことや、 `match /cities/{city}` のようにワイルドカードを使用して、指定されたパスのすべてのドキュメントを指すことができる。  

上の例では、match ステートメントで `{city}` ワイルドカード構文を使用している。つまり、このルールは、`cities` コレクションの `cities/Tokyo` や `cities/Paris` などのすべてのドキュメントに適用される。match ステートメントの `allow` 式が評価されると、`city` 変数は `Tokyo` や `Paris` などの具体的なドキュメント名に解決される。

### 詳細なオペレーション
`read` と `write` をより詳細なオペレーションに分割する方法。たとえば、単一のドキュメントの読み取りは許可するけど大規模なクエリを拒否するとか、ドキュメントの作成はいいけど削除は別の条件付けを適用するとか。  
  
`read` ルールは `get` と `list` に分割でき、`write` ルールは `create`、`update`、`delete` に分割できる。

```
service cloud.firestore {
    match /databases/{database}/documents {
        match /cities/{city} {
            // 単一のドキュメントの読み取り
            allow get: if <condition>;

            // クエリやコレクションの読み取り
            allow list: if <condition>;
        }

        match /cities/{city} {
            allow create: if <condition>;
            allow update: if <condition>;
            allow delete: if <condition>;
        }
    }
}
```

### 階層データ
Firestore のデータはドキュメントのコレクションに編成され、各ドキュメントはサブコレクションによって階層を拡張できる。さて、 `cities` コレクションの各ドキュメントに `landmarks` サブコレクションが含まれている状況を考えてみる。セキュリティルールは一致したパスにのみ適用されるため、`cities` コレクションで定義されたアクセス制御は `landmarks` サブコレクションには適用されない。かわりに、サブコレクションへのアクセスを制御する明示的なルールを記述する。

```
service cloud.firestore {
    match /databases/{database}/documents {
        match /cities/{city} {
            allow read, write if <condition>;

                // 明示的に `landmarks` サブコレクションへのアクセス制御のルールを記述
                match /landmarks/{landmark} {
                    allow read, write: if <condition>;
                }
        }
    }
}
```

`match` ステートメントをネストする場合、内部の `match` ステートメントのパスは、つねに、外側の `match` ステートメントのパスに相対的になる（上の例では、 ` match /cities/{city}/landmarks/{landmark}` と全部書かなくていいということ）。

### 再帰ワイルドカード
ルールが任意の不快階層に適用されるようにするには、再帰ワイルドカード構文 `{name=**}` を使用する。

```
service cloud.firestore {
    match /databases/{database}/documents {
        // サブコレクション内のドキュメント含め、すべてのドキュメントに match する
        match /cities/{document=**} {
            allow read, write: if <condition>;
        }
    }
}
```

再帰ワイルドカードは **0 個以上** のパスアイテムに一致する。`match/cities/{city}/{document=**}` は、任意のサブコレクション内のドキュメントと `cities` コレクション内のドキュメントに一致（0 個のパスアイテムに一致ということはそのとき `match/cities/{city}` が評価されるから）する。  

また、match ステートメントごとに使用できる再帰ワイルドカードは 1 つだけだが、このワイルドカードを match ステートメント内の任意の場所に配置できるため、`match /{path=**}/songs/{song}` というような書き方ができる。

### match ステートメントの重複
ある 1 つのドキュメントが複数の `match` ステートメントと一致する可能性があるが、複数の `allow` 式がリクエストと一致する場合、 **いずれか** の条件が `true` と評価されると、アクセスが許可される。（つまり AND 条件ではなく OR 条件で評価されるということで、どこかで弱いルールが書かれてると他の部分で強いルールを書いていても突破される。ワイルドカードを使った場合など注意が必要）

### [セキュリティルールの制限](https://firebase.google.com/docs/firestore/security/rules-structure?hl=ja#security_rule_limits)
公式ドキュメント参照。

## [Cloud Firestore セキュリティ ルールの条件の記述](https://firebase.google.com/docs/firestore/security/rules-conditions?hl=ja)
Firestore セキュリティルールの腫瘍な構成要素は「条件」である。条件とは、特定のオペレーションを許可するか拒否するかを決定するブール式。

### 認証
ユーザーの認証状態に基づいてアクセスを制御する。以下の例で、 `request.auth` 変数には、データを要求しているクライアントの認証情報が含まれている。

- アプリにログインしているユーザーのみがデータの書き込みを許可されるようにする

```
service cloud.firestore {
    match /databases/{database}/documents {
        match /cities/{city} {
            allow read, write: if request.auth.uid != null;
        }
    }
}
```

- ユーザーが自分のデータのみを読み書きできるようにする

```
service cloud.firestore {
    match /databases/{database}/documents {
        match /users/{userId} {
            allow read, update, delete: if request.auth.uid == userId;
            allow create: if request.auth.uid != null;
        }
    }
}
```

### データの検証
多くのアプリでは、アクセス制御情報を DB 内のドキュメントのフィールドとして格納する。
ドキュメントデータに基づいて動的にアクセスを許可または拒否できる。
以下の例で、`resource` は **要求されたドキュメントを参照** する。`resource.data` はドキュメントに格納されているすべてのフィールドと値のマップ。

```
service cloud.firestore {
    match /databases/{database}/documents {
        match /cities/{city} {
            allow read: if resource.data.visibility == 'public';
        }
    }
}
```

*データを書き込むとき* に、受信データと既存のデータを比較できる。このケースによって ルールセットが保留中の書き込み を許可する場合、`request.resource` 変数にはドキュメントの **将来の状態** が含まれる。ドキュメントフィールドのサブセットのみを変更する `update` オペレーションの場合、 `request.resource` 変数にはオペレーション後の保留中のドキュメントの状態が含まれる。`request.resource` のフィールド値をチェックすることで、望ましくない、または生合成のないデータの更新を防ぐことができる。この文章だとなんのことかよくわからないが、以下の例を見ればわかる。

```
service cloud.firestore {
    match /databases/{database}/documents {
        match /cities/{city} {
            allow update: if request.resource.data.population > 0
                          && request.resource.data.name == resource.data.name;
        }
    }
}
```

### 他のドキュメントへのアクセス
セキュリティルールでは `get()` 関数と `exists()` 関数を使用して、データベース内の他のドキュメントに対する受信リクエストを評価できる。`get()` 関数と `exists()` 関数は、完全なドキュメントパスを指定する。変数を使用して `get()` と `exists()` のためのパスを構築する場合は、`$(variable)` 構文を使用して明示的に変数をエスケープする必要がある。  
  
次の例では、`database` 変数が match ステートメント `match /databases/{database}/documents` によってキャプチャされ、次のパスの形成に使用される。

```
service cloud.firestore {
    match /databases/{database}/documents {
        match /cities/{city} {
            // リクエストをしてきたユーザーの uid が users コレクションの中にドキュメントとしてある（ユーザー登録済みというイメージだろう）場合のみ create を許可
            allow create: if exists(/databases/$(database)/documents/users/$(request.auth.uid))

            // get(.../$(request.auth.uid)).data で当該ドキュメントに格納されているフィールドと値のマップを得る。その中の admin フィールドが true であれば delete を許可
            allow delete: if get(/databases/$(database)/documents/users/$(request.auth.uid)).data.admin == true
        }
    }
}
```

書き込みの場合は、`getAfter()` 関数を使用すると、トランザクションまたはバッチ書き込みが完了した後（トランザクションまたはバッチが commit される前）のドキュメントの状態にアクセスできる。`get()` と同様に `getAfter()` 関数は完全なドキュメントパスを受け取る。`getAfter()` を使用すると、トランザクションまたはバッチとしてまとめて実行する必要がある一連の書き込みを定義できる。

### カスタム関数
一連の条件を関数にまとめることができる。関数の構文は JavaScript に似ているが、以下の点に注意。

- 関数には 1 つの `return` ステートメントのみを含めることができる。
- 中間変数を作成したり、ループを実行したり、外部サービスを呼び出したりといった、追加ロジックを含めることはできない。
- 関数は、定義されているスコープから、関数と変数に自動的にアクセスすることができる。たとえば、`service cloud.firestore` スコープ内で定義された関数は、`resource` 変数、および `get()` や `exists()` などの組み込み関数にアクセスできる。
- 関数は他の関数を呼び出すことはできるが、再帰はできない。コールスタックの深さは合計 10 に制限されている。
- 関数を定義するには `function` キーワードを使用し、0 個以上の引数をとる。

```
service cloud.firestore {
    match /databases/{database}/documents {
        function signedInOrPublic() {
            return request.auth.uid != null || resource.data.visibility == 'public'
        }

        match /cities/{city} {
            allow read, write: if signedInOrPublic();
        }

        match /cities/{user} {
            allow read, write: if signedInOrPublic();
        }
    }
}
```

### ルールはフィルタではない
Firestore によって返されるのは、現在のクライアントのアクセス権が設定されているドキュメントのみ。
たとえば、次のルールがあるとする。

```
service cloud.firestore {
    match /databases/{database}/documents {
        match /cities/{city} {
            allow read: if resource.data.visibility == 'public';
        }
    }
}
```

❌拒否： 次のクエリは拒否される。このクエリが実行された場合、結果セットには `visibility` が `public` でないドキュメントを含めることができるため。

```javascript
db.collection("cities").get()
    .then(querySnapshot => {
        querySnapshot.forEach(doc => {
            console.log(doc.id, '=>', doc.data())
        })
    })
```

✅許可：結果セットがルールの条件を満たすことが `where` 句によって保証されるため、このルールでは次のクエリは許可される。

```javascript
db.collection("cities").where("visibility", "==", "public").get()
    .then(querySnapshot => {
        querySnapshot.forEach(doc => {
            console.log(doc.id, '=>', doc.data())
        })
    })
```

つまり、セキュリティルールをもってアプリ側の読み書きロジックを代用できるわけではない（「セキュリティルールをちゃんと書けばアプリ側は雑に全取得をかけるだけで所望のデータだけを取ってこれるでしょ」ということにはならない）。