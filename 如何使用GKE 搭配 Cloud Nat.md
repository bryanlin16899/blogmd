*2024-01-12*

[[technical]]

This article will explain how GCP cloud nat works.I will use GKE as service that behind the cloud nat
## Create VPC (virtual private network)
create a new VPC, your service and cloud nat will be in this VPC,
If you want to set your region in specific area, you can select when you create subnet
![[Pasted image 20240131101928.png|500]]
## Claim Static external IP in IP Addresses
Our cloud nat will need a IP address
![[Pasted image 20240131103717.png|500]]

## Create Cloud Nat
First of all, choose the network that we just created.
Then you need to create a cloud router in this section
![[Pasted image 20240131104030.png|500]]
## Create GKE cluster
Notice that region must be same with you VPC subnet.
- In step 2, choose network and subnet that we created
- enable private cluster for security (optional)
- enable access control plane ip (optional)
![[Pasted image 20240131104515.png|500]]
![[Pasted image 20240131105152.png|500]]
![[Pasted image 20240131105240.png|500]]

## Check external IP from Pod to network
All set! you can run a ubuntu pod to check external IP is same with cloud nat IP
1. Create a pod that run with ubuntu system
```shell
$ kubectl apply -f https://gist.githubusercontent.com/tcdowney/b8a0297241b74f94ef1fc6627f7ea69a/raw/eaae035f5adca37ca00d4a49f1c1958fe3db89e3/ubuntu-sleep.yaml
```
2. Enter into pod's shell
```shell
$ kubectl exec -it ubuntu -- /bin/bash
```
3. install curl then check external ip from this pod 
```shell
$ apt update
$ apt install curl -y
$ curl ifconfig.co
>>> 44.33.222.111 (cloud nat ip) ✅
```
![[Pasted image 20240131111917.png|500]]
## Optional
### Set backup plan for old gke cluster
6. 備份原本的cluster
	1. 建立備份方案
	2. 手動執行備份
7. 還原到新的cluster
8. load balancer更新
	1. 刪除新舊的service
	2. 重新到新的cluster中apply，建立新的load balancer

