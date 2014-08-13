---
layout: post
title: "Getting Started With ElasticSearch and Plugin Development"
date: 2014-08-10 19:33:53 +0900
comments: true
categories: Lucence Search Java
---

Coming across to `ElasticSearch` when preparing for the programming chanllege held by WorksApplications in Tokyo Event, and falling love with her after then. Well, with no much new techonoligies involved, `ElasticSearch` is definitely a beautiful piece of software and a successful product. Claimed that with the core search engine powered by `Lucene`, providing `RESTful interface` and born as a distributed and scheme-free search oriented product, `ElasticSearch` has boosted the huge
business of **Github** with billions of lines and **Stackoverflow** with billions of Q&A.

After digging some 101 tutorials around, I found it easy to set up `ES`(abbreviated for ElasticSearch and will use afterwards) and begin to play in 10 minutes. However, to understand what is going on underhood and what is the most tricy and interesting part, also to those who try to develop plugins to enpower `ES`, I will share something which would be useful for newbies. There are already tons of tutorials online, to same your time, please see
[elasticsearch-getting-started](https://github.com/hustcalm/elasticsearch-getting-started) hosted on my github for a good cheat sheet.

<!--more-->

### Basic Concepts of ES
Someone got to do the dirty work when something happens so naturally and gracefully. `ES` is graceful enough to just give users `RESTful` interfaces and everything is done! You can just do `CRUD` operations as naturally as just sending some HTTP request to some urls according to some predefined rules. So, we know at this moment that, `RESTful` interfaces are a great method to bridge `ES` and `end users`. So, for the `CRUD`, what are mainly involved? 

To start with familiar concepts, let's go with `Database`. OK, when you get a `Database`, it is basically a container for all your data. You can create varies of `Tables` in a `Database`, for a specified `Table`, you would firstly define a scheme for it, as the first column should be **_id** as integer numbers, the second should be **_name** as strings, etc. Then in a `Table`, you would have many `Rows`, each `Row` relates to a specified `record`. If you know this well, it's easy to catch up
with `ES`, after some adaptations to shift with the search engine background.

Forget about the storage engine for good, let's see what is included in a `RESTful` url.

#### Index
An `index` is basically a `Table` to contain varies of `Documents`, here `Document` again means `record`. So basically when you do a search, you are searching on an `index` or multiple `indexes` at the same time. No trick at all here, `ES` will maintain `inverted index` for each `field` of each `document` in an `index`, also some other `inverted index` will be generated for search, so an action of search is actually matching the `query` or user input keywords with the `inverted index`
records. For easy understanding, think it as a book, when you try to lookup a word, instead of searching for it from the very beginning page by page, just see the `Word Index` for a happy ending.

#### Type
Yes, `ES` can have different kinds of `Documents` in an `index`. What does that mean? In `MySQL`, can you have different kinds of `Records` in a single `Table`? Found it impossible, huh? `ES` can save you for free, here different kinds, we can also say different `Types`. Think it this way, you have many documents in an `index`, and not all the documents are the same kind, let's say, doc1 contains just a long text paragraph, doc2 contains another number indicating the number of
words of one paragraph besides a long text paragraph, doc3 maybe contain another field for storing some comments and doc 4, etc. Thus we can have many kinds of `Documents` in a single `Index`, maybe each document is one kind, or there are m tpyes for n documents(n >= m). Got any idea about this?

#### Mapping
Claimed as `scheme-free`, the magic is really played by `guess` or `a default scheme framework`. If you try to creat an `index` and put `documents` in it, `ES` will try to construct the index according to a default manner, for example, storing the field as `string` and `anylyzed` using the default anylyzer.(An anylyzer is used to splitted a sentence or phrase into independent words, in order to let people search partial keywords in a document.) If the dafault scheme is just working good,
you need not do anything. However, unfortunately this is not always the case, so we just mentioned that we have many `Types` of `Documents` in an `Index`, to change the way that `ES` is treating how each `Type` of `Document` is processed, we can specify a `Mapping` for each `Type`. Well, for `Mapping`, it is actually a `scheme` specified by user, isn't it?

#### ID
Well, well, finally we can put our `Documents` now. For each document, specify a **ID** when uploading to `ES`, such as `PUT /megacorp/employee/1` and `PUT /megacorp/employee/2`, here `1` and `2` are both `IDs`. Again for retrieving a specified document from `ES`, specify the **ID** also, as `GET /megacorp/employee/1`.

OK, I think you are ready to go after being familiar with the basic concepts. Let's talk about something about `search`.

### Search using ES
Simple and beautiful, `ES` is smart enough for basic searcing, even real-time suggestions and near real-time search is supported. So you want to search?

    GET /megacorp/employee/_search
    GET /megacorp/employee/_search?q=last_name:Smith

Try this!

For more sophiscated searches, try using `Query DSL`, like:

    GET /megacorp/employee/_search
    {
        "query" : {
            "match" : {
                "last_name" : "Smith"
            }
        }
    }

I won't talk too much about `Query DSL` here, go dig the [ES Reference](http://www.elasticsearch.org/guide/en/elasticsearch/client/java-api/current/query-dsl-queries.html).

### Client API
As `Luncene` is born with `Java` blood, `Java` is the native client API for `ES`. Everything you can do with `RESTful` interfaces, you can get it done with `Java API` or other programming language bindings. However, comparing to the `RESTful` style, all the bindings suck a little especially when having to construct complicated searches or aggregations, using `Qeury DSL` is too much graceful than any `Client API`.

The [ES Reference](http://www.elasticsearch.org/guide/en/elasticsearch/client/) has detailed documentation on each `Client API`, go and find your own one.

### Plugin Development
Well, as the development of `ES` is evolving too rapidly, I guess, the APIs and even fundamental classes are not stable enough, so for plugin development, there are not good official tutorials to follow. However, some good guys who have the training maybe, give some useful references to getting people started. Or, maybe they just hack the source code to figure out how, admirable! For `ES` plugins, there are basically two types, one is **site** plugin, the other is **non-site** plugin.

#### Site Plugin
As we said, all the **CRUD** operations can be done through **RESTful** interfaces, so basically speaking, a **site** plugin is a wrapper or delegation for easier `RESTful`, mainly focusing on the **front-end**. Any programming language or tools which are capable of sending and receiving of HTTP requests can be used, however `JavaScript` seems to be the most chosen one since it is so flexible and repid prototyping.

For a great example, see [ElasticSearch Head](http://mobz.github.io/elasticsearch-head/).

#### Non-Site Plugin
As a **site** plugin, we are actually doing the user end work, as we are not playing with the `ES` source code or core component directly. For **non-site** plugin, we will write some `Java` code, inherate from some abstract classes relating to `Plugin` and `Module`.

A `Plugin` will be instanced when loaded by `Plugin Manager` of `ES`, then the related `Modules` will be loaded, and the real job will be done through `Module Handlers`.

#### Good Example
For a complete reference and vivid example for plugin development, see [elasticsearch-hotsearch](https://github.com/hustcalm/elasticsearch-getting-started) developed by **Cgroups** during the **Tokyo Event**.

### Summary
For good overview and other references after reading this blog, see my github repo [elasticsearch-getting-started](https://github.com/hustcalm/elasticsearch-getting-started).

We learn basic concepts of `ES` here, knowing `Client API` and `Plugin Development`, however, we don't cover `Distributed` and other interesting parts of `ES` here. By starting from here, I believe you are capable of digging deeper by yourself by referencing the official site and other guys's posts. Though the official documentation is under construction by the time when writing this post, people like you who read this post will find it no difficulty to move on:-)
