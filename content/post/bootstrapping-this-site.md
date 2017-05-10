+++
banner = "banners/bootstrapping-this-site.jpg"
categories = []
date = "2017-05-09T07:59:31-05:00"
images = []
menu = ""
tags = ["howto", "hugo", "git", "github"]
title = "Bootstrapping a Website with Hugo"

+++

This is the first of a two-part series in how I built this site with [Hugo](https://gohugo.io) (a tool for generating static websites), and hosted it on [GitHub User Pages](https://pages.github.com/) (a free static site hosting service). This is a step-by-step log -- including mistakes -- of how I bootstrapped this site. If you're new to Hugo, you'll also want to read Hugo's great [documentation](https://gohugo.io/overview/introduction/).

# First Steps

### Getting Hugo

I decided to do this work on an Ubuntu box, whose version of Hugo was pretty outdated. The Hugo website has [prebuilt downloads](https://github.com/spf13/hugo/releases), though, so I installed manually instead of using apt. Your mileage will vary on this step, of course, but this was mine:

    charles@gamera ~> wget https://github.com/spf13/hugo/releases/download/v0.20.7/hugo_0.20.7_Linux-64bit.deb
    charles@gamera ~> sudo dpkg -i hugo_0.20.7_Linux-64bit.deb
    charles@gamera ~> hugo version
    Hugo Static Site Generator v0.20.7 linux/amd64 BuildDate: 2017-05-03T02:39:25-05:00

### Creating a New Hugo Site

I created a [new GitHub repo](https://github.com/new) called 'charlesk-hugo' where all the themes, markdown files, etc. will go. Then I cloned it locally:

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

The stdout from `hugo new` is right -- it's time to pick a theme. I chose [Icarus](https://github.com/digitalcraftsman/hugo-icarus-theme), whose README steps tell me to `clone` it into themes/ and then copy its config and l10n files. Since we're already working in a git repo, I used `submodule add` instead for the first step:

    charles@gamera /s/charlesk-hugo> git submodule add https://github.com/digitalcraftsman/hugo-icarus-theme.git themes/hugo-icarus-theme
    Cloning into '/home/charles/src/charlesk-hugo/themes/hugo-icarus-theme'...
    remote: Counting objects: 938, done.
    remote: Total 938 (delta 0), reused 0 (delta 0), pack-reused 938
    Receiving objects: 100% (938/938), 2.31 MiB | 0 bytes/s, done.
    Resolving deltas: 100% (608/608), done.
    charles@gamera ~/s/charlesk-hugo> ls themes/
    hugo-icarus-theme/
    charles@gamera ~/s/charlesk-hugo> cp themes/hugo-icarus-theme/exampleSite/data/l10n.toml data/
    charles@gamera ~/s/charlesk-hugo> cp themes/hugo-icarus-theme/exampleSite/config.toml config.toml 

### Editing config.toml

Icarus's README suggests updating config.toml next. Since my goal was to bootstrap a first draft as quickly as possible, I only did the quickest things: change the name and baseurl, add github/linkedin profiles, etc. I didn't have a logo or avatar yet, so I commented those lines out.

### First Post

Then I added a 'hello world' style first post so that the site will have some content:

    charles@gamera ~/s/charlesk-hugo> hugo new post/bootstrapping-this-site.md
    /home/charles/src/charlesk-hugo/content/post/bootstrapping-this-site.md created

To recap, the process so far:

1. Installed Hugo
2. Initialized a site
3. Added a theme
4. Edited config.toml
5. Added a first post

Now it's time to run Hugo in server mode to see what we've got:

    charles@gamera ~/s/charlesk-hugo> hugo server -t hugo-icarus-theme
    ERROR 2017/05/09 08:02:14 Error while rendering "home": template: theme/index.html:4:7: executing "theme/index.html" at <partial "header" .>: error calling partial: template: theme/partials/header.html:29:105: executing "theme/partials/header.html" at <absURL>: wrong number of args for absURL: want 1 got 0

### Commit in Haste, Debug at Leisure

Oops, we got an error message and Hugo is serving up an empty page. Let's see what's going on in header.html:29 that the error mentioned:

    charles@gamera ~/s/charlesk-hugo> sed -n 28,30p (find ./ -name header.html)
          <div class="profile" id="profile-nav">
            <a id="profile-anchor" href="javascript:;"><img class="avatar" src="{{ .Site.Params.avatar | absURL }}"><i class="fa fa-caret-down"></i></a>
          </div>

(*aside*: the parenthetical syntax above is a [fish](https://fishshell.com/)ism. The bash equivalent is `$(find ./ -name header.html)` or `` `find ./ -name header.html` ``)

It's breaking while trying to make the avatar link. Commenting out the avatar a few steps ago was clearly not a shortcut after all. So I fixed this by adding my avatar to the static/ directory and telling config.toml where to find it:

    charles@gamera ~/s/charlesk-hugo> mkdir -p static/images
    charles@gamera ~/s/charlesk-hugo> cp path/to/real/avatar.jpg ./static/images/avatar.jpg
    charles@gamera ~/s/charlesk-hugo> vi config.toml # editing occurs...
    charles@gamera ~/s/charlesk-hugo> grep avatar config.toml
    avatar = "images/avatar.jpg"

Now I try `hugo server -t hugo-icarus-theme` again and... success!

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

The default title was just a stripped version of the filename. Let's capitalize it properly:

    -title = "bootstrapping this site"
    +title = "Bootstrapping This Site"

and add some tags:

    -tags = []
    +tags = ["howto", "hugo", "git", "github"]


# Other Loose Ends

### Missing Social Links [PEBCAK](https://www.urbandictionary.com/define.php?term=pebkac) Error

Another problem is that none of my social links are showing up. Is it a bug in the theme? Did I break something else in config.toml? Let's see. I added links for github and linkedin. "linkedin" will be the rarer key -- Since Hugo code is often hosted on github -- so I grepped for that:

    charles@gamera ~/s/charlesk-hugo> cd themes/hugo-icarus-theme
    charles@gamera ~/s/c/t/hugo-icarus-theme> grep --files-with-matches -r linkedin *
    exampleSite/config.toml
    layouts/partials/social.html
    static/css/font-awesome.min.css
    static/fonts/fontawesome-webfont.ttf
    static/fonts/FontAwesome.otf

We already know about config.toml. The three fontawesome hits are likely related to a font that provides the linkedin logo. That conveniently leaves us one suspect, `social.html`. In that, we find:

    {{ with .Site.Social.linkedin }}
    <td><a href="//linkedin.com/in/{{.}}" target="_blank" title="LinkedIn"><i class="fa fa-linkedin"></i></a></td>
    {{ end }}

At first glance, this looks fine -- the conditional looks consistent with the .Site.Foo.bar naming seen earlier when debugging the self-induced avatar breakage. Let's step backwards and see where social.html is included from:

    charles@gamera ~/s/c/t/hugo-icarus-theme> ag "\"social\""
    layouts/partials/profile.html
    33:          {{ partial "social" . }}

That looks right, too.

Hmm, what does View Source say?

    <div class="profile-block social-links">
      <table>
        <tr>

..the social links are being generated, so why can't I see them? The answer turns out to be my adblocker:

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

This put a 'Resume' link in the top menu.

### Mind the Submodules

Important note if you're storing your content in a git repository. Since themes/hugo-icarus-theme is a submodule that points to the upstream Icarus theme, remember to use `--recursive` when cloning to get the submodule content that Hugo needs:

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

So now we have Hugo generating a good-looking blog on localhost:1313. In the next post I'll discuss getting that content onto GitHub's User Pages.
