---
title: "Hexo And Travis Install" 
catalog: true
date: 2017-05-09 10:23:13
subtitle: "Install hexo and auto publish with Travis"
header-img: 
tags:
- Hexo
- Travis
- Blog
catagories:
- Hexo
---
#Install hexo and auto publish with travis 
This page will tell you how to install Hexo and auto publish your blog with travis that is a publish tool.So you can publish your blog only git push your hexo program.
##Hexo Installation 
###Requirements
Before installing Hexo,you do need to intall a couple of other things:
* [Node.js](https://nodejs.org)
* [Git](https://git-scm.com/)
###Install
```bash
$ npm install -g hexo-cli
```
And now you have finished the installation of Hexo.<br>
<font color="#A9A9A9">Tip:Remmenber put all bin path above into environment variables on your opreration system.</font>
##Hexo Init
###Init
Once Hexo is intalled,run the following commands to init Hexo.
```bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```
###Modify
Modify `_config.yml` file with your own info.<br>
* Site
```yml
title	        The title of your website
subtitle	The subtitle of your website
description	The description of your website
author	        Your name
language	The language of your website
timezone
```
* Deployment(Replace to your own repo)
```yml
deploy:
  type: git
  repo: https://github.com/<yourAccount>/<repo>
  branch: <your-branch>
```

##Hexo Basics
Some Hexo commands:
```bash
$ hexo new post "<post name>"  # New a post,also you can change post to another one layout if you want
$ hexo clean & hexo generate # Generate the static file
$ hexo deploy # Hexo will push the static files(in the `public` path) into specific branch of your git repo
$ hexo server # Run hexo in local environment,default 4000 port
```

##Travis(auto publish your repo)
###The Whole Following Process:
* git push your resource code(awayls on master branch)
* Github message Travis something had changed
* Travis start a task to build what you had writed in the `.travis.yml`

###Install
Before installing Travis,you should install [Ruby](https://www.ruby-lang.org).And then
```bash
#install Travis CI
$ gem install travis
```
###Setting
You need just register the [Travis CI](https://travis-ci.org/),and relate to the your Github repo.

###Git Personal Access Token
Create your Personal Access Token in your github profile page and encrypt it with Travis CI. 
```bash
# encrypt the Personal Access Token
$ travis encrypt -r <your github name>/<your github repo> GH_TOKEN=<Personal Access Token> 
```
<font color="#A9A9A9">Tip:If your repo is public,you should add `--org` at the end.</font>

###Modify `.travis.yml`
* add environment vaviables
```yml
# the "xxx" is the cryptograph of your github personal access token
env:
    global:
      - GH_ADDR: github.com/<your github name>/<your github repo>.git
      - secure: "xxx"
```

* the whole file is that:
```yml
language: node_js
node_js: stable

# Travis-CI Caching
cache:
    directories:
      - node_modules

# S: Build Lifecycle
install:
  - npm install

before_script:
  - npm install -g hexo-cli

script:
  - hexo clean & hexo generate

after_script:
  - cd public
  - git init
  - git config user.name <your github name>
  - git config user.email <your github enail>
  - git add .
  - git commit -m "Update docs"
  - git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:gh-pages
# E: Build LifeCycle


branches:
  only:
    - master

env:
  global:
    - GH_ADDR: github.com/<your github name>/<your github repo>.git
    - secure: "xxx"
```