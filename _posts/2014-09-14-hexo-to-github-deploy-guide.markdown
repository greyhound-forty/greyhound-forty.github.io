---
layout: post
title: "Hexo to Github: Deploy Guide"
date: 2014-09-14
---

Today we're going to look at deploying [Hexo](http://hexo.io/) via Github Pages. This will allow us to control everything on our local system and then push the changes to Github as needed. This tutorial is designed for Ubuntu 12/14 but should work on Cent/RHEL as well.

### First install NVM and npm

Our first step is to actually install node.js. For this we have a few different methods we can choose from. We can use [NVM](https://github.com/creationix/nvm) or a PPA maintained by nodesource:

#### NVM Install
The version number may be different, so be sure to check the [NVM Github page](https://github.com/creationix/nvm) for the most up to date install script:

	curl https://raw.githubusercontent.com/creationix/nvm/v0.16.1/install.sh | sh
	nvm install 0.10.29

#### PPA Install
This will probably have more up-to-date versions of Node.js than the official Ubuntu repositories. First, you need to install the PPA in order to get access to its contents:

	curl -sL https://deb.nodesource.com/setup | sudo bash -
	
The PPA will be added to your system and an `apt-get update` will be run automatically. You can then install nodejs using the command:

	sudo apt-get install nodejs

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
    git clone git@github.com:inchhighassassins/psychic-bugfixes.git ~/testing-site
	cd ~/testing-site

Now since we are publishing to Github pages we need to switch to a gh-pages branch and remove any files that were present in the repo:

	git checkout --orphan gh-pages
	git rm -rf .

Let's go ahead and initialize a new hexo setup in the current directory and make sure it is ready to go. We'll also create a test page so that we can

	hexo init .
	npm install
	npm install hexo-deployer-git --save

Create a new page and push to Github:

	hexo new "Testing Deploy"
	git add .
	git commit -a -m "First pages commit"
	git push origin gh-pages

Now we will edit the configuration file with our Github repository details. You have the option of adding a line for a specific branch you want to publish to but Hexo can figure this out automatically so I am leaving it out. From my experience if you are deploying to a main Github page (i.e. youruser.github.io) then you can leave this blank. If you are pushing to a child page like http://youruser.github.io/blog then it is best to to add the line `branch: gh-pages` under the repository section.

{% highlight bash %}
deploy:
   type: git  
   repository: git@github.com:inchhighassassins/psychic-bugfixes.git  
   branch: gh-pages  
{% endhighlight %}
	  
Now generate and deploy

	hexo generate
	hexo d

If we've set up everything correctly the output should look something like this:

{% highlight bash %}
ryan@neptune|~/psychic-bugfixes on gh-pages
± hexo d

INFO  Deploying: git
INFO  Setting up Git deployment...
Initialized empty Git repository in /home/ryan/psychic-bugfixes/.deploy_git/.git/
[master (root-commit) d3f86e9] First commit
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 placeholder
INFO  Clearing .deploy folder...
INFO  Copying files from public folder...
[master 9c897d8] Site updated: 2015-05-03 16:34:38
 30 files changed, 6050 insertions(+)
 create mode 100644 2015/05/03/Testing-Deploy/index.html
 create mode 100644 2015/05/03/hello-world/index.html
 create mode 100644 archives/2015/05/index.html
 create mode 100644 archives/2015/index.html
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
To git@github.com:inchhighassassins/psychic-bugfixes.git
 + 1ddef9d...9c897d8 master -> gh-pages (forced update)
Branch master set up to track remote branch gh-pages from git@github.com:inchhighassassins/psychic-bugfixes.git.
INFO  Deploy done: git
{% endhighlight %}

It takes about 10 minutes for Github Pages sites to show up, but you can see my test page [here](https://inchhighassassins.github.io/psychic-bugfixes/).

### Clean up
Remove .deploy folder.

	rm -rf .deploy


### Custom Domain

If you want to use a custom domain instead of using a github.io page, create a file named `CNAME` in `source` folder with the following content.

	yourcustomdomain.com

You will then need to run `hexo generate` and `hexo deploy` to update the site with the custom domain.
