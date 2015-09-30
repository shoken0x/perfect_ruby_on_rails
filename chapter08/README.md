# 8章 Railsのインフラと運用

担当: @syokenz

## Summary

- DevOpsの背景、最近の兆候をキャッチアップする
- VagrantでローカルにVMを起動
- Chefでプロビジョニング入門
- New RelicなどのSaaSを把握する


## 8-1. はじめに DevOpsとは何か？ p.275

Development EngineerとOperations Engineerが歩み寄る

- 開発運用のあらゆる場面での自動化の進歩
- インフラのコード化(Infrastructure as Code)
- クラウドの発展とコモディティ化(AWS, Heroku, New Relic)


## 8-2. VagrantでローカルにVMを作る p.277

### VagrantとVirtualBoxのインストール

公式サイトからダウンロードしてインストール

### Vagrant経由でVMを立ち上げる

Ubuntu 14.04 LTSを立ち上げる  
=> 14.04だとRubyのinstallで失敗するので、12.04をinstallする  
see: [agrant+Chef Soloでのサーバー環境構築自動化を試してみる(参考: パーフェクトRuby on Rails 第8章「Railsのインフラと運用」) - south37のブログ](http://goo.gl/J298CO)


box: http://www.vagrantbox.es

```
# 14.04だとrubyのinstallで失敗
# vagrant box add rails https://oss-binaries.phusionpassenger.com/vagrant/boxes/latest/ubuntu-14.04-amd64-vbox.box

vagrant box add opscode-ubuntu-12.04 \
    http://opscode-vm-bento.s3.amazonaws.com/vagrant/virtualbox/opscode_ubuntu-12.04_chef-provisionerless.box
```

boxの確認

```
vagrant box list
```

Vagrantfileのひな形の作成

```
vagrant init .
```

Vagrantfileのconfig.vm.boxを修正する

```
config.vm.box = "rails"
```

起動

```
vagrant up
```

sshで接続

```
vagrant ssh
```

### VMの設定変更

Vagrantfileの変更

```
config.vm.network :forwarded_port, guest: 80, host: 8080
もしくは
config.vm.network :private_network, ip: "192.168.33.199"
```

ssh forward-agent ホストマシンのssh-agentをログイン先のVMでも利用できるようにする機能

参考: [ssh-agentのforwardを利用し、ホストマシンとローカルVMの非公開鍵を共有する - MANA-DOT](http://blog.manaten.net/entry/ssh-agent-forward)


```
config.ssh.forward_agent = true
```

ホスト/ゲストでディレクトリの共有

```
config.vm.synced_folder "host_dir", "guest_dir"
```

VMのリロード

```
vagrant reload
```

### Chefを動かしてみる

Gemfileの作成

```
bundle init
```

Gemfile

```
source 'https://rubygems.org'

gem "chef", "11.12.4"
gem "berkshelf", "3.1.2"
```

Gemfile作成後、bundle install

```
bundle install
```

Vagrantfileと同じディレクトリにBerksfileというファイルを作成。gitをインストールするように記述する。

Berksfile

```
source "https://api.berkshelf.com"

cookbook 'git'
```

cookbooks以下にダウンロード

```
bundle exec berks vendor cookbooks
```

### ChefのレシピをVagrant上のVMに適用する

Vagrantfileに追記

```
config.vm.provision :chef_solo do |chef|
  chef.cookbooks_path = ["./cookbooks"]
  chef.add_recipe 'git'
end
```

設定を反映して、プロビジョニングを実行

```
vagrant reload
vagrant provision
```

vagrant sshで接続して、gitのバージョンを確認する

## 8-3. Chefを用いた本格的なサーバ構成管理 p.284

### Chefの基本的な概念について

- ノード
- ノード属性(Attribute)
- ロール
- クックブック
- レシピ
- リソース

### クックブックのひな形を作る

Gemfileにknife-soloを追加しbundle install  
knifeコマンドでひな形を作成する  

```
bundle exec knife solo init rails_book_cookbook
```

BerksfileとVagrantfileをプロジェクトのルートにコピーする

```
cp Berksfile rails_book_cookbook
cp Vagrantfile rails_book_cookbook
```

Berksfileへ追記

```
source "https://api.berkshelf.com"

cookbook 'git'
cookbook 'memcached'
cookbook 'nodejs'
cookbook 'database'
cookbook 'xml'
cookbook 'ruby_build'
cookbook 'rbenv', git: 'git://github.com/fnichol/chef-rbenv.git', ref: 'v0.7.2'
cookbook 'nginx'
cookbook 'imagemagick'
```

Vagrantfileへ追記

```
config.vm.provision :chef_solo do |chef|
  chef.cookbooks_path = ["./cookbooks", "./site-cookbooks"]
  chef.add_recipe 'build-essential'
  chef.add_recipe 'git'
  chef.add_recipe 'memcached'
  chef.add_recipe 'nodejs'
  chef.add_recipe 'database'
  chef.add_recipe 'xml'
  chef.add_recipe 'ruby_build'
  chef.add_recipe 'rbenv::system'
  chef.add_recipe 'nginx'
  chef.add_recipe 'imagemagick'

  chef.json = {
    "rbenv" => {
      "global" => "2.1.2",
      "rubies" => [ "2.1.2" ],
      "gems" => {
        "2.1.2" => [
          { 'name' => 'bundler' }
        ]
      }
    }
  }
end
```

プロビジョニング実行

```
cd rails_book_cookbook
rm -rf cookbooks
bundle exec berks vendor cookbooks

vagrant reload
vagrant provision
```

### カスタムレシピを作る


## 8-4. デプロイをする p.302

- 作業が簡単になる
- 作業のミスが減る
- 作業が再現可能になる

Capistranoを使った自動デプロイ

### 必要な作業

- SSH鍵の作成
- staging environmentの追加

**githubのmasterにコミット（マージ）したらstaging環境へ自動でデプロイ、などということができる**

フックポイントが多いのも特徴。柔軟に処理を追加できる。

## 8-5. New Relicによるアプリケーション監視 p.313

- New Relicはアプリケーション監視ツールのSaaSである
- 監視ツールはサービスの死活やパフォーマンスを継続的に監視・記録し、可視化する
- 従来はnagios, zabbix, muninなどといったプロダクトをインストールしていた
- 時代はNew Relic

### New Relicでできること

- 死活監視
- パフォーマンス監視(レスポンスタイム Apdex)
- エラー監視(Error rate)

### 他サービスの紹介

- Airbrake: エラー通知と解析に特化したSaaS
- Fluentd/kibana: ログ収集とビジュアライズ


## 8-6. serverspecとインフラのテスト p.329

serverspecとは、rspec風にインフラのテストが書けるようになるgem

パッケージがインストールされているか？ 80ポートがリッスンされているか? などをテストできる。


## 8-7. 終わりに p.330

- あらゆる箇所で自動化、「秘伝のタレ」の形式知化をすすめること
- プログラミングで解決できる課題を、職種の壁を乗り越えて解決すること


## 補足: Immutable InfrastructureとDocker

### Immutable Infrastructure

Blue-Green DeploymentからのImmutable Infrastructure

[「Blue-Green Deployment」とは何か、マーチン・ファウラー氏の解説 － Publickey](http://www.publickey1.jp/blog/14/blue-green_deployment.html)

[Immutable Infrastructureはアプリケーションのアーキテクチャを変えていく、伊藤直也氏（前編） － Publickey](http://www.publickey1.jp/blog/14/immutable_infrastructure_1.html)

[Vagrant+Chef Soloでのサーバー環境構築自動化を試してみる(参考: パーフェクトRuby on Rails 第8章「Railsのインフラと運用」) - south37のブログ](http://goo.gl/J298CO)

### Docker

ESX, Xen, KVM, bhyve, xhyve そして、LXCからCoreOSとDockerへ

- [最近の仮想化界隈を知る：VMWareからCoreOSまで](http://tkng.org/b/2013/11/17/vm-and-container/)
- [Docker と LXC - Qiita](http://qiita.com/Surgo/items/709a07d68c6eafbad267)
- [Docker 誕生から現在までの過程を俯瞰する 1 (2013/3～2014/10) - Qiita](http://qiita.com/Arturias/items/c12441c87f4de3102ea7)

