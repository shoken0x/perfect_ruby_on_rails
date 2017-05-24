# 2章 Ruby on RailsとMVC

担当: @syokenz

## Summary

- MVCの基本
- Model層: ActiveRecordの基本的な操作、バリデーション、コールバック
- Controller層: ModelとViewをつなぐこと、リクエストオブジェクトやアクションコールバック、脆弱生への対処
- View層: 受け取ったModelを表示すること、テンプレートエンジン、ヘルパー、さまざまなフォーマットで表示
- Skinny Controller, Fat Model?

参考
[Life is beautiful: Ruby on Railsの「えせMVC」の弊害](http://satoshi.blogs.com/life/2009/10/rails_mvc.html)


## MVCアーキテクチャ

- M=Model, データベースとの接続とデータに対する操作、およびビジネスロジック
- V=View, Modelの内容を参照し視覚表現を行う部分
- C=Controller, Modelのロジック呼び出し、必要なViewの選択など、ModelとViewをつなぐ部分

![mvc_framework.jpg](https://qiita-image-store.s3.amazonaws.com/0/32750/1c81c4df-0548-70ce-3675-01b719eb34c5.jpeg "mvc_framework.jpg")

```
rails _4.1.1_ new book_admin
```

## モデルの役割

### ActiveRecordでモデルを実装する

```
rails g model book name:string publisherd_on:date price:integer number_of_page:integer
```

作成されたmigrationファイルをDBに反映

```
rake db:create
rake db:migrate
```

ActiveRecordによるモデルには2つの側面がある。

1. データベースと接続し、データベースのレコードとエンティティを結びつけるという役割（ORM)。自動でカラムを取得し、メソッドを作成する機能がある。
2. ビジネスロジックの実装的な振る舞い、すなわちバリデーションやコールバックを実行する役割。内部でActiveModelというモジュールが担っている。


### モデルとCRUD

CRUDに対応したメソッドがある。
create, save, find, all, delete, destroyなど

### ActiveRecord::RelationとQuery Interface

1. findやfind_byではなく、whereで複数のレコードが返ってくる時、返り値はActiveRecord::Relationインスタンスとなる。
2. ActiveRecordに対してQuery Interface(whereなど)が呼ばれると、ActiveRecord::Relationインスタンスが生成される。
3. Query Interfaceは繰り返し呼び出すことができ、ActiveRecord::Relationインスタンスに情報が蓄積し、どんなSQLを発行するかの情報が更新されていく。
4. 実際にデータが必要になった時点でSQLを発行紙、データを取得する。


### scope

よく利用する検索結果に名前をつけてクラスメソッドのように使える。メソッドチェーンが可能。

例

```
scope :costly, -> { where("price > ?", 3000)}
```

### モデル同士のリレーション

1対多 = has_manyとbelongs_to  
1対1 = has_one  
多対多 = has_many, through: 中間テーブル  


### バリデーション

- save時に呼ばれる
- バリデーションに失敗したら、インスタンス名.errorsに情報が入る。インスタンス名.errors.full_messagesで情報を参照できる。
- !を伴うメソッドはバリデーション失敗時には例外が起こる
- 構文その1 `validates カラム名, バリデーション処理のハッシュ`
- 構文その2 `validate ブロック`


### コールバック

任意の処理を埋め込むために多くのフックポイントが用意されている。before_save, after_saveなど。
`:if`でコールバックの条件を設定できる。

```ruby
def high_price?
  price >= 5000
end

after_destroy :if => :high_price? do |book|
  Rials.logger.warn "Please check!!"
end
```

### ActiveRecord enums

宣言

```ruby
class Book < ActiveRecord::Base
  enum status: %w(reservation now_on_sale end_of_print)
  
end
```

- DB側では数値で保存される。数値は1番目から0,1,2...
- 述語メソッドが自動で付与される。例: book.now_on_sale?
- 数値自体を参照したい場合は、book[:status]


## コントローラの役割

ルーティングは`config/routes.rb`で行われる。
1. ルーティングパターンでの記述
2. リソースを使った記述
の2種類の書き方がある。

`app/controllers/application_controller.rb`には共通する処理を記述する。

### ビューの決定

```ruby
respond_to do |format|
  format.html
  format.csv
  format.json
end
```

/books/1.jsonでアクセスされた場合にはjsonフォーマットのレスポンスを返す。

### アクションコールバック

それぞれのアクションにコールバック処理を定義できる。before_action, after_actionなど。ログイン処理で必要になる。

コールバックのスキップも可能。skip_before_actionなど。

### ルーティングとリソース

#### resources

```
resources :publishers
```

上記で生成されるルーティング

1. index
2. create
3. new
4. edit
5. show
6. update
7. destroy

の7つ

resourcesには2種類の拡張ができる。

1. memberリソース(/publishers/:id/detail)
2. collectionリソース(/publishers/search)

```ruby
resources :publishers do
  member do
    get 'detail'
  end
  
  collection do
    get 'search'
  end
end
```

resourcesの入れ子も可能。


#### resource

```
resource :profile
```

1人のユーザーから見て1つしか存在しないようなリソースはresourceで定義する。

index以外のルーティングが定義される。:idを伴わないURLという箇所に注意。

### only, except

onlyやexceptでルーティングの制限が可能

```
resource :profile, only: %i{show edit update}
```

### 例外処理

Railsでは特定の例外を投げるとそれに対応したステータスコードを返す。例: `ActiveRecord::RecordNotFound`では404 Not Foundが返ってくる。

ちなみに、MongoidではRecordNotFoundに相当する例外でDocumentNotFoundがあるが、Railsは404では無く500を返す。


`rescue_from`メソッドで例外を特定の処理にひも付けることが可能。

### StrongPrameters

4からの新機能。Mass Assignment脆弱性を防ぐ機構。  
事前にホワイトリストでMass Assignmentで利用しても良いHashのkeyを定義する。

```ruby
class ProfileController < ApplicationController
  def update
    current_user.update(user_params)
  end

  private
  def user_params
    params.require(:user).permit(:name, :email)
  end
end
```

- リクエストに:userというkeyが必要であること
- userの中で受け付けても良いのは、`:name`,`:email`の2つのkeyのみ

## ビューの役割

### 基本

コントローラのアクション名と一致するテンプレートを表示する。

例:

```ruby
class BooksController
  def index
    @books = Book.all
  end
end
```

上記の場合、app/views/books/index.html.erbを表示する。


### render

renderでアクションを指定することが可能。

```
render :show # またはrender action: 'show'
```

### respond_to

respond_toでコンテンツタイプを出し分ける。
このような書き方が可能。

```ruby
  def index
    @entries = Entry.all

    respond_to do |format|
      format.html
      format.json { render json: @entries.to_json, status: 200 and return }
    end
  end
```

### partialテンプレートとlayout

#### partial

テンプレートファイル内で、

```
<%= render 'form' # またはrender partial: 'form'%> 
```

と記述すると、`_form.html.erb`を表示する。

#### layout

メタタグやヘッダー・フッターは`app/views/layouts/`にレイアウト用テンプレートを配置する。デフォルトは`app/views/layouts/application.html.erb`。layoutをrenderメソッドのオプションで指定可能。

```
render :show, layout: 'awesome_book'
```

### variantsによるテンプレートの切り替え

controller内で`request.variant`に`:mobile`などの値を入力することで、`index.html+mobile.erb`というテンプレートに切り替えることが可能。

### テンプレートエンジンの紹介

ERB, Haml, Slimの紹介。省略。

### ヘルパーの利用

- url_for
- form_tag/form_for
- stylesheet_link_tag/javascript_include_tag

独自ヘルパーは`app/helpers/`にファイルを作成する。共通のヘルパーは`app/helpers/application_helper.rb`に定義するのが慣習。


### エスケープ処理

`raw`ヘルパー、もしくは`Stiring#html_safe`メソッドを呼ぶとエスケープが解除される。

### APIサーバにとってのビューについて

jbuilderでjsonを定義できる。jbuilderはRails4から標準に取り込まれた。

例: `app/views/books/show.json.jbuilder`

```
json.extract! @book, :id, :name, :price, :created_at
```

メモ: Rails5から提供されるrails-apiではviewを使わないのでjbuilderも使わない。
