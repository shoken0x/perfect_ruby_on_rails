# 4章 Railsのロードパスとレイヤーの定義方法
担当: @hiroki-tkg


## Summary
* MVC以外の構成要素を書く場合
* app/libディレクトリに関して
* レイヤーを追加するgem


## 4-1 MVC以外の要素の扱い

MVCに当てはまらないロジックを書く必要がある場合  
(外部のAPIと通信する、非同期処理、データの暗号化,etc)

* モデルに書く
* /libに書く
* 新しいレイヤーを定義する

のどれかに書く
※ 基本的には、モデルに書く  
※ 開発者の考え方や、そのアプリの性質によるので常に正しい回答はない。


### app/lib以下に書く場合
もともとそうゆうライブラリを配置するためのディレクトリが用意されている。

例  
*lib/api_client.rb*

    class ApiClient
    end

と書き、コードを読みこみたいところで、 `require 'api_client'` で呼び出せば、OK


#### libディレクトリ
libディレクトリは、独自定義のRakeタスクを配置しておく場所でもある。

rakeタスクは、`lib/tasks`というディレクトリに置く  
ここに○○.rakeで保存しておくと、rakeコマンドで使える


__オートローディング__

Railsは命名規則に沿ったファイル名とクラス名を採用していると、  
自動でファイルをrequireしてくれる

これをオートローディングという

つまり命名規則に従えば勝手にファイルをロードしてくれる  
ファイル名 :スネークケース  
クラス名 :キャメル  


クラスやモジュールのネームスペースはディレクトリで表現  

デフォでパスが繋がっているのは以下  
* app/controllers  
* app/models  
* app/helpers  
* app/mailers  
* app/controllers/concerns  
* app/models/concerns

※ libディレクトリははいってない

lib以下をオートロードにしたいときは、
`config/application.rb`を編集する

※libには、rakeを置いたり、assetsディレクトリがあったりするので、libを直でロードしない方が良い

*config/application.rb*

	class Application < Rails::Appication
		config.autoload_paths += %W(#{config.root}/lib/autoload)


読むファイル
*lib/autoload/my_library.rb*

    module MyLibrary
	    def self.echo
		    'My Library is loaded.'
	    end
    end


### 新しいレイヤーを配置する場合

新しいレイヤーの配置とオートローディング

非同期処理とか

※ 新しいレイヤーを作りたい時
app/ にサブディレクトリを作成

autoload_pathsに追加

*config/application.rb*

	config.autoload_paths += %W(#{config.root}/app/workers)


`app/workers/image_converter.rb`

モデルの肥大化に対して、有効策になる
詳しくは9章で





## 4-2 レイヤーを追加するgem
* active_decorator  
* Sidekiq

### active_decorator  
*出来ること*
-
複雑なビューを表示するときは、Helperを使うのが一般的だが、単純な関数として定義されるので、あまりスマートな書き方でない。
そこで、モデルから表示を整形するactive_decoratorを使うとよりオブジェクト指向と親和性の高い記述が可能になる。


### Sidekiq
*出来ること*
-
バックグラウンドで非同期処理を行う


参考 : Redisとは
(http://japan.zdnet.com/article/35063104/)
