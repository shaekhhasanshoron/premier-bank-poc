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

sudo yum update -y
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
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION="v1.32.4+k3s1" INSTALL_K3S_EXEC="--cluster-cidr=10.42.0.0/16 --service-cidr=10.43.0.0/16 --disable traefik --kubelet-arg=max-pods=500 --node-name=master-1 --node-label=svccontroller.k3s.cattle.io/lbpool=pool1 --node-label=svccontroller.k3s.cattle.io/enablelb=true" sh -
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

K10fb1307cc6dae263cfd2052149e5d4bcd94c7177157f8c1a5da3bacd179b95d08::server:4e65198a7a0396b09f4f90165053e93b
```

Run on Worker machines node machine `192.168.3.36`:
```
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION="v1.32.4+k3s1" INSTALL_K3S_EXEC="--kubelet-arg=max-pods=500 --node-label=svccontroller.k3s.cattle.io/lbpool=pool2 --node-name=worker-1 --node-label=svccontroller.k3s.cattle.io/enablelb=true" K3S_URL="https://192.168.3.38:6443" K3S_TOKEN="K10fb1307cc6dae263cfd2052149e5d4bcd94c7177157f8c1a5da3bacd179b95d08::server:4e65198a7a0396b09f4f90165053e93b" sh -
```

Run on Worker machines node machine `192.168.3.37`:
```
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION="v1.32.4+k3s1" INSTALL_K3S_EXEC="--kubelet-arg=max-pods=500 --node-label=svccontroller.k3s.cattle.io/lbpool=pool3 --node-name=worker-2 --node-label=svccontroller.k3s.cattle.io/enablelb=true" K3S_URL="https://192.168.3.38:6443" K3S_TOKEN="K10fb1307cc6dae263cfd2052149e5d4bcd94c7177157f8c1a5da3bacd179b95d08::server:4e65198a7a0396b09f4f90165053e93b" sh -
```

# Longhorn

## Installation

### Prepare Configuration

If you are installing longhorn on a VM where you have setup the kubernetes cluster.

> **INFO**: By default, Longhorn only supports ReadWriteOnce (RWO), but RWX is possible via Longhorn NFS provisioner or CSI NFS driver with Longhorn backend

for Debian/Ubuntu
```
sudo apt update

sudo apt install open-iscsi nfs-common -y  # Debian/Ubuntu
```

for rhel(Red Hat enterprise linux)
```
sudo yum update
# or
sudo yum install iscsi-initiator-utils nfs-utils -y 
```

### Apply Manifest

You can just apply the manifest, Check the release [https://github.com/longhorn/longhorn?tab=readme-ov-file#releases](https://github.com/longhorn/longhorn?tab=readme-ov-file#releases)

```
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.8.1/deploy/longhorn.yaml
```

### Check longhorn

```
kubectl get po -n longhorn-system
```

Checkk the UI

```
kubectl port-forward -n longhorn-system svc/longhorn-frontend 8080:80
```

## Storage Path

The detault storage mount path for longhorn is `/var/lib/longhorn/`. Check if you have enough storage on that path with `lsblk` command.

If you need to add storage to `/var/lib/longhorn/` path, you can for the details here [https://github.com/shaekhhasanshoron/Important-Documents/blob/master/server/linux/disk-management/increasing-disk-partition.md](https://github.com/shaekhhasanshoron/Important-Documents/blob/master/server/linux/disk-management/increasing-disk-partition.md)

### Add new Disk or Remove Storage Mount Path

Before starting with process, you need to create a folder where you want to mount the longhorn storage path. Lets say, you want
to use `/home/data` path and `/data` does not exists under `/home`. So first create that and provide permission with the following commands:

```
cd /home

mkdir data

chown -R root:root /home/data
```

#### Add a new Disk

If you want to add new disk to the node or update the existing `/var/lib/longhorn/` follow the instructions bellow.
* Open the Longhorn UI or port forward longhorn ui using `kubectl port-forward -n longhorn-system svc/longhorn-frontend 8080:80`
* Click on the `Node` tab on the top bar. You will be able to see all the nodes of the cluster
* Select the desired cluster and expand by click `+` button,
* Go to the right in the `Operations` colum and click on the drop down option and click `Edit node and disk`
* Just add a new disk, click on the `Add Disk` button.
    * Set a disk name (not mandatory)
    * set Disk path. for example if you want to mount it `/home/data` set this path.
    * Set reserved storage to 20 Gib (or any amount you want)
    * **Important:** Make sure to enable scheduling. Click on the enable button on Scheduling
    * Apply the changes by click Apply button.

After applying, if everthing goes ok then after a while you will see the new path is scheduled. You can check it by selecting
the desired cluster and expand by click `+` button.

#### Remove a new Disk
If you want to remove existing disk from the node for example remove the existing  `/var/lib/longhorn/` follow the instructions bellow.
* Open the Longhorn UI or port forward longhorn ui using `kubectl port-forward -n longhorn-system svc/longhorn-frontend 8080:80`
* Click on the `Node` tab on the top bar. You will be able to see all the nodes of the cluster
* Select the desired cluster and expand by click `+` button,
* Go to the right in the `Operations` colum and click on the drop down option and click `Edit node and disk`. You will see all the disk that are mounted
* Click on the disable button on Scheduling for the desired disk and the click on the delete icon.
* Apply the changes by click Apply button.


## Check Longhorn

Check the storage class
```
kubectl get storageclass
```

Update the pvc storageClass on the `/storage/longhorn/manifests/pvc-rwm.yaml` and apply the manifests.

```
kubectl apply -f /storage/longhorn/manifests/pvc-rwm.yaml

kubectl apply -f /storage/longhorn/manifests/pod-mount-rwm.yaml
```

Check if the pod is running and pvc in mounted or not.

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



