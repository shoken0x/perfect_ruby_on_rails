# 7章 Railsアプリケーションのテスト

担当: @hiroki_tkg

## 7-1. なぜテストコードを書くのか？

- テスト対象となる仕様とアプリケーションの設計を考える機会が増える
- 手作業によるテストを減らすことが出来る
- 想定通り動くのか...という不安を払拭できる
- リファクタリングや仕様変更に自信を持って対応できる

ザックリ言うと...  
テストコードを書いておくと、そのコードがちゃんと動くか簡単に分かるし、何かと安心。

## 7-2. minitest と RSpec

Rubyのテストで有名な2つのフレームワーク

### minitest
- Ruby1.9からRubyの標準ライブラリに
- Railsでデフォルトではminitestを利用
- xUnit風、RSpec風、2パターンの書き方ができる
- RailsではデフォルトでxUnit風の書き方のみサポートしている
- minitest-railsというgemを導入する必要がある

### RSpec
- Railsのデファクトスタンダード

### minitest と RSpecの比較
#### minitest  
- 最小限
- 約1500行
- 機能の把握にかかる時間は短い


#### RSpec  
- 多機能
- 約1万行
- 短く書ける

普及度を考えると、RSpecを採用するのがベター(?)

## 7-3. テストを実行するための環境を整える

#### RSpecのインストール
Gemfile

```
group :development, :test do
  gem 'rspec-rails','~> 3.0.0.beta2', github: 'rspec/rspec-rails'
  gem 'rspec-core','~> 3.0.0.beta2', github: 'rspec/rspec-core'
  gem 'rspec-expectations','~> 3.0.0.beta2', github: 'rspec/rspec-expectations'
  gem 'rspec-mocks','~> 3.0.0.beta2', github: 'rspec/rspec-mocks'
  gem 'rspec-support','~> 3.0.0.beta2', github: 'rspec/rspec-support'
end
```
```
./bin/bundle
```

RSpec用の設定ファイルを作成

```
./bin/rails g rspec:install
```
この時作られる、spec_helper.rbはRSpecを使ったテストで使う設定ファイルなので、全てのテストコードのファイルの行頭に

```
require 'spec_helper'
```
を記入する。


## 7-4. モデルのテスト

```
./bin/rails g rspec:model event
```
上記のコードで、spec/models/event_spec.rb が作成される。

spec/models/event_spec.rb

```

```

describe => オブジェクト、メソッド
context  => 条件
it       => 何を期待しているかを文字列で

実際の処理は、ブロック中に記述

※ 階層構造にすることで、見通しがよくなる

expectの引数で指定されたオブジェクトが「どのような状態であるべきか」を指定するもの => マッチャ


shouldaを使うと、簡潔に書ける

fixture replacementの中で良く利用される factory_girl を使ってみる
factory_girl_railsをインストールした状態で、モデルを生成したり、テストファイルを作った場合、同時にfactory_girl用のファイルも生成される








