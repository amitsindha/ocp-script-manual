# Install Openshift on bare metal platform

* [User Setup](docs/adminuser)
* [Disk Space Setup](docs/disk-space/README.md)

```sh
ssh ocpadmin@your_server_ip_address
```

login with sudo access

```sh
sudo su -
```

## KVM Setup

```sh
sudo su -
apt update && apt upgrade -y
apt install bridge-utils qemu-kvm virtinst li
bvirt-daemon virt-manager -y
virsh net-list --all
cd /home/ocpadmin
mkdir ocp
cd /home/ocpadmin/ocp
wget -O public_network.xml https://raw.githubusercontent.com/amitsindha/ocp-script-manual/main/templates/public_network.xml
wget -O private_network.xml https://raw.githubusercontent.com/amitsindha/ocp-script-manual/main/templates/private_network.xml
virsh net-define /home/ocpadmin/ocp/public_network.xml
virsh net-autostart  public_network
virsh net-start public_network
virsh net-define /home/ocpadmin/ocp/private_network.xml
virsh net-autostart  private_network
virsh net-start private_network
virsh net-list --all
rm -rf public_network.xml
rm -rf private_network.xml
```

* [OCP Healper Node Setup](docs/openshift-helper-node/README.md) 

## Firewall, Network Zone and DNS 

Connect to OCP Helper node (Change your IP)

```sh
ssh ocphelperadmin@192.168.10.154
```

```sh
sudo nmcli connection modify enp2s0 connection.zone internal
sudo nmcli connection modify enp1s0 connection.zone external
firewall-cmd --get-active-zones
sudo firewall-cmd --zone=external --add-masquerade --permanent
sudo firewall-cmd --zone=internal --add-masquerade --permanent
sudo firewall-cmd --reload
sudo firewall-cmd --list-all --zone=internal
sudo firewall-cmd --list-all --zone=external
```

## OCP Zone on Bind DNS

```sh
ssh ocphelperadmin@192.168.10.154
```

```sh
sudo dnf install bind bind-utils -y
sudo cp /etc/named.conf /etc/named_bkp.conf
cd /home/ocpadmin/ocp
sudo wget -O named.conf https://raw.githubusercontent.com/amitsindha/ocp-script-manual/main/templates/named.conf
sudo cp named.conf /etc/named.conf
sudo rm named.conf 
sudo mkdir -p  /etc/named/zones/
sudo wget -O db.massuite.online https://raw.githubusercontent.com/amitsindha/ocp-script-manual/main/templates/db.massuite.online
sudo cp db.massuite.online /etc/named/zones/db.massuite.online 
sudo rm db.massuite.online 
sudo wget -O db.reverse https://raw.githubusercontent.com/amitsindha/ocp-script-manual/main/templates/db.reverse
sudo cp db.reverse /etc/named/zones/db.reverse
sudo rm db.reverse
sudo chown named:named /etc/named/zones/db.massuite.online
sudo chown named:named /etc/named/zones/db.reverse
sudo firewall-cmd --add-port=53/udp --zone=internal --permanent
sudo firewall-cmd --add-port=53/tcp --zone=internal --permanent
sudo firewall-cmd --reload
sudo systemctl enable named
sudo systemctl start named
#sudo systemctl status named
#ping ocp-healper
sudo wget -O resolv.conf https://raw.githubusercontent.com/amitsindha/ocp-script-manual/main/templates/resolv.conf
sudo cp resolv.conf /etc/resolv.conf
sudo rm resolv.conf
sudo wget -O 90-dns-none.conf https://raw.githubusercontent.com/amitsindha/ocp-script-manual/main/templates/90-dns-none.conf
sudo cp 90-dns-none.conf /etc/NetworkManager/conf.d/90-dns-none.conf
sudo rm 90-dns-none.conf
sudo systemctl reload NetworkManager
#ping ocp-healper
```

## DHCP Setup

```sh
ssh ocphelperadmin@192.168.10.210
```

```sh
sudo  dnf install dhcp-server -y
cd /home/ocpadmin/ocp
sudo wget -O dhcpd.conf https://raw.githubusercontent.com/amitsindha/ocp-script-manual/main/templates/dhcpd.conf
sudo cp dhcpd.conf /etc/dhcp/dhcpd.conf
sudo rm dhcpd.conf 
sudo firewall-cmd --add-service=dhcp --zone=internal --permanent
sudo firewall-cmd --reload
sudo systemctl enable dhcpd
sudo systemctl start dhcpd
#sudo systemctl status dhcpd 
```

