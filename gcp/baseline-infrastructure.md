# Cloud Storage: Qwik Start - Console

<h2 id="step2">概要</h2>
<p>Google Cloud Storage では、いつでも世界中のどこからでも、データを保存、取得できます。データの量に制限はありません。ウェブサイト コンテンツの提供、アーカイブと障害復旧のためのデータの保存、直接ダウンロードによるユーザーへの大きなデータ オブジェクトの配布など、さまざまなシナリオで Google Cloud Storage を使用できます。このハンズオンラボでは、Google Cloud Platform Console を使用してストレージ バケットを作成し、オブジェクトをアップロードしてフォルダとサブフォルダを作成し、オブジェクトを一般公開する方法を学びます。</p>

<h2 id="step4">バケットの作成</h2>
<p>コンソールで、<strong>ナビゲーション メニュー</strong> &gt; [<strong>Storage</strong>] &gt; [<strong>ブラウザ</strong>] に移動し、[<strong>バケットを作成</strong>] をクリックします。</p>
<p><img alt="create-bucket.png" src="https://cdn.qwiklabs.com/K3M3mS%2F5XbEv%2BtqWj7%2FyU7B98yazE3fgw%2FHwBbznwzo%3D"></p>
<p><strong>名前:</strong> 一意のバケット名を入力します。</p>
<p><strong>バケットの命名規則は次のとおりです。</strong></p>
<ul>
<li>バケットの名前空間はグローバルであり、一般公開されるため、バケット名に機密情報を含めないでください。</li>
<li>バケット名に使用できる文字は、小文字、数字、ダッシュ（-）、下線（_）、ドット（.）のみです。ドットを使用している名前には<a href="https://cloud.google.com/storage/docs/domain-name-verification">確認</a>が必要です。</li>
<li>バケット名の先頭と末尾は数字か文字でなければなりません。</li>
<li>バケット名の長さは 3～63 文字とする必要があります。ドットを使用している名前には最大 222 文字を使用できますが、ドットで区切られている各要素は 63 文字以下としてください。</li>
<li>バケット名はドット区切りの十進表記（たとえば、192.168.5.4）の IP アドレス形式で表すことはできません。</li>
<li>バケット名の先頭に接頭辞「goog」は使用できません。</li>
<li>バケット名に「google」または「google」と類似する表記を含めることはできません。</li>
<li>また、DNS の基準に準拠し、将来的な互換性を確保するため、下線（_）を使用したり、ドットを連続して使用したり、ドットとダッシュを隣同士で使用したりしないでください。たとえば、「..」、「-.」、「.-」は、DNS 名では有効ではありません。</li>
</ul>
<p><strong>ストレージ クラス:</strong> マルチリージョン</p>
<p><strong>ロケーション:</strong> 米国</p>
<p>バケットの構成が完了したら、[<strong>作成</strong>] をクリックします。</p>
<p><img alt="bucket.png" src="https://cdn.qwiklabs.com/zu%2BzJrePKI8%2BzSw%2FJh5fEWrmiY8ZVgni4qYvYOGnbUw%3D"></p>
<p>これで Cloud Storage バケットの作成は完了です。</p>

<p><ql-multiple-select-probe answerindices="[0, 1, 3, 4]" optiontitles='[
"Multi-Regional Storage",
"Regional Storage",
"クロスリージョン ストレージ",
"Nearline Storage",
"Coldline Storage",
"ローカル ストレージ"
]' shuffle="" stem="Cloud Storage は以下の 4 つのストレージ クラスを提供します。">
</ql-multiple-select-probe></p>
<h2 id="step6">オブジェクトをバケットにアップロードする</h2>
<p>このセクションでは、バケットにオブジェクトを追加します。このラボでは、オブジェクトは画像です。</p>
<ol>
<li>
<p>画像を取得します。この<a href="https://upload.wikimedia.org/wikipedia/commons/thumb/a/a4/Ada_Lovelace_portrait.jpg/800px-Ada_Lovelace_portrait.jpg">エイダ ラブレスのポートレートへのリンク</a>をクリックし、画像をローカルのパソコンに保存します。</p>
</li>
<li>
<p>[バケットの詳細] 画面が開いた状態になっています。エイダ ラブレスのポートレートを [バケットの詳細] 画面の [<strong>ここにファイルをドロップ</strong>] 領域にドラッグします。</p>
</li>
</ol>
<p><img alt="drag.png" src="https://cdn.qwiklabs.com/n1VFky%2BpITzj%2FVxVE3llbhvsvyv4sGTg2tuZZxo7ZaA%3D"></p>
<p>画像が [<strong>バケット</strong>] リストに表示されます。</p>
<ol start="3">
<li>
<p>バケット内の画像を削除します。ファイル名の横にあるチェックボックスをオンにして、[<strong>削除</strong>] をクリックします。次に、もう一度 [<strong>削除</strong>] をクリックして、選択したファイルを削除するかどうかを尋ねられたら、[はい] をクリックします。</p>
</li>
<li>
<p>もう一度画像をアップロードします。今回は [<strong>ファイルをアップロード</strong>] をクリックします。</p>
</li>
<li>
<p>ローカルのパソコン上で画像を見つけて、[<strong>開く</strong>] をクリックします。</p>
</li>
</ol>
<p>画像が [<strong>バケット</strong>] リストに表示されます。</p>
<ol start="6">
<li>ファイルの名前を変更します。画像の行の右端にあるプルダウン メニュー（3 つの点が縦に並んだアイコン）をクリックし、[<strong>名前を変更</strong>] を選択します。</li>
</ol>
<aside>プルダウン メニュー（3 つの点が縦に並んだアイコン）が表示されるように、必要に応じてブラウザ ウィンドウを広げてください。

</aside>
<ol start="7">
<li>ファイル名を「ada.jpg」に変更して、[<strong>名前を変更</strong>] をクリックします。</li>
</ol>
<p>バケットに <code>ada.jpg</code> という 1 つのファイルが表示されます。</p>

<h2 id="step8">オブジェクトを一般公開で共有する</h2>
<p>オブジェクトを一般公開する URL を作成するには、プルダウン メニュー（3 つの点が縦に並んだアイコン）をクリックします。</p>
<p>プルダウン メニューから [<strong>権限を編集</strong>] を選択します。</p>
<p>表示されるダイアログで、[<strong>+ 項目を追加</strong>] ボタンをクリックします。</p>
<p>以下のように入力して、すべてのユーザーに権限を追加します。</p>
<ul>
<li>[エンティティ] には [<strong>ユーザー</strong>] を選択します。</li>
<li>[名前] には「<strong>allUsers</strong>」と入力します。</li>
<li>[アクセス] には [<strong>読み取り</strong>] を選択します。</li>
</ul>
<p>最後に [<strong>保存</strong>] をクリックします。</p>
<p><img alt="permissions.png" src="https://cdn.qwiklabs.com/W9Ghvk%2BM%2BdNjbKT%2BUWLRRZNCO2aSXNgjst0oGY6M6uc%3D"></p>
<p>公開の状態で共有されると、公開アクセス列にリンクアイコンが表示されます。リンクをクリックすると、そのファイルが新しいタブで開きます。</p>

<h2 id="step9">フォルダを作成する</h2>
<p>このセクションでは、フォルダを作成します。</p>
<ol>
<li>
<p>ページの上部にある [フォルダを作成] リンクをクリックします。</p>
</li>
<li>
<p>フォルダの名前を「<code>folder1</code>」として、[<strong>作成</strong>] をクリックします。</p>
</li>
</ol>
<p>バケット内のフォルダには、オブジェクトと区別できるようにフォルダ アイコンが表示されます。</p>
<p><img alt="storage_console.png" src="https://cdn.qwiklabs.com/hsGrET8b6nFUDknvP3Vs%2B%2B2L7LkWqQj6gvVyafhNPLY%3D"></p>
<h3>サブフォルダを作成する</h3>
<p>次に「<code>folder1</code>」内にフォルダを作成し、ここにファイルをアップロードします。</p>
<ol>
<li>
<p><strong>folder1</strong> をクリックし、ページの上部にある [フォルダを作成] をクリックします。</p>
</li>
<li>
<p>フォルダの名前を「<code>folder2</code>」として、[<strong>作成</strong>] をクリックします。</p>
</li>
<li>
<p>作成した<strong>folder2</strong>をクリックします。</p>
</li>
<li>
<p>エイダ ラブレスのポートレートをローカルのパソコンからドラッグして、[<strong>ファイルをここにドロップ</strong>] 領域にドロップします。</p>
</li>
</ol>
<p>このファイルのアップロードが完了すると、バケットのサブフォルダ内に表示されます。</p>
<ol start="5">
<li>
<p>ファイル名を「ada.jpg」に変更します。</p>
</li>
<li>
<p>オブジェクトを一般公開する URL を作成するには、<code>ada.jpg</code> オブジェクトの行の右端にあるプルダウン メニュー（3 つの点が縦に並んだアイコン）をクリックします。</p>
</li>
<li>
<p>プルダウン メニューから [<strong>権限を編集</strong>] を選択します。</p>
</li>
<li>
<p>表示されるダイアログで、[<strong>+ 項目を追加</strong>] ボタンをクリックします。</p>
</li>
<li>
<p>以下のように入力して、すべてのユーザーに権限を追加します。</p>
</li>
</ol>
<ul>
<li>
<p>[エンティティ] には [<strong>ユーザー</strong>] を選択します。</p>
</li>
<li>
<p>[名前] には「<strong>allUsers</strong>」と入力します。</p>
</li>
<li>
<p>[アクセス] には [<strong>読み取り</strong>] を選択します。</p>
</li>
</ul>
<ol start="10">
<li>最後に [<strong>保存</strong>] をクリックします。</li>
</ol>
<p><img alt="permissions.png" src="https://cdn.qwiklabs.com/W9Ghvk%2BM%2BdNjbKT%2BUWLRRZNCO2aSXNgjst0oGY6M6uc%3D"></p>
<p>公開の状態で共有されると、公開アクセス列にリンクアイコンが表示されます。リンクをクリックすると、そのファイルが新しいタブで開きます。</p>
<aside>
この画像は、最初のコンピュータ プログラマーであると評される<a href="https://en.wikipedia.org/wiki/Ada_Lovelace">エイダ ラブレス</a>です。<a href="https://en.wikipedia.org/wiki/Analytical_Engine">解析エンジン</a>を提唱した数学者でコンピュータの先駆者でもあるチャールズ バベッジと共同で、研究に従事しました。彼女は解析エンジンに関心を持ち、イタリア人数学者ルイジ メナブレアが記述したマシンに関する記事を翻訳し、独自の注釈を加えました。この注釈が、最初のコンピュータ プログラム - 機械で実行されることを意図したアルゴリズムであると考えられています。ラブレスは、数値演算にとどまらないコンピュータ能力の構想を立ち上げ、個人や社会がコラボレーション ツールとしてのテクノロジーにどのように関わるべきかについて、研究を重ねました。
引用: <a href="https://commons.wikimedia.org/w/index.php?title=Ada_Lovelace&amp;oldid=176490980">Ada Lovelace</a>（最終更新: 2018 年 8 月 16 日）

</aside>
<h2 id="step10">フォルダを削除する</h2>
<p>このセクションでは、バケットから <code>folder1</code> とその内容を削除します。</p>
<ol>
<li>バケット/[ご自分のバケット] に戻ります。バケットの内容のリストに <code>folder1</code> が表示されます。</li>
<li>[名前] folder1 の横にあるチェックボックスをオンにして、[<strong>削除</strong>] をクリックし、確認するメッセージが表示されたら、もう一度 [<strong>削除</strong>] をクリックします。</li>
</ol>
<p><code>folder1</code> とその内容がバケットに表示されなくなりました。</p>
<h2 id="step11">これで完了です。</h2>

<p><a href="https://google.qwiklabs.com/quests/33">Baseline: Infrastructure</a> で、さらにクエストを続けましょう。クエストとは、学習パスを構成する一連のラボです。完了すると、成果が認められて上のバッジが贈られます。バッジは公開して、オンライン レジュメやソーシャル メディア アカウントからリンクすることができます。このラボを終えて<a href="http://google.qwiklabs.com/learning_paths/33/enroll">こちらのクエストに登録</a>すると、すぐにクレジットを受け取ることができます。<a href="http://google.qwiklabs.com/catalog">受講可能なその他の Qwiklabs のクエストもご覧ください</a>。</p>
<h2 id="step12">次のステップと詳細情報</h2>
<p>このラボは、Google Cloud が提供する多くの機能を体験できる「Qwik Starts」と呼ばれるラボシリーズの一部です。<a href="https://google.qwiklabs.com/catalog">ラボカタログ</a>で「Qwik Starts」を検索し、興味のあるラボを探してみてください。</p>
<h3>Google Cloud Training &amp; Certification</h3>
<p>Google Cloud 技術を最大限に活用できるようになります。<a href="https://cloud.google.com/training/courses">このクラス</a>では、必要な技術力とベスト プラクティスを習得し、継続的に学習することができます。トレーニングは基礎レベルから上級レベルまであり、オンデマンド、ライブ、仮想環境など、多忙なスケジュールに対応できるオプションが用意されています。<a href="https://cloud.google.com/certification/">認定資格</a>を取得することで、Google Cloud の技術のスキルと知識を証明できます。</p>

# Cloud Storage: Qwik Start - CLI / SDK（上と同じ内容をコマンドラインでやる）

<h2 id="step2">概要</h2>
<p>Google Cloud Storage を使うと、いつでも世界中のどこからでも、データを保存したり取得したりすることができます。データの量に制限はありません。Google Cloud Storage は、ウェブサイト コンテンツの配信や、アーカイブおよび災害復旧目的でのデータの保存、直接ダウンロードを通してのユーザーへの大規模なデータ オブジェクトの配布など、さまざまなシナリオで利用することができます。</p>
<p>このハンズオンラボでは、ストレージ バケットの作成、作成したバケットへのオブジェクトのアップロード、フォルダとサブフォルダの作成、Google Cloud Platform コマンドラインを使用したオブジェクトの一般公開などを行う方法について学習します。</p>
<p>Console で行ったラボの作業内容は、<strong>ナビゲーション メニュー</strong> &gt; [<strong>ストレージ</strong>] で確認できます。コマンドを実行したら、ブラウザを更新するだけで新しく作成したものが表示されます。</p>

<h3>バケットを作成する</h3>
<p>名前を固有のものに置き換えて gsutil mb コマンドを実行し、バケットを作成します。</p>
<pre><code>gsutil mb gs://YOUR-BUCKET-NAME/&#x000A;</code></pre>
<p><strong>バケットの命名規則は次のとおりです。</strong></p>
<ul>
<li>バケットの名前空間はグローバルであり、一般公開されるため、バケット名に機密情報を含めないでください。</li>
<li>バケット名に使用できる文字は、小文字、数字、ダッシュ（-）、下線（_）、ドット（.）のみです。ドットを使用している名前には<a href="https://cloud.google.com/storage/docs/domain-name-verification">確認</a>が必要です。</li>
<li>バケット名の先頭と末尾は数字か文字でなければなりません。</li>
<li>バケット名の長さは 3～63 文字とする必要があります。ドットを使用している名前には最大 222 文字を使用できますが、ドットで区切られている各要素は 63 文字以下としてください。</li>
<li>バケット名はドット区切りの十進表記（たとえば、192.168.5.4）の IP アドレス形式で表すことはできません。</li>
<li>バケット名の先頭に接頭辞「goog」は使用できません。</li>
<li>バケット名に「google」または「google」と類似する表記を含めることはできません。</li>
<li>また、DNS の基準に準拠し、将来的な互換性を確保するため、下線（_）を使用したり、ドットを連続して使用したり、ドットとダッシュを隣同士で使用したりしないでください。たとえば、「..」、「-.」、「.-」などの表記は DNS 名として使用することはできません。</li>
</ul>
<p>問題なければ、次の内容が返されます。</p>
<pre><code class="language-bash prettyprint">Creating gs://YOUR-BUCKET-NAME/...&#x000A;</code></pre>
<p>これでデータを保存できるバケットが作成されました。</p>
<ql-infobox>
<p><strong>注:</strong> 指定したバケット名がすでに使用されている場合は、次のメッセージが返されます。</p>
<p><code> Creating gs://YOUR-BUCKET-NAME/...</code> <code>ServiceException: 409 Bucket YOUR-BUCKET-NAME already exists.</code></p>
<p>異なるバケット名でもう一度お試しください。</p>
</ql-infobox>

<h2 id="step5">オブジェクトのバケットへのアップロード</h2>
<p>次に、オブジェクトをバケットにアップロードします。</p>
<p>まずこの画像を Cloud Shell で一時インスタンス（ada.jpg）としてダウンロードします。</p>
<pre><code>wget --output-document ada.jpg https://upload.wikimedia.org/wikipedia/commons/thumb/a/a4/Ada_Lovelace_portrait.jpg/800px-Ada_Lovelace_portrait.jpg&#x000A;</code></pre>
<p><code>gsutil cp</code> コマンドを使用して、画像を保存した場所から、上の手順で作成したバケットにアップロードします。</p>
<pre><code>gsutil cp ada.jpg gs://YOUR-BUCKET-NAME&#x000A;</code></pre>
<ql-infobox>
<strong>ヒント:</strong> バケット名を入力する際は、Tab キーでオートコンプリートを使用できます。
</ql-infobox>
<p>画像がバケットにアップロードされたことを、コマンドラインで確認します。これでオブジェクトをバケットに保存できました。</p>
<p>ここで、ダウンロードした画像を削除しておきます。</p>
<pre><code>rm ada.jpg&#x000A;</code></pre>
<h2 id="step6">バケットからオブジェクトをダウンロードする</h2>
<p><code>gsutil cp</code> コマンドを使用して、バケットに保存した画像を Cloud Shell にダウンロードします。</p>
<pre><code>gsutil cp -r gs://YOUR-BUCKET-NAME/ada.jpg .&#x000A;</code></pre>
<p>問題なければ、次の内容が返されます。</p>
<pre><code class="language-bash prettyprint">Copying gs://YOUR-BUCKET-NAME/ada.jpg...&#x000A;/ [1 files][299.6 KiB/299.6 KiB]&#x000A;Operation completed over 1 objects/299.6 KiB.&#x000A;</code></pre>
<p>バケットから画像をダウンロードできました。</p>
<h2 id="step7">バケット内のフォルダにオブジェクトをコピーする</h2>
<p><code>gsutil cp</code> コマンドを使用して <code>image-folder</code> という名前のフォルダを作成し、そこに画像（ada.jpg）をコピーします。</p>
<pre><code>gsutil cp gs://YOUR-BUCKET-NAME/ada.jpg gs://YOUR-BUCKET-NAME/image-folder/&#x000A;</code></pre>
<ql-infobox>
<strong>注:</strong> Google Cloud Storage のフォルダには、ローカル ファイル システムと比較すると<a href="https://cloud.google.com/storage/docs/gsutil/addlhelp/HowSubdirectoriesWork" target="_blank">いくつかの制約があります</a>が、多くの操作をローカル フォルダと同様に行うことができます。
</ql-infobox>
<p>問題なければ、次の内容が返されます。</p>
<pre><code class="language-bash prettyprint">Copying gs://YOUR-BUCKET-NAME/ada.jpg [Content-Type=image/png]...&#x000A;- [1 files] [ 299.6 KiB/ 299.6 KiB]&#x000A;Operation completed over 1 objects/299.6 KiB&#x000A;</code></pre>
<p>これで、バケット内の新しいフォルダに画像ファイルがコピーされました。</p>

