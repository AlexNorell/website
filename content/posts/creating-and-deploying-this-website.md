---
title: Creating and Deploying This Website
summary: A detailed explination on how this website is generated and hosted. 
categories:
  - web
tags:
  - hugo 
  - jenkins
  - docker
  - nginx
draft: False
---

## Project Goals ##

* Create a website that can be statically generated from markdown files
* Put all files needed for the static generation in version control
* Create infastructure around the generation and deployment of the website
* Deploy the webite to a public cloud


## About the Website ##

This website is generated using [Hugo](https://gohugo.io), a static website generator written in [Go](https://golang.org/). The content for this website is written in [Markdown](https://daringfireball.net/projects/markdown/) files that are then parsed by Hugo and a static website is generated. During generation, a theme is used for generating the HTML pages.

### Theme ###

The theme for this website is [Strata](https://html5up.net/strata) from [HTML5Up](https://html5up.net/) by [aj](https://twitter.com/ajlkn). I have been using them theme for my landing page for the last couple of year. When I made the decision to switch to Hugo, I went looking for this theme and happy that someone had already ported it. Strata was then [ported to a Hugo theme](https://github.com/digitalcraftsman/hugo-strata-theme) by [digitalcraftsman](https://github.com/digitalcraftsman/), but the theme hadn't been maintained in a couple year. 

I [forked](https://github.com/alexnorell/hugo-strata-theme) the theme and merged in all of the PR's that had been ignored over the years. I also went through the theme and added some features that I wanted, like the ability to just list a Social Media side and username in an arbitraty order, and have the links generated in that order. I also updated the [Font Awesome](https://fontawesome.com/) font icons to the latest version and I decided to self host them rather than use their CDN.

### Sections ###

The website supports several sections. The only section that is currently being used in the `posts` section, which contains this blog post. There is also a `portfolio` section that I will use for hosting static pages of projects that I build.

### Generation ###

The site is generated using:

```shell
hugo
```

This builds the website and places it into the `public` directory. This directory can then be copied directly to a web host.

## Version Control ##

Hugo has several built in features that tie in with git. Page creation and modification dates can be inferred using the git history of the file. This allows for less manual, arbitrary data to be updated by Hugo rather than me.