hetzner-ocp
===========
My setup for OpenShift on a Hetzner server

What is this about?
-------------------
I want to run OpenShift based on RHEL on a Hetzner server.

Preparing a RHEL image for the Hetzner server
---------------------------------------------
based on
https://keithtenzer.com/2019/10/24/how-to-create-a-rhel-8-image-for-hetzner-root-servers/
and
http://boernig.de/wordpress/2018/07/03/running-rhel-on-hetzner-hosted-servers/

* download "Red Hat Enterprise Linux 8.2 Binary DVD" from https://access.redhat.com/downloads/content/479/ver=/rhel---8/8.2/x86_64/product-software
* create a virtual machine based on the downloaded ISO 
* login into the virtual RHEL system and run
```
[root@localhost ~]# rm /boot/vmlinuz-0-rescue-*
[root@localhost ~]# rm /boot/initramfs-0-rescue-*.img
[root@localhost ~]# ln -s /usr/bin/dracut /sbin/dracut
[root@localhost ~]# subscription-manager register
[root@localhost ~]# subscription-manager attach
[root@localhost ~]# subscription-manager repos --disable=*
[root@localhost ~]# subscription-manager repos \
                      --enable=rhel-8-for-x86_64-baseos-rpms \
                      --enable=rhel-8-for-x86_64-appstream-rpms \
                      --enable=rhel-8-for-x86_64-highavailability-rpms \
                      --enable=ansible-2.9-for-rhel-8-x86_64-rpms \
                      --enable=openstack-15-for-rhel-8-x86_64-rpms
[root@localhost ~]# yum install -y mdadm tar screen vim
[root@localhost ~]# tar cJvf CentOS-75-el-x86_64-minimal.tar.xz --exclude=/dev --exclude=/proc --exclude=/sys --exclude=/root/CentOS-75-el-x86_64-minimal.tar.xz /
[root@localhost ~]# ssh-keygen
[root@localhost ~]# cat .ssh/id_rsa.pub
```
* now copy the output of the last command (starting with `ssh-rsa` and ending with `root@localhost.localdomain`) and paste it into the authorized_keys file on your Hetzner server running in rescue mode
```
root@rescue ~ # cat >> .ssh/authorized_keys <<EOF
[paste your copied key here]
EOF
```
* from your RHEL virtual system now run
```
[root@localhost ~]# scp CentOS-75-el-x86_64-minimal.tar.xz root@hetzner_ip:/root
```
* in your Hetzner server run (adapt the following content to your needs)
```
root@rescue#  cat > config.txt <<EOF
DRIVE1 /dev/sda 
DRIVE2 /dev/sdb 
SWRAID 1 
SWRAIDLEVEL 1
BOOTLOADER grub 
HOSTNAME jos.lab 
PART /boot ext3 512M 
PART lvm vg0 150G 

LV vg0 root / ext4 50G 
LV vg0 swap swap swap 8G 
LV vg0 var /var ext4 10G 
LV vg0 tmp /tmp ext4 30G

IMAGE /root/CentOS-75-el-x86_64-minimal.tar.xz
EOF
```
* and finally let the Hetzner installation tool install your RHEL system
```
installimage -a -c config.txt
```
* after the installation (you can safely ignore the error about CentOS specific functions in case you haven't subscribed the RHEL system yet), run
```
yum -y update
```

