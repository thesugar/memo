# [ユーザーとグループのデータアクセスを保護する](https://firebase.google.com/docs/firestore/solutions/role-based-access?hl=ja)
たとえば、ドキュメント編集アプリでは、限られたユーザーにドキュメントの読み書きを許可すると同時に、不要なアクセスをブロックすることが必要になる場合がある。

## Solution: 役割ベースのアクセス制御
Firestore のデータモデルとセキュリティルールを活用し、アプリに役割ベースのアクセス制御を実装することができる。  
  
たとえば、次のセキュリティ要件で「ストーリー」と「コメント」を作成できる文章作成のアプリを構築しているとする。  

- 各ストーリーには所有者（owner）が 1 人存在する。また、各ストーリーは「作成者（writer）」、「コメント投稿者（commenter）」、「読者（reader）」と共有することができる。
- 読者はストーリーとコメントのみを表示でき、編集はできない。
- コメント投稿者は読者のすべてのアクセス権を持ち、さらにストーリーにコメントを追加することもできる。
- 作成者はコメント投稿者のすべてのアクセス権を持ち、さらにストーリーコンテンツも編集できる。
- 所有者はストーリーのあらゆる部分を編集でき、他のユーザーのアクセスを制御することもできる。

## データ構造
アプリには `stories` コレクションがあり、そこでは各ドキュメントが 1 つのストーリーを表すとする。各ストーリーには `comments` サブコレクションもあり、そこでは各ドキュメントがそのストーリーのコメントになる。  
アクセス役割を管理するために、ユーザー ID と役割とのマッピングとなる `roles` フィールドを追加する。

**/stories/{storyid}**
```js
{
    title: "A Great Story",
    content: "Once upon a time...",
    roles: {
        alice: "owner",
        bob: "reader",
        david: "writer",
        jane: "commenter"
        // ...
    }
}
```
（roles というフィールドを持たせること自体も、そのフィールドが「人名がキーで役割がバリューのオブジェクト」であることも、あまり今の自分ではできない発想。。役割をキー、人名の配列をバリューにしたオブジェクトにしちゃいそう。）

コメントには、投稿者のユーザー ID とコンテンツの 2 つのフィールドのみを含める。

**/stories/{storyid}/comments/{commentid}**
```js
{
    user: "alice",
    content: "I think this is a great story!"
}
```

## ルール
以上でユーザーの役割がデータベースに記録されたので、次はセキュリティルールを作成してそれらを検証する必要がある。

**ステップ 1** : 最初に基本ルールファイルを作成する。これには、ストーリー用とコメント用の空のルールを含める。

```
service cloud.firestore {
    match /databases/{database}/documents {
        match /stories/{story} {
            // TODO: Story rules go here...

            match /comments/{comment} {
                // TODO: Comment rules go here...
            }
        }
    }
}
```

**ステップ 2** : 所有者がストーリーを完全に制御できるようにする単純な `write` ルールを追加する。以下に定義されている各関数は、ユーザーの役割と、新しい文書が有効かどうかを判断するために利用される。

```
service cloud.firestore {
    match /databases/{database}/documents {
        match /stories/{story} {
            function isSignedIn() {
                return request.auth != null;
            }

            function getRole(resource) {
                return resource.data.roles[request.auth.uid];
            }

            function isOneOfRoles(resource, array) {
                return isSignedIn() && (getRole(resource) in array);
            }

            function isValidNewStory() {
                // 今の時点でストーリーが存在せず、新しいストーリーの所有者がリクエストをかけてきているユーザーの uid と一致すれば valid。request.resource は将来のデータを表す。
                return resource == null
                        && request.resource.data.roles[request.auth.uid] == 'owner';
            }

            // 新しいストーリーの場合は isValidNewStory() で判定、既存のストーリーの場合は owner なら write できる
            allow write: if isValidNewStory() || isOneOfRoles(resource, ['owner'])

            match /comments/{comment} {
                // TODO: Comment rules go here...
            }
        }
    }
}
```

**ステップ 3**: すべての役割のユーザーがストーリーとコメントを読むことができるルールを作成する。前のステップで定義した関数を使用することで、ルールが簡潔でわかりやすくなる。

```
service cloud.firestore {
    match /databases/{database}/documents {
        match /stories/{story} {
            function isSignedIn() {
                return request.auth != null;
            }

            function getRole(resource) {
                return resource.data.roles[request.auth.uid];
            }

            function isOneOfRoles(resource, array) {
                return isSignedIn() && (getRole(resource) in array);
            }

            function isValidNewStory() {
                return resource == null
                        && request.resource.data.roles[request.auth.uid] == 'owner';
            }

            allow write: if isValidNewStory() || isOneOfRoles(resource, ['owner']);
            allow read: if isOneOfRoles(resource, ['owner', 'writer', 'commenter', 'reader']);

            match /comments/{comment} {
                // get('<fullpath>') は resource を返す
                allow read: if isOneOfRoles(get(/databases/$(database)/documents/stories/$(story)), ['owner', 'writer', 'commenter', 'reader']);
            }
        }
    }
}
```

**ステップ 4** : ストーリー作成者、コメント投稿者、所有者がコメントを投稿できるようにする。このルールでは、コメントの `owner` がリクエスト元のユーザーと一致しているかどうかも検証し、ユーザーがお互いのコメントを上書きすることがないようにする。

```
service cloud.firestore {
    match /databases/{database}/documents {
        match /stories/{story} {
            function isSignedIn() {
                return request.auth != null;
            }

            function getRole(resource) {
                return resource.data.roles[request.auth.uid];
            }

            function isOneOfRoles(resource, array) {
                return isSignedIn() && (getRole(resource) in array);
            }

            function isValidNewStory() {
                return resource == null
                        && request.resource.data.roles[request.auth.uid] == 'owner';
            }

            allow write: if isValidNewStory() || isOneOfRoles(resource, ['owner']);
            allow read: if isOneOfRoles(resource, ['owner', 'writer', 'commenter', 'reader']);

            match /comments/{comment} {
                allow read: if isOneOfRoles(get(/databases/$(database)/documents/stories/$(story)), ['owner', 'writer', 'commenter', 'reader']);

                allow create: if isOneOfRoles(get(/databases/$(database)/documents/stories/$(story)), ['owner', 'writer', 'commenter'])
                              && request.resource.data.user == request.auth.uid;
            }
        }
    }
}
```

**ステップ 5** : 作成者（writer）に、ストーリーのコンテンツは編集できるが、ストーリーの役割を編集したり、ドキュメントの他のプロパティを変更したりすることはできない権限を付与する。そのためには、次のようにストーリーの `write` ルールを `create`、`update`、`delete` の別個のルールに分割し、作成者がストーリーのみを更新できるようにする必要がある。

```
service cloud.firestore {
    match /databases/{database}/documents {
        match /stories/{story} {
            function isSignedIn() {
                return request.auth != null;
            }

            function getRole(resource) {
                return resource.data.roles[request.auth.uid];
            }

            function isOneOfRoles(resource, array) {
                return isSignedIn() && (getRole(resource) in array);
            }

            function isValidNewStory() {
                return resource == null
                        && request.resource.data.roles[request.auth.uid] == 'owner';
            }

            // この関数を追加
            function onlyContentChanged() {
                // タイトルと役割はこのリクエストを許可しても変わらず、また、新しいフィールドがドキュメントに追加されることもないことを確かめる
                return request.resource.data.title == resource.data.title
                    && request.resource.data.roles == resource.data.roles
                    && request.resource.size() == resource.size()
            }

            allow create: if isValidNewStory();
            allow delete: if isOneOfRoles(resource, ['owner']);
            allow update: if isOneOfRoles(resource, ['owner'])
                            || (isOneOfRoles(resource, ['writer'])) && onlyContentChanged());
            allow read: if isOneOfRoles(resource, ['owner', 'writer', 'commenter', 'reader']);

            match /comments/{comment} {
                allow read: if isOneOfRoles(get(/databases/$(database)/documents/stories/$(story)), ['owner', 'writer', 'commenter', 'reader']);

                allow create: if isOneOfRoles(get(/databases/$(database)/documents/stories/$(story)), ['owner', 'writer', 'commenter'])
                              && request.resource.data.user == request.auth.uid;
            }
        }
    }
}
```

### 制限事項
- 粒度：上記の例では、複数の役割（writer と owner）が同じドキュメントへの書き込みアクセス権を持っているが、制限事項がそれぞれ異なる。このような方法は複雑なドキュメントでは管理が難しくなるため、 1 つのドキュメントを複数のドキュメントに分割し、それぞれを単一の役割に所有させるほうが良い場合がある。
- 大規模グループ：非常に大規模なグループまたは不k図厚なグループと共有する必要がある場合は、役割を対象ドキュメントのフィールドとしてではなく、専用のコレクションに格納するシステムを検討せよ。