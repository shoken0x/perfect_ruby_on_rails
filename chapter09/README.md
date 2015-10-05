##9-4 値オブジェクト  
オブジェクトは「同一性」をどう捉えるかによって2種類に分けられる。

####エンティティ
オブジェクトの同一性が重要な意味を持つもの。
例 : idなど

####値オブジェクト
何であるか。値が同じであればアプリケーション上は同一であると見なして良いオブジェクト。
例 : メールアドレス、住所など


##その文字列は本当に単なる文字列か
ActiveRecordオブジェクトは比較的単純なオブジェクトしか扱えない

String, Fixnum, Time, Date
しかし、それだけでは十分ではない

####単純な文字列による実装の例
```
class User < ActiveRecord::Base
 def same_prefecture?(other)
  prefecture == other.prefecture
 end
 
 def same_city?(other)
  city == other.city
 end
end
```

####値オブジェクトによる実装の例
```
class User < ActiveRecord::Base
 def address
   @address ||= Address.new(prefecture, city, house_number)
 end
 
 def address=(address)
  self.prefecture   = address.prefecture
  self.city         = address.city
  self.house_number = address.house_number
  @address = address
 end
end
```

値オブジェクトであるAddressクラス

```
class Address
 attr_accessor :prefecture, :city, :house_number
 
 def initialize(prefecture = nil, city = nil, house_number = nil)
  @prefecture   = prefecture
  @city         = city
  @house_number = house_number
 end
 
 def hash
  prefecture.hash + city.hash + house_number.hash
 end
 
 def ==(other)
  return false unless other.is_a?(Address)
  
  same_prefecture?(other) && same_city?(other) && same_house_number?(other)
 end
 
 def same_prefecture?(other)
   prefecture == other.prefecture
 end
 
 def same_city?(other)
  city == other.city
 end
 
 def same_house_number?(other)
  house_number == other.house_number
 end
end

```
- 値オブジェクトを使うことで、責任範囲を限定できる。
- また実装を再利用できる

###composed of
オブジェクトの属性値を値オブジェクトにマッピングする機能

```
class User < ActiveRecord::base
  comosed_of :addrss, mapping: [%w(prefecture prefecture), %w(city city), %w(house_number house_number)]
end
```


##9-5 Concern
ある機能を実現するために必要な処理をConcernモジュールとして分離することで、  
モデルやコントローラから特定の機能のためにのみ利用されるメソッドやバリデーションの実装を分離できる。

####横断的関心事  (複数のモデルや機能にまたがって影響するようなもの)

- アプリケーション全体で利用されるロギング機能
- 変更に対する証跡の記録
- タグによる分類機能
- データの論理削除や、有効化・無効化処理
- 他のサービスやバックエンドプロセスと連携するためのシリアライズ

このような横断的関心事を実装する場合に、Concernモジュールとして実装する


#### ActiveSupport::Concern
記述を補助してくれる機能

モジュールに対して
- includedメソッドにブロックを渡す機能を追加。includeされた時にフックして実行する処理を直感的に記述できる
- クラスメソッドを追加するための記述を補助する
- モジュールの依存関係を管理

includeされたときの処理を分かりやすく記述  
簡単にクラスメソッドを追加する  
モジュール同士の依存関係を解決  


##9-6 サービスクラス

