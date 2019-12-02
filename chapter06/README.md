# 6章 Railsアプリケーションの開発

担当: @shoken0x

## Summary

- イベントの登録・削除ができるアプリケーションの作成を通して、Railsでできることと典型的な処理を学ぶ
- ログイン関連、多対多テーブル、バリデーション、関連モデルの削除、コールバックなどは汎用的に使える
- 著者が公開しているソースコード [https://github.com/willnet/awesome_events](https://github.com/willnet/awesome_events)

## 6-1. イベント告知アプリケーションを作る p.160

### 主な仕様

- ユーザは作成されたイベント情報を閲覧することができる
- ユーザはtwitterアカウントでログインできる
- ログインしたユーザはイベントを作成できる
- ユーザは自分で作成したイベントを削除できる
- ログインしたユーザはイベントに参加できる
- ログインしたユーザは退会できる
- 退会したユーザの情報は削除される
- 退会したユーザが作成したイベントはそのまま残る
- イベントを作成したユーザが退会している場合、作成したユーザの情報として「退会したユーザです」と表示される
- イベントに参加したユーザが退会している場合、参加したユーザの情報として「退会したユーザです」と表示される

詳細な仕様とルーティングの仕様はp.160を参照


## 6-2. アプリケーションの作成と下準備 p.161

### 概要

- scaffoldを使ってアプリの下地を作る
- Bootstrapを導入する


### rails newでアプリケーションの作成

```
rails _4.1.1_ new awesome_events -T
```

-TはTest::Unit用のファイルを生成しないオプション。7章でRSpecを使用するため。

起動確認

```
cd awesome_events
rails s
```

### TOPページ表示

```
rails g controller welcome index
```

config/routes.rbを編集。`root to: 'welcome#index'`を追加。


### Bootstrapを導入

```
curl -o /tmp/bootstrap.zip -L https://github.com/twbs/bootstrap/releases/download/v3.1.1/bootstrap-3.1.1-dist.zip
unzip /tmp/bootstrap.zip -d /tmp
cp /tmp/bootstrap-3.1.1-dist/css/bootstrap.min.css vendor/assets/stylesheets/
cp /tmp/bootstrap-3.1.1-dist/js/bootstrap.min.js vendor/assets/javascripts/
```

Asset Pipelineのリストに含めるために`app/assets/javascripts/application.js`と`app/assets/stylesheets/application.css`に配置したファイル名を追記

`app/views/layouts/application.html.erb`を編集して、metaタグとheaderタグを追加。


## 6-3. OAuthを利用して「Twitterでログイン」機能を作る p.168

### 概要

- OAuthでTwitterでログイン
- config/secrets.yml
- モデルの作成、制約とindexの追加、migrate
- controllerとviewで使えるメソッドをapplication_controller.rbに定義する

### Twitterアプリケーションの登録

[https://apps.twitter.com](https://apps.twitter.com)


|key|value|
|---|---|
|name|awecome events|
|description|web application for awesome events|
|Website|http://lvm.me:3000/|
|Callback URL|http://lvm.me:3000/auth/twitter/callback|

### OmniAuthのインストールと設定


```
gem 'omniauth', '~> 1.2.1'
gem 'omniauth-twitter', '~> 1.0.1'
```

```
bundle install
```

`config/secrets.yml`にapi_keyとapi_secretを登録。

`config/initializers/omniauth.rb`を作成し、api_key/api_secretを取り出す。


### ユーザモデルの作成

```
rails g model user provider:string uid:string nickname:string image_url:string
```

migrationファイルにnot null制約とユニークインデックスを追加。

追加後にmigrate

```
rake db:migrate
```

### ログイン処理を作成する

ヘッダー右上に「Twitterでログイン」リンクを追加。Bootstrapの機能を使って、ブラウザの画面が小さい時に、リンク部分をボタンで表示/非表示と切り替えられるようにしておく。


`app/views/layouts/application.html.erb`


ログイン兼ユーザ登録用のコントローラを作成する

`rails g controller sessions`

`app/models/user.rb`に`find_or_create_from_auth_hash`を実装

`config/routes.rb`にログイン用のルーティングを追加

```
get '/auth/:provier/callback' => 'sessions#create'
```

### ログアウト処理を作成する

`app/controllers/sessions_controller.rb`にdestroyアクションを追加。

`app/views/layouts/application.html.erb`でログイン/ログアウトを出し分ける

`app/controllers/application_controller.rb`に`logged_in?`を作成

`config/routes.rb`にlogout用のルーティングを追加

```
get '/logout' => 'session#destroy', as: :logout
```


## 6-4. イベントの登録機能を作る p.181

### 概要

- タイムゾーン設定
- scaffoldでrecourceの追加、routes.rbでrecource
- モデルのバリデーション
- モデル間の関連、適切な名前
- `current_user`, `before_action :authenticate`などログイン状態の管理に必要な実装
- i18nでエラーメッセージを日本語化


### タイムゾーンを設定する

`config/application.rb`でコメントアウトされている`config.time_zone`のコメントを外し、設定をTokyoに変更する。


### イベント用のモデルを作成する

controllerも同時に作るので`rails g resource`コマンドを使用する。

```
rails g resource event owner_id:integer name:string place:string start_time:datetime end_time:datetime content:text
```
マイグレーション用ファイルにnot null制約とインデックスを追加後、maigrate

```
rake db:migrate
```

`app/models/event.rb`にバリデーションを追加

ヘッダーに「イベントを作る」リンクを追加し、イベント登録フォームへ遷移できるようにする。

`app/views/layouts/application.rb`

`config/routes.rb`を確認。`resources :events`が追加されている。


### ログイン状態を管理する処理を作る

`current_user`, `authenticate`を定義

`app/controllers/application_controller.rb`

`authenticate`を`events_controller.rb`のbefore_actionに追加

`app/models/user.rb`にeventsとの関連を定義

### イベント登録用のフォームを作る

`app/views/events/new.html.erb`を作成


### i18nの設定をする

`config/application.rb`

```
config.i18n.default_locale = :ja
```

辞書データをダウンロードする

```
curl -o config/locales/ja.yml -L https://raw.github.com/svenfuchs/rails-i18n/master/rails/locale/ja.yml
```

`config/locales/ar_ja.yml`を作成


## 6-5. イベントの閲覧機能を作る p.192

### 概要

- ログインしていなくても参照できるページの作成 `before_action xxx, except: :xxx`

### イベント詳細ページの作成

`events_controller.rb`にshowアクションを追加

before_actionに`except: :show`を追加

`app/views/events/show.html.erb`を作成

EventモデルにUserモデルの関連を追加

### イベント一覧ページの作成

`app/views/welcome/index.html.erb`を修正

`app/controllers/welcome_controller.rb`を修正。indexアクションでEventを取得する。

### コラム: 名前重要

- ユーザが作成したイベントの関連名`created_events`
- イベントを作成したユーザの関連名`owner`
- どんな名前が作るものの実体を表しているかを考える

## 6-6. イベントの編集・削除機能を作る p.196

### 概要

- 自分で作成したイベントなら編集, 削除できる機能
- 確認用ダイアログ

### イベント編集機能を作る

`app/views/events/show.html.erb`に編集ボタンを追加

`app/models/event.rb`に`created_by?`を追加

### イベント編集用アクションとビューの追加

`app/controllers/events_controller.rb`にeditアクションとupdateアクションを追加

`app/views/events/edit.html.erb`を作成

### イベント削除機能を作る

`app/views/events/show.html.erb`に「イベントを削除する」リンクを追加。ボタンを押したら確認用のダイアログを表示させる。

`app/controllers/events_controller.rb`にdestroyアクションを追加。


## 6-7. 登録されたイベントへの参加機能、参加キャンセル機能を作る p.203

### 概要

- UserとEventの多対多の中間テーブル`tickets`の作成
- 複合インデックス
- DRYを実現するためのヘルパーメソッド
- N+1問題
- 親レコードが削除された時に子レコードも同時に削除する`dependent: :destroy`

### イベント参加用モデルとコントローラの作成

```
rails g model ticket user:references event:references comment:string
```

not null制約とユニークインデックスを追加してmigrate

```
add_index :tickets, [:user_id, :event_id], unique: true
add_index :tickets, [:event_id, :user_id], unique: true
```

>複合インデックスは「キーとして構成されているカラムの全てが検索条件に指定されていなくても、キーの先頭から途中までのカラムが指定されていれば、インデックスが使われる」という 特徴をもっています。

[Rails 複合インデックスの貼り方 | OpenBook](http://openbook4.me/sections/621)


```
rake db:migrate
```

モデルの各関連とcommentのバリデーションを記述

`app/models/ticket.rb`  
`app/models/user.rb`  
`app/models/event/rb`  

コントローラの作成

```
rails g controller tickets
```

`app/assets/javascripts/tickets.js.coffee`に送信ボタンを押した後、レスポンスが返ってきた時の処理をjsで記述。

### 参加者一覧表示機能を作る

ヘルパーに`url_for_twitter`メソッドを実装


### コラム: N+1問題について p.212

`app/controllers/events_controller.rb`

```
@tickets = @event.tickets.includes(:user).order(:created_at)
```

[Rails - ActiveRecordのjoinsとpreloadとincludesとeager_loadの違い - Qiita](http://qiita.com/k0kubun/items/80c5a5494f53bb88dc58)

[» Railsライブラリ紹介: N+1問題を検出する「bullet」 TECHSCORE BLOG](http://www.techscore.com/blog/2012/12/25/rails%E3%83%A9%E3%82%A4%E3%83%96%E3%83%A9%E3%83%AA%E7%B4%B9%E4%BB%8B-n1%E5%95%8F%E9%A1%8C%E3%82%92%E6%A4%9C%E5%87%BA%E3%81%99%E3%82%8B%E3%80%8Cbullet%E3%80%8D/)

### イベント参加をキャンセルする機能

キャンセルボタンを`app/views/events/show.html.erb`に追加

キャンセル処理を`app/controllers/events_controller.rb`,`app/controllers/tickets_controller.rb`に追加

親レコードが削除された時に子レコードも同時に削除する`dependent: :destroy`オプションを`app/models/event.rb`に追加



## 6-8. 退会機能を作る p.215

### 概要

- Railsのデフォルト以外のアクションを追加
- 削除時に実行されるコールバック`before_destroy`
- 退会したユーザの表示処理

### 退会用コントローラ、ビュー、ルーティングの作成

```
rails g controller users
```

単数形リソースとしてuserを設定し、その配下にretireアクション用のルーティングを追加する。

### 退会処理の作成

UserControllerにretireアクション、destroyアクションを追加

Userモデルに削除時に実行されるコールバックを追加 `before_destroy`

ユーザ削除に対応するように関連を修正 `check_all_events_finished`用に`participating_events`という関連を新しく定義。

`dependent: :nullify`オプションで削除時に関連レコードの外部キーをnullに。

ユーザがいなかったら「退会したユーザです」と表示するように`app/views/events/show.html.erb`を修正

## 6-9. 落穂ひろい p.222

### 概要

- エラー画面の表示
- Production環境のエラー通知方法

### エラーハンドリング

`app/controllers/application_controller.rb`に404と500をハンドリングする処理を追加

ビューを追加

- app/views/application/error404.html.erb
- app/views/application/error500.html.erb

`config/routes.rb`の最後にすべてのURLをキャッチする設定を追加する。

```
match '*path' => 'application#error404', via: :all
```

`config/routes.rb`でresourcesやresourceに、onlyオプションやexceptオプションを利用して不要なルーティングを除外することで、作成していないアクションを実行しようとして500エラーになることを防ぐ。


### コラム: エラーの通知方法 p.224

New Relicおすすめ

## 6-10. gemで機能拡張をする p.224

### 概要

- Kaminariでページネーション機能を作る
- ransackでイベント検索機能を作る
- carrierwaveで画像を添付する

### Kaminariでページネーション機能を作る


### ransackでイベント検索機能を作る


### carrierwaveで画像を添付する


#### アップロードにおけるその他の注意点

- アップロードしたファイルをどこに配置するか
  -  carrierwaveのデフォルトではファイルに保存するが、設定でS3などのクラウドストレージにも保存できる。
- サイズが大きい画像をアップロードした場合にどうするか
  - 容量問題: アプリケーションサーバー、もしくはWebサーバー(Nginxなど)でXMB以上の画像をアップロードできないようにする、などの制限を設定する
  - 縦横サイズ問題: ImageMagickとの連携で、アップロード時に一定の大きさに縮小して保存する。
- 画像以外のファイルをアップロードした場合はどうするか
  - carrierwaveのアップロードするファイルの拡張子をチェックする機能を利用する。


## まとめ

p.239
> Railsはとても多くの機能を提供しています。その機能を学ぶ際には、単純に丸暗記するのではなく「なぜこのような機能があるのか」を積極的に調べるようにしましょう。Railsが提供している機能の多くは、Webアプリケーション開発全般で使えるベストプラクティスです。
