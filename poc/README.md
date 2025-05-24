# POC Setup

## Tasks
* Bypass Subscription Manager on Each machine
* Disable Firewalld
* Yum update, git install and csi install for RWM
* Install K3s
* Install k3s agent
* Install Helm
* Clone KloverCloud Platform Setup Git Repo
* Install cert manager (Optional)
* Install Nginx Ingress Controller
* Install Longhorn, Update Mount Path
* Install Temporal Postgres
* Install Temporal
* Setup KloverCloud Management Cluster
* Install Harbor
* Install Prometheus
* Install Loki
* Install ArgoCD
* Install Istio
* Install Kiali
* Setup Klovercloud Agent Cluster
* Install Gitlab

## Task: Bypass Subscription Manager on Each machine
Install in every machine
```
sudo nano /etc/yum.repos.d/almalinux.repo
```

Add the following context
```
[baseos]
name=AlmaLinux $releasever - BaseOS
mirrorlist=https://mirrors.almalinux.org/mirrorlist/$releasever/baseos
enabled=1
gpgcheck=1
countme=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-AlmaLinux

[appstream]
name=AlmaLinux $releasever - AppStream
mirrorlist=https://mirrors.almalinux.org/mirrorlist/$releasever/appstream
enabled=1
gpgcheck=1
countme=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-AlmaLinux

[extras]
name=AlmaLinux $releasever - Extras
mirrorlist=https://mirrors.almalinux.org/mirrorlist/$releasever/extras
enabled=1
gpgcheck=1
countme=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-AlmaLinux
```

Save and exit the editor with (cntl + 'x' -> press 'y' --> press 'enter').

Import the AlmaLinux 9 GPG key:
````
sudo curl -o /etc/pki/rpm-gpg/RPM-GPG-KEY-AlmaLinux-9 https://repo.almalinux.org/almalinux/RPM-GPG-KEY-AlmaLinux-9
sudo rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-AlmaLinux-9
````

To prevent the subscription-manager plugin from interfering with yum:
```
sudo nano /etc/yum/pluginconf.d/subscription-manager.conf
```

Set enabled=0:

```
enabled=0
```

Save and exit the editor with (cntl + 'x' -> press 'y' --> press 'enter').

```
sudo yum clean all

sudo yum update
```

## Disable Firewalld
Run on each VM
```
systemctl stop firewalld
```

## Yum update, git install and csi install for RWM
Install in every machine
```
sudo su
```

```
yum install git iscsi-initiator-utils nfs-utils -y 
```

## Install K3s

```
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION="v1.32.4+k3s1" INSTALL_K3S_EXEC="--cluster-cidr=10.42.0.0/16 --service-cidr=10.43.0.0/16 --disable traefik --kubelet-arg=max-pods=500 --node-label=svccontroller.k3s.cattle.io/lbpool=pool1 --node-label=svccontroller.k3s.cattle.io/enablelb=true" sh -
```

If `/usr/local/bin` does not exits then add it
```
echo 'export PATH=$PATH:/usr/local/bin' >> ~/.bashrc
source ~/.bashrc
```

Copy kube-config file as `~/.kube/config`
```
mkdir -p ~/.kube

cp /etc/rancher/k3s/k3s.yaml  ~/.kube/config
```


## Install K3s Agent

Run on Master node machine `192.168.3.38`:
```
cat /var/lib/rancher/k3s/server/node-token

K107fce5074e2f88ac9e9baeda8b92edb50b78508807e795e3bfa2dea56b9e491cf::server:2971c5dc07137861a06e94711b5e40be
```

Run on Worker machines node machine `192.168.3.36`, `192.168.3.37`:
```
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION="v1.32.4+k3s1" INSTALL_K3S_EXEC="--kubelet-arg=max-pods=500 --node-label=svccontroller.k3s.cattle.io/lbpool=pool2 --node-label=svccontroller.k3s.cattle.io/enablelb=true" K3S_URL="https://192.168.3.38:6443" K3S_TOKEN="K1042c05d18567da61541ac81cafa6eb4a53467a24c5c4d45241c46629a80de86ff::server:d8af15b9d82c243eac72f2c93545a862" sh -
```

## Install Helm
Run on Master node machine `192.168.3.38`:
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

## Clone KloverCloud Platform Setup Git Repo

```
git clone https://github.com/shaekhhasanshoron/klovercloud-container-platform-setup.git
```



