---
layout: post
title:  "Introduction to *Nix: Part 1"
date:   2015-05-01 09:49:55
categories: jekyll update
---

This tutorial (Introduction to *Nix: Part 1) is for those who are new to Linux, or those who want to review their understanding of fundamental Linux concepts and commands. For beginners, a lot of this material will be new. For more experienced users you can use this as a refresher or jump ahead to [Part 2]() or [Part 3]() **Or you will be able to once I get around to writing those sections**

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

The way the move command actually works is that it copies the original file to the new path or name and then deletes the original. You can see this more readily if you use the `-v` command

{% highlight bash %}
[<*>]  ~ ls -i about.md
4461464 about.md
[<*>]  ~ mv about.md aboutyou.md
[<*>]  ~ ls -i aboutyou.md
4461464 aboutyou.md

[<*>]  ~ ls -i aboutyou.md
4461464 aboutyou.md

[<*>]  ~ mv aboutyou.md /bacula
[<*>]  ~ ls -i /bacula/aboutyou.md
12 /bacula/aboutyou.md
{% endhighlight %}

We can also move multiple files to a new location. The last path in the command is where the files will be moved to:

{% highlight bash %}
[<*>]  ~  ls -l thing
total 0
[<*>]  ~  mv -v foo bar css foo1 thing
‘foo’ -> ‘thing/foo’
‘bar’ -> ‘thing/bar’
‘css’ -> ‘thing/css’
‘foo1’ -> ‘thing/foo1’

[<*>]  ~  ls thing
bar  css  foo  foo1
{% endhighlight %}

### cp
To copy a file, you need to pass source and destination to the copy command. If you want to copy a file from one folder to another with the same name you only need to specify the new location and not the exact filename
{% highlight bash %}
[<*>]  ~  cp -v apthist thing1
‘apthist’ -> ‘thing1/apthist’

[<*>]  ~  cp -v thing1/apthist ~/thing/foo/new_apthist
‘thing1/apthist’ -> ‘/home/ryan/thing/foo/new_apthist’
{% endhighlight %}

If you want to copy a directory and all of its contents from source to destination you would use the `-r` flag:

{% highlight bash %}
[<*>]  ~  ls ~/things
_includes  _layouts  _posts  _sass  _config.yml  feed.xml  index.html
[<*>]  ~  mkdir ~/backup
[<*>]  ~  ls -l ~/backup
total 0
[<*>]  ~  cp -r ~/things/* backup/
[<*>]  ~  ls  ~/backup
_includes  _layouts  _posts  _sass  _config.yml  feed.xml  index.html
{% endhighlight %}

**Caveat** When you execute the cp command on a file that is a symlink the file is copied but the link is not preserved. To preserve the link you need to use the `-d` flag:

{% highlight bash %}
[<*>] $  ls -l
total 0
lrwxrwxrwx 1 ryan ryan 23 May  5 08:55 start.sh -> /home/ryan/bin/start.sh
[<*>] $  cp start.sh ~/
[<*>] $  ls -l ~/start.sh
-rw-rw-r-- 1 ryan ryan 0 May  5 08:55 /home/ryan/start.sh

[<*>] $  cp -d start.sh ~/things
[<*>] $  ls -l ~/things/start.sh
lrwxrwxrwx 1 ryan ryan 23 May  5 08:58 /home/ryan/things/start.sh -> /home/ryan/bin/start.sh
{% endhighlight %}
[More cp examples](http://www.thegeekstuff.com/2013/03/cp-command-examples/)

### rm
The rm command is used to remove objects from the filesystem. You can use the `-f` option to not be prompted about the removal. If you are still getting used to the rm command, it can be useful to add the following line to your ~/.bashrc file: alias rm='rm -i'

{% highlight bash %}
[<*>]  ~  rm -i testfile
rm: remove regular file ‘testfile’? y

[<*>]  ~  rm -f tokenfile
[<*>]  ~

[<*>]  ~  touch readme.txt
[<*>]  ~  for i in {1..10}; do touch file$i; done
[<*>]  ~  ls -l
total 0
-rw-rw-r-- 1 ryan ryan 0 May  5 09:53 file1
-rw-rw-r-- 1 ryan ryan 0 May  5 09:53 file10
-rw-rw-r-- 1 ryan ryan 0 May  5 09:53 file2
-rw-rw-r-- 1 ryan ryan 0 May  5 09:53 file3
-rw-rw-r-- 1 ryan ryan 0 May  5 09:53 file4
-rw-rw-r-- 1 ryan ryan 0 May  5 09:53 file5
-rw-rw-r-- 1 ryan ryan 0 May  5 09:53 file6
-rw-rw-r-- 1 ryan ryan 0 May  5 09:53 file7
-rw-rw-r-- 1 ryan ryan 0 May  5 09:53 file8
-rw-rw-r-- 1 ryan ryan 0 May  5 09:53 file9
-rw-rw-r-- 1 ryan ryan 0 May  5 09:53 readme.txt

[<*>]  ~  rm -f file[1-10]
[<*>]  ~  ls -l
-rw-rw-r-- 1 ryan ryan 0 May  5 09:53 readme.txt
{% endhighlight %}

{% highlight bash %}
[<*>]  ~  rm -f things
rm: cannot remove ‘things’: Is a directory

[<*>]  ~  rmdir things
rmdir: failed to remove ‘things’: Directory not empty

[<*>]  ~  rm -i -rf things
[<*>]  ~  ls things
ls: cannot access things: No such file or directory
{% endhighlight %}
Be very careful when using rm -rf, since its power can be used for both good and evil :)

### man
The man pages are a user manual that is by default built into most Linux distributions (i.e., versions) and most other Unix-like operating systems during installation. They provide extensive documentation about commands and other aspects of the system, including configuration files, system calls, library routines and the kernel (i.e., the core of the operating system).

### less
Less is similar to more command, but less allows both forward and backward movements. Moreover, less don’t require to load the whole file before viewing. Common commands:

    / = search for a pattern which will take you to the next occurrence.  
    n = for next match in forward
    ? = search for a pattern which will take you to the previous occurrence.  
    g = go to the start of file  
    Shift + g = go to the end of file
    Shift + f = same as tail -f (benefit is that you can still use / to search)  
    10j = 10 lines forward  
    10k = 10 lines backward
    v = using the configured editor edit the current file  

### head
Head prints the first N number of data of the given input. By default, it prints first 10 lines of each given file.

{% highlight bash %}
[<*>]  ~  head -n5 testfile.txt
Lorem
ipsum
dolor
sit
amet
[<*>]  ~  head testfile.txt
Lorem
ipsum
dolor
sit
amet
consectetur
adipiscing
elit
Proin
blandit
{% endhighlight %}

You can also pass the output of another command to head:

{% highlight bash %}
[<*>]  ~  ls | head
backup
bar1
bin
my
project
test2
thing
thing1
apthist
apthist_mark1
{% endhighlight %}

### tail
Tail prints the last N number of data of the given input. By default, it prints last 10 lines of each given file. You can also use the `-f` flag to have the output update in real time. As with the head command you can also pass the output of other commands to tail
{% highlight bash %}
[<*>]  ~  tail -n 2 testfile.txt
lacus
luctus

[<*>]  ~  ls | tail -5
readme.txt
readme.txt~
record_552867.mp4
start.sh
testfile.txt
{% endhighlight %}

### grep

{% highlight bash %}

{% endhighlight %}

### which

{% highlight bash %}

{% endhighlight %}

### history

{% highlight bash %}

{% endhighlight %}

### chown
chown changes the user and/or group ownership of each given file.  If only an owner (a user name or numeric user ID) is given, that user is made the owner of each given file, and the files' group is not changed. You can use the `-R` command to make this recursive in to subdirectories
{% highlight bash %}
[<*>]  ~  ls -lart tmpfile
-rw-rw-r-- 1 ryan ryan 0 May  5 12:54 tmpfile

[<*>]  ~  chown root tmpfile
[<*>]  ~  ls -lart tmpfile
-rw-rw-r-- 1 root ryan 0 May  5 12:54 tmpfile

[<*>]  ~  ls -l backup/*
-rw-rw-r-- 1 ryan ryan    0 May  5 09:53 backup/file2
-rw-rw-r-- 1 ryan ryan    0 May  5 09:53 backup/file3
-rw-rw-r-- 1 ryan ryan    0 May  5 09:53 backup/file4
-rw-rw-r-- 1 ryan ryan    0 May  5 09:53 backup/file5
-rw-rw-r-- 1 ryan ryan  995 May  5 09:57 backup/readme.txt

backup/subfolder:
total 0
-rw-rw-r-- 1 ryan ryan 0 May  5 12:58 text_1
-rw-rw-r-- 1 ryan ryan 0 May  5 12:58 text_2
-rw-rw-r-- 1 ryan ryan 0 May  5 12:58 text_3

[<*>]  ~  _ chown -R root backup/
[<*>]  ~  ls -l backup/*
-rw-rw-r-- 1 root ryan    0 May  5 09:53 backup/file2
-rw-rw-r-- 1 root ryan    0 May  5 09:53 backup/file3
-rw-rw-r-- 1 root ryan    0 May  5 09:53 backup/file4
-rw-rw-r-- 1 root ryan    0 May  5 09:53 backup/file5
-rw-rw-r-- 1 root ryan  995 May  5 09:57 backup/readme.txt

backup/subfolder:
total 0
-rw-rw-r-- 1 root ryan 0 May  5 12:58 text_1
-rw-rw-r-- 1 root ryan 0 May  5 12:58 text_2
-rw-rw-r-- 1 root ryan 0 May  5 12:58 text_3
{% endhighlight %}

To just change the group of a file or directory use the `:group` syntax. Again you can use `-R` to make the changes recursive.
{% highlight bash %}
[<*>]  ~  _ chown :users backup/file2
[<*>]  ~  ls -l backup/
total 8
drwxrwxr-x 2 root ryan  4096 May  5 12:58 subfolder
-rw-rw-r-- 1 root users    0 May  5 09:53 file2
-rw-rw-r-- 1 root ryan     0 May  5 09:53 file3
-rw-rw-r-- 1 root ryan     0 May  5 09:53 file4
-rw-rw-r-- 1 root ryan     0 May  5 09:53 file5
-rw-rw-r-- 1 root ryan   995 May  5 09:57 readme.txt

[<*>]  ~  _ chown -R :users backup/
[<*>]  ~  ls -l backup/*
-rw-rw-r-- 1 root users    0 May  5 09:53 backup/file2
-rw-rw-r-- 1 root users    0 May  5 09:53 backup/file3
-rw-rw-r-- 1 root users    0 May  5 09:53 backup/file4
-rw-rw-r-- 1 root users    0 May  5 09:53 backup/file5
-rw-rw-r-- 1 root users  995 May  5 09:57 backup/readme.txt

backup/subfolder:
total 0
-rw-rw-r-- 1 root users 0 May  5 12:58 text_1
-rw-rw-r-- 1 root users 0 May  5 12:58 text_2
-rw-rw-r-- 1 root users 0 May  5 12:58 text_3
{% endhighlight %}

You can change the user and group at the same time using the syntax `user:group`. If the user and the group name are the same you can use `user:`. Again `-R` makes the change recursive.
{% highlight bash %}
[<*>]  ~  _ chown -R ryan: backup/
[<*>]  ~  ls -l backup/*
-rw-rw-r-- 1 ryan ryan    0 May  5 09:53 backup/file2
-rw-rw-r-- 1 ryan ryan    0 May  5 09:53 backup/file3
-rw-rw-r-- 1 ryan ryan    0 May  5 09:53 backup/file4
-rw-rw-r-- 1 ryan ryan    0 May  5 09:53 backup/file5
-rw-rw-r-- 1 ryan ryan  995 May  5 09:57 backup/readme.txt

backup/subfolder:
total 0
-rw-rw-r-- 1 ryan ryan 0 May  5 12:58 text_1
-rw-rw-r-- 1 ryan ryan 0 May  5 12:58 text_2
-rw-rw-r-- 1 ryan ryan 0 May  5 12:58 text_3

[<*>]  ~  _ chown -R ryan:users backup/
[<*>]  ~  ls -l backup/*
-rw-rw-r-- 1 ryan users    0 May  5 09:53 backup/file2
-rw-rw-r-- 1 ryan users    0 May  5 09:53 backup/file3
-rw-rw-r-- 1 ryan users    0 May  5 09:53 backup/file4
-rw-rw-r-- 1 ryan users    0 May  5 09:53 backup/file5
-rw-rw-r-- 1 ryan users  995 May  5 09:57 backup/readme.txt

backup/subfolder:
total 0
-rw-rw-r-- 1 ryan users 0 May  5 12:58 text_1
-rw-rw-r-- 1 ryan users 0 May  5 12:58 text_2
-rw-rw-r-- 1 ryan users 0 May  5 12:58 text_3
{% endhighlight %}

### chmod
chmod changes the file mode bits of each given file according to mode, which can be either a symbolic representation of changes to make, or an octal number representing the bit pattern for the new mode bits.
{% highlight bash %}
[<*>]  ~  ls -l tmpfile
-rw-rw-r-- 1 root ryan 0 May  5 12:54 tmpfile
{% endhighlight %}
This breaks down as `(user)(group)(world)` or `(rw-)(rw-)(r--)`

> The first character (in this case an underscore) is the special permission flag that can vary.  
> The first set of three characters (rw-) is for the owner permissions.  
> The second set of three characters (rw-) is for the Group permissions.  
> The third set of three characters (r--) is for the All Users permissions.  
> r = read = 4  
> w = write = 2  
> x = execute = 1

You can set the executable bit by using the `-x` option or setting the perms to `775`
{% highlight bash %}
[<*>]  ~  chmod +x tmpfile
[<*>]  ~  ls -l tmpfile
-rwxrwxr-x 1 ryan ryan 0 May  5 12:54 tmpfile

[<*>]  ~  stat -c '%n %a' tmpfile
tmpfile 775

[<*>]  ~  chmod 444 tmpfile
[<*>]  ~  stat -c '%n %a' tmpfile
tmpfile 444

[<*>]  ~  chmod u+rw,g+rwx tmpfile
[<*>]  ~  stat -c '%n %a' tmpfile
tmpfile 674
{% endhighlight %}
[7 Chmod Command Examples for Beginners](http://www.thegeekstuff.com/2010/06/chmod-command-examples/)


### find
The Find command used to search and locate list of files and directories based on conditions you specify for files that match the arguments. The syntax is `find location comparison-criteria search-term`
{% highlight bash %}
[<*>]  ~  find ./backup -iname "file*"
./backup/file5
./backup/file2
./backup/file3
./backup/file4

[<*>]  ~  find ./backup -type d
./backup
./backup/subfolder

[<*>]  ~  find ./backup  -maxdepth 1 -iname "file*"
./backup/file5
./backup/file2
./backup/file3
./backup/file4
[<*>]  ~  find ./backup  -maxdepth 2 -iname "file*"
./backup/file5
./backup/file2
./backup/file3
./backup/subfolder/file_2
./backup/subfolder/file_3
./backup/subfolder/file_1
./backup/file4
{% endhighlight %}

You can also use the `-not` option to search for files that do no match a given name or pattern.

{% highlight bash %}
[<*>]  ~  find ./backup -not -iname "file*"
./backup
./backup/readme.txt
./backup/subfolder
./backup/subfolder/text_1
./backup/subfolder/text_3
./backup/subfolder/text_2
{% endhighlight %}  

You can use the `-o` option to search for multiple files. You can also use the `!` to prune itmes from the search. In the second example we are looking for files that begin with `file` but do not end in `3`

{% highlight bash %}
[<*>]  ~  find ./backup -type f -iname 'file*' -o -iname 'text*'
./backup/file5
./backup/file2
./backup/file3
./backup/subfolder/text_1
./backup/subfolder/text_3
./backup/subfolder/file_2
./backup/subfolder/file_3
./backup/subfolder/file_1
./backup/subfolder/text_2
./backup/file4

[<*>]  ~  find ./backup  -iname 'file*' ! -iname '*3'
./backup/file5
./backup/file2
./backup/subfolder/file_2
./backup/subfolder/file_1
./backup/file4
{% endhighlight %}

You can also search by size and time of file. To find by size you can use the `-size` option with the following parameters:  

    c = bytes
    k = Kilobytes
    M = Megabytes
    G = Gigabytes

By time we can use the following options:

    "-atime" = Access Time: Last time a file was read or written to.
    "-mtime" = Modification Time: Last time the contents of the file were modified.
    "-ctime" = Change Time: Last time the file's inode meta-data was changed.

[35 find examples](http://www.binarytides.com/linux-find-command-examples/)

### locate

{% highlight bash %}

{% endhighlight %}
