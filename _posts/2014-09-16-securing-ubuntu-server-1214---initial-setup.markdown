---
layout: post
title: "Securing Ubuntu Server 12/14 - Initial Setup"
date: 2014-08-18
---

Most VPS provider provide root login via SSH when the instance is initially provisioned. For obvious reasons we want to turn that off. I recommend having 2 terminal sessions open while doing your initial security update, in case you end up locking yourself out.

For those of you whose instances are provisioned with a user other than root, you have most likely been granted sudo access. If that is the case you can skip this bit:

    :~# useradd -m -g users -G sudo -s /bin/bash newuser  
    :~# passwd newuser  
    :~# EDITOR=nano visudo // yes I prefer nano


With our new sudo user created we can start updating our SSH configuration. As someone that works in tech support I am paranoid about obliterating configuration files. So for my sanity I will create a backup

    :~# cp -v /etc/ssh/sshd_config{,.bak}


Now let's check the PermitRootLogin and Port entry in our config file:

    :~# egrep 'PermitRootLogin|Port' /etc/ssh/sshd_config
    Port 22  
    PermitRootLogin yes  


Now you can use your favorite editor to update the file but why not make life easier on yourself with sed:

    :~# sed -i 's|Port 22|Port 3322|g' /etc/ssh/sshd_config  
    :~# sed -i 's|PermitRootLogin yes|PermitRootLogin no|g' /etc/ssh/sshd_config


Time to set up our initial firewall rules

    >> Run as root or use sudo  
    :~# sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT  
    :~# sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT  
    :~# sudo iptables -A INPUT -p tcp --dport 3322 -j ACCEPT  
    :~# sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT  
    :~# sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT  
    :~# sudo iptables -A INPUT -j DROP
    :~# sudo iptables -I INPUT 1 -i lo -j ACCEPT  


Let's check on our rules before we save them just to make sure we'll be able to get back in

    :~# root@ubuntu:~# iptables -nL
    Chain INPUT (policy ACCEPT)
    target     prot opt source               destination
    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0
    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:3322
    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:80
    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:443
    DROP       all  --  0.0.0.0/0            0.0.0.0/0

    Chain FORWARD (policy ACCEPT)
    target     prot opt source               destination

    Chain OUTPUT (policy ACCEPT)
    target     prot opt source               destination


To ensure our firewall rules are saved through restarts we'll install and use iptables-persistent

    :~# sudo apt-get install -y iptables-persistent  
    :~# sudo service iptables-persistent start


Now we can take our login a step further by setting up SSH keys. On our local machine we can generate our SSH keys for the newuser, since this is how we will be accessing the server after reboot. If your local system is linux you can use ssh-keygen to generate the key pair. If your local machine is Windows see this [guide][1] for generating and using SSH keys:

    root@vbox:~# ssh-keygen -t rsa -C "vboxhost"
    Generating public/private rsa key pair.
    Enter file in which to save the key (/root/.ssh/id_rsa):
    /root/.ssh/id_rsa already exists.
    Overwrite (y/n)? y
    Enter passphrase (empty for no passphrase):    // # I'd highly recommend passphrases
    Enter same passphrase again:
    Your identification has been saved in /root/.ssh/id_rsa.
    Your public key has been saved in /root/.ssh/id_rsa.pub.
    The key fingerprint is:
    ac:db:b4:ce:ee:dc:e5:99:64:1d:a7:69:11:7a:ad:3b vboxhost
    The key's randomart image is:
    +--[ RSA 2048]----+
    |                 |
    |                 |
    |              .  |
    |       .     . o |
    |        S   . + o|
    |       .     o B |
    |      . .   + *  |
    |       * o = +E. |
    |      .+O . + .. |
    +-----------------+


As this is is our first key we'll use the defaults. As noted above I would highly recommend providing a pass-phrase. If someone manages to get your key then the pass-phrase provides an extra barrier to entry. By default your keys are stored in $HOME/.ssh. To get the new key to the remote server we'll be using the ssh-copy-id utility. This tool lets you copy a public key to a server when you are still using password based login. This is why we did not start the ssh service after our changes earlier.

    :~# root@vbox:~# ssh-copy-id newuser@192.168.1.68
    The authenticity of host '192.168.1.68 (192.168.1.68)' can't be established.
    ECDSA key fingerprint is c1:09:ed:e8:20:66:fa:c7:5c:fc:2b:21:ec:5d:5f:eb.
    Are you sure you want to continue connecting (yes/no)? yes
    /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
    /usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
    newuser@192.168.1.68's password:

    Number of key(s) added: 1

    Now try logging into the machine, with:   "ssh 'newuser@192.168.1.68'"
    and check to make sure that only the key(s) you wanted were added.


