# Installing k8s with kubeadm on arch linux

[![N|Solid](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSusgjRl2BRNe91vYfgu_Ely7ZAADZBK8iZLiszj8mCNUiWGMh6-UCZZTGr92BsxKrK7oA&usqp=CAU)](https://kubernetes.io/docs/reference/setup-tools/kubeadm/)

## Overview
We'll be using proxmox as our hypervisor and arch servers as our cluster nodes for this setup. Using arch for a cluster can be useful if you want the latest version of kubernetes running in your homelab without the hassle of managing versions of several packages that you'll need in order to setup k8s on a cluster. As arch usually provides the latest versions of most packages, you can leave the package managemet aspect to pacman when upgrading the cluster in future.
And although you could go for arch with kvm as the hypervisor for your cluster, you might want to stay on the LTS kernel if you want good compatibility  with k8s. Proxmox v7 is a good option as it has a new enough kernel version and a great web UI for managing your nodes.

## Tech

- [Proxmox v7.0](https://www.proxmox.com/en/downloads/category/iso-images-pve) - Our Hypervisor
- [Arch Linux ISO](https://archlinux.org/download/) - OS for all VMs

## Cluster node config
### My Host
16 Vcpus

32gb ram

1000gb storage

### VMs
#### kmaster x 1 
6 Vcpus

12gb ram

300gb storage

#### kworker x 2 
4 Vcpus

8gb ram

200gb storage

Rest of the resources are left for LXC containers and Proxmox itself

Ususally you don't need a bigger node for master role in k8s, but since I already had some other services running in kmaster, it has some more resources assigned to it.
Learn more about resource distribution [here](https://learnk8s.io/kubernetes-node-size)

## Installing proxmox on a host
- Download the latest version of Proxmox VE
- Burn the iso to a USB
```sh
sudo dd bs=4M if=path/to/proxmox.iso of=/dev/sd<?> conv=fdatasync  status=progress
```
- Boot into the iso
- If you have a Nvidia gpu, add an extra kernel parameter by pressing "e" and typing "nomodeset" at the end of the line that starts with "linux". Save with Ctrl+X
- Follow the installer and setup Proxmox on your drive for proxmox. Make sure there is notthing important on that drive as it will be wiped clean.

## Installing arch on all VMs

##### Manual approach
We'll be using archinstall package for a faster guided install. Make sure to create another user who has sudo access.
```sh
sudo pacman -Syyy
sudo pacman -S archinstall
archinstall
```

##### Automated approach(Optional)
You can also optionally use the sample [config](https://raw.githubusercontent.com/Naman1997/kubernetes-configs/master/kubeadm/archinstall_config.json) provided in this repo. Make sure to update the following settings:
 - Passwords for all users
 - Hostname
 - Timezone

Then install like so
```sh
sudo pacman -Syyy
sudo pacman -S archinstall git
git clone https://github.com/Naman1997/kubernetes-configs.git
cd kubernetes-configs/kubeadm/
archinstall --config archinstall_config.json
```


Setup your arch installation for all vms in the same manner. Once done, remove the installation medium and boot into the OS.

## Setting up password-less SSH access for all VMs
You'll be copy pasting a lot of stuff from and to the VMs. Setting up password-less SSH access is a good idea.
Follow these steps in order:
 - Copy contents of your ~/.ssh/id_rsa.pub file or whichever key you'll use to ssh into these VMs
 - SSH into each VM's non-root user with the password. Usually you'll need to either ssh with a non-root user or allow root ssh access from Proxmox
 - Paste contents from the 1st step in a new file : ~/.ssh/authorized_keys
## Setting up the nodes for k8s
These instructions might change in the future. Please refer to [this](https://wiki.archlinux.org/title/Kubernetes) page for reference.
### Setting up kmaster node

##### Installing dependencies
```sh
sudo pacman -S kubernetes-control-plane kubeadm kubelet containerd kubectl cni-plugins git lxd docker vi vim
sudo systemctl enable kubelet.service
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
yay -S etcd
```

##### Setting up master and control plane roles
```sh
sudo kubeadm init phase kubelet-start
sudo kubeadm init --apiserver-advertise-address=<IP address of kmaster> --pod-network-cidr=<CIDR range>
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml
```
Btw, initially I used <CIDR range> = 192.168.0.0/16 and had to patch the nodes later to fix some pods. You want to make sure nothing is running in the range that you provide.
Learn more [here](https://stackoverflow.com/a/58618952/9931915)

##### Troubleshooting

###### Pod with name like kube-flannel-ds* is in CrashLoopBackOff OR Pod with name like coredns* is in a not ready state
You might need to patch all your nodes. Example: The suggested CIDR for flannel and canal networks is 10.244.0.0/16 and for calico network it could be 192.168.0.0/16.
```sh
kubectl patch node <NODE_NAME> -p '{"spec":{"podCIDR":"10.244.0.0/16"}}'
```

##### Kubelet.service complaining that /var/lib/kubelet/config.yaml was not found
```sh
kubeadm reset
rm -rf $HOME/.kube
sudo rm -rf /etc/cni/net.d
rm /var/lib/kubelet/config.yaml
sudo systemctl stop kubelet.service
```
After the above steps retry from [kubeadm init step](https://github.com/Naman1997/kubernetes-configs/blob/master/kubeadm/README.md#setting-up-master-and-control-plane-roles)

### Setting up kworker nodes

##### Installing dependencies
```sh
sudo pacman -S kubernetes-node kubeadm kubelet containerd kubectl cni-plugins git lxd docker vi vim
sudo systemctl enable kubelet.service
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
yay -S etcd
```

##### Joining the cluster as a worker
In the kmaster node execute the below command
```sh
sudo kubeadm token create --print-join-command
```
The command above will provide a string that you need to paste and execute in the kworker nodes.