## Apache Web server and HAproxy


```sh
ssh ocphelperadmin@192.168.10.210
```


```sh
sudo dnf install httpd -y
cd /home/ocpadmin/ocp
sudo sed -i 's/Listen 80/Listen 0.0.0.0:8080/' /etc/httpd/conf/httpd.conf
sudo firewall-cmd --add-port=8080/tcp --zone=internal --permanent
sudo firewall-cmd --reload
sudo systemctl enable httpd 
sudo systemctl start httpd
sudo dnf install haproxy -y
sudo cp /etc/haproxy/haproxy.cfg /etc/haproxy/bkp_haproxy.cfg
sudo wget -O haproxy.cfg https://raw.githubusercontent.com/amitsindha/ocp-script-manual/main/templates/haproxy.cfg
sudo cp haproxy.cfg /etc/haproxy/haproxy.cfg
sudo rm haproxy.cfg 
sudo firewall-cmd --add-port=6443/tcp --zone=internal --permanent
sudo firewall-cmd --add-port=6443/tcp --zone=external --permanent
sudo firewall-cmd --add-port=22623/tcp --zone=internal --permanent 
sudo firewall-cmd --add-service=http --zone=internal --permanent
sudo firewall-cmd --add-service=http --zone=external --permanent
sudo firewall-cmd --add-service=https --zone=internal --permanent 
sudo firewall-cmd --add-service=https --zone=external --permanent 
sudo firewall-cmd --add-port=9000/tcp --zone=external --permanent
sudo firewall-cmd --reload
sudo setsebool -P haproxy_connect_any 1 
sudo systemctl enable haproxy
sudo systemctl start haproxy
#sudo systemctl status haproxy
```

## NFS Server for OCP Registery


```sh
ssh ocphelperadmin@192.168.10.210
```

```sh
sudo dnf install nfs-utils -y
cd /home/ocpadmin/ocp
sudo mkdir -p /shares/registry
sudo chown -R nobody:nobody /shares/registry
sudo chmod -R 777 /shares/registry
sudo wget -O exports https://raw.githubusercontent.com/amitsindha/ocp-script-manual/main/templates/exports
sudo cp exports /etc/exports
sudo rm exports
sudo exportfs -rv
sudo firewall-cmd --zone=internal --add-service mountd --permanent
sudo firewall-cmd --zone=internal --add-service rpc-bind --permanent
sudo firewall-cmd --zone=internal --add-service nfs --permanent
sudo firewall-cmd --reload
sudo systemctl enable nfs-server rpcbind
sudo systemctl start nfs-server rpcbind nfs-mountd
sudo systemctl status nfs-server
```

## TFTP Setup


```sh
ssh ocphelperadmin@192.168.10.210
```