<h2 id="step8">バケットまたはフォルダの内容の一覧表示</h2>
<p><code>gsutil ls</code> コマンドを使用して、バケットの内容を一覧表示します。</p>
<pre><code>gsutil ls gs://YOUR-BUCKET-NAME&#x000A;</code></pre>
<p>問題なければ、次のような内容が返されます。</p>
<pre><code class="language-bash prettyprint">gs://YOUR-BUCKET-NAME/ada.jpg&#x000A;gs://YOUR-BUCKET-NAME/image-folder/&#x000A;</code></pre>
<p>ここではバケット内にあるものすべてが表示されます。</p>
<h2 id="step9">オブジェクトの詳細を表示する</h2>
<p><code>gsutil ls</code> コマンドを <code>-l</code> フラグと一緒に使うと、バケットにアップロードした画像ファイルの詳細を表示することができます。</p>
<pre><code>gsutil ls -l gs://YOUR-BUCKET-NAME/ada.jpg&#x000A;</code></pre>
<p>問題なければ、次のような内容が返されます。</p>
<pre><code class="language-bash prettyprint">306768  2017-12-26T16:07:570Z  gs://YOUR-BUCKET-NAME/ada.jpg&#x000A;TOTAL: 1 objects, 30678 bytes (299.58 KiB)&#x000A;</code></pre>
<p>画像のサイズと作成日時が確認できます。</p>
<h2 id="step10">オブジェクトを一般公開する</h2>
<p><code>gsutil acl ch</code> コマンドを使用すると、バケットに保存されているオブジェクトの閲覧権限をすべてのユーザーに付与することができます。</p>
<pre><code>gsutil acl ch -u AllUsers:R gs://YOUR-BUCKET-NAME/ada.jpg&#x000A;</code></pre>
<p>問題なければ、次の内容が返されます。</p>
<pre><code class="language-bash prettyprint">Updated ACL on gs://YOUR-BUCKET-NAME/ada.jpg&#x000A;</code></pre>
<p>これで画像が一般公開され、誰もが見ることができるようになりました。</p>

<p>画像が一般公開されていることを確認しておきましょう。<strong>ナビゲーション メニュー</strong> &gt; [<strong>ストレージ</strong>] に移動してバケットの名前をクリックします。対象の画像の [<strong>公開リンク</strong>] ボックスがオンになっていることを確認します。ファイルの名前をクリックすると、新しいブラウザで画像が開かれます。</p>
<ql-infobox>
この画像は、最初のコンピュータ プログラマーであると評される<a href="https://en.wikipedia.org/wiki/Ada_Lovelace" target="_blank">エイダ ラブレス</a>です。<a href="https://en.wikipedia.org/wiki/Analytical_Engine" target="_blank">解析エンジン</a>を提唱した数学者でコンピュータの先駆者でもあるチャールズ バベッジと共同で、研究に従事しました。彼女は解析エンジンに関心を持ち、イタリア人数学者ルイジ メナブレアが記述したマシンに関する記事を翻訳し、独自の注釈を加えました。この注釈が、最初のコンピュータ プログラム - 機械で実行されることを意図したアルゴリズムであると考えられています。ラブレスは、数値演算にとどまらないコンピュータの持つ能力の構想を開発し、個人や社会がコラボレーション ツールとしてのテクノロジーにどのように関わるべきかについて、研究を重ねました。
<strong>引用:</strong> Ada Lovelace、<a href="https://commons.wikimedia.org/w/index.php?title=Ada_Lovelace&amp;oldid=176490980" target="_blank">https://commons.wikimedia.org/w/index.php?title=Ada_Lovelace&amp;oldid=176490980</a>（最終更新: 2017 年 12 月 6 日）
</ql-infobox>

<h2 id="step12">公開アクセス権を削除する</h2>
<p>上で付与した閲覧権限を削除するには、次のコマンドを使用します。</p>
<pre><code>gsutil acl ch -d AllUsers gs://YOUR-BUCKET-NAME/ada.jpg&#x000A;</code></pre>
<p>問題なければ、次の内容が返されます。</p>
<pre><code class="language-bash prettyprint">Updated ACL on gs://YOUR-BUCKET-NAME/ada.jpg&#x000A;</code></pre>
<p>これで、このオブジェクトの公開アクセス権が削除されました。Console で [<strong>更新</strong>] ボタンをクリックして、実際に検証してみることができます。チェックマークが表示されなくなっており、画像のページを再度読み込むと、エラーが表示されます。</p>

<h3>オブジェクトの削除</h3>
<p><code>gsutil rm</code> コマンドを使用すると、オブジェクト（バケット内の画像ファイル）を削除できます。</p>
<pre><code>gsutil rm gs://YOUR-BUCKET-NAME/ada.jpg&#x000A;</code></pre>
<p>問題なければ、次の内容が返されます。</p>
<pre><code class="language-bash prettyprint">Removing gs://YOUR-BUCKET-NAME/ada.jpg...&#x000A;</code></pre>
<p>Console で更新を行います。Cloud Storage に保存されていた画像ファイルのコピーがなくなっています（ただし、<code>image-folder/</code> フォルダ内に作成したコピーはなくなりません）。</p>
<h2 id="step14">これで終了です。</h2>

# Cloud IAM: Qwik Start
<h2 id="step2">概要</h2>
<p>GCP リソースに対する権限の作成と管理は、Google Cloud の Identity and Access Management（IAM）サービスを使って行います。Cloud IAM は、GCP サービスに関するアクセス制御を 1 つのシステムに統合し、一貫性のある運用プロセスを実現します。このハンズオンラボでは、2 人目のユーザーに役割を割り当てる方法や、Cloud IAM に紐づけた割り当て済みの役割を取り消す方法について学習します。具体的には、2 つの異なる認証情報セットでログインし、GCP プロジェクトのオーナーの役割と閲覧者の役割で権限の付与と取り消しを行う方法を確認します。</p>
<h2 id="step3">要件</h2>
<p>これは<strong>入門レベル</strong>のラボです。Cloud IAM の予備知識はほとんど、またはまったく必要ありません。Cloud Storage の使用経験はこのラボで作業を進めるのに役立ちますが、必須ではありません。開始にあたっては、使用可能な .txt または .html 形式のファイルをご用意ください。Cloud IAM に関する高度な演習をお探しの場合は、次のタブをご確認ください。</p>
<ul>
<li><a href="https://google.qwiklabs.com/catalog_lab/955">IAM Custom Roles</a></li>
</ul>
<p>準備ができたら下にスクロールし、手順に従ってラボ環境をセットアップします。</p>
<h2 id="step4">2 人のユーザーのセットアップ</h2>
<p>前述したように、このラボでは 2 つの認証情報セットを使用して、IAM ポリシーと、それぞれの役割で使用できる権限について説明します。</p>
<p>ラボの左側のパネルに、次のような認証情報のリストが表示されます。</p>
<p><img alt="2_users.png" src="https://cdn.qwiklabs.com/HvRjPlDrsdpXJmdjfWpV1C%2B6i5wxGswMCs1%2F2bfkhkU%3D"></p>
<p>ユーザー名 1 とユーザー名 2 の 2<em></em> つのユーザー名がありますのでご注意ください。これらは Cloud IAM での ID を示し、それぞれに異なるアクセス権限が割り当てられています。これらの GCP の「役割」により、割り当てられたプロジェクトで GCP リソースに対して行える作業と行えない作業が決まります。</p>

<h2 id="step5">IAM コンソールとプロジェクト レベルの役割</h2>
<ol>
<li>
<strong>ユーザー名 1</strong> の GCP Console ページに戻ります。</li>
<li>
<strong>ナビゲーション メニュー</strong> &gt; [<strong>IAM と管理</strong>] &gt; [<strong>IAM</strong>] を選択します。[IAM と管理] コンソールが表示されます。</li>
<li>ページ上部にある [<strong>+ 追加</strong>] ボタンをクリックし、[役割を選択] プルダウン メニューをクリックして、プロジェクトに関連付けられているプロジェクトの役割を表示します。</li>
</ol>
<p><img alt="iam-roles.gif" src="https://cdn.qwiklabs.com/Fj2raSqxxSSaysd18h01Ky0DK7M%2BpUc%2FJsmmSDhvPno%3D"></p>
<p>参照者、編集者、オーナー、閲覧者の役割が表示されますが、これらは GCP の基本の役割です。<em></em>基本の役割はプロジェクト レベルの権限を設定するもので、別途指定しない限り、すべての GCP サービスに対するアクセスと管理を制御します。</p>
<p>次の表は、<a href="https://cloud.google.com/iam/docs/understanding-roles#primitive_roles">GCP の役割について説明しているドキュメント</a>の定義を抜粋したもので、参照者、閲覧者、編集者、オーナーのそれぞれの役割の権限が簡潔にまとめられています。</p>
<table>
<tr>
<td colspan="1" rowspan="1">
<p><strong>役割名</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>権限</strong></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>roles/viewer</p>
</td>
<td colspan="1" rowspan="1">
<p>既存のリソースやデータの表示（ただし変更は不可能）など、状態に影響しない読み取り専用アクションに必要な権限。</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>roles/editor</p>
</td>
<td colspan="1" rowspan="1">
<p>すべての閲覧者権限と、状態を変更するアクション（既存のリソースの変更など）に必要な権限。</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>roles/owner</p>
</td>
<td colspan="1" rowspan="1">
<p>すべての編集者権限と、以下のアクションを実行するために必要な権限。</p>
<ul>
<li>プロジェクトおよびプロジェクト内のすべてのリソースの権限と役割を管理する。</li>
<li>プロジェクトの課金情報を設定する。</li>
</ul>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>roles/browser（ベータ版）</p>
</td>
<td colspan="1" rowspan="1">
<p>
フォルダ、組織、Cloud IAM ポリシーなど、プロジェクトの階層を参照するための読み取りアクセス権。この役割には、プロジェクトのリソースを表示する権限は含まれていません。</p>
</td>
</tr>
</table>
<p>ユーザー名 1 でこのプロジェクトの役割と権限を管理できたので、ユーザー名 1 はプロジェクト オーナーの権限を持っていることがわかります。</p>
<ol start="4">
<li>
<p>[<strong>キャンセル</strong>] をクリックして [メンバーを追加] パネルを終了します。</p>
</li>
</ol>
<h3>編集者の役割を調べる</h3>
<p>次に、<strong>ユーザー名 2</strong> コンソールに切り替えます。</p>
<ol>
<li>
<p>[IAM と管理] コンソールに移動し、<strong>ナビゲーション メニュー</strong> &gt; [<strong>IAM と管理</strong>] &gt; [<strong>IAM</strong>] を選択します。</p>
</li>
<li>
<p>テーブルでユーザー名 1 とユーザー名 2 を検索し、それらに付与されている役割を確認します。次のように表示されます。</p>
</li>
</ol>
<p><img alt="user-roles.png" src="https://cdn.qwiklabs.com/RaAyxDSY%2FVrK0ziQlHLdMwvkAUY3rftmvxp5Nvndwcg%3D"></p>
<p>以下のように表示されます。</p>
<ul>
<li>ユーザー名 2 には「閲覧者」の役割が付与されています。</li>
<li>上部にある [<strong>+ 追加</strong>] ボタンはグレー表示されており、クリックしようとすると次のメッセージが表示されます。</li>
</ul>
<p><img alt="more-permissions.png" src="https://cdn.qwiklabs.com/cNeK9TAhskx4dQPXm4lmHqPk9Vb14sS7KFHtmnW150A%3D"></p>
<p>これは GCP で実行できることとできないことが IAM の役割によって決まることを示す一例です。</p>
<ol start="3">
<li>
<p>次のステップのために<strong>ユーザー名 1</strong> のコンソールに切り替えます。</p>
</li>
</ol>
<h2 id="step6">アクセステストのためのリソースの準備</h2>
<p><strong>ユーザー名 1</strong> のコンソールにいることを確認します。</p>
<h3>バケットの作成</h3>
<ol>
<li>
<p>一意の名前で GCS バケットを作成します。コンソールから<strong>ナビゲーション メニュー</strong> &gt; [<strong>Storage</strong>] &gt; [<strong>ブラウザ</strong>] を選択します。</p>
</li>
<li>
<p>[<strong>バケットを作成</strong>] をクリックします。</p>
</li>
</ol>
<aside>
<b>注</b>: バケットの作成で権限エラーが発生した場合は、ログアウトしてからユーザー名 1 の認証情報でログインし直してください。

</aside>
<ol start="3">
<li>次のフィールドを更新し、他のフィールドはすべてデフォルト値のままにします。</li>
</ol>
<table>
<tr>
<td colspan="1" rowspan="1">
<p><strong>特性</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>値</strong></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p><strong>名前</strong>：</p>
</td>
<td colspan="1" rowspan="1">
<p><em>グローバルな一意の名前（自分で作成してください）</em></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p><strong>デフォルトのストレージ クラス:</strong></p>
</td>
<td colspan="1" rowspan="1">
<p>Multi-Regional</p>
</td>
</tr>
</table>
<p>バケット名をメモしてください。この値は、後のステップで使用します。</p>
<ol start="4">
<li>[<strong>作成</strong>] をクリックします。</li>
</ol>
<aside>
<b>注</b>: バケットの作成で権限エラーが発生した場合は、ログアウトしてからユーザー名 1 の認証情報でログインし直してください。

</aside>
<h3><strong>サンプル ファイルのアップロード</strong></h3>
<ol>
<li>
<p>[バケットの詳細] ページで、[<strong>ファイルをアップロード</strong>] ボタンをクリックします。</p>
</li>
<li>
<p>パソコン内を探して、使用するファイルを見つけます。任意のテキスト ファイルまたは html ファイルを使用できます。</p>
</li>
<li>
<p>ファイルがある行の最後にあるその他アイコンをクリックし、[<strong>名前を変更</strong>] をクリックします。</p>
</li>
<li>
<p>ファイル名を「<code>sample.txt</code>」に変更します。</p>
</li>
<li>
<p>[<strong>名前を変更</strong>] をクリックします。</p>
</li>
</ol>

<h3><strong>プロジェクト閲覧者のアクセス権の確認</strong></h3>
<ol>
<li>
<p><strong>ユーザー名 2</strong> のコンソールに切り替えます。</p>
</li>
<li>
<p>コンソールから<strong>ナビゲーション メニュー</strong> &gt; [<strong>Storage</strong>] &gt; [<strong>ブラウザ</strong>] を選択します。このユーザーがバケットを表示できることを確認します。</p>
</li>
</ol>
<p>ユーザー名 2 には閲覧者の役割が付与されており、状態に影響を与えない読み取り専用アクションが許可されています。この例では、閲覧者が、アクセス権を付与された GCP プロジェクトでホストされている Cloud Storage バケットとファイルを表示できることがわかります。</p>
<h2 id="step7">プロジェクトへのアクセス権の削除</h2>
<p><strong>ユーザー名 1</strong> のコンソールに切り替えます。</p>
<h3><strong>ユーザー名 2 のプロジェクト閲覧者の権限の削除</strong></h3>
<ol>
<li>
<strong>ナビゲーション メニュー</strong> &gt; [<strong>IAM と管理</strong>] &gt; [<strong>IAM</strong>] を選択します。<strong>ユーザー名 2</strong> の横にある鉛筆アイコンをクリックします。</li>
</ol>
<aside>
鉛筆アイコンが表示されるように、必要に応じて画面を広げてください。

</aside>
<p><img alt="edit.png" src="https://cdn.qwiklabs.com/klPjwWYwm8JsmtfNIjAw2jy5jblVy68CIyTeqiAJRcc%3D"></p>
<ol start="2">
<li>役割名の横にあるゴミ箱アイコンをクリックし、<strong>ユーザー名 2</strong> のプロジェクト閲覧者のアクセス権を削除します。最後に [<strong>保存</strong>] をクリックします。</li>
</ol>
<p><img alt="IAM_role_trash.png" src="https://cdn.qwiklabs.com/J3a%2BJeH31q7Sk1IFkOPBWSO13jbUkP3duf5nn%2F9rKp4%3D"></p>
<p>このユーザーがリストから消去されていると、アクセス権が取り消されていることになります。</p>
<aside>
<b>注</b>: 変更が反映されるまで 80 秒ほどかかることがあります。詳しくは<a href="https://cloud.google.com/iam/docs/faq">こちら</a>をご覧ください。

</aside>
<h3><strong>ユーザー名 2 がアクセス権を失ったことの確認</strong></h3>
<ol>
<li>
<p><strong>ユーザー名 2</strong> のコンソールに切り替えます。ユーザー名 2 の認証情報でログインしたままで、権限が取り消された後にプロジェクトからログアウトしていないことを確認します。ログアウトしている場合は、適切な認証情報で再度ログインしてください。</p>
</li>
<li>
<p><strong>ナビゲーション メニュー</strong> &gt; [<strong>ホーム</strong>] を選択します。</p>
</li>
<li>
<p><strong>ナビゲーション メニュー</strong> &gt; [<strong>Storage</strong>] &gt; [<strong>ブラウザ</strong>] を選択して、Cloud Storage に戻ります。</p>
</li>
</ol>
<p>次のような権限エラーが表示されます。</p>
<p><img alt="Ipermission-error.png" src="https://cdn.qwiklabs.com/3kzLKfVcqhSq7UsD%2BpIg8rLfiTVSAuS2p%2FcNYPNcO90%3D"></p>
<aside>
<b>注</b>: 前述したように、権限が取り消されるまでに 80 秒ほどかかることがあります。権限エラーが発生しなかった場合は、2 分待ってからコンソールを更新してみてください。

</aside>

<h2 id="step8">Storage の権限の追加</h2>
<ol>
<li>Qwiklabs の [接続の詳細] パネルから名前「<strong>ユーザー名 2</strong>」をコピーします。</li>
</ol>
<p><img alt="2_users.png" src="https://cdn.qwiklabs.com/HvRjPlDrsdpXJmdjfWpV1C%2B6i5wxGswMCs1%2F2bfkhkU%3D"></p>
<ol start="2">
<li>
<p><strong>ユーザー名 1</strong> のコンソールに切り替えます。ユーザー名 1 の認証情報でログインしたままであることを確認します。ログアウトしている場合は、適切な認証情報で再度ログインしてください。</p>
</li>
<li>
<p>コンソールで、<strong>ナビゲーション メニュー</strong> &gt; [<strong>IAM と管理</strong>] &gt; [<strong>IAM</strong>] を選択します。</p>
</li>
<li>
<p>[<strong>+ 追加</strong>] をクリックして、<strong>ユーザー名 2</strong> の名前を [新しいメンバー] フィールドに貼り付けます。</p>
</li>
<li>
<p>[<strong>役割を選択</strong>] のプルダウン メニューから、[<strong>Cloud Storage</strong>] &gt; [<strong>ストレージ オブジェクト閲覧者</strong>] を選択します。</p>
</li>
<li>
<p>[<strong>保存</strong>] をクリックします。</p>
</li>
</ol>
<h2 id="step9">アクセス権の確認</h2>
<ol>
<li>
<strong>ユーザー名 2</strong> のコンソールに切り替えます。</li>
</ol>
<p><strong>ユーザー名 2</strong> にはプロジェクト閲覧者の役割がないため、コンソールにプロジェクトやプロジェクトのリソースは表示されません。ただし、このユーザーには Cloud Storage に対する特別なアクセス権があります。そのアクセス権を確認してみましょう。</p>
<ol start="2">
<li>[<strong>Cloud Shell をアクティブにする</strong>] アイコンをクリックし、Cloud Shell コマンドラインを開きます。</li>
</ol>
<p><img alt="start-cloud-shell.png" src="https://cdn.qwiklabs.com/MwTIT1EPNZG9oo8k53JUFkWo3YG2G0Uk31h9Tfhh4M8%3D"></p>
<ol start="3">
<li>
<p>Cloud Shell セッションを開き、次のコマンドを入力します。<code>[YOUR_BUCKET_NAME]</code> は前に作成したバケットの名前に置き換えます。</p>
</li>
</ol>
<pre><code class="language-bash prettyprint">gsutil ls gs://[YOUR_BUCKET_NAME]&#x000A;</code></pre>
<ol start="3">
<li>
<p>次のような出力が返されます。</p>
</li>
</ol>
<pre><code class="language-bash prettyprint">gs://[YOUR_BUCKET_NAME]/sample.txt&#x000A;</code></pre>
<ol start="4">
<li>これにより、<strong>ユーザー名 2</strong> に Cloud Storage バケットへの表示アクセス権が付与されていることがわかります。</li>
</ol>

