---
layout: post
title: "omniauth authentication with facebook tutorial"
date: 2014-04-21 20:00:24 +0800
comments: true
categories: omniauth facebook
---

This is a basic tutorial using Ruby on Rails 4 on how to make facebook login enable in your website by [Omniauth](http://intridea.github.io/omniauth/), the Multi-Provider Authentication for Web Application, and it's very easy to extend this feature to support other service, such as using google or twitter account login to your apps.


First of all, in order to support your web application user to login by facebook account, you need to create a facebook app, and get two specific key, FACEBOOK_APP_ID and FACEBOOK_SECRET, This tutorial assume you have already got it, and then let's build rails app first

``` bash
$ rails new omniauth_facebook_example
$ cd omniauth_facebook_example
```

add omniauth-facebook gem in your Gemfile file
``` ruby Gemfile
gem 'omniauth-facebook'
```

then make sure your have install this gem by bundle
``` bash
 $bundle install
```

and let's make a home page for your web application.
``` bash
 $rails generate controller home index
      create  app/controllers/home_controller.rb
      route  get "home/index"
      invoke  erb
      create    app/views/home
      create    app/views/home/index.html.erb
      invoke  helper
      create    app/helpers/home_helper.rb
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/home.js.coffee
      invoke    scss
      create      app/assets/stylesheets/home.css.scss
```

and then modify your route file, point your home page to home#index
``` ruby config/route.rb
  #get "home/index"
  root "home#index"
```

let's try to make sure all these steps work, by using rails server command
``` bash
$ rails server
```

This will create a web server in your machine, and under port 3000, so that you can test that in your favour browser.

<img src="https://farm8.staticflickr.com/7056/13949755071_71133deb46_o.png" width="321" height="297">

If you point to your browser with "localhost:3000", then get the result like the image above, then this is cool, you can go ahead.

Next step, we need to make a initializer for omniauth, create a file in config/initializers/omniauth.rb, and tell omniauth what the secret is. You need to tell omniauth to use which provide, and tell it to use the secret key that you get from Facebook. For security issue, we just use bash Environment variable to setup this two key, and beware of that, we need to setup these two key before we start web server so that Omniauth can accept it.

``` ruby config/initializers/omniauth.rb
OmniAuth.config.logger = Rails.logger

Rails.application.config.middleware.use OmniAuth::Builder do
  provider :facebook, ENV['FACEBOOK_APP_ID'], ENV['FACEBOOK_SECRET']
end
```

After setup omniauth initializer, let's create a User model to store user's information. We need uid and provider to store user's identity, and oauth_token for further query about facebook connection, or facebook friends, and since facebook will expire our authentication in two months, so we also need to keep that information.

``` bash
$ rails generate model User uid:string name:string oauth_token:string oauth_expires_at:datetime
      invoke  active_record
      create    db/migrate/20140421125833_create_users.rb
      create    app/models/user.rb
```

This command will create two file, one migration file, and a model file. Let's migrate it, so that users table is populate in our database.
``` bash
$ rake db:migrate
==  CreateUsers: migrating ====================================================
-- create_table(:users)
   -> 0.0016s
==  CreateUsers: migrated (0.0017s) ===========================================
```

And we need a method to create user records, let's do it!
``` ruby app/models/user.rb
class User < ActiveRecord::Base
  def self.from_omniauth( auth )
    where(auth.slice(:provider, :uid)).first_or_initialize.tap do |user|
      user.provider = auth.provider
      user.uid = auth.uid
      user.name = auth.info.nickname
      user.oauth_token = auth.credentials.token
      if auth.credentials.expires && auth.credentials.expires_at
        user.oauth_expires_at = Time.at(auth.credentials.expires_at)
      end
      user.save!
    end
  end
end
```

In this method, the parameter auth is provide by facebook response, and we can access form request.env['omniauth.auth'] from controller. There's one example auth demonstrated in [omniauth-facebook github page](https://github.com/mkdynamic/omniauth-facebook), and just copy down here in case your need it:

``` ruby
{
  :provider => 'facebook',
  :uid => '1234567',
  :info => {
    :nickname => 'jbloggs',
    :email => 'joe@bloggs.com',
    :name => 'Joe Bloggs',
    :first_name => 'Joe',
    :last_name => 'Bloggs',
    :image => 'http://graph.facebook.com/1234567/picture?type=square',
    :urls => { :Facebook => 'http://www.facebook.com/jbloggs' },
    :location => 'Palo Alto, California',
    :verified => true
  },
  :credentials => {
    :token => 'ABCDEF...', # OAuth 2.0 access_token, which you may wish to store
    :expires_at => 1321747205, # when the access token expires (it always will)
    :expires => true # this will always be true
  },
  :extra => {
    :raw_info => {
      :id => '1234567',
      :name => 'Joe Bloggs',
      :first_name => 'Joe',
      :last_name => 'Bloggs',
      :link => 'http://www.facebook.com/jbloggs',
      :username => 'jbloggs',
      :location => { :id => '123456789', :name => 'Palo Alto, California' },
      :gender => 'male',
      :email => 'joe@bloggs.com',
      :timezone => -8,
      :locale => 'en_US',
      :verified => true,
      :updated_time => '2011-11-11T06:21:03+0000'
    }
  }
}
```

As a tutorial, we just need basic parameter ( uid, provider, name, token, expires ), if you need more information, you can add more columns into your user model, and store it when you create the user model.

For a typical facebook auth procedure, user will first access /auth/facebook, then browser will redirect user to facebook to enter credentials, after authentication on facebook, browser will redirect back to /auth/facebook/callback, in this callback route, we can create a user record, by User.from_omniauth, and then sign in this user. So let's create these route first.

``` ruby config/route.rb
OmniauthFacebookExample::Application.routes.draw do
  #get "home/index"
  root "home#index"

  get 'auth/facebook/callback', to: 'sessions#create'
  get 'auth/failure', to: redirect('/')
  get 'signout', to: 'sessions#destroy', as: 'signout'

  ...
  ...
end
```

First, we point auth/facebook/callback to sessions#create, which we will talk about soon. and for sign-out, It's handle by sessions#destroy. And for some failure case, we redirect back to root.

If you need more authentication methods provide by services like twitter or github, you may change the callback path with:

``` ruby config/route.rb
OmniauthFacebookExample::Application.routes.draw do
  ...
  #get 'auth/facebook/callback', to: 'sessions#create'
  get 'auth/:provider/callback', to: 'sessions#create'

  ...
  ...
```


Let's create a session controller to deal with sign in / out mechanism.

``` ruby app/controllers/sessions_controller.rb
class SessionsController < ApplicationController
  def create
    user = User.from_omniauth(env["omniauth.auth"])
    session[:user_id] = user.id
    redirect_to root_url
  end

  def destroy
    session[:user_id] = nil
    redirect_to root_url
  end
end
```

When user sign in, we use User.from_omniauth method to create a new user record in database, and record this user's id into session ( This is a typical way to identify if user is login or not! ), after user sign in, we redirect back to root path.

When user sign out, we just delete it's record from session, then redirect back to root path.

Before we process to create a link to let user sign-in using facebook, we need a helper method, to identify if current user is login or not. Let's make it.

``` ruby app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  # Prevent CSRF attacks by raising an exception.
  # For APIs, you may want to use :null_session instead.
  protect_from_forgery with: :exception

  private

  def current_user
    @current_user ||= User.find(session[:user_id]) if session[:user_id]
  end
  helper_method :current_user
end
```

This helper method called current_user, if current session have user_id, then we query database to cached this user as current_user, and by using helper_method, we populate this method in view. It's quite nice to have this helper method in view.

The final step will be to create a link for user to sign in or sign out. For demonstrated easily, I just paste the code under layout file. It's a bit of ugly, but It's enough for testing purpose.

``` ruby app/views/layouts/application.html.erb
<!DOCTYPE html>
<html>
<head>
  <title>OmniauthFacebookExample</title>
  <%= stylesheet_link_tag    "application", media: "all", "data-turbolinks-track" => true %>
  <%= javascript_include_tag "application", "data-turbolinks-track" => true %>
  <%= csrf_meta_tags %>
</head>
<body>

<div id="user_nav">
  <% if current_user %>
    Signed in as <strong><%= current_user.name %></strong>!
    <%= link_to "Sign out", signout_path, id: "sign_out" %>
  <% else %>
    <%= link_to "Sign in with Facebook", "/auth/facebook", id: "sign_in" %>
  <% end %>
</div>

<%= yield %>

</body>
</html>
```

When the user have been log in, it will show user's name and "Sign out" link provided, otherwise, It will have "Sign in with Facebook" only.

Before we test, we need to populate two variable into bash, and then start web server!
``` bash
$ export FACEBOOK_APP_ID=1234567890123412
$ export FACEBOOK_SECRET=134123412123412341234123
$ rails server
```


Let's see what we got! This is the first page when you hit this web server.
<img src="https://farm6.staticflickr.com/5520/13972908843_6540ee1cf6_o.png">

After login with facebook credentials, redirect back to our app, you will see you have already sign-in, with nickname you use in your Facebook Account, And you can sign-out easily!

<img src="https://farm8.staticflickr.com/7424/13972909353_a6669c4d54_o.png">


That's it! This tutorial show that how easy you can to add facebook authentication in your web app, and if you want, you can add as more services as your want, allow your user using their google account or twiiter account, even Weibo account to sign in to your app.


And for all the source code using here, you guys can checkout form my personal [github](https://github.com/simoniong/omniauth_facebook_example).


If you have any questions, please drop down a notes.
