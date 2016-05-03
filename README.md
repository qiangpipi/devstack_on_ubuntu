#### devstack_on_ubuntu
```
sudo apt-get update
sudo apt-get dist-upgrade
sudo apt-get install git

groupadd stack
useradd -g stack -s /bin/bash -d /opt/stack -m stack

chmod 640 /etc/sudoers
echo "stack ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
chmod 440 /etc/sudoers

su - stack
ssh-keygen -t rsa
ssh-keygen -t dsa

git clone https://git.openstack.org/openstack-dev/devstack

cd devstack
cp ./sample/local.conf
vi local.conf
```

Contents of the local.conf need to be added in control node:   
[[local|localrc]]   
ADMIN_PASSWORD=agtech123   
DATABASE_PASSWORD=$ADMIN_PASSWORD   
RABBIT_PASSWORD=$ADMIN_PASSWORD   
SERVICE_PASSWORD=$ADMIN_PASSWORD   
HOST_IP=192.169.23.41 #Modify this according to your local env   
FLAT_INTERFACE=eth0 #Modify this according to your local env   
FIXED_RANGE=172.100.100.0/24 #Modify this according to your local env   
FLOATING_RANGE=192.169.100.0/24 #Modify this according to your local env   
PUBLIC_NETWORK_GATEWAY=192.169.0.254   

MULTI_HOST=1 #If you have multiple computing node set this to 1. Otherwise 0.   

Contents of the local.conf need to be added in computing node:  
[[local|localrc]]
ADMIN_PASSWORD=agtech123   
DATABASE_PASSWORD=$ADMIN_PASSWORD   
RABBIT_PASSWORD=$ADMIN_PASSWORD   
SERVICE_PASSWORD=$ADMIN_PASSWORD   

HOST_IP=192.169.24.40 #Modify this according to your local env   

FLAT_INTERFACE=bond0 #Modify this according to your local env   
FIXED_RANGE=172.100.100.0/24 #Modify this according to your local env   

FLOATING_RANGE=192.169.200.0/24 #Modify this according to your local env   
PUBLIC_NETWORK_GATEWAY=192.169.0.254   

MULTI_HOST=1   

DATABASE_TYPE=mysql   
SERVICE_HOST=192.169.23.40   
MYSQL_HOST=$SERVICE_HOST   
RABBIT_HOST=$SERVICE_HOST   
GLANCE_HOSTPORT=$SERVICE_HOST:9292   
ENABLED_SERVICES=n-cpu,n-net,n-api-meta,c-vol   
NOVA_VNC_ENABLED=True   
NOVNCPROXY_URL="http://$SERVICE_HOST:6080/vnc_auto.html"   
VNCSERVER_LISTEN=$HOST_IP   
VNCSERVER_PROXYCLIENT_ADDRESS=$VNCSERVER_LISTEN   

```
./stack.sh
```
Volume create related as below. After rejoin-stack.sh, the vg will be created by cinder-volume automatically.
```
dd if=/dev/zero of=/opt/stack/data/stack-volumes-default-backing-file bs=1M seek=50000 count=0
dd if=/dev/zero of=/opt/stack/data/stack-volumes-lvmdriver-1-backing-file bs=1M seek=50000 count=0
sudo losetup /dev/loop0 /opt/stack/data/stack-volumes-default-backing-file
sudo losetup /dev/loop1 /opt/stack/data/stack-volumes-lvmdriver-1-backing-file
##Still neet to create vg here
##sudo vgcreate stack-volumes-default /dev/loop0
##sudo vgcreate stack-volumes-lvmdriver-1 /dev/loop1
sudo echo "dd if=/dev/zero of=/opt/stack/data/stack-volumes-default-backing-file bs=1M seek=50000 count=0" >> /etc/init.d/cinder-setup-backing-file
sudo echo "dd if=/dev/zero of=/opt/stack/data/stack-volumes-lvmdriver-1-backing-file bs=1M seek=50000 count=0" >> /etc/init.d/cinder-setup-backing-file
sudo echo "chown stack:stack /opt/stack/data/stack-volumes-default-backing-file"
sudo echo "chown stack:stack /opt/stack/data/stack-volumes-lvmdriver-1-backing-file"
sudo echo "losetup /dev/loop1 /opt/stack/data/stack-volumes-default-backing-file" >> /etc/init.d/cinder-setup-backing-file
sudo echo "losetup /dev/loop2 /opt/stack/data/stack-volumes-lvmdriver-1-backing-file" >> /etc/init.d/cinder-setup-backing-file
sudo echo "exit 0" >> /etc/init.d/cinder-setup-backing-file
sudo ln -s /etc/init.d/cinder-setup-backing-file /etc/rc2.d/S10cinder-setup-backing-file
```
Notes:
Don't forget to add access strategy for the vm. Otherwise the vms can't be pinged and sshed.   
To restart the tgt service if error occurs in cinder-volume. tgt will be stopped during unstack but not startup in rejoin.   
To manually start keystone by keystone-all &   
G-api and g-reg may have some errors in the screens but... please check if the g-api and g-reg process are running since g-api and g-reg seem not shutdowned by unstack.   
