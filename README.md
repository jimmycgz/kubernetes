# Kubernetes playground on local PC Mac OS

Refer to cka.md for how to setup everything manually.

### Task List
* Setup 3 VMs via Vagrant and VirtualBox 
* Test #1 Associate PV=>PVC=>Pod
* Test #2 Associate PV => Statefulset via volumeClaimTemplates
* Test #3 DB statefulset
* Test #4 Blue/Green deployment
* Test #5 Canary deployment

### Setup 3 VMs via Vagrant and VirtualBox 
* Install Virtuabox by downloading the package from Oracle web, see below command for Mac PC
```
brew install virtualbox
```
* Install Vagrant by downloading the package from Hashcorp web
https://learn.hashicorp.com/vagrant/getting-started/
* Provision a test website 
https://learn.hashicorp.com/vagrant/getting-started/provisioning

* Backup kube config 
* Clone test repo
cd ~/proj/k8s/
git clone https://github.com/jimmycgz/kubernetes.git

* Follow this video to setup k8s cluster using vagrantfile
https://www.youtube.com/watch?v=wPdIBeWJJsg&t=1145s
Steps are described here: https://github.com/jimmycgz/kubernetes/blob/master/docs/install-cluster-centos-7.md

* Access this cluster via host
cd ~/.kube
mv config bkp-conf-paytm
scp vagrant@kmaster.example.com:.kube/config cf-local-vagrant
k config view --flatten > cf-merged
cat cf-merged
cp cf-merged config

#Switch context by docker icon in Mac OS
kubectl get nodes

### Get faimiliar with PV/PVC/Statefulset
Storage management workflow for TEST #1:
```
Mac OS local folder of Vagrant provision 
=> /vagrant on Vagrant VM as k8s node local folder
=> PV named manual-mnt-local
=> PVC for nginx pod
```
#### Test #1 Associate PV=>PVC=>Pod
1. mount pv folder on vagrant provision folder
```
vagrant halt
mkdir ~/proj/k8s/kubernetes/vagrant-provisioning/local-vol
mkdir ~/proj/k8s/kubernetes/vagrant-provisioning/local-vol/data
cd ~/proj/k8s/kubernetes/vagrant-provisioning/local-vol
echo 'test local pv' >index.html
echo 'test data' >data/test-data.txt
vagrant up
vagrant ssh kworker1
ll /vargant
```
2. create pv volume k8s on host pc
Follow https://www.bogotobogo.com/DevOps/Docker/Docker_Kubernetes_PersistentVolumes_PersistentVolumeClaims.php
3. create pvc and associate with pv
4. Test pvc on nginx pod
k apply -f pv-pod.yamlkubectl exec -it task-pv-pod -- /bin/bash
apt-get update
apt-get install curlcurl localhost

follow below post if get stuck deleting old pv/pvc

https://stackoverflow.com/questions/51358856/kubernetes-cant-delete-persistentvolumeclaim-pvc
```
 k patch pvc task-pv-claim -p '{"metadata":{"finalizers": []}}' --type=merge
```
 
#### Test #2 Associate PV => Statefulset via volumeClaimTemplates
Storage management workflow:
```
Mac OS local folder of Vagrant provision 
=> /vagrant on Vagrant VM as k8s node local folder
=> PV named manual-mnt-local
=> PVC for statefulset nginx
=> Nginx pod relica
```
Follow below docs:
#Single static pv/pvc
https://www.bogotobogo.com/DevOps/Docker/Docker_Kubernetes_StatefulSet.php

#Dynamic multiple pv/pvc
https://www.magalix.com/blog/kubernetes-statefulsets-101-state-of-the-pods#:~:text=A%20StatefulSet%20is%20another%20Kubernetes,pods%20with%20well%2Ddefined%20names.
https://docs.openshift.com/container-platform/3.5/install_config/storage_examples/storage_classes_dynamic_provisioning.html

Delete the pvc and pvc created in Test #1
```
k patch pvc task-pv-claim -p '{"metadata":{"finalizers": []}}' --type=merge
k delete pvc task-pv-claim
kg pv
#PV status should be releasing

k delete pv task-pv-volume
```
Re-user the pv from Test #1 and associate it to statefulset nginx
```
k apply -f pv-volume.yaml
k apply -f web-stateful-pvc.yaml
k apply -f pv-volume.yaml 
kg pv
#PV status should be available

k apply -f web-stateful-pvc.yaml
kg pods
kubectl exec -it web-0 -- /bin/bash
      apt-get update
      apt-get install curl
      curl localhost
      #Outcome:test local-volume mnt from vagrant vm, associated from vagrant provision folder on local pc
```

#### Test #3 DB statefulset
PostgreSQL cluster using statefulsets https://kubernetes.io/blog/2017/02/postgresql-clusters-kubernetes-statefulsets

#### Test #4 Blue/Green deployment
Refer to the light weight approach: https://www.ianlewis.org/en/bluegreen-deployments-kubernetes
Basically switch over k8s service via spec.selector to different labels(like name and version)
```
apiVersion: v1
kind: Service
metadata: 
  name: nginx
  labels: 
    name: nginx
spec:
  selector: 
    name: nginx
    version: "1.11" # switch between 1.10 and 1.11
    ...
```

#### Test #5 Canary deployment
Refer to the light weight approach: https://kubesphere.io/docs/quick-start/ingress-canary/
Solution is to use Ingress-Nginx Annotation to define the percentage of canary-weight
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: canary
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "10"
...
```


