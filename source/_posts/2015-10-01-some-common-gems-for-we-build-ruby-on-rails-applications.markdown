---
layout: post
title: "Some common gems for we build Ruby on Rails applications"
date: 2015-10-01 11:16:53 +0800
comments: true
categories: [ruby on rails, gem]
---
In this post, I will share some common gems that we use for building RoR applications.

######bcrypt
Sometimes, we build user authentication features without the use of a devise gem. To solve the storing of passwords we use bcrypt. It allows you to easily store a secure hash of users passwords.

Follow some steps below
1. Install
In your Gemfile, add codes below
<pre><code>gem 'bcrypt'
</code></pre>
then run "bundle install" commands.
2. Migrate your user table
<pre><code>add_column :table_name, :password_digest, :string
</code></pre>

3.Update user model file.
In your users model file, add the code below
<pre><code> has_secure_password
</code></pre>
That's it.

###### simple_form
We always encounter CURD tasks when we build an admin    dashboard. A simple form gem is really the best option, it can help us to solve a ton of html tags problems. 

Basic Usage
<pre><code>＃ definition form url
<%= simple_form_for @post do |f| %>
＃ model collection, dropdown list html tag
<%= f.input :user_id, collection @users %>
＃ textfield html tag
<%= f.input :content %>
＃ submit
<%= f.submit '提交' %>
<% end %>
</code></pre>

###### annotate
You don't always know the schemas when you look at the models. Try annotate gem, it helps you to add comment summaraizing to the current schema.
When you have installed and run 'bundle exec annotate', you will find it is very clear to understand models. Like this:
<pre><code># == Schema Information
# Table name: users
#  id              :integer          not null, primary key
#  nickname        :string(255)
#  created_at      :datetime         not null
#  updated_at      :datetime         not null
#  password_digest :string(255)
#  remember_token  :string(255)
#

class User < ActiveRecord::Base
  before_save { self.email = email.downcase }
  before_create :create_remember_token
</code></pre>



###### will_paginate
In Ruby on Rails world, To develop paginate feature is really easy, will_paginate gem helps you to handle a lot of things about pagination. 

