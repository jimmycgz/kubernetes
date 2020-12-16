# Preparation for CKA

## Sources

### Details about the exam
https://github.com/cncf/curriculum/blob/master/CKA_Curriculum_v1.19.pdf

### Work hard on below tasks
* https://github.com/kelseyhightower/kubernetes-the-hard-way
Cloned to $HOME/certs/cka

### What you can access during the exam:
* The official doc during
https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/

* Your own dotfile repo

### Key concepts to understand well:
* static pod vs daemon set vs sts
* 

### What you don't need to spend too much time on
* Creating yaml files from scratch, just copy/paste from the offical doc

### Build your own cluster on MAC PC using Virtualbox, Vagrant with Ubuntu 20.04 image
https://github.com/justmeandopensource/kubernetes/blob/master/vagrant-provisioning/Vagrantfile

## Exercises

### Build your own cluster on MAC PC using Virtualbox, Vagrant with Ubuntu 20.04 image
* Setup 2 VMs via Vagrant and VirtualBox, bootstrap them manually. 

### Setup 2 VMs via Vagrant and VirtualBox 
* Install latest version of Virtuabox  V6.16
You need to install the latest version if got error launching a vm: Stderr: VBoxManage ... NS_ERROR_FAILURE 
Failed installation by downloading the package from Oracle web, fixed it by enabling Oracle app installation following below post
https://support.apple.com/en-ca/guide/mac-help/mh40616/mac

Alternative option: 
``` 
brew install virtualbox
```

* Install/upgrade Vagrant by downloading the package from Hashcorp web
The package successfully upgraded Vagrant from 2.2.12 to 2.2.14

practice: https://learn.hashicorp.com/vagrant/getting-started/

* Launch 2 VMs by Vagrant, can leverage below launch template and disable all of the bootstrap scripts
https://github.com/justmeandopensource/kubernetes/blob/master/vagrant-provisioning/Vagrantfile
Run `vagrant up` in the folder where has the above launch template file
More commands
```
vagrant box add
vagrant status
vagrant box list
```

* Add hostnames to MAC local and all VMs
```
sudo nano /etc/hosts
172.16.16.100   kmaster.example.com     kmaster
172.16.16.101   kworker1.example.com    kworker1
```

* Install kubeadm and all needed components, scroll down to find the correct calico version and join command for worker if errors.
https://computingforgeeks.com/deploy-kubernetes-cluster-on-ubuntu-with-kubeadm/

  ```
  sudo nano /etc/hosts # add kmaster
  
  sudo kubeadm init --apiserver-advertise-address=172.16.16.100 --pod-network-cidr=192.168.0.0/16 
  sudo cp /etc/kubernetes/admin.conf .kube/config
  sudo chown $(id -u):$(id -g) .kube/config
  kg po
  kg ns
  k create -f https://docs.projectcalico.org/v3.16/manifests/calico.yaml  
  watch kubectl get po -A
  kg po -A
  kubeadm token create --print-join-command
  
  # Copy the last command from the above output to Join worker, don't use the host name for api endpoint or you may see error like rpc refused x.x.x.x:443 call
  wrong: sudo kubeadm join kmaster:6443 --token balu7g.ejrk2vmrghfngmmh     --discovery-token-ca-cert-hash sha256:907170746b6aa1f9da2e018b1cb0f9d5afdb2447e98530727c2723fbc6cfb1f1

  # Correct one:
  sudo kubeadm join 172.16.16.100:6443 --token kybpxx.w483w4k265sokxil     --discovery-token-ca-cert-hash sha256:610a7eafd8e53f3dab14b113e94c1d23978b4b49a672eea9b19b703e989249be
  ```
### How to ssh to the vagrant boxes

Option #1: vagrant ssh kmaster
Option #2: use normal ssh after setting ssh config
```
cp ~/.ssh/config ~/.ssh/bkp-conf
vagrant ssh-config >> ~/.ssh/config
ssh kmaster
```

### How to access the k8s cluster at local PC
Copy the admin.conf to kube config
```
mv ~/.kube/config ~/.kube/bkp-conf
scp kmaster:/home/vagrant/.kube/config ~/.kube/config
kg po
```

### How to destroy the cluster and start again
Just reset on each master and worker node, then init the master and join the worker(s). If you wanna change the dns/hosts config for any node, make sure the lagacy ones are reachable when reseting, and verify the new endpoints belore re-building. 
```
sudo kubeadm reset -f
```
