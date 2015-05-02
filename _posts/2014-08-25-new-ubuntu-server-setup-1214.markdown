---
layout: post
title: "New Ubuntu Server Setup 12/14"
date: "2014-08-25 08:29"
---

This is a quick bash script that I use on new deployments of Ubuntu server 12 or 14. The script does the following:

> Changes the default SSH port to a random 4 digit number Create a new user named user if they do not already exist on the system  
> Assigns a 12 character, randomly generated password for the new user  
> Changes root login to only be allowed if using SSH keys  
> Installs the base packages needed for a jekyll server  
> Emails all the new server information to your email and then restarts the server

    #!/usr/bin/env bash

    dt=$(date +"%b-%d-%y")
    pass=$(< /dev/urandom tr -dc _A-Z-a-z-0-9 | head -c12; echo)
    ips=$(ifconfig|grep inet\ addr|awk '{print $2}'|cut -d ':' -f2|grep -v 127.0.0.1)
    hst=$(hostname -f)
    ssh_port=$(< /dev/urandom tr -dc 1-9 | head -c4)
    provision_log="/root/provision.${dt}.log"
    os=$(lsb_release -i -r)

    base_install () {
    {
       echo -e "Post install details for server $hst\n";
       echo -e "Provision date: $dt\n";
       echo -e "OS: $os\n";
       echo -e "Server IPs:\n$ips\n";
       echo -e "Password for user user: $pass\n";
       echo -e "New ssh port: $ssh_port\n";

    } >> "$provision_log" 2>&1

    if ! grep -q ryan /etc/passwd;
      then useradd -m -g users -G wheel -s /bin/bash user >> "$provision_log" 2>&1 ; echo user:"$pass" | chpasswd >> "$provision_log" 2>&1;
    fi  

    apt-get update
    apt-get install sudo wget jwhois lsof nano curl sendmail mailutils git python-software-properties -y
    apt-add-repository ppa:chris-lea/node.js -y
    apt-get update
    apt-get install nodejs -y

    echo "nodejs version $(node -v)" >> "$provision_log" 2>&1

    ssh-keygen -t rsa -P '' -f "$HOME"/.ssh/id_rsa
    sudo su - ryan -c "ssh-keygen -t rsa -P '' -f /home/user/.ssh/id_rsa"
    }

    ### Grab Base Configuration Files
    configure_system() {
    git clone https://github.com/greyhound-forty/Configuration-Files.git
    mv {,original}.bashrc
    mv /home/ryan/{,original}.bashrc
    cp "$HOME"/Configuration-Files/Bash/.bashrc "$HOME"
    cp "$HOME"/Configuration-Files/Bash/.bashrc /home/user
    cp "$HOME"/Configuration-Files/Bash/.bash-config "$HOME"
    cp "$HOME"/Configuration-Files/Bash/.bash-config /home/user

    mv /etc/sudoers{,.bak}

    cp "$HOME"/Configuration-Files/Server/sudoers /etc/sudoers
    chmod 0440 /etc/sudoers

    cp "$HOME"/Configuration-Files/Server/authorized_keys "$HOME"/.ssh/authorized_keys
    cp "$HOME"/Configuration-Files/Server/authorized_keys /home/user/.ssh/authorized_keys

    mv /etc/ssh/sshd_config{,.bak}
    cp "$HOME"/Configuration-Files/Server/sshd_config /etc/ssh/sshd_config

    /bin/sed -ie "s|#Port 22|Port ${ssh_port}|" /etc/ssh/sshd_config
    /bin/sed -ie "s|#PermitRootLogin no|PermitRootLogin without-password|g" /etc/ssh/sshd_config
    }


    install_jekyll() {
            curl -L https://get.rvm.io | bash -s stable
            source /etc/profile.d/rvm.sh
            rvm requirements
            rvm install ruby
            rvm use ruby --default
            rvm rubygems current
            gem install jekyll sinatra thin json
    }

    base_install
    configure_system
    install_jekyll


    mail -s "New Server Provision" user@email.com < "$provision_log" && shutdown -r now
