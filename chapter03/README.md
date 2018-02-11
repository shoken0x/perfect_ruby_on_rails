# 3章 アセット

担当: @syokenz

## Summary

- Sprockets: アセットファイルを管理する仕組みAsset Pipelineの基盤パッケージ https://github.com/rails/sprockets
- CoffeeScript, Sass: 資料では省略。自習しましょう。
- Turbolinks: Pjaxを利用したページ遷移を提供する。Rails5でもTurbolinks5として入っている機能。


## Sprockets p.94

### asset pipeline

> - Sprockets performs the asset packaging which takes the assets from all the specified paths, compiles them together and places them in the target path (public/assets).
- Tilt is the template engine that Sprockets uses. This allows file types like scss and erb to be used in the asset pipeline. See the Tilt Readme for a list of supported template engines.

![asset_pipeline_flow.png](https://qiita-image-store.s3.amazonaws.com/0/32750/fdc23bb8-cdc5-d503-1a78-52d52b9f61db.png "asset_pipeline_flow.png")


http://coderberry.me/blog/2012/04/24/asset-pipeline-for-dummies/

Railsdoc
http://railsdoc.com/asset_pipeline

RailsGuides
http://railsguides.jp/asset_pipeline.html

### Sprocketsの概要 p.94

- アセットファイルにアクセスするためのパスを管理する
- アセットのコンパイル
- アセットファイル同士の依存性を管理する

Sprocketsの最も基本的な機能はアセットファイルのパスを仮想的なディレクトリ構造にまとめて管理すること。

#### 仮想的なディレクトリ構造とは？

`app/assets/javascripts/xxx.js`  
`app/assets/stylesheets/xxx.css`  
という風にJavaScriptとcssはディレクトリを分けて管理したい。

また同じJavaScriptでも共通で利用するライブラリやサブ機能に関わるアセットファイルは`lib/assets`に、外部から取得するアセットファイルは`vendor/assets`という風にディレクトリをわけたい。

実際にファイルを管理しているディレクトリは別れているが、URL上は1つのディレクトリにあるかのようにアクセスする機能をSprocketsは提供する。

上記のアセットファイルは全て`/assets/ファイル名`でアクセス可能になる。


#### KitchHikeの例

DIR: `app/assets/images/kh_logo.png`  
URL: https://kitchhike.com/assets/kh_logo-b1ec0b61ee0a50e250a5425e5afa456b.png

JavaScriptとcssはminifyとconcatenate(結合)されて1つのファイルとなっている。

URL: https://kitchhike.com/assets/application-eb43c7de58f6e93021aaa58b13945107.js  
URL: https://kitchhike.com/assets/application-d50924e9690f10f54e70e390fb96cd41.css


### コンパイルとAsset Pipeline p.97

#### Asset Pipelineとは？

SproketsがCoffeeScriptやsassファイルの内容を変換（コンパイル）してレスポンスを適切な形にしてから返却する機能の通称。

- development環境では、アクセスの度にコンパイルされる
- production環境では、`rake assets:precompile`でデプロイ時にコンパイルを実行し、コンパイル済みのファイルを返すためレスポンスが向上する




### 依存性の管理 p.99

ファイルにディレクティブを追加することで、依存性を管理できる。  
例: requireに指定されたファイルが無かったらエラー

`//=require jquery`

### マニフェストファイル p.100

`app/assets/javascripts/application.js`  
`app/assets/stylesheets/application.css`

マニフェストファイルにはディレクティブを書く。  
実際には、`require`などで指定されている全てのファイルが結合されるので、マニフェストファイルにアクセスするだけですべて必要なJavaScriptやcssが読み込まれるようになっている。

KitchHike
https://kitchhike.com/assets/application-eb43c7de58f6e93021aaa58b13945107.js  
https://kitchhike.com/assets/application-d50924e9690f10f54e70e390fb96cd41.css

#### Railsにおけるdevelopment環境とproduction環境の違い p.101

マニフェストファイルはヘルパーメソッドの`stylesheet_link_tag`, `javascript_include_tag`で読み込まれる。  
development環境では、それぞれ個別のファイルの読み込みタグに分割されて出力される。ディレクティブで指定した読み込みの順番がちゃんと反映された結果となる。

例

```
<script src="/assets/jquery.js?body=1"></script>
<script src="/assets/jquery_ujs.js?body=1"></script>
<script src="/assets/twitter/bootstrap/transition.js?body=1"></script>
<script src="/assets/twitter/bootstrap/alert.js?body=1"></script>
<script src="/assets/twitter/bootstrap/modal.js?body=1"></script>
<script src="/assets/twitter/bootstrap/dropdown.js?body=1"></script>
<script src="/assets/twitter/bootstrap/scrollspy.js?body=1"></script>
<script src="/assets/twitter/bootstrap/tab.js?body=1"></script>
...
```

#### マニフェストファイルのプリコンパイル p.103

production環境ではプリコンパイルを実施する。事前にプリコンパイルしたファイルを静的ファイルとして提供することで、通常のJavaScriptやcssと変わらないレスポンスを実現できる。

`rake assets:precompile`

- fingerprint（ダイジェスト）をファイル名に追加したアセットファイルを、`public/assets`以下に配置(p.129のcolumn参考)
- JavaScript,cssのminifyを実行
- ディレクティブに従ってconcatenateを実行


## CoffeeScript

省略

## Sass

省略



## Turbolinks p.124

Pjaxを用いたページ遷移の高速化手法。Pjaxでは、URLを更新しつつページ全体を描画せずに必要な部分のみを更新できる。TurbolinksはPjaxを簡易化し簡単に利用できるようにした仕組み。

### Turbolinksの動作について p.124

1. ページ内のクリックイベントを補足し、それがAタグに対するクリックだった時Turbolinksによる遷移を実行する。(GETのみ。POST/DELETE/PUTでは実行しない)
2. TurbolinksはAjaxによるリクエストを実行し、リンク先のHTMLを取得する。
3. 取得したHTMLのheadタグ内にあるアセットを確認し、現在のページのアセットと同じであれば処理を続行する。異なっていれば、location.hrefを書き換えリンク先のURLにリダイレクトし処理を終了する。
4. 取得したHTMLのタイトルとbodyタグの中身を抽出する。
5. 現在のページのタイトルとbodyタグを抽出したものとを入れ替えて更新する。
6. window.history.pushStateを実行し、参照しているURLをリンク先のURLに更新する。


### Turbolinksの利用方法と無効にする方法 p.125

デフォルトで有効となっている。
無効にするには、`app/assets/javascripts/application.js`から`//=require turbolinks`を削除。Gemfileからも`gem 'turbolinks`を削除。


### Turbolinksを利用する時の注意点 p.125

参考サイトを読もう。


#### Turbolinksの問題
[Rails - Turbolinksをオフしないためにやった事 - Qiita](http://qiita.com/saboyutaka/items/bcc0966313c6f7399a6e)

>よく言われるTurbolinksの問題

>- $(document).ready()が発火しない
- $(window).load()が発火しない
- metaタグが更新されない


#### 最低限抑えること

[Ruby - Turbolinksさんと上手く付き合う10の方法 - Qiita]
(http://qiita.com/seri_k/items/164accd9ef8ddb4a942e)


>（チームメンバーには）最低限以下のことは抑えておいてもらいます。

>- 同一ドメイン内のリンククリックでの移動は全てAjaxでHTMLをロードして差し替える処理にturbolinksのjsが差し替えている
- リンククリックでページ遷移してURLが変わってもwindowオブジェクトやdocumentオブジェクトの状態はそのままになる
- 一旦ランディングしたらPOST/PATCH/DELETEなどで遷移しない限りjsの再ロードは実行されない
- js使いまくってて超複雑なSPAっぽいページがあって、そのページに来たらオフにしたいような場合は対応出来るので分かる人に相談しよう
- 自分でjsを実装したら他のページからリンククリックで遷移してきてもちゃんと動作するか？また他のページに行って往復してきても動作するか必ず確認する


#### 仕組みの解説と効果/問題

[Rails4でturbolinksを謳歌するためのまとめ \[俺の備忘録\]](http://o.inchiki.jp/obbr/97)

>#### 2.turbolinksとは何なのか？
HTML5のpushStateとajaxを組み合わせてページを高速でレンダリングするための仕組み。
turbolinksを導入するとリンクをクリックした時にGETで取得する全てのページがajaxで処理されるようになる。
（POST、PUT、DELETEの場合はturbolinksを導入していない場合と同じように処理される。）

>具体的にはページ遷移が発生する時に（これはaタグがクリックされたタイミング）、
htmlのtitleタグの中身とbodyタグの中身をajaxで入れ替え、
ブラウザの現在のURLを遷移先のURLに、戻り先のURLを遷移前のURLに更新する。

>これによってページ遷移の際にheadタグの中に書かれたJavaScriptとStyleSheetの
パースが行われなくなるので、特にJavaScriptを多用しているサイトではかなりの
高速化になるんじゃないかと。

>ただこれは、JavaScriptとStyleSheetがアセットコンパイルされて
どのページでも常に同じファイルを読み込むようになっているという前提なので、
ページごとに異なるJavaScriptやStyleSheetを利用するようになっている場合は、
そもそもturbolinksの導入には向いていない。
- 中略 -
>turbolinksを導入した時にどのくらい速くなるのかは、
このサイトもturbolinksでページ遷移をしているので、
このサイト内で何度かページを遷移させてみて下さい。

>#### 3.何が問題になるのか？

>今のところturbolinksを導入しても、
そのままの状態だとほとんどの場合不都合が多いと思う。

>問題になったのはいずれも他のJavaScriptとの共存に関する問題で次の通り。

>- window.onload が発火しない
- scriptタグのsrcがキックされない問題

>これで何が問題になるのかというと、

>#### window.onload()が読み込まれない
ajaxでページ遷移を行った場合、 window.onload が発火しないのはもちろん、
jQueryを使っている場合、$(document).ready() に定義した項目も発火しない。
#### scriptタグのsrcがキックされない問題
ツイートボタンやいいね！ボタンなど外部のサイトから提供されたコードを
scriptタグで貼り付けたりしている場合、それが動かない。