How to use? There is a post class need to paginate.
<pre><code>
＃ In model
@posts = Post.paginate(page: params[:paeg]
＃ or 
@posts = Post.paginate(page: params[:page], per_page: 10)
＃ In page
<%= will_paginate @posts %></code></pre>


###### puma
Unlike other Ruby Webservers, Puma was built for speed and parallelism. Puma is a small library that provides a very fast and concurrent HTTP1.1 server for Ruby web applications. It is designed for running Rack apps only.

You can install puma gem very easily, add codes below in your gemfile. 
<pre><code>gem 'puma'
</code></pre>
Then start your server 
<pre><code>rails s Puma
</code></pre>

###### better_errors
Provides a better page for Rails and other Rack apps. Includes sources code inspection, a live REPL and local/instance variable inspection for all stack frames. 

After we finish installing beter_errors gem, when the error pages appear, like this:
{<1>}![](http://asciicasts.com/system/photos/1446/original/E402I02.png)
Woo, It looks very nice, it helps us to find the problems exactly. 


###### rails_panel
This is an extension for Chrome which adds a development log withn the brower. As we visit various pages in our application we should see an entry for each one in the panel. This works for AJAX requests too.
{<2>}![](http://asciicasts.com/system/photos/1450/original/E402I06.png)

##### capistrano
Capistrano gem is a remote server automation tool. It help us to deploy a rails server more convenient.
First, you need to install the gem in your gemfile, and bundle it.
<pre><cod>group :development do
  gem 'capistrano', '~> 3.3.0'
  gem 'capistrano-rails'
  gem 'capistrano-bundler'
end
</code></pre>
The second step is to initiate it.
<pre><code>cap install</code></pre>
The third step, the complicated step, to configure the cap file.
<pre><code># config valid only for current version of Capistrano
lock '3.3.5'

set :application, '[projectname]'
set :repo_url, 'git@github.com:[username]/[projectname].git'

＃ Default branch is :master
＃ ask :branch, proc { `git rev-parse --abbrev-ref HEAD`.chomp }.call

＃ Default deploy_to directory is /var/www/my_app_name
set :deploy_to, '/var/www/[projectname]'

＃ Default value for :scm is :git
set :scm, :git
set :rbenv_type, :system
set :rbenv_path, '/home/deploy/.rbenv'
set :rbenv_ruby, '2.2.0'
set :rbenv_prefix, "RBENV_ROOT=#{fetch(:rbenv_path)} RBENV_VERSION=#{fetch(:rbenv_ruby)} #{fetch(:rbenv_path)}/bin/rbenv exec"
set :rbenv_map_bins, %w{rake gem bundle ruby rails}

set :default_environment, {
    'PATH' => "$HOME/.rbenv/shims:$HOME/.rbenv/bin:$PATH"
}


＃ puma
set :puma_rackup, -> { File.join(current_path, 'config.ru') }
set :puma_state, "#{shared_path}/tmp/pids/puma.state"
set :puma_pid, "#{shared_path}/tmp/pids/puma.pid"
set :puma_bind, "unix://#{shared_path}/tmp/sockets/puma.sock"    #accept array for multi-bind
set :puma_conf, "#{shared_path}/puma.rb"
set :puma_env, fetch(:rack_env, fetch(:rails_env, 'staging'))
set :puma_access_log, "#{shared_path}/log/puma_error.log"
set :puma_error_log, "#{shared_path}/log/puma_access.log"
set :puma_role, :app
set :puma_threads, [0, 16]
set :puma_workers, 0
set :puma_worker_timeout, nil
set :puma_init_active_record, false
set :puma_preload_app, true


＃ Default value for :format is :pretty
＃ set :format, :pretty

＃ Default value for :log_level is :debug
＃ set :log_level, :debug

＃ Default value for :pty is false
＃ set :pty, true

＃ Default value for :linked_files is []
set :linked_files, %w{config/database.yml config/secrets.yml config/env.yml}

＃ Default value for linked_dirs is []
set :linked_dirs, %w{bin log tmp/pids tmp/cache tmp/sockets vendor/bundle public/system public/uploads}

＃ Default value for default_env is {}
＃ set :default_env, { path: "/opt/ruby/bin:$PATH" }

＃ Default value for keep_releases is 5
 set :keep_releases, 5

set :keep_releases, 5

namespace :deploy do

  desc 'Restart application'
  task :restart do
    on roles(:app), in: :sequence, wait: 5 do
    end
  end

  after :restart, :'puma:restart'
  after :publishing, :restart

  after :restart, :clear_cache do
    on roles(:web), in: :groups, limit: 3, wait: 10 do
    end
  end

end

</code></pre>
More details can be found at the following link in our team manual.  https://zhaozijie.gitbooks.io/intro\_rails/content/cong\_yi\_tai\_kong\_de\_fu\_wu\_qi\_shang\_bu\_shu\_rails\_ying\_yong.html


##### letter_opener
When we develop mail features, we always want to preview our mail template, and we don't want to send mail for testing. We use letter_opener gem.

1 Install
<pre><code>gem 'letter_opener' 
</code></pre>

2  Configuration

To set the codes in config/environments/development.rb

<pre><code>config.action_mailer.delivery_method = :letter_opener</code></pre>

Ok, That's it. The email will pop up in your brower, they are stored in tm/letter_openrer.


##### rack\_mini_profiler

We use rake\_mini_profiler gem to monitor the speed of the rails app.

First, add codes below to gemfile and install

<pre><code>gem 'rack-mini-profiler' </code></pre>

Second, there is no second step if you want :). Restart your server, and visit your app in the brower. You will find the speed data on the top-left of brower. Like this:

{<3>}![](/content/images/2015/09/D5059D17-B88F-40B1-9474-B774FC6F3D5E.png)

##### rack-attack

We advise you to use the rack-attack gem to protects the rails apps. It allows you to create whitelisting, blacklisting, throttling, and tracking. Let's add the gem to your web apps.

Install, add it to your Gemfile
<pre><code>gem 'rack-attack'</code></pre>

Set configuration in the application.rb
<pre><code># In config/application.rb
config.middleware.use Rack::Attack </code></pre>  

Add custom codes to rack-attack.rb
<pre><code># In config/initializers/rack-attack.rb
class Rack::Attack
  # custom codes
  
  #for example, white list
  Rack::Attack.whitelist('allow from localhost') do |req|
    '127.0.0.1' == req.ip
  end
  
  #for example, black list
  Rack::Attack.blacklist('block 1.2.3.4.') do |req|
    '1.2.3.4' == req.ip
  end
end</code></pre>  
About rack-attack, I will give more details in next post.