Now let's test that our keyworks before we reboot the instance. You will be prompted for your key passphrase unless you use a utility like ssh-agent which I'll discuss in part 2 of this guide:

    :~# root@vbox:~# ssh 'newuser@192.168.1.68'
    Enter passphrase for key '/root/.ssh/id_rsa':
    Welcome to Ubuntu 12.04.4 LTS (GNU/Linux 3.2.0-61-virtual x86_64)

     * Documentation:  https://help.ubuntu.com/
    newuser@ubuntu:~$


Now it is time to cross your fingers and reboot. Once the system is back online we'll add a few more things to help secure our server:

## SSH-Agent

Before we go further however I would like to mention SSH-Agent. This little utility does you a favor by managing your SSH keys for you. This is helpful when you have assigned a passphrase to your SSH key. With this tool you enter the passphrase once, and after that, ssh-agent keeps your key in its memory and pulls it up whenever it is needed.

    :~# ssh-agent
    SSH_AUTH_SOCK=/tmp/ssh-tBYKKVasWnft/agent.14140; export SSH_AUTH_SOCK;
    SSH_AGENT_PID=14141; export SSH_AGENT_PID;
    echo Agent pid 14141;

    :~# eval `ssh-agent -s`
    Agent pid 14144

    :~# ssh-add ~/.ssh/id_rsa
    Enter passphrase for /root/.ssh/id_rsa:
    Identity added: /root/.ssh/id_rsa (/root/.ssh/id_rsa)

    :~# ssh newuser@192.168.1.68
    Welcome to Ubuntu 12.04.4 LTS (GNU/Linux 3.2.0-61-virtual x86_64)

     * Documentation:  https://help.ubuntu.com/
    Last login: Thu May 22 18:55:34 2014 from 192.168.1.68.static.local
    newuser@ubuntu:~$


As you can see we no longer need to provide our passphrase when ssh-ing in to our new server. You can also use the command `ssh-add -l` to look at your currently loaded keys.

Now that we can easily get back in to our box let's continue plugging some holes.

## /dev/shm

The `/dev/shm` mount can be used in an attack against a running service, such as httpd or nginx. To get around this we will modify /etc/fstab to make it more secure. Add the following to the bottom of the fstab file:

    tmpfs     /dev/shm     tmpfs     defaults,noexec,nosuid     0     0


## Fail2ban

The Fail2ban program scans your servers log files and bans IPs that look malicious. Examples of malicious activity would include things like; too many password failures, probing for exploits, etc.

    :~# sudo apt-get install fail2ban


After the install open up the /etc/fail2ban/jail.conf file for editing - Find this line - how to tell which line

    :~# egrep -n -i '# Jails' /etc/fail2ban/jail.conf  
    78:# JAILS


Delete everything beneath that line and replace it with the content below and save. If you're using nano/vi you can easily get to that spot with +N where N is the line number. In this case it would be `nano +78 /etc/fail2ban/jail.conf` or `vi +78 /etc/fail2ban/jail.conf`

    [ssh]

    enabled  = true
    port     = 3322
    filter   = sshd
    logpath  = /var/log/auth.log
    maxretry = 6

    [ssh-ddos]

    enabled  = true
    port     = 3322
    filter   = sshd-ddos
    logpath  = /var/log/auth.log
    maxretry = 10

    [recidive]

    enabled  = true
    filter   = recidive
    logpath  = /var/log/fail2ban.log
    action   = iptables-allports[name=recidive]
           sendmail-whois-lines[name=recidive, logpath=/var/log/fail2ban.log]
    bantime  = 604800  ; 1 week
    findtime = 86400   ; 1 day
    maxretry = 5


You can activate any services you would like fail2ban to monitor by changing enabled = false to enabled = true in the configuration file. You can also create rule filters for various services that are not monitored by default with fail2ban. This is done with the help of the `/etc/fail2ban/jail.local` file. Check out [this Digital Ocean guide][2] for more information on using the jail.local file.

After we're done configuring fail2ban we need to restart the service and check the status:

    :~# sudo service fail2ban restart  

    :~# sudo fail2ban-client status
    Status
    |- Number of jail:      1
    `- Jail list:           ssh


This completes our initial setup of our Ubuntu server. If you'd like to dive deeper in to securing your server check out the following guides:

[Hardening Ubuntu with Bastille][3]  
[Securing Ubuntu with Tiger, PSAD, and more][4]  
[Ubuntu Secure GUI installer script][5]

 [1]: http://sshcontrol.com/help/puttygen_keys
 [2]: https://www.digitalocean.com/community/articles/how-to-protect-ssh-with-fail2ban-on-ubuntu-12-04
 [3]: https://www.digitalocean.com/community/articles/how-to-install-and-use-bastille-to-harden-an-ubuntu-12-04
 [4]: http://www.andrewault.net/2010/05/17/securing-an-ubuntu-server/
 [5]: http://www.thefanclub.co.za/how-to/how-secure-ubuntu-1204-lts-server-part-2-gui-installer-script