# Cloud Monitoring: Qwik Start
<h2 id="step2">概要</h2>
<p>Cloud Monitoring では、クラウドで実行されるアプリケーションのパフォーマンスや稼働時間、全体的な動作状況を確認できます。Cloud Monitoring は、Google Cloud、Amazon Web Services、ホストされた稼働時間プローブ、アプリケーション インストゥルメンテーション、よく使われるさまざまなアプリケーション コンポーネント（Cassandra、Nginx、Apache ウェブサーバー、Elasticsearch など）から、指標、イベント、メタデータを収集します。データを取り込むと、Cloud Monitoring はダッシュボード、グラフ、アラートを使用して分析情報を提供します。Cloud Monitoring のアラート機能を Slack、PagerDuty、HipChat、Campfire などと統合すると、共同作業に便利です。</p>
<p>このハンズオンラボでは、Google Compute Engine 仮想マシン（VM）インスタンスを Cloud Monitoring でモニタリングする方法を学びます。さらに、インスタンスから情報を収集する VM に Monitoring エージェントと Logging エージェントをインストールします。収集する情報には、サードパーティ製アプリの指標やログが含まれる場合があります。</p>

<h2 id="step4">Compute Engine インスタンスを作成する</h2>
<ol>
<li>Cloud Console ダッシュボードで、<strong>ナビゲーション メニュー</strong> &gt; [<strong>Compute Engine</strong>] &gt; [<strong>VM インスタンス</strong>] に移動し、[<strong>作成</strong>] をクリックします。</li>
</ol>
<p><img alt="nav_compute.png" src="https://cdn.qwiklabs.com/48XM2fQQVCp7ObiNAyXTPs7VFh%2BuW7M2gyc3pPGK62E%3D"></p>
<ol start="2">
<li>
<p>以下の各フィールドを次のように指定し、他のフィールドはデフォルト値のままにします。</p>
<p><strong>名前</strong>: lamp-1-vm</p>
<p><strong>リージョン</strong>: us-central1（アイオワ）または asia-south1（ムンバイ）</p>
<p><strong>ゾーン</strong>: us-central1-a または asia south1-a</p>
<p><strong>マシンタイプ</strong>: n1-standard-2</p>
<p><strong>ファイアウォール</strong>: [HTTP トラフィックを許可する] を選択</p>
</li>
<li>
<p>[<strong>作成</strong>] をクリックします。</p>
</li>
</ol>
<p><img alt="Stackdriver_QS_vm.png" src="https://cdn.qwiklabs.com/h%2FKKsrehia9UP3lT6v7BJzCxZ4kf9TCEm%2Be4kyxUe84%3D"></p>

<h2 id="step5">インスタンスに Apache2 HTTP Server を追加する</h2>
<ol>
<li>コンソールで [<strong>SSH</strong>] をクリックして、インスタンスのターミナルを開きます。</li>
</ol>
<p><img alt="SSH.png" src="https://cdn.qwiklabs.com/157XnA6MKqd0jjbTNhOenyTQWI31YJZVWQUn30E8wTY%3D"></p>
<ol start="2">
<li>
<p>SSH ウィンドウで次のコマンドを実行し、Apache2 HTTP Server をセットアップします。</p>
</li>
</ol>
<pre><code>sudo apt-get update&#x000A;</code></pre>
<pre><code>sudo apt-get install apache2 php7.0&#x000A;</code></pre>
<p>続行を確認するメッセージが表示されたら、「<strong>Y</strong>」と入力します。</p>
<aside class="special"><p><strong>注:</strong> php7.0 をインストールできない場合は、php5 を使用してください。</p>
</aside>
<pre><code>sudo service apache2 restart&#x000A;</code></pre>

<ol start="3">
<li>コンソールの VM インスタンスのページに戻ります。<code>lamp-1-vm</code> インスタンスの外部 IP をクリックして、このインスタンスの Apache2 のデフォルト ページを表示します。</li>
</ol>
<p><img alt="ext_IP_address.png" src="https://cdn.qwiklabs.com/IZlq%2Fs%2Fjkwv08kKY5OMQouAGmM2e%2BZD83AOMfY1LVqM%3D"></p>
<p><img alt="d1b14dc18bc7a72d.png" src="https://cdn.qwiklabs.com/UYEHvdQG6fv%2FFFuZk6xiz4oSzaVznBOY%2BrvHOFPnpt8%3D"></p>

<h3>モニタリング ワークスペースを作成する</h3>
<p>ここで、Qwiklabs GCP プロジェクトに関連付けられたモニタリング ワークスペースを設定します。次の手順にて、モニタリングの無料トライアル付きの新しいアカウントを作成します。</p>
<p>1. Google Cloud Platform コンソールで、[<strong>ナビゲーション メニュー </strong>] &gt; [<strong>モニタリング</strong>] をクリックします。</p>
<p>2. ワークスペースがプロビジョニングされるのを待ちます。</p>
<p>モニタリング ダッシュボードが開いたら、モニタリング ワークスペースが利用できます。</p>
<p><img alt="monitoring_overview.png" src="https://cdn.qwiklabs.com/BGS8QF9AXRWwP0pKwugIg7rSBz9hP%2B2r88V4czj0m4o%3D"></p>

<h3>Monitoring エージェントと Logging エージェントをインストールする</h3>
<p>エージェントはデータを収集し、コンソール内の Cloud Monitoring に情報を送信またはストリーミングします。</p>
<p>Cloud Monitoring エージェント<em></em>は、collectd ベースのデーモンで、仮想マシン インスタンスからシステム指標とアプリケーション指標を収集して Monitoring に送信します。デフォルトでは、Monitoring エージェントはディスク、CPU、ネットワーク、プロセスの指標を収集します。Monitoring エージェントを構成すると、サードパーティ アプリケーションでエージェント指標の包括的なリストを取得することができます。詳細</p>
<p>Cloud Logging エージェント<em></em>は、VM インスタンスや選択したサードパーティ ソフトウェア パッケージから取得したログを Cloud Logging にストリーミングします。すべての VM インスタンスで Cloud Logging エージェントを実行することをおすすめします。詳細</p>
<p>VM にエージェントをインストールするには:</p>
<ol>
<li>[Monitoring の概要] ウィンドウで、[<strong>Install agents</strong>] タイル内にある [<strong>INSTALL AGENTS</strong>] をクリックします。</li>
</ol>
<p><img alt="install_agents.png" src="https://cdn.qwiklabs.com/i6WE%2FhZDlMJPvl3ENl6gojIwIH5NpsO%2FJPmQVi4tATA%3D"></p>
<p>Monitoring エージェントのウィンドウが開き、Monitoring エージェントと Logging エージェントのインストール スクリプトがそれぞれ表示されます。</p>
<ol start="2">
<li>VM インスタンスの SSH ターミナルで Monitoring エージェント インストール スクリプトのコマンドを実行して、Cloud Monitoring エージェントをインストールします。</li>
</ol>
<p><img alt="monitoring-agent-install.png" src="https://cdn.qwiklabs.com/otppQOdeSvrGDLUC4eyw5G15b4EE5uWDLqUNvdGWWBU%3D"></p>
<ol start="3">
<li>VM インスタンスの SSH ターミナルで Logging エージェント インストール スクリプトのコマンドを実行して、Cloud Logging エージェントをインストールします。</li>
</ol>
<p><img alt="logging-agent-install.png" src="https://cdn.qwiklabs.com/nXmqfk7yq98yljAcQWdbA3N5X9QWmsfZaNbVQTxkwdU%3D"></p>
<h2 id="step6">稼働時間チェックを作成する</h2>
<p>稼働時間チェックでは、リソースが常にアクセス可能かどうかを確認します。練習のために、稼働時間チェックを作成して VM が稼働していることを確認します。</p>
<ol>
<li>コンソールの左側のメニューで、[<strong>稼働時間チェック</strong>] をクリックし、[<strong>稼働時間チェックの作成</strong>] をクリックします。</li>
</ol>
<p><img alt="create_uptime_check.png" src="https://cdn.qwiklabs.com/mdt35y5ElNzYoCkdrVkodx81TWTuTWAbBYfoj0Ps5r4%3D"></p>
<ol start="2">
<li>次のフィールドを設定します。</li>
</ol>
<p><strong>Title</strong>: Lamp Uptime Check</p>
<p><strong>Check type</strong>: HTTP</p>
<p><strong>Resource Type</strong>: Instance</p>
<p><strong>Applies To</strong>: Single、lamp-1-vm</p>
<p><strong>Path</strong>: デフォルトのまま</p>
<p><strong>Check every:</strong> 1 min</p>
<p><img alt="783246b4ceb3af54.png" src="https://cdn.qwiklabs.com/nUzuLABHMWMATVt6KBYX0qiMA2aLwgSupusp2oK31Gs%3D"></p>
<ol start="3">
<li>
<p>[<strong>Test</strong>] をクリックして、稼働時間チェックがリソースに接続できることを確認します。</p>
</li>
<li>
<p>緑色のチェックマークが表示されたら、問題なく接続できています。[<strong>Save</strong>] をクリックします。</p>
</li>
<li>
<p>このチェックのアラート ポリシーを作成するかどうか尋ねるメッセージが表示されたら、[<strong>No, thanks</strong>] をクリックします。</p>
</li>
</ol>
<p>構成した稼働時間チェックがアクティブになるには、少し時間がかかります。ラボを続行し、後で結果を確認します。待っている間に、別のリソースのアラート ポリシーを作成します。</p>
<h2 id="step7">アラート ポリシーを作成する</h2>
<p>Cloud Monitoring を使用して 1 つ以上のアラート ポリシーを作成します。</p>
<ol>
<li>
<p>左側のメニューで [<strong>アラート</strong>]、[<strong>Create policy</strong>] の順にクリックします。</p>
</li>
<li>
<p>このアラート ポリシーに「Inbound Traffic Alert」という名前を付けます。</p>
</li>
</ol>
<p>次に、Conditions, Notifications と Documents を設定します。</p>
<ol>
<li>
<strong>Conditions</strong>: [<strong>Add condition</strong>] をクリックします。</li>
</ol>
<p>表示されたパネルで次のように設定し、他のフィールドはデフォルト値のままにします。</p>
<p><strong>Target</strong>:
Resource type and metric のフィールドに「GCE」と入力してから、以下のように選択します。</p>
<ul>
<li>
<p><strong>Resource type</strong>: GCE VM instance（gce_instance）</p>
</li>
<li>
<p><strong>Metric</strong>: 「network」と入力してから、network traffic（gce_instance+1）を選択します。<code>agent.googleapis.com/interface/traffic</code> の network traffic リソースを選択するようにします。</p>
<p><img alt="alert-target.png" src="https://cdn.qwiklabs.com/l2K91yBTLpR5asD9rXP%2FZJieVeYI9yxjso4UlOvt1ps%3D"></p>
</li>
</ul>
<p><strong>Configuration</strong></p>
<ul>
<li>
<p><strong>Condition</strong>: is above</p>
</li>
<li>
<p><strong>Threshold</strong>: 500</p>
</li>
<li>
<p><strong>for</strong>: 1 minute</p>
<p><img alt="alert-condition.png" src="https://cdn.qwiklabs.com/IMlbSx56lUSSGwsz26XBBp8PtGhvWTGhCtHDAyFQmPk%3D"></p>
</li>
</ul>
<p>[<strong>Add</strong>] をクリックします。</p>
<ol start="2">
<li>
<strong>Notifications</strong>: [<strong>Add Notification Channel</strong>] をクリックします。</li>
</ol>
<p>[<strong>Email</strong>] を選択し、[<strong>メール</strong>] フィールドに自分のメールアドレスを入力します。</p>
<p><img alt="email.png" src="https://cdn.qwiklabs.com/5bJ93siu12bF2M87sL2McI3jgC9tWOBYkHhOhWgqkzQ%3D"></p>
<ol start="3">
<li>
<p><strong>Documentation</strong>: ドキュメントにメッセージを追加します。このメッセージは、メールで送信されるアラートに表示されます。</p>
<p><img alt="documentation.png" src="https://cdn.qwiklabs.com/NzG%2FobEp3eFEDdFYboAGO95vj0sF93kjFAjfy%2BPgOQs%3D"></p>
</li>
</ol>
<p>[新しいアラート ポリシーの作成] ダイアログの下部にある [<strong>Save</strong>] をクリックします。</p>
<p>これでアラートが作成されました。システムによってアラートがトリガーされるのを待つ間、ダッシュボードとグラフを作成し、Cloud Logging を確認します。</p>

<h2 id="step8">ダッシュボードとグラフを作成する</h2>
<p>Cloud Monitoring で収集された指標は、ユーザー独自のグラフやダッシュボードに表示できます。このセクションでは、ラボの指標を表すグラフと、カスタム ダッシュボードを作成します。</p>
<ol>
<li>
<p>左側のメニューで、[<strong>ダッシュボード</strong>] を選択し、[<strong>Create Dashboard</strong>] を選択します。</p>
</li>
<li>
<p>ダッシュボードに「Cloud monitoring LAMP Qwik Start Dashboard」という名前を付け、[<strong>確認</strong>] をクリックします。</p>
</li>
</ol>
<h3>1 つ目のグラフを追加する</h3>
<ol>
<li>
<p>画面の右上で [<strong>Add chart</strong>] をクリックします。</p>
</li>
<li>
<p>グラフに「CPU Load」という名前を付けます。</p>
</li>
<li>
<p>[<strong>Find resource type and metric</strong>] フィールド内をクリックし、「CPU load」と入力して CPU Load（1 m）を選択します。</p>
</li>
<li>
<p>Resource type を [GCE VM instance] に設定します。</p>
</li>
</ol>
<p><img alt="a92b8e31e569d19d.png" src="https://cdn.qwiklabs.com/tBsIRGuAmtNLQTWiKVXJiL%2FADaaO1gndhRfjoyDsXJQ%3D"></p>
<ol start="5">
<li>
<p>[<strong>Save</strong>] をクリックします。</p>
</li>
</ol>
<h3>2 つ目のグラフを追加する</h3>
<ol>
<li>
<p>右上の [<strong>Add Chart</strong>] を選択します。</p>
</li>
<li>
<p>このグラフに「Recieved Packets」という名前を付けます。</p>
</li>
<li>
<p>指標のフィールドで次のことを実行します。</p>
</li>
</ol>
<ul>
<li>
<p>CPU load（1）の指標の横にある [<strong>X</strong>] をクリックして指標を削除します。</p>
</li>
<li>
<p>「Network」と入力して「Received Packets」を選択します。</p>
</li>
</ul>
<ol start="4">
<li>
<p>他のフィールドはデフォルト値のままにします。[プレビュー] セクションにグラフのデータが表示されます。</p>
</li>
<li>
<p>[<strong>Save</strong>] をクリックします。</p>
</li>
</ol>
<h2 id="step9">ログを表示する</h2>
<p>Cloud Monitoring と Cloud Logging は密接に統合されています。ラボのログを確認しましょう。</p>
<ol>
<li>
<p><strong>ナビゲーション メニュー</strong> &gt; [<strong>Logging</strong>] &gt; [<strong>ログビューア</strong>] を選択します。</p>
</li>
<li>
<p>表示するログを選択します。この場合、このラボの最初に作成した lamp-1-vm インスタンスのログを選択します。</p>
</li>
</ol>
<ul>
<li>リソース（1 つ目）のプルダウン メニューで、[<strong>GCE VM インスタンス</strong>] &gt; [<strong>lamp-1-vm</strong>] を選択します。</li>
<li>ログ（2 つ目）のプルダウン メニューで、[<strong>syslog</strong>] を選択し、[<strong>OK</strong>] をクリックします。</li>
<li>他のフィールドはデフォルト値のままにします。</li>
<li>
<p>「<strong>ログのストリーミングを開始する</strong>」アイコンをクリックします。</p>
<img alt="streaming_logs.png" src="https://cdn.qwiklabs.com/6IraJ8XzEEqzROQFwM9Hf9bCUPAliP%2BvfW6iwzrvp5w%3D">
</li>
</ul>
<p>VM インスタンスのログが表示されます。</p>
<p><img alt="3abc0bd1765ae743.png" src="https://cdn.qwiklabs.com/34rkXIFn4VAsHTTAhD7WrHu20qUx6Cb0v6xAj4DA9S4%3D"></p>
<h3>VM インスタンスを開始および停止するとどうなるかを確認します。</h3>
<p>Cloud Monitoring と Cloud Logging によって VM インスタンスの変更が反映されるのを適切に確認するには、1 つのブラウザ ウィンドウでインスタンスに変更を加え、まず Cloud Monitoring で何が起こるかを確認し、その後 Cloud Logging ウィンドウで確認します。</p>
<ol>
<li>新しいブラウザ ウィンドウで Compute Engine ウィンドウを開きます。<strong>ナビゲーション メニュー</strong> &gt; [<strong>Compute Engine</strong>] を選択し、[<strong>VM インスタンス</strong>] を右クリックして [<strong>新しいウィンドウで開く</strong>] を選択します。</li>
<li>ブラウザの [ログビューア] ウィンドウを Compute Engine ウィンドウの横に移動します。こうすることで、VM への変更がログにどのように反映されるかを簡単に確認できます。</li>
</ol>
<p><img alt="both-consoles.png" src="https://cdn.qwiklabs.com/Zh6mz5qTeKgDvcEzDQje71r0gmY4v1mX0n15TKdy1zk%3D"></p>
<ol start="3">
<li>Compute Engine ウィンドウで lamp-1-vm インスタンスを選択し、画面上部にある [<strong>停止</strong>] をクリックしてインスタンスが停止するのを確認します。</li>
</ol>
<p><img alt="stop-vm.png" src="https://cdn.qwiklabs.com/Cc091icpheV9nzMvh1ZKwdETxIk4UztCuM9COeYfFJo%3D"></p>
<p>インスタンスが停止するまでに数分かかります。</p>
<ol start="4">
<li>
<p>[ログビューア] タブで、VM が停止するのを確認します。</p>
</li>
<li>
<p>VM インスタンスの詳細ウィンドウで、画面上部の [<strong>開始</strong>] をクリックして動作を確認します。
インスタンスが再開されるまでに数分かかります。ログのメッセージを確認して、起動をモニタリングします。</p>
</li>
</ol>
<h2 id="step10">稼働時間チェックの結果とトリガーされたアラートを確認する</h2>
<h3>Cloud Monitoring で稼働時間チェックの結果を確認します。</h3>
<ol>
<li>Cloud Logging ウィンドウで、<strong>ナビゲーション メニュー</strong> &gt; [<strong>モニタリング</strong>] &gt; [<strong>稼働時間チェック</strong>] を選択します。このビューでは、異なるロケーションのアクティブな稼働時間チェックとステータスを一覧で確認できます。</li>
</ol>
<p>「Lamp Uptime Check」が含まれていることがわかります。インスタンスを再起動したばかりなので、リージョンは失敗のステータスになっています。リージョンがアクティブになるまでに、最長で 5 分ほどかかる場合があります。リージョンがアクティブになるまで、必要に応じてブラウザ ウィンドウを再読み込みします。</p>
<ol start="2">
<li>稼働時間チェックの名前（Lamp Uptime Check）をクリックします。</li>
</ol>
<p>インスタンスを再起動したばかりなので、リージョンがアクティブになるまでに数分かかる場合があります。必要に応じてブラウザ ウィンドウを再読み込みします。</p>
<h3>アラートがトリガーされているかどうかを確認する</h3>
<ol>
<li>
<p>左側のメニューで、[<strong>アラート</strong>] をクリックします。</p>
</li>
<li>
<p>[アラート] ウィンドウにインシデントとイベントが表示されていることを確認できます。</p>
</li>
<li>
<p>ご自分のメール アカウントを確認します。Cloud Monitoring のアラートを受信しているはずです。</p>
</li>
</ol>

# Cloud Functions: Qwik Start - Console
<h2 id="step2">概要</h2>
<p>Google Cloud Functions は、クラウド サービスの構築と接続に使用できるサーバーレスのランタイム環境です。Cloud Functions を使用すると、クラウドのインフラストラクチャやサービスで生じたイベントに関連する、単一目的のシンプルな関数を作成することができます。対象のイベントが発生すると、Cloud Functions の関数がトリガーされ、コードがフルマネージドの環境で実行されます。インフラストラクチャをプロビジョニングする必要はなく、サーバーの管理に悩まされることもありません。</p>
<p>Cloud Functions の関数は JavaScript で作成され、Google Cloud Platform の Node.js 環境で実行されます。（Python, Go も可）標準的な Node.js ランタイムで実行できるため、環境に依存せず、ローカルのテストをスムーズに行えます。</p>
<h3>クラウド サービスの接続と拡張</h3>
<p>Cloud Functions では、論理リンク層を使用して複数のクラウド サービスを接続・拡張するコードを記述できます。たとえば、Cloud Storage へのファイルのアップロード、ログの変更、Cloud Pub/Sub トピックに対するメッセージの着信をリッスンし、応答することが可能です。Cloud Functions を使用することで、既存のクラウド サービスを強化し、独自のプログラミング ロジックでより多くのユースケースに対応できます。また、Google サービス アカウントの認証情報へのアクセスにより、Datastore、Cloud Spanner、Cloud Translation API、Cloud Vision API など、大半の Google Cloud Platform サービスでシームレスに認証を行えます。さらに、非常に多くの <a href="https://cloud.google.com/nodejs/apis">Node.js クライアント ライブラリ</a>でサポートされているため、他のツールとの統合を簡単に行うことができます。</p>
<h3>イベントとトリガー</h3>
<p>クラウド イベントとは、クラウド環境で発生する変化のことを指します。<em></em>たとえば、データベースでのデータの変更、ストレージ システムへのファイルの追加、新しい仮想マシン インスタンスの作成などがイベントとして扱われます。</p>
<p>イベントは対応の有無にかかわらず発生します。イベントへの対応（レスポンス）は、トリガーを作成して行います。<em></em>トリガーは、いわば特定のイベントや一連のイベントに関心があることの宣言です。トリガーに関数をバインドすれば、イベント情報を取得して何らかの対応をとることができます。トリガーを作成して関数に関連付ける方法について詳しくは、<a href="https://cloud.google.com/functions/docs/concepts/events-triggers">イベントとトリガー</a>をご確認ください。</p>
<h3>サーバーレス</h3>
<p>Cloud Functions では、サーバーの管理やソフトウェアの構成、フレームワークの更新、オペレーティング システムへのパッチ適用などを行う必要がありません。ソフトウェアとインフラストラクチャは完全に Google で管理されるため、コードそのものに集中することができます。また、イベントに応答してリソースが自動的にプロビジョニングされるため、特別な操作を行うことなく、1 日に何度でも関数を呼び出すことができます。</p>
<h3>ユースケース</h3>
<p>軽量の ETL のような非同期ワークロードや、アプリケーション ビルドのトリガーなどのクラウド上での自動処理に適しています。独自のサーバーも専任のデベロッパーも必要ありません。イベントに紐づけた Cloud Functions の関数をデプロイするだけで、作業が完了します。</p>
<p>Cloud Functions ではきめ細かい処理をオンデマンドで実行できるため、軽量の API や Webhook にも最適です。また、HTTP 関数のデプロイ時に HTTP のエンドポイントが自動的にプロビジョニングされるため、他のサービスのように複雑な構成を行う必要がありません。その他の Cloud Functions の使用例は、下の表でご確認いただけます。</p>
<table>
<tr>
<td colspan="1" rowspan="1">
<p>ユースケース</p>
</td>
<td colspan="1" rowspan="1">
<p>説明</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>データ処理 / ETL</p>
</td>
<td colspan="1" rowspan="1">
<p>ファイルの作成、変更、削除などの <a href="https://cloud.google.com/storage" target="blank">Cloud Storage</a> イベントをリッスンし、イベントに応答します。画像処理、動画のコード変換、データの検証や変換、インターネット上の任意のサービスの呼び出しといった操作を、Cloud Functions の関数から実行できます。</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>Webhook</p>
</td>
<td colspan="1" rowspan="1">
<p>単純な <a href="https://cloud.google.com/functions/docs/calling/http" target="blank">HTTP トリガー</a>によって、サードパーティ システム（GitHub、Slack、Stripe など）をはじめ、HTTP リクエストを送信できるあらゆる場所から送られたイベントに応答できます。</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>軽量 API</p>
</td>
<td colspan="1" rowspan="1">
<p>疎結合・軽量のロジックを組み合わせることで、アプリケーションのビルドや拡張を迅速に行えます。関数は、イベントでトリガーされるようにすることも、HTTP/S で直接呼び出すこともできます。</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>モバイル バックエンド</p>
</td>
<td colspan="1" rowspan="1">
<p>アプリのデベロッパー向けに Google が提供しているモバイル プラットフォーム、<a href="https://firebase.google.com/docs/functions/" target="blank">Firebase</a> を使用すれば、Cloud Functions でモバイル バックエンドを記述できます。これにより、Firebase Analytics、Firebase Realtime Database、Firebase Authentication、Firebase Storage からのイベントをリッスンし、イベントに応答できるようになります。</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>IoT</p>
</td>
<td colspan="1" rowspan="1">
<p>無数のデバイスのストリーミング データを Cloud Pub/Sub に取り込み、Cloud Functions を呼び出してそのデータを処理、変換、保存するといった操作を、完全にサーバーレスな形で実現できます。</p>
</td>
</tr>
</table>
<p>このハンズオンラボでは、Google Cloud Console で Cloud Functions の関数を作成してデプロイする方法を説明します。</p>
<h4>ラボの内容</h4>
<ul>
<li>
<p>Cloud Functions の関数を作成する</p>
</li>
<li>
<p>関数をデプロイしてテストする</p>
</li>
<li>
<p>ログを表示する</p>
</li>
</ul>

<h2 id="step4">関数を作成する</h2>
<p>このステップでは、Google Cloud Console を使用して Cloud Functions の関数を作成します。</p>
<ol>
<li>
<p>Google Cloud Console で、<strong>ナビゲーション メニュー</strong> &gt; [<strong>Cloud Functions</strong>] をクリックします。</p>
<p><img alt="cf985da53a07ac51.png" src="https://cdn.qwiklabs.com/KbMVUAeYk76QP%2FAX14%2BBP9brAf4kNEs9FS9KW9p9GRo%3D"></p>
</li>
<li>
<p>[<strong>関数を作成</strong>] をクリックします。</p>
<p><img alt="7af8908703476091.png" src="https://cdn.qwiklabs.com/BzIXGT5hiO7DUGvM7gji7hg3cu4dBDIF2tvgTOp1alM%3D"></p>
</li>
<li>
<p>[<strong>関数の作成</strong>] ダイアログで、次の値を入力します。</p>
</li>
</ol>
<table>
<tr>
<td colspan="1" rowspan="1">
<p><strong>フィールド</strong></p>
</td>
<td colspan="1" rowspan="1">
<p><strong>値</strong></p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>名前</p>
</td>
<td colspan="1" rowspan="1">
<p>GCFunction</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>割り当てられるメモリ</p>
</td>
<td colspan="1" rowspan="1">
<p>デフォルト</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>トリガー</p>
</td>
<td colspan="1" rowspan="1">
<p>HTTP トリガー</p>
</td>
</tr>
<tr>
<td colspan="1" rowspan="1">
<p>ソースコード</p>
</td>
<td colspan="1" rowspan="1">
<p>インライン エディタ（すでに index.js に実装されているデフォルトの <code>helloWorld</code> 関数を使用してください）。</p>
</td>
</tr>
</table>
<p><img alt="Cloud_Function_Edit_Page.png" src="https://cdn.qwiklabs.com/SNGtBSYfdapxZK2kKh4qB4tFxW3ETL1OuKeSnn%2Fi13g%3D"></p>
<p>次のセクションではこの関数をデプロイします。</p>
<h2 id="step5">関数をデプロイする</h2>
<p>引き続き [<strong>関数の作成</strong>] ダイアログで、下にある [<strong>作成</strong>] をクリックして関数をデプロイします。</p>
<p>[<strong>作成</strong>] をクリックすると、Google Cloud Console に <strong>Cloud Functions の概要</strong>ページが表示されます。</p>
<p>関数のデプロイ中は、関数の横に実行中のアイコンが表示されます。デプロイが完了すると、これが緑のチェックマークに変わります。</p>
<p><img alt="ab5cfeca3290b61f.png" src="https://cdn.qwiklabs.com/KgVOk%2BSqqP5pUjyoleVuXNv8P03B2%2BpDthn61SsDmCE%3D"></p>

<h2 id="step6">関数をテストする</h2>
<p>デプロイされた関数をテストします。</p>
<ol>
<li>
<p><strong>Cloud Functions の概要</strong>ページで、関数のメニューを表示し、[<strong>関数をテスト</strong>] をクリックします。</p>
<p><img alt="74e310ee6663bb3c.png" src="https://cdn.qwiklabs.com/n%2BFGkZis%2FEC73qY5XSUfj4YfC9FjF1RsPRRQSw0rfQU%3D"></p>
</li>
<li>
<p>[トリガーとなるイベント] フィールドの <code>{}</code> 内に次のテキストを入力して、[<strong>関数をテスト</strong>] をクリックします。</p>
<pre><code>"message":"Hello World!"&#x000A;</code></pre>
</li>
</ol>
<p>[<strong>アウトプット</strong>] フィールドに「<code>Success: Hello World!</code>」というメッセージが表示されます。</p>
<p>[<strong>ログ</strong>] フィールドにステータス コード <strong>200</strong> が表示されていれば、成功しています（ログが表示されるまでに数分かかることがあります）。</p>
<p><img alt="fbb41cf62deb1716.png" src="https://cdn.qwiklabs.com/DdJicbHKaSVzKdoNehiABtgMSeSCXkZoUBCeqybQqhk%3D"></p>
<h2 id="step7">ログを表示する</h2>
<p>Cloud Functions の概要ページからログを表示します。</p>
<ol>
<li>
<p>青い矢印をクリックして <strong>Cloud Functions の概要</strong>ページに戻ります。</p>
<p><img alt="8917a2bfa4fb9502.png" src="https://cdn.qwiklabs.com/9H5sRV3E8z2I%2FoUmhqD3ODBGMAdnPlX2E3TUzSkOnmg%3D"></p>
</li>
<li>
<p>関数のメニューを表示して、[<strong>ログを表示</strong>] をクリックします。</p>
<p><img alt="e97e6ec1fc17dfd7.png" src="https://cdn.qwiklabs.com/tbb%2B4z8Bp8nC8ETvhr3hcFpnFhW6X3ecJg2J4h%2BRJ4A%3D"></p>
<p>表示されるログ履歴の例:</p>
<p><img alt="logs_new.png" src="https://cdn.qwiklabs.com/svoybLLNsiSGLe17zvf2NDero1iVxFtKvLZQ%2FEI2efM%3D"></p>
<p>これでアプリケーションのデプロイ、検証、ログの表示が完了しました。</p>
</li>
</ol>

# Cloud Functions （コマンドライン）
<h2 id="step8">関数を作成する</h2>
<p>最初に、helloWorld という名前の簡単な関数を作成します。これは Cloud Functions のログにメッセージを書き込む関数です。この関数は Cloud Functions のイベントでトリガーされ、関数が実行されたことを通知するコールバック関数を受信します。</p>
<p>このラボでは、Cloud Pub/Sub トピックに関する Cloud Functions のイベントを扱います。Pub/Sub は、メッセージの送信者とメッセージの受信者を切り離す方式のメッセージ サービスです。Pub/Sub の詳細については、<a href="https://cloud.google.com/pubsub/architecture">Google Cloud Pub/Sub: Google 規模のメッセージ サービス</a>をご確認ください。</p>
<p>イベント パラメータとコールバック パラメータについて詳しくは、<a href="https://cloud.google.com/functions/docs/writing/background">バックグラウンド関数</a>をご確認ください。</p>
<p>Cloud Functions の関数を作成するには、次の手順を実施します。</p>
<ol>
<li>
<p>Cloud Shell コマンドラインで、関数コード用のディレクトリを作成します。</p>
<pre><code>mkdir gcf_hello_world&#x000A;</code></pre>
</li>
<li>
<p><code>gcf_hello_world</code> ディレクトリに移動します。</p>
<pre><code>cd gcf_hello_world&#x000A;</code></pre>
</li>
<li>
<p><code>index.js</code> を作成し、編集するために開きます。</p>
<pre><code>nano index.js&#x000A;</code></pre>
</li>
<li>
<p>次の内容を <code>index.js</code> ファイルにコピーします。</p>
<pre><code>/**&#x000A; * Cloud Function.&#x000A; *&#x000A; * @param {object} event The Cloud Functions event.&#x000A; * @param {function} callback The callback function.&#x000A; */&#x000A;exports.helloWorld = function helloWorld (event, callback) {&#x000A;  console.log(`My Cloud Function: ${JSON.stringify(event.data.message)}`);&#x000A;  callback();&#x000A;};&#x000A;</code></pre>
</li>
<li>
<p>nano エディタを終了し（Ctrl+X）、ファイルを保存します（Y）。</p>
</li>
</ol>
<h2 id="step9">Cloud Storage バケットを作成する</h2>
<p>次のコマンドを使用して、関数用に新しい Cloud Storage バケットを作成します。</p>
<pre><code>gsutil mb -p [PROJECT_ID] gs://[BUCKET_NAME]&#x000A;</code></pre>
<ul>
<li>
<p><strong>PROJECT-ID</strong> は、このラボの接続の詳細で確認できる GCP プロジェクト ID に置き換えてください。</p>
<p><img alt="6e0da7d1a8b3dbcb.png" src="https://cdn.qwiklabs.com/MSmsUcrgV8c3%2FThlG2E67Tdm3HoWI8hsTdDO8iQouqw%3D"></p>
</li>
<li>
<p><strong>BUCKET_NAME</strong> はバケットに付ける名前です。グローバルに一意なものにする必要があります。</p>
</li>
</ul>

<h2 id="step10">関数をデプロイする</h2>
<p>新しい関数をデプロイするには、<code>--trigger-topic</code>、<code>--trigger-bucket</code>、<code>--trigger-http</code> のいずれかを指定する必要があります。既存の関数にアップデートをデプロイすると、特に指定のない限り、その関数は既存のトリガーを維持します。</p>
<p>このラボでは、<code>--trigger-topic</code> に <code>hello_world</code> を設定します。</p>
<ol>
<li>
<p><strong>hello_world</strong> という Pub/Sub トピックに関数をデプロイします。[<code>BUCKET_NAME</code>] はバケットの名前で置き換えます。</p>
<pre><code>gcloud functions deploy helloWorld \&#x000A;  --stage-bucket [BUCKET_NAME] \&#x000A;  --trigger-topic hello_world \&#x000A;  --runtime nodejs6&#x000A;</code></pre>
</li>
<li>
<p>関数のステータスを確認します。</p>
<pre><code>gcloud functions describe helloWorld&#x000A;</code></pre>
</li>
</ol>
<p>ステータスが ACTIVE の場合は、関数がデプロイされたことを表します。</p>
<pre><code class="language-bash prettyprint">entryPoint: helloWorld&#x000A;eventTrigger:&#x000A;  eventType: providers/cloud.pubsub/eventTypes/topic.publish&#x000A;  failurePolicy: {}&#x000A;  resource:&#x000A;...&#x000A;status: ACTIVE&#x000A;...&#x000A;</code></pre>
<p>トピックにメッセージが配信されるたびに関数が実行され、メッセージの内容が入力データとして送信されます。</p>

<h2 id="step11">関数をテストする</h2>
<p>関数をデプロイしてそれがアクティブになっていることを確認したら、関数がイベントを検出した後にクラウド ログにメッセージを書き込むかどうかをテストします。</p>
<p>次のコマンドを入力して、関数のメッセージ テストを作成します。</p>
<pre><code>gcloud functions call helloWorld --data '{"message":"Hello World!"}'&#x000A;</code></pre>
<p>Cloud ツールから関数の実行 ID が返され、メッセージがログに書き込まれたことが示されます。</p>
<p>次のような内容が返されます。</p>
<pre><code class="language-bash prettyprint">executionId: 3zmhpf7l6j5b&#x000A;</code></pre>
<p>該当するログを表示して、その実行 ID のログメッセージが存在することを確認します。</p>
<h2 id="step12">ログを表示する</h2>
<p>該当のログを開き、ログ履歴の中に対象のメッセージがあることを確認します。</p>
<pre><code>gcloud functions logs read helloWorld&#x000A;</code></pre>
<p>関数が正常に実行されると、次のようなメッセージがログに表示されます。</p>
<pre><code>LEVEL  NAME        EXECUTION_ID  TIME_UTC                 LOG&#x000A;D      helloWorld  3zmhpf7l6j5b  2017-12-05 22:17:42.585  Function execution started&#x000A;I      helloWorld  3zmhpf7l6j5b  2017-12-05 22:17:42.650  My Cloud Function: Hello World!&#x000A;D      helloWorld  3zmhpf7l6j5b  2017-12-05 22:17:42.666  Function execution took 81 ms, finished with status: 'ok'&#x000A;</code></pre>
<p>これでアプリケーションのデプロイ、検証、ログの表示が完了しました。</p>

