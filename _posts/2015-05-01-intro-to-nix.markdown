---
layout: post
title:  "Introduction to *Nix: Part 1"
date:   2015-05-01 09:49:55
categories: jekyll update
---

This tutorial (Introduction to *Nix: Part 1) is for those who are new to Linux, or those who want to review their understanding of fundamental Linux concepts and commands. For beginners, a lot of this material will be new. For more experienced users you can use this as a refresher or jump ahead to [Part 2]() or [Part 3]()

Why does this say *nix? Well I am not going to go down the rabbit hole of the Unix vs Linux discussion so a quick explanation of both:

> Unix: a family of multitasking, multiuser computer operating systems that derive from the original AT&T Unix, developed in the 1970s at the Bell Labs research center by Ken Thompson, Dennis Ritchie, and others  

> Linux: a Unix-like and mostly POSIX-compliant computer operating system assembled under the model of free and open-source software development and distribution [whose] defining component ... is the Linux kernel, an operating system kernel first released [in] 1991 by Linus Torvalds ↑

### Basic directory structure

{% highlight bash %}
/
Also know as "slash" or the root partition

/bin
Common programs, shared by the system, the system administrator and the users.

/boot
Boot files, boot loader (grub), kernels, vmlinuz

/dev
Contains references to system devices, files with special properties.

/etc
Important system and application configuration files.

/home
Home directories for system users.

/lib
Library files, includes files for all kinds of programs needed by the system and the users.

/lost+found
Files that were saved during failures are here.

/mnt
Standard mount point for external file systems.

/media
Mount point for external file systems (on some distros).

/net
Standard mount point for entire remote file systems - nfs.

/opt
Typically contains extra and third party software.

/proc
A virtual file system containing information about system resources.

/root
root users home dir.

/sbin
Programs for use by the system and the system administrator.

/tmp
Temporary space for use by the system, cleaned upon reboot.

/usr
Programs, libraries, documentation etc. for all user-related programs.

/var
Storage for all variable files and temporary files created by users, such as log files, mail queue, print spooler. Web servers, Databases etc.
{% endhighlight %}

### Basic shell commands

While I could list a few dozen commands that I use everyday I think I'll keep this intro section to the top 20 or so most useful for beginners or people looking to refresh their shell-fu. Here are the commands we will be looking at in this section:

+ [pwd](#pwd)
+ [ls](#ls)
+ [cd](#cd)
+ [mkdir](#mkdir)
+ [echo](#echo)
+ [cat](#cat)
+ [mv](#mv)
+ [cp](#cp)
+ [rm](#rm)
+ [man](#man)
+ [less](#less)
+ [head](#head)
+ [tail](#tail)
+ [grep](#grep)
+ [which](#which)
+ [history](#history)
+ [chown](#chown)
+ [chmod](#chmod)
+ [find](#find)
+ [locate](#locate)
+ [clear](#clear)
+ [stat](#stat)
+ [file](#file)

### pwd
To see bash's current working directory, you can type `pwd`

{% highlight bash %}
ryan@bebop [homebase] (master)$ pwd
/home/ryan/homebase

tinybot± pwd
/home/ryan/tinybot
{% endhighlight %}

#### Absolute Path
Alternatively referred to as the file path and full path, the absolute path contains the root directory and all other subdirectories that contain a file or folder. As you can see from the examples below, the one thing that all absolute paths have in common is that they begin with /. With a path of /var/www/html, we're telling cd to enter the / directory, then the var directory under that, and then www and html respectively. In other words we can say absolute path is a complete path from start of actual filesystem from the / directory.

{% highlight bash %}
/usr/local/bin
/var/www/html
/home/ryan
{% endhighlight %}

#### Relative Path
The other kind of path is called a relative path. bash, cd and other commands always interpret these paths relative to the current directory. Relative paths never begin with a /. So, if we're in /usr we would issue a cd to bin to get to /usr/bin. If we issued the command `cd /bin` we would end up in /bin and not /usr/bin. Relative paths may also contain one or more `..` directories. The `..` directory is a special directory that points to the parent directory.

{% highlight bash %}
[<*>] $ pwd
/usr/local

[<*>] $ cd bin
[<*>] $ pwd
/usr/local/bin

[<*>] $ cd /usr/local
[<*>] $ cd /bin
[<*>] $ pwd
/bin

[<*>] $ cd /usr/local
[<*>] $ cd ../share
/usr/share

[<*>] $ cd /usr/local/bin
[<*>] $ cd ..
[<*>] $ pwd
/usr/local
{% endhighlight %}

### ls
Lists the contents of a directory. Sort entries alphabetically if no flags are passed.

{% highlight bash %}
[<*>]  things  ls -l |wc -l
10
[<*>]  things  ls -la |wc -l
13
[<*>]  things  ls -la
total 48
drwxrwxr-x  7 ryan ryan 4096 May  4 20:55 .
drwxr-xr-x 11 ryan ryan 4096 May  4 20:56 ..
drwxrwxr-x  2 ryan ryan 4096 May  4 20:55 css
drwxrwxr-x  2 ryan ryan 4096 May  4 20:55 _includes
drwxrwxr-x  2 ryan ryan 4096 May  4 20:55 _layouts
drwxrwxr-x  2 ryan ryan 4096 May  4 20:55 _posts
drwxrwxr-x  2 ryan ryan 4096 May  4 20:55 _sass
-rw-r--r--  1 ryan ryan  470 May  4 20:55 about.md
-rw-r--r--  1 ryan ryan  557 May  4 20:55 _config.yml
-rw-r--r--  1 ryan ryan 1291 May  4 20:55 feed.xml
-rw-r--r--  1 ryan ryan   18 May  4 20:55 .gitignore
-rw-r--r--  1 ryan ryan  506 May  4 20:55 index.html
{% endhighlight %}

List the contents of all subdirectories. Use the `-d` flag to just list directories.
{% highlight bash %}
± ls -l */
node_modules/:
total 40
drwxrwxr-x 5 ryan ryan 4096 May  3 16:24 hexo
drwxrwxr-x 4 ryan ryan 4096 May  3 16:29 hexo-deployer-git
drwxrwxr-x 4 ryan ryan 4096 May  3 16:23 hexo-generator-archive
drwxrwxr-x 4 ryan ryan 4096 May  3 16:23 hexo-generator-category
drwxrwxr-x 4 ryan ryan 4096 May  3 16:23 hexo-generator-index
drwxrwxr-x 4 ryan ryan 4096 May  3 16:23 hexo-generator-tag
drwxrwxr-x 3 ryan ryan 4096 May  3 16:23 hexo-renderer-ejs
drwxrwxr-x 4 ryan ryan 4096 May  3 16:23 hexo-renderer-marked
drwxrwxr-x 4 ryan ryan 4096 May  3 16:23 hexo-renderer-stylus
drwxrwxr-x 4 ryan ryan 4096 May  3 16:23 hexo-server

public/:
total 28
drwxrwxr-x 3 ryan ryan 4096 May  3 16:27 2015
drwxrwxr-x 3 ryan ryan 4096 May  3 16:27 archives
drwxrwxr-x 4 ryan ryan 4096 May  3 16:27 css
drwxrwxr-x 3 ryan ryan 4096 May  3 16:27 fancybox
drwxrwxr-x 2 ryan ryan 4096 May  3 16:27 js
-rw-rw-r-- 1 ryan ryan 7549 May  3 16:27 index.html

scaffolds/:
total 12
-rw-rw-r-- 1 ryan ryan 29 May  3 16:23 draft.md
-rw-rw-r-- 1 ryan ryan 40 May  3 16:23 page.md
-rw-rw-r-- 1 ryan ryan 46 May  3 16:23 post.md

source/:
total 4
drwxrwxr-x 2 ryan ryan 4096 May  3 16:24 _posts

themes/:
total 4
drwxrwxr-x 5 ryan ryan 4096 May  3 16:23 landscape

ryan@neptune|~/testing-site on gh-pages!
± ls -d */
node_modules/  public/  scaffolds/  source/  themes/
{% endhighlight %}

List files sorted by the time they were last modified. You can use the `-r` flag to show last modified in reverse order (most recently modified files last).

{% highlight bash %}
± ls -lt
total 32
drwxrwxr-x  7 ryan ryan 4096 May  3 16:34 public
drwxrwxr-x 13 ryan ryan 4096 May  3 16:33 node_modules
drwxrwxr-x  3 ryan ryan 4096 May  3 16:32 source
drwxrwxr-x  3 ryan ryan 4096 May  3 16:32 themes
drwxrwxr-x  2 ryan ryan 4096 May  3 16:32 scaffolds
-rw-rw-r--  1 ryan ryan  174 May  3 16:36 db.json
-rw-rw-r--  1 ryan ryan 1600 May  3 16:36 _config.yml
-rw-rw-r--  1 ryan ryan  481 May  3 16:33 package.json

± ls -ltr
total 32
drwxrwxr-x  2 ryan ryan 4096 May  3 16:32 scaffolds
drwxrwxr-x  3 ryan ryan 4096 May  3 16:32 themes
drwxrwxr-x  3 ryan ryan 4096 May  3 16:32 source
drwxrwxr-x 13 ryan ryan 4096 May  3 16:33 node_modules
drwxrwxr-x  7 ryan ryan 4096 May  3 16:34 public
-rw-rw-r--  1 ryan ryan  481 May  3 16:33 package.json
-rw-rw-r--  1 ryan ryan 1600 May  3 16:36 _config.yml
-rw-rw-r--  1 ryan ryan  174 May  3 16:36 db.json
{% endhighlight %}

### cd
With no arguments, cd will change to your home directory, which is /root for the superuser and typically /home/username for a regular user.


### mkdir
Let's take a quick look at the mkdir command, which can be used to create new directories. The following example creates three new directories, foo, bar, and thing, all under my home directory:
{% highlight bash %}
[<*>]  ~  mkdir {foo,bar,thing}
[<*>]  ~  ls
bar  foo  thing

[<*>]  ~  mkdir foo1 bar1 thing1
[<*>]  ~  ls
bar  bar1  foo  foo1  thing  thing1
{% endhighlight %}

By default, the mkdir command doesn't create parent directories for you, so if you wanted to create the directory structure `$HOME/my/bin/folder` you need to add the `-p` flag

{% highlight bash %}
[<*>]  ~  mkdir my/bin/folder
mkdir: cannot create directory 'my/bin/folder' : No such file or directory

[<*>]  ~  mkdir -p my/bin/folder
[<*>]  ~  tree -d -L 3
.
├── bar
├── bar1
├── foo
├── foo1
├── my
│   └── bin
│       └── folder
{% endhighlight %}

### echo
The echo command takes its arguments and prints them to standard output. By default this is the terminal itself but can also redirect in to files and other commands

{% highlight bash %}
[<*>]  ~  echo "I am a thing"
I am a thing

[<*>]  ~  echo "So am I" > testfile
[<*>]  ~  cat testfile
So am I

[<*>]  ~  echo `whoami`
ryan

[<*>]  ~  echo $(ping -c3 -q google.com)
PING google.com (74.125.227.164) 56(84) bytes of data. --- google.com ping statistics --- 3 packets transmitted, 3 received, 0% packet loss, time 2002ms rtt min/avg/max/mdev = 5.332/5.374/5.454/0.101 ms
{% endhighlight %}

### cat

{% highlight bash %}

{% endhighlight %}


### mv

{% highlight bash %}

{% endhighlight %}



### cp

{% highlight bash %}

{% endhighlight %}


### rm

{% highlight bash %}

{% endhighlight %}



### man

{% highlight bash %}

{% endhighlight %}


### less

{% highlight bash %}

{% endhighlight %}



### head


### tail


### grep


### which


### history


### chown


### chmod


### find


### locate


### clear