```sh
sudo yum -y install tftp-server syslinux
cd /home/ocpadmin/ocp
sudo firewall-cmd --zone=internal --add-service=tftp --permanent
sudo firewall-cmd --reload
sudo wget -O helper-tftp.service https://raw.githubusercontent.com/amitsindha/ocp-script-manual/main/templates/TFTP/helper-tftp.service
sudo cp helper-tftp.service /etc/systemd/system/helper-tftp.service
sudo rm helper-tftp.service
sudo tee /usr/local/bin/start-tftp.sh<<EOF
#!/bin/bash
/usr/bin/systemctl start tftp > /dev/null 2>&1
##
##
EOF
sudo chmod a+x /usr/local/bin/start-tftp.sh
sudo systemctl daemon-reload
sudo systemctl enable --now tftp helper-tftp
sudo mkdir -p  /var/lib/tftpboot/pxelinux.cfg
sudo cp -rvf /usr/share/syslinux/* /var/lib/tftpboot
sudo mkdir -p /var/lib/tftpboot/rhcos
wget https://mirror.openshift.com/pub/openshift-v4/x86_64/dependencies/rhcos/4.11/latest/rhcos-installer-kernel-x86_64
sudo mv rhcos-installer-kernel-x86_64 /var/lib/tftpboot/rhcos/kernel
wget https://mirror.openshift.com/pub/openshift-v4/x86_64/dependencies/rhcos/4.11/latest/rhcos-installer-initramfs.x86_64.img
sudo mv rhcos-installer-initramfs.x86_64.img /var/lib/tftpboot/rhcos/initramfs.img
sudo restorecon -RFv  /var/lib/tftpboot/rhcos
ls /var/lib/tftpboot/rhcos
sudo mkdir -p /var/www/html/rhcos
wget https://mirror.openshift.com/pub/openshift-v4/x86_64/dependencies/rhcos/4.11/latest/rhcos-live-rootfs.x86_64.img
sudo mv rhcos-live-rootfs.x86_64.img /var/www/html/rhcos/rootfs.img
sudo restorecon -RFv /var/www/html/rhcos
sudo chcon -R -t httpd_sys_content_t /var/www/html/rhcos/
sudo chown -R apache: /var/www/html/rhcos/
sudo chown -R apache: /var/www/html/rhcos/
sudo chmod 755 /var/www/html/rhcos/
sudo wget -O 01-52-54-00-4e-a5-b6 https://raw.githubusercontent.com/amitsindha/ocp-script-manual/main/templates/TFTP/01-52-54-00-4e-a5-b6
sudo cp 01-52-54-00-4e-a5-b6 /var/lib/tftpboot/pxelinux.cfg/01-52-54-00-4e-a5-b6
sudo rm 01-52-54-00-4e-a5-b6
sudo wget -O 01-52-54-00-62-b0-cb https://raw.githubusercontent.com/amitsindha/ocp-script-manual/main/templates/TFTP/01-52-54-00-62-b0-cb
sudo cp 01-52-54-00-62-b0-cb /var/lib/tftpboot/pxelinux.cfg/01-52-54-00-62-b0-cb
sudo rm 01-52-54-00-62-b0-cb
sudo wget -O 01-52-54-00-ac-c8-d2 https://raw.githubusercontent.com/amitsindha/ocp-script-manual/main/templates/TFTP/01-52-54-00-ac-c8-d2
sudo cp 01-52-54-00-ac-c8-d2 /var/lib/tftpboot/pxelinux.cfg/01-52-54-00-ac-c8-d2
sudo rm 01-52-54-00-ac-c8-d2
sudo wget -O 01-52-54-00-ac-1b-49 https://raw.githubusercontent.com/amitsindha/ocp-script-manual/main/templates/TFTP/01-52-54-00-ac-1b-49
sudo cp 01-52-54-00-ac-1b-49 /var/lib/tftpboot/pxelinux.cfg/01-52-54-00-ac-1b-49
sudo rm 01-52-54-00-ac-1b-49
sudo wget -O 01-52-54-00-08-56-ed https://raw.githubusercontent.com/amitsindha/ocp-script-manual/main/templates/TFTP/01-52-54-00-08-56-ed
sudo cp 01-52-54-00-08-56-ed /var/lib/tftpboot/pxelinux.cfg/01-52-54-00-08-56-ed
sudo rm 01-52-54-00-08-56-ed
sudo wget -O 01-52-54-00-a0-5a-d2 https://raw.githubusercontent.com/amitsindha/ocp-script-manual/main/templates/TFTP/01-52-54-00-a0-5a-d2
sudo cp 01-52-54-00-a0-5a-d2 /var/lib/tftpboot/pxelinux.cfg/01-52-54-00-a0-5a-d2
sudo rm 01-52-54-00-a0-5a-d2
sudo wget -O 01-52-54-00-47-5d-95 https://raw.githubusercontent.com/amitsindha/ocp-script-manual/main/templates/TFTP/01-52-54-00-47-5d-95
sudo cp 01-52-54-00-47-5d-95 /var/lib/tftpboot/pxelinux.cfg/01-52-54-00-47-5d-95
sudo rm 01-52-54-00-47-5d-95
sudo wget -O 01-52-54-00-85-41-f8 https://raw.githubusercontent.com/amitsindha/ocp-script-manual/main/templates/TFTP/01-52-54-00-85-41-f8
sudo cp 01-52-54-00-85-41-f8 /var/lib/tftpboot/pxelinux.cfg/01-52-54-00-85-41-f8
sudo rm 01-52-54-00-85-41-f8
```