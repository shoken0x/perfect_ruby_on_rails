# 1章 Ruby on Railsの概要

担当: @syokenz

## Summary

- gem, rakeコマンドの使い方
- rake db:migrateの使い方
- rails new, scaffoldで作成されたファイルの確認
- ルーティングを理解する

## 1-1 Railsを使う前に

### Railsの環境構築について Win/Mac/Linuc

すでに環境構築済みなのでスキップ

### GemコマンドとRakeコマンド

gem: パッケージ管理システムであるRubyGemsを扱うためのコマンド

https://rubygems.org

よく使うGemコマンド

```
gem seach {gemパッケージ名}
gem install {gemパッケージ名} # gem i でも可
gem uninstall {gemパッケージ名}
```

オプション

```
--no-ri --no-rdoc  #riとrdocをインストールしない
```

rake: Rakeと呼ばれるビルドツールを実行するためのコマンド

```
rake -T #タスクを確認
```

rails開発でよく使うrakeコマンド

```
rake routes
rake spec
rake db:migrate # mongodbだと不要
```

### bundlerでgemパッケージを束ねる

基本的にはgemコマンドでインストールするのはBundlerのみで、その他のgemパッケージはBundler経由でインストールするという方針が良い。

よく使うBundlerのサブコマンド

```
bundle install
bundle update
bundle list
bundle exec
bundle init  #ひな形Gemfileを作成
```

#### 理解のポイント

`bundle install` と `bundle update` の違い

Gemfileでよく使う書き方、バージョンについて

http://xxxcaqui.hatenablog.com/entry/2013/02/11/013421

```
gem "rails", "3.0.0.beta3" # 3.0.0.beta3で固定
gem "rack",  ">=1.0"       # 1.0以降のバージョンに制限
gem "thin",  "~>1.1"       # 1.1以降2.0未満のバージョンに制限。~>1.1.0とすると1.1.0以上1.2.0未満の最新のものを使用
gem 'rails', :git => 'git://github.com/rails/rails.git' # 最新のRailsを使用, branch指定もできる
```

## 1-2 Railsの思想

Railsの4つの思想

1. CoC (Convention over Configuration)
1. DRY (Don't Repeat Yourself)
1. REST (Representational State Transfer)
1. 自動テスト

### CoC

設定より規約

- DBのテーブル名はモデル名の複数形にする
- /employeesというURLはemployeeの一覧を表す
- ID:1のemployee情報を表すURLは/employees/1である


### DRY

同じことを繰り返さない。1つのことは1箇所に。

### REST

- すべてのリソースに一意となる識別子(URI)がある
- URIを通してリソースを操作する手段を提供する

### 自動テスト

自動テストを行う文化を重要視している。


## 1-3 Railsをはじめよう!!

```
gem install rails -v 4.1.1 --no-ri --no-rdoc
```

### プロジェクトのひな形を作成する

```
rails _4.1.1_ new todo # 本書でのバージョンが 4.1.1 なので合わせる
```

#### 理解ポイント

ディレクトリ構成をテキストを見ながら確認する。

よく使われるRailsのサブコマンド

```
rails s  # server起動
rails c  # concole起動
rails db # dbコンソール起動。mongodbでは使えない
```

## 1-4 scaffoldを使ってRailsでの開発を体験しよう

### scaffoldで作成されたディレクトリ/ファイルを確認する。

```
rails g scaffold task content:text
```

以下のディレクトリに注目

- app/controllers/tasks_controller.rb
- app/views/tasks/*.erb
- app/models/task.rb
- db/migrate/*


### migrationファイルを元にDBを作成する

```
rake db:create
rake db:migrate
```

確認

```
sqlite3 db/development.sqlite3 ".schema tasks"
```

### migrationの管理


```
rake db:migrate:status
```


### ルーティングの一元管理とRESTの思想

http://localhost:3000/rails/info/routes

または、`rake routes`


### scaffoldで生成されたアクション

1. index
1. show
1. new
1. edit
1. create(POST)
1. update(PUT/PATCH)
1. destroy(DELETE)

app/controllers/tasks_controller.rbを見る。

#### 理解ポイント

1. Railsで生成されるアクションは覚える
1. `Resource` と `Resources` の違い
1. 暗黙的に描画するビューテンプレートの命名規則
1. インスタンス変数の共有


ルーティングでURLとコントローラ内のメソッドをひも付けることでURLに対するアクションを定義し、そのアクションの中でモデルを通じてデータを取得し、テンプレートファイルを通じてHTMLを生成、描画する。


## 応用編

### ルーティングについての理解を深める

[DHH流のルーティングで得られるメリットと、取り入れる上でのポイント - KitchHike Tech Blog](http://tech.kitchhike.com/entry/2017/03/07/190739) を読んで理解する。
