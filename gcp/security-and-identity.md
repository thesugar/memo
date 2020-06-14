**セキュリティとアイデンティティの基礎**

# IAM のカスタムの役割
※ [IAM の基本はこちら](https://github.com/thesugar/memo/blob/master/gcp/baseline-infrastructure.md#cloud-iam-qwik-start)。

<h2 id="step2">概要</h2>
<p>Cloud IAM にはリソースの権限を管理するための適切なツールが用意されており、少ない手間で高度な自動化を実現できます。ユーザーに権限を直接付与する代わりに、権限を組み合わせた役割を割り当てます。これにより、社内の職務権限をグループや役割にマッピングできます。ユーザーは自分の行うべきジョブにのみアクセスでき、管理者はユーザー グループ全体に対してデフォルトの権限を簡単に付与できます。</p>
<p>Cloud IAM では、次の 2 種類の役割があります。</p>
<ul>
<li>事前定義された役割</li>
<li>カスタムの役割</li>
</ul>
<p><strong>事前定義された役割</strong>は、Google によって作成、管理されます。その権限は、GCP に新しい機能やサービスが追加された場合など、必要に応じて自動的に更新されます。</p>
<p><strong>カスタムの役割</strong>とは、使用可能な権限を特定のニーズに合わせて組み合わせた、ユーザー定義の役割を指します。カスタムの役割は Google によって管理されないため、新しい権限、機能、サービスが GCP に追加されても、こうしたカスタムの役割が自動的に更新されることはありません。カスタムの役割は、使用可能な Cloud IAM 権限を組み合わせて作成します。ユーザーは権限に応じて、Google Cloud Platform のリソースで特定の操作を実行できます。</p>
<h3>演習内容</h3>
<ul>
<li>
<p>カスタムの役割の作成、更新、削除、削除の取り消しを行う</p>
</li>
</ul>
<h3>前提条件</h3>
<ul>
<li>
<p>IAM 役割に関する基本的な知識</p>
</li>
</ul>


<h2 id="step4">IAM のカスタムの役割について理解する</h2>
<p>Cloud IAM では、Cloud IAM の役割をカスタマイズすることもできます。ここでは、1 つ以上の権限を持つカスタムの Cloud IAM の役割を作成してユーザーに付与できます。Cloud IAM には、カスタムの役割を作成して管理するための UI と API が用意されています。</p>
<p><strong>重要なポイント:</strong> カスタムの役割を使用すると、組織内のユーザー アカウントとサービス アカウントで目的の機能を使用するために必要な権限のみが付与されるように、最小限の権限の原則を適用できます。</p>
<aside>
<strong>注:</strong> カスタムの役割は、組織レベルとプロジェクト レベルで作成できますが、フォルダレベルで作成することはできません。

</aside>
<p>カスタムの役割は、使用可能な Cloud IAM 権限を組み合わせて作成します。ユーザーは権限に応じて、Google Cloud Platform のリソースで特定の操作を実行できます。</p>
<p>Cloud IAM の環境では、権限が次の形式で表されます。</p>
<pre><code>&lt;service&gt;.&lt;resource&gt;.&lt;verb&gt;&#x000A;</code></pre>
<p>たとえば、<code>compute.instances.list</code> 権限を持つユーザーは所有する Google Compute Engine インスタンスを一覧表示でき、<code>compute.instances.stop</code> 権限を持つユーザーは VM を停止できます。</p>
<p>通常、権限は REST メソッドと一対一で対応しています。つまり、GCP の各サービスには、各 REST メソッドに対して関連付けられている権限があります。メソッドを呼び出すには、呼び出し元にその権限が必要です。たとえば、<code>topic.publish()</code> の呼び出し元には <code>pubsub.topics.publish</code> 権限が必要です。</p>
<p>カスタムの役割は、役割やリソースを所有するのと同じプロジェクトまたは組織のポリシーで権限を付与するためにのみ使用できます。あるプロジェクトまたは組織のカスタムの役割を、別のプロジェクトまたは組織が所有するリソースに対して付与することはできません。</p>
<h2 id="step5">必要な権限と役割</h2>
<p>カスタムの役割を作成するには、呼び出し元に <code>iam.roles.create</code> 権限が必要です。</p>
<p>オーナー以外のユーザー（組織管理者を含む）には、「組織の役割の管理者」の役割（roles/iam.organizationRoleAdmin）または「IAM 役割の管理者」の役割（roles/iam.roleAdmin）のいずれかを割り当てる必要があります。IAM セキュリティ レビュー担当者の役割（roles/iam.securityReviewer）を使用すると、カスタムの役割を表示できますが管理はできません。</p>
<p>カスタムの役割のユーザー インターフェースは、Google Cloud Platform Console の [IAM] の [役割] にあり、カスタムの役割を作成または管理する権限を持つユーザーのみが使用できます。デフォルトでは、プロジェクト オーナーのみが新しい役割を作成できます。プロジェクト オーナーは、同じプロジェクトの他のユーザーに「IAM 役割の管理者」の役割を付与することで、この機能へのアクセスを制御できます。組織の場合は、組織管理者のみが「組織の役割の管理者」の役割を付与できます。</p>
<h2 id="step6">カスタムの役割を作成する準備を行う</h2>
<p>カスタムの役割を作成する前に、次を確認しておくことをおすすめします。</p>
<ul>
<li>
<p>リソースに適用できる権限の種類</p>
</li>
<li>
<p>リソースに付与できる役割の種類</p>
</li>
<li>
<p>役割メタデータの内容</p>
</li>
</ul>
<h2 id="step7">リソースで使用可能な権限を表示する</h2>
<p>カスタムの役割を作成する前に、リソースに適用できる権限を確認しておくことをおすすめします。ここでは、gcloud コマンドライン ツール、Cloud Console、または IAM API を使用して、リソースに適用できるすべての権限のほか、階層内のその下にあるリソースを取得できます。たとえば、組織およびその組織内のプロジェクトに適用できるすべての権限を取得できます。</p>
<p>以下のコマンドを実行して、プロジェクトで利用可能な権限の一覧を確認します。</p>
<pre><code>gcloud iam list-testable-permissions //cloudresourcemanager.googleapis.com/projects/$DEVSHELL_PROJECT_ID&#x000A;</code></pre>
<p>（出力）</p>
<pre><code>name: appengine.applications.create&#x000A;stage: GA&#x000A;---&#x000A;name: appengine.applications.get&#x000A;stage: GA&#x000A;---&#x000A;name: appengine.applications.update&#x000A;stage: GA&#x000A;---&#x000A;name: appengine.instances.delete&#x000A; stage: GA&#x000A;---&#x000A;name: appengine.instances.get&#x000A;stage: GA&#x000A;---&#x000A;name: appengine.instances.list&#x000A;stage: GA&#x000A;---&#x000A;customRolesSupportLevel: TESTING&#x000A;name: appengine.memcache.addKey&#x000A;stage: BETA&#x000A;---&#x000A;customRolesSupportLevel: TESTING&#x000A;name: appengine.memcache.flush&#x000A;stage: BETA&#x000A;---&#x000A;</code></pre>
<h2 id="step8">役割メタデータを取得する</h2>
<p>カスタムの役割を作成する前に、事前定義済みの役割とカスタムの役割の両方のメタデータを取得しておくことをおすすめします。役割メタデータには、その役割の ID と権限が含まれます。メタデータを表示するには、Google Cloud Platform Console または IAM API を使用します。</p>
<p>役割メタデータを表示するには、以下のコマンドを使用します。<code>[ROLE_NAME]</code> は役割（<code>roles/viewer</code> や <code>roles/editor</code> など）に置き換えます。</p>
<pre><code>gcloud iam roles describe [ROLE_NAME]&#x000A;</code></pre>
<p><strong>出力例（roles/viewer の場合）:</strong></p>
<pre><code>description: Read access to all custom roles in the project.&#x000A;etag: AA==&#x000A;includedPermissions:&#x000A;- iam.roles.get&#x000A;- iam.roles.list&#x000A;- resourcemanager.projects.get&#x000A;- resourcemanager.projects.getIamPolicy&#x000A;...&#x000A;...&#x000A;name: roles/iam.roleViewer&#x000A;stage: GA&#x000A;title: Role Viewer&#x000A;</code></pre>
<h2 id="step9">リソースに対して付与できる役割を表示する</h2>
<p>特定のリソースに適用できるすべての役割の一覧を取得するには、<code>gcloud iam list-grantable-roles</code> コマンドを使用します。</p>
<p>次の <code>gcloud</code> コマンドを実行して、プロジェクトから付与できる役割を一覧表示します。</p>
<pre><code>gcloud iam list-grantable-roles //cloudresourcemanager.googleapis.com/projects/$DEVSHELL_PROJECT_ID&#x000A;</code></pre>
<p>出力は次のようになります。</p>
<pre><code>---&#x000A;description: Full management of App Engine apps (but not storage).&#x000A;name: roles/appengine.appAdmin&#x000A;title: App Engine Admin&#x000A;---&#x000A;description: Ability to view App Engine app status.&#x000A;name: roles/appengine.appViewer&#x000A;title: App Engine Viewer&#x000A;---&#x000A;description: Ability to view App Engine app status and deployed source code.&#x000A;name: roles/appengine.codeViewer&#x000A;title: App Engine Code Viewer&#x000A;---&#x000A;...&#x000A;...&#x000A;</code></pre>
<h2 id="step10">カスタムの役割を作成する</h2>
<p>カスタムの役割を作成するには、呼び出し元に <code>iam.roles.create</code> 権限が必要です。デフォルトでは、プロジェクトや組織のオーナーがこの権限を持ち、カスタムの役割を作成して管理できます。</p>
<p>オーナー以外のユーザー（組織の管理者など）には、「組織の役割の管理者」または「IAM 役割の管理者」の役割のいずれかを割り当てる必要があります。</p>
<p>新しいカスタムの役割を作成するには、<code>gcloud iam roles create</code> コマンドを使用します。このコマンドは次の 2 つの方法で使用できます。</p>
<ul>
<li>役割の定義が含まれる YAML ファイルを使用する</li>
<li>フラグを使用して役割の定義を指定する
（カスタムの役割を作成する場合は <code>--organization [ORGANIZATION_ID]</code> フラグまたは <code>--project [PROJECT_ID]</code> フラグを使用して、組織レベルまたはプロジェクト レベルのどちらに適用するかを指定する必要があります）。下の各例では、プロジェクト レベルでカスタムの役割を作成します。</li>
</ul>
<p>次の演習では、プロジェクト レベルでカスタム役割を作成します。</p>
<h2 id="step11">YAML ファイルを使用してカスタムの役割を作成する</h2>
<p>カスタムの役割の定義が含まれる YAML ファイルを作成します。このファイルは次のような構造にする必要があります。</p>

```yaml
title: [ROLE_TITLE]
description: [ROLE_DESCRIPTION]
stage: [LAUNCH_STAGE]
includedPermissions:
- [PERMISSION_1]
- [PERMISSION_2]
```

<p>各プレースホルダ値の詳細は、以下のとおりです。</p>
<ul>
<li>[<code>ROLE-TITLE</code>] は役割のわかりやすいタイトルです。例: <strong>Role Viewer</strong>。</li>
<li>
<code>[ROLE_DESCRIPTION]</code> は役割についての短い説明です。例: <strong>My custom role description</strong>。</li>
<li>
<code>[LAUNCH-STAGE]</code> はリリースのライフサイクルにおける役割の段階を示します。例: ALPHA、BETA、GA。</li>
<li>
<code>includedPermissions</code> はカスタムの役割に含める権限（複数可）のリストを指定します。例: <strong>iam.roles.get</strong>。</li>
</ul>
<p>次に、以下を実行して役割の定義の YAML ファイルを作成します。</p>
<pre><code>nano role-definition.yaml&#x000A;</code></pre>
<p>このカスタムの役割の定義を YAML ファイルに追加します。</p>

```yaml
title: "Role Editor"
description: "Edit access for App Versions"
stage: "ALPHA"
includedPermissions:
- appengine.versions.create
- appengine.versions.delete
```

<p>ここでファイルを保存して閉じます。</p>
<p>次の <code>gcloud</code> コマンドを実行します。</p>
<pre><code>gcloud iam roles create editor --project $DEVSHELL_PROJECT_ID \&#x000A;--file role-definition.yaml&#x000A;</code></pre>
<p>役割が正常に作成された場合、次のレスポンスが返されます。</p>
<pre><code>Created role [editor].&#x000A;description: Edit access for App Versions&#x000A;etag: BwVs4O4E3e4=&#x000A;includedPermissions:&#x000A;- appengine.versions.create&#x000A;- appengine.versions.delete&#x000A;name: projects/[PROJECT_ID]/roles/editor&#x000A;stage: ALPHA&#x000A;title: Role Editor&#x000A;</code></pre>
<h2 id="step12">フラグを使用してカスタムの役割を作成する</h2>
<p>次に、フラグを使用する方法で新しいカスタムの役割を作成します。フラグは YAML ファイルと同様の形式なので、コマンドの構築は確認できます。</p>
<p>次のように <code>gcloud</code> コマンドを実行し、フラグを指定して新しい役割を作成します。</p>
<pre><code>gcloud iam roles create viewer --project $DEVSHELL_PROJECT_ID \&#x000A;--title "Role Viewer" --description "Custom role description." \&#x000A;--permissions compute.instances.get,compute.instances.list --stage ALPHA&#x000A;</code></pre>
<p>出力例:</p>
<pre><code>Created role [viewer].&#x000A;description: Custom role description.&#x000A;etag: BwVs4PYHqYI=&#x000A;includedPermissions:&#x000A;- compute.instances.get&#x000A;- compute.instances.list&#x000A;name: projects/[PROJECT_ID]/roles/viewer&#x000A;stage: ALPHA&#x000A;title: Role Viewer&#x000A;</code></pre>
<h2 id="step13">カスタムの役割を一覧表示する</h2>
<p>プロジェクト レベルまたは組織レベルのカスタムの役割を指定して、次の <code>gcloud</code> コマンドを実行してカスタムの役割を一覧表示します。</p>
<pre><code>gcloud iam roles list --project $DEVSHELL_PROJECT_ID&#x000A;</code></pre>
<p><strong>出力例:</strong></p>
<pre><code>---&#x000A;description: Edit access for App Versions&#x000A;etag: BwVxLgrnawQ=&#x000A;name: projects/[PROJECT_ID]/roles/editor&#x000A;title: Role Editor&#x000A;---&#x000A;description: Custom role description.&#x000A;etag: BwVxLg18IQg=&#x000A;name: projects/[PROJECT_ID]/roles/viewer&#x000A;title: Role Viewer&#x000A;</code></pre>
<p>削除された役割を一覧表示するには、<code>--show-deleted</code> フラグを使用することもできます。</p>
<p>事前定義済みの役割を一覧表示するには、次の <code>gcloud</code> コマンドを実行します。</p>
<pre><code>gcloud iam roles list&#x000A;</code></pre>
<h2 id="step14">既存のカスタムの役割を編集する</h2>
<p>カスタムの役割などのリソースのメタデータの更新では、一般にリソースの現在の状態の読み取り、ローカルでのデータの更新、変更されたデータの送信と書き込みが行われます。このような処理では、2 つ以上の独立したプロセスが一連の操作を同時に試行する場合に競合が発生することがあります。</p>
<p>たとえば、プロジェクトの 2 人のオーナーが、1 つの役割に対して相反する変更を同時に行うと、一部の変更が失敗する可能性があります。</p>
<p>Cloud IAM では、カスタムの役割の <code>etag</code> プロパティを使用してこの問題を解決します。このプロパティは、カスタムの役割が最後のリクエスト以降に変更されているかどうかを確認するために使用されます。etag 値を保持している Cloud IAM にリクエストを出すと、Cloud IAM はリクエスト内の etag 値と、カスタムの役割に関連付けられている既存の etag 値を比較します。etag 値が一致した場合にのみ変更を書き込みます。</p>
<p>カスタムの役割を更新するには、<code>gcloud iam roles update</code> コマンドを使用します。このコマンドは次の 2 つの方法で使用できます。</p>
<ul>
<li>更新された役割の定義が含まれる YAML ファイルを使用する</li>
<li>フラグを使用して、更新された役割の定義を指定する</li>
</ul>
<p>カスタムの役割を更新する際には、<code>--organization [ORGANIZATION-ID]</code> フラグまたは <code>--project [PROJECT-ID]</code> フラグを使用して、組織レベルまたはプロジェクト レベルのどちらに適用するかを指定する必要があります。下の各例では、プロジェクト レベルでカスタムの役割を作成します。</p>
<p><code>describe</code> コマンドは役割の定義を返し、その定義には役割の現在のバージョンを一意に特定する etag 値が含まれます。役割の同時変更が上書きされないように、更新された役割の定義に etag 値を指定する必要があります。</p>
<h2 id="step15">YAML ファイルを使用してカスタムの役割を更新する</h2>
<p>次の <code>gcloud</code> コマンドを実行して、役割の最新の定義を取得します。<code>[ROLE_ID]</code> は <strong>editor</strong> に置き換えます。</p>
<pre><code>gcloud iam roles describe [ROLE_ID] --project $DEVSHELL_PROJECT_ID&#x000A;</code></pre>
<p><code>describe</code> コマンドは次の出力を返します。</p>
<pre><code>description: [ROLE_DESCRIPTION]&#x000A;etag: [ETAG_VALUE]&#x000A;includedPermissions:&#x000A;- [PERMISSION_1]&#x000A;- [PERMISSION_2]&#x000A;name: [ROLE_ID]&#x000A;stage: [LAUNCH_STAGE]&#x000A;title: [ROLE_TITLE]&#x000A;</code></pre>
<p>出力値を基に新しい YAML ファイルを作成するので、このコマンドの出力をコピーします。</p>
<p>エディタで <code>new-role-definition.yaml</code> ファイルを作成します。</p>
<pre><code>nano new-role-definition.yaml&#x000A;</code></pre>
<p>最後のコマンドの出力値を貼り付けて、<code>includedPermissions</code> に以下の 2 つの権限を追加します。</p>
<pre><code>- storage.buckets.get&#x000A;- storage.buckets.list&#x000A;</code></pre>
<p>完了すると、YAML ファイルは次のようになります。</p>

```yaml
description: Edit access for App Versions
etag: BwVxIAbRq_I=
includedPermissions:
- appengine.versions.create
- appengine.versions.delete
- storage.buckets.get
- storage.buckets.list
name: projects/[PROJECT_ID]/roles/editor
stage: ALPHA
title: Role Editor
```

<p>ファイルを保存して閉じます。</p>
<p>次に <code>update</code> コマンドを使って役割を更新します。<code>[ROLE_ID]</code> を <strong>editor</strong> に置き換えて、次の <code>gcloud</code> コマンドを実行します。</p>
<pre><code>gcloud iam roles update [ROLE_ID] --project $DEVSHELL_PROJECT_ID \&#x000A;--file new-role-definition.yaml&#x000A; </code></pre>
<p>役割が正常に更新された場合、次のレスポンスが返されます。</p>
<pre><code>description: Edit access for App Versions&#x000A;etag: BwVxIBjfN3M=&#x000A;includedPermissions:&#x000A;- appengine.versions.create&#x000A;- appengine.versions.delete&#x000A;- storage.buckets.get&#x000A;- storage.buckets.list&#x000A;name: projects/[PROJECT_ID]/roles/editor&#x000A;stage: ALPHA&#x000A;title: Role Editor&#x000A;</code></pre>
<h2 id="step16">フラグを使用してカスタムの役割を更新する</h2>
<p>役割の定義の各部分は、対応するフラグを使用して更新できます。使用可能なすべてのフラグの一覧については、<a href="https://cloud.google.com/sdk/gcloud/reference/iam/roles/update">gcloud iam roles update</a> トピックをご覧ください。</p>
<p>次のフラグを使用して、権限を追加または削除できます。</p>
<ul>
<li>
<code>--add-permissions [PERMISSIONS]: </code>権限（複数の場合はカンマで区切る）を役割に追加します。</li>
<li>
<code>--remove-permissions [PERMISSIONS]: </code>権限（複数の場合はカンマで区切る）を役割から削除します。</li>
</ul>
<p>または、<code>--permissions [PERMISSIONS]</code> フラグを使用して新しい権限を指定することもできます。権限のカンマ区切りのリストを指定すると、既存の権限のリストが置き換えられます。</p>
<p>次のように <code>gcloud</code> コマンドを実行し、フラグを指定して <strong>viewer</strong> 役割に権限を追加します。</p>
<pre><code>gcloud iam roles update viewer --project $DEVSHELL_PROJECT_ID \&#x000A;--add-permissions storage.buckets.get,storage.buckets.list&#x000A;</code></pre>
<p>役割が正常に更新された場合、次のレスポンスが返されます。</p>
<pre><code>description: Custom role description.&#x000A;etag: BwVxLi4wTvk=&#x000A;includedPermissions:&#x000A;- compute.instances.get&#x000A;- compute.instances.list&#x000A;- storage.buckets.get&#x000A;- storage.buckets.list&#x000A;name: projects/[PROJECT_ID]/roles/viewer&#x000A;stage: ALPHA&#x000A;title: Role Viewer&#x000A;</code></pre>
<h2 id="step17">カスタムの役割を無効化する</h2>
<p>役割を無効にすると、その役割に関連するポリシー バインディングはすべて無効になります。つまり、ユーザーに役割を付与しても、その役割の権限は付与されません。</p>
<p>既存のカスタムの役割を無効にする最も簡単な方法は、<code>--stage</code> フラグを使用して役割を DISABLED に設定する方法です。</p>
<p>次の <code>gcloud</code> コマンドを実行して <strong>viewer</strong> 役割を無効にします。</p>
<pre><code>gcloud iam roles update viewer --project $DEVSHELL_PROJECT_ID \&#x000A;--stage DISABLED&#x000A;</code></pre>
<p>役割が正常に更新された場合、次のレスポンスが返されます。</p>
<pre><code>description: Custom role description.&#x000A;etag: BwVxLkIYHrQ=&#x000A;includedPermissions:&#x000A;- compute.instances.get&#x000A;- compute.instances.list&#x000A;- storage.buckets.get&#x000A;- storage.buckets.list&#x000A;name: projects/[PROJECT_ID]/roles/viewer&#x000A;stage: DISABLED&#x000A;title: Role Viewer&#x000A;</code></pre>
<h2 id="step18">カスタムの役割を削除する</h2>
<p>カスタムの役割を削除するには、<code>gcloud iam roles delete</code> コマンドを使用します。削除された役割は無効になり、新しい IAM ポリシー バインディングの作成に使用できなくなります。</p>
<pre><code>gcloud iam roles delete viewer --project $DEVSHELL_PROJECT_ID&#x000A;</code></pre>
<p><strong>出力例:</strong></p>
<pre><code>description: Custom role description.&#x000A;etag: BwVxLkf_epw=&#x000A;includedPermissions:&#x000A;- compute.instances.get&#x000A;- compute.instances.list&#x000A;- storage.buckets.get&#x000A;- storage.buckets.list&#x000A;name: projects/[PROJECT_ID]/roles/viewer&#x000A;stage: DISABLED&#x000A;title: Role Viewer&#x000A;</code></pre>
<p>役割が削除された後、既存のバインディングは残りますが無効になります。7 日以内であれば、役割の削除を取り消すことができます。7 日間後に役割の完全削除プロセスが開始され、そのプロセスが 30 日間続きます。37 日後に、役割 ID は再び使用可能になります。</p>
<aside>
<strong>注: </strong>役割を段階的に廃止する場合は、<strong>role.stage</strong> プロパティを <strong>DEPRECATED</strong> に変更し、<code>deprecation_message</code> を設定して、使用する必要がある代替の役割や詳細情報の入手場所をユーザーに知らせます。

</aside>
<h2 id="step19">カスタムの役割の削除を取り消す</h2>
<p>7 日以内ならば、役割を元に戻すことができます。削除された役割は <strong>DISABLED</strong> 状態にあります。以下のように <code>--stage</code> フラグを更新することで再び利用可能にすることができます。</p>
<pre><code>gcloud iam roles undelete viewer --project $DEVSHELL_PROJECT_ID&#x000A;</code></pre>

# Securing Google Cloud with CFT Scorecard
<h2 id="step2">Overview</h2>
<p>CFT Scorecard is an open-sourced command line client of  <a href="https://github.com/forseti-security/policy-library/blob/master/docs/user_guide.md">Forseti Config Validator</a> and part of the broader  <a href="https://github.com/GoogleCloudPlatform/cloud-foundation-toolkit">Cloud Foundation Toolkit</a>. It provides visibility into misconfigurations and violations of an established set of standards for Google Cloud resources, projects, folders, or even organizations.</p>
<p>There are over 86 distinct Google Cloud resource types, and they're growing.  With the move to public cloud, it is easier than ever to federate cloud operations and resource deployment out to many individuals.  Along with federation and agility in the deployment of infrastructure, resources, and policy, it has become increasingly difficult to keep policies and standards in order.</p>
<p>In this lab you will configure <a href="https://github.com/GoogleCloudPlatform/cloud-foundation-toolkit/blob/master/cli/docs/scorecard.md">CFT Scorecard</a> to improve visibility into a Google Cloud project and detect misconfigurations.</p>
<h3>What will you do in this Lab?</h3>
<p>This lab highlights the challenges with using the cloud with multiple concurrent users.  You will enable the CFT Scorecard and extend its resource configuration monitoring and violation detection capabilities through integration with  <a href="https://cloud.google.com/asset-inventory/docs/overview">Cloud Asset Inventory</a> and the open-sourced  <a href="https://github.com/forseti-security/policy-library">Policy Library</a>.  You will set up the tooling for detecting misconfigurations and over-exposed resources while still allowing other individuals within your team, and ultimately the Google Cloud project, to be agile within established policies.</p>
<h4>Topics Covered</h4>
<ul>
<li>Setting up CFT Scorecard</li>
<li>Running a CFT Scorecard assessment</li>
<li>Adding new CFT Scorecard policy</li>
</ul>
<p><img alt="df265ec3493f04c6.png" src="https://cdn.qwiklabs.com/n2I54wqTe%2BlAHa1eadcQYdQrdumXyKQfpXz2co8Hnik%3D"></p>
<h2 id="step3">Scenario</h2>
<p>Imagine you are the Technical Lead of a 3 person team.  Your remote teammates, Alice and Bob, are working very closely with you and deploying many resources into the same shared Google Cloud project as you.  After a few weeks of working together, you start to notice a few red flags.  You soon discover that both Alice and Bob have cut corners and introduced project configurations that you consider misconfiguration.  One misconfiguration exposed a Google Cloud Storage bucket publicly.  This is merely one misconfiguration that you have uncovered, but you fear that there could be many more.</p>
<p>After doing a quick Google search, you come across the Cloud Foundation Toolkit (CFT) Scorecard CLI utility.  After a quick read, you decide this can help you administer policies into your Google Cloud environment and determine where misconfigurations are occurring. You decide to give it a try.</p>
<h2 id="step4">Setup and requirements</h2>
<h4>Before you click the Start Lab button</h4>
<p>Read these instructions. Labs are timed and you cannot pause them. The timer, which starts when you click <strong>Start Lab</strong>, shows how long Google Cloud resources will be made available to you.</p>
<p>This Qwiklabs hands-on lab lets you do the lab activities yourself in a real cloud environment, not in a simulation or demo environment. It does so by giving you new, temporary credentials that you use to sign in and access Google Cloud for the duration of the lab.</p>

<h3>Setup environment</h3>
<ol>
<li>
<p>You open Cloud Shell and set a couple of environment variables to begin.</p>
</li>
</ol>
<pre><code class="language-bash prettyprint">export GOOGLE_PROJECT=$DEVSHELL_PROJECT_ID&#x000A;export CAI_BUCKET_NAME=cai-$GOOGLE_PROJECT&#x000A;</code></pre>
<ol start="2">
<li>After reading the docs, you understand that CFT scorecard has two dependencies:</li>
</ol>
<ul>
<li>
<p>Cloud Asset Inventory</p>
</li>
<li>
<p>Policy Library
Proceed to enable Cloud Asset API in your project:</p>
</li>
</ul>
<pre><code class="language-bash prettyprint">gcloud services enable cloudasset.googleapis.com \&#x000A;    --project $GOOGLE_PROJECT&#x000A;</code></pre>
<ol start="3">
<li>
<p>Clone the Policy Library:</p>
</li>
</ol>
<pre><code class="language-bash prettyprint">git clone https://github.com/forseti-security/policy-library.git&#x000A;</code></pre>
<ol start="4">
<li>
<p>You realize Policy Library enforces policies that are located in the policy-library/policies/constraints folder, in which case you can copy a sample policy from the samples directory into the constraints directory.</p>
</li>
</ol>
<pre><code class="language-bash prettyprint">cp policy-library/samples/storage_blacklist_public.yaml policy-library/policies/constraints/&#x000A;</code></pre>
<ol start="5">
<li>
<p>Create the bucket that will hold the data that Cloud Asset Inventory (CAI) will export:</p>
</li>
</ol>
<pre><code class="language-bash prettyprint">gsutil mb -l us-central1 -p $GOOGLE_PROJECT gs://$CAI_BUCKET_NAME&#x000A;</code></pre>

<h3>Collect the data using Cloud Asset Inventory (CAI)</h3>
<p>Now that you have set up your environment, start collecting the data for CFT Scorecard.</p>
<ol start="6">
<li>As mentioned earlier, input to CFT Scorecard is resource and IAM data, and the policy-library folder.</li>
</ol>
<p>You'll need to use CAI to generate the resource and IAM policy information for the project.</p>
<p>Use the command below to create the data:</p>
<pre><code class="language-bash prettyprint"># Export resource data&#x000A;gcloud asset export \&#x000A;    --output-path=gs://$CAI_BUCKET_NAME/resource_inventory.json \&#x000A;    --content-type=resource \&#x000A;    --project=$GOOGLE_PROJECT&#x000A;&#x000A;# Export IAM data&#x000A;gcloud asset export \&#x000A;    --output-path=gs://$CAI_BUCKET_NAME/iam_inventory.json \&#x000A;    --content-type=iam-policy \&#x000A;    --project=$GOOGLE_PROJECT&#x000A;</code></pre>
<p>(Example Output)</p>
<pre><code class="language-bash prettyprint">Export in progress for root asset [projects/qwiklabs-gcp-01-68169ed6dd00].&#x000A;Use [gcloud asset operations describe projects/97186664469/operations/ExportAssets/RESOURCE/2295255602305764396] to check the status of the operation.&#x000A;&#x000A;Export in progress for root asset [projects/qwiklabs-gcp-01-68169ed6dd00].&#x000A;Use [gcloud asset operations describe projects/97186664469/operations/ExportAssets/IAM_POLICY/11771734913762837428] to check the status of the operation.&#x000A;</code></pre>
<ol start="7">
<li>Ensure CAI has finished data collection. Look at the output from the previous command and use the provided <code>gcloud asset operations describe</code> from the output of the above two commands to verify these operations have finished.</li>
</ol>

<h3>Analyze the CAI data with CFT Scorecard</h3>
<ol start="8">
<li>
<p>You need to download the CFT scorecard application and make it executable:</p>
</li>
</ol>
<pre><code class="language-bash prettyprint">curl -o cft https://storage.googleapis.com/cft-cli/latest/cft-linux-amd64&#x000A;# make executable&#x000A;chmod +x cft&#x000A;</code></pre>
<ol start="9">
<li>
<p>Now that you have configured everything, go ahead and run the CFT scorecard application:</p>
</li>
</ol>
<pre><code class="language-bash prettyprint">./cft scorecard --policy-path=policy-library/ --bucket=$CAI_BUCKET_NAME&#x000A;</code></pre>
<p>(Example Output)</p>
<pre><code class="language-bash prettyprint">Generating CFT scorecard&#x000A;1 total issues found&#x000A;Operational Efficiency: 0 issues found&#x000A;----------&#x000A;Security: 1 issues found&#x000A;----------&#x000A;blacklist_public_users: 1 issues&#x000A;- //storage.googleapis.com/fun-bucket-qwiklabs-gcp-00-2d8ed2a5cc0e is publicly accessable&#x000A;Reliability: 0 issues found&#x000A;----------&#x000A;Other: 0 issues found&#x000A;----------&#x000A;</code></pre>
<p>This was the public bucket you identified earlier.</p>
<h3>Adding more constraints to CFT Scorecard</h3>
<ol start="10">
<li>
<p>You forgot about IAM! Add the following constraint to ensure you are entirely aware who has the <code>roles/owner</code> role aside from your whitelisted user:</p>
</li>
</ol>
<pre><code class="language-bash prettyprint"># Add a new policy to blacklist the IAM Owner Role&#x000A;cat &gt; policy-library/policies/constraints/iam_whitelist_owner.yaml &lt;&lt; EOF&#x000A;apiVersion: constraints.gatekeeper.sh/v1alpha1&#x000A;kind: GCPIAMAllowedBindingsConstraintV1&#x000A;metadata:&#x000A;  name: whitelist_owner&#x000A;  annotations:&#x000A;    description: List any users granted Owner&#x000A;spec:&#x000A;  severity: high&#x000A;  match:&#x000A;    target: ["organization/*"]&#x000A;    exclude: []&#x000A;  parameters:&#x000A;    mode: whitelist&#x000A;    assetType: cloudresourcemanager.googleapis.com/Project&#x000A;    role: roles/owner&#x000A;    members:&#x000A;    - "serviceAccount:admiral@qwiklabs-services-prod.iam.gserviceaccount.com"&#x000A;EOF&#x000A;</code></pre>
<ol start="11">
<li>
<p>Rerun CFT scorecard:</p>
</li>
</ol>
<pre><code class="language-bash prettyprint">./cft scorecard --policy-path=policy-library/ --bucket=$CAI_BUCKET_NAME&#x000A;</code></pre>
<p>Ok, it all looks clean, but you decide to look at <code>roles/editor</code> too.</p>
<ol start="12">
<li>
<p>Set two extra variables to help with the new constraint creation:</p>
</li>
</ol>
<pre><code class="language-bash prettyprint">export USER_ACCOUNT="$(gcloud config get-value core/account)"&#x000A;export PROJECT_NUMBER=$(gcloud projects describe $GOOGLE_PROJECT --format="get(projectNumber)")&#x000A;</code></pre>
<ol start="13">
<li>
<p>Create the following constraint that will whitelist all the valid accounts:</p>
</li>
</ol>
<pre><code class="language-bash prettyprint"># Add a new policy to whitelist the IAM Editor Role&#x000A;cat &gt; policy-library/policies/constraints/iam_identify_outside_editors.yaml &lt;&lt; EOF&#x000A;apiVersion: constraints.gatekeeper.sh/v1alpha1&#x000A;kind: GCPIAMAllowedBindingsConstraintV1&#x000A;metadata:&#x000A;  name: identify_outside_editors&#x000A;  annotations:&#x000A;    description: list any users outside the organization granted Editor&#x000A;spec:&#x000A;  severity: high&#x000A;  match:&#x000A;    target: ["organization/*"]&#x000A;    exclude: []&#x000A;  parameters:&#x000A;    mode: whitelist&#x000A;    assetType: cloudresourcemanager.googleapis.com/Project&#x000A;    role: roles/editor&#x000A;    members:&#x000A;    - "user:$USER_ACCOUNT"&#x000A;    - "serviceAccount:*$PROJECT_NUMBER*gserviceaccount.com"&#x000A;    - "serviceAccount:$GOOGLE_PROJECT*gserviceaccount.com"&#x000A;EOF&#x000A;</code></pre>
<ol start="14">
<li>
<p>Rerun CFT scorecard:</p>
</li>
</ol>
<pre><code class="language-bash prettyprint">./cft scorecard --policy-path=policy-library/ --bucket=$CAI_BUCKET_NAME&#x000A;</code></pre>
<p>(Example Output)</p>
<pre><code class="language-bash prettyprint">Generating CFT scorecard&#x000A;2 total issues found&#x000A;Reliability: 0 issues found&#x000A;----------&#x000A;Other: 1 issues found&#x000A;----------&#x000A;identify_outside_editors: 1 issues&#x000A;- IAM policy for //cloudresourcemanager.googleapis.com/projects/1044418630080 grants roles/editor to user:qwiklabs.lab.user@gmail.com&#x000A;Operational Efficiency: 0 issues found&#x000A;----------&#x000A;Security: 1 issues found&#x000A;----------&#x000A;blacklist_public_users: 1 issues&#x000A;- //storage.googleapis.com/fun-bucket-qwiklabs-gcp-00-2d8ed2a5cc0e is publicly accessable&#x000A;</code></pre>
<p>You should now see an editor who is decidedly not in your organization.  Time to talk to Alice and Bob.</p>

# VPC ネットワーク ピアリング
<h2 id="step2">概要</h2>
<p>Google Cloud Platform（GCP）の Virtual Private Cloud（VPC）ネットワーク ピアリングを利用すると、同じプロジェクトや組織に属しているかどうかにかかわらず、2 つの VPC ネットワーク間でプライベート空間での接続を行うことができます。</p>
<p>VPC ネットワーク ピアリングでは、GCP で SaaS（Software-as-a-Service）エコシステムを構築できます。組織内や組織間の異なる VPC ネットワークでサービスを非公開で提供し、プライベート空間で通信を行うことが可能です。</p>
<p>VPC ネットワーク ピアリングは、次のような場合に役立ちます。</p>
<ul>
<li>組織に複数のネットワーク管理ドメインがある場合</li>
<li>他の組織とピアリングを行う場合</li>
</ul>
<p>組織内に複数のネットワーク管理ドメインが存在する場合、VPC ネットワーク ピアリングを利用すると、プライベート空間で VPC ネットワーク全体にサービスを提供できます。他の組織にサービスを提供する場合、VPC ネットワーク ピアリングを使用すると、他の組織がプライベート空間でこのようなサービスを利用できるようになります。組織を超えてサービスを提供できれば、他の企業にサービスを提供することが可能になります。また、合併吸収の結果あるいは独自の要件のために、自社内に複数の組織ノードが存在する場合にも便利です。</p>
<p>VPC ネットワーク ピアリングは、ネットワークを接続する場合に外部 IP アドレスや VPN を使用するより次の点で優れています。</p>
<ul>
<li>
<p><strong>ネットワークのレイテンシ: </strong>プライベート ネットワークは、パブリック IP ネットワークよりもレイテンシが小さくなります。</p>
</li>
<li>
<p><strong>ネットワークのセキュリティ: </strong>サービス オーナーは、サービスをインターネットに公開して関連するリスクに対処する必要がありません。</p>
</li>
<li>
<p><strong>ネットワーク コスト: </strong>ネットワークがピアリングされている場合、内部 IP を使用して通信することで、GCP 下りの帯域幅のコストを節約できます。この場合でも、通常のネットワーク料金はすべてのトラフィックに適用されます。</p>
</li>
</ul>

<h2 id="step4">VPC ネットワーク ピアリングを設定する</h2>
<p>同じ組織ノード内の 1 つのネットワークで、同じまたは異なるプロジェクト内の他の VPC ネットワークからアクセス可能であることが必要なサービスをホストする場合があります。</p>
<p>また、組織によってはサードパーティのサービスへのアクセスが必要になる場合もあります。</p>
<p>GCP 全体で重複するプロジェクト名はないため、ピアリングの設定時に組織を指定する必要はありません。GCP は、プロジェクト名に基づいて組織を識別します。</p>
<h3>プロジェクトでカスタム ネットワークを作成する</h3>
<p>このラボでは 2 つのプロジェクト（最初のプロジェクト A と 2 番目のプロジェクト B）がプロビジョニングされています。</p>
<p>2 つのプロジェクトを管理するには、<strong>+</strong> アイコンをクリックして新しい Cloud Shell を起動します。</p>
<p>2 番目の Cloud Shell で、次のコマンドを実行してプロジェクト ID を設定します。その際に &lt;PROJECT_ID2&gt; は、ラボを開始した Qwiklabs ページの 2 番目のプロジェクトの GCP プロジェクト ID で置き換えます。</p>
<pre><code>gcloud config set project &lt;PROJECT_ID2&gt;&#x000A;</code></pre>
<h4>Project-A:</h4>
<p>最初の Cloud Shell に戻り、カスタム ネットワークを作成します。</p>
<pre><code>gcloud compute networks create network-a --subnet-mode custom&#x000A;</code></pre>
<p>次のコマンドを実行して、この VPC 内にサブネットを作成し、リージョンと IP 範囲を指定します。</p>
<pre><code>gcloud compute networks subnets create network-a-central --network network-a \&#x000A;    --range 10.0.0.0/16 --region us-central1&#x000A;</code></pre>
<p>VM インスタンスを作成します。</p>
<pre><code>gcloud compute instances create vm-a --zone us-central1-a --network network-a --subnet network-a-central&#x000A;</code></pre>
<p>次のコマンドを実行して SSH と <code>icmp</code> を有効にします。これは、接続テスト中に VM と通信するためのセキュアな shell が必要になるためです。</p>
<pre><code>gcloud compute firewall-rules create network-a-fw --network network-a --allow tcp:22,icmp&#x000A;</code></pre>
<p>次に、同じ方法で project-b をセットアップします。</p>
<h4>Project-B:</h4>
<p>2 つ目の Cloud Shell に切り替えて、カスタム ネットワークを作成します。</p>
<pre><code>gcloud compute networks create network-b --subnet-mode custom&#x000A;</code></pre>
<p>次のコマンドを実行して、この VPC 内にサブネットを作成し、リージョンと IP 範囲を指定します。</p>
<pre><code>gcloud compute networks subnets create network-b-central --network network-b \&#x000A;    --range 10.8.0.0/16 --region us-central1&#x000A;</code></pre>
<p>VM インスタンスを作成します。</p>
<pre><code>gcloud compute instances create vm-b --zone us-central1-a --network network-b --subnet network-b-central&#x000A;</code></pre>
<p>次のコマンドを実行して SSH と <code>icmp</code> を有効にします。これは、接続テスト中に VM と通信するためのセキュアな shell が必要になるためです。</p>
<pre><code>gcloud compute firewall-rules create network-b-fw --network network-b --allow tcp:22,icmp&#x000A;</code></pre>
<h2 id="step5">VPC ネットワーク ピアリング セッションを設定する</h2>
<p>たとえば、組織が project-A の network-A と project-B の network-B の間で VPC ネットワーク ピアリングを確立する必要があるとします。この場合は、network-A と network-B の管理者がそれぞれのネットワークでピアリングを設定する必要があります。</p>
<h3>network-a を network-b とピアリングする</h3>
<p><img alt="network_peering_01.png" src="https://cdn.qwiklabs.com/i6fwOoVXTt4J7oToas%2BFV61tUgLuDiaw5y7zGEnr6lU%3D"></p>
<p>設定を適用する前に、コンソールで正しいプロジェクトを選択する必要があります。そのためには、画面上部の GCP プロジェクト ID の横にある下矢印をクリックしてから、該当するプロジェクトの ID を選択します。</p>
<p><img alt="Console_choose_project.png" src="https://cdn.qwiklabs.com/drUmdZs%2FLDlS23RnHLQq%2BeHorjM3n%2B2ulUTsus0L%2FRc%3D"></p>
<p><strong>Project-A</strong></p>
<p>Google Cloud Platform Console の左側のメニューで、[ネットワーキング] セクションから [<code>VPC ネットワーク</code>] &gt; [<code>VPC ネットワーク ピアリング</code>] をクリックして [<code>VPC ネットワーク ピアリング</code>] に移動します。次に以下を実行します。</p>
<ol>
<li>[<strong>接続を作成</strong>] をクリックします。</li>
<li>[<strong>続行</strong>] をクリックします。</li>
<li>[<strong>名前</strong>] に「peer-ab」と入力します。</li>
<li>[<strong>Your VPC network</strong>] で、ピアリングするネットワーク（network-a）を選択します。</li>
<li>[<strong>ピアリングした VPC ネットワーク</strong>] で [<strong>別のプロジェクト</strong>] を選択します。</li>
<li>[<strong>プロジェクト ID</strong>] は、2 番目のプロジェクトの ID で置き換えます。</li>
<li>[<strong>VPC ネットワークの名前</strong>]（network-b）を入力します。</li>
<li>[<strong>作成</strong>] をクリックします。</li>
</ol>
<p>この段階では project-b の network-b に一致する構成がないため、ピアリング状態は無効のままです。</p>
<p>出力例:</p>
<p><img alt="waiting_peering.png" src="https://cdn.qwiklabs.com/eNgJkqRkHoqR8Z1YVIS9xAJxJVwnbxzoYj0ny6UsP%2Fs%3D"></p>
<h3>network-b を network-a とピアリングする</h3>
<p><img alt="network_peering_02.png" src="https://cdn.qwiklabs.com/u1lfqiOZTxehsnL%2FYaYF7Xg0XA%2FFXsq8YubRiAYwx4w%3D"></p>
<p><strong>注: </strong>コンソールで 2 番目のプロジェクトに切り替えます。</p>
<p><strong>Project-B</strong></p>
<ol>
<li>[<strong>接続を作成</strong>] をクリックします。</li>
<li>[<strong>続行</strong>] をクリックします。</li>
<li>[<strong>名前</strong>] に「peer-ba」と入力します。</li>
<li>[<strong>Your VPC network</strong>] で、ピアリングするネットワーク（network-b）を選択します。</li>
<li>同じプロジェクト内でピアリングする場合を除き、[<strong>ピアリングした VPC ネットワーク</strong>] で [<strong>別のプロジェクト</strong>] を選択します。</li>
<li>最初のプロジェクトの [<strong>プロジェクト ID</strong>] を指定します。</li>
<li>[<strong>VPC ネットワークの名前</strong>]（network-a）を指定します。</li>
<li>[<strong>作成</strong>] をクリックします。</li>
</ol>
<p>出力例:</p>
<p><img alt="peer_success" src="https://cdn.qwiklabs.com/rq%2FOiYDCXEYdnpBKZGe49cfgyh9G3tXpsl%2Bp3uWao%2FE%3D"></p>
<p>VPC ネットワーク ピアリングが有効になり、ルートが交換されます。
ピアリングが有効状態になるとすぐに次のトラフィック フローが設定されます。</p>
<ul>
<li>ピアリングされたネットワークの VM インスタンス間: フルメッシュ接続</li>
<li>一方のネットワークの VM インスタンスから、ピアリングされたネットワークの内部負荷分散エンドポイント</li>
</ul>
<p><img alt="network_peering_03.png" src="https://cdn.qwiklabs.com/a3mCwnSLHBiYoGJjFqeP4kgj7HgaY87W9CevTJR0eK4%3D"></p>
<p>VPC ネットワーク ピア間で、ピアリングされたネットワークの CIDR プレフィックスへのルートが公開されます。これらのルートは、有効なピアリング用に生成される暗黙的なルートです。対応するルートリソースはありません。次のコマンドを実行すると、project-A のすべての VPC ネットワークのルートが一覧表示されます。</p>
<pre><code>gcloud compute routes list --project &lt;FIRST_PROJECT_ID&gt;&#x000A;&#x000A;</code></pre>
<p>出力例:</p>
<pre><code>NAME                              NETWORK    DEST_RANGE     NEXT_HOP                  PRIORITY&#x000A;default-route-2a865a00fa31d5df    network-A  0.0.0.0/0      default-internet-gateway  1000&#x000A;default-route-8af4732e693eae27    network-A  10.0.0.0/16                              1000&#x000A;peering-route-4732ee69e3ecab41    network-A  10.8.0.0/16    peer-ab                   1000&#x000A;&#x000A;</code></pre>
<h2 id="step6">接続テスト</h2>
<h3>Project-A</h3>
<p>VM インスタンス コンソールに移動します。
<strong>ナビゲーション メニュー</strong> &gt; [<strong>Compute Engine</strong>] &gt; [<strong>VM インスタンス</strong>] の順にクリックします。</p>
<p><code>vm-a</code> の <strong>INTERNAL_IP</strong> をコピーします。</p>
<h3>Project-B</h3>
<p>[<strong>Product &amp; services</strong>] &gt; [<strong>Compute</strong>] &gt; [<strong>Compute Engine</strong>] &gt; [<strong>VM インスタンス</strong>] の順にクリックします。</p>
<p><code>vm-b</code> インスタンスに SSH 接続します。</p>
<p><code>vm-b</code> の SSH Shell で、<code>&lt;INTERNAL_IP_OF_VM_A&gt;</code> を vm-a のインスタンスの INTERNAL_IP に置き換えて次のコマンドを実行します。</p>
<pre><code>ping -c 5 &lt;INTERNAL_IP_OF_VM_A&gt;&#x000A;</code></pre>
<p>出力例:</p>
<pre><code>PING 10.8.0.2 (10.8.0.2) 56(84) bytes of data.&#x000A;64 bytes from 10.8.0.2: icmp_seq=1 ttl=64 time=1.07 ms&#x000A;64 bytes from 10.8.0.2: icmp_seq=2 ttl=64 time=0.364 ms&#x000A;64 bytes from 10.8.0.2: icmp_seq=3 ttl=64 time=0.205 ms&#x000A;64 bytes from 10.8.0.2: icmp_seq=4 ttl=64 time=0.216 ms&#x000A;64 bytes from 10.8.0.2: icmp_seq=5 ttl=64 time=0.164 ms&#x000A;&#x000A;--- 10.8.0.2 ping statistics ---&#x000A;5 packets transmitted, 5 received, 0% packet loss, time 4065ms&#x000A;rtt min/avg/max/mdev = 0.164/0.404/1.072/0.340 ms&#x000A;</code></pre>
<p>クラウド環境にあるプロジェクト間で VPC ピアリングを設定する方法を学びました。</p>

# ユーザー認証: Identity-Aware Proxy
**⭐️これはけっこう便利な気がする！**
<h2 id="step2">概要</h2>
<p>このラボでは、Google App Engine を使用して簡単なウェブ アプリケーションを構築し、Identity-Aware Proxy（IAP）を使用して、そのアプリケーションへのアクセスを制限したり、ユーザー ID 情報をそのアプリケーションに提供したりするさまざまな方法を学習します。構築するアプリの機能は次のとおりです。</p>
<ul>
<li>
<p>ウェルカム ページを表示する</p>
</li>
<li>
<p>IAP が提供したユーザー ID 情報にアクセスする</p>
</li>
<li>
<p>暗号化検証を使用してユーザー ID 情報のなりすましを防ぐ</p>
</li>
</ul>
<h3><strong>ラボの内容</strong></h3>
<ul>
<li>
<p>Python を使用して簡単な App Engine アプリを作成してデプロイする</p>
</li>
<li>
<p>アプリへのアクセスを制限する IAP を有効または無効にする</p>
</li>
<li>
<p>ユーザー ID 情報を IAP からアプリに取り込む</p>
</li>
<li>
<p>なりすましを防ぐために IAP からの情報を暗号的に検証する</p>
</li>
</ul>
<h2 id="step3">はじめに</h2>
<p>多くの場合、ウェブアプリでのユーザー認証は必須ですが、通常はご使用のアプリで特別なプログラミングが必要となります。Google Cloud Platform アプリの場合、この役割を <a href="https://cloud.google.com/iap/">Identity-Aware Proxy</a> サービスに任せることができます。選択したユーザーのみにアクセスを制限する場合は、アプリケーションを変更する必要はありません。アプリケーションがユーザー ID を認識する必要がある場合（ユーザー設定をサーバー側で保持する場合など）、Identity-Aware Proxy では最小限のアプリケーション コードを使用してそれを実現します。</p>
<h3><strong>Identity-Aware Proxy とは</strong></h3>
<p>Identity-Aware Proxy（IAP）は Google Cloud Platform のサービスです。アプリケーションに送信されたウェブ リクエストをインターセプトし、リクエストを送信したユーザーを Google の ID サービスを使用して認証し、認証されたユーザーからのリクエストのみを通過させます。さらに、リクエスト ヘッダーを変更して認証されたユーザーに関する情報を含めることができます。</p>
<h3><strong>必要なもの</strong></h3>
<p>Python プログラミング言語の基本的な知識があれば効率的に学習できます。</p>
<p>このラボでは Google App Engine と IAP を中心に学びます。関連のない概念やコードブロックについては詳しく触れず、コードはコピーして貼るだけの状態で提供されています。</p>

<h3><strong>コードをダウンロードする</strong></h3>
<p>コマンドを入力できるように Cloud Shell のコマンドラインの領域をクリックします。Github からコードを取得して、コードフォルダに移動します。</p>
<pre><code>git clone https://github.com/googlecodelabs/user-authentication-with-iap.git&#x000A;</code></pre>
<pre><code>cd user-authentication-with-iap&#x000A;</code></pre>
<p>このフォルダには、ラボのステップごとにサブフォルダが 1 つ格納されています。ステップごとに適切なフォルダに変更します。</p>
<h2 id="step5">アプリケーションをデプロイして IAP で保護する</h2>
<p>「Hello, World」というシンプルなウェルカム ページを表示する、Python で作成した App Engine スタンダード アプリケーションをデプロイします。デプロイしてテストした後、IAP を使用してアクセスを制限します。</p>
<h3><strong>アプリケーション コードを確認する</strong></h3>
<p>以下のコマンドを使用して、メインのプロジェクト フォルダから、このステップのコードを含む <code>1-HelloWorld</code> サブフォルダに変更します。</p>
<pre><code>cd 1-HelloWorld&#x000A;</code></pre>
<p>アプリケーション コードは <code>main.py</code> ファイル内に含まれています。このコードは <a href="http://flask.pocoo.org/">Flask</a> ウェブ フレームワークを使用し、テンプレートの内容を利用してウェブ リクエストに応答します。そのテンプレート ファイルは <code>templates/index.html</code> 内にあり、このステップではプレーン HTML のみが含まれています。<code>templates/privacy.html</code> 内の 2 番目のテンプレート ファイルには、簡単なプライバシー ポリシーのサンプルが含まれています。</p>
<p>他にファイルが 2 つあります。<code>requirements.txt</code> にはアプリケーションが使用する Python ライブラリ（デフォルト以外）がすべてリストされています。<code>app.yaml</code> はこのアプリが Python App Engine アプリケーションであることを Google Cloud Platform に通知するファイルです。</p>
<p>次のような cat コマンドを使用してシェル内の各ファイルを一覧表示できます。</p>
<pre><code>cat main.py&#x000A;</code></pre>
<p>または、Cloud Shell ウィンドウの右上にある鉛筆アイコンをクリックし、Cloud Shell コードエディタを起動してコードを調べることもできます。</p>
<p>このステップでは、どのファイルも変更する必要はありません。</p>
<h3><strong>App Engine にデプロイする</strong></h3>
<p>Python 向けの App Engine スタンダード環境にアプリをデプロイします。</p>
<pre><code>gcloud app deploy&#x000A;</code></pre>
<p>「supports standard」と示されている、お近くのリージョンを選択します。</p>
<p>続行を確認するメッセージが表示されたら、「<strong>Y</strong>」を入力します。</p>
<p>数分でデプロイが完了します。<code>gcloud app browse</code> を使用すると、アプリケーションを表示できる旨を示すメッセージが表示されます。</p>
<p>ここで、次のコマンドを入力します。</p>
<pre><code>gcloud app browse&#x000A;</code></pre>
<p>表示されたリンクをクリックして新しいタブで開くか、必要な場合は手動で開いた新しいタブにコマンドをコピーします。このアプリを実行するのは今回が初めてなので、クラウド インスタンスが起動され、数秒後に次のウィンドウが表示されます。</p>
<p><img alt="7ef19a3c07c423c3.png" src="https://cdn.qwiklabs.com/BUrEJObysrNmE%2FqmU234RAj3kMiAvwOswH%2FAmSdJ%2FNY%3D"></p>
<p>インターネットに接続されていれば、どのパソコンからも同じ URL でそのウェブページにアクセスできます。アクセスはまだ制限されていません。</p>
<h3><strong>IAP を使用してアクセスを制限する</strong></h3>
<ol>
<li>Cloud Console ウィンドウで、<strong>ナビゲーション メニュー</strong> &gt; [<strong>セキュリティ</strong>] &gt; [<strong>Identity-Aware Proxy</strong>] をクリックします。</li>
</ol>
<p><img alt="88279e090e0f6ba9.png" src="https://cdn.qwiklabs.com/5q8mZgj9fTa1imV5khOG0YmVBNGXVk9ce2n07dVzxVw%3D"></p>
<p>このプロジェクトの認証オプションを有効にするのは今回が初めてなので、IAP を使用する前に OAuth 同意画面を構成するためのメッセージが表示されます。</p>
<p><img alt="4afcdecf6ca153c9.png" src="https://cdn.qwiklabs.com/b5zj1%2FXG7%2FY6kQbN%2BhSYuHpGHNgmhZRJNYnb565p918%3D"></p>
<ol start="2">
<li>[<strong>同意画面を構成</strong>] をクリックします。同意画面を構成するための新しいタブが開きます。</li>
</ol>

**internal と external を選択することになるが、ここでは external としてもとくに Google に審査申請などすることなくいけた。**

<p><img alt="ebbfa4939d1836ab.png" src="https://cdn.qwiklabs.com/dZu0mCpleXpQpJz7reYy%2B9n4BV7cIwYx0%2Bnu9oJV%2B%2F0%3D"></p>
<ol start="3">
<li>下記の必須項目に適切な値を入力してください。</li>
</ol>
<table>
<tr>
<td colspan="1" rowspan="1">
<p><strong>項目</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>値</strong></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>アプリケーション名</p>
</td>
<td colspan="1" rowspan="1">
<p>IAP Example</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>サポートメール</p>
</td>
<td colspan="1" rowspan="1">
<p><em>メールアドレス。自動的に入力されている場合があります。</em></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>承認済みドメイン</p>
</td>
<td colspan="1" rowspan="1">
<p><em>アプリケーションの URL のホスト名部分（iap-example-999999.appspot.com など）。以前に開いた Hello World ウェブページのアドレスバーで確認できます。URL の先頭の <code>https://</code> と末尾の <code>/</code> は入れないでください。</em><em></em><em></em></p>
<p><strong><em>この値を入力したら、Enter キーを押してください。</em></strong></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>[アプリケーション ホームページ] リンク</p>
</td>
<td colspan="1" rowspan="1">
<p><em>アプリの表示に使用した URL</em></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>[アプリケーション プライバシー ポリシー] リンク</p>
</td>
<td colspan="1" rowspan="1">
<p><em>アプリ内のプライバシー ページへのリンク、ホームページへのリンクの末尾に /privacy を追加したもの</em></p>
</td>
</tr>
</table>
<ol start="4">
<li>[<strong>保存</strong>] をクリックします。認証情報の作成を求めるメッセージが表示されます。このラボでは認証情報を作成する必要はないので、ブラウザタブを閉じます。</li>
<li>[Identity-Aware Proxy] に戻ってページを更新をします。保護できるリソースの一覧が表示されます。</li>
</ol>
<p>App Engine アプリの行の [IAP] 列の切り替えスイッチをクリックして、<strong>IAP</strong> をオンにします。</p>
<p><img alt="ebfaee8d6b54cc32.png" src="https://cdn.qwiklabs.com/aVwXNkbc%2Bu9Echzlxk3UzWdu9fS3YjUh20cXrqYhh8Y%3D"></p>
<ol start="6">
<li>IAP によって保護されるドメイン名が表示されます。[<strong>有効にする</strong>] をクリックします。</li>
</ol>
<p><img alt="c529b259e92a78fa.png" src="https://cdn.qwiklabs.com/wK%2FcfdJgatl3wXoIfMaQYkD%2Fnc18BkqfOPrHy2mi1w8%3D"></p>
<h3><strong>IAP がオンになっていることをテストする</strong></h3>
<ol>
<li>ブラウザタブを開き、アプリの URL に移動します。「Google でログイン」画面が開き、アプリにアクセスするためのログインが求められます。</li>
</ol>
<p><img alt="d8f4ff4fd2dce188.png" src="https://cdn.qwiklabs.com/t1A%2Fnlq81904J1ybMrXbXxm3tyHPL6LED%2FXgF%2FeqvPk%3D"></p>
<ol start="2">
<li>コンソールへのログインに使用したアカウントでログインすると、アクセス拒否の画面が表示されます。</li>
</ol>
<p><img alt="dc9b398c45bbeb6.png" src="https://cdn.qwiklabs.com/Dg%2B3nEsZCER%2FgugnifNQO9%2B5jDs3sGDHPZOn025x0Bc%3D"></p>
<p>アプリは IAP で正常に保護されていますが、どのアカウントを通過させるのかを IAP にまだ指示していません。</p>
<ol>
<li>コンソールの [Identity-Aware Proxy] ページに戻り、[App Engine] アプリの隣にあるチェックボックスをオンにすると、ページの右側にサイドバーが表示されます。</li>
</ol>
<p><img alt="eefc750e07495ec8.png" src="https://cdn.qwiklabs.com/zeicjr8naXKxcRfTgXBkObuz3zIM4ygrHNbuGir72RI%3D"></p>
<p>アクセスを許可する必要がある各メールアドレス（または Google グループ アドレスや G Suite ドメイン名）をメンバーとして追加しなければなりません。</p>
<ol start="2">
<li>[<strong>メンバーを追加</strong>] をクリックします。ご使用のメールアドレスを入力し、<strong>[Cloud IAP] と [IAP で保護されたウェブアプリ ユーザー]</strong> 役割を選んでそのアドレスに割り当てます。同じようにして追加のアドレスや G Suite ドメインを入力できます。</li>
</ol>
<p><img alt="eab3d5cdd7ed299d.png" src="https://cdn.qwiklabs.com/mAbwQJBaXogbvkazE88Q%2Fc6m1wb3spF206JZpVtXsHw%3D"></p>
<ol start="3">
<li>[<strong>保存</strong>] をクリックします。</li>
</ol>
<p>「ポリシーを更新しました」というメッセージがウィンドウの下部に表示されます。</p>
<h3><strong>アクセスをテストする</strong></h3>
<p>アプリに戻ってページを再読み込みします。承認されたユーザーでログインしているのでウェブアプリが表示されるようになります。</p>
<p>引き続き「アクセス権がありません」ページが表示される場合は、IAP が承認を再確認していません。その場合は次のステップに従います。</p>
<ol>
<li>ホームページ アドレスの URL の末尾に <code>/_gcp_iap/clear_login_cookie</code> を追加して（<code>https://iap-example-999999.appspot.com/_gcp_iap/clear_login_cookie</code> など）、ウェブブラウザでそのアドレスを開きます。</li>
<li>アカウントがすでに表示された状態で、新たに「Google でログイン」画面が表示されます。アカウントをクリックせずに、[別のアカウントを使用] をクリックして認証情報を再入力ます。</li>
</ol>
<p>これで、IAP によりアクセス権が再確認されたので、アプリケーションのホーム画面が表示されます。</p>
<p>別の有効な Gmail アカウントや G Suite アカウントをお持ちのうえで、別のブラウザにアクセスできる場合、またはブラウザでシークレット モードを使用できる場合は、そのブラウザを使用してアプリページに移動し、他のアカウントでログインをお試しください。アカウントは承認されていないため、アプリが代わりに「アクセス権がありません」の画面が表示されます。</p>
<h2 id="step6">ユーザー ID 情報へアクセスする</h2>
<p>アプリが IAP で保護されると、通過するウェブ リクエスト ヘッダーで IAP により提供される ID 情報を使用できるようになります。このステップでアプリケーションが取得するのは、ログイン ユーザーのメールアドレスと、Google ID サービスによってそのユーザーに割り当てられた永続的な一意のユーザー ID です。そのデータはウェルカム ページでユーザーに表示されます。</p>
<p>このステップは、<code>iap-codelab/1-HelloWorld</code> フォルダを Cloud Shell で開く最後のステップです。次のステップのフォルダに移動します。</p>
<pre><code>cd ~/user-authentication-with-iap/2-HelloUser&#x000A;</code></pre>
<h3>App Engine にデプロイする</h3>
<p>デプロイには数分かかるので、まず Python の App Engine スタンダード環境へのアプリケーションのデプロイから始めます。</p>
<pre><code>gcloud app deploy&#x000A;</code></pre>
<p>続行を確認するメッセージが表示されたら、「<strong>Y</strong>」を入力します。</p>
<p>デプロイは数分で完了します。待っている間に、以下の説明のようにアプリケーション ファイルを調べることができます。</p>
<h3><strong>アプリケーション ファイルを調べる</strong></h3>
<p>このフォルダには、<code>1-HelloWorld</code> と同じ一連のファイルが含まれていますが、<code>main.py</code> と <code>templates/index.html</code> の 2 つのファイルが変更されています。プログラムは、IAP によってリクエスト ヘッダーに提供されるユーザー情報を取得するように変更されました。また、テンプレートにそのデータが表示されるようになりました。</p>
<p>IAP 提供の ID データを取得する <code>main.py</code> には以下の 2 行が含まれています。</p>
<pre><code>user_email = request.headers.get('X-Goog-Authenticated-User-Email')&#x000A;user_id = request.headers.get('X-Goog-Authenticated-User-ID')&#x000A;</code></pre>
<p><strong>X-Goog-Authenticated-User-</strong> ヘッダーが IAP によって提供されています。名前の大文字と小文字は区別されないため、必要に応じてすべて小文字または大文字で指定できます。render_template ステートメントにこれらの値が含まれるようになったので、表示が可能です。</p>
<pre><code>page = render_template('index.html', email=user_email, id=user_id)&#x000A;</code></pre>
<p>index.html テンプレートでこれらの値を表示するには、二重の中括弧で名前を囲みます。</p>
<pre><code>Hello, {{ email }}! Your persistent ID is {{ id }}.&#x000A;</code></pre>
<p>ご覧のとおり、提供されたデータには先頭に <code>accounts.google.com</code> が付いており、情報の出所を示します。必要な場合、アプリケーションではコロンまでのすべてを削除して未加工の値を取得できます。</p>
<h3><strong>更新された IAP をテストする</strong></h3>
<p>Deployment に戻ると、準備が整ったときに <code>gcloud app browse</code> アプリケーションを表示できるというメッセージが表示されます。</p>
<ol>
<li>
<p>以下のコマンドを入力します。</p>
</li>
</ol>
<pre><code>gcloud app browse&#x000A;</code></pre>
<ol start="2">
<li>ブラウザで新しいタブが開かない場合は表示されているリンクをコピーしますが、通常は新しいタブで開きます。次のようなページが表示されます。</li>
</ol>
<p><img alt="bcf47001573f0a17.png" src="https://cdn.qwiklabs.com/fe3%2F6PJvDcVemwODFLnePjFaHoMPvNhbWWsNmuJQC4s%3D"></p>
<p>アプリケーションが旧バージョンから新しいバージョンに置き換わるまで、数分かかる場合があります。必要に応じてページを更新して上記のようなページを表示します。</p>
<h3><strong>IAP をオフにする</strong></h3>
<p>IAP が無効になっている場合、またはなんらかの理由で（同じクラウド プロジェクトで実行されている他のアプリケーションなどによって）バイパスされている場合の、このアプリはどうなるのか、確認するには IAP をオフにします。</p>
<p>Cloud Console ウィンドウで、<strong>ナビゲーション メニュー</strong> &gt; [<strong>セキュリティ</strong>] &gt; [<strong>Identity-Aware Proxy</strong>] をクリックします。App Engine のアプリの隣の [<strong>IAP</strong>] 切り替えスイッチをクリックして、<strong>IAP</strong> をオフにします。</p>
<p><img alt="switch-to-off.png" src="https://cdn.qwiklabs.com/%2Bq8o2okiLz70YlU0HgYjMo4AUgWIJWBzKv9WRZuoLgg%3D"></p>
<p>すべてのユーザーがアプリにアクセスできるようになることを示す警告が示されます。</p>
<p>アプリケーションのウェブページを更新します。同じページが表示されますが、ユーザー情報は表示されなくなります。</p>
<p><img alt="9b341e63124d320f.png" src="https://cdn.qwiklabs.com/60irmGWAbgzDgFX1H3yCFeBha4t3oo%2F%2B2HmbxvVa2vQ%3D"></p>
<p>アプリケーションは保護されなくなったので、IAP を経由するように見えるウェブ リクエストを送信してみます。たとえば、Cloud Shell から次の curl コマンドを実行してリクエスト（<code>&lt;your-url-here&gt;</code> をアプリケーションの正しい URL に置き換える）を実行します。</p>
<pre><code>curl -X GET &lt;your-url-here&gt; -H "X-Goog-Authenticated-User-Email: totally fake email"&#x000A;</code></pre>
<p>ウェブページは次のようにコマンドラインに表示されます（コピーしないでください）<em></em>。</p>
<pre><code class="language-bash prettyprint">&lt;!doctype html&gt;&#x000A;&lt;html&gt;&#x000A;&lt;head&gt;&#x000A;  &lt;title&gt;IAP Hello User&lt;/title&gt;&#x000A;&lt;/head&gt;&#x000A;&lt;body&gt;&#x000A;  &lt;h1&gt;Hello World&lt;/h1&gt;&#x000A;&#x000A;  &lt;p&gt;&#x000A;    Hello, totally fake email! Your persistent ID is None.&#x000A;  &lt;/p&gt;&#x000A;&#x000A;  &lt;p&gt;&#x000A;    This is step 2 of the &lt;em&gt;User Authentication with IAP&lt;/em&gt;&#x000A;    codelab.&#x000A; &lt;/p&gt;&#x000A;&#x000A;&lt;/body&gt;&#x000A;&lt;/html&gt;&#x000A;</code></pre>
<p>IAP が無効になっていたりバイパスされていたりすることをアプリケーションが認識する手段はありません。潜在的なリスクがある場合は、暗号検証が解決します。</p>
<h2 id="step7">暗号検証を使用する</h2>
<p>IAP がオフであったりバイパスされたりするリスクがある場合、アプリでは受信した ID 情報が有効であることを確認できます。その場合、<code>X-Goog-IAP-JWT-Assertion</code> と呼ばれる、IAP によって追加された 3 番目のウェブ リクエスト ヘッダーを使用します。ヘッダーの値は暗号で署名されたオブジェクトで、ユーザー ID データも含まれています。アプリケーションはそのデジタル署名を検証し、そのオブジェクトで提供されたデータを使用します。そのデータは IAP から改変されずに提供されたものです。</p>
<p>デジタル署名の検証には、最新の Google 公開鍵セットの取得などの追加のステップがいくつか必要です。アプリケーションでこれらの追加のステップが必要かどうかは、他のユーザーが IAP を無効にしたりバイパスしたりできる可能性があるか、またはアプリケーションの感度などに基づいて決めることができます。</p>
<p>次が <code>iap-codelab/2-HelloUser</code> フォルダで Cloud Shell を開く最後のステップです。次のステップのフォルダに移動します。</p>
<pre><code>cd ~/user-authentication-with-iap/3-HelloVerifiedUser&#x000A;</code></pre>
<h3><strong>App Engine にデプロイする</strong></h3>
<p>Python 用の App Engine スタンダード環境にアプリをデプロイします。</p>
<pre><code>gcloud app deploy&#x000A;</code></pre>
<p>続行を確認するメッセージが表示されたら、「<strong>Y</strong>」を入力します。デプロイは数分で完了します。待っている間に、以下の説明のようにアプリケーション ファイルを調べることができます。</p>
<h3><strong>アプリケーション ファイルを調べる</strong></h3>
<p>このフォルダには <code>2-HelloUser</code> と同じ一連のファイルが含まれ、そのうちの 2 つは変更されたファイル、1 つは新規のファイルです。新しいファイルは <code>auth.py</code> です。これは、暗号で署名された ID 情報を取得して検証するための <code>user()</code> メソッドを提供します。そのメソッドの結果を使用するのが、変更された 2 つのファイルである <code>main.py</code> と <code>templates/index.html</code> です。比較のため、前回のデプロイの未確認ヘッダーも表示されます。</p>
<p>新しい機能は主に <code>user()</code> 関数に含まれています。</p>
<pre><code>def user():&#x000A;    assertion = request.headers.get('X-Goog-IAP-JWT-Assertion')&#x000A;    if assertion is None:&#x000A;        return None, None&#x000A;&#x000A;    info = jwt.decode(&#x000A;        assertion,&#x000A;        keys(),&#x000A;        algorithms=['ES256'],&#x000A;        audience=audience()&#x000A;    )&#x000A;&#x000A;    return info['email'], info['sub']&#x000A;</code></pre>
<p><code>assertion</code> は、指定されたリクエスト ヘッダーで提供される暗号的に署名されたデータです。コードはライブラリを使用して、このデータを検証およびデコードします。検証では、署名されたデータを確認し、そのデータが（基本的には保護されている Google Cloud プロジェクト用に）準備されていることを対象に知らせるために Google 提供の公開鍵を使用します。ヘルパー関数の <code>keys()</code> と <code>audience()</code> がそれらの値を収集して返します。</p>
<p>署名されたオブジェクトには、検証済みのメールアドレスと一意の ID 値（サブスクライバーの場合は、<code>sub</code> 標準フィールドに指定）の 2 つのデータが必要です。</p>
<p>これでステップ 3 は完了です。</p>
<h3><strong>暗号検証をテストする</strong></h3>
<p>デプロイの準備が整うと、<code>gcloud app browse</code> を使用してアプリケーションを表示できるというメッセージが表示されます。</p>
<p>ここで、次のコマンドを入力します。</p>
<pre><code>gcloud app browse&#x000A;</code></pre>
<p>ブラウザで新しいタブが開かない場合は、表示されているリンクをコピーして貼り付けると、通常は新しいタブで開きます。</p>
<p>前のステップで IAP を無効にしたため、アプリケーションによって IAP データが提供されることはありません。次のようなページが表示されます。</p>
<p><img alt="35350b93c86e21bc.png" src="https://cdn.qwiklabs.com/12FJLOT7gwhCPABz0vIPhRKHeE9UwzkCkhPywu5XULw%3D"></p>
<p>前のタスクと同様、最新バージョンが有効になって新しいページが表示されるまでに、場合によっては数分待つ必要があります。</p>
<p>IAP が無効になっているため、参照できるユーザー情報はありません。ここで IAP をオンに戻します。</p>
<ol>
<li>Cloud Console ウィンドウで、<strong>ナビゲーション メニュー</strong> &gt; [<strong>セキュリティ</strong>] &gt; [<strong>Identity-Aware Proxy</strong>] をクリックします。</li>
<li>App Engine のアプリの <strong>IAP</strong> の切り替えスイッチをクリックして、IAP をオンに戻します。</li>
</ol>
<p><img alt="switch-to-off.png" src="https://cdn.qwiklabs.com/%2Bq8o2okiLz70YlU0HgYjMo4AUgWIJWBzKv9WRZuoLgg%3D"></p>
<ol start="3">
<li>ページを更新すると、次のようになります。</li>
</ol>
<p><img alt="f738310ac4866fac.png" src="https://cdn.qwiklabs.com/UsOXUWd8w0x29naS7Dgiii1kedVbpV5BGxi7PLKFR98%3D"></p>
<p>検証方法で提供されるメールアドレス（<code>verified_email</code>）には、先頭に <code>accounts.google.com:</code> が付いてないことにご注意ください。</p>

**このようにすると、`curl` で IAP 経由っぽいリクエストを打っても、`verified_email` は None になる**

<p>IAP がオフになっているかバイパスされている場合、検証されたデータは有効な署名を使用できないため紛失または無効になります。これは、Google の秘密鍵の所有者がそのデータを作成した場合を除きます。</p>
<h2 id="step8">お疲れさまでした</h2>
<p>App Engine ウェブ アプリケーションをデプロイしました。まず、アプリケーションへのアクセスを、選択したユーザーのみに制限しました。次に、IAP によりアプリケーションへのアクセスが許可されたユーザー ID を取得して表示し、IAP が無効だったりバイパスされていたりする場合はその情報がなりすましの可能性があることを確認しました。最後に、なりすましが不可能な、暗号で署名されたユーザー ID のアサーションを検証しました。</p>

# Cloud KMS の使い方
<h2 id="step2">概要</h2>
<p>このラボでは、Google Cloud Security および Privacy API の次のような高度な機能の使い方について学びます。</p>
<ul>
<li>安全な Cloud Storage バケットの設定</li>
<li>Key Management Storage を使用した鍵と暗号化データの管理</li>
<li>Cloud Storage 監査ログの表示</li>
</ul>
<p>Enron Corpus から要約データを入手して暗号化し、Cloud Storage に読み込みます。</p>
<h4>演習内容</h4>
<ul>
<li>
<p>データを暗号化する方法と、鍵管理サービス（KMS）を使用した暗号鍵の管理方法</p>
</li>
</ul>
<h4>必要なもの</h4>
<ul>
<li>課金を有効にした Google Cloud プロジェクト</li>
<li>Google Cloud Storage</li>
<li>Google Cloud SDK</li>
</ul>

<h2 id="step4">Cloud Storage バケットを作成する</h2>
<p>このラボのデータを保存するために、独自の Cloud Storage バケットを作成する必要があります。</p>
<p><code>[自分の名前]_enron_corpus</code> など、Cloud Storage バケットの名前を指定します。バケットの命名について詳しくは、Cloud Storage バケットの<a href="https://cloud.google.com/storage/docs/naming">命名ガイドライン</a>をご覧ください。Cloud Shell で次のコマンドを実行して、変数にバケット名を設定します。</p>
<pre><code class="language-bash prettyprint">BUCKET_NAME=[自分の名前]_enron_corpus&#x000A;</code></pre>
<p>次のコマンドを実行してバケットを作成します。</p>
<pre><code class="language-bash prettyprint">gsutil mb gs://${BUCKET_NAME}&#x000A;</code></pre>
<p>このコマンドを実行することで、<code>gsutil</code> コマンドライン クライアントが正しく設定されていること、認証が機能していること、操作しているクラウド プロジェクトへの書き込みアクセス権があることも確認できます。</p>
<p>バケットが作成されたら、次の手順に進んで Enron Corpus をダウンロードします。</p>

<h2 id="step5">データをチェックアウトする</h2>
<p><a href="https://en.wikipedia.org/wiki/Enron_Corpus">Enron Corpus</a> は大規模なデータベースで、Enron Corporation の従業員 158 人によって作成された 60 万通を超えるメールが管理されています。このデータは GCS バケット <code>gs://enron_emails/</code> にコピーされています。</p>
<p><img alt="fc99ebca471332d0.png" src="https://cdn.qwiklabs.com/0tOYdCuto2nH4H9ZtNtukERk%2BPw8joSz7ynkdW4G280%3D"></p>
<p>次のコマンドを実行してソースファイルの 1 つをローカルにダウンロードすると、その内容を確認することができます。</p>
<pre><code class="language-bash prettyprint">gsutil cp gs://enron_emails/allen-p/inbox/1. .&#x000A;</code></pre>
<p>ダウンロードしたファイルの末尾を <code>tail</code> で表示して、メールのテキストが格納されていることを確認します。</p>
<pre><code class="language-bash prettyprint">tail 1.&#x000A;</code></pre>
<p>次の出力が表示されます。</p>
<pre><code>Attached is the Delta position for 1/18, 1/31, 6/20, 7/16, 9/24&#x000A;&#x000A; &lt;&lt; File: west_delta_pos.xls &gt;&gt;&#x000A;&#x000A;Let me know if you have any questions.&#x000A;</code></pre>
<p>プレーン テキストのメールファイルの内容が表示されます。表示されるファイルには、プレーン テキストのメールファイルと画像ファイルの 2 種類があります。ご興味があれば、同様のメカニズムを使用して他のファイルの内容も確認してみてください。</p>
<h2 id="step6">Cloud KMS を有効にする</h2>
<p><a href="https://cloud.google.com/kms/">Cloud KMS</a> は、GCP の暗号鍵管理サービス（KMS）です。KMS を使用するには、前もってプロジェクトで有効にしておく必要があります。プロビジョニングされた Qwiklabs の GCP プロジェクトでは、すでに KMS が有効になっていますが、これは <code>gcloud</code> CLI コマンドの 1 つを使用して確認できます。Cloud Shell セッションで次のコマンドを実行してください。</p>
<pre><code class="language-bash prettyprint">gcloud services enable cloudkms.googleapis.com&#x000A;</code></pre>
<aside class="special"><p><strong>注: </strong><a href="https://console.cloud.google.com/apis/api/cloudkms.googleapis.com" target="blank">Cloud Console の UI</a> を使用して KMS やその他のサービスをプロジェクトで有効にすることもできます。</p>
</aside>
<p>出力は特にありません。これで、Cloud KMS がプロジェクトで有効になりました。</p>
<h2 id="step7">キーリングと暗号鍵を作成する</h2>
<p>データを暗号化するには、キーリングと暗号鍵を作成する必要があります。キーリングは鍵をグループ化する際に役立ちます。鍵は環境別（<strong>テスト</strong>、<strong>ステージング</strong>、<strong>本番</strong>など）、またはその他の概念別にグループ化できます。このラボでは、キーリングの名前は <code>test</code>、暗号鍵の名前は <code>qwiklab</code> です。Cloud Shell で次のコマンドを実行して、環境変数を設定します。</p>
<pre><code class="language-bash prettyprint">KEYRING_NAME=test CRYPTOKEY_NAME=qwiklab&#x000A;</code></pre>
<p><code>gcloud</code> コマンドを実行してキーリングを作成します。このラボではグローバル ロケーションを使用しますが、特定のリージョンに設定することもできます。</p>
<pre><code class="language-bash prettyprint">gcloud kms keyrings create $KEYRING_NAME --location global&#x000A;</code></pre>
<p>次に、新しいキーリングを使用して <code>qwiklab</code> という名前の暗号鍵を作成します。</p>
<pre><code class="language-bash prettyprint">gcloud kms keys create $CRYPTOKEY_NAME --location global \&#x000A;      --keyring $KEYRING_NAME \&#x000A;      --purpose encryption&#x000A;</code></pre>
<aside class="special"><p><strong>注: </strong>Cloud KMS では暗号鍵もキーリングも削除できません。</p>
</aside>
<p>出力は特にありません。これで、キーリングと暗号鍵が作成されました。Cloud Platform Console で、<strong>ナビゲーション メニュー</strong> &gt; [<strong>IAM と管理</strong>] &gt; [<strong>暗号鍵</strong>] &gt; [<strong>Key Management に移動</strong>] に移動して、[<a href="https://console.cloud.google.com/iam-admin/kms">暗号鍵</a>] を開きます。</p>
<p><img alt="crypto.gif" src="https://cdn.qwiklabs.com/Aawqa2QNZNWH2V8og3vZ9NewUgsawYJJfakzEz5LtSg%3D"></p>
<p>[キー管理] ウェブ UI では、暗号鍵とキーリングを管理できます。後で権限を管理するときにこの UI を使用します。</p>
<p><img alt="1276961ef4490af1.png" src="https://cdn.qwiklabs.com/uSvSTuI6BKeS4obR9Xn85PQjKoG%2FM77QNZp9cDYopaA%3D"></p>

<h2 id="step8">データを暗号化する</h2>
<p>次に、データを暗号化してみましょう。上記で表示したメールの内容を、次のコマンドを実行して <code>base64</code> エンコードします。</p>
<pre><code class="language-bash prettyprint">PLAINTEXT=$(cat 1. | base64 -w0)&#x000A;</code></pre>
<aside class="special"><p><strong>メモ: </strong>Base-64 エンコードを行うと、バイナリデータをプレーン テキストとして API に渡すことができます。このコマンドの対象は、画像、動画、または他の任意の種類のバイナリデータです。</p>
</aside>
<p>暗号化エンドポイントを使用すると、暗号化対象の base64 エンコードされたテキストを、指定した鍵に渡すことができます。</p>
<p>以下のコマンドを実行します。</p>
<pre><code class="language-bash prettyprint">curl -v "https://cloudkms.googleapis.com/v1/projects/$DEVSHELL_PROJECT_ID/locations/global/keyRings/$KEYRING_NAME/cryptoKeys/$CRYPTOKEY_NAME:encrypt" \&#x000A;  -d "{\"plaintext\":\"$PLAINTEXT\"}" \&#x000A;  -H "Authorization:Bearer $(gcloud auth application-default print-access-token)"\&#x000A;  -H "Content-Type: application/json"&#x000A;</code></pre>
<aside class="special"><p><strong>注: </strong>同じテキストと鍵を使用しても、<code>encrypt</code> アクションのたびに異なる結果が返されます。</p>
</aside>
<p>レスポンスは、属性 <code>ciphertext</code> の暗号化テキストが含まれた JSON ペイロードになります。</p>
<p>これで、暗号化されたデータをファイルに保存し、Cloud Storage バケットにアップロードできるようになりました。JSON レスポンスから暗号化テキストを取得してファイルに保存するには、コマンドライン ユーティリティ <a href="https://stedolan.github.io/jq/">jq</a> を使用します。前述の呼び出しのレスポンスを jq にパイプで渡すと、それを解析して <code>ciphertext</code> プロパティを取り出し、ファイル <code>1.encrypted</code> に保存できます。</p>
<p>以下のコマンドを実行します。</p>
<pre><code class="language-bash prettyprint">curl -v "https://cloudkms.googleapis.com/v1/projects/$DEVSHELL_PROJECT_ID/locations/global/keyRings/$KEYRING_NAME/cryptoKeys/$CRYPTOKEY_NAME:encrypt" \&#x000A;  -d "{\"plaintext\":\"$PLAINTEXT\"}" \&#x000A;  -H "Authorization:Bearer $(gcloud auth application-default print-access-token)"\&#x000A;  -H "Content-Type:application/json" \&#x000A;| jq .ciphertext -r &gt; 1.encrypted&#x000A;</code></pre>
<p>暗号化データが復号可能かどうかは、<code>decrypt</code> エンドポイントを呼び出して、復号されたテキストが元のメールと一致するかどうかで確認できます。暗号化データには、その暗号化に使用された暗号鍵バージョンに関する情報が含まれているため、復号エンドポイントに特定のバージョンが渡されることはありません。</p>
<p>以下のコマンドを実行します。</p>
<pre><code class="language-bash prettyprint">curl -v "https://cloudkms.googleapis.com/v1/projects/$DEVSHELL_PROJECT_ID/locations/global/keyRings/$KEYRING_NAME/cryptoKeys/$CRYPTOKEY_NAME:decrypt" \&#x000A;  -d "{\"ciphertext\":\"$(cat 1.encrypted)\"}" \&#x000A;  -H "Authorization:Bearer $(gcloud auth application-default print-access-token)"\&#x000A;  -H "Content-Type:application/json" \&#x000A;| jq .plaintext -r | base64 -d&#x000A;</code></pre>
<aside class="special"><p><strong>注: </strong>通常、復号はアプリケーション層で行われます。複数のプログラミング言語でのデータの暗号化と復号のチュートリアルについては、<a href="https://cloud.google.com/kms/docs/quickstart" target="blank">Cloud KMS のクイックスタート</a>をご確認ください。</p>
</aside>
<p>これで、テキストが正常に暗号化されたことを確認できたので、暗号化ファイルを Cloud Storage バケットにアップロードします。</p>
<pre><code class="language-bash prettyprint">gsutil cp 1.encrypted gs://${BUCKET_NAME}&#x000A;</code></pre>

<h2 id="step9">IAM 権限を構成する</h2>
<p>KMS には注目すべき主要な権限が 2 つあります。一方の権限ではユーザーまたはサービス アカウントで <strong>KMS リソースを管理</strong>でき、もう一方ではユーザーまたはサービス アカウントで鍵を使用してデータの<strong>暗号化と復号</strong>が可能です。</p>
<p>鍵を管理する権限は <code>cloudkms.admin</code> で、この権限を持つすべてのユーザーがキーリングを作成し、暗号鍵を作成、変更、無効化、破壊できます。暗号化および復号する権限は <code>cloudkms.cryptoKeyEncrypterDecrypter</code> で、暗号化および復号 API エンドポイントを呼び出すために使用されます。</p>
<p>この演習では、現在の承認済みユーザーを使用して IAM 権限を割り当てます。現在の承認済みユーザーを取得するには、次のコマンドを実行します。</p>
<pre><code class="language-bash prettyprint">USER_EMAIL=$(gcloud auth list --limit=1 2&gt;/dev/null | grep '@' | awk '{print $2}')&#x000A;</code></pre>
<p>次に、そのユーザーに KMS リソースの管理権限を割り当てます。次の <code>gcloud</code> コマンドを実行し、作成したキーリングを管理する IAM 権限を割り当ててください。</p>
<pre><code class="language-bash prettyprint">gcloud kms keyrings add-iam-policy-binding $KEYRING_NAME \&#x000A;    --location global \&#x000A;    --member user:$USER_EMAIL \&#x000A;    --role roles/cloudkms.admin&#x000A;</code></pre>
<p>暗号鍵はキーリングに属し、キーリングはプロジェクトに属するため、その階層の上位レベルでユーザーが持つ役割と権限は子リソースに継承されます。たとえば、あるプロジェクトに対してオーナー役割を持つユーザーは、そのプロジェクトのすべてのキーリングおよび暗号鍵に対してもオーナーになります。同様に、あるユーザーにキーリングの <code>cloudkms.admin</code> 役割が付与されている場合は、そのキーリングの暗号鍵に関連する権限も付与されます。</p>
<p><code>cloudkms.cryptoKeyEncrypterDecrypter</code> 権限を持たない承認済みユーザーは、データを暗号化または復号するための鍵を使用できません。次の <code>gcloud</code> コマンドを実行して、作成したキーリングに属するすべての暗号鍵でデータを暗号化および復号する IAM 権限を割り当ててください。</p>
<pre><code class="language-bash prettyprint">gcloud kms keyrings add-iam-policy-binding $KEYRING_NAME \&#x000A;    --location global \&#x000A;    --member user:$USER_EMAIL \&#x000A;    --role roles/cloudkms.cryptoKeyEncrypterDecrypter&#x000A;</code></pre>
<p>これで、割り当てられた権限が <a href="https://console.cloud.google.com/iam-admin/kms">Cloud Key Management Service</a> の [暗号鍵] に表示されます。</p>
<p>キーリングの名前（<code>test</code>）の横にあるボックスをオンにし、右の列で [<strong>権限</strong>] をクリックします。</p>
<p>メニューが表示され、今追加したキーリングのアカウントと権限を確認できます。</p>
<p><img alt="4db0ceae69aa4422.png" src="https://cdn.qwiklabs.com/t3QrDWL4UyMYcua%2FtUE0bcP0XwE5VrIPhqZsc9CVJJE%3D"></p>
<h2 id="step10">コマンドラインでデータのバックアップを作成する</h2>
<p>単一のファイルを暗号化する方法を理解し、そのための権限が付与されると、ディレクトリ内のすべてのファイルのバックアップを作成するスクリプトを実行できるようになります。この例では、<strong>allen-p</strong> のすべてのメールをコピーして暗号化し、Cloud Storage バケットにアップロードします。</p>
<p>最初に、<strong>allen-p</strong> のすべてのメールを現在の作業ディレクトリにコピーします。</p>
<pre><code class="language-bash prettyprint">gsutil -m cp -r gs://enron_emails/allen-p .&#x000A;</code></pre>
<p>今度は、以下をコピーして Cloud Shell に貼り付け、<strong>allen-p</strong> ディレクトリ内のすべてのファイルのバックアップを Cloud Storage バケットに作成して暗号化します。</p>
<pre><code class="language-bash prettyprint">MYDIR=allen-p&#x000A;FILES=$(find $MYDIR -type f -not -name "*.encrypted")&#x000A;for file in $FILES; do&#x000A;  PLAINTEXT=$(cat $file | base64 -w0)&#x000A;  curl -v "https://cloudkms.googleapis.com/v1/projects/$DEVSHELL_PROJECT_ID/locations/global/keyRings/$KEYRING_NAME/cryptoKeys/$CRYPTOKEY_NAME:encrypt" \&#x000A;    -d "{\"plaintext\":\"$PLAINTEXT\"}" \&#x000A;    -H "Authorization:Bearer $(gcloud auth application-default print-access-token)" \&#x000A;    -H "Content-Type:application/json" \&#x000A;  | jq .ciphertext -r &gt; $file.encrypted&#x000A;done&#x000A;gsutil -m cp allen-p/inbox/*.encrypted gs://${BUCKET_NAME}/allen-p/inbox&#x000A;</code></pre>
<p>このスクリプトは指定されたディレクトリ内のすべてのファイルに対してループ実行され、KMS API を使用してファイルを暗号化し、それを Google Cloud Storage にアップロードします。</p>

<p>スクリプトの完了後、Cloud Platform Console の左側のメニューで [Storage] をクリックすると、暗号化されたファイルが表示されます。ファイルを見つけるには、[<strong>Storage</strong>] &gt; [<strong>ブラウザ</strong>] &gt; [<strong>&lt;該当するバケット&gt;</strong>] &gt; [<strong>allen-p</strong>] &gt; [<strong>inbox</strong>] に移動します。次のように表示されます。</p>
<p><img alt="3fcab808bab9d6fb.png" src="https://cdn.qwiklabs.com/LBBvUXIIKTjshgLeh04UeLEtvf70BTbi6CKxtuSYNwE%3D"></p>
<aside class="special"><p><strong>注: </strong>Cloud Storage では、<a href="https://cloud.google.com/storage/docs/encryption" target="blank">サーバー側の暗号化</a>がサポートされています。サーバー側の暗号化では、データに対する鍵のローテーションがサポートされているため、Cloud Storage でデータを暗号化する方法としておすすめです。上記の例はデモを目的とするものです。</p>
</aside>
<h2 id="step11">Cloud 監査ログを表示する</h2>
<p>Google Cloud Audit Logging は、管理アクティビティとデータアクセスという 2 つのログストリームで構成されています。これらは Google Cloud Platform サービスによって生成され、Google Cloud Platform のプロジェクト内で、誰が、何を、どこで、いつ行ったかを調べるのに役立ちます。</p>
<p>KMS 内のリソースに対するアクティビティを表示するには、[暗号鍵] ページに戻り（[<strong>IAM と管理</strong>] &gt; [<strong>暗号鍵</strong>] &gt; [<strong>Key Management に移動</strong>]）、キーリングの横にあるボックスをオンにしてから、右のメニューの [<strong>アクティビティ</strong>] タブをクリックします。クラウド アクティビティの UI で、キーリングの作成とキーリングに対するすべての変更が表示されます。</p>
<p><img alt="f05f4807b88e41e4.png" src="https://cdn.qwiklabs.com/qGaNMJgiLQJPsXQ2D%2BMnkshK4k%2FvRM1M%2Fg2FC7DyBM0%3D"></p>
<aside class="special"><p><strong>注: </strong>何も表示されない場合は、右上にある [情報パネルを表示] をクリックします。</p>
</aside>
<p>これで、KMS と Cloud Storage を使用して、データを暗号化し、アップロードしました。</p>
<h3><strong>学習した内容</strong></h3>
<ul>
<li>IAM を使用した KMS 権限の管理。</li>
<li>KMS を使用したデータの暗号化。</li>
<li>GCS を使用した暗号化データの格納。</li>
<li>Cloud Audit Logging を使用した、暗号鍵およびキーリングに対するすべてのアクティビティの表示。</li>
</ul>

# 限定公開 Kubernetes クラスタのセットアップ
<h2 id="step2">概要</h2>
<p>Kubernetes Engine における限定公開クラスタとは、公共のインターネットからマスターにアクセスできないようにするクラスタのことです。このクラスタのノードにはプライベート アドレスしかない（パブリック IP アドレスがない）ため、隔離された環境でワークロードが実行されます。ノードとマスターは、VPC ピアリングを使用して相互に通信します。</p>
<p>Kubernetes Engine API では、アドレス範囲が CIDR（Classless Inter-Domain Routing）ブロックとして表されます。</p>
<p>このラボでは、限定公開 Kubernetes クラスタを作成する方法を学びます。</p>
<h3>演習内容</h3>
<ul>
<li>
<p>限定公開 Kubernetes クラスタを作成します。</p>
</li>
</ul>
<h3>前提条件</h3>
<ul>
<li>
<p>Kubernetes クラスタを作成および起動した経験があり、CIDR 範囲での IP アドレス指定について熟知している必要があります。</p>
</li>
</ul>

<h2 id="step4">ゾーンの設定</h2>
<p>次のコマンドを実行して、デフォルトのゾーンを設定します。</p>
<pre><code>gcloud config set compute/zone us-central1-a&#x000A;</code></pre>
<aside>
次のコマンドを実行すると、利用可能なゾーンをすべて表示できます。

<code>gcloud compute zones list</code>
</aside>
<h2 id="step5">限定公開クラスタの作成</h2>
<p>限定公開クラスタを作成する場合、Kubernetes のマスター コンポーネントを実行する VM に対して CIDR 範囲を <code>/28</code> と指定し、IP エイリアスを有効にする必要があります。</p>
<p>次に、<code>private-cluster</code> という名前のクラスタを作成し、マスターに対して CIDR 範囲を <code>172.16.0.16/28</code> と指定します。IP エイリアスを有効にして、Kubernetes Engine が自動的にサブネットワークを作成するようにします。</p>
<p><code>--private-cluster</code>、<code>--master-ipv4-cidr</code>、<code>--enable-ip-alias</code> のフラグを使用して限定公開クラスタを作成します。</p>
<p>次のコマンドを実行してクラスタを作成します。</p>
<pre><code>gcloud beta container clusters create private-cluster \&#x000A;    --private-cluster \&#x000A;    --master-ipv4-cidr 172.16.0.16/28 \&#x000A;    --enable-ip-alias \&#x000A;    --create-subnetwork ""&#x000A;</code></pre>

<h2 id="step6">サブネットとセカンダリ アドレス範囲の表示</h2>
<p>デフォルト ネットワークのサブネットを一覧表示します。</p>
<pre><code>gcloud compute networks subnets list --network default&#x000A;</code></pre>
<p>出力で、クラスタに対して自動的に作成されたサブネットワークの名前を確認します（<code>gke-private-cluster-subnet-xxxxxxxx</code> のようになっています）。次のステップで使用するため、クラスタの名前を保存しておきます。</p>
<p>次に自動的に作成されたサブネットの情報を取得します。次のコマンドの <code>[SUBNET_NAME]</code> を対象のサブネットに置き換えて実行します。</p>
<pre><code>gcloud compute networks subnets describe [SUBNET_NAME] --region us-central1&#x000A;</code></pre>
<p>出力には、プライマリ アドレス範囲と GKE の限定公開クラスタの名前、およびセカンダリ範囲が表示されます。</p>
<pre><code class="language-bash prettyprint">...&#x000A;ipCidrRange: 10.0.0.0/22&#x000A;kind: compute#subnetwork&#x000A;name: gke-private-cluster-subnet-163e3c97&#x000A;...&#x000A;privateIpGoogleAccess: true&#x000A;...&#x000A;secondaryIpRanges:&#x000A;- ipCidrRange: 10.40.0.0/14&#x000A;  rangeName: gke-private-cluster-pods-163e3c97&#x000A;- ipCidrRange: 10.0.16.0/20&#x000A;  rangeName: gke-private-cluster-services-163e3c97&#x000A;...&#x000A;</code></pre>
<p>出力からは、セカンダリ範囲の 1 つ目が<strong>ポッド</strong>用で、2 つ目が<strong>サービス</strong>用であることがわかります。</p>
<p><code>privateIPGoogleAccess</code> が <code>true</code> に設定されていることに注意してください。これにより、プライベート IP アドレスだけが指定されているクラスタホストが、Google API やサービスと通信できるようになります。</p>
<h2 id="step7">マスター承認済みネットワークの有効化</h2>
<p>この時点で、マスターにアクセスできる IP アドレスは、次の範囲のアドレスだけです。</p>
<ul>
<li>ノードに使用される、サブネットワークのプライマリ範囲。</li>
<li>ポッドに使用される、サブネットワークのセカンダリ範囲。</li>
</ul>
<p>追加でマスターにアクセスできるようにするには、選択したアドレス範囲を承認する必要があります。</p>
<h4>VM インスタンスを作成する</h4>
<p>Kubernetes クラスタへの接続を確認するために使用するソース インスタンスを作成します。</p>
<pre><code>gcloud compute instances create source-instance --zone us-central1-a --scopes 'https://www.googleapis.com/auth/cloud-platform'&#x000A;</code></pre>

<p>次のコマンドで、<code>source-instance</code> の <code>&lt;External_IP&gt;</code> を取得します。</p>
<pre><code>gcloud compute instances describe source-instance --zone us-central1-a | grep natIP&#x000A;</code></pre>
<p><strong>出力例:</strong></p>
<pre><code class="language-bash prettyprint">natIP: 35.192.107.237&#x000A;</code></pre>
<p>後のステップで使用するために、<code>&lt;nat_IP&gt;</code> アドレスをコピーして保存します。</p>
<p>次のコマンドを実行して、外部アドレス範囲を承認します。<code>[MY_EXTERNAL_RANGE]</code> は、前の出力で取得した外部アドレスの CIDR 範囲（ <code>natIP/32</code>）に置き換えます。CIDR 範囲を <code>natIP/32</code> にすると、1 つの IP アドレスがホワイトリストに登録されます。</p>
<pre><code>gcloud container clusters update private-cluster \&#x000A;    --enable-master-authorized-networks \&#x000A;    --master-authorized-networks [MY_EXTERNAL_RANGE]&#x000A;</code></pre>
<aside>
本番環境では、<code>[MY_EXTERNAL_RANGE]</code> を該当のネットワーク外部アドレスの CIDR 範囲に置き換えます。

</aside>

<p>これで外部アドレスの範囲からマスターにアクセスできるようになったので、今度は <code>kubectl</code> をインストールしてクラスタの情報を取得します。たとえば、<code>kubectl</code> を使用すると、ノードに外部 IP アドレスがないことを確認できます。</p>
<p>次のコマンドで <code>source-instance</code> に SSH 接続します。</p>
<pre><code>gcloud compute ssh source-instance --zone us-central1-a&#x000A;</code></pre>
<p><code>Y</code> キーを押して続行します。パスフレーズの質問への回答を入力して <strong>Enter</strong> キーを押します。</p>
<p>SSH シェルで、Cloud SDK の <code>kubectl</code> コンポーネントをインストールします。</p>
<pre><code>gcloud components install kubectl&#x000A;</code></pre>
<aside>
「<code>You cannot perform this action because the Cloud SDK component manager is disabled for this installation</code>」というエラーが表示された場合は、次のコマンドで <code>kubectl</code> コンポーネントをインストールしてみてください。

<code>sudo apt-get install kubectl</code>
</aside>
<p>次のコマンドで、SSH シェルから Kubernetes クラスタへのアクセスを設定します。</p>
<pre><code>gcloud container clusters get-credentials private-cluster --zone us-central1-a&#x000A;</code></pre>
<p>クラスタノードに外部 IP アドレスがないことを確認します。</p>
<pre><code>kubectl get nodes --output yaml | grep -A4 addresses&#x000A;</code></pre>
<p>ノードに内部 IP アドレスが指定されていて、外部アドレスが指定されていないことが出力に示されます。</p>
<pre><code class="language-bash prettyprint">...&#x000A;addresses:&#x000A;- address: 10.0.0.4&#x000A;  type: InternalIP&#x000A;- address: ""&#x000A;  type: ExternalIP&#x000A;...&#x000A;</code></pre>
<p>次のコマンドでも、ノードに外部 IP アドレスが指定されていないことを確認できます。</p>
<pre><code>kubectl get nodes --output wide&#x000A;</code></pre>
<p><code>EXTERNAL-IP</code> が空白の出力が返されます。</p>
<pre><code class="language-bash prettyprint">STATUS ... VERSION        EXTERNAL-IP   OS-IMAGE ...&#x000A;Ready      v1.8.7-gke.1                 Container-Optimized OS from Google&#x000A;Ready      v1.8.7-gke.1                 Container-Optimized OS from Google&#x000A;Ready      v1.8.7-gke.1                 Container-Optimized OS from Google&#x000A;</code></pre>
<p>次のコマンドを入力して SSH シェルを閉じます。</p>
<pre><code>exit&#x000A;</code></pre>
<h2 id="step8">クリーンアップ</h2>
<p>Kubernetes クラスタを削除します。</p>
<pre><code>gcloud container clusters delete private-cluster --zone us-central1-a&#x000A;</code></pre>
<p><code>Y</code> キーを押して続行します。</p>

<h2 id="step9">カスタム サブネットワークを使用する限定公開クラスタの作成（省略可）</h2>
<p>前のセクションでは、Kubernetes Engine によって自動的にサブネットワークが作成されました。このセクションでは、独自のカスタム サブネットワークを作成し、限定公開クラスタを作成します。
このサブネットワークは、プライマリ アドレス範囲と 2 つのセカンダリ アドレス範囲を備えているものにします。</p>
<p>サブネットワークとセカンダリ範囲を作成します。</p>
<pre><code>gcloud compute networks subnets create my-subnet \&#x000A;    --network default \&#x000A;    --range 10.0.4.0/22 \&#x000A;    --enable-private-ip-google-access \&#x000A;    --region us-central1 \&#x000A;    --secondary-range my-svc-range=10.0.32.0/20,my-pod-range=10.4.0.0/14&#x000A;</code></pre>

<p>サブネットワークを使用する限定公開クラスタを作成します。</p>
<pre><code>gcloud beta container clusters create private-cluster2 \&#x000A;    --private-cluster \&#x000A;    --enable-ip-alias \&#x000A;    --master-ipv4-cidr 172.16.0.32/28 \&#x000A;    --subnetwork my-subnet \&#x000A;    --services-secondary-range-name my-svc-range \&#x000A;    --cluster-secondary-range-name my-pod-range&#x000A;</code></pre>

<p>外部アドレス範囲を承認します。<code>[MY_EXTERNAL_RANGE]</code> は、前の出力で取得した外部アドレスの CIDR 範囲に置き換えます。</p>
<pre><code>gcloud container clusters update private-cluster2 \&#x000A;    --enable-master-authorized-networks \&#x000A;    --master-authorized-networks [MY_EXTERNAL_RANGE]&#x000A;</code></pre>

<p>次のコマンドで <code>source-instance</code> に SSH 接続します。</p>
<pre><code>gcloud compute ssh source-instance --zone us-central1-a&#x000A;</code></pre>
<p>次のコマンドで、SSH シェルから Kubernetes クラスタへのアクセスを構成します。</p>
<pre><code>gcloud container clusters get-credentials private-cluster2 --zone us-central1-a&#x000A;</code></pre>
<p>クラスタノードに外部 IP アドレスがないことを確認します。</p>
<pre><code>kubectl get nodes --output yaml | grep -A4 addresses&#x000A;</code></pre>
<p>ノードに内部 IP アドレスが指定されていて、外部アドレスが指定されていないことが出力に示されます。</p>
<pre><code>...&#x000A;addresses:&#x000A;- address: 10.0.4.3&#x000A;  type: InternalIP&#x000A;- address: ""&#x000A;  type: ExternalIP&#x000A;...&#x000A;</code></pre>
<p>この時点で、マスターにアクセスできる IP アドレスは、次の範囲のアドレスだけです。</p>
<ul>
<li>
<p>作成したサブネットワークのプライマリの範囲で、ノードに使用される。この例のノードの範囲は <code>10.0.4.0/22</code> です。</p>
</li>
<li>
<p>作成したサブネットワークのセカンダリ範囲で、ポッドに使用される。この例のポッドの範囲は <code>10.4.0.0/14</code> です。</p>
</li>
</ul>
<h2 id="step10">お疲れさまでした</h2>