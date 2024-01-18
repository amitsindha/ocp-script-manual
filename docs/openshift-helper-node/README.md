
## Create OCP Healper node which will be expose to Internet

Login to Your ubuntu server with ocpadmin user 

```sh
ssh ocpadmin@your_server_ip_address
```

login with sudo access

```sh
sudo su -
```

Navigate to ocp working directory

```sh
cd /home/ocpadmin/ocp
```

Download Centos 8 Operating System

```sh
curl -O http://mirror.uv.es/mirror/CentOS/8-stream/isos/x86_64/CentOS-Stream-8-x86_64-latest-dvd1.iso
```

Create VM Image

```sh
virt-install --name ocp-healper.massuite.online --ram 8192 \
--disk path=/var/lib/libvirt/images/ocp-healper-server.qcow2,size=800 \
--vcpus 4 --os-variant centos-stream8 \
--network network:public_network \
--network network:private_network \
--graphics none --console pty,target_type=serial \
--location /home/ocpadmin/ocp/CentOS-Stream-8-x86_64-latest-dvd1.iso \
--extra-args 'console=ttyS0,115200n8 serial'
```

During the Installation System will ask you to update some information as below,

![image](https://github.com/amitsindha/Openshift-local/assets/6096922/c698f5d9-531c-4106-9982-7a4ee35260d2)

```sh
Step 1: Press 8 - to set the root password
Step 2: Press 5 - Installation Destination (Automatic pastitioning selected)
        Press 1 to select DISK Partition
        Press C to coninue
        Press C to use all disks pace
        Press C to use LVM
Step 3: Press B to begin the installation

After installation, it will ask to accept the license information. 
        Press 1 to accept license
        Press 2 to read accept license
        Press C to continue
```

Once installation will completed, then it will ask to accept the license

![image](https://github.com/amitsindha/Openshift-local/assets/6096922/c890d886-67fd-485d-ae16-fca973fffa0c)

```sh
Step 1: Press 1 to accept license
        Press 2 to read and accept the license
        Press C to continue
```

It will ask for login (fill user id = root) and provide the password)
        
Edit Network interface settings (Public network) - Note interface name can be different in your environment. 

```sh
nmtui-edit enp1s0
```

If you can not find the interface name then execute below command which will show you list of existing interface name 

```sh
nmtui
```

Select Automatically connect checkbox and click OK
        
![image](https://github.com/amitsindha/Openshift-local/assets/6096922/59b5160a-eb1c-42f3-a45d-5829b4e3f6fd)

run IP commnad to check which IP generated

```sh
ip a
```
![image](https://github.com/amitsindha/Openshift-local/assets/6096922/88deaf82-d7ba-4599-8c0b-a416a366fc79)

Disconnect from this node and re login again

Update host name on the OCP Helper node

```sh
hostnamectl set-hostname ocp-healper.massuite.online --static
hostname ocp-healper.massuite.online 
```

Create admin user on OCP helper node


## Create admin user and provide sudo access

Login to Your ubuntu server and create admin user with suo access


Adding a New User to the System

```sh
adduser ocphelperadmin
passwd ocphelperadmin
echo "ocphelperadmin ALL=(root) NOPASSWD:ALL" | tee -a /etc/sudoers.d/ocphelperadmin
chmod 0440 /etc/sudoers.d/ocphelperadmin
```

Disconnetct from the root user

```sh
exit
```

Connect again to OCP helper node by using ocphelperadmin user (Change the IP of your environment)

```sh
ssh ocphelperadmin@192.168.10.154
```

Check internet connectivity (It should give reply)

```sh
ping 8.8.8.8
ping google.com
```

Edit Network interface settings (Private network) - - Note interface name can be different in your environment. 

```sh
sudo nmtui-edit enp2s0
```

Change IPV4 configuiration from Automatic to Manual

![image](https://github.com/amitsindha/Openshift-local/assets/6096922/e3381e35-6682-4725-9475-dbb27006b313)

Clcik on Address and fille details

Address: 10.10.10.2/24
DNS Server: 127.0.0.1
Search domain: massuite.online

Important - Select the check box for Never use this network for default route,
Select the check box Automatically connect

![image](https://github.com/amitsindha/Openshift-local/assets/6096922/8efe289b-7f61-43b2-aee1-5a58effb469a)


Update host entery in the ocp healper node

```sh
vi /etc/hosts
```


Reference URL 

https://www.cyberciti.biz/faq/howto-setting-rhel7-centos-7-static-ip-configuration/





