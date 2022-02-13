+++
date = 2013-10-02
description = "twitter-bootstrap notes"
title = "Use twitter-bootstrap in a rails project"

[taxonomies]
categories = ["Programming"]
tags = ["Ruby", "Rails"]
+++

昨天在看[Railscasts](http://railscasts.com/)上看到了这个[视频](http://railscasts.com/episodes/328-twitter-bootstrap-basics)，于是实践一把，新建一个demo项目

```sh
liul85:RubyOnRails MAC$ rails new demo
      create
      create  README.rdoc
      create  Rakefile
      create  config.ru
      create  .gitignore
      create  Gemfile
      create  app
      create  app/assets/javascripts/application.js
      create  app/assets/stylesheets/application.css
      create  app/controllers/application_controller.rb
      create  app/helpers/application_helper.rb
      create  app/views/layouts/application.html.erb
      ......
      ......
Using sdoc (0.3.20)
Using sqlite3 (1.3.8)
Using turbolinks (1.3.0)
Using uglifier (2.2.1)
Your bundle is complete!
Use `bundle show [gemname]` to see where a bundled gem is installed.
```

然后编辑Gemfile在其中加入bootstrap，再运行bundle安装gem

```ruby
gem 'twitter-bootstrap-rails'
gem "therubyracer"
gem "less-rails"
```

接下来可以进行自己的项目，比如利用脚手架创建一个简单的微博
```sh
liul85:demo MAC$ rails g scaffold post title:string content:text --skip-stylesheets
      invoke  active_record
      create    db/migrate/20131002193338_create_posts.rb
      create    app/models/post.rb
      invoke    test_unit
      create      test/models/post_test.rb
      create      test/fixtures/posts.yml
      invoke  resource_route
       route    resources :posts
      invoke  scaffold_controller
      create    app/controllers/posts_controller.rb
      invoke    erb
      create      app/views/posts
      create      app/views/posts/index.html.erb
      create      app/views/posts/edit.html.erb
      create      app/views/posts/show.html.erb
      create      app/views/posts/new.html.erb
      create      app/views/posts/_form.html.erb
      invoke    test_unit
      create      test/controllers/posts_controller_test.rb
      invoke    helper
      create      app/helpers/posts_helper.rb
      invoke      test_unit
      create        test/helpers/posts_helper_test.rb
      invoke    jbuilder
      create      app/views/posts/index.json.jbuilder
      create      app/views/posts/show.json.jbuilder
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/posts.js.coffee
      invoke    scss
liul85:demo MAC$
```

别忘了运行dg:migrate来创建迁移
```sh
liul85:demo MAC$ rake db:migrate
==  CreatePosts: migrating ====================================================
-- create_table(:posts)
   -> 0.0019s
==  CreatePosts: migrated (0.0020s) ===========================================
```

启动rails服务，你看到的是一个没有任何前端的页面，很土

下来我们在assets中安装bootstrap-rails
```sh
liul85:demo MAC$ rails g bootstrap:install
      insert  app/assets/javascripts/application.js
      create  app/assets/javascripts/bootstrap.js.coffee
      create  app/assets/stylesheets/bootstrap_and_overrides.css.less
      create  config/locales/en.bootstrap.yml
        gsub  app/assets/stylesheets/application.css
        gsub  app/assets/stylesheets/application.css
liul85:demo MAC$
```

在我们的项目中使用全局布局，会提示与rails本身生成的布局视图有冲突，我们选择替换
```sh
iul85:demo MAC$ rails g bootstrap:layout application fixed
    conflict  app/views/layouts/application.html.erb
Overwrite /Users/MAC/Documents/RubyOnRails/demo/app/views/layouts/application.html.erb? (enter "h" for help) [Ynaqdh] y
       force  app/views/layouts/application.html.erb
liul85:demo MAC$
```

然后重启rails s，是不是发现立马看到页面显示高端大气上档次了

这样，bootstrap的基本使用就学会了，它提供了很方便的前端效果，对于学习rails的初学者来说很有用。接下来可以再修改bootstrap_and_overides.css.less文件来提升前端效果了。

更多内容可以参考[这里](https://github.com/seyhunak/twitter-bootstrap-rails)
