# 5章 開発を効率化するgem
担当: @hiroki-tkg


## Summary
* Pry：コンソール
* Hirb：コンソール上のモデルの出力を整形
* Better Errors：エラー画面をリッチに
* Spring：コマンドを高速化
* rails-erd：ER図を生成


### Pry
*pryで出来ること*  
irb環境で対話形式で色々チェックできる

・DBの構造チェック
・
などなど

◆ブレークポイント
binding.pryと入力しておくと、この行が実行された時にプログラムが中断される

pry, pry-railsだけでは、オブジェクトの参照は出来るが、ステップ実行が出来ない

→　byebug

pry-byebugをインストールするとステップ実行のnextやstep等が利用できる



### Hirb
Hirbでコンソール上のモデルの出力を整形


### Better Errors
エラー画面をリッチに表現

*導入手順*
*Gemfile*
	
	gem 'better_errors', group: [:development, :test]
	gem 'binding_of_caller', group: [:development, :test]


`bundle install`

![エラー画面](http://hiroki-tkg.com/wp-content/uploads/better_errors.png)

エラーが起きた時に、ブラウザ画面上で、rails consoleの様に作業できる

![エラー画面②](http://hiroki-tkg.com/wp-content/uploads/better_errors_2.png)

また例外が発生した時点での変数の値も見れる


### Spring
はやくなる


### rails-erd
モデルのER図を簡単に生成できる

*導入手順*

	brew install graphviz

*Gemfile*
	
	gem 'rails-erd', group: [:development, :test]

コマンド `rake erd` をすると、app/ にpdfファイルが出力される

![ER図](http://hiroki-tkg.com/wp-content/uploads/er.png)

※ オプション等を使えば、プライマリーキーやデータ型なども出すことが出来る

![ER図②](http://hiroki-tkg.com/wp-content/uploads/er_2.png)


	

