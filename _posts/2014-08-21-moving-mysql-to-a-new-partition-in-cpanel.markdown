---
layout: post
title: "Moving Mysql to a new partition in cPanel"
date: "2014-08-21 08:25"
---

I've had to deal with quiet a few clients who have filled up their /var partition due to mysql being stored by default on a small partition. This is a short little guide on how to move mysql to another parition on the system.

The nice thing about this method is that it requires no mucking with the my.cnf file. Additionally if the stock-default MySQL data directory is retained via a symlink, this still allows for the directory contents to be saved when WHM runs its backup utility.

First we need to tell cPanel not to monitor/manage Mysql so that it does not try to restart the service while we are moving our data. To do this log in to WHM >> Main >> Service Configuration >> Service Manager and disable mysql. Once that is complete here are the commands to safely move Mysql to a new, larger partition (in this example I am moving mysql to the mount point /newvar):

`service mysql stop  
mkdir -p /newvar  
mount /dev/xvdc1 /newvar  
cp -r /var/lib/mysql/ /newvar/  
mkdir -p /newvar/mysql/backupmysql/  
mv -v /var/lib/mysql/* /newvar/mysql/backupmysql/  
chown -R mysql:mysql /newvar/mysql/  
rm -rf /var/lib/mysql/  
ln -s /newvar/mysql /var/lib/mysql  
rm -rf /tmp/mysql.sock  
ln -s /newvar/mysql/mysql.sock /tmp/mysql.sock`

Now that we've moved everything we need to log back in to WHM and re-enable mysql. To do this we log in and go to Main >> Service Configuration >> Service Manager and enable mysql. Once the service is re-enabled let's test it to make sure everything starts back up properly. You can do so with the following command:

`/scripts/restartsrv mysql`
