+++ 
draft = true
date = 2020-04-19T11:33:30+02:00
title = "Creating own blog"
description = "Full tutorial"
tags = ["hugo", "headless cms", "github", "blog"]
categories = ["web", "tools"]
+++

1. Choosing of technology 
1. Choosing of static site generator
* jeckyll
* hugo
* gatsby
* some node.js stuff and VueJS
1. Choosing of hosting, the way of deployment
1. Registering domain or not
1. Choosing template
I guess you need to try some to make a decision so let's install template engine first.
1. Stackbit
1. Installing go and hugo
1. Quick start guide
1. Commit and push - important
* add .gitignore with public/ dir
* git init
* git commit
* git push
1. Deploy to github using gh-pages branch
* push gh-pages branch
* change repo settings
    - branch (it should set automatically)
    - domain - CNAME
* set-up github action - missing in docs
1. Disqus - set up a page
1. Adding search to a page
1. Analitics
* too much hassle - GDPR
1. Headless CMS - is it worth it?

# Choosing of the technology

Conventional websites are built with front-end and back-end part.
On the other hand one can think of so called JAM stack.

JAM stack is an emerging trend in a web development. JAM stack is shorthand for JavaScript, API, M??
It means that to run the website no server side processing is needed to generate the website code.
If you need some services or functionalities public available APIs are used instead of traditional backend.
For example for commenting functionality you can take Discuss API.

You just need to host your static HTML/CSS/JavaScript code somewhere.

It makes creating websites faster and more cost effective.

# Choosing of the static site generator

It would be difficult to build a static website without any code generation tool.
Normally, back-end of the website serve this purpose.
If we want to build a static website we can use static site generator which allows for templating, styling and similar.

SSG often use Markdown as markup language which makes creating even faster and easier for non-technical publishers.

To this day there is a bunch of solutions you can check. Below the list of the SSGs I am aware of:
* Hugo
* Gatsby
* Jeckyl
* Ten oparty na Node

Hugo is build on top of Go language, and it is king of the speed.
Fans of React would love Gatsby.
Traditionalists would choose Jeckyl.
And Node enthusiasts would consider ...

My decision was Hugo.

# Choosing the hosting

Actually you can go with any hosting which offering web server.
Although there are specialised hosting services that simplifies process of deploying the site from the template. 
You can also consider using GitHub or GitLab along with its Continous Integration.
I decided to use GitHub and GitHub actions which were added recently.
I use GitHub for some time and configuring GitHub action was fairly simple, 
so it was convenient for me to take advantage of this.