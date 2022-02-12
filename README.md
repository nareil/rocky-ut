# rocky-ut
#This file explains how to setup a UT2004 server on Rocky Linux

What you will need
compat-libstdc++-33-3.2.3-72.el7.i686.rpm
dedicatedserver3339-bonuspack.zip
ut2004-lnxpatch3369-2.tar.bz2

The following link takes you to the guide that helped me get it running
https://www.lucaswilliams.net/index.php/2017/12/04/setting-up-unreal-tournament-2004-game-server-on-ubuntu-16-04/

Create directory for UT2004 install

mkdir /usr/local/games/UT2004

Unzip the server package into the created directory

sudo unzip -d /usr/local/games/UT2004 dedicatedserver3339-bonuspack.zip

Untar the patch file and copy all files into their respective locations
This can be done as a normal user

Next, install libstdc++5 package

dpkg --add-architecture i386
wget http://mirror.centos.org/centos/7/os/x86_64/Packages/compat-libstdc++-33-3.2.3-72.el7.i686.rpm
rpm -i compat-libstdc++-33-3.2.3-72.el7.i686.rpm

Add the following to /root/.bash_profile

export LD_LIBRARY_PATH=/usr/lib/

Check the necessary libraries are installed/ what provides

dnf provides libstdc++.so.5
ldd libstdc++.so.5

Edit /usr/local/games/UT2004/System/UT2004.ini with the following

bEnabled=True
ListenPort=8080

Create server daemon (check /root/UT2004 for the config)

UT2004-Server.service

Edit it to match updated file

UT2004-Server.edited

Place daemon file in correct location

/lib/systemd/system/UT2004-Server.service

Change permissions of daemon

chmod 644 UT2004-Server.service
chown root:root UT2004-Server.service

Change permissions of /usr/local/games/UT2004/System/ucc-bin

chmod 755 ucc-bin

Start up the daemon

systemctl start UT2004-Server.service
systemctl status UT2004-Server.service

Navigate to the Webadmin page in your browser

localhost:8080

Use your credentials

ut2004/ admin
password

And voila! You should hopefully see the UT2004 Remote Webadmin for Rocky!

To allow connection via friends, do/ create the following

Change network interface from NAT to Bridged

Set static IP in /etc/sysconfig/network-scripts/ifcfg-enp0s3
Old config can be found here, under the filename oldifcfg3
IP must be on the same subnet as PC i.e 192.168.1.0/24

firewall-cmd --permanent --new-zone=ut2004
firewall-cmd --permanent --zone=ut2004 --change-interface=enp0s3
**firewall-cmd --zone=ut2004 --permanent --add-service=http
firewall-cmd --zone=ut2004 --permanent --add-service=https
**firewall-cmd --zone=ut2004 --permanent --add-service=ssh
firewall-cmd --zone=ut2004 --permanent --add-port=7777/udp
firewall-cmd --zone=ut2004 --permanent --add-port=7778/udp
firewall-cmd --zone=ut2004 --permanent --add-port=7787/udp
firewall-cmd --zone=ut2004 --permanent --add-port=7788/udp
firewall-cmd --zone=ut2004 --permanent --add-port=28902/tcp
firewall-cmd --zone=ut2004 --permanent --add-port=8080/tcp

firewall-cmd --reload
firewall-cmd --list-all --zone=ut2004
systemctl restart firewalld
systemctl status firewalld

You can test if you can connect by going to the IP in UT2004
This will prompt for the password, can be admin or game password

Navigate to 192.168.1.254
Open up Port Forwarding in advanced settings
Will require Wifi Admin Password

Select UT2004 and device to Kraken

Complete! Friends should now be able to connect via your public IP!
Must add ":7777" onto the end of my public IP to join!

**Neil Munday mentioned not to forward these ports

firewall-cmd --list-all --zone=ut2004
firewall-cmd --zone=ut2004 --permanent --remove-service=http
firewall-cmd --zone=ut2004 --permanent --remove-service=ssh
firewall-cmd --reload
firewall-cmd --list-all --zone=ut2004
systemctl restart firewalld
firewall-cmd --list-all --zone=ut2004

As I chose default UT2004 port forwarding on my router, I don't know
what ports have been forwarded. I took these rules out of Kraken's
firewall just in case. I should look up what is actually forwarded by
the default rules on my router.
