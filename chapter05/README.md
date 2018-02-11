# 5章 開発を効率化するgem
担当: @hiroki-tkg


## Summary
* Pry：コンソール
* Hirb：コンソール上のモデルの出力を整形
* Better Errors：エラー画面をリッチに
* Spring：コマンドを高速化
* rails-erd：ER図を生成


### Pry
irb環境で、対話形式でrails consoleをより便利に

*導入手順*
*Gemfile*

	gem 'pry-rails', group: [:development, :test]


`bundle install` をすると、rails c でirbではなく、pryが起動する。

* show-models 全てのモデルの情報を出力
* show-model モデル名 モデルの情報を出力
* show-routes ルーティングを出力

などなど、
pryから様々な情報を見れる。

![Hirb on pry](http://hiroki-tkg.com/wp-content/uploads/pry.png)


※ブレークポイント※
ファイル内に、`binding.pry`と入力しておくと、この行が実行された時にプログラムが中断される。

pry-byebugをインストールするとステップ実行のnextやstep等が利用できる

`binding.pry` をファイル内に記述すると、その時点で処理がストップ  
ステップ実行ができる。


### Hirb
Hirbでコンソール上のモデルの出力を整形

*導入手順*
*Gemfile*

	gem 'hirb', group: [:development, :test]
	gem 'hirb-unicode', group: [:development, :test]


`bundle install`

`Hirb.enable`

で、データを表形式で表示させることが出来る。

※ pryでHirbを利用したい場合  
プロジェクト直下に.pryrcを作成

*.pryrc*
	begin                                                                                                                                                     	require 'hirb'
	rescue LoadError
	end

	if defined? Hirb
		Hirb::View.instance_eval do
		def enable_output_method
			@output_method = true
			@old_print = Pry.config.print
			Pry.config.print = proc do |output, value|
				Hirb::View.view_or_page_output(value) || @old_print.call(output, value)
			end
		end

		def disable_output_method
			Pry.config.print = @old_print
				@output_method = nil     
			end
		end

		Hirb.enable
	end                          


pry上でもHirbを利用できる

![Hirb on pry](http://hiroki-tkg.com/wp-content/uploads/hirb.png)



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

参考  
[http://qiita.com/kanpou_/items/74eca1846101e6db3387](http://qiita.com/kanpou_/items/74eca1846101e6db3387)
