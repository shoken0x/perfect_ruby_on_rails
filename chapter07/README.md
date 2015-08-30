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

著者ソースコード: [https://github.com/willnet/awesome_events](https://github.com/willnet/awesome_events)

```
git clone git@github.com:willnet/awesome_events.git
```


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
require 'spec_helper'

describe Event do
  describe '#name' do
    context '空白のとき' do
      it 'validでないこと' do
        event = Event.new(name: '')
        event.valid?
        expect(event.errors[:name]).to be_present
      end
    end
  end
end
```

describeの引数 => オブジェクト、メソッド名
contextの引数  => 条件
itの引数       => 何を期待しているかを文字列で

実際の処理は、ブロック中に記述

※ 階層構造にすることで、見通しがよくなる

expectの引数で指定されたオブジェクトが「どのような状態であるべきか」を指定するもの => マッチャ

```
./bin/bundle exec rspec spec/models/event_spec.rb
```

##### shoulda というgemを使うと、簡潔に書ける

```
require 'spec_helper'

describe Event do
  describe '#name' do
    it { should validate_presene_of(:name) }
    it { should ensure_length_of(:name).is_at_most(50) }
  end
end
```

### fixture replacement

今回はよく利用される factory_girl を使ってみる

```
group :development, :test do
	gem 'factory_girl_rails', '~> 4.4.1'
end
```

```
./bin/bundle install
```
spec/factories/events.rb を編集

```
FactoryGirl.define do
  factory :event do
  	owner
  	sequence(:name){ |i| "イベント名#{i}"}
  	sequence(:place){ |i| "イベント開催場所#{i}"}
  	sequence(:content){ |i| "イベント本文#{i}"}
  	start_time { rand(1..30).days.from_now }
  	end_time{start_time + rand(1..30).hours}
  end
end
```

spec/factories/users.rb

```
FactoryGirl.define do
  factory :user, aliases: [:owner] do
    provider 'twitter'
    sequence(:uid) { |i| "uid#{i}" }
    sequence(:nickname) { |i| "nickname#{i}" }
    sequence(:image_url) { |i| "http://example.com/image#{i}.jpg"}
  end
end
```
Rails consoleで確認してみる

```
./bin/rails c
FactoryGirl.create(:event)
```


factory_girl_railsをインストールした状態で、モデルを生成したり、テストファイルを作った場合、同時にfactory_girl用のファイルも生成される





## 7-5 コントローラのテスト

- アクションがビューに渡すインスタンス変数に期待通りのオブジェクトを代入してるか
- アクションが期待通りのステータスコードを返しているか
- アクションが期待通りのテンプレートを選択しているか

```
rails g rspec:controller events
```

spec/controllers/events_controller_spec.rb


```
require 'spec_helper'

describe EventsController do
	describe 'GET #new' do
		context '未ログインユーザーがアクセスした時' do
			before { get :new }

			it 'トップページにリダイレクトすること' do
				expect(response).to redirect_to(root_path)
			end
		end

		context 'ログインユーザーがアクセスした時' do
			before do
				user = create :user
				session[:user_id] = user.id
				get :new
			end

			it 'ステータスコードとして200が返ること' do
				expect(response.status).to eq(200)
			end

			it '@eventに、新規Eventオブジェクトが格納されていること' do
				expect(assigns(:event)).to be_a_new(Event)
			end

			it 'newテンプレートをrenderしていること' do
				expect(response).to render_template :new
			end
		end
	end
end
```

テストの実行は、

```
bundle exec rspec spec/controllers/event_controllers_spec.rb
```

「get :new」で、describeの引数で指定したコントローラのnewアクションにアクセス    
responseメソッド => レスポンス  
assignsメソッド => インスタンス変数

## 7-6 ビューのテスト
コントローラで設定されるインスタンスが特定の条件の時に、期待通りのhtmlを返すかどうかをチェックする

```
rails g rspec:view events show
```

```
require 'spec_helper'

describe "events/show" do
  context '未ログインユーザがアクセスしたとき' do
    before do
      allow(view).to receive(:logged_in?) { false }
      allow(view).to receive(:current_user) { nil }
    end

    context 'かつ @event.owner が nil のとき' do
      before do
        assign(:event, create(:event, owner: nil))
        assign(:tickets, [])
      end

      it '"退会したユーザです"と表示されていること' do
        render
        expect(rendered).to match /退会したユーザです/
      end
    end
  end
end
```
```
bundle exec rspec spec/views/events/show.html.erb_spec.rb
```

## 7-7 エンドツーエンドのテスト
####エンドツーエンドテストとは？
クライアントを操作して、サーバーの入力出力をチェックすること  
例: ブラウザからフォームに文字列を入力し、送信ボタンを押すと、「入力受け付けました！」という画面が表示される ...etc

capybaraが一般的
RSpec単体でも出来るが、ビューやコントローラのテストを統合したような書き方になる


## 7-8 JavaScriptのテスト
エンドツーエンドテスト内で、JavaScriptをテストする
capybaraでJavaScriptをテストするには、対応するドライバーを指定する必要がある。


## 7-9 TDD(テスト駆動開発)の考え方
「まずテストを書き、次に実装を書き、最後にリファクタリングする」を繰り返して、開発していく。

テストコードを書く → テストが通るためのコードを書く → リファクタリング

## 7-10 CI(継続的インテグレーション)
いままでのテストでは、実行するために「rake spec」などのコマンドを実行する必要があったが、コードを頻繁に更新するようになると上記コマンドを実行するのも手間になる。

そこで、
#####コードの変更を検知して、自動でテストコードを実行してくれる。
のが、CIである。

Jenkins  
[https://jenkins-ci.org/](https://jenkins-ci.org/)

Travis CI  
[https://travis-ci.org/](https://travis-ci.org/)

#7-11 カバレッジと静的解析

####カバレッジ  
テストを実行した時に、アプリケーションのコード全体のうち、テストできたコードの割合がどの位かという意味。

#####カバレッジを表示するgem
simplecov  
カバレッジをhtmlで出力してくれる

テストのカバレッジを表示してくれるwebサービスとしては、coverallが有名

#####静的解析

プログラムを実行せずに、プログラムの問題点やバグを調べ分析すること

Railsアプリケーションの脆弱性を静的解析するgem  
brakeman

