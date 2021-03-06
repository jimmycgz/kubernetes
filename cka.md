# My Journey to Clear CKA (Certified on March 1st, 2021)

### The best training course I recommend:

Certified Kubernetes Administrator (CKA) with Practice Tests on Udemy
https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/

It offers labs to exercise after each training session, also has Lighting Labs and Mock exams at the end.
Just go through the full course and then practice as many times (5+) as you can with all of the labs, especially below ones:

* Ingress
* Networkpolicy
* Install and upgrade k8s cluster via kubeadm
* PV, PVC, storage class with POD
* Lighting labs
* RBAC
* POD: Static Pod, multi-containers, init-containers
* svc
* deployment
* All Mock exams

More to practice:
* Assume the k8s is crashed, how could you recover/restore?

## The only Tips worked for me
Fast, time management is key. How to answer it quickly and correctly.

## More tips:

### Is it Difficult?
Not at all for me, after working in this area for some years. So I thought I would get 95% after writing it, but actually I lost points for several tasks and don't even know why. I thought I knew every piece of the concept and knowledge and theory, but many of the tasks don't specify how to answer it via what way. Eg: I know there are 7 pods are up running, but I'm not sure how should I write this number to the output file, may I `echo 7 > filename.txt`, or manually edit the file, or I must use `jsonpath and wc -l`? Hopefully you can figure out that for me when you pass it. 

### Mistakes during the exam
First of all read the questions carefully, because you'll very easily to make mistake, like below:
* Forgot switching over contexts
* Done it in a wrong namespace, not every step goes to Default ns in one task.
* Didn't specify some arguments on the command, like --target-port when exposing a svc

### Shall I use aliases? May I use them during the exam?
I do want to use aliases and don't like auto-completion. But I can't guarantee the proctor will allow you to download anything. I listed 50 aliases in Notepad and asked her if I could copy/paste them to the exam terminal? The answer I got is: the Notepad is not allowed. So take your own risk here...

Find my 50 golden aliases here: https://github.com/jimmycgz/dotfiles/blob/master/k8s

### Can I use my own dotfiles for aliases and vim config?
Someone said yes who did that on 3 hours version. I was not allowed to install anything on the 2 hours version. 

## Some Notes: 
* Many of the public training courses, tips, and example tests on the internet are for the old-3-hour version, don't waste too much time on them.
* You have one time free re-take, so don't be too stressed before and during the first exam, you'll get to know what to practice more and more if failed the first time.


## Sources

### Details about the exam
https://github.com/cncf/curriculum/

### What you can access during the exam:
* The official doc during
https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/

### What you don't need to spend too much time on
* Creating yaml files from scratch, just copy/paste from the official doc

### Build your own cluster on MAC PC using Virtualbox, Vagrant with Ubuntu 20.04 image, details see below exercise.
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

#### Strongly recommend to disable all of the bash scripts in the vagrant file, then bootstrap the master and worker nodes manually by yourself, you'll learn all the steps via kubeadm.

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
Copy the admin.conf from the master node to local PC as kube config
```
mv ~/.kube/config ~/.kube/bkp-conf
ssh kmaster 'sudo cp /etc/kubernetes/admin.conf /home/vagrant'
ssh kmaster 'sudo chown vagrant:vagrant /home/vagrant/admin.conf'
scp kmaster:/home/vagrant/admin.conf ~/.kube/config
kg po
```

### How to destroy the cluster and start again
Just reset on each master and worker node, then init the master and join the worker(s). If you wanna change the dns/hosts config for any node, make sure the legacy ones are reachable when reseting, and verify the new endpoints before re-building. 
```
sudo kubeadm reset -f
```
