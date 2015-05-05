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

{% highlight shell %}
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
+ [more](#more)
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



#### Relative Path


### ls


### cd


### mkdir


### echo


### cat


### mv


### cp


### rm



### man


### less


### more


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
