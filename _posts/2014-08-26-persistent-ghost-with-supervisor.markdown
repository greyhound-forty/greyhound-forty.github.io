---
layout: post
title: "Persistent Ghost with Supervisor"
date: "2014-08-26 09:45"
---

I love trying out new blogging services and while they are usually maintained for only a short period of time that is more of a failing on my part and less on the actual technology. My current favorite for self-hosted blogging is [Ghost](https://ghost.org/). By default if you start Ghost using the command line it will stop whenever you are closing the terminal window or log out from your current SSH session. To get around this limitation we will be using [Supervisor](http://supervisord.org/). In this guide I am using an Ubuntu 14 server but this has been tested on 12 and works without issue. Now that we have the diatribe out of the way, let's begin:

Our first step is of course to make sure that our system is up to date and to install the supervisor package

	apt-get update
    apt-get install supervisor -y


Now that we have installed Supervisor we need to make sure it starts.

	service supervisor start

If all goes well you should have Supervisor up and running. The main configuration file for Supervisor is located at `/etc/supervisor/supervisord.conf` but Supervisor also includes a conf.d directory to store the individual program files:

	[include]
	files = /etc/supervisor/conf.d/*.conf


Any files found in `/etc/supervisor/conf.d` ending in .conf will be read by Supervisor. Use your favorite text editor and create a ghost.conf:


	nano -w /etc/supervisor/conf.d/ghost.conf

Here is my ghost.conf:

{% highlight bash %}
[program:ghost]
command = node /var/ghost/index.js
directory = /var/ghost
user = ghost
autostart = true
autorestart = true
stdout_logfile = /var/log/supervisor/ghost.log
stderr_logfile = /var/log/supervisor/ghost_err.log
environment = NODE_ENV="production"
{% endhighlight %}

As you can see from the file I have ghost running from `/var/ghost` so you'll need to adjust this to fit your specific needs if you have ghost running from another directory

Now we need to reread the `/etc/supervisor/conf.d` directory to pick up the new configuration file and update the Supervisor process monitor

	supervisorctl reread
	supervisorctl update

Once the service reloads let's go ahead and start up the ghost script

	supervisorctl start ghost

You can check the status of any running supervisor script by executing `supervisorctl status`

	root@ghost:~# supervisorctl status
	ghost                            RUNNING    pid 7324, uptime 0:06:38

