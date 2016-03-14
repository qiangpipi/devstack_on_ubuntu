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

Contents of the local.conf need to be added:
[[local|localrc]]
ADMIN_PASSWORD=agtech123   
DATABASE_PASSWORD=$ADMIN_PASSWORD   
RABBIT_PASSWORD=$ADMIN_PASSWORD   
SERVICE_PASSWORD=$ADMIN_PASSWORD   
HOST_IP=192.169.23.41 #Modify this according to your local env   
FLAT_INTERFACE=eth0 #Modify this according to your local env   
FIXED_RANGE=172.100.100.0/24 #Modify this according to your local env   
FLOATING_RANGE=192.169.100.0/24 #Modify this according to your local env   
MULTI_HOST=1

```
./stack.sh
```
