+++
banner = "banner.jpg"
categories = []
date = "2017-05-09T11:50:43-05:00"
description = ""
images = []
menu = ""
tags = ["howto", "hugo", "git", "github"]
title = "Hosting my Hugo site on a Personal GitHub Page"

+++

This is the second half of my "Bootstrapping This Site" series. You may want to check out [the first half](fixme) first.

At the end of the last post, I had this site rendering in `hugo server` mode so that I could read it from localhost. In this second article, I discuss how to get that content onto http://charleskerr.com/ .

Registrar
=========

As the GitHub Pages [custom domain docs](https://help.github.com/articles/using-a-custom-domain-with-github-pages/) say, I need a CNAME file in the toplevel. [Hugo's docs](https://gohugo.io/tutorials/github-pages-blog/) say to add that CNAME to the `static` directory, so I did that:

    charles@gamera ~/s/charlesk-hugo> echo "charleskerr.com" > static/CNAME
    charles@gamera ~/s/charlesk-hugo> git add static/CNAME
    charles@gamera ~/s/charlesk-hugo> git commit -m "add CNAME for github pages custom domain"

I set this up with [Namecheap](https://namecheap.com) as my registrar, so after 
As [David Singer](http://davidensinger.com/2013/03/setting-the-dns-for-github-pages-on-namecheap/) writes, 

![Advanced DNS](advanced-dns.png "A custom domain at namecheap, configured for GitHub Pages")
