# niubility.org.cn using jekyll theme 

A Simple, Bootstrap Based Theme. Especially for developers who like to show their projects on website and like to take notes. There are also some magical features to discover. 

## [niubility.org.cn](http://niubility.org.cn/)

Open issues if you find bugs or even have trouble installing jekyll or dependencies. :D

Or contact: gaoxinzhong1992@gmail.com

> Strongly suggest to fork and change project name to create your GitHub Pages instead of downloading it directly. Because in the future, I will develop many funny modules like 'footprint' to show your world wide trip. Could be easier to merge new features in the future.

# install and setup

Before using it, you may need [Bower](http://bower.io/) and [Bundler](http://bundler.io/) on your local to install dependencies.

1. Fork code and clone
2. Run `bower install` to install all dependencies in [bower.json](https://github.com/DONGChuan/DONGChuan.github.io/blob/master/bower.json)
3. Run `bundle install` to install all dependencies in [Gemfile](https://github.com/DONGChuan/DONGChuan.github.io/blob/master/Gemfile)
4. Update `_config.yml` with your own settings.
5. Add posts in `/_posts`
6. Commit to your own Username.github.io repository.
7. Then come back to star this theme!

> When install dependencies by bundler or gem, you may have some errors depending on your environment.

> Error about `json`. Check response of [Massimo Fazzolari on Stackoverflow](http://stackoverflow.com/questions/8100891/the-json-native-gem-requires-installed-build-tools) to quick fix your problem. (Please also use latest version instead of 1.9.3 mentioned in the response)
  
> Error about `jekyll-paginate`. Please check [here](http://stackoverflow.com/questions/35401566/dont-have-jekyll-paginate-or-one-of-its-dependencies-installed)

> Error about `SSL_connect`. Please check [here](http://stackoverflow.com/questions/15305350/gem-install-fails-with-openssl-failure) and [here](http://railsapps.github.io/openssl-certificate-verify-failed.html)

> For the moment, when you test on your local, you need to keep internet connection. Bug will be fixed soon.

## How to use

#### Create a new post

Create a `.md` file inside `_posts` folder.

Name the file according to the standard jekyll format.

```
2016-01-19-i-love-yummy.md
```

Write the Front Matter and content in the file.

```
---
layout: post
title: Post title
category: Category
tags: [tag1, tag2]
---
```
