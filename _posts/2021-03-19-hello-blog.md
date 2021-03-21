---
layout: post
title: "Hello, Blog"
date: 2021-03-16 13:43:51
categories: jekyll
tags: jekyll imagemagick
---
Hello. Welcome to my first blog.  In this blog, I'll briefly introduce jekyll blog environment, blog template, dns configuration.





* content
{:toc}
# Setup jekyll environment

Jekyll is a static website generator. You can write content in markdown and jeyll can generte html for you. To setup jekyll environment, you can follow the offical guide [here](https://jekyllrb.com/docs/step-by-step/01-setup/). Here's how I setup my jekyll environment.

System: macOS

Prerequist: [homebrew](https://docs.brew.sh)

1. Install ruby with `brew install ruby `

2. As promoted by homebrew, add ruby to `PATH`. For zsh, the command is `echo 'export PATH=" :$PATH"' >> ~/.zshrc`

3. Reopen zsh shell

4. Install bundle `gem install bundle`

5. Go to the root folder of your blog, create `GemFile`

   ```ruby
   source "https://rubygems.org"
   
   gem "jekyll"
   gem "jekyll-paginate"
   gem "webrick"
   ```
   
6. Create a demo html file. You can use this one.

   ```html
   <!DOCTYPE html>
   <html>
     <head>
       <meta charset="utf-8">
       <title>Home</title>
     </head>
     <body>
       <h1>Hello World!</h1>
     </body>
   </html>
   ```

7. Execute`bundle install` and  `bundle exec jekyll serve

# Host Website on Github pages

You can fellow this [offical guide](https://pages.github.com/)

# Config Domain and SSL Certificates

## Register Domain

If your serious about your blog, you must have a domain. When you move from one blog hosting platform to another, a domain ensures  the blog address remains the same. And you wouldn't lost any fellowers.

There are many domain register and management services, like [GoDaddy.com](http://godaddy.com/)

[NameCheap.com](http://namecheap.com/). They are bot top level service provider. My suggestion is search your desired domain on both, and register your domain at the more inexpensive one.

## Config DNS rules

After you registered a domain, you can config a dns service mapping your domain or subdomain to your blog. I add following DNS rules for my github pages hosting blog.

| type | host     | value |
| ----- | -------- | -------------------- |
| CNAME | blog     | tianshi25.github.io. |
| CNAME | www.blog | tianshi25.github.io. |

With this configuration, I can visit my blog [tianshi25.github.io](tianshi25.github.io) using [blog.tianshi25.com](blog.tianshi25.com) or [www.blog.tianshi25.com](www.blog.tianshi25.com). If your don't know how to find the dns configuration page, just google CNAME + your domain service provider.

## Apply SSL Certificate

You should be able to visit your blog with http protocol now. To enable https protocol for your domain, you should apply certificate. You can get ssl certiface when you register your domain. But there's usually a fee. You can get a free DV ssl ceritifate form Aliyun China. Here's the offical guide [申请免费DV证书](https://help.aliyun.com/document_detail/156645.htm?spm=a2c4g.11186623.2.2.61fa2a15lBW5UP#task-2436672) .

After your cerificate has been issued, you will be promoted to config a spcific dns record at your domain service to validate the applicateion. Wait a few sceconds, you can view visit your blog using https.

# Setup Blog

## Select Your Blog Template

There are numerous jekyll blog templates. The template I use is [HGY_blog_template](https://github.com/Gaohaoyang/gaohaoyang.github.io). This template is simple and powerful. It supports all the function you need for a blog, archives, categories, tags, table of content and about page.

On the other hand, a few functions of this website is out of date or not suitable for personal blog. I deleted the demo page, temporarily removed not working comment functions, add gemfile and removed rss. You can view source code [here](https://github.com/tianshi25/tianshi25.github.io).

## Get your FavIcon

Faveicon is the small image associated with a paticular website. Browser display them as visual remind of the site. 

![image-20210321213939965](https://raw.githubusercontent.com/tianshi25/ImageService/main/image-20210321213939965.png)

You can create a favicon from text fellowing using [Favicon Generator](https://favicon.io/favicon-generator/). Your can edit txt size, color and font on this website. When you are happy with your design, you can download the icon.

The inco generate by this web site is not transparent, which means the background color does not automatically fits the browser tab background color. If you want a transparent icon, you can use imagemagick.

```bash
# install imagemagick
brew install imaagemagick
# change background to transparent, and resize icon to favicon
convert origin.png -fuzz 10% -transparent white -resize 32x32 1.png
# -transparent the color you want to set to transparent, usually black or white
# -fuzz how much fuzz when processing images, it's useful if your icon has gradient effect
# -resize 32x32 is default favicon size
```

Now, you can replace favicon in the blog template with your own.

