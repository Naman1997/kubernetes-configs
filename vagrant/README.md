# ![](https://upload.wikimedia.org/wikipedia/commons/thumb/8/87/Vagrant.png/150px-Vagrant.png)


To get a proper k8s setup with VMs, cd into this dir and run the following commands -
```sh
vagrant up
vagrant ssh kmaster
sudo cat /etc/kubernetes/admin.conf
```

Copy the output in your kube config file. 

Check if kubectl is working -
```sh
kubectl version --short --client
```

Check if nodes are available -
```sh
kubectl get nodes
kubectl label nodes kworker1 kworker2 kubernetes.io/role=worker
```
If you're using public networking, you might need to
 ```sh
vagrant reload
```
after you have changed 'private' to 'public' in the Vagrantfile

Troubleshooting
It has been noticed that sometimes vagrant is unable to ssh into kmaster due to timeout after some time. The solution so far is to instantly ssh into the vm after a vagrant reload.
