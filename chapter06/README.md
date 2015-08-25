# 6章 Railsアプリケーションの開発

担当: @syokenz

## Summary

- イベントの登録・削除ができるイベント告知アプリケーションを作る
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

### イベント詳細ページの作成

`events_controller.rb`にshowアクションを追加

before_actionに`except: :show`を追加

`app/views/events/show.html.erb`を作成

EventモデルにUserモデルの関連を追加

### イベント一覧ページの作成

`app/views/welcome/index.html.erb`


## 6-6. イベントの編集・削除機能を作る p.196

## 6-7. 登録されたイベントへの参加機能、参加キャンセル機能を作る p.203

## 6-8. 退会機能を作る p.215

## 6-9. 落穂ひろい p.222

## 6-10. gemで機能拡張をする p.224



p.239
> Railsはとても多くの機能を提供しています。その機能を学ぶ際には、単純に丸暗記するのではなく「なぜこのような機能があるのか」を積極的に調べるようにしましょう。Railsが提供している機能の多くは、Webアプリケーション開発全般で使えるベストプラクティスです。