# Google Cloud Pub/Sub: Qwik Start - Console
<h2 id="step2">概要</h2>
<p>Google Cloud Pub/Sub は、アプリケーションやサービスの間でイベントデータを交換するためのメッセージング サービスです。データのプロデューサーは Pub/Sub のトピックにメッセージをパブリッシュし、コンシューマはそのトピックにサブスクリプションを作成します。サブスクライバーは、サブスクリプションからメッセージを pull するか、push サブスクリプションの webhook として設定されます。メッセージは、設定可能な特定の期間内に確認する必要があります。</p>
<h3>内容</h3>
<ul>
<li>データを保持するトピックを用意する</li>
<li>トピックにサブスクリプションを作成し、そのデータにアクセスする</li>
<li>メッセージを配信し、pull で呼び出して確認する</li>
</ul>

<h2 id="step4">Pub/Sub の設定</h2>
<p>Google Cloud Shell コンソールを使用して、Google Cloud Pub/Sub でのオペレーションを行えます。</p>
<p>Pub/Sub を使用するには、データを保持するトピックを作成し、そのトピックにパブリッシュされたデータにアクセスするためのサブスクリプションを作成します。</p>
<ol>
<li>[<strong>ナビゲーション メニュー</strong>] &gt; [<strong>Pub/Sub</strong>] &gt; [<strong>トピック</strong>] をクリックします。</li>
</ol>
<p><img alt="5366c9bf03c0680d.png" src="https://cdn.qwiklabs.com/xZ3SEfVz2XSS%2F%2F1znllxjwCt6Sw91sr8KuWwOH%2FDBoo%3D"></p>
<ol start="2">
<li>[<strong>トピックを作成</strong>] をクリックします。</li>
</ol>
<p><img alt="541bcc9fa0935574.png" src="https://cdn.qwiklabs.com/%2B5DjzluijKP1xciKT%2BmsugTk9wWkuM9Tf87XVq4Cc9s%3D"></p>
<ol start="3">
<li>トピックには一意の名前を付ける必要があります。このラボでは、トピックに MyTopic という名前を付けます。[トピックを作成] ダイアログで </li>
</ol>
<ul>
<li>[名前] に「MyTopic」と入力します。</li>
<li>暗号化をデフォルト値のままにします。</li>
<li>[トピックを作成]をクリックします。</li>
</ul>

<p><img alt="a19c7c36f4a6a840.png" src="https://cdn.qwiklabs.com/mtnbgHKCE%2Fvitvqz%2BWAMjiYuTlCBflcqWHzQNf%2F3pIQ%3D"></p>
<p>これで、トピックが作成されました。</p>
<p><img alt="348c3f65791bb33a.png" src="https://cdn.qwiklabs.com/WA4Oair6SIFOdZM7XPj4isuHEh9%2Fi%2FFwjfbe0q2uwu4%3D"></p>

<h2 id="step5">サブスクリプションの追加</h2>
<p>トピックにアクセスするためのサブスクリプションを作成します。</p>
<ol>
<li>左パネルのトピックをクリックしてトピックダイアログに戻ります。先ほど作成したトピックに対して、ケバブアイコン　&gt; [サブスクリプションを作成] をクリックします。</li>
</ol>
<p><img alt="1539b00b3d29d94b.png" src="https://cdn.qwiklabs.com/pmK%2F6B9Zi6jTDefr3uTGsKjJqNgjFWtKwLnqu7bL%2Bxk%3D"></p>
<ol start="2">
<li>ダイアログで[サブスクリプションをトピックに追加]</li>
</ol>
<ul>
<li>サブスクリプションの名前を入力し（MySub など）</li>
<li>[配信タイプ] を [pull] に設定します。</li>
<li>他のすべてのオプションはデフォルト値のままにします。</li>
<li>[作成] をクリックします。</li>
</ul>

<p><img alt="afbd464b52e67bf3.png" src="https://cdn.qwiklabs.com/aCXRjDeRJp8XR767Psjd3267n2Cm7%2BmmOlBAdR33GdE%3D"></p>
<p>サブスクリプションがサブスクリプションリストに表示されます。</p>
<p><img alt="create-subscription.png" src="https://cdn.qwiklabs.com/VFUtLeblMjxJR%2F1fGoaYqwrliSE%2FhvERHYs6viYefLc%3D"></p>

<h2 id="step6">理解度を確認する</h2>
<p>今回のラボで学習した内容の理解を深めるため、以下の選択問題を用意しました。正解を目指して頑張ってください。</p>
<p><ql-multiple-choice-probe answerindex="0" optiontitles='[
"トピック, サブスクリプション",
"サブスクリプション, トピック",
"トピック, トピック",
"サブスクリプション, サブスクリプション"
]' shuffle="" stem="パブリッシャー アプリケーションは、メッセージを作成して____に送信します。サブスクライバー アプリケーションは、トピックからメッセージを受信するために____をトピックに作成します。">
</ql-multiple-choice-probe></p>
<p><ql-true-false-probe answer="true" stem="Cloud Pub/Subは、信頼性と拡張性が高くなるように設計された非同期メッセージング サービスである。">
</ql-true-false-probe></p>
<h2 id="step7">トピックへのメッセージのパブリッシュ</h2>
<ol>
<li> [トピック] ダイアログで、[メッセージをパブリッシュ] をクリックします。[メッセージをパブリッシュ] オプションを表示するには、ブラウザウィンドウを広げる必要があります。</li>
</ol>
<p><img alt="86c12b4d8d6c6a73.png" src="https://cdn.qwiklabs.com/MUkPo4R0LO%2Fye6ye3vuRKR7bZw8j%2Fr3d7k84GN6mSRQ%3D"></p>
<ol start="2">
<li>[<strong>メッセージ</strong>] フィールドに「<code>Hello World</code>」と入力し、[<strong>公開</strong>] をクリックします。</li>
</ol>
<p><img alt="10a2fe7542f3eb21.png" src="https://cdn.qwiklabs.com/N85EFAtx4x1Y1WY7ZVUVA9wsQz5V8%2FZPbkvIFc6fQHI%3D"></p>
<h2 id="step8">メッセージの表示</h2>
<p>メッセージを表示するには、サブスクリプション（<code>MySub</code>）を使用し、トピック（<code>MyTopic</code>）からメッセージ（<code>Hello World</code>）を pull します。</p>
<p>コマンドラインに次のコマンドを入力します。</p>
<pre><code>gcloud pubsub subscriptions pull --auto-ack MySub&#x000A;</code></pre>
<p>コマンド出力の DATA フィールドにメッセージが表示されます。</p>
<p><img alt="6f1565505c570a0d.png" src="https://cdn.qwiklabs.com/Dn4h%2BsG0zaCpkL5QExZmYOdPIe%2BgDNpjmFrmWJm2IJs%3D"></p>
<p>Pub/Sub トピックを作成してトピックにパブリッシュし、サブスクリプションを作成しました。次に、サブスクリプションを使用して、トピックからデータを pull しました。</p>
<h2 id="step9">これで完了です。</h2>

# Google Cloud Pub/Sub: Qwik Start - コマンドライン

<h2 id="step2">概要</h2>
<p>Google Pub/Sub は、アプリケーションやサービスの間でイベントデータを交換するためのメッセージング サービスです。送信者と受信者を分離することで、独立して作成されたアプリケーション間で安全かつ高可用性が確保された通信を行えます。Google Cloud Pub/Sub によるメッセージングは低レイテンシで耐久性に優れており、デベロッパーは一般的に、非同期ワークフローの実装、イベント通知の配信、さまざまなプロセスやデバイスからのデータ ストリーミングに使用しています。</p>
<h3>演習内容</h3>
<p>このラボの内容は次のとおりです。</p>
<ul>
<li>
<p>Pub/Sub の基礎を学びます。</p>
</li>
<li>
<p>Pub/Sub トピックの作成、削除、一覧表示を行います。</p>
</li>
<li>
<p>Pub/Sub サブスクリプションの作成、削除、一覧表示を行います。</p>
</li>
<li>
<p>トピックにメッセージをパブリッシュします。</p>
</li>
<li>
<p>pull サブスクライバーを使用して、トピック メッセージを個別に出力します。</p>
</li>
<li>
<p>pull サブスクライバーをフラグとともに使用して、複数のメッセージを出力します。</p>
</li>
</ul>
<h3>要件</h3>
<p>これは<strong>入門</strong>レベルのラボです。Pub/Sub の経験がほとんどないか、まったくないことを前提としており、GCP サービスの設定と使用の基礎を学びます。</p>
<p>このラボを始める前に、ご自身の Pub/Sub の習熟度を検討してください。以下に示すラボは、このラボよりも難易度が高く、Pub/Sub の知識を異なるクラウド サービスやユースケースに応用できます。</p>
<ul>
<li><a href="https://google.qwiklabs.com/catalog_lab/934">データフロー: Qwik Start - テンプレート</a></li>
<li><a href="https://google.qwiklabs.com/catalog_lab/694">Google Cloud Platform を使用した IoT アナリティクス パイプラインの構築</a></li>
<li>
<a href="https://google.qwiklabs.com/catalog_lab/1109">Cloud Video Intelligence と Cloud Vision API を使用したユーザー生成コンテンツのスキャン</a>
Pub/Sub を Google Cloud Console で使用する場合は、次のラボを確認してください。</li>
<li><a href="https://google.qwiklabs.com/catalog_lab/912">Google Cloud Pub/Sub: Qwik Start - Console</a></li>
</ul>
<p>準備ができたら下にスクロールし、手順に沿ってラボ環境をセットアップします。</p>
<h2 id="step3">設定と要件</h2>
<h3>Qwiklabs の設定</h3>
<h4>[ラボを開始] ボタンをクリックする前に</h4>
<p>こちらの手順をお読みください。ラボの時間は制限されており、一時停止することはできません。[ラボを開始] ボタンをクリックするとタイマーが開始され、Cloud リソースを利用できる時間が表示されます。</p>
<p>この Qwiklabs ハンズオンラボでは、シミュレーションやデモ環境ではなく、実際のクラウド環境を使ってご自身でラボのアクティビティを行うことができます。一時的な認証情報が新しく提供されるため、ラボ受講中の Google Cloud Platform へのログインおよびアクセスにはその認証情報を使用してください。</p>
<h4>前提条件</h4>
<p>このラボを完了するには、次のものが必要です。</p>
<ul>
<li>標準的なインターネット ブラウザ（Chrome を推奨）。</li>
<li>ラボを完了するために必要な時間。</li>
</ul>
<p><strong><em>注:</em></strong> すでに個人の GCP アカウントやプロジェクトをお持ちの場合でも、そのアカウントやプロジェクトはラボでは使用しないでください。</p>

<p>これでラボが開始されました。Google Cloud Shell コンソールにログインし、コマンドライン ツールを起動します。</p>
<h4>ラボを開始して Console にログインする方法</h4>
<ol>
<li>
<p>[<strong>ラボを開始</strong>] ボタンをクリックします。ラボの料金をお支払いいただく必要がある場合は、表示されるポップアップでお支払い方法を選択してください。
左側のパネルには、このラボで使用する必要がある一時的な認証情報が表示されます。</p>
<p><img alt="Google Console を開く" src="https://cdn.qwiklabs.com/%2FtHp4GI5VSDyTtdqi3qDFtevuY014F88%2BFow%2FadnRgE%3D"></p>
</li>
<li>
<p>ユーザー名をコピーし、[<strong>Google Console を開く</strong>] をクリックします。
ラボでリソースが起動し、別のタブで [<strong>アカウントの選択</strong>] ページが表示されます。</p>
<p><strong>ヒント:</strong> タブをそれぞれ別のウィンドウで開き、並べて表示しておきましょう。<em></em></p>
</li>
<li>
<p>[アカウントの選択] ページで [<strong>別のアカウントを使用</strong>] をクリックします。</p>
<p><img alt="アカウントを選択" src="https://cdn.qwiklabs.com/eQ6xPnPn13GjiJP3RWlHWwiMjhooHxTNvzfg1AL2WPw%3D"></p>
</li>
<li>
<p>[ログイン] ページが開きます。[接続の詳細] パネルでコピーしたユーザー名を貼り付けます。パスワードもコピーして貼り付けます。</p>
<p><strong>重要:</strong> 認証情報は [接続の詳細] パネルに表示されたものを使用してください。<em></em>ご自身の Qwiklabs 認証情報は使用しないでください。請求が発生する事態を避けるため、GCP アカウントをお持ちの場合でもそのアカウントはラボで使用しないでください。</p>
</li>
<li>
<p>以降のページでは次の点にご注意ください。</p>
<ul>
<li>利用規約に同意してください。</li>
<li>復元オプションや 2 要素認証プロセスは設定しないでください（一時的なアカウントであるため）。</li>
<li>無料トライアルには登録しないでください。</li>
</ul>
</li>
</ol>
<p>しばらくすると、このタブで GCP Console が開きます。</p>
<aside>
<b>注:</b> 左上の [Google Cloud Platform] の横にある<b>ナビゲーション メニュー</b>をクリックすると、GCP のプロダクトやサービスのリストが含まれるメニューが表示されます。
<img alt="Cloud Console メニュー" src="https://cdn.qwiklabs.com/9vT7xPlxoNP%2FPsK0J8j0ZPFB4HnnpaIJVCDByaBrSHg%3D">
</aside>

<h3>Google Cloud Shell</h3>
<h3>Google Cloud Shell の有効化</h3>
<p>Google Cloud Shell は、デベロッパー ツールと一緒に読み込まれる仮想マシンです。5 GB の永続ホーム ディレクトリが用意されており、Google Cloud で稼働します。Google Cloud Shell では、コマンドラインで GCP リソースにアクセスできます。</p>
<ol>
<li>
<p>GCP Console の右上のツールバーにある [Cloud Shell をアクティブにする] ボタンをクリックします。</p>
<p><img alt="Cloud Shell アイコン" src="https://cdn.qwiklabs.com/vdY5e%2Fan9ZGXw5a%2FZMb1agpXhRGozsOadHURcR8thAQ%3D"></p>
</li>
<li>
<p>[<strong>続行</strong>] をクリックします。</p>
<p><img alt="cloudshell_continue" src="https://cdn.qwiklabs.com/lr3PBRjWIrJ%2BMQnE8kCkOnRQQVgJnWSg4UWk16f0s%2FA%3D"></p>
</li>
</ol>
<p>環境のプロビジョニングと接続には少し時間がかかります。接続すると、すでに認証されており、プロジェクトは<em> PROJECT_ID </em>に設定されています。例えば:</p>
<p><img alt="Cloud Shell 端末" src="https://cdn.qwiklabs.com/hmMK0W41Txk%2B20bQyuDP9g60vCdBajIS%2B52iI2f4bYk%3D"></p>
<p><strong>gcloud</strong> は Google Cloud Platform のコマンドライン ツールです。このツールは、Cloud Shell にプリインストールされており、タブ補完がサポートされています。</p>
<p>次のコマンドを使用すると、有効なアカウント名を一覧表示できます。</p>
<pre><code>gcloud auth list&#x000A;</code></pre>
<p>出力:</p>
<pre><code class="language-output prettyprint">Credentialed accounts:&#x000A;- &lt;myaccount&gt;@&lt;mydomain&gt;.com (active)&#x000A;	</code></pre>
<p>出力例:</p>
<pre><code class="language-Output prettyprint">Credentialed accounts:&#x000A;- google1623327_student@qwiklabs.net&#x000A;	</code></pre>
<p>次のコマンドを使用すると、プロジェクト ID を一覧表示できます。</p>
<pre><code>gcloud config list project&#x000A;	</code></pre>
<p>出力:</p>
<pre><code class="language-output prettyprint">[core]&#x000A;project = &lt;project_ID&gt;&#x000A;	</code></pre>
<p>出力例:</p>
<pre><code class="language-Output prettyprint">[core]&#x000A;project = qwiklabs-gcp-44776a13dea667a6&#x000A;	</code></pre>
<aside>
<strong>gcloud</strong> について詳しくは、<a href="https://cloud.google.com/sdk/gcloud">gcloud の概要</a>をご確認ください。

	</aside>

<h2 id="step4">Pub/Sub—基本情報</h2>
<p>前述のように、Google Cloud Pub/Sub は非同期グローバル メッセージング サービスです。Pub/Sub では、<code>topics</code>、<code>publishing</code>、<code>subscribing</code> という 3 つのキーワードが頻出します。</p>
<p><code>topic</code> は、複数のアプリケーションが共通のスレッドを通じて相互に接続できる共有文字列です。</p>
<p>パブリッシャーは、メッセージを Cloud Pub/Sub トピックに push（<code>publish</code>）します。サブスクライバーは、そのスレッドへの「<code>subscription</code>」を作成し、トピックからメッセージを pull するか、push サブスクリプション用の webhook を構成します。すべてのサブスクライバーは、構成可能な時間の範囲内で各メッセージに確認応答を返信する必要があります。</p>
<p>まとめると、パブリッシャーはメッセージを作成してトピックに送信し、サブスクライバーはトピックへのサブスクリプションを作成してトピックからメッセージを受け取ります。</p>
<h2 id="step5">Pub/Sub トピック</h2>
<p>Pub/Sub は Google Cloud Shell に事前にインストールされているので、このサービスを利用するために必要なインストールおよび構成はありません。</p>
<p>最初にトピックを作成します。次のコマンドを実行して <code>myTopic</code> というトピックを作成します。</p>
<pre><code>gcloud pubsub topics create myTopic&#x000A;</code></pre>

<p>さらに 2 つのトピックを作成します。1 つは <code>Test1</code>、もう 1 つは <code>Test2</code> とします。</p>
<pre><code>gcloud pubsub topics create Test1&#x000A;</code></pre>
<pre><code>gcloud pubsub topics create Test2&#x000A;</code></pre>
<p>作成した 3 つのトピックを確認します。次のコマンドを実行します。</p>
<pre><code>gcloud pubsub topics list&#x000A;</code></pre>
<p>出力は次のようになります。</p>
<pre><code>name: projects/qwiklabs-gcp-3450558d2b043890/topics/myTopic&#x000A;---&#x000A;name: projects/qwiklabs-gcp-3450558d2b043890/topics/Test2&#x000A;---&#x000A;name: projects/qwiklabs-gcp-3450558d2b043890/topics/Test1&#x000A;</code></pre>
<p>クリーンアップを行います。次のコマンドを実行して、<code>Test1</code> と <code>Test2</code> を削除します。</p>
<pre><code>gcloud pubsub topics delete Test1&#x000A;</code></pre>
<pre><code>gcloud pubsub topics delete Test2&#x000A;</code></pre>
<p>もう一度 <code>gcloud pubsub topics list</code> コマンドを実行します。出力は次のようになります。</p>
<pre><code>---&#x000A;name: projects/qwiklabs-gcp-3450558d2b043890/topics/myTopic&#x000A;</code></pre>
<h2 id="step6">Pub/Sub サブスクリプション</h2>
<p>トピックの作成、表示、削除の方法がわかったところで、次にサブスクリプションの操作に進みます。</p>
<p>次のコマンドを実行して、トピック <code>myTopic</code> へのサブスクリプション <code>mySubscription</code> を作成します。</p>
<pre><code>gcloud  pubsub subscriptions create --topic myTopic mySubscription&#x000A;</code></pre>

