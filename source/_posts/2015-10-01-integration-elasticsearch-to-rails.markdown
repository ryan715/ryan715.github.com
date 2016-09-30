---
layout: post
title: "Integration ElasticSearch to Rails"
date: 2015-10-01 11:01:21 +0800
comments: true
categories: [ruby on rails, elasticsearch]
---

### 导言

  这两天项目要求，把现有的搜索改成ElasticSearch(后面简称es)。之前接触 过一些es，后来就开始捣鼓。记得railcasts上面有讲过相关视频，重温了下就  开始弄，没弄多久发现上面用的[tire](https://github.com/karmi/retire)已经retire了。为了让更多的朋友们不走冤枉路，所以才有了此文。

### 简介
  
  大致说下什么是es，详细的[Wikipedia](https://en.wikipedia.org/wiki/Elasticsearch)有介绍。es其实就是一个搜索的引擎，从开源项目lucene出来，lucene是java编写的，比较复杂，要使用它必须了解核心一大堆东西。es就是包装了一层，然后提供RESTFUL API调用，从而让全文搜索变得更加简单。
  
### 过程

##### 安装
  在安装es之前，先安装jdk。
  Mac环境, 运行 `brew install elasticsearch `, 然后运行`elasticsearch --config=/usr/local/opt/elasticsearch/config/elasticsearch.yml`启动，访问[ http://localhost:9200]( http://localhost:9200)，访问成功就表示安装完成了。
  
  其他环境，访问[官网]( http://elasticsearch.org/download)相关安装包下载。 

##### 使用
  在Gemfile中加入

```ruby 
 gem 'elasticsearch-model' 
 gem 'elasticsearch-rails' 
```

 注意：es-model自带了分页插件，如果你在gemfile中有分页，如`will_paginate` 或者 `kaminari`，要把他们放到es-model和es-rails的前面。
 
 在需要添加搜索的model添加以下代码：

```ruby
 class University < ActiveRecord::Base
   include Elasticsearch::Model
   include Elasticsearch::Model::Callbacks
 end
```

 完成引用后，我们可以编写search方法了：
 
```ruby
 def self.search(search)
    response = __elasticsearch__.search(search)
 end
```

 这是一个很简单的search，通过传入的参数直接进行检索。我们可以使用DSL来使我们的检索语句更加满足我们的业务需要，以下是我需要检索一个状态为1，并且从栏目名为name的一个检索:

```ruby
def self.search_filter(params)
  response = __elasticsearch__.search(
      "query": {
        "filtered": {
          "filter":   {
            "bool": {
              "must":     { "term":  { "status": 1 }},
              "must": {
                "query": { 
                  "match": { "name": params }
                }
              }
            }
          }
        }
      }  
    )
end
```

关于es的DSL更多写法，大家可以访问[这里](https://www.elastic.co/guide/en/elasticsearch/guide/current/_most_important_queries_and_filters.html)。里面详细的讲解了query,filter等一些常用查询，大家可以根据业务需要自行改装。
 
 
 然后我们为model创建index, 主要给es使用：

```ruby
 mapping dynamic: false do
   indexes :name
   indexes :tag
 end
```
 
 我们继续往下走，model是可以serialized成json的，我们使用`as_indexed_json`这个方法。我们可以这样写：
 
```ruby
 def as_indexed_json(options={})
   self.as_json(
     only: [:id, :name, :description, :status],   
     include: { tags: { only: [:name]}}
   )
 end
```

 `include`的部分是处理association的，`only`是model本身的字段属性。完成了以上调整，我们的model搜索基本完成了。如果你现在使用搜索，我估计还是搜索不出数据。我们要把数据导入给es，使用这个命令

```bash
 rake environment elasticsearch:import:model CLASS='your_model_name' FORCE=y
```

好啦。基本就完成。es默认自带中文分词，但是有些posts反馈说不大好用，可以使用[es-rtf](https://github.com/medcl/elasticsearch-rtf)，集成了中文的分词插件，下载直接可以用。必须要安装jdk，里面有详尽的使用方法。
 
  

  
###相关资料
gem的[README](https://github.com/elastic/elasticsearch-rails/tree/master/elasticsearch-model),有耐心慢慢看可以了解很多

一些快速的tutorial

http://www.sitepoint.com/full-text-search-rails-elasticsearch/
http://aaronvb.com/articles/intro-to-elasticsearch-ruby-on-rails-part-1.html
http://www.spacevatican.org/2012/6/3/fun-with-elasticsearch-s-children-and-nested-documents/

可以了解，railcast的es介绍 http://railscasts.com/episodes/306-elasticsearch-part-1