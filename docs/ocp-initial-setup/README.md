## Setup OCP initial configuration

Connect again to OCP helper node by using ocphelperadmin user (Change the IP of your environment)

```sh
ssh ocphelperadmin@192.168.10.210
```

Setup OC environments

```sh
export KUBECONFIG=/home/ocphelperadmin/ocp4/auth/kubeconfig
```

Create directory

```sh
mkdir ~/.kube
```


Copy files

```sh
sudo cp /home/ocphelperadmin/ocp4/auth/kubeconfig ~/.kube/config
```

Give permission

```sh
sudo chown $USER ~/.kube/config
```

Enable bash completion for oc and kubectl commands

Update the file with below content 

source <(oc completion bash)
source <(kubectl completion bash)

```sh
sudo vi ~/.bashrc
```

Source bashrc file

```sh
source  ~/.bashrc
```
Monitor installation

```sh
openshift-install --dir ~/ocp4 wait-for bootstrap-complete --log-level=debug
```

It will show message to remove the bootstrap once installation get completed

```sh
sudo vi /etc/haproxy/haproxy.cfg
```

Remove configuration for bootstrap, so no request will go to that node
Comment the line for the bootsrap server on API and Machine configs
```sh
sudo vi /etc/haproxy/haproxy.cfg
```

Restart HA Proxy server

```sh
sudo systemctl restart haproxy.service
```

Check nodes

```sh
oc get nodes
```

Check any approvals pending

```sh
oc get csr
```

Approve pending items

```sh
oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs oc adm certificate approve
```

Check if all operatora are up and running

```sh
oc get co
```

Change directory

```sh
cd ~/ocp4/auth
```

Get default password

```sh
cat kubeadmin-password
```

OC Login to the Main H/W hardware server and add host entery as below

```sh
ssh ocpadmin@mainhwserver
```

```sh
sudo vi /etc/hosts
```

Add below host entery (Change the IP of the healper node)

```sh
192.168.10.15 console-openshift-console.apps.ocp4.massuite.online
192.168.10.15 downloads-openshift-console.apps.ocp4.massuite.online
192.168.10.15 api.ocp4.massuite.online
192.168.10.15 oauth-openshift.apps.ocp4.massuite.online
```

OCP Remote Login faile with certificate error solution

https://access.redhat.com/solutions/4505101

sudo ufw status verbose
sudo ufw allow OpenSSH
sudo ufw allow ssh
sudo ufw allow https
sudo ufw allow 6443/tcp
sudo ufw show added
sudo ufw enable

https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-18-04



Install Ubuntu RDP

https://www.digitalocean.com/community/tutorials/how-to-enable-remote-desktop-protocol-using-xrdp-on-ubuntu-22-04

