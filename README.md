# hetzner-ocp
My setup for OpenShift on a Hetzner server

## What is this about?
I want to run OpenShift based on RHEL on a Hetzner server.

## Preparing a RHEL image for the Hetzner server
based on
https://keithtenzer.com/2019/10/24/how-to-create-a-rhel-8-image-for-hetzner-root-servers/
http://boernig.de/wordpress/2018/07/03/running-rhel-on-hetzner-hosted-servers/

* download "Red Hat Enterprise Linux 8.2 Update KVM Guest Image" from https://access.redhat.com/downloads/content/479/ver=/rhel---8/8.2/x86_64/product-software
* run virt-customize -a <qcow2 image file name> --root-password password:<password> --uninstall cloud-init
* import the image into virt-manager naming it rhel8 (you can choose a value for RAM as low as 4GB)
* login with virsh console rhel8
* [root@localhost ~]# rm /boot/vmlinuz-0-rescue-adcc72dfe3ed4c049ffff0ec950a90d9
* # rm /boot/initramfs-0-rescue-adcc72dfe3ed4c049ffff0ec950a90d9.img
* # ln -s /usr/bin/dracut /sbin/dracut
  subscription-manager register
  subscription-manager attach
  subscription-manager repos --disable=*
  subscription-manager repos \
    --enable=rhel-8-for-x86_64-baseos-rpms \
    --enable=rhel-8-for-x86_64-appstream-rpms \
    --enable=rhel-8-for-x86_64-highavailability-rpms \
    --enable=ansible-2.9-for-rhel-8-x86_64-rpms \
    --enable=openstack-15-for-rhel-8-x86_64-rpms
    
* # tar cJvf CentOS-75-el-x86_64-minimal.tar.xz --exclude=/dev --exclude=/proc --exclude=/sys --exclude=/CentOS-75-el-x86_64-minimal.tar.xz /
* # scp CentOS-75-el-x86_64-minimal.tar.xz root@hetzner_ip:/root
