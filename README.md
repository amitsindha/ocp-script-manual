# Install Openshift on bare metal platform

* [User Setup](docs/adminuser)

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