<p><code>myTopic</code> に 2 つのサブスクリプションを追加します。次のコマンドを実行して、<code>Test1</code> と <code>Test2</code> のサブスクリプションを作成します。</p>
<pre><code>gcloud  pubsub subscriptions create --topic myTopic Test1&#x000A;</code></pre>
<pre><code>gcloud  pubsub subscriptions create --topic myTopic Test2&#x000A;</code></pre>
<p>サブスクリプションが作成されたことを確認します。次のコマンドを実行して、myTopic へのサブスクリプションのリストを表示します。</p>
<pre><code>gcloud pubsub topics list-subscriptions myTopic&#x000A;</code></pre>
<p>出力は次のようになります。</p>
<pre><code>---&#x000A;  projects/qwiklabs-gcp-3450558d2b043890/subscriptions/Test2&#x000A;---&#x000A;  projects/qwiklabs-gcp-3450558d2b043890/subscriptions/Test1&#x000A;---&#x000A;  projects/qwiklabs-gcp-3450558d2b043890/subscriptions/mySubscription&#x000A;</code></pre>
<h2 id="step7">理解度を確認する</h2>
<p>今回のラボで学習した内容の理解を深めるため、以下の選択問題を用意しました。正解を目指して頑張ってください。</p>
<p><ql-true-false-probe answer="true" stem="トピックにパブリッシュされたメッセージを受信するには、そのトピックへのサブスクリプションを作成しなければならない。">
</ql-true-false-probe></p>
<p>ここで、<code>Test1</code> と <code>Test2</code> のサブスクリプションを削除します。次のコマンドを実行します。</p>
<pre><code>gcloud pubsub subscriptions delete Test1&#x000A;</code></pre>
<pre><code>gcloud pubsub subscriptions delete Test2&#x000A;</code></pre>
<p><code>Test1</code> と <code>Test2</code> の各サブスクリプションが削除されたことを確認します。もう一度 <code>gcloud pubsub topics list-subscriptions myTopic</code> コマンドを実行します。出力は次のようになります。</p>
<pre><code>---&#x000A;  projects/qwiklabs-gcp-3450558d2b043890/subscriptions/mySubscription&#x000A;</code></pre>
<h2 id="step8">Pub/Sub のパブリッシュと 1 つのメッセージの pull</h2>
<p>サブスクリプションの作成、表示、削除の方法がわかったところで、次に Pub/Sub トピックにメッセージをパブリッシュする方法を学びます。</p>
<p>次のコマンドを実行して、作成したトピック（<code>myTopic</code>）にメッセージ <code>"hello"</code> をパブリッシュします。</p>
<pre><code>gcloud pubsub topics publish myTopic --message "Hello"&#x000A;</code></pre>
<p>さらにいくつかメッセージを <code>myTopic</code> にパブリッシュするため、次のコマンドを実行します（<code>&lt;YOUR NAME&gt;</code> にはご自分の名前、<code>&lt;FOOD&gt;</code> には好きな食べ物を指定します）。</p>
<pre><code>gcloud pubsub topics publish myTopic --message "Publisher's name is &lt;YOUR NAME&gt;"&#x000A;</code></pre>
<pre><code>gcloud pubsub topics publish myTopic --message "Publisher likes to eat &lt;FOOD&gt;"&#x000A;</code></pre>
<pre><code>gcloud pubsub topics publish myTopic --message "Publisher thinks Pub/Sub is awesome"&#x000A;</code></pre>
<p>ここで、<code>pull</code> コマンドを使用して、トピックからメッセージを取得します。pull コマンドはサブスクリプション ベースです。つまり、以前にトピック <code>myTopic</code> へのサブスクリプション <code>mySubscription</code> を設定したため、コマンドは正常に機能するはずです。</p>
<p>次のコマンドを使用して、パブリッシュしたメッセージを Pub/Sub トピックから pull します。</p>
<pre><code>gcloud pubsub subscriptions pull mySubscription --auto-ack&#x000A;&#x000A;</code></pre>
<p>出力は次のようになります。</p>
<p><img alt="food.png" src="https://cdn.qwiklabs.com/59Crmi1YmFDkTUDc1DPQCFiS4inFJjWs8iYQj8xBREM%3D"></p>
<p>何が起きているのでしょうか。このトピックには 4 つのメッセージをパブリッシュしましたが、出力されたのは 1 つだけです。</p>
<p>ここで、デベロッパーがつまづくことが多い <code>pull</code> コマンドの機能の注意点を挙げます。</p>
<ul>
<li><strong>フラグを指定せずに pull コマンドを使用すると、メッセージが 1 つだけ出力されます。これは、サブスクライブしているトピックに複数のメッセージがある場合も同じです。</strong></li>
<li><strong>特定のサブスクリプション ベースの pull コマンドから 1 つのメッセージが出力された後は、pull コマンドを使って同じメッセージに再びアクセスすることはできません。</strong></li>
</ul>
<p>上記の 2 点目を確認するために、最後のコマンドをさらに 3 回実行してみましょう。パブリッシュしたメッセージが順番に出力されることがわかります。さらに 4 回目の実行では、出力は次のように、返すメッセージがないことを示します。</p>
<pre><code>gcpstaging20394_student@cloudshell:~ (qwiklabs-gcp-3450558d2b043890)$ gcloud pubsub subscriptions pull mySubscription --auto-ack&#x000A;Listed 0 items.&#x000A;</code></pre>
<p>次の最後のセクションでは、<code>flag</code> を指定して、トピックから複数のメッセージを pull する方法を学びます。</p>
<h2 id="step9">Pub/Sub でサブスクリプションからすべてのメッセージを pull する</h2>
<p>最後の例で、トピックからすべてのメッセージを pull したため、<code>myTopic</code> にさらにいくつかメッセージを追加します。次のコマンドを実行します。</p>
<pre><code>gcloud pubsub topics publish myTopic --message "Publisher is starting to get the hang of Pub/Sub"&#x000A;</code></pre>
<pre><code>gcloud pubsub topics publish myTopic --message "Publisher wonders if all messages will be pulled"&#x000A;</code></pre>
<pre><code>gcloud pubsub topics publish myTopic --message "Publisher will have to test to find out"&#x000A;</code></pre>
<p>これでトピックにメッセージが追加されました。コマンドに <code>flag</code> を追加し、3 つのメッセージを一度に出力できるようにします。気づかなかったかもしれませんが、実はこれまでずっとフラグを使っていました。<code>pull</code> コマンドの <code>--auto-ack</code> の部分がフラグで、枠に合わせたメッセージの書式設定をこのフラグで行っていました。</p>
<p>さらに <code>limit</code> も、pull するメッセージ数に上限を設定するフラグです。次のように、<code>limit</code> フラグを指定して pull コマンドを実行します。</p>
<pre><code>gcloud pubsub subscriptions pull mySubscription --auto-ack --limit=3&#x000A;</code></pre>
<p>出力は次のようになります。</p>
<p><img alt="cli_PS.png" src="https://cdn.qwiklabs.com/RqPezOvS%2F0IYEIkWliflxdg5XtRApdqMRlcPhMXel%2Bk%3D"></p>
<p>これで、Pub/Sub コマンドにフラグを追加して、より多くのメッセージを出力する方法を学びました。Pub/Sub マスターへの道をまた一歩、前進しましたね。</p>
<h2 id="step10">これで完了です。</h2>

# Google Cloud Pub/Sub: Qwik Start - Python
<h2 id="step2">概要</h2>
<p>Google Cloud Pub/Sub サービスを使用すると、メッセージをアプリケーション間ですばやく確実に非同期で交換できます。そうするには、データのプロデューサーが Cloud Pub/Sub トピックにメッセージをパブリッシュし、その後、サブスクライバー クライアントがそのトピックへのサブスクリプションを作成し、サブスクリプションからのメッセージを使用します。Cloud Pub/Sub は、確実に配信できなかったメッセージを最大 7 日間保持します。</p>
<p>このラボでは、Cloud Pub/Sub で Python クライアント ライブラリを使ってメッセージをパブリッシュする方法を学びます。</p>
<h3>演習内容</h3>
<p>このラボでは以下を行います。</p>
<ul>
<li>
<p>Pub/Sub の基礎を学びます。</p>
</li>
<li>
<p>Pub/Sub トピックの作成と一覧表示を行います。</p>
</li>
<li>
<p>Pub/Sub サブスクリプションの作成と一覧表示を行います。</p>
</li>
<li>
<p>トピックにメッセージをパブリッシュします。</p>
</li>
<li>
<p>pull サブスクライバーを使用して、トピック メッセージを個別に出力します。</p>
</li>
</ul>

<h2 id="step4">GCP へのクライアント ライブラリのインストール</h2>
<p>クライアント ライブラリをインストールします。</p>
<pre><code>sudo pip install --upgrade google-cloud-pubsub&#x000A;</code></pre>
<p>GitHub リポジトリをクローンしてサンプルコードを入手します。</p>
<pre><code>git clone https://github.com/GoogleCloudPlatform/python-docs-samples.git&#x000A;</code></pre>
<p>ディレクトリに移動します。</p>
<pre><code>cd python-docs-samples/pubsub/cloud-client/&#x000A;</code></pre>
<h2 id="step5">Pub/Sub - 基本情報</h2>
<p>Google Cloud Pub/Sub は、非同期のグローバル メッセージング サービスです。Pub/Sub では、topics<em></em>、publishing<em></em>、subscribing<em></em> という 3 つのキーワードが頻出します。</p>
<p>topic は、複数のアプリケーションが共通のスレッドを通じて相互に接続できる共有文字列です。</p>
<p>パブリッシャーは、メッセージを Cloud Pub/Sub トピックに push（publish）します。サブスクライバーは、そのスレッドへの「subscription」を作成し、トピックからメッセージを pull するか、push サブスクリプション用の webhook を構成します。すべてのサブスクライバーは、構成可能な時間の範囲内で各メッセージに確認応答を返信する必要があります。</p>
<p>まとめると、パブリッシャーはメッセージを作成してトピックに送信し、サブスクライバーはトピックへのサブスクリプションを作成してトピックからメッセージを受け取ります。</p>
<h3>GCP 内の Pub/Sub</h3>
<p>Pub/Sub は Google Cloud Shell に事前にインストールされているので、このサービスを利用するために必要なインストールや構成はありません。このラボでは、Python を使用してトピックとサブスクライバーを作成してから、メッセージを表示します。メッセージをトピックにパブリッシュするには、gcloud コマンドを使用します。</p>
<h2 id="step6">トピックの作成</h2>
<p>Cloud Pub/Sub にデータをパブリッシュするには、トピックを作成してから、トピックへのパブリッシャーを構成します。</p>
<p>環境変数 <code>GLOBAL_CLOUD_PROJECT</code> を設定します。GCP のプロジェクト ID は接続の詳細で確認できます。</p>
<pre><code>export GLOBAL_CLOUD_PROJECT=GCP Project ID&#x000A;</code></pre>
<p><code>publisher.py</code> は、Cloud Pub/Sub API を使ってトピックに対して基本的な操作を実行する方法を示すスクリプトです。次のコマンドで publisher スクリプトの内容を確認します。</p>
<pre><code>cat publisher.py&#x000A;</code></pre>
<aside class="special"><p>nano や vim など、Cloud Shell にインストールされているシェルエディタを使用するか、Cloud Shell コードエディタを使用して <code>python-docs-samples/pubsub/cloud-client/publisher.py</code> の内容を確認することもできます。</p>
</aside>
<p>publisher スクリプトについて詳しくは、次のコマンドを実行してください。</p>
<pre><code>python publisher.py -h&#x000A;</code></pre>
<p><em>出力例</em></p>
<pre><code>usage: publisher.py [-h]&#x000A;                    project&#x000A;                    {list,create,delete,publish,publish-with-custom-attributes,publish-with-futures,publish-with-error-handler,publish-with-batch-settings}&#x000A;                    ...&#x000A;&#x000A;This application demonstrates how to perform basic operations on topics&#x000A;with the Cloud Pub/Sub API.&#x000A;&#x000A;For more information, see the README.md under /pubsub and the documentation&#x000A;at https://cloud.google.com/pubsub/docs.&#x000A;&#x000A;positional arguments:&#x000A;  project               Your Google Cloud project ID&#x000A;  {list,create,delete,publish,publish-with-custom-attributes,publish-with-futures,publish-with-error-handler,publish-with-batch-settings}&#x000A;    list                Lists all Pub/Sub topics in the given project.&#x000A;    create              Create a new Pub/Sub topic.&#x000A;    delete              Deletes an existing Pub/Sub topic.&#x000A;    publish             Publishes multiple messages to a Pub/Sub topic.&#x000A;    publish-with-custom-attributes&#x000A;                        Publishes multiple messages with custom attributes to&#x000A;                        a Pub/Sub topic.&#x000A;    publish-with-futures&#x000A;                        Publishes multiple messages to a Pub/Sub topic and&#x000A;                        prints their message IDs.&#x000A;    publish-with-error-handler&#x000A;                        Publishes multiple messages to a Pub/Sub topic with an&#x000A;                        error handler.&#x000A;    publish-with-batch-settings&#x000A;                        Publishes multiple messages to a Pub/Sub topic with&#x000A;                        batch settings.&#x000A;&#x000A;optional arguments:&#x000A;  -h, --help            show this help message and exit&#x000A;</code></pre>
<p>publisher スクリプトを実行して Pub/Sub トピックを作成します。</p>
<pre><code>python publisher.py $GLOBAL_CLOUD_PROJECT create MyTopic&#x000A;</code></pre>
<p><em>出力例</em></p>
<pre><code>Topic created: name: "projects/qwiklabs-gcp-fe27729bc161fb22/topics/MyTopic"&#x000A;</code></pre>

<p>このコマンドは、当該プロジェクトのすべての Pub/Sub トピックのリストを返します。</p>
<pre><code>python publisher.py $GLOBAL_CLOUD_PROJECT list&#x000A;</code></pre>
<p><em>出力例</em></p>
<pre><code>name: "projects/qwiklabs-gcp-fe27729bc161fb22/topics/MyTopic"&#x000A;</code></pre>
<p>作成したトピックは Google Cloud Console で確認することもできます。</p>
<p><strong>ナビゲーション メニュー</strong> &gt; [<strong>Pub/Sub</strong>] &gt; [<strong>トピック</strong>] に移動します。</p>
<p><img alt="29f5381c6ffbb8ca.png" src="https://cdn.qwiklabs.com/%2BOe9PeVxmoke4%2F38la7GFatJpnmtCWu9jQw1kiJUuJA%3D"></p>
<p><code>MyTopic</code> が表示されます。</p>
<p><img alt="badf10f6700c3fc8.png" src="https://cdn.qwiklabs.com/9B%2BZPgLBxWXokZDwYuPnAt%2FowY36uKcQ1BNvyvAkxxk%3D"></p>
<h2 id="step7">サブスクリプションの作成</h2>
<p><code>subscriber.py</code> スクリプトを使用して、トピックの Pub/Sub サブスクリプションを作成します。</p>
<pre><code>python subscriber.py $DEVSHELL_PROJECT_ID create MyTopic MySub&#x000A;</code></pre>

<p>このコマンドは、当該プロジェクトのサブスクライバーのリストを返します。</p>
<pre><code>python subscriber.py $GLOBAL_CLOUD_PROJECT list-in-project&#x000A;</code></pre>
<p>作成したサブスクリプションは 1 つだけなので、表示されるサブスクリプションは 1 つだけです。</p>
<p><em>出力例</em></p>
<pre><code class="language-bash prettyprint">projects/qwiklabs-gcp-7877af129f04d8b3/subscriptions/MySub&#x000A;</code></pre>
<p>作成したサブスクリプションをコンソールで確認します。左側のペインで [<strong>サブスクリプション</strong>] をクリックします。サブスクリプション名とその他の詳細情報が表示されます。</p>
<p><img alt="2866f66f3030244d.png" src="https://cdn.qwiklabs.com/3t4Iwn7UhRKwxBpY65x7k46Wo2WLbBWPc6jMEC3pr0Y%3D"></p>
<p><code>subscriber</code> スクリプトについて詳しくは、次のコマンドを実行します。</p>
<pre><code>python subscriber.py -h&#x000A;</code></pre>
<p><em>出力</em></p>
<pre><code>usage: subscriber.py [-h]&#x000A;                     project&#x000A;                     {list_in_topic,list_in_project,create,create-push,delete,update,receive,receive-custom-attributes,receive-flow-control,receive-synchronously,listen_for_errors}&#x000A;                     ...&#x000A;&#x000A;This application demonstrates how to perform basic operations on&#x000A;subscriptions with the Cloud Pub/Sub API.&#x000A;&#x000A;For more information, see the README.md under /pubsub and the documentation&#x000A;at https://cloud.google.com/pubsub/docs.&#x000A;&#x000A;positional arguments:&#x000A;  project               Your Google Cloud project ID&#x000A;  {list_in_topic,list_in_project,create,create-push,delete,update,receive,receive-custom-attributes,receive-flow-control,receive-synchronously,listen_for_errors}&#x000A;    list_in_topic       Lists all subscriptions for a given topic.&#x000A;    list_in_project     Lists all subscriptions in the current project.&#x000A;    create              Create a new pull subscription on the given topic.&#x000A;    create-push         Create a new push subscription on the given topic.&#x000A;    delete              Deletes an existing Pub/Sub topic.&#x000A;    update              Updates an existing Pub/Sub subscription's push&#x000A;                        endpoint URL. Note that certain properties of a&#x000A;                        subscription, such as its topic, are not modifiable.&#x000A;    receive             Receives messages from a pull subscription.&#x000A;    receive-custom-attributes&#x000A;                        Receives messages from a pull subscription.&#x000A;    receive-flow-control&#x000A;                        Receives messages from a pull subscription with flow&#x000A;                        control.&#x000A;    receive-synchronously&#x000A;                        Pulling messages synchronously.&#x000A;    listen_for_errors   Receives messages and catches errors from a pull&#x000A;                        subscription.&#x000A;&#x000A;optional arguments:&#x000A;  -h, --help            show this help message and exit&#x000A;</code></pre>
<h2 id="step8">メッセージをパブリッシュする</h2>
<p><code>MyTopic</code>（トピック）、<code>MyTopic</code> へのサブスクリプション（<code>MySub</code>）の設定が完了したので、gcloud コマンドを使って <code>MyTopic</code> にメッセージをパブリッシュしてみます。</p>
<p><code>myTopic</code> にメッセージ "Hello" をパブリッシュします。</p>
<pre><code>gcloud pubsub topics publish MyTopic --message "Hello"&#x000A;</code></pre>
<p>さらにいくつかメッセージを <code>myTopic</code> にパブリッシュするため、次のコマンドを実行します（&lt;YOUR NAME&gt; にはご自分の名前、&lt;FOOD&gt; には好きな食べ物を指定します）。</p>
<pre><code>gcloud pubsub topics publish MyTopic --message "Publisher's name is &lt;YOUR NAME&gt;"&#x000A;</code></pre>
<pre><code>gcloud pubsub topics publish MyTopic --message "Publisher likes to eat &lt;FOOD&gt;"&#x000A;</code></pre>
<pre><code>gcloud pubsub topics publish MyTopic --message "Publisher thinks Pub/Sub is awesome"&#x000A;</code></pre>
<h2 id="step9">メッセージの表示</h2>
<p>MyTopic にメッセージをパブリッシュできたので、MySub を使ってメッセージを pull して表示します。</p>
<p>MySub を使って MyTopic からメッセージを pull します。</p>
<pre><code>python subscriber.py $GLOBAL_CLOUD_PROJECT receive MySub&#x000A;</code></pre>
<p><em>出力例</em></p>
<pre><code class="language-bash prettyprint">Listening for messages on projects/qwiklabs-gcp-7877af129f04d8b3/subscriptions/MySub&#x000A;Received message: Message {&#x000A;  data: 'Publisher thinks Pub/Sub is awesome'&#x000A;  attributes: {}&#x000A;}&#x000A;Received message: Message {&#x000A;  data: 'Hello'&#x000A;  attributes: {}&#x000A;}&#x000A;Received message: Message {&#x000A;  data: "Publisher's name is Harry"&#x000A;  attributes: {}&#x000A;}&#x000A;Received message: Message {&#x000A;  data: 'Publisher likes to eat cheese'&#x000A;  attributes: {}&#x000A;}&#x000A;</code></pre>
<p>リッスンを停止するには <strong>Ctrl</strong>+<strong>C</strong> キーを押します。</p>

