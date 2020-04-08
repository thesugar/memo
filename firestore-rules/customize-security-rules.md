# [Cloud Firestore セキュリティルールをカスタマイズする](https://firebase.google.com/docs/firestore/security/secure-data?hl=ja)

## 階層データを操作する
Firestore では、コレクション、ドキュメント、サブコレクションの階層でデータが保存され、階層深くの複数のレイヤで構成されるデータの保護が必要になることがよくある。したがって、この階層データを適切に操作する方法を理解することが重要。

Firestore では、 「`/databases/` データベース名 `/documents`」で始まるパスにすべてのドキュメントが保存される。一般に、このパスは Firestore を使用するクライアントには開示されない。

### 完全一致検索
特定のドキュメントに対してルールを指定する場合は、該当するドキュメントのフルパスを記述する。

```
service cloud.firestore {
    match /databases/{database}/documents {
        match /spaceships/serenity {
            // rules go here...
        }
    }
}
```

### 検索のネスト
`match` 句はネストできる。

### ルールはカスケードされない
`=**` ワイルドカードを明示的に使用しないかぎり、カスケードされない。つまり、コレクション内のドキュメントに対してセキュリティルールを指定した場合、それらのルールはそのドキュメントにのみ適用され、 **そのドキュメントのサブコレクション内のドキュメントには適用されない** 。  

次の例では、ルールは `/spaceships/serenity` ドキュメントには適用されるが、 `/spaceships/serenity/crew/hoban` ドキュメントには適用されない（したがって、デフォルトでは誰もそのクルーメンバーにアクセスできない）。

```
service cloud.firestore {
    match /databases/{database}/documents {
        match /spaceships/serenity {
            // Rule go here for the Serenity spaceship.
            // serenity の下のサブコレクションにはなんのルールも適用されない。
        }
    }
}
```

コレクションのルールは、そのコレクション内のドキュメントには適用されない。ちなみに、セキュリティルールをドキュメントレベルではなくコレクションレベルで貴j通することはまれであり、またおそらく *間違いでもある* 。  
次の例では、ドキュメントレベルでルールが記述されていないため、`spaceships` コレクション内のドキュメントを誰も読み取ることができない。

❌　意図通りに動かない
```
service cloud.firestore {
    match /databases/{database}/documents {
        match /spaceships {
            allow read;
            // ドキュメントに対してルールを記述していないので、spaceships 内のドキュメントを誰も読み取ることができない
        }
    }
}
```

### ワイルドカードの使用
実際には特定のコレクションだけでなく、コレクション内のすべてのドキュメントに適用されるルールを記述する必要がある場合がほとんど。そのような状況では、ワイルドカードを使用して、パス内のすべての要素を指定できる。

```
service cloud.firestore {
    match /databases/{database}/documents {
        match /spaceships/{spaceship} {
            // Rules go here
        }
    }
}
```

### ワイルドカードを使用した再帰的検索
ワイルドカード検索の最後に `=**` を追加すると、そのレベルのパス要素だけでなく、下位にあるすべてのドキュメントまたはサブコレクションも再帰的に検索される。

```
service cloud.firestore {
    match /databases/{database}/documents {
        match /spaceships/{spaceship=**} {
            // Rules go here...
            // spaceships コレクションに含まれるすべてのドキュメントおよびそのサブコレクションに存在するすべてのドキュメントに適用される
        }
    }
}
```

このやり方だとなかなか小回りが効かないといえば効かない。以下のような書き方も便利。

```
service cloud.firestore {
    match /databases/{spaceship} {
        // "spaceships" の中のドキュメントに適用されるルール
        
        match /{allChildren=**} {
            // サブコレクションの中のドキュメントに適用されるルール
        }
    }
}
```

### 複数のルールが同じドキュメントに一致する場合
AND 条件ではなく OR 条件。どこかで緩いルールを書いていると他で厳しいルールを書いていても突破される。

## セキュリティルールを評価する
Firestore セキュリティルールはすべて、アクションを許可するかどうかを決定するブール式の形式を取る。デフォルトの動作は false。

## 条件付きロジック
多くの場合、特定の基準を満たす場合に条件に応じてアクションを許可するルールを記述することが必要になる。ここでは、よく使用されるメソッドを紹介する。

- `==` および `!=` を使用して、数値と文字列の両方の等価チェックを行う。
- `== null` または `!= null` を使用して、ドキュメント内にフィールドまたは値が存在するかどうかを確認する
- `is` を使用して、データが特定の型であることを確認する。確認できるデータ型は `string`、`int`、`float`、`bool`、`null`、`timestamp`、`list`、`map`。
- `in` を使用して、値がリストまたはマップ内にあることを確認する。
- `get()` 関数を使用して、ドキュメントをマップに変換する。
- `exists()` を使用して、テータベースにドキュメントが存在するかどうかを確認する。

### マップの操作
Firestore セキュリティルールで扱うデータの多くは、マップ形式、つまり Key-Value ペアのコレクション。マップ内のフィールドにアクセスするには、ドット表記または角カッコを使用する。角カッコを使用する場合は、評価しているフィールドの名前を二重引用符で囲む必要がある。

```
allow read: if request.auth.token.displayName == "Malcolm Reynolds";
allow read: if request["auth"]["token"]["displayName"] == "Malcolm Reynolds";
```

### request 変数
`request` 変数は、クライアントから受信したリクエストに関する情報を含むマップ。よく使用されるフィールドは `auth` と `resource` の 2 つ。前者は Firebase Authentication から取得した現在ログインしているユーザーに関する情報を含み、後者は送信されたドキュメントに関する情報を含む。  
  
request オブジェクトの一般的な使用例は次のとおり。
- `request.auth != null` として、ユーザがログインしていることを確認する
- `request.auth.uid` の値を使用して、Firebase Authentication によって確認されたユーザの ID を取得する。
- `request.resource.data.<some_key>` を使用して、ドキュメントの書き込みのために送信された値が特定の書式を満たしていることを確認する。これは、データベース内でデータ検証を実行する最も一般的な方法。

```
service cloud.firestore {
    match /databases/{database}/documents {
        match /bulletinBoard/{note} {
            allow read, write: if request.auth != null;
        }

        match /users/{userID}/myNotes/{note} {
            allow read, write: if request.auth.uid == userID;
        }

        match /spaceships/{spaceship} {
            allow read;
            // Spaceship documents can only contain three fields
            // その他の条件はコードの通り。型の指定や数値の範囲など。また、is string などとすることで null は拒否される。
            allow write: if request.resource.data.size() == 3
                        && request.resource.data.name is string
                        && request.resource.data.slogan is string
                        && request.resource.data.cargo is int
                        && request.resource.data.cargo > 6500;
        }
    }
}
```

### resource 変数
`resource` 変数は `request.resource` 変数と似ているが、 **データベースにすでに存在するドキュメントを参照する** 点が異なる。読み取りオペレーションの場合は読み取るドキュメント、書き込みオペレーションの場合は更新または変更する古いドキュメントを参照する。  
  
`resource` オブジェクトと `request.resource` オブジェクトはどちらもマップ。どちらにもドキュメント名を示す `__name__` プロパティと、すべてのユーザー定義プロパティを宣言する `data` プロパティが含まれる。これらのマップの `data` プロパティ内のキーにアクセスすることによって、参照されるドキュメントに属するさまざまなフィールドの値を取得できる。

**例**
```
allow read: if resource.data.visibility in ["public", "read-only"]
allow update: if request.resource.data.headline == resource.data.headline
                && resource.data.authorID == request.auth.uid
                && request.resource.data.authorID == request.auth.uid;
```

