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

## OCP Helper Node - Setup Firewall, Network Zone and DNS (Change your IP)

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

## OCP Helper Node - Setup OCP Zone on Bind DNS




