---
layout: post
title: "Server Monitoring with Uptime and Supervisord"
date: 2014-08-28
---

### Introduction

While there are certainly no shortage of website/server monitoring utilities around these days, utilities like [Nagios](http://www.nagios.org/) or [Zabbix](http://www.zabbix.com/) are often complex to set up for new users that simply want to be alerted when their site or service is down. Today I'll be setting up [Uptime](http://redotheweb.com/uptime/), a node.js and mongodb powered monitoring application. Here are some of the benefits of Uptime:

 + Monitor thousands of websites (powered by Node.js asynchronous programming)  
 + Record availability statistics for further reporting (powered by MongoDB)  
 + Get details about failed checks (HTTP error code, etc.)
 + Complete API for integration with third-party monitoring services

If you want to see it in action you can watch a quick demo on [Vimeo](https://vimeo.com/39302164). While the video shows the install it skips some of the actual configuration steps needed to get it up and running quickly on your server such as the install of node.js and setting up mongodb.

### Installation and configuration

First we need to install the needed applications for Uptime to run properly. For this we will need some basic development packages as well as the latest version of [Node.js](http://nodejs.org/):

	apt-get update
	apt-get install gcc make g++ git python-software-properties -y
	apt-add-repository ppa:chris-lea/node.js -y
	apt-get update
	apt-get install nodejs -y

Additionally you'll need MongoDB. The one that comes with the default install of Ubuntu 12 seemed woefully out of date so I installed the latest version using the following:

	apt-key adv --keyserver keyserver.ubuntu.com --recv 7F0CEB10
	echo "deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen" | tee -a /etc/apt/sources.list.d/10gen.list
	apt-get -y update
	apt-get -y install mongodb-10gen

I am putting my uptime app in /var/www so you may need to adjust your commands depending on the path that you want to run it from. Let's clone the uptime project to our system and install it.

	cd /var/www
	git clone git://github.com/fzaninotto/uptime.git
	cd uptime
	npm install

If we attempt to start Uptime now it will fail out because we need to create a MongoDB user and assign it a password and role. For this guide I am using MongoDB 2.4. If you are using 2.6 see [this guide](http://docs.mongodb.org/manual/tutorial/add-user-to-database/) for how to create a new user as the syntax has changed. To drop in to the mongodb shell just type in `mongo` and then switch to the uptime database that was created when you installed the app:

	use uptime
	db.addUser({ user:"uptime", pwd:"gallifrey", roles:["readWrite"] })

Now it is time to make sure that our user is properly assigned:

```
[root@hyperion ~ ] # mongo  
MongoDB shell version: 2.4.9  
connecting to: test  
> use uptime  
switched to db uptime  
> show users  
{  
        "_id" : ObjectId("53ece0a0a33f66451a56e86c"),  
        "user" : "uptime",  
        "pwd" : "51b70f4d284218eb962c10b8269c352d",  
        "roles" : [  
                "readWrite"  
        ]  
}  
> exit  
bye  
```

Now that we have the user created, we need to adjst the configuration file `/var/www/uptime/config/default.yaml` with the username and password. Here is what the updated mongodb part of the configuration file will look like:

```
url: 'http://localhost:8082'

mongodb:
  server:   localhost
  database: uptime
  user:     uptime
  password: gallifrey
  connectionString:       # alternative to setting server, database, user and password separately
```

By default uptime is not secure, and anyone that can get to your hostname or IP can see your healthchecks. To get around this let's update the authorization section as follows (obviously you should use a noce secure password and not tardis):

```
basicAuth:
  username:    uptime_admin
  password:    tardis
```

After we make this change we also need to uncomment the basicAuth plugin line in the default.yaml


```
[hyperion ~ ] #grep 'plugins/basicAuth' /var/www/uptime/config/default.yaml
  # - ./plugins/basicAuth

[hyperion ~ ] # sed -i 's|# - ./plugins/basicAuth|- ./plugins/basicAuth|' /var/www/uptime/config/default.yaml

[hyperion ~ ] # grep 'plugins/basicAuth' /var/www/uptime/config/default.yaml
  - ./plugins/basicAuth

```

It is time to test that we've got everything set up correctly by starting the app:

	cd /var/www/uptime/
	node app.js

If everything starts as expected you will get a lot of console output. You should now visit `http://x.x.x.x:8082`, where x.x.x.x is the IP of the server you are running this on. If you are testing this on a local system you can also use `http://localhost:8082`


{<1>}![Uptime Initial Screen](https://dl.dropboxusercontent.com/u/131462/Images/uptime1.png)

You can now start setting up your uptime checks. I'm not going to go over that aspect of the process here, but you can find more information on the [Uptime Project Page](http://redotheweb.com/uptime/), the [Uptime Github page](https://github.com/fzaninotto/uptime) or this [Google Group](https://groups.google.com/forum/#!forum/node-uptime) dedicated to uptime.

### Adding process control

If we don't fork the process to the background it will end as soon as we terminate our session or reboot the machine. This is where Supervisord comes in. [Supervisord](http://supervisord.org/) is a process monitoring control system that allows for the starting of processes at boot and can also restart services if they die unexpectedly.

Any files found in /etc/supervisor/conf.d ending in .conf will be read by Supervisor. Use your favorite text editor and create a uptime.conf file that has the following:


```
[program:uptime]
command = node /var/www/uptime/app.js
directory = /var/www/uptime
user = root
autostart = true
autorestart = true
stdout_logfile = /var/log/supervisor/uptime.log
stderr_logfile = /var/log/supervisor/uptime_err.log
```

We now need to reread the supervisor conf.d directory, update the process monitor, and start our service:

```
[hyperion ~ ] # supervisorctl reread
uptime: available

[hyperion ~ ] # supervisorctl update
uptime: added process group

[hyperion ~ ] # supervisorctl start uptime
uptime: started
```


If all goes well you can again visit `http://x.x.x.x:8082` or `http://localhost:8082` and start setting our health checks. Happy Monitoring!!
	
