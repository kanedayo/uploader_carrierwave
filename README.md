#■アプリ雛形の作成
```
 # console
rails new uploader_carrierwave
cd uploader_carrierwave
vim Gemfile
 #gem 'rmagick'
 gem 'rmagick', :require => 'RMagick' # 「stack level too deep」対策
 #gem 'mini_magick' # https://github.com/minimagick/minimagick
 gem 'carrierwave'
bundle install
rails g scaffold user name
rake db:migrate
```


#■carrierwaveの設定
```
 rails g migration add_avatar_to_users avatar:string
 rake db:migrate
 rails g uploader avatar
 # => app/uploaders/avatar_uploader.rb
```

```
 # app/model/avatar.rb
 class AvatarUploader < CarrierWave::Uploader::Base
   include CarrierWave::RMagick
   #include CarrierWave::MiniMagick
 
   storage :file
 
   version :thumb do
     process :resize_to_fill => [50,50]
   end
 
   def extension_white_list
     %w(jpg jpeg gif png)
   end
 end
```

```
 # app/models/user.rb
 class User < ActiveRecord::Base
   mount_uploader :avatar, AvatarUploader
   validates :name, presence: true
 end
```

```
 # app/controllers/users_controller.rb
  private
    def user_params
      params.require(:user).permit(:name,:avatar)
    end
```

```
 # app/views/users/_form.html.erb
 # 追加
 <%# form_for @user do |f| %>
 <%= form_for @user, :html => {:multipart => true} do |f| %>
 # 追加
   <div class="field">
     <%= f.label :avatar %><br/>
     <% if @user.avatar? %>
       <%= image_tag @user.avatar.thumb %>
       <%= f.hidden_field :avatar_cache if @user.avatar_cache %>
     <label><%= f.check_box :remove_avatar %>Remove avatar</label>
     <% end %>
     <%= f.file_field :avatar %><br/>
   </div>
```

```
 # app/views/users/show.html.erb
 # 追加(抜粋)
 <% image_tag @user.avater %>
```

```
 # app/views/users/show.html.erb
 # 追加(抜粋)
 <%= image_tag user.avatar.thumb %>
```

#■FAQ
##「ActiveRecord::PendingMigrationError 」

rake db:migrateのし忘れ。。。

##「Carrierwave undefined method `avatar_changed?' for User」

http://stackoverflow.com/questions/20285552/carrierwave-undefined-method-avatar-changed-for-user

##「stack level too deep」

大文字と小文字を区別しないファイルシステム(=windows)で起きるエラーみたい。

Gemfileで「gem 'rmagick', require: 'RMagick'」でOK。

http://qiita.com/eleven_2012/items/eb4099555358b770915b

http://stackoverflow.com/questions/22206350/stack-level-too-deep-when-using-carrierwave-versions
