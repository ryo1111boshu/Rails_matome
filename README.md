# Rails学習まとめ
- [改訂4版 基礎Ruby on Rails](https://book.impress.co.jp/books/1117101135)
- [現場で使える Ruby on Rails 5速習実践ガイド](https://www.amazon.co.jp/dp/B07JHQ9B5T/ref=dp-kindle-redirect?_encoding=UTF8&btkr=1)
- [パーフェクトRuby on Rails](https://www.amazon.co.jp/%E3%83%91%E3%83%BC%E3%83%95%E3%82%A7%E3%82%AF%E3%83%88Ruby-Rails-%E3%81%99%E3%81%8C%E3%82%8F%E3%82%89%E3%81%BE%E3%81%95%E3%81%AE%E3%82%8A-ebook/dp/B00P0UR1RU/ref=sr_1_1?__mk_ja_JP=%E3%82%AB%E3%82%BF%E3%82%AB%E3%83%8A&keywords=%E3%83%91%E3%83%BC%E3%83%95%E3%82%A7%E3%82%AF%E3%83%88rails&qid=1555740150&s=digital-text&sr=1-1-catcorr)

## 環境構築の手順(2018/11/04)

- Xcodeのインストール(AppStoreから)

- コマンドラインデベロッパーツールのインストール

ターミナルで以下のコマンドを記述

```terminal
$ xcode-select --install
```

- Homebrewのインストール（サイトから）

サイトに載ってるインストール用のコマンドを自分のターミナルで打つ


- rbenvとruby-buildのインストール

ターミナルで以下のコマンドを打つ

```terminal
$ brew install rbenv ruby-build
$ echo 'eval "#(rbenv init -)"' >> ~/.bash_profile
$ source ~/.bash_profile
```

- Rubyのインストール

```terminal
$ rbenv install 2.5.1
$ rbenv global 2.5.1

* 新しいバージョンが出た場合
$ brew upgrade ruby-build
$ rbenv install 2.5.1
$ rbenv global 2.5.2

* rbenv install -l
* rbenv local 2.4.1
```

- Railsのインストール

```terminal
gem install rails --version "5.2.0" -N
* Nオプションはインストール時のドキュメントの生成をキャンセルし、インストールの所要時間を短くする
```

- rails newコマンドでアプリケーションの作成

バージョン指定をしたい場合は以下のようにする

```terminal
rails _5.1.2_ new <アプリ名>
```

- bundle lockコマンドの実行

これを省略するとbundle install時に警告文が出る

```terminal
$ bundle lock --add-platform x64-mingw32 x86-mingw32
```

- bundle installコマンドの実行

## Facebookログイン実装手順

- https://developers.facebook.com/ にアクセス

-  右上のマイアプリから「アプリを追加」を選択

- 新規アプリを作成したら、左にある「設定」から「ベーシック」→「プラットフォームを追加」を選択（サイトURLは開発環境の場合http://localhost:3000）

- railsのgemfileに以下の記述追加

```gemfile
gem 'omniauth'
gem 'omniauth-facebook'
```

- Userモデルにfacebookログイン情報保存用のカラム追加

- config/initializers/devise.rbに以下の記述追加
 
```rb
config.omniauth :facebook, ENV['FACEBOOK_APP_ID'], ENV['FACEBOOK_APP_SECRET'], scope: 'email', info_fields: 'email, name'
# 環境変数は.envで設定済
```

## CarrierWave(gem)使用手順

1.gemをインストール

```
gem 'carrierwave'
gem 'mini_magick'
```

2.bundle実行

3.rails g uploader image をターミナルで実行し、アップローダーを作成

4.app/models/message(仮).rbに以下の記述

```
mount_uploader :image, ImageUploader
```

5.app/uploaders/image_uploader.rbに以下の記述

```
include CarrierWave::MiniMagick
process resize_to_fit: [800,800]
```

※5番目までの処理をしてエラーが出る場合以下の記述をconfig/application.rbに追加

```
config.autoload_paths += Dir[Rails.root.join('app', 'uploaders')]
```

## gem geocoder を使うときに住所を日本語対応にする方法

- ターミナルで以下のコマンドを打つ

```terminal
rails generate geocoder:config

```

- config/initializers/geocoder.rbが生成されるので、以下のように記述

```config/initializers/geocoder.rb
Geocoder.configure(
language: :ja
# 単位をkmにしたい場合
units: :km
)


```

参考：https://github.com/alexreisner/geocoder/blob/master/lib/geocoder/configuration.rb
https://github.com/alexreisner/geocoder/blob/master/lib/geocoder/configuration.rb
http://railscasts.com/episodes/273-geocoder


## request.refererとは

request.refererでリファラーを取得する。リファラーとは該当ページに遷移する直前に閲覧されていた参照元ページのURLのこと

```rb
# room_controller.rb
  redirect_back(fallback_location: request.referer)
```

## renderの部分テンプレートの呼び出しの際の注意点

```rb
前提
@group = Group.find(params[:id])
@messages = (@)group.messages.includes(:user)
部分テンプレート名 : _message.html.haml
```

- （ファイル名に縛りあり）モデルオブジェクトの集合を部分テンプレートで各要素ごとに展開したい場合

```messages/_message.html.erb
render @message
```

- （ファイル名に縛りなし）モデルオブジェクトの集合を部分テンプレートで各要素ごとに展開したい場合

```
render partial: partial/message', collection: @messages
```

## groupに関連するエラーメッセージを表示したい。

```rb

group.errors.full_messages.each do |message|
  <%= message %>
end
```

## toastr表示法

### application.jsでrequireする時jqueryよりも下に配置しなければならない

```application.js
//= require rails-ujs
//= require turbolinks
//= require_tree .
//= require jquery3
//= require popper
//= require toastr　<--ここ
//= require bootstrap-sprockets
```
### toastrの最新版は記述内容が旧版と異なる
　　旧版：　https://github.com/tylergannon/toastr-rails

　　新版：https://github.com/CodeSeven/toastr

新版での記述の仕方は以下の通り

```_message.html.rb
<% unless flash.empty? %>
  <script type="text/javascript">
#旧版はflash.each do |key, value|と記述 
    <% flash.each do |f| %>
      <% type = f[0].to_s.gsub('alert', 'error').gsub('notice', 'success') %>
#旧版はtoastr['<%= type %>']('<%= value %>')と記述
        toastr['<%= type %>']('<%= f[1] %>')
    <% end %>
  </script>
<% end %>
```

## テーブル作成の際、references型で外部キーを指定した時のメリット
indexを自動で貼ってくれる。

## リファクタリング

### 同じ処理を繰り返し書く場合
- before

```books/show.html.erb
<% if user_signed_in? %>
  <% if current_user == @book.user %>
    <%= link_to "編集", edit_book_path(@book) %>
  <% end %>
<% end %>
```

```comments/show.html.erb
<% if user_signed_in? %>
  <% if current_user == @comment.user %>
    <%= link_to "編集", edit_comment_path(@comment) %>
  <% end %>
<% end %>
```
同じ処理が繰り返されており、無駄が多い。


リファクタリングのため、この処理をヘルパーでメソッド化すると簡潔に記述できる。
ファイル特有のインスタンス変数はメソッドの引数とすることで解決。

- after

```helpers/application_helper.rb
def current_user_has?(instance)
  user_signed_in? && current_user == instance.user
end
```
```books/show.html.erb
<% if current_user_has?(@book) %>
   <%= link_to "編集", edit_book_path(@book) %>
<% end %>
```

### 複雑な条件分岐がある場合

- before

```note.html.erb
<% if @book.published? %>
  <% if @book.popular? %>
    <%=  link_to "詳細", book_path(@book) ,class: "popular" %>
  <% else %>
    <%=  link_to "詳細", book_path(@book) ,class: "normal" %>
  <% end %>
<% else %>
  <div class=normal>閲覧不可</div>
<% end %>
```

これも処理内容をヘルパーで定義すると以下のようになる

- after

```helper.rb
def book_link(book) 
  class_name = book.popular? ? "popular" : "normal"
 if book.published?
    content_tag :a, "詳細", class: class_name
 else
   content_tag :div, "閲覧不可", class: class_name
  end
end
```

```note.html.erb
<%= book_link(@book) %>
```

### 複数のコントローラーに同じ処理が記述されている場合

controllers/concernsにファイルを追加して、共通するコードをモジュールとして定義し、必要箇所で読み込ませる

- before

```products_controller.rb
class ProductsController < ApplicationController
  before_action :set_cart

  ~ ~ ~ 

  private

  def set_cart
    @cart = Cart.find_by(id: session[:cart_id])
    if @cart.nil?
      @cart = Cart.create
      session[:cart_id] = @cart.id
    end
  end
end
```

```top_controller.rb
class TopController < ApplicationController
  before_action :set_cart

  ~ ~ ~

  private

  def set_cart
    @cart = Cart.find_by(id: session[:cart_id])
    if @cart.nil?
      @cart = Cart.create
      session[:cart_id] = @cart.id
    end
  end
end
```
- after

```rb
# app/controllers/concerns/current_cart.rb
module CurrentCart
  extend ActiveSupport::Concern

  private

  def set_cart
    @cart = Cart.find_by(id: session[:cart_id])
    if @cart.nil?
      @cart = Cart.create
      session[:cart_id] = @cart.id
    end
  end
end
```

```app/controllers/products_controller.rb
class ProductsController < ApplicationController
  include CurrentCart
  before_action :set_cart
end
```

```app/controllers/top_controller.rb
class TopController < ApplicationController
  include CurrentCart
  before_action :set_cart
end
```

### コントローラに複雑な処理を記述している場合

モデルにコードを切り出す

- before

```rb
# controller
@user = User.find(params[:user_id])
@user.name = params[:user_name]
if params[company_name]
  @user.company_name = params[company_name]
  company = Conpany.find_by(name: params[company_name])
  if company
    @user.company = company
  end
end
@user.save
```

- after

```rb
# controller
@user = User.find(params[:user_id])
@user.update_company(params[company_name])

# model
class User < ActiveRecord::Base
  def update_company(comnapy_name)
    if company_name
      self.company_name = comnapy_name
      company = Conpany.find_by(name: params[company_name])
      if company
        self.company = company
      end
    end
    save
  end
end
```

## マイグレーションファイルをバージョン指定して削除する。

`rails db:migrate:status`でマイグレーションファイルのバージョンを確認。

`rails db:migrate:down VERSION=<バージョン番号>`でマイグレーションファイルをdown状態にする。

`rm db/migrate/<マイグレーションファイル>`でマイグレーションファイルを削除。


## テーブルを削除する

- `rails g migration drop_<削除したいテーブル名>`でマイグレーションファイルを作成(例：photosテーブルを削除する)

- マイグレーションファイルを以下のように編集

```rb
class DropPhotos < ActiveRecord::Migration[5.1]
  def change
    drop_table :photos
  end
end
```

- `rails db:migrate`してテーブル削除完了

## カラムを削除する

- `rails g migration remove_<カラム名>_from_<テーブル名> カラム名:型名`でマイグレーションファイル作成

- 以下のようなマイグレーションファイルが作成される。（例：roomsテーブルからphotoカラムを削除）

```rb
class RemovePhotoFromRooms < ActiveRecord::Migration[5.1]
  def change
    remove_column :rooms, :photo, :string
  end
end
```

- `rails db:migrate`を入力し、完了。

## モデルやルートを簡単に確認する
- `pry-rails`を使うと使えるコマンド
  - `show-models`: 全テーブルを確認できる
  - `show-routes`: 全ルートを確認できる

## fetchメソッド
- `hash.fetch(第一引数, 第二引数)`
- hashから第一引数で指定したキーの値を取り出して返す。
- キーが存在しない場合は、第二引数を返す

## ヘッダー情報を取得
- request.headersを使用
- ipアドレス取得は`request.remote_ip`
- 環境変数取得は`request.env`

## ヘルパーメソッドを自作するときはhメソッドが便利
- `h`メソッドは`>`を`&lt;`のようにHTML特殊文字変換してくれる
- `html_safe`メソッドは自動的に特殊文字変換するのを防ぐ

```ruby
def sample_method(text)
  h(text).gsub("\n", "<br />").html_safe
end
```
