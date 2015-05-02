---
layout: post
title: "VirtualBox Headless on Ubuntu 12/14"
date: 2014-09-10
---

Today we are showing you how to install and run [VirtualBox](https://www.virtualbox.org/) in a headless environment. This is perfect for use on a server or even on a desktop where you would rather work from the command line. For this guide we will be using Ubuntu 12/14. The commands will be the same for RHEL/Centos/Arch once you get passed the actual installation process.  

## Installation

For Ubuntu 12 you would add the following line to `/etc/apt/sources.list`

    deb http://download.virtualbox.org/virtualbox/debian precise contrib


For Ubuntu 14 add the following entry:

    deb http://download.virtualbox.org/virtualbox/debian trusty contrib

Now we need to add the Virtualbox public key to our server. Regardless of your Ubuntu version you can do so with the following command:

    wget -q http://download.virtualbox.org/virtualbox/debian/oracle_vbox.asc -O- | sudo apt-key add -


Now that we have the key added and the appropriate updates to our sources list, let's go ahead and update everything and install the needed packages:

    sudo apt-get update
    sudo apt-get install linux-headers-$(uname -r) build-essential virtualbox-4.3 dkms


We'll also need to install the appropriate extension pack now. Go to the [VirtualBox Downloads page](http://www.virtualbox.org/wiki/Downloads), and you will find a link to the following extension pack:


    sudo VBoxManage extpack install Oracle_VM_VirtualBox_Extension_Pack-4.3.14-95030.vbox-extpack

If everything went well we can now create our Virtual machine.  Virtualbox also supports OSTypes to make the installation a bit less painful. You can see all the OS types with the command `VBoxManage list ostypes`

    VBoxManage list ostypes|grep Arch
    ID:          ArchLinux
    Description: Arch Linux (32 bit)
    ID:          ArchLinux_64
    Description: Arch Linux (64 bit)

For this guide I am creating a 64bit Arch Linux VM with a 32GB hard drive and 1GB of Ram. I am setting a variable for the VM name as I would like to be able to script this later and having a consistent naming convention helps me a lot. I'm going to go through each command and explain what we are doing here:

    VM='Arch-64bit'

Step 1: Setting the variable, this one is pretty self explanatory.

    VBoxManage createvm --name $VM --ostype "ArchLinux_64" --register

Step 2: Creating the actual Virtual machine and registering the Operating System type

    VBoxManage createhd --filename ~/VMs/$VM.vdi --size 32768

Step 3: Creating the virtual hard drive and storing it in our $HOME/VMs directory. This will also name the hard drive Arch-64bit.vdi for easier tracking.

    VBoxManage modifyvm $VM --memory 1024 --acpi on --boot1 dvd --nic1 bridged --bridgeadapter1 eth0

Step 4: We are adjusting the settings of our new VM. We are setting the RAM, first boot device and Ethernet Adapter

    VBoxManage storagectl $VM --name "IDE Controller" --add ide

Step 5: Creating a new IDE controller so that we can attach a Hard Drive and DVD drive for the install

    VBoxManage storageattach $VM --storagectl "IDE Controller" --port 0 --device 0 --type hdd --medium ~/VMs/$VM.vdi

Step 6: We are assigning the hard drive to the first IDE slot

    VBoxManage storageattach $VM --storagectl "IDE Controller" --port 1 --device 0 --type dvddrive --medium ~/ISO\ Storage/archlinux-2014.08.01-dual.iso

Step 7: Now we are attaching the DVD drive to the second IDE slot.

    VBoxHeadless --startvm $VM

Step 8: Profit? Okay maybe not..but this command will start the Virtual Machine and launch an RDP session so that you can complete the installation process. I know using RDP to install Arch makes the brain hurt a bit, but the whole process is actually quite painless. If you are new to installing Arch, check out this [Guide from Lifehacker](http://lifehacker.com/5680453/build-a-killer-customized-arch-linux-installation-and-learn-all-about-linux-in-the-process) that walks you through the process.

{<1>}![RDP Connection to Arch VM](https://dl.dropboxusercontent.com/u/131462/Images/rdp-arch.png)

Once you have installed your Virtual Machine go ahead and close out of the RDP session and then power off the VM and detach the ISO:

    VBoxManage controlvm $VM poweroff
    VBoxManage storageattach $VM --storagectl "IDE Controller" --port 1 --device 0 --type dvddrive --medium none

To start the Virtual Machine up without the RDP session you would issue the following command:

    VBoxHeadless --startvm $VM --vrde off &

To make it easier to start and stop the VM's at boot I also created an init file that you can see [here](https://gist.github.com/greyhound-forty/aeb98cb4473a09be3fe3)


With this guide we have just scratched the surface of what you can do with Vboxmanage/VboxHeadless, for a more complete list of what you can do see the [official docs](https://www.virtualbox.org/manual/ch08.html).
