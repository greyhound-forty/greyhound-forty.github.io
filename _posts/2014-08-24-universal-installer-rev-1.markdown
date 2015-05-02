---
layout: post
title: "Universal Installer Rev. 1"
date: "2014-08-24 08:31"
---

This is an attempt at a universal installer script in bash. The goal of the script is to determine the underlying OS and then run a set of functions based on the specific operating system. This is very much a version 1 type of script that I will be tweaking in later posts:

    #!/usr/bin/env bash
    #set -o nounset
    #set -o errexit

    ############ [ VARIABLES ] ############
    dt=$(date +"%b-%d-%y")
    d_dir="$HOME/Deployment"
    hst=$(hostname)
    #######################################

    if [ ! -d "$d_dir" ]
    then
    mkdir -p "$d_dir"
    fi

    # Start logging
    echo "Server Post Provisioning Results:"$'\n' > "$dt.$hst.log"

    os() {
    if [ -e /etc/redhat-release ]
      then
        os="centos"
    elif [ -e /etc/lsb-release ]
      then
       os="ubuntu"
    elif [ -e /etc/arch-release ]
      then
        os="arch"
    elif [ -e /var/log/installer/lsb-release ]
      then
        os="debian"
    fi
    }

    base_setup() {
      case "$os" in
        "Centos" | "centos" )
            os_ver=$(cat /etc/redhat-release)
            echo "$os_ver" >> "$dt.$hst.log"
            yum update -y && yum upgrade -y
            yum install screen curl wget jwhois bind-utils samba samba-client make gcc git -y
      ;;
        "Ubuntu" | "ubuntu" )
            os_ver=$(lsb_release -r|grep Release|awk '{print $2}'|cut -d '.' -f1)
            echo "System is running Ubuntu '$os_ver'" >> "$dt.$hst.log"
            apt-get update && apt-get upgrade -y && apt-get install curl git -y
      ;;
        "Arch" | "arch" )
            os_ver=$(cat < /etc/issue|awk '{print $1,$2 }')
            echo "$os_ver" >> "$dt.$hst.log"
            pacman -Syu --noconfirm
            pacman -S screen curl git wget fakeroot make gcc python dnsutils --noconfirm
      ;;
        "Debian" | "debian" )
            os_ver=$(cat /etc/debian_version)
            echo "$os_ver" >> "$dt.$hst.log"
            apt-get update && apt-get upgrade -y
            apt-get install screen sudo wget jwhois git lsof nano curl sendmail mailutils -y
      ;;
      esac
    }

    ## Grab all needed files from github
    grab_files() {
        cd "$d_dir"
        git clone https://github.com/greyhound-forty/Post_Install.git
        git clone https://github.com/greyhound-forty/Configuration-Files.git
    }

    update_system() {
      case "$os_ver" in
        "Centos" | "centos" )
            chmod +x "$d_dir/Post_Install/centos-6-post-install-script.sh"
            screen -S Centos -d -m /bin/bash "$d_dir/Post_Install/centos-6-post-install-script.sh"
      ;;
        "12" )
            chmod +x "$d_dir/Post_Install/ubuntu-12-post-install-script.sh"
            echo "screen -S testing -d -m /bin/bash '$d_dir/Post_Install/ubuntu-12-post-install-script.sh'" >> "$dt.$hst.log"
      ;;
        "14" )
            chmod +x "$d_dir/Post_Install/ubuntu-14-post-install-script.sh"
            echo "screen -S testing -d -m /bin/bash '$d_dir/Post_Install/ubuntu-14-post-install-script.sh'" >> "$dt.$hst.log"
      ;;
        "Debian" | "debian" )
            chmod +x "$d_dir/Post_Install/debian-7-post-install-script.sh"
            screen -S testing -d -m /bin/bash "$d_dir/Post_Install/debian-7-post-install-script.sh"
      ;;
        "Arch" | "arch" )
            chmod +x "$d_dir/Post_Install/arch-post-install-script.sh"
            screen -S Arch -d -m /bin/bash "$d_dir/Post_Install/arch-post-install-script.sh"
      ;;
      esac
    }

    os;
    base_setup;
    grab_files;
    update_system;
