+++
banner = "banners/bootstrapping-this-site.jpg"
categories = []
date = "2017-05-09T07:59:31-05:00"
description = "I built this site with Hugo. This is how I did it."
images = []
menu = ""
tags = ["hugo", "howto", "github"]
title = "Bootstrapping This Site"

+++

These are the steps I followed to build this site with Hugo.

# First Steps

### Getting Hugo

The version of Hugo in my Debian repos was a little old, and the Hugo website has [prebuilt downloads](https://github.com/spf13/hugo/releases) page, so I installed manually instead of using apt. YMMV on this step, naturally.

    charles@gamera ~> sudo dpkg -i hugo_0.20.7_Linux-64bit.deb
    charles@gamera ~> hugo version
    Hugo Static Site Generator v0.20.7 linux/amd64 BuildDate: 2017-05-03T02:39:25-05:00

### Creating a New Hugo Site

Since I wanted to host this content on github, I created a [new repo](https://github.com/new) there called 'charlesk-hugo' and clone it locally:

    charles@gamera ~/src> git clone git@github.com:charlesk/charlesk-hugo.git
    Cloning into 'charlesk-hugo'...
    remote: Counting objects: 3, done.
    remote: Compressing objects: 100% (2/2), done.
    remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
    Receiving objects: 100% (3/3), done.
    charles@gamera ~/src> cd charlesk-hugo/
    charles@gamera ~/s/charlesk-hugo> ls
    README.md

The next step was to have Hugo initialize the site. That's the sole purpose of the repo, so I used the toplevel repo directory:

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

### Installing a Theme

The stdout from `hugo new` is right -- it's time to pick a theme. I chose [Icarus](https://github.com/digitalcraftsman/hugo-icarus-theme), whose README steps tell me to clone it into themes/ and then copy its config and l10n files:

    charles@gamera /t/charlesk-hugo> git submodule add https://github.com/digitalcraftsman/hugo-icarus-theme.git themes/hugo-icarus-theme
    Cloning into '/tmp/charlesk-hugo/themes/hugo-icarus-theme'...
    remote: Counting objects: 938, done.
    remote: Total 938 (delta 0), reused 0 (delta 0), pack-reused 938
    Receiving objects: 100% (938/938), 2.31 MiB | 0 bytes/s, done.
    Resolving deltas: 100% (608/608), done.
    charles@gamera /t/charlesk-hugo> ls themes/
    hugo-icarus-theme/
    charles@gamera ~/s/charlesk-hugo> cp themes/hugo-icarus-theme/exampleSite/data/l10n.toml data/
    charles@gamera ~/s/charlesk-hugo> cp themes/hugo-icarus-theme/exampleSite/config.toml config.toml 

### Editing config.toml

Icarus next suggests poking around in config.toml. Since the goal is to bootstrap a first draft as quickly as possible, I only did the quickest things: change the name and baseurl, add github/linkedin profiles, etc. I don't have a logo or avatar yet, so I commented those pieces out.

### First Post

To round it off, I added a first post so that the site will have something to show:

    charles@gamera ~/s/charlesk-hugo> hugo new post/bootstrapping-this-site.md
    /home/charles/src/charlesk-hugo/content/post/bootstrapping-this-site.md created

That article may look familiar... because it's what you're reading now. :)

To recap the story so far:

1. Installed Hugo
2. Initialized a site
3. Installed a theme
4. Added a first post

Time to see what the output looks like, by running Hugo in server mode:

    charles@gamera ~/s/charlesk-hugo> hugo server --buildDrafts -t hugo-icarus-theme
    ERROR 2017/05/09 08:02:14 Error while rendering "home": template: theme/index.html:4:7: executing "theme/index.html" at <partial "header" .>: error calling partial: template: theme/partials/header.html:29:105: executing "theme/partials/header.html" at <absURL>: wrong number of args for absURL: want 1 got 0

### Commit in Haste, Debug at Leisure

Oops, no content. An empty page. Let's see what's going on in header.html:29 that the error mentioned:

    charles@gamera ~/s/charlesk-hugo> sed -n 28,30p (find ./ -name header.html)
          <div class="profile" id="profile-nav">
            <a id="profile-anchor" href="javascript:;"><img class="avatar" src="{{ .Site.Params.avatar | absURL }}"><i class="fa fa-caret-down"></i></a>
          </div>

(NB: the parenthetical syntax above might look confusing if you're not using [fish](https://fishshell.com/). The bashism version is `$(find ./ -name header.html)` or `` `find ./ -name header.html` ``)

Clearly, commenting out the avatar a few steps ago was not such a shortcut after all. I'll revert config.toml to restore the default location (`css/images/avatar.png`) and put my avatar there.

    charles@gamera ~/s/charlesk-hugo> find ./ -name avatar.png
    ./themes/hugo-icarus-theme/static/css/images/avatar.png
    charles@gamera ~> cp path/to/real/avatar.png ./themes/hugo-icarus-theme/static/css/images/avatar.png

Now I try `hugo server --buildDrafts -t hugo-icarus-theme` again and... success!

# Finishing the First Post

### Adding a Banner

Now that the website is loading, the first post is showing some loose ends: there's no banner, no tags, and the title is lowercase. To fix the first problem, I looked for a "hello world" banner on Google images and filtered by free-for-reuse licensing, then saved it to the static `banners` folder recommended by Icarus:

    charles@gamera ~/s/charlesk-hugo> mkdir -p static/banners
    charles@gamera ~/s/charlesk-hugo> cd static/banners
    charles@gamera ~/s/c/s/banners> wget -O bootstrapping-this-site.jpg https://static.pexels.com/photos/106582/pexels-photo-106582.jpeg

...and tell content/posts/bootstrapping-this-site.md about it:

    @@ -1,5 +1,5 @@
     +++
    -banner = ""
    +banner = "banners/bootstrapping-this-site.jpg"
     categories = []

### Cleaning up the Front Matter

The default title was just a stripped version of the filename. Let's tweak this to capitalize it properly

    -title = "bootstrapping this site"
    +title = "Bootstrapping This Site"

and to add some tags:

    -tags = []
    +tags = ["hugo", "github", "howto"]


# Other Loose Ends

### Missing Social Links [PEBCAK](https://www.urbandictionary.com/define.php?term=pebkac) Error

The first thing I notice is that none of my social links appeared! Is it a bug in the theme? Did I edit something else wrong in config.toml? Let's see what's going on. I added two social sites, github and linkedin. Since hugo themes are often hosted on github, "linkedin" will be the rarer key. Let's search for that:

    charles@gamera ~/s/c/t/hugo-icarus-theme> grep --files-with-matches -r linkedin *
    exampleSite/config.toml
    layouts/partials/social.html
    static/css/font-awesome.min.css
    static/fonts/fontawesome-webfont.ttf
    static/fonts/FontAwesome.otf

We already know about config.tml, and the three fontawesome hits are probably some font that includes the linkedin logo. That conveniently leaves one suspect, `social.html`. Looking at that, we find:

    {{ with .Site.Social.linkedin }}
    <td><a href="//linkedin.com/in/{{.}}" target="_blank" title="LinkedIn"><i class="fa fa-linkedin"></i></a></td>
    {{ end }}

This looks consistent with the .Site.Foo.bar nomenclature we found earlier when debugging the no-avatar-breaks-page issue above. At first glance, this looks fine. Let's walk backwards and see where social.html is included from:

    charles@gamera ~/s/c/t/hugo-icarus-theme> ag "\"social\""
    layouts/partials/profile.html
    33:          {{ partial "social" . }}

That looks right, too. Hmm, what if I do what I should've done in the first place and view source?

    <div class="profile-block social-links">
      <table>
        <tr>

.. so the social links are, actually, being generated. Why can't I see them?

The answer turns out to be my adblocker:

    Static filter ##.social-links found in:

        Fanboy’s Annoyance List
        Fanboy’s Social Blocking List

So I disable adblocking for localhost, reload, and... the links are "fixed"!

### Adding Static Content

Lastly, I added some static content: [my resume](http://www.charleskerr.com/resume/).

    charles@gamera ~/s/charlesk-hugo> cp -R path/to/resume static/

and add these lines to config.toml:
 
     [[params.menu]]
         before = false
    +    label  = "Resume"
    +    link   = "resume/"
    +
    +[[params.menu]]
    +    before = false
         label  = "Tags"
         link   = "tags/"

This put a 'resume' link in the top menu.

### Future clones

If you clone my repo, you may get a blank page served to you because the themes/hugo-icarus-theme/ folder is empty. Remember, since that's a submodule, we need to clone recursively:

    charles@gamera ~/src> git clone --recursive git@github.com:charlesk/charlesk-hugo.git
    Cloning into 'charlesk-hugo'...
    remote: Counting objects: 73, done.
    remote: Compressing objects: 100% (45/45), done.
    remote: Total 73 (delta 19), reused 70 (delta 19), pack-reused 0
    Receiving objects: 100% (73/73), 3.89 MiB | 5.70 MiB/s, done.
    Resolving deltas: 100% (19/19), done.
    Submodule 'themes/hugo-icarus-theme' (https://github.com/digitalcraftsman/hugo-icarus-theme.git) registered for path 'themes/hugo-icarus-theme'
    Cloning into '/home/charles/src/charlesk-hugo/themes/hugo-icarus-theme'...
    remote: Counting objects: 938, done.        
    remote: Total 938 (delta 0), reused 0 (delta 0), pack-reused 938        
    Receiving objects: 100% (938/938), 2.31 MiB | 3.85 MiB/s, done.
    Resolving deltas: 100% (608/608), done.
    Submodule path 'themes/hugo-icarus-theme': checked out '855e14be1ceabc65a7de8a16b8c1717157d9bef9'

Now the content will work again.


And that's it! Those are the steps I followed to get this site started using Hugo and the Icarus theme. In the next post I'll discuss hosting the static content on Github's Personal Pages.
