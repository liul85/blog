+++
date = 2013-09-29
description = "rails learning notes"
title = "Default http method of form_for"

[taxonomies]
categories = ["Programming"]
tags = ["Ruby", "Rails"]
+++

在看RubyonRails Guides时候，在posts的new视图里用到了form_for方法来向控制器提交表单，在做了这节之后一直不明白这个表单是在哪里定义提交给控制器中的create动作的，因为我没有看到任何地方写了这个是提交给create动作.

```erb
<%= form_for :post, url: posts_path do |f| %>
  <% if @post.errors.any? %>
  <div id="errorExplanation">
    <h2><%= pluralize(@post.errors.count, "error") %> prohibited
      this post from being saved:</h2>
    <ul>
    <% @post.errors.full_messages.each do |msg| %>
      <li><%= msg %></li>
    <% end %>
    </ul>
  </div>
  <% end %>
  <p>
    <%= f.label :title %><br>
    <%= f.text_field :title %>
  </p>
 
  <p>
    <%= f.label :text %><br>
    <%= f.text_area :text %>
  </p>
 
  <p>
    <%= f.submit %>
  </p>
<% end %>
 
<%= link_to 'Back', posts_path %>
```

当再看到Updating Posts章节的时候，才看到一些解释，原来默认form_for是向POST提交表单，而如果我们用rake routes来看路由设置的话，会看到POST对应的就是控制器中的create方法。

```sh
iul85:blog MAC$ rake routes
   Prefix Verb   URI Pattern               Controller#Action
    posts GET    /posts(.:format)          posts#index
          POST   /posts(.:format)          posts#create
 new_post GET    /posts/new(.:format)      posts#new
edit_post GET    /posts/:id/edit(.:format) posts#edit
     post GET    /posts/:id(.:format)      posts#show
          PATCH  /posts/:id(.:format)      posts#update
          PUT    /posts/:id(.:format)      posts#update
          DELETE /posts/:id(.:format)      posts#destroy
     root GET    /                         welcome#index
```

同时也可以在form_for中指定要提交的方法，比如在update的时候，我们是提交给PATCH，转向用户的这条blog，可以这么写，在form_for中添加method，指定提交的方法就可以了，不过要注意method和后面url的对应关系，用update和destroy时候需要指定对象来操作.
```erb
<%= form_for :post, url: post_path(@post.id) }, method: :patch do |f| %>
```

这个在文档中说明如下，同时可以参考[form_helpers](http://guides.rubyonrails.org/form_helpers.html)文档。
```
The method: :patch option tells Rails that we want this form to be submitted via the PATCH HTTP method which is the HTTP method you are expected to use to update resources according to the REST protocol.
By default forms built with the form_for helper are sent via POST.
```
