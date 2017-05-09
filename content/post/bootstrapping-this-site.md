+++
banner = ""
categories = []
date = "2017-05-09T07:59:31-05:00"
description = ""
images = []
menu = ""
tags = []
title = "bootstrapping this site"

+++

https://github.com/spf13/hugo/releases
https://github.com/spf13/hugo/releases/download/v0.20.7/hugo_0.20.7_Linux-64bit.deb
sudo dpgk -i hugo_0.20.7_Linux-64bit.deb

    charles@gamera ~/s/charlesk-hugo> hugo version
    Hugo Static Site Generator v0.20.7 linux/amd64 BuildDate: 2017-05-03T02:39:25-05:00

Create a repo that will hold the Hugo content
https://github.com/new
I called mine "charlesk-hugo"
Clone it locally

    charles@gamera ~/src> git clone git@github.com:charlesk/charlesk-hugo.git
    Cloning into 'charlesk-hugo'...
    remote: Counting objects: 3, done.
    remote: Compressing objects: 100% (2/2), done.
    remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
    Receiving objects: 100% (3/3), done.

Now we have a nearly-empty directory to put content in:

    charles@gamera ~/src> cd charlesk-hugo/
    charles@gamera ~/s/charlesk-hugo> ls
    README.md

Let's init the Hugo content here.
Since that's the sole point of this repo, we init the site in the toplevel directory:

    charles@gamera ~/s/charlesk-hugo> hugo new site . --force
    Congratulations! Your new Hugo site is created in /home/charles/src/charlesk-hugo.

    Just a few more steps and you're ready to go:

    1. Download a theme into the same-named folder.
       Choose a theme from https://themes.gohugo.io/, or
       create your own with the "hugo new theme <THEMENAME>" command.
    2. Perhaps you want to add some content. You can add single files
       with "hugo new <SECTIONNAME>/<FILENAME>.<FORMAT>".
    3. Start the built-in live server via "hugo server".

    Visit https://gohugo.io/ for quickstart guide and full documentation.

Now, just like the output of {{{hugo new}}} said, it's time to pick a theme.
I chose [Icarus](https://github.com/digitalcraftsman/hugo-icarus-theme)
which comes with its own README, so I follow that here:

    charles@gamera ~/s/charlesk-hugo> cd themes/
    charles@gamera ~/s/c/themes> git clone https://github.com/digitalcraftsman/hugo-icarus-theme.git
    Cloning into 'hugo-icarus-theme'...
    remote: Counting objects: 938, done.
    remote: Total 938 (delta 0), reused 0 (delta 0), pack-reused 938
    Receiving objects: 100% (938/938), 2.31 MiB | 0 bytes/s, done.
    Resolving deltas: 100% (608/608), done.

    charles@gamera ~/s/c/themes> cd ..
    charles@gamera ~/s/charlesk-hugo> cp themes/hugo-icarus-theme/exampleSite/data/l10n.toml data/
    charles@gamera ~/s/charlesk-hugo> cp themes/hugo-icarus-theme/exampleSite/config.toml config.toml 
    charles@gamera ~/s/charlesk-hugo> vi config.toml

Add a first post:

    charles@gamera ~/s/charlesk-hugo> hugo new post/bootstrapping-this-site.md
    /home/charles/src/charlesk-hugo/content/post/bootstrapping-this-site.md created

Now we get to see what the rough draft will look like:

    charles@gamera ~/s/charlesk-hugo> hugo server --buildDrafts -t hugo-icarus-theme
    ERROR 2017/05/09 08:02:14 Error while rendering "home": template: theme/index.html:4:7: executing "theme/index.html" at <partial "header" .>: error calling partial: template: theme/partials/header.html:29:105: executing "theme/partials/header.html" at <absURL>: wrong number of args for absURL: want 1 got 0

Oops, an empty page. Let's look at that error:

    charles@gamera ~/s/charlesk-hugo> find ./ -name header.html
    ./themes/hugo-icarus-theme/layouts/partials/header.html
    charles@gamera ~/s/charlesk-hugo> view ./themes/hugo-icarus-theme/layouts/partials/header.html

Line 29 there reads:

    <a id="profile-anchor" href="javascript:;"><img class="avatar" src="{{ .Site.Params.avatar | absURL }}"><i class="fa fa-caret-down"></i></a>

So it looks like we can't just comment out the avatar. Let's set it to an empty string for now:

    --- a/config.toml
    +++ b/config.toml
    @@ -21,9 +21,9 @@ theme = "hugo-icarus-theme"
         #bio = "Programmer"
         location = "New Orleans"
         site_description = ""
    -    #copyright  = "Powered by [Hugo](//gohugo.io). Theme by [PPOffice](http://github.com/ppoffice)."
    -    #avatar = "css/images/avatar.png"
    -    #logo = "css/images/logo.png"
    +    copyright = "" # "Powered by [Hugo](//gohugo.io). Theme by [PPOffice](http://github.com/ppoffice)."
    +    avatar = "" # "css/images/avatar.png"
    +    logo = "" # "css/images/logo.png"
         disable_mathjax = false # set to true to disable MathJax
 
         # Format dates with Go's time formatting

Now we restart {{{hugo server --buildDrafts -t hugo-icarus-theme}}} and try again.

