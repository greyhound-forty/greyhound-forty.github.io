---
layout: post
title: "Hexo to Github: Deploy Guide"
date: 2014-09-14
---

Today we're going to look at deploying [Hexo](http://hexo.io/) via Github Pages. This will allow us to control everything on our local system and then push the changes to Github as needed. This tutorial is designed for Ubuntu 12/14 but should work on Cent/RHEL as well.

### First install NVM and npm

Our first step is to actually install node.js, but we are going to be using [NVM](https://github.com/creationix/nvm) to help us avoid permissions issues with a default install of node. For this guide we are installing node v0.10.29:

	curl https://raw.githubusercontent.com/creationix/nvm/v0.11.1/install.sh | bash
	nvm install 0.10.29


Let's check our installed version:

    » node -v
	v0.10.29



Now we will install hexo using NPM:

	npm install hexo -g

To test that hexo is installed properly we'll create a test blog and start the server:


	ryan@ghost ~ % hexo init testing-site
    [info] Copying data
	[info] You are almost done! Don't forget to run `npm install` before you start blogging with Hexo!

	ryan@ghost ~ % cd testing-site
	ryan@ghost ~/testing-site % npm install
    <output output output>

	ryan@ghost ~/testing-site % hexo server
	[info] Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.

If all went well we can now visit http://localhost:4000/ to see the site up and running. While we could use something like the rather awesome [ngrok](https://ngrok.com/usage) to expose our locally hosted version to the internet for testing and the like, if we want to make this a fully functional, rock solid site we can also deploy it to Github Pages.

### Set up a new repository on Github

If you are new to Github, here is a guide on creating a new Repository: [Create a Repo](https://help.github.com/articles/create-a-repo). I would also highly recommend that you have SSH-keys set up in Github so that you do not need to enter in your username and password everytime you run `hexo deploy`. For setting up SSH keys, see [this guide](https://help.github.com/articles/generating-ssh-keys)

After we've set up our new repo we'll head back to our local system and clone our newly created repo:

	rm -rf ~/testing-site
    git clone git@github.com:greyhound-forty/ballin-octo-batman.git ~/testing-site
	cd ~/testing-site

Now since we are publishing to Github pages we need to switch to a gh-pages branch and remove any files that were present in the repo:

	git checkout --orphan gh-pages
	git rm -rf .

Let's go ahead and initialize a new hexo setup in the current directory and make sure it is ready to go. We'll also create a test page so that we can

	hexo init .
	npm install

Create a new page and push to Github:

	hexo new "Testing Deploy"
	git add .
	git commit -a -m "First pages commit"
	git push origin gh-pages

Now we will edit the configuration file with our Github repository details. You have the option of adding a line for a specific branch you want to publish to but Hexo can figure this out automatically so I am leaving it out. From my experience if you are deploying to a main Github page (i.e. youruser.github.io) then you can leave this blank. If you are pushing to a child page like http://youruser.github.io/blog then it is best to to add the line `branch: gh-pages` under the repository section.

```
deploy:
  type: github
  repository: git@github.com:greyhound-forty/fuzzy-octo-meme.git
```

Now generate and deploy

	hexo generate
	hexo deploy

If we've set up everything correctly the output should look something like this:

```
ryan@ghost ~/testing-site [gh-pages *] % hexo generate
[info] Files loaded in 0.136s
[create] Generated: archives/index.html (74ms)
[create] Generated: archives/2014/index.html (21ms)
[create] Generated: archives/2014/08/index.html (15ms)
[create] Generated: index.html (13ms)
[create] Generated: 2014/08/25/testing-deploy/index.html (13ms)
[create] Generated: 2014/08/25/hello-world/index.html (18ms)
[create] Generated: js/script.js (2ms)
[create] Generated: css/style.css (691ms)
[create] Generated: css/fonts/FontAwesome.otf (2ms)
[create] Generated: css/fonts/fontawesome-webfont.eot (1ms)
[create] Generated: css/fonts/fontawesome-webfont.svg (1ms)
[create] Generated: css/fonts/fontawesome-webfont.ttf (1ms)
[create] Generated: css/fonts/fontawesome-webfont.woff (1ms)
[create] Generated: css/images/banner.jpg (2ms)
[create] Generated: fancybox/blank.gif (1ms)
[create] Generated: fancybox/fancybox_loading.gif (1ms)
[create] Generated: fancybox/fancybox_loading@2x.gif (1ms)
[create] Generated: fancybox/fancybox_overlay.png (1ms)
[create] Generated: fancybox/fancybox_sprite.png (1ms)
[create] Generated: fancybox/fancybox_sprite@2x.png (1ms)
[create] Generated: fancybox/jquery.fancybox.css (1ms)
[create] Generated: fancybox/jquery.fancybox.js (1ms)
[create] Generated: fancybox/jquery.fancybox.pack.js (1ms)
[create] Generated: fancybox/helpers/fancybox_buttons.png (1ms)
[create] Generated: fancybox/helpers/jquery.fancybox-buttons.css (1ms)
[create] Generated: fancybox/helpers/jquery.fancybox-buttons.js (1ms)
[create] Generated: fancybox/helpers/jquery.fancybox-media.js (2ms)
[create] Generated: fancybox/helpers/jquery.fancybox-thumbs.css (0ms)
[create] Generated: fancybox/helpers/jquery.fancybox-thumbs.js (0ms)
[info] 29 files generated in 0.884s

ryan@ghost ~/testing-site [gh-pages *] % hexo deploy
[info] Start deploying: github
[info] Setting up GitHub deployment...
Initialized empty Git repository in /home/ryan/testing-site/.deploy/.git/
[master (root-commit) b62505d] First commit
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 placeholder
[info] Clearing .deploy folder...
[info] Copying files from public folder...
[gh-pages b1df09b] Site updated: 2014-08-26 00:00:55
 30 files changed, 6207 insertions(+)
 create mode 100644 2014/08/25/hello-world/index.html
 create mode 100644 2014/08/25/testing-deploy/index.html
 create mode 100644 archives/2014/08/index.html
 create mode 100644 archives/2014/index.html
 create mode 100644 archives/index.html
 create mode 100644 css/fonts/FontAwesome.otf
 create mode 100644 css/fonts/fontawesome-webfont.eot
 create mode 100644 css/fonts/fontawesome-webfont.svg
 create mode 100644 css/fonts/fontawesome-webfont.ttf
 create mode 100644 css/fonts/fontawesome-webfont.woff
 create mode 100644 css/images/banner.jpg
 create mode 100644 css/style.css
 create mode 100644 fancybox/blank.gif
 create mode 100644 fancybox/fancybox_loading.gif
 create mode 100644 fancybox/fancybox_loading@2x.gif
 create mode 100644 fancybox/fancybox_overlay.png
 create mode 100644 fancybox/fancybox_sprite.png
 create mode 100644 fancybox/fancybox_sprite@2x.png
 create mode 100644 fancybox/helpers/fancybox_buttons.png
 create mode 100644 fancybox/helpers/jquery.fancybox-buttons.css
 create mode 100644 fancybox/helpers/jquery.fancybox-buttons.js
 create mode 100644 fancybox/helpers/jquery.fancybox-media.js
 create mode 100644 fancybox/helpers/jquery.fancybox-thumbs.css
 create mode 100644 fancybox/helpers/jquery.fancybox-thumbs.js
 create mode 100644 fancybox/jquery.fancybox.css
 create mode 100644 fancybox/jquery.fancybox.js
 create mode 100644 fancybox/jquery.fancybox.pack.js
 create mode 100644 index.html
 create mode 100644 js/script.js
 delete mode 100644 placeholder
Warning: Permanently added the RSA host key for IP address 'x.x.x.x' to the list of known hosts.
To git@github.com:greyhound-forty/ballin-octo-batman.git
 + e6fe5eb...b1df09b gh-pages -> gh-pages (forced update)
Branch gh-pages set up to track remote branch gh-pages from git@github.com:greyhound-forty/ballin-octo-batman.git.
[info] Deploy done: github
```

It takes about 10 minutes for Github Pages sites to show up, but you can see my test page [here](http://greyhound-forty.github.io/ballin-octo-batman).

### Clean up
Remove .deploy folder.

{% highlight bash %}
	rm -rf .deploy
{% endhighlight %}

### Custom Domain

If you want to use a custom domain instead of using a github.io page, create a file named `CNAME` in `source` folder with the following content.

	yourcustomdomain.com

You will then need to run `hexo generate` and `hexo deploy` to update the site with the custom domain.
