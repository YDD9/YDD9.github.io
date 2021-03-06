---
layout: post
title:  "VirutalBox Linux"
date:   2017-11-14 14:58:39 +0100
comments: true 
categories: VirtualBox Linux
---


Linux in VirtualBox

- [Start with virtual box](#start-with-virtual-box)
- [Linux file read](#linux-file-read)
- [CoreOS linux inside VirtualBox](#coreos-linux-inside-virtualbox)
- [SSH explained](#ssh-explained)
- [CoreOS add user](#coreos-add-user)
- [Debian OS](#debian-os)   
        - [remote connection Bridge](#remote-connection-bridge)    
        - [(Optional) remote connection NAT](#optional-remote-connection-nat)   
        - [(Optional) Remote connection host-only](#optional-remote-connection-host-only)   
- [Share host folder](#share-host-folder)
- [Linux network](#linux-network)
- [Linux desktop resize](#linux-desktop-resize)


# Start with virtual box

To play with linux virtual machine, you can choose to use VMplayer or VirtualBox or HyperV on win10.
VirtualBox from Oracle has a better licence.

You can change VirtualBox [language](http://ccm.net/faq/40865-virtualbox-how-to-change-the-language-settings)


You can download your Linux image and verify the download [checksum](https://www.lifewire.com/validate-md5-checksum-file-4037391), make sure that your download is perfect.
```
certutil -hashfile bodhi-4.1.0-64.iso MD5 
```
output should be the same as your MD5 file, if the values don't match then the file is not valid and you should download it again.


To share your host harddisk with guest linux [virtual machine](https://www.youtube.com/watch?v=h8cA3pIAeZQ).
https://www.htpcbeginner.com/mount-virtualbox-shared-folder-on-ubuntu-linux/


# Linux file read
[cat vs grep vs awk vs sed comparison](https://askubuntu.com/questions/564944/cat-vs-grep-vs-awk-command-get-the-file-content-which-one-is-more-efficient-and)
```
# show your centOS version
$ cat /etc/centos-release
$ awk 1  /etc/centos-release

$ time for i in {1..1000}; do cat temp.txt; done
$ time for i in {1..1000}; do grep "$" temp.txt; done
$ time for i in {1..1000}; do awk "/$/" temp.txt; done
```

# CoreOS linux inside VirtualBox
CoreOS is widely used for Docker, kubernetes and cloud based VMs. So it's time to see how to create CoreOS linux with VirtualBox.  
If your host OS is linux as well, then you can follow official guide https://coreos.com/os/docs/latest/booting-on-virtualbox.html

In my case host OS is win10, so I follow https://deis.com/blog/2015/coreos-on-virtualbox/ and https://gist.github.com/noonat/9fc170ea0c6ddea69c58 do all steps manually.  
A tutorial with screenshots https://linoxide.com/distros/install-coreos-virtualbox-iso/  
Two tutorial with video https://www.youtube.com/watch?v=yiWa0KFJDfI and https://www.youtube.com/watch?v=jBXcWsGY540   

Several important things to know:   
memery should provide at least 2G, otherwise you will have kernel panic, out of memory errors when start up iso.  
![storage settings must be carefull:]({{ site.url }}/images/VirtualBoxStorage.png)

once CoreOS start, by default it login as user core without password. Then you start to cloud_config.yml to create users, ssh keys...
https://coreos.com/os/docs/latest/cloud-config.html    
This file is saved /home/core by run `cat > cloud_config.yml <contents...>`, ctl+z quit the edit. Inside the contents, you can't not move left,right,up and down, so type carefully. You could try install nano etc follow https://unix.stackexchange.com/questions/203606/is-there-any-way-to-install-nano-on-coreos but not tested
1. Boot up the CoreOS box and connect as the core user   
2. Run the /bin/toolbox command to enter the stock Fedora container.    
3. Install any software you need. To install nano in this case, it would be as simple as doing a dnf -y install nano (dnf has replaced yum)    
4. Use nano to edit files. "But wait -- I'm in a container!" Don't worry -- the host's file system is mounted at /media/root when inside the container. So just save a sample file at /media/root/home/core/cloud-config.yml, then exit the container, and finally go list the files in /home/core. Notice your cloud-config.yml file?    

Or you can setup FTP server on host win10 https://youtu.be/Cd2k9nQSdro, then upload cloud_config.yml there and download to CoreOS via 
https://superuser.com/questions/265062/downloading-file-from-ftp-using-curl
```
curl -u 'user:password' 'ftp://mysite/%2fusers/myfolder/myfile/raw' -o ~/cloud-config.yml
```

ssh rsa public key, you can generate from git bash as instructed by https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/
just keep press enter to ignore password for your private key, then paste the generated public key to below file. If you have generated ssh before, just use them    

```
#!
#cloud-config
users:
- name: nitish
  # passwd are hashed and generated by    sudo openssl passwd -1
  # passwd generation on Linux do not do it on win10
  passwd: $1$yxV9YDKT$s.fAj5dlFyrPwrH0xAQJy/
  groups:
    - sudo
    - docker
  ssh_authorized_keys:
    - "ssh_rsa AAAAB3NzaC1...snip"
```
Valide this cloud-config.yml in website https://coreos.com/validate/

Reboot and install CoreOS into the harddisk /dev/sda  https://coreos.com/os/docs/latest/installing-to-disk.html    
```
sudo coreos-install -d /dev/sda -C stable -c ~/cloud-config.yml
```
Run `sudo fdisk -l`. You should have a /dev/sda device. 

Then you can poweroff `sudo shutdown -h now` and unmount ISO in CD and continu further test.   

Connecting the CoreOS with ssh, you need config ![port forwarding]({{ site.url }}/images/VirtualBoxPortForwarding.png)
open a cmd in your Host win10
```
>> ssh -p 22022 core@127.0.0.1
Last login: Thu Apr 23 15:50:31 2015 from 192.168.59.3
CoreOS stable (633.1.0)
core@localhost ~ $


# https://www.cyberciti.biz/faq/force-ssh-client-to-use-given-private-key-identity-file/
>> ssh -p 2222 -i 'c:\user\ydd9\.ssh\id_rsa' core@ip
```

# SSH explained   
https://wiki.debian.org/SSH    
In addition this directory contains the private/public key pairs identifying your host :   

ssh_host_dsa_key   
ssh_host_dsa_key.pub    
ssh_host_rsa_key   
ssh_host_rsa_key.pub   

Since OpenSSH 5.73, a new private/public key pair is available:    
  
ssh_host_ecdsa_key   
ssh_host_ecdsa_key.pub    

Since OpenSSH 6.54, a new private/public key pair is available:   

ssh_host_ed25519_key   
ssh_host_ed25519_key.pub   

if on your local Windows machine, key files: `~/.ssh/id_rsa` and `~/.ssh/id_rsa.pub` exist already you can use them without creating again, otherwise create 
```
ssh-keygen -t rsa -b 2048 -C user@localMachineNameOrIp
```

Next you need either manually or via command to transfer your local machine public key file contents to remote linux.<br />
[Add Your Public Key to the Linux Machine](https://www.codeproject.com/Articles/497728/HowplustoplusUseplusSSHplustoplusAccessplusaplusLi)<br />

Append your local Windows machine's public key into Linux file `~/.ssh/authorized_keys` (create this file if not existing).<br />
```
# create dir
mkdir ~/.ssh && cd .ssh

# append into file
echo YourPublicKey >> authorized_keys
```
YourPublicKey eg:
ssh-rsa AAAAB3NzaC1yc2EAAAAD-PVKxPOhbnAIeDJvXQECvH6gFCW2RU2IuwuvYep/v9IBH9C0DnD6w4p47Q/MFiH+1qFZTfLcMsvu8QeB-65Rg5elvMJfW4QPAC990hx0MbeVJC6gi/zfEab8DHuEAkeVkrFDCMDrXeL5S4ykDuztgTZ2ZznwKfJv5s5/4dRIfS3mnY7cc4WRq6pvdetAmwdxJ ydd9@JSDSDSLFDS

Or your host is Linux and you have `ssh-copy-ed` installed
```
# ssh-copy-id demo@127.0.1.1
```

Or your Linux does not have `ssh-copy-ed`:
```
cat ~/.ssh/id_rsa.pub | ssh demo@198.51.100.0 "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >>  ~/.ssh/authorized_keys"
```

check remote linux ssh server is up and running<br />
https://linuxconfig.org/how-to-install-start-and-connect-to-ssh-server-on-fedora-linux   
```
$ systemctl start sshd
$ netstat -ant | grep 22
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN 
tcp6       0      0 :::22                   :::*                    LISTEN
```

if ssh core@ip always asks password https://www.digitalocean.com/community/questions/coreos-ssh-asks-for-password  
always doesn't connnect, you will see problem and a cloud-config.yaml https://github.com/coreos/bugs/issues/791:   
Make sure that your local machine public key are present in remote linux `~/.ssh/authorized_keys` and try running your SSH client in verbose mode `-v`, specifying your private key file `-i filePath` <br />
Next time your connection should be simply `ssh core@10.255.100.167` without any password worries.  
```
ssh -p 2222 -A -v -i 'c:\user\ydd9\.ssh\id_rsa' core@10.255.100.167
```



# CoreOS add user   
https://coreos.com/os/docs/latest/adding-users.html   
Be carefull about the Container Linux Config file you see, its format is not the same as cloud-config.yml  
1. add user when create the VM  

2. add user by command  
The "*" creates a user that cannot login with a password but can log in via SSH key. 
-U creates a group for the user, -G adds the user to the existing sudo group and -m creates a home directory.
If you'd like to add a password for the user, run:

```
$ sudo useradd -p "*" -U -m user1 -G sudo

$ sudo passwd user1
New password:
Re-enter new password:
passwd: password changed.
```


# Debian OS
http://www.brianlinkletter.com/installing-debian-linux-in-a-virtualbox-virtual-machine/  
Debian OS full CD ISO(not the small version netinst) is very easy to install with graphical user interface.  

During installation steps, it always failed to install standard system utilities, but it seems not a big issue for a functional Debian, just not convenient to use if some moduels you will need later.  
https://unix.stackexchange.com/questions/307600/whats-the-consequences-if-i-dont-install-the-standard-system-utilities-of-de

Fix the apt-get install error: “Media change: please insert the disc labeled ...” on your Linux VPS   
https://www.velocihost.net/clients/knowledgebase/29/Fix-the-apt-get-install-error-Media-change-please-insert-the-disc-labeled--on-your-Linux-VPS.html   
```
Media change: please insert the disc labeled
 'Debian GNU/Linux 7.0.0 _Wheezy_ - Official amd64 CD Binary-1 20130504-14:44'
in the drive '/media/cdrom/' and press enter

$ nano /etc/apt/sources.list 
or
$ vi /etc/apt/sources.list

You might find a 
deb cdrom:[Debian GNU/Linux 7.0.0 _Wheezy_ - Official amd64 CD Binary-1 20130504-14:44]/ wheezy main 
line indicating a local CDROM as a package source. 

Comment it out by placing a # symbol at the beginning of the line and save the file.
Or place cursor at the beginning of the line and type dd to delete the line, :wq save and quit vi editor.
```

To update the sources.list and config it as your own Debian distribution, here mine is stretch https://wiki.debian.org/SourcesList  
PS: Wheezy, jessie, stretch are different debian distributions, be consistent everywhere with your case!!!   
Example sources.list for Debian stretch
```
deb  http://deb.debian.org/debian stretch main contrib non-free
deb-src  http://deb.debian.org/debian stretch main contrib non-free

deb  http://deb.debian.org/debian stretch-updates main contrib non-free
deb-src  http://deb.debian.org/debian stretch-updates main contrib non-free

deb http://security.debian.org/ stretch/updates main contrib non-free
deb-src http://security.debian.org/ stretch/updates main contrib non-free
```

Then you can update Debian stretch
```
apt-get update
apt update
```

### remote connection Bridge
(network settings in VirtualBox is Bridge)
This is by my test the best, you will have internet, host <===> VM all traffic allowed.

### (Optional) remote connection NAT
(network settings in VirtualBox is NAT, needs port forwarding when host to VM)
Now install ssh in Debian
```
root@Debian: $ apt-get install openssh-server
# just for convevience
root@Debian: $ apt-get install sudo
```
Then open CMD on your host win10 and login as a no root
```
# ssh -p 2222 node1@127.0.0.1
node1 password: 
```

### (Optional) Remote connection host-only 
(network settings in VirtualBox is host-only, no internet for VMs.)
```
# reboot and login as root
# append desired IP hostanme in config file /etc/hosts
192.168.99.20   kubemaster.test.com kubemaster
192.168.99.21   kubeslave1.test.com kubeslave1

# assign this host a name
$ hostnamectl set-hostname kubemaster
$ ip a
...
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:f0:e9:6d brd ff:ff:ff:ff:ff:ff
    inet 192.168.99.100/24 brd 192.168.99.255 scope global dynamic enp0s3
       valid_lft 1119sec preferred_lft 1119sec
    inet6 fe80::a00:27ff:fef0:e96d/64 scope link
       valid_lft forever preferred_lft forever
...
```
To connect it remotely from host
```
# ssh user@192.168.99.100
```


To enable root login with password https://linuxconfig.org/enable-ssh-root-login-on-debian-linux-server  
you need to first configure SSH server. Open /etc/ssh/sshd_config and add the following line
under `# PermitRootLogin prohibit-password`  
```
PermitRootLogin yes
```
Once you made the above change restart your SSH server:
```
$ /etc/init.d/ssh restart
[ ok ] Restarting ssh (via systemctl): ssh.service.

$ or try systemctl
systemctl restart sshd.service
```

config SSH key password: two ways, and both ways start by generating SSH public key and private key
Before start, make sure `apt-get install openssh-server` is installed on remote.
### generate Secure Shell public and private key
https://www.debian.org/devel/passwordlessssh you need to generate key/pass `id_rsa.pub` / `id_rsa` on Debian with 
```
ssh-keygen -t rsa
# or you want have your email inside
ssh-keygen -t rsa -C "your_email@example.com"
``` 
and verify the created files under `~/.ssh/id_rsa` in Linux. </br>

Same command `ssh-keygen -t rsa` can be run on windows, generated key / pass are saved under `C:\Users\ydd9\.ssh` in win10.

ONE WAY</br>
copy your public key to a remote host with the command ssh-copy-id
```
ssh-copy-id -i ~/.ssh/id_rsa.pub $remote_user@$remote_host
```
This allows connection for remote_user only, if you create remote accesses for root(not good practice) and other users, repeat the command.

THE OTHER WAY</br>
It's manually copy and paste, but you get a good understand what you're doing.

Add the contents of the public key file into the file `~/.ssh/authorized_keys` on the remote (the file should be mode 600) for each users you would like to use.   
Note that once you've set this up, if an intruder breaks into your account/site, they are given access to the site you are allowed in without a password, too! For this reason, this should never be done from root.   

Below steps will force all users can't access with password, use with caution.
Even after config SSH KEY PASS, it still asks pass, then check https://askubuntu.com/questions/346857/how-do-i-force-ssh-to-only-allow-uses-with-a-key-to-log-in to config sshd_config and set `PasswordAuthentication no` then restart sshd.service     


If connection doesn't work with such warning:
```
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
```
try the key/pass generated from linux and check the 'C:\Users\ydd9\.ssh\know_host' file in windows10, `nano ~/.ssh/known_hosts` in Linux. if there's old entries with same remote IP, just delete those.


# (Optional) install Virtualbox in Debian stretch
command `sudo apt-get install virtualbox` won't work as https://wiki.debian.org/VirtualBox explains, you must do following ofr virtualbox 5.1
```
# Add virtualbox.list to /etc/apt/sources.list.d
deb http://download.virtualbox.org/virtualbox/debian stretch contrib

# Add Oracle VirtualBox public key:
wget https://www.virtualbox.org/download/oracle_vbox_2016.asc
sudo apt-key add oracle_vbox_2016.asc

# Install virtualbox-5.1
sudo apt-get update
sudo apt-get install virtualbox-5.1
```

If you go for latest version, carefully choose virtualbox.deb installer for your Debian stretch. Download manually or via wget. You can install it using sudo apt install /path/to/package/name.deb. Normally it install dependencies as well, if not do installation in two steps:
```
dpkg -i /path/to/pakcage/virtualbox-5.2_5.2.4-119785~Debian~stretch_amd64.deb
# -f fix the dependencies
apt-get -f install
```

Once you run virtualbox `/sbin/vboxconfig` to check installation, you will be asked to have a correct version of header as said below link, you can `apt-get install linux-headers-4.9.0-4-amd64`, always be concious about the versions.
https://askubuntu.com/questions/873374/virtualbox-on-ubuntu-16-04-lts-system-is-not-currently-set-up-to-build-kernel
```
error  
Please install the gcc make perl packages from your distribution.
Please install the Linux kernel "header" files matching the current kernel
```

https://forums.virtualbox.org/viewtopic.php?f=7&t=82317    

package conflicts is not good to search and install one by one, because you will face issues like avalanche.   
NOT A GOOD PRATICE IN MY POINT OF VIEW   
When you really have package conflicts issues, you can google debian packages <conflict package>,  
then downloads them and install one by one with dpkg -i <path of deb>     

Nested Virtualization  
If you see error in your Guest OS Debian an error like   
```
This computer doesn't have VT-X/AMD-v enabled. Enabling it in the BIOS is mandatory
```

You mostly have activate VT-x/AMD-v in your virtualbox settings, but it does not work, most because similar issue https://github.com/docker/machine/issues/4271 and http://wiki.osdev.org/VirtualBox


# Share host folder
Create a shared folder in WIN10;      
VB device --> [Insert guest addition CD images](https://www.howtogeek.com/189974/how-to-share-your-computers-files-with-a-virtual-machine/);    
Follow these [steps](https://virtualboxes.org/doc/installing-guest-additions-on-ubuntu/) to install the Guest Additions on your virtual machine:    

* Login to Debian;    
* Click on Applications/System/Terminal (or on Applications/Terminal, if you are using the 6.06.1 Dapper Drake release);    
* Update your APT database with `sudo apt-get update`, and typing your password, if requested;     
* Install the latest security updates with `sudo apt-get upgrade`;    
* Install required packages with `sudo apt-get install build-essential module-assistant`;   
* Configure your system for building kernel modules by running `sudo m-a prepare`;    
* Click on Install Guest Additions… from the Devices menu, then choose to browse the content of the CD when requested.
Run `sudo sh /media/cdrom/VBoxLinuxAdditions.run`, and follow the instructions on screen.

This is the scenario that you run Windows as your host operating system and Ubuntu in a VirtualBox, and that you want to access a specific Windows folder from Ubuntu.     
http://www.giannistsakiris.com/2008/04/09/virtualbox-access-windows-host-shared-folders-from-ubuntu-guest/

In  VBox settings, give a name eg. `Public` to a shared folder on win10 `c:\temp`, then in linux `mount -t vboxsf Public /media/windows-share`, make sure you create the `/media/windows-share` folder in linux and has rights to access.


# Linux network
https://wiki.debian.org/NetworkConfiguration
Note: The Linux bridge supports only STP, no RSTP (Rapid Spanning Tree). Therefore it supports only the old STP Costs, not the new RSTP Costs (see Spanning_Tree_Protocol). This is usually fine with Cisco Switches, but eg. Juniper switches use the RSTP costs and therefore this may lead to different spanning tree calculations and loop problems. This can be fixed by settings the costs manually, either on the switch or on the server. Setting the cost on the switch is preferred as Linux switches back to the default costs whenever an interface does down/up.

https://linuxconfig.org/how-to-setup-a-static-ip-address-on-debian-linux
**Enable Static IP**
By default you will find the following configuration within the /etc/network/interfaces network config file:
```
source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug eth0
iface eth0 inet dhcp
```
Update the `iface eth0 inet dhcp` to `iface eth0 inet static` and appending IP addr, netmask, gateway. 
```
# The primary network interface
allow-hotplug eth0
iface eth0 inet static
      address 10.1.1.125
      netmask 255.0.0.0
      gateway 10.1.1.1
```

# Linux desktop resize

[I have VirtualBox instance of Centos 5. The screen size is quite small (800*600) and I'd like to increase it to 1280*1080.](https://unix.stackexchange.com/questions/25236/increasing-screen-size-resolution-on-a-virtualbox-instance-of-centos?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)

A maximum resolution of 800x600 suggests that your X server inside the virtual machine is using the SVGA driver. SVGA is the highest resolution for which there is standard support; beyond that, you need a driver.

VirtualBox emulates a graphics adapter that is specific to VirtualBox, it does not emulate a previously existing hardware component like most other subsystems. The guest additions include a driver for that adapter. Insert the guest additions CD from the VirtualBox device menu, then run the installation program. Log out, restart the X server (send Ctrl+Alt+Backspace from the VirtualBox menu), and you should have a screen resolution that matches your VirtualBox window. If you find that you still need manual tweaking of your xorg.conf, the manual has some pointers.

There's a limit to how high you can get, due to the amount of memory you've allocated to the graphics adapter in the VirtualBox configuration. 8MB will give you up to 1600x1200 in 32 colors. Going beyond that is mostly useful if you use 3D.

[This is tested and working](https://unix.stackexchange.com/questions/18435/how-to-install-virtualbox-guest-additions-on-centos-via-command-line-only?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa), should also work for anybody else trying to install VirtualBox Guest Additions on a CentOS (x86_64) virtual server in command line mode.
```
# yum update
# yum install dkms gcc make kernel-devel bzip2 binutils patch libgomp glibc-headers glibc-devel kernel-headers
# mkdir -p /media/cdrom
# mount /dev/scd0 /media/cdrom
# sh /media/cdrom/VBoxLinuxAdditions.run
```