**参考（コード）**

`publisher.py`

```python
#!/usr/bin/env python

# Copyright 2016 Google LLC. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

"""This application demonstrates how to perform basic operations on topics
with the Cloud Pub/Sub API.

For more information, see the README.md under /pubsub and the documentation
at https://cloud.google.com/pubsub/docs.
"""

import argparse


def list_topics(project_id):
    """Lists all Pub/Sub topics in the given project."""
    # [START pubsub_list_topics]
    from google.cloud import pubsub_v1

    # TODO(developer)
    # project_id = "your-project-id"

    publisher = pubsub_v1.PublisherClient()
    project_path = publisher.project_path(project_id)

    for topic in publisher.list_topics(project_path):
        print(topic)
    # [END pubsub_list_topics]


def create_topic(project_id, topic_id):
    """Create a new Pub/Sub topic."""
    # [START pubsub_quickstart_create_topic]
    # [START pubsub_create_topic]
    from google.cloud import pubsub_v1

    # TODO(developer)
    # project_id = "your-project-id"
    # topic_id = "your-topic-id"

    publisher = pubsub_v1.PublisherClient()
    topic_path = publisher.topic_path(project_id, topic_id)

    topic = publisher.create_topic(topic_path)

    print("Topic created: {}".format(topic))
    # [END pubsub_quickstart_create_topic]
    # [END pubsub_create_topic]


def delete_topic(project_id, topic_id):
    """Deletes an existing Pub/Sub topic."""
    # [START pubsub_delete_topic]
    from google.cloud import pubsub_v1

    # TODO(developer)
    # project_id = "your-project-id"
    # topic_id = "your-topic-id"

    publisher = pubsub_v1.PublisherClient()
    topic_path = publisher.topic_path(project_id, topic_id)

    publisher.delete_topic(topic_path)

    print("Topic deleted: {}".format(topic_path))
    # [END pubsub_delete_topic]


def publish_messages(project_id, topic_id):
    """Publishes multiple messages to a Pub/Sub topic."""
    # [START pubsub_quickstart_publisher]
    # [START pubsub_publish]
    from google.cloud import pubsub_v1

    # TODO(developer)
    # project_id = "your-project-id"
    # topic_id = "your-topic-id"

    publisher = pubsub_v1.PublisherClient()
    # The `topic_path` method creates a fully qualified identifier
    # in the form `projects/{project_id}/topics/{topic_id}`
    topic_path = publisher.topic_path(project_id, topic_id)

    for n in range(1, 10):
        data = u"Message number {}".format(n)
        # Data must be a bytestring
        data = data.encode("utf-8")
        # When you publish a message, the client returns a future.
        future = publisher.publish(topic_path, data=data)
        print(future.result())

    print("Published messages.")
    # [END pubsub_quickstart_publisher]
    # [END pubsub_publish]


def publish_messages_with_custom_attributes(project_id, topic_id):
    """Publishes multiple messages with custom attributes
    to a Pub/Sub topic."""
    # [START pubsub_publish_custom_attributes]
    from google.cloud import pubsub_v1

    # TODO(developer)
    # project_id = "your-project-id"
    # topic_id = "your-topic-id"

    publisher = pubsub_v1.PublisherClient()
    topic_path = publisher.topic_path(project_id, topic_id)

    for n in range(1, 10):
        data = u"Message number {}".format(n)
        # Data must be a bytestring
        data = data.encode("utf-8")
        # Add two attributes, origin and username, to the message
        future = publisher.publish(
            topic_path, data, origin="python-sample", username="gcp"
        )
        print(future.result())

    print("Published messages with custom attributes.")
    # [END pubsub_publish_custom_attributes]


def publish_messages_with_error_handler(project_id, topic_id):
    # [START pubsub_publish_messages_error_handler]
    """Publishes multiple messages to a Pub/Sub topic with an error handler."""
    import time

    from google.cloud import pubsub_v1

    # TODO(developer)
    # project_id = "your-project-id"
    # topic_id = "your-topic-id"

    publisher = pubsub_v1.PublisherClient()
    topic_path = publisher.topic_path(project_id, topic_id)

    futures = dict()

    def get_callback(f, data):
        def callback(f):
            try:
                print(f.result())
                futures.pop(data)
            except:  # noqa
                print("Please handle {} for {}.".format(f.exception(), data))

        return callback

    for i in range(10):
        data = str(i)
        futures.update({data: None})
        # When you publish a message, the client returns a future.
        future = publisher.publish(
            topic_path, data=data.encode("utf-8")  # data must be a bytestring.
        )
        futures[data] = future
        # Publish failures shall be handled in the callback function.
        future.add_done_callback(get_callback(future, data))

    # Wait for all the publish futures to resolve before exiting.
    while futures:
        time.sleep(5)

    print("Published message with error handler.")
    # [END pubsub_publish_messages_error_handler]


def publish_messages_with_batch_settings(project_id, topic_id):
    """Publishes multiple messages to a Pub/Sub topic with batch settings."""
    # [START pubsub_publisher_batch_settings]
    from google.cloud import pubsub_v1

    # TODO(developer)
    # project_id = "your-project-id"
    # topic_id = "your-topic-id"

    # Configure the batch to publish as soon as there is ten messages,
    # one kilobyte of data, or one second has passed.
    batch_settings = pubsub_v1.types.BatchSettings(
        max_messages=10,  # default 100
        max_bytes=1024,  # default 1 MB
        max_latency=1,  # default 10 ms
    )
    publisher = pubsub_v1.PublisherClient(batch_settings)
    topic_path = publisher.topic_path(project_id, topic_id)

    # Resolve the publish future in a separate thread.
    def callback(future):
        message_id = future.result()
        print(message_id)

    for n in range(1, 10):
        data = u"Message number {}".format(n)
        # Data must be a bytestring
        data = data.encode("utf-8")
        future = publisher.publish(topic_path, data=data)
        # Non-blocking. Allow the publisher client to batch multiple messages.
        future.add_done_callback(callback)

    print("Published messages with batch settings.")
    # [END pubsub_publisher_batch_settings]


def publish_messages_with_retry_settings(project_id, topic_id):
    """Publishes messages with custom retry settings."""
    # [START pubsub_publisher_retry_settings]
    from google.cloud import pubsub_v1

    # TODO(developer)
    # project_id = "your-project-id"
    # topic_id = "your-topic-id"

    # Configure the retry settings. Defaults will be overwritten.
    retry_settings = {
        "interfaces": {
            "google.pubsub.v1.Publisher": {
                "retry_codes": {
                    "publish": [
                        "ABORTED",
                        "CANCELLED",
                        "DEADLINE_EXCEEDED",
                        "INTERNAL",
                        "RESOURCE_EXHAUSTED",
                        "UNAVAILABLE",
                        "UNKNOWN",
                    ]
                },
                "retry_params": {
                    "messaging": {
                        "initial_retry_delay_millis": 100,  # default: 100
                        "retry_delay_multiplier": 1.3,  # default: 1.3
                        "max_retry_delay_millis": 60000,  # default: 60000
                        "initial_rpc_timeout_millis": 5000,  # default: 25000
                        "rpc_timeout_multiplier": 1.0,  # default: 1.0
                        "max_rpc_timeout_millis": 600000,  # default: 30000
                        "total_timeout_millis": 600000,  # default: 600000
                    }
                },
                "methods": {
                    "Publish": {
                        "retry_codes_name": "publish",
                        "retry_params_name": "messaging",
                    }
                },
            }
        }
    }

    publisher = pubsub_v1.PublisherClient(client_config=retry_settings)
    topic_path = publisher.topic_path(project_id, topic_id)

    for n in range(1, 10):
        data = u"Message number {}".format(n)
        # Data must be a bytestring
        data = data.encode("utf-8")
        future = publisher.publish(topic_path, data=data)
        print(future.result())

    print("Published messages with retry settings.")
    # [END pubsub_publisher_retry_settings]


if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description=__doc__, formatter_class=argparse.RawDescriptionHelpFormatter,
    )
    parser.add_argument("project_id", help="Your Google Cloud project ID")

    subparsers = parser.add_subparsers(dest="command")
    subparsers.add_parser("list", help=list_topics.__doc__)

    create_parser = subparsers.add_parser("create", help=create_topic.__doc__)
    create_parser.add_argument("topic_id")

    delete_parser = subparsers.add_parser("delete", help=delete_topic.__doc__)
    delete_parser.add_argument("topic_id")

    publish_parser = subparsers.add_parser("publish", help=publish_messages.__doc__)
    publish_parser.add_argument("topic_id")

    publish_with_custom_attributes_parser = subparsers.add_parser(
        "publish-with-custom-attributes",
        help=publish_messages_with_custom_attributes.__doc__,
    )
    publish_with_custom_attributes_parser.add_argument("topic_id")

    publish_with_error_handler_parser = subparsers.add_parser(
        "publish-with-error-handler", help=publish_messages_with_error_handler.__doc__,
    )
    publish_with_error_handler_parser.add_argument("topic_id")

    publish_with_batch_settings_parser = subparsers.add_parser(
        "publish-with-batch-settings",
        help=publish_messages_with_batch_settings.__doc__,
    )
    publish_with_batch_settings_parser.add_argument("topic_id")

    publish_with_retry_settings_parser = subparsers.add_parser(
        "publish-with-retry-settings",
        help=publish_messages_with_retry_settings.__doc__,
    )
    publish_with_retry_settings_parser.add_argument("topic_id")

    args = parser.parse_args()

    if args.command == "list":
        list_topics(args.project_id)
    elif args.command == "create":
        create_topic(args.project_id, args.topic_id)
    elif args.command == "delete":
        delete_topic(args.project_id, args.topic_id)
    elif args.command == "publish":
        publish_messages(args.project_id, args.topic_id)
    elif args.command == "publish-with-custom-attributes":
        publish_messages_with_custom_attributes(args.project_id, args.topic_id)
    elif args.command == "publish-with-error-handler":
        publish_messages_with_error_handler(args.project_id, args.topic_id)
    elif args.command == "publish-with-batch-settings":
        publish_messages_with_batch_settings(args.project_id, args.topic_id)
    elif args.command == "publish-with-retry-settings":
        publish_messages_with_retry_settings(args.project_id, args.topic_id)
```

`subscriber.py`

