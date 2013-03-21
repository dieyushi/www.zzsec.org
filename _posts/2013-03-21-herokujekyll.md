---
layout: post
title: "heroku部署jekyll"
description: ""
category: "life"
tags: [misc]
---
{% include JB/setup %}

### 说明

在heroku上部署jekyll站点通常的做法是搭建jekyll环境，在heroku上进行编译，或者是使用rack-jekyll.为了部署方便，我们采用了在服务器上不安装jekyll的情况下，将jekyll编译好的文件通过git传到heroku。

`cat config.ru`
{% highlight ruby %}
require 'rack/contrib/static_cache'
require 'rack/contrib/try_static'
require 'rack/contrib/not_found'

use Rack::Deflater

use Rack::StaticCache,
  :urls => %w[/assets /favicon.ico],
  :root => "_site"

use Rack::TryStatic,
  :urls => %w[/],
  :root => "_site",
  :try  => ['.html', 'index.html', '/index.html']

run Rack::NotFound.new('_site/404.html')
{% endhighlight %}
<!--more-->
`cat Gemfile`
{% highlight ini %}
source "https://rubygems.org"

gem "rack-contrib", "~> 1.1.0"
gem "puma", "~> 1.6.3"
{% endhighlight %}

`Gemfile.lock`可以由`bundle install`生成

`cat Procfile`
{% highlight ini %}
web:     bundle exec puma -p $PORT config.ru
{% endhighlight %}

建立子目录`_site`，在`_config.yaml`中设置`destination:./.site/_site`，每次通过jekyll生成的文件放到这个目录中，然后在`./site`中可以通过git传到heroku。