### ワイルドカードから作成された変数の使用
ワイルドカードを含むデータベースパスを使用してルールを一致させる際に、中カッコ内に含まれる文字列に一致する名前の変数が作成される。その後、セキュリティルールでその変数を参照できるようになる。

**例**
```
match /users/{userID} {
    allow read, write: if request.auth.uid == userID;
}
```

### 現在データベースにあるドキュメントの評価
`resource` 変数を使用して、アクセスしようとしている現在のドキュメントを評価できるが、データベースの他の部分にある他のドキュメントを評価したい場合もある。  

これを行うには、データベース内のドキュメントパスを指定して `get()` 関数を呼び出す。これにより、現在のドキュメントが取得され、`resource` オブジェクトと同様に、`data` プロパティを使用してカスタムフィールドにアクセスできるようになる。  
  
この機能の一般的な使用方法の 1 つは、アクセス制御リストを評価し、現在のユーザーの userID がそのリストに含まれるかどうかを確認すること。  

また、`exists()` 関数を使用して、ドキュメントがすでに存在するかどうかを確認することもできる。
  
`exists()` 関数または `get()` 関数にパスを入力する際は、引用符を含めない。パス名に変数を含める場合は、変数をカッコで囲み、その前にドル記号をつける。

```
service cloud.firestore {
    match /databases/{database}/documents {
        match /games/{game}/playerProfiles/{playerID} {
            allow write: if get(/databases/$(database)/documents/games/$(game)).data.referee == request.auth.uid;

            allow read: if exists(/databases/$(database)/documents/games/$(game)/playerProfiles/$(request.auth.uid));
        }

        match /guilds/{guildID}/bulletinBoard/{post} {
            allow write: if get(/databases/$(database)/documents/guilds/$(guildID)).data.users[(request.auth.uid)] in ["Admin", "Member"];
        }
    }
}
```

### カスタム関数の記述
関数は単一の return 文のみで構成する。関数はルール内のどこでも宣言できる。一致ブロック内で宣言すると、関数はそのブロック内で定義されているすべてのワイルドカード変数にアクセスできる。

## 書き込みの保護
Firestore は `create`、`update`、`delete` の 3 種類の書き込みオペレーションがある。これらは、クライアントライブラリの `set()`、`add()`、`update()`、`remove()`、`transaction()` メソッドに対応している。`write` オペレーションを実行すると、これらの処理をすべて行うことができる。  
  
書き込みルールは主にドキュメントに書き込まれるコンテンツの検証に使用される。たとえば、特定のフィールドが存在し、他のフィールドが存在しないことを確認したり、フィールドが特定のタイプであることを確認したりする。  
  
```
service cloud.firestore {
    match /databases/{database}/documents {
        match /rooms/{roomId} {
            match /messages/{messageId} {
            // A message must only contain two string fields, named "name" and "text"
            allow write: if request.resource.data.keys().hasAll(["name", "text"])
                        && request.resource.data.size() == 2
                        && request.resource.data.name is string
                        && request.resource.data.text is string;
            }
        }
    }
}
```

次のコードを使用すると、ルールをテストできる。

```js
const messageRef = db.collection("rooms").doc("chat").collection("messages");

// ✅　条件に合致するので成功する
messageRef.add({
    name: "River Tam",
    text: "My food is problematic."
});

// 🛑　text が不動小数点になっているうえ、余分な "timestamp" フィールドがあるため失敗する
messageRef.add({
    name: "River Tam",
    text: 3.1415,
    timestamp: Data.now()
})
```

データを更新するときに、更新前後で、ある値（フィールド）が変更されていないことを制約とするには以下のように書く：

```
allow update: if request.resource.data.name == resource.data.name;
```

### 読み取りの保護
Firestore の読み込みとクエリには、`get` と `list` という 2 種類のオペレーションがある。これらは、クライアントライブラリの `get()` と `where().get()` メソッドに対応する。`read` オペレーションを実行すると、この両方の処理が行われる。