```python
#!/usr/bin/env python

# Copyright 2016 Google Inc. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

"""This application demonstrates how to perform basic operations on
subscriptions with the Cloud Pub/Sub API.

For more information, see the README.md under /pubsub and the documentation
at https://cloud.google.com/pubsub/docs.
"""

import argparse


def list_subscriptions_in_topic(project_id, topic_id):
    """Lists all subscriptions for a given topic."""
    # [START pubsub_list_topic_subscriptions]
    from google.cloud import pubsub_v1

    # TODO(developer)
    # project_id = "your-project-id"
    # topic_id = "your-topic-id"

    publisher = pubsub_v1.PublisherClient()
    topic_path = publisher.topic_path(project_id, topic_id)

    for subscription in publisher.list_topic_subscriptions(topic_path):
        print(subscription)
    # [END pubsub_list_topic_subscriptions]


def list_subscriptions_in_project(project_id):
    """Lists all subscriptions in the current project."""
    # [START pubsub_list_subscriptions]
    from google.cloud import pubsub_v1

    # TODO(developer)
    # project_id = "your-project-id"

    subscriber = pubsub_v1.SubscriberClient()
    project_path = subscriber.project_path(project_id)

    # Wrap the subscriber in a 'with' block to automatically call close() to
    # close the underlying gRPC channel when done.
    with subscriber:
        for subscription in subscriber.list_subscriptions(project_path):
            print(subscription.name)
    # [END pubsub_list_subscriptions]


def create_subscription(project_id, topic_id, subscription_id):
    """Create a new pull subscription on the given topic."""
    # [START pubsub_create_pull_subscription]
    from google.cloud import pubsub_v1

    # TODO(developer)
    # project_id = "your-project-id"
    # topic_id = "your-topic-id"
    # subscription_id = "your-subscription-id"

    subscriber = pubsub_v1.SubscriberClient()
    topic_path = subscriber.topic_path(project_id, topic_id)
    subscription_path = subscriber.subscription_path(project_id, subscription_id)

    # Wrap the subscriber in a 'with' block to automatically call close() to
    # close the underlying gRPC channel when done.
    with subscriber:
        subscription = subscriber.create_subscription(subscription_path, topic_path)

    print("Subscription created: {}".format(subscription))
    # [END pubsub_create_pull_subscription]


def create_subscription_with_dead_letter_topic(
    project_id, topic_id, subscription_id, dead_letter_topic_id
):
    """Create a subscription with dead letter policy."""
    # [START pubsub_dead_letter_create_subscription]
    from google.cloud import pubsub_v1
    from google.cloud.pubsub_v1.types import DeadLetterPolicy

    # TODO(developer)
    # project_id = "your-project-id"
    # endpoint = "https://my-test-project.appspot.com/push"
    # TODO(developer): This is an existing topic that the subscription
    # with dead letter policy is attached to.
    # topic_id = "your-topic-id"
    # TODO(developer): This is an existing subscription with a dead letter policy.
    # subscription_id = "your-subscription-id"
    # TODO(developer): This is an existing dead letter topic that the subscription
    # with dead letter policy will forward dead letter messages to.
    # dead_letter_topic_id = "your-dead-letter-topic-id"

    subscriber = pubsub_v1.SubscriberClient()
    topic_path = subscriber.topic_path(project_id, topic_id)
    subscription_path = subscriber.subscription_path(project_id, subscription_id)
    dead_letter_topic_path = subscriber.topic_path(project_id, dead_letter_topic_id)

    dead_letter_policy = DeadLetterPolicy(
        dead_letter_topic=dead_letter_topic_path, max_delivery_attempts=10
    )

    with subscriber:
        subscription = subscriber.create_subscription(
            subscription_path, topic_path, dead_letter_policy=dead_letter_policy
        )

    print("Subscription created: {}".format(subscription.name))
    print(
        "It will forward dead letter messages to: {}".format(
            subscription.dead_letter_policy.dead_letter_topic
        )
    )
    print(
        "After {} delivery attempts.".format(
            subscription.dead_letter_policy.max_delivery_attempts
        )
    )
    # [END pubsub_dead_letter_create_subscription]


def create_push_subscription(project_id, topic_id, subscription_id, endpoint):
    """Create a new push subscription on the given topic."""
    # [START pubsub_create_push_subscription]
    from google.cloud import pubsub_v1

    # TODO(developer)
    # project_id = "your-project-id"
    # topic_id = "your-topic-id"
    # subscription_id = "your-subscription-id"
    # endpoint = "https://my-test-project.appspot.com/push"

    subscriber = pubsub_v1.SubscriberClient()
    topic_path = subscriber.topic_path(project_id, topic_id)
    subscription_path = subscriber.subscription_path(project_id, subscription_id)

    push_config = pubsub_v1.types.PushConfig(push_endpoint=endpoint)

    # Wrap the subscriber in a 'with' block to automatically call close() to
    # close the underlying gRPC channel when done.
    with subscriber:
        subscription = subscriber.create_subscription(
            subscription_path, topic_path, push_config
        )

    print("Push subscription created: {}".format(subscription))
    print("Endpoint for subscription is: {}".format(endpoint))
    # [END pubsub_create_push_subscription]


def delete_subscription(project_id, subscription_id):
    """Deletes an existing Pub/Sub topic."""
    # [START pubsub_delete_subscription]
    from google.cloud import pubsub_v1

    # TODO(developer)
    # project_id = "your-project-id"
    # subscription_id = "your-subscription-id"

    subscriber = pubsub_v1.SubscriberClient()
    subscription_path = subscriber.subscription_path(project_id, subscription_id)

    # Wrap the subscriber in a 'with' block to automatically call close() to
    # close the underlying gRPC channel when done.
    with subscriber:
        subscriber.delete_subscription(subscription_path)

    print("Subscription deleted: {}".format(subscription_path))
    # [END pubsub_delete_subscription]


def update_push_subscription(project_id, topic_id, subscription_id, endpoint):
    """
    Updates an existing Pub/Sub subscription's push endpoint URL.
    Note that certain properties of a subscription, such as
    its topic, are not modifiable.
    """
    # [START pubsub_update_push_configuration]
    from google.cloud import pubsub_v1

    # TODO(developer)
    # project_id = "your-project-id"
    # topic_id = "your-topic-id"
    # subscription_id = "your-subscription-id"
    # endpoint = "https://my-test-project.appspot.com/push"

    subscriber = pubsub_v1.SubscriberClient()
    subscription_path = subscriber.subscription_path(project_id, subscription_id)

    push_config = pubsub_v1.types.PushConfig(push_endpoint=endpoint)

    subscription = pubsub_v1.types.Subscription(
        name=subscription_path, topic=topic_id, push_config=push_config
    )

    update_mask = {"paths": {"push_config"}}

    # Wrap the subscriber in a 'with' block to automatically call close() to
    # close the underlying gRPC channel when done.
    with subscriber:
        result = subscriber.update_subscription(subscription, update_mask)

    print("Subscription updated: {}".format(subscription_path))
    print("New endpoint for subscription is: {}".format(result.push_config))
    # [END pubsub_update_push_configuration]


def update_subscription_with_dead_letter_policy(
    project_id, topic_id, subscription_id, dead_letter_topic_id
):
    """Update a subscription's dead letter policy."""
    # [START pubsub_dead_letter_update_subscription]
    from google.cloud import pubsub_v1
    from google.cloud.pubsub_v1.types import DeadLetterPolicy, FieldMask

    # TODO(developer)
    # project_id = "your-project-id"
    # TODO(developer): This is an existing topic that the subscription
    # with dead letter policy is attached to.
    # topic_id = "your-topic-id"
    # TODO(developer): This is an existing subscription with a dead letter policy.
    # subscription_id = "your-subscription-id"
    # TODO(developer): This is an existing dead letter topic that the subscription
    # with dead letter policy will forward dead letter messages to.
    # dead_letter_topic_id = "your-dead-letter-topic-id"

    subscriber = pubsub_v1.SubscriberClient()
    topic_path = subscriber.topic_path(project_id, topic_id)
    subscription_path = subscriber.subscription_path(project_id, subscription_id)
    dead_letter_topic_path = subscriber.topic_path(project_id, dead_letter_topic_id)

    subscription_before_update = subscriber.get_subscription(subscription_path)
    print("Before the update: {}".format(subscription_before_update))

    # Indicates which fields in the provided subscription to update.
    update_mask = FieldMask(paths=["dead_letter_policy.max_delivery_attempts"])

    # Construct a dead letter policy you expect to have after the update.
    dead_letter_policy = DeadLetterPolicy(
        dead_letter_topic=dead_letter_topic_path, max_delivery_attempts=20
    )

    # Construct the subscription with the dead letter policy you expect to have
    # after the update. Here, values in the required fields (name, topic) help
    # identify the subscription.
    subscription = pubsub_v1.types.Subscription(
        name=subscription_path, topic=topic_path, dead_letter_policy=dead_letter_policy,
    )

    with subscriber:
        subscription_after_update = subscriber.update_subscription(
            subscription, update_mask
        )

    print("After the update: {}".format(subscription_after_update))
    # [END pubsub_dead_letter_update_subscription]
    return subscription_after_update


def remove_dead_letter_policy(project_id, topic_id, subscription_id):
    """Remove dead letter policy from a subscription."""
    # [START pubsub_dead_letter_remove]
    from google.cloud import pubsub_v1
    from google.cloud.pubsub_v1.types import FieldMask

    # TODO(developer)
    # project_id = "your-project-id"
    # TODO(developer): This is an existing topic that the subscription
    # with dead letter policy is attached to.
    # topic_id = "your-topic-id"
    # TODO(developer): This is an existing subscription with a dead letter policy.
    # subscription_id = "your-subscription-id"

    subscriber = pubsub_v1.SubscriberClient()
    topic_path = subscriber.topic_path(project_id, topic_id)
    subscription_path = subscriber.subscription_path(project_id, subscription_id)

    subscription_before_update = subscriber.get_subscription(subscription_path)
    print("Before removing the policy: {}".format(subscription_before_update))

    # Indicates which fields in the provided subscription to update.
    update_mask = FieldMask(
        paths=[
            "dead_letter_policy.dead_letter_topic",
            "dead_letter_policy.max_delivery_attempts",
        ]
    )

    # Construct the subscription (without any dead letter policy) that you
    # expect to have after the update.
    subscription = pubsub_v1.types.Subscription(
        name=subscription_path, topic=topic_path
    )

    with subscriber:
        subscription_after_update = subscriber.update_subscription(
            subscription, update_mask
        )

    print("After removing the policy: {}".format(subscription_after_update))
    # [END pubsub_dead_letter_remove]
    return subscription_after_update


def receive_messages(project_id, subscription_id, timeout=None):
    """Receives messages from a pull subscription."""
    # [START pubsub_subscriber_async_pull]
    # [START pubsub_quickstart_subscriber]
    from concurrent.futures import TimeoutError
    from google.cloud import pubsub_v1

    # TODO(developer)
    # project_id = "your-project-id"
    # subscription_id = "your-subscription-id"
    # Number of seconds the subscriber should listen for messages
    # timeout = 5.0

    subscriber = pubsub_v1.SubscriberClient()
    # The `subscription_path` method creates a fully qualified identifier
    # in the form `projects/{project_id}/subscriptions/{subscription_id}`
    subscription_path = subscriber.subscription_path(project_id, subscription_id)

    def callback(message):
        print("Received message: {}".format(message))
        message.ack()

    streaming_pull_future = subscriber.subscribe(subscription_path, callback=callback)
    print("Listening for messages on {}..\n".format(subscription_path))

    # Wrap subscriber in a 'with' block to automatically call close() when done.
    with subscriber:
        try:
            # When `timeout` is not set, result() will block indefinitely,
            # unless an exception is encountered first.
            streaming_pull_future.result(timeout=timeout)
        except TimeoutError:
            streaming_pull_future.cancel()
    # [END pubsub_subscriber_async_pull]
    # [END pubsub_quickstart_subscriber]


def receive_messages_with_custom_attributes(project_id, subscription_id, timeout=None):
    """Receives messages from a pull subscription."""
    # [START pubsub_subscriber_async_pull_custom_attributes]
    from concurrent.futures import TimeoutError
    from google.cloud import pubsub_v1

    # TODO(developer)
    # project_id = "your-project-id"
    # subscription_id = "your-subscription-id"
    # Number of seconds the subscriber should listen for messages
    # timeout = 5.0

    subscriber = pubsub_v1.SubscriberClient()
    subscription_path = subscriber.subscription_path(project_id, subscription_id)

    def callback(message):
        print("Received message: {}".format(message.data))
        if message.attributes:
            print("Attributes:")
            for key in message.attributes:
                value = message.attributes.get(key)
                print("{}: {}".format(key, value))
        message.ack()

    streaming_pull_future = subscriber.subscribe(subscription_path, callback=callback)
    print("Listening for messages on {}..\n".format(subscription_path))

    # Wrap subscriber in a 'with' block to automatically call close() when done.
    with subscriber:
        try:
            # When `timeout` is not set, result() will block indefinitely,
            # unless an exception is encountered first.
            streaming_pull_future.result(timeout=timeout)
        except TimeoutError:
            streaming_pull_future.cancel()
    # [END pubsub_subscriber_async_pull_custom_attributes]


def receive_messages_with_flow_control(project_id, subscription_id, timeout=None):
    """Receives messages from a pull subscription with flow control."""
    # [START pubsub_subscriber_flow_settings]
    from concurrent.futures import TimeoutError
    from google.cloud import pubsub_v1

    # TODO(developer)
    # project_id = "your-project-id"
    # subscription_id = "your-subscription-id"
    # Number of seconds the subscriber should listen for messages
    # timeout = 5.0

    subscriber = pubsub_v1.SubscriberClient()
    subscription_path = subscriber.subscription_path(project_id, subscription_id)

    def callback(message):
        print("Received message: {}".format(message.data))
        message.ack()

    # Limit the subscriber to only have ten outstanding messages at a time.
    flow_control = pubsub_v1.types.FlowControl(max_messages=10)

    streaming_pull_future = subscriber.subscribe(
        subscription_path, callback=callback, flow_control=flow_control
    )
    print("Listening for messages on {}..\n".format(subscription_path))

    # Wrap subscriber in a 'with' block to automatically call close() when done.
    with subscriber:
        try:
            # When `timeout` is not set, result() will block indefinitely,
            # unless an exception is encountered first.
            streaming_pull_future.result(timeout=timeout)
        except TimeoutError:
            streaming_pull_future.cancel()
    # [END pubsub_subscriber_flow_settings]


def synchronous_pull(project_id, subscription_id):
    """Pulling messages synchronously."""
    # [START pubsub_subscriber_sync_pull]
    from google.cloud import pubsub_v1

    # TODO(developer)
    # project_id = "your-project-id"
    # subscription_id = "your-subscription-id"

    subscriber = pubsub_v1.SubscriberClient()
    subscription_path = subscriber.subscription_path(project_id, subscription_id)

    NUM_MESSAGES = 3

    # Wrap the subscriber in a 'with' block to automatically call close() to
    # close the underlying gRPC channel when done.
    with subscriber:
        # The subscriber pulls a specific number of messages.
        response = subscriber.pull(subscription_path, max_messages=NUM_MESSAGES)

        ack_ids = []
        for received_message in response.received_messages:
            print("Received: {}".format(received_message.message.data))
            ack_ids.append(received_message.ack_id)

        # Acknowledges the received messages so they will not be sent again.
        subscriber.acknowledge(subscription_path, ack_ids)

        print(
            "Received and acknowledged {} messages. Done.".format(
                len(response.received_messages)
            )
        )
    # [END pubsub_subscriber_sync_pull]


def synchronous_pull_with_lease_management(project_id, subscription_id):
    """Pulling messages synchronously with lease management"""
    # [START pubsub_subscriber_sync_pull_with_lease]
    import logging
    import multiprocessing
    import random
    import time

    from google.cloud import pubsub_v1

    # TODO(developer)
    # project_id = "your-project-id"
    # subscription_id = "your-subscription-id"

    subscriber = pubsub_v1.SubscriberClient()
    subscription_path = subscriber.subscription_path(project_id, subscription_id)

    NUM_MESSAGES = 2
    ACK_DEADLINE = 30
    SLEEP_TIME = 10

    # The subscriber pulls a specific number of messages.
    response = subscriber.pull(subscription_path, max_messages=NUM_MESSAGES)

    multiprocessing.log_to_stderr()
    logger = multiprocessing.get_logger()
    logger.setLevel(logging.INFO)

    def worker(msg):
        """Simulates a long-running process."""
        RUN_TIME = random.randint(1, 60)
        logger.info(
            "{}: Running {} for {}s".format(
                time.strftime("%X", time.gmtime()), msg.message.data, RUN_TIME
            )
        )
        time.sleep(RUN_TIME)

    # `processes` stores process as key and ack id and message as values.
    processes = dict()
    for message in response.received_messages:
        process = multiprocessing.Process(target=worker, args=(message,))
        processes[process] = (message.ack_id, message.message.data)
        process.start()

    while processes:
        for process in list(processes):
            ack_id, msg_data = processes[process]
            # If the process is still running, reset the ack deadline as
            # specified by ACK_DEADLINE once every while as specified
            # by SLEEP_TIME.
            if process.is_alive():
                # `ack_deadline_seconds` must be between 10 to 600.
                subscriber.modify_ack_deadline(
                    subscription_path, [ack_id], ack_deadline_seconds=ACK_DEADLINE,
                )
                logger.info(
                    "{}: Reset ack deadline for {} for {}s".format(
                        time.strftime("%X", time.gmtime()), msg_data, ACK_DEADLINE,
                    )
                )

            # If the processs is finished, acknowledges using `ack_id`.
            else:
                subscriber.acknowledge(subscription_path, [ack_id])
                logger.info(
                    "{}: Acknowledged {}".format(
                        time.strftime("%X", time.gmtime()), msg_data
                    )
                )
                processes.pop(process)

        # If there are still processes running, sleeps the thread.
        if processes:
            time.sleep(SLEEP_TIME)

    print(
        "Received and acknowledged {} messages. Done.".format(
            len(response.received_messages)
        )
    )

    # Close the underlying gPRC channel. Alternatively, wrap subscriber in
    # a 'with' block to automatically call close() when done.
    subscriber.close()
    # [END pubsub_subscriber_sync_pull_with_lease]


def listen_for_errors(project_id, subscription_id, timeout=None):
    """Receives messages and catches errors from a pull subscription."""
    # [START pubsub_subscriber_error_listener]
    from google.cloud import pubsub_v1

    # TODO(developer)
    # project_id = "your-project-id"
    # subscription_id = "your-subscription-id"
    # Number of seconds the subscriber should listen for messages
    # timeout = 5.0

    subscriber = pubsub_v1.SubscriberClient()
    subscription_path = subscriber.subscription_path(project_id, subscription_id)

    def callback(message):
        print("Received message: {}".format(message))
        message.ack()

    streaming_pull_future = subscriber.subscribe(subscription_path, callback=callback)
    print("Listening for messages on {}..\n".format(subscription_path))

    # Wrap subscriber in a 'with' block to automatically call close() when done.
    with subscriber:
        # When `timeout` is not set, result() will block indefinitely,
        # unless an exception is encountered first.
        try:
            streaming_pull_future.result(timeout=timeout)
        except Exception as e:
            streaming_pull_future.cancel()
            print(
                "Listening for messages on {} threw an exception: {}.".format(
                    subscription_id, e
                )
            )
    # [END pubsub_subscriber_error_listener]


def receive_messages_with_delivery_attempts(project_id, subscription_id, timeout=None):
    # [START  pubsub_dead_letter_delivery_attempt]
    from concurrent.futures import TimeoutError
    from google.cloud import pubsub_v1

    # TODO(developer)
    # project_id = "your-project-id"
    # subscription_id = "your-subscription-id"

    subscriber = pubsub_v1.SubscriberClient()
    subscription_path = subscriber.subscription_path(project_id, subscription_id)

    def callback(message):
        print("Received message: {}".format(message))
        print("With delivery attempts: {}".format(message.delivery_attempt))
        message.ack()

    streaming_pull_future = subscriber.subscribe(subscription_path, callback=callback)
    print("Listening for messages on {}..\n".format(subscription_path))

    # Wrap subscriber in a 'with' block to automatically call close() when done.
    with subscriber:
        # When `timeout` is not set, result() will block indefinitely,
        # unless an exception is encountered first.
        try:
            streaming_pull_future.result(timeout=timeout)
        except TimeoutError:
            streaming_pull_future.cancel()
    # [END  pubsub_dead_letter_delivery_attempt]


if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description=__doc__, formatter_class=argparse.RawDescriptionHelpFormatter,
    )
    parser.add_argument("project_id", help="Your Google Cloud project ID")

    subparsers = parser.add_subparsers(dest="command")
    list_in_topic_parser = subparsers.add_parser(
        "list-in-topic", help=list_subscriptions_in_topic.__doc__
    )
    list_in_topic_parser.add_argument("topic_id")

    list_in_project_parser = subparsers.add_parser(
        "list-in-project", help=list_subscriptions_in_project.__doc__
    )

    create_parser = subparsers.add_parser("create", help=create_subscription.__doc__)
    create_parser.add_argument("topic_id")
    create_parser.add_argument("subscription_id")

    create_with_dead_letter_policy_parser = subparsers.add_parser(
        "create-with-dead-letter-policy",
        help=create_subscription_with_dead_letter_topic.__doc__,
    )
    create_with_dead_letter_policy_parser.add_argument("topic_id")
    create_with_dead_letter_policy_parser.add_argument("subscription_id")
    create_with_dead_letter_policy_parser.add_argument("dead_letter_topic_id")

    create_push_parser = subparsers.add_parser(
        "create-push", help=create_push_subscription.__doc__
    )
    create_push_parser.add_argument("topic_id")
    create_push_parser.add_argument("subscription_id")
    create_push_parser.add_argument("endpoint")

    delete_parser = subparsers.add_parser("delete", help=delete_subscription.__doc__)
    delete_parser.add_argument("subscription_id")

    update_push_parser = subparsers.add_parser(
        "update-push", help=update_push_subscription.__doc__
    )
    update_push_parser.add_argument("topic_id")
    update_push_parser.add_argument("subscription_id")
    update_push_parser.add_argument("endpoint")

    update_dead_letter_policy_parser = subparsers.add_parser(
        "update-dead-letter-policy",
        help=update_subscription_with_dead_letter_policy.__doc__,
    )
    update_dead_letter_policy_parser.add_argument("topic_id")
    update_dead_letter_policy_parser.add_argument("subscription_id")
    update_dead_letter_policy_parser.add_argument("dead_letter_topic_id")

    remove_dead_letter_policy_parser = subparsers.add_parser(
        "remove-dead-letter-policy", help=remove_dead_letter_policy.__doc__
    )
    remove_dead_letter_policy_parser.add_argument("topic_id")
    remove_dead_letter_policy_parser.add_argument("subscription_id")

    receive_parser = subparsers.add_parser("receive", help=receive_messages.__doc__)
    receive_parser.add_argument("subscription_id")
    receive_parser.add_argument("timeout", default=None, type=float, nargs="?")

    receive_with_custom_attributes_parser = subparsers.add_parser(
        "receive-custom-attributes",
        help=receive_messages_with_custom_attributes.__doc__,
    )
    receive_with_custom_attributes_parser.add_argument("subscription_id")
    receive_with_custom_attributes_parser.add_argument(
        "timeout", default=None, type=float, nargs="?"
    )

    receive_with_flow_control_parser = subparsers.add_parser(
        "receive-flow-control", help=receive_messages_with_flow_control.__doc__
    )
    receive_with_flow_control_parser.add_argument("subscription_id")
    receive_with_flow_control_parser.add_argument(
        "timeout", default=None, type=float, nargs="?"
    )

    synchronous_pull_parser = subparsers.add_parser(
        "receive-synchronously", help=synchronous_pull.__doc__
    )
    synchronous_pull_parser.add_argument("subscription_id")

    synchronous_pull_with_lease_management_parser = subparsers.add_parser(
        "receive-synchronously-with-lease",
        help=synchronous_pull_with_lease_management.__doc__,
    )
    synchronous_pull_with_lease_management_parser.add_argument("subscription_id")

    listen_for_errors_parser = subparsers.add_parser(
        "listen-for-errors", help=listen_for_errors.__doc__
    )
    listen_for_errors_parser.add_argument("subscription_id")
    listen_for_errors_parser.add_argument(
        "timeout", default=None, type=float, nargs="?"
    )

    receive_messages_with_delivery_attempts_parser = subparsers.add_parser(
        "receive-messages-with-delivery-attempts",
        help=receive_messages_with_delivery_attempts.__doc__,
    )
    receive_messages_with_delivery_attempts_parser.add_argument("subscription_id")
    receive_messages_with_delivery_attempts_parser.add_argument(
        "timeout", default=None, type=float, nargs="?"
    )

    args = parser.parse_args()

    if args.command == "list-in-topic":
        list_subscriptions_in_topic(args.project_id, args.topic_id)
    elif args.command == "list-in-project":
        list_subscriptions_in_project(args.project_id)
    elif args.command == "create":
        create_subscription(args.project_id, args.topic_id, args.subscription_id)
    elif args.command == "create-with-dead-letter-policy":
        create_subscription_with_dead_letter_topic(
            args.project_id,
            args.topic_id,
            args.subscription_id,
            args.dead_letter_topic_id,
        )
    elif args.command == "create-push":
        create_push_subscription(
            args.project_id, args.topic_id, args.subscription_id, args.endpoint,
        )
    elif args.command == "delete":
        delete_subscription(args.project_id, args.subscription_id)
    elif args.command == "update-push":
        update_push_subscription(
            args.project_id, args.topic_id, args.subscription_id, args.endpoint,
        )
    elif args.command == "update-dead-letter-policy":
        update_subscription_with_dead_letter_policy(
            args.project_id,
            args.topic_id,
            args.subscription_id,
            args.dead_letter_topic_id,
        )
    elif args.command == "remove-dead-letter-policy":
        remove_dead_letter_policy(args.project_id, args.topic_id, args.subscription_id)
    elif args.command == "receive":
        receive_messages(args.project_id, args.subscription_id, args.timeout)
    elif args.command == "receive-custom-attributes":
        receive_messages_with_custom_attributes(
            args.project_id, args.subscription_id, args.timeout
        )
    elif args.command == "receive-flow-control":
        receive_messages_with_flow_control(
            args.project_id, args.subscription_id, args.timeout
        )
    elif args.command == "receive-synchronously":
        synchronous_pull(args.project_id, args.subscription_id)
    elif args.command == "receive-synchronously-with-lease":
        synchronous_pull_with_lease_management(args.project_id, args.subscription_id)
    elif args.command == "listen-for-errors":
        listen_for_errors(args.project_id, args.subscription_id, args.timeout)
    elif args.command == "receive-messages-with-delivery-attempts":
        receive_messages_with_delivery_attempts(
            args.project_id, args.subscription_id, args.timeout
        )
```