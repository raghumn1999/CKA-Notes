# CKA --> Certified Kubernetes Administrator
* Exam duration --> 3 hour --> need to check exam duration once.
* Below are some references:
  * Certified Kubernetes Administrator: https://www.cncf.io/certification/cka/
  * Exam Curriculum (Topics): https://github.com/cncf/curriculum
  * Candidate Handbook: https://www.cncf.io/certification/candidate-handbook
  * Exam Tips: http://training.linuxfoundation.org/go//Important-Tips-CKA-CKAD
![allocation](./images/allocation.png)
* Focus more on cluster architechture,installation and configuration,
  troubleshooting,scheduling,cluster maintainance,security, networking sections.
* Read CKAD notes for K8s cluster architechture and details of each component.
* To know more about the docker and containerd,read CKAD notes.

# core concepts --> covered in ckad course --> revise the ckad notes

# Section 3: Scheduling

# Manual Scheduling

**How scheduler work extactly in backend:**
* Every pod yaml have nodeName property. By default this is not set by user.
* Kubernetes will adds automatically.
* Scheduler goes to all pod and look for the pod which don't set the nameName property.
* Identity right node by run scheduler algorith,once identify node schedule pod on node by setting nodeName to node 
  and by creating bindling object.

**What will happen it no scheduler in cluster.**
* If no scheduler to monitor and schedule node,pod continue to be in pending state.

**How to schedule pod manually or can we pod without scheduler:**
* we can schedule pod to node without scheduler by adding nodeName property set to node name in yaml.
* Pod got assgined to specified node
* You can mention nodeName only at pod creation time.

**How to assign node to exiting pod:**
* By default kubernetes will not allow to modify nodeName in already created pod.
* we have to send post request to pod binding api by having binding configuration in json format.


**Scheduling the pod without scheduler component on cluster:**
we can schedule pod on cluster without by using the nodeName property


# Labels and selectors:
Labels and Selectors are standard method to group together.
Labels are property attached to each item and selector help to filter this items with the help of labels.

**How exactly you specify the labels in pod definition file:**
* Under metadata create property called labels,add labels in key value format and you can mention how many numbers of key values you need to mention.

**Selecting the pod using labels:**
* $ kubectl get po --selector app=Fronend

for selecting other k8s object replace po with other k8s object:
* $ kubectl get <k8s object> --selector app=Fronend

* kubernetes objects use labels and selectors internally to select object.
* Example: if we want to create replicaset,we can mention labels of pod in selector section to select pods.

**selecting the pod using multiple labels:**
* $ kubectl get po --selector env=prod --selector bu=finance --selector tier=frontend

**Anotations:**
* Annotations are used to record other information related to application. for example tool information,version information etc 

**Taints and Toleration:**
Taints and Toleration are used to set restriction on what pod can be scheduled on a node.

Ex: we have three node cluster,in node01 we have dedicated resoures only some pods need to schedule on node01.
we can apply taint to node01 and use toleration on pods which need to schedule on node01. In this way we can restict other pods to schedule on node01

**How to set the taint on node:**
* $ kubectl taint nodes <node-name> key=value:taint-effect
* the taint effect defines the what will happen to pod if pod don't tolerate. 
* NoSchedule --> which means no pod scheduled on node
* PreferNoSchedule --> system will try to avoid placing on node but that not garantee
* NoExecute --> this will not allow new pod to schedule and also stop running the existing pods if they don't tolerate
* $ kubectl taint nodes node1 app=blue:NoSchedule
* command to check taints associated with nodes
  $ kubectl desctribe node <node-name>  --> come down and check Taints key

Node Selectors:
* Use below syntax in pod under container section:
   nodeSelector:
     label-key: label-value
* Adding label into node
 $ kubectl label nodes <node-name> <label-key>=<label-value>
* Node selectors have limitations like we can't check two or more label to place pod, or can't select node which label match.

Node Affinity:
we can check multiple labels on node and place pod
we can tell not to schedule pod on where label is matching. 
![node-affinity1](./images/node-affinity1.png)
![node-affinity2](./images/node-affinity2.png)

* To check labels on nodes
  $  k get nodes --show-labels
* creation of deployment with 3 replicas
  $ kubectl create deploy blue --image=nginx --replicas=3
* we have many operators to use in node affinity, please read below document
  https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#operators

Static pods:
i go through the below notes, that is enough
https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/03-Scheduling/16-Static-Pods.md

Priorites:
* Priority classes help us to define priorities for different work loads.
* if high priority application need to schedule,kubernetes will terminate low priority application and schedule high priority application.
* use the below range to define priority for applications
1000000000 (highest) to -2147483648
* Seperate range defined for kubernetes internal components
  2000000000 to 1000000000
* kubectl get priorityclass --> get the priorty class for components
* How to create priority class and make use in our application to make priority
![priorityclass1](./images/priorityclass1.png)
* By default priority value for pod is zero
* if we want to set priority to all pods, set globalDefault: true in priorityclass definition file.
![priorityclass2](./images/priorityclass2.png)
* We have jobs running in cluster with priority as 5, no resources available on cluster to scheduler higher priority class.
  what will happens?
  behaviour depends on preemptionPolicy on Priorityclass defition.
  if preemptionPolicy set to PreemptLowerPriority,kubernetes will kill lower priority jobs and schedule higher priority jobs
  if preemptionPolicy set to never, kubernetes wait for resources to be available to scheduler higher priority jobs and don't kill lower priority jobs.
![priorityclass3](./images/priorityclass3.png)

  priority ranges,preemptionPolicy behavior and highest range numbers are impartant


Multiple schedulers on cluster:
* we can setup multiple scheduler on cluser if we don't want to use default scheduler
* we can create scheduler definition file, and create scheduler pod by mounting scheduler definition file.
* It required service account,role and role binding etc, get details from below kubernetes docouments
https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/#define-a-kubernetes-deployment-for-the-scheduler 
* Once scheduler created,we can use schedulerName key in pod difinition to use custom scheduler.
* Check evernts of pod to see which scheduler is used to scheduler pod
  kubectl get events -o wide 
* https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/03-Scheduling/18-Multiple-Schedulers.md
* Deploying as pod using kubeadm is latest and most used method.
  
  how to create additional scheduler and use in pod are impartant.

* Scheduling involves different stages
  **scheduling queue**: pods are queued to schedule, kubernetes will check high priority app based on priority class
  and make queue
  **filtering**: scheduler filter nodes and get suitable nodes like which node have enough cpu and memory
  **scoring**: make score on filtered nodes based on free space after reversing the resource for pod.
  which one get highest score,choose that node.
  **binding**: Bind pod to node which scored high

![configuring_profile1](./images/configuring_profile1.png)


All these activities happen on certain plugins called scheduler plugins.

![scheduler_plugins1](./images/scheduler_plugins1.png)

we can write our own custom scheduler and plugin using Extention Points
each stage have extention points.

![extention_points1](./images/extention_points1.png)

we have some more extention points in filering,scoring and binding

![extention_points2](./images/extention_points2.png)

Scheduler Profiles: it means configuring multiple scheduler profile on single scheduler.
we can create different scheduler profiles and mention it in single scheduler. this feature supported 
from kubernertes 1.18 version

![scheduler_profiles1](./images/scheduler_profiles1.png)
![scheduler_profiles2](./images/scheduler_profiles2.png)

Admission Controllers:
what if we want to do more instead of assigning role to user for list,create update and delete.
like
Permit to allow image from particular image registry
Don't allow to use latest tag in image
Don't permit to runAs root user and allow certain capabilities only.
Enforce metadata always contains labels.
This things can't be achieved by RBAC, so admission controllers comes into figure.

Admission controller help us **to implement better security measures to enforce how a cluster used**.
Number of admission controller come as pre built-in from kubernetes like Alwayspullimage,default storage class,
Event rate limit, namespace exits etc.
Ex: once the request is authenticated and autherized, namespace exits admission controller check is the namespace exits or not,if not reject the request and show the error message.

NamespaceAutoProvision admission controller is not enable by default, this controller will create namespace if don't exits then allow cluster to create pod.

Listing enabled admission controllers
$ kube-apiserver -h | grep enable-admission-plugin.

if kube-apiserver running as pod, run below command
$ kubectl exec kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep enable-admission-plugins

Enabling and Disabling admission controllers:
![admission_controllers1](./images/admission_controllers1.png)

now namespace exits and namespaceautoprovision admission controllers are replaces by namespace lifecycle admission controller. namespace lifecycle admission controller make sure that request to non-exits namespace rejected and ensure defult and kube-system namespace can't be deleted.

Validating and Mutating Admission controllers:
When we look at the namespace lifecycle admission controller,if namespace exits allow to create if not reject request, this is validating admission controller.

defult storage class admissio controller watch for request to create PVC and check storage classs mentioned in it.
if not it will modify your request to add default storage class to your request,so this type of admission controllers are called as mutating admission controllers.

there may be admission controller do both validating and mutating, generally mutating admission controller invoked first followed by validating admission controller.

if we want our own admission controller,there are two special admission controller that are mutating admission webhook and balidating admission webhook

we can configure this webhook points to a sever that host our own admission controller logic,once it hit the webhook,it make a call to admission webhook server by passing in admission review object in json format. this object have all details about the request such as user made request,type etc webhook admissin controller response true or false to allowed field to accept or reject request.

![custom_admission_controllers1](./images/custom_admission_controllers1.png)

Logging and Monitoring:

Monitoring Kubernetes Cluster Components:
* One metric server per kubernetes cluster, metric server retrives metrics from each of pods and nodes, aggregate them and stores them in-memory.
* As metric server is in-memory monitoring solution and does not store on disk as a result can't see the historical data so you need to relay on external monitoring solution like prometheus,dynatrace,datadog.

how are the metric generated on cluster?
kubelet contains the sub-components called cAdvisor(container advisor), cAdvisor is responsible for retriving performance metrics for pods and exposing them through kube-let api and make metric available to metric server.

$ kubectl top node 
$ kubectl top pod

Application logs:
$ kubectl logs -f <pod-name>

if pod have multiple containers, we have to mention container name
$ kubectl logs -f <pod-name> <container-name>


# Section 5: Application Lifecycle Management:

Rolling updates and Rollbacks
$ kubectl rollout status deploy <name-of-deploy> --> shows the rollout status
$ kubectl rollout history deploy <name-of-deploy> --> shows the revision history

upgrades: when you update definition file and apply,deployment create new replicaset and start creating pods in new replicaset.
$ kubectl set image deploy <name-of-deployment> nginx=nginx:1.9.1

![upgrades1](./images/upgrades1.png)

Rollback: when you do rollback,delete pods in new replicaset and re-create pods on old replicaset
$ kubectl rollout undo deploy <name-of-deployment> --> rollback into previous revision

![rollback1](./images/rollback1.png)

commands and arguments in kubernetes:

![command_arg](./images/command_arg.png)

Setting environment variables in application:
* env in pod definition to set environment variable

![env_variables1](./images/env_variables1.png)
![env_variables2](./images/env_variables2.png)

configruing configmaps in application:
$ kubectl create cm <configmap-name> --from-literal=<key1>=<value1> --from-literal=<key2>=<value2>
$ kubectl get cm
$ kubectl describe cm <configmap-name>

create configmap and then inject into pod.

![configmaps1](./images/configmaps1.png)

![configmaps2](./images/configmaps2.png)

configuring secrets in application:
$ kubectl create secret generic <secret-name> --from-literal=<key1>=<value1> --from-literal=<key2>=<value2>
$ kubectl create secret generic <secret-name> --from-file=app_secret.properties --> file should contains key and value and file name can be anything

while creating the secret in declarative way, encode the data with base64 and mention in file.
echo -n "mysql" | base64  --> encode the text
echo -n "base64 encoded value" | base64 --decode

![secrets1](./images/secrets1.png)

![secrets2](./images/secrets2.png)

Encrypting secret data at Rest: --> not require, just read this from ckad notes

multi container pods: 

![multi_container_in_pod](./images/multi_container_in_pod.png)

multi container pod design patterns:
* co-located containers --> As simple as both container running in a pod. both container are meant to run throughout the entire pod lifecycle
* Init-containers: Run and complete task by container before main application container start.
* sidecard container: sidecard container start first,does its task and instead of ending it continues to run throughout the lifecycle of pod.

the main difference between co-located and sidecard container is we can't control order of containers start in co-located containers and we can control in sidecard containers.

![co-located_containers](./images/co-located_containers.png)
![init-containers](./images/init-containers.png)
![sidecard-containers](./images/sidecard-containers.png)


Self healing application:
Kubernetes supports self-healing applications through ReplicaSets and Replication Controllers. The replication controller helps in ensuring that a POD is re-created automatically when the application within the POD crashes. It helps in ensuring enough replicas of the application are running at all times.

Autoscaling:
for knowledge purpose:
vertical scaling: Adding more resource to existing application like if application running in a sever, adding more resource to that server
horizontal scaling: Adding more instances or servers to server application.

![vertical_and_horizontal_scaling](./images/vertical_and_horizontal_scaling.png)

Scaling in kubernetes:
there are two type of scaling in kubernetes,
 * scaling cluster infra 
 * scaling workloads

 there are two ways of scaling in each type
 Horizontal scaling in cluster refer to increasing number of nodes on cluster
 Vertical scaling in cluster refer to increasing resources on existing nodes of cluster.

 Horizontal scaling in workloads refer to increasing number of pods 
 Vertical scaling in workloads refer to increasing resource of pods

 ![scaling_types](./images/scaling_types.png)

 how we can scale?
 Manual: Use kubeadm join to increase nodes and we don't use vertical scaling as we have bring down,increase resource and bring up in scaling infra type.
 Use kubectl scale command to increase number of pods and kubectl edit command to increase resource on existing pods

 Automated: Cluster autoscaler for scaling cluster infra.
            HPA for horizontal scaling in workloads and VPA for vertical scaling in workloads.

 ![scaling_ways](./images/scaling_ways.png)

Horizontal Pod Autoscaler:
HPA continuously monitor the metrics,increases or decreases pods based on cpu,memory or custom metrics.

 ![hpa1](./images/hpa1.png)

$ kubectl autoscale deployment <deployment-name> --cpu-percent=50 --min=1 --max=10 ---> creating hpa

$ kubectl get hpa

$ kubectl delete hpa <name-of-hpa>

 ![hpa2](./images/hpa2.png)

 ![hpa3](./images/hpa3.png)

* we can use metric server to get metric, use custom metrics adaptor to get workload related metric(internal).
we can also refer to external sources such as datadog,dynatrace etc.

 ![hpa4](./images/hpa4.png)

Vertical Pod Autoscaller:
VPA also continuosuly monitor metrics,increases or decreases pod resources(but kill pod and create new pod)

 ![vpa1](./images/vpa1.png)

VPA don't have kubernetes built-in , hence we must deploy it.

we have vertical pod autoscaler file in github repo,we have deploy using that file.
 kubectl apply -f <path-to-vpa-defintion>

three components in VPA
  VPA admission controller
  VPA updater
  VPA recommender

  VPA recommender --> it is resposible for observing the metric server 
  VPA updater track pod and terminate that pod 
  VPA admission controller start the pod creation process and use recommendation from VPA recommender

 ![vpa2](./images/vpa2.png)

no imperative command to create VPA as it is not built-in component.

 ![vpa3](./images/vpa3.png)

Section 6: Cluster Maintenance

you will perform end-to-end upgrade by yourself with running application on cluster.
you will take backup of cluster,go through to diastor and asked to bring back cluster to previous state.

Operating system upgrade:
* kubenetes will consider node as dead if node won't come back within 5min and pods on that node terminated.
* if node come back within 5min, kubenetes will recreate pods.
* if replicaset set to pod, pod are created on other node.
* time is wait for pod to come back online is known as pod eviction time and it set on controller manager 
  tolerationSeconds: 3600  --> 5min is defualt time set
   
  don't consider node will come up within 5min and do os upgrade

safer way to do os upgrade:
* purposefully drain node,so workloads move to other nodes and marked as unschedulable
  $ kubectl drain <node-name>
* reboot node, once it comes online, uncordon node to schedule pod on node
  $ kubectl uncordon <node-name>
* remember the pods are moved other node will not rollback to previous node
* $ kubectl cordon <name>

Kubernetes Releases:

$ kubctl get nodes --> version column will show which version of kubenetes installed.

* Ex: v1.33.0 --> consists of major,minor and patch
  patch will be release more offen as bug fixes
  minor release will have new features and functionalities
* New features will released first in alpha release and disable by defult. Requires a flag to enable them.
* Then new features will be released in beta and enabled by defult.

 ![kubenetes_releases1](./images/kubenetes_releases1.png)

* When you extract kubernetes zip file, all control plan components will have same version but etcd cluster and core DNS have different version as they are seperate project

 ![kubenetes_releases2](./images/kubenetes_releases2.png)

Cluster Upgrade Introduction:

* is it mandatory to have all control plan components on same version?
  no, componenst can have different version as show in below image 

 ![components_version](./images/components_version.png)

* when we can upgrade kubernets?
  kubernetes will support latest three releases
  Ex: if your in v1.10 and kubernertes released v1.11 and v1.12, you have to upgrade your cluster before v1.13 release 

 ![supported_versions](./images/supported_versions.png)

* how do will upgrade?
  upgrade into next version of your cluster. if your using v1.10, upgrade to v1.11 then to v1.12 
* upgrade plan depends on how your cluster setup.
  if your setup cluster on google cloud, google cloud provide option to upgrade cluster
  if your setup cluster using kubeadm,use kubeadm upgrade plan and kubeadm upgrade apply commands
  if your setup cluster from scratch,then manually upgrade components

* First upgrade control plane node,as your upgrading control plan node workloads on data plan node nor affected but not able to communicate using kubectl and all management activity down
* Once control plan upgrade, now time to upgrade data plane nodes.we have different stratergy to upgrade
  stratergy 1 --> bring down all worker nodes and upgrade at a time but applicaiton will not access
  stratergy 2 --> upgrade one by one, upgrade one node then go to another node 
  stratergy 3 --> add new worker node with newer software vesion,move workload into new and remove old node

Cluster upgrade using kubeadm:  --> important to practice upgrade 
* $kubeadm upgrade plan --> give details about current vesion,latest stable vesion,which component manually need to upgrade after upgraded control plan,give command to upgrade cluster and upgrade kubeadm before perform upgrade 
  upgrade kubelet on each node as kubeadm will not upgrade kubelet
 
 ![kubeadm_upgrade_plane](./images/kubeadm_upgrade_plane.png)

* Remember we have to one minor vesion at a time. 
* upgrade kubeadm first --> apt-get upgrade -y kubeadm=1.12.0-00
* Fiste upgrade master 
  $ kubeadm upgrade apply <next minor version of kubenetes>
   
  $ kubeadm upgrade apply v1.12.0 --> if we are in v1.11.0 version
* kubectl get nodes --> still show older vesion beacuse it show vesions on kubelet registered with api server 
* Depending on your cluster setup, kubelet may no be running on master node. if your using kubeadm then kubelet running on master node so upgrade kubelet
* upgrade kubelet on master node 
  $ apt-get upgrade -y kubelet=<kubelet version>
  $ apt-get upgrade -y kubelet=1.12.0-00
* restart the kubelet service --> $ systemctl restart kubelet 
* running kubectl get node show master node upgraded into newer version
* Move workload from one node into another node
  $ kubectl drain <node-name> --> move worklaods and down the node to no new pods scheduled
* upgrade kubeadm and kubelet on worker node
  $ apt-get upgrade -y kubeadm=1.12.0-00
  $ apt-get upgrade -y kubelet=1.12.0-00
* update node configuration for new kubelet
  $ kubectl upgrade node config --kubelet-version v1.12.0
* restart kubelet service
  $ systemctl restart kubelet
* we have to unmark to use worker node
  $ kubectl uncordon <node-name>
* Perform same steps to upgrade on other worker nodes

Backup and Restore Methods:

Backup resource config:
we can take back of all resources configured on cluster using below command
 $ kubectl get all --all-namespace -o yaml > all-deploy-services.yaml
we can use velero tool to take resources backup,formally called ARK by Heptlo

 ![backup_resource_config](./images/backup_resource_config.png)

Backup - etcd:
As etc configured on master , we specify the configuration where all data stores.
--data-dir=<path-to-folder>

 ![etcd_configuration](./images/etcd_configuration.png)

etcd comes with built-in snapshot solution
  $ etcdctl snapshot save <path-to-folder/snapshot.db>
  $ etcdctl snapshot status snapshot.db --> view the status of snapshot

you have to pass endpoint of etcd,cacert,cert and key to authenticate to take backup
  ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key 


Restore etcd:
stop kube-api server 
  $ systemctl stop kube-apiserver
restore snapshot
  $ etcdctl restore snapshot.db --data-dir /var/lib/etcd-from-backup
update etcd configuration by updating in etcd.service file
  update path of new folder where we restored in --data-dir option
reload service daemon
  $ systemctl daemon-reload
restart etcd
  $ systemctl restart etcd 
start kube-apiserver service
  $ systemctl start kube-apiserver

Certificate Exam tip:
Here's a quick tip. In the exam, you won't know if what you did is correct or not as in the practice tests in this course. You must verify your work yourself. For example, if the question is to create a pod with a specific image, you must run the the kubectl describe pod command to verify the pod is created with the correct name and correct image.

Notes are avaiable in security folder. Read this notes alone with see the notes given from kode kloud.

# Section 7: Security

**Security Primitives:**
* kube-api server is central to all operation,we interact with kube-apiserver through the kube control utility or by accessing the API directly.
* we need to make two types of decision
  1) who can access the cluster
  2) what can they do 
* who can access the kube-api server is defined by authentication mechanisms, mechanisms are certificates or integration with external provider like LDAP and service account creations for machines
* what can they do is defined by authorization mechanisms, we can do by Node autherization,RBAC autherization,ABAC autherization,webhook mode
* All communications within cluster between components are secured and TLS encrypted
* All pods within the cluster are accessable each other,we can restrict the access between pods using Network Policy.

**Authentication:**
* Kubernetes will not manage the users in cluster,kubernetes require static file with credentials or certificates or third party services like LDAP
* In case of service accounts for machines,kubernetes can manage them
* All users access is manages by kube-api server weather your accessing the cluster with kube control utility or API directly.
* how kube-api server authenticate the user? they are different authentication mechanisms are used such as static file with credentials or certificates or third party services like LDAP 

# TLS Basics:
* A certificate is used to guarantee trust between two parties during the transaction. For example when user try to access the web server TLS certificate ensure that communication between the users is encrypted

* There are two types of encryption
  **symmetric encryption** --> data will be encrypted and decrypted by same key, and send key over same network, hacker may collect the encrypted data and key,easily decrypt. this not secure
  **Asymmetric encryption** --> it uses the pair of keys to enable secure communication.
    Ex:
    Generate the private and public key using the ssh-keygen.it create two files id_rsa is private key and id_rsa.pub is public key. 
    Secure the server access by adding the public key to ssh autherization_keys file,so private key is available only on your local system
    and access the server using private key. for multiple servers access,copy public key into all servers.

* how Asymmetric encryption is secure?
  In server private key and public key generated using openssl command, when user try to access server public key is shared by server.
  user encrypt the data using pubic key and also hacker got public key and encrypted data, hacker don't have private key to decrypt the data,this way Asymmetric encryption is secure.

* what if hacker tweak your network to route the request to his server?
  when your request go to hacker server,you will get public key from hacker and you will encrypt data using public key shared by hacker.
  here certificate authority come into picture. if certificate is signed by himself,all of web browsers have feature to detect self signed certificate and then you should not be continue on that browse.

* how do you create original certificate?
Certificate Autherities(CA) are well know organization, they will sign and validate certificate. some of popular ones are symantec,digicert, global sign etc.

* how this signing will work?
You will create the Certificate signing request using openssl with your domain,CA will validate and sign the request. if hacker send request for signing,it will be failed in validation process. Browser will trust if certificate sign by CA.

* how does browser know CA itself is valid?for example what if certificate signed by fake CA?
The CA themself have set of public and private keys pairs,CA's use their private key to sign the certificates, the public keys of all CAsare built in into browsers, the browser uses the public key to validate that certificate was actually signed by the CA themself.
you can actually see them in settings of your browser under certificates.

**Summarize:**
* Admin uses a pair of keys to secure ssh connection to servers. Place the public key of user who need to access server on autherized_keys file under .ssh folder. Connect to server from your local using private key. Hacker don't have private key to connect.

* The server uses a pair of keys to secure https traffic.
For server to secure https traffic,admin generate the certificate signing request(CSR) to CA,CA will validate and sign the request using private key. All users have a copy of the CAs public keys. The signed certificate send back to admin. Admin configure the web application with signed certificate. whenever user access web application,server first send the certificate with public key. Browser reads the certificate and use the public keys of CA to validate is this certificate is signed by trusted CA and retrieve server's public key,then it generate symmetric key that wishes to use going forward for all communication. The symmetric key is encrypted with server public key and send back to server. server use private key to decrypt message and retrieve the symmetric key. The symmetric key is used for communication going forward.

* how server validate that the client is who they are?
In initial trust building exercises,the server can request a certificate from client,so that client must generate a pair of keys and a signed certificate from a valid CA,client can send this client certificate to server to verify who is the client. the client certificate can't be generated on server,generate client certificate, get signed from valid CA and we good to go.

The process of generating CSR,signing from CA,distributing and maintaining digital certificate is know as Public key infrastructure or PKI 
we can't encrypt data only using public key.
we can encrypt using any one them and only decrypt data with the other. 
if you encrypt data using private key,then remember anyone with your public key will be able to decrypt data.

**Naming conventions:**
* certificates with public keys are named by extention crt or pem. for server certificate server.pem or server.crt and for client certificate client.pem or client.crt
* certificates with private keys are usually with extention key or -key. private keys have word key in them or in extention.

![naming](./images/naming_in_certs.png)

**TLS in Kubernetes**
* Two primary requirements to have all the various services within the cluster to use server certificate and all clients to use client certificates
   * Server certificates for servers
   * Client certificates for clients

Server certificates for server:
* Kube apiserver expose the https service to have communication from internal components as well as external to manage the cluster.kube-apiserver is server which require certificated to secure all communication with its clients.
* etcd server it stores all information about the cluster,it require pair of server certificates 
* kubelet server also expose the https endpoint that kube-apiserver talk with worker node. it also require pair of server certificates

![server_components](./images/servers_components.png)

Client certificates for clients:
* Admin who need to interact with k8s cluster to manage is client
* Scheduler is also client,it will interact with kube-api server to provide details about node to schedule
* Controll manager is also client,interact with kube-api server
* Kube-proxy is also client,interact with kube-api server
* kube-apiserver will communicate with etcd so kube-apiserver is client for etcd
* kube-apiserver will communicate with kubelet server so kube-apiserver is client for kubelet

![client_components](./images/client_components.png)

**TLS in kubernetes: Certificate Creation**

* Tools available to generate the certificates --> easyrsa, openssl, cfssl etc
* ca.key and ca.crt generation 
  ![ca-cert-creation](./images/ca-cert-creation.png)

clients cert creation
* admin cert creation as client and sign by ca cert
  ![admin-cert-creation](./images/admin-cert-creation.png)
  ![using-admin-cert](./images/using-admin-cert.png)
* In same way of admin cert creation,create cert for kube-scheduler
  ![kube-scheduler-cert](./images/kube-scheduler-cert.png)
* same way for kube-proxy
  ![kube-proxy-cert](./images/kube-proxy-cert.png)
* using admin cert for communication
  ![using-admin-cert](./images/using-admin-cert.png)

server cert creation
* etcd cert configuration
   ![etcd-cert](./images/etcd-cert.png)
* All components in cluster will communicate with kube-api server.  some of them call kube-api server as kubernetes,kubernetes.default,kubernetes.default.svc etc. so we need to create certificate with multiple names
   ![kube-api-cert1](./images/kube-api-cert1.png)
* kube-api server is client for etcd,kubelet and in kube-api.config we have to pass client certs. tls option will accept the server cert for kube-api server
* configuring the certs in kube-api.config
   ![kube-api-cert-config](./images/kube-api-cert-config)
* create server certs for kubelet, as kubelet running on multiple worker nodes,we have use node name for cert generation and use it in kubelet-config.yaml
   ![kubelet-server-cert](images/kubelet-server-cert.png)
* kubelet will communicate with kube-api server, kubelet will require client cert also. for kube-api server to identify and give pemission,we need to mention group while creating
  ![kubelet-client-cert](./images/kubelet-client-cert.png)

**Viewing the certificates in k8s**
* there is issue in cluster,asked to check the certificates,how to check them.
* undertstand how cluster is setup. is it using manual on server level or using kubeadm
* the certificates generation method and path will be stores based on how cluster is setup.
* If you setup manually,you have to generate certificate as how we did it in previous lecture. if you used kubeadm it will takecare on generating certs and updating
* we are going to check certs which is setup by kubeadm. In kubeadm all components are running as pods
* To check and validat certificate,check the respective yaml files.
* check kube-apiserver.yaml for certs used by kube-apiserver
  ![kube-apisever-config](./images/kube-apiserverconfig.png)
  ![cert-details](./images/cert-details.png)
* In same way,we will check for other components
* what and all details need to check
    openssl -x509 -in <path-to-cert> -text -noout
   ![details-to-check](details-to-check.png)
* if you are runnin into issue in cluster and setup from manual,services are availale on system
   $ journalctl -u etcd.service -l
* if you setup using kubeadm,use kubectl command to check logs
   $ kubectl logs  etcd-master
* even if the kube-apiserver not reachable, or etcd not working,we have to go to one step ahead. use container runtime to list containers
   $ crictl ps -a --> listing containers
   $ crictl logs container-name  --> checking the logs of container
* Check the kubernetes-certs-checker.xlsx sheet inside of security folder to know what and all need to check

**Lab Practice Questions**
  $ kubectl --namespace kube-system get po
  $ kubectl --namespace kube-system describe po kube-apiserver-controlplane
* To check which cert is used on component,go to /etc/kubernetes/manifests/ folder,you will get all component yaml file, check for cmd and paths used.
* check the command mention in cmd to identify from which path certs are coming or using. On sever also same path they will used to mount certs.
* Kubectl suddenly stops responding to your commands. Check it out! Someone recently modified the /etc/kubernetes/manifests/etcd.yaml file
  You are asked to investigate and fix the issue. Once you fix the issue wait for sometime for kubectl to respond. Check the logs of the ETCD container.
* check the all cert are available in the mention path -> Ex: ls -la /etc/kubernetes/pki/etcd/server-certificate.crt
* sometimes wrong cert may be using by component


** certificat management using kubernetes api **
* we have to keep ca.crt and ca.key in secure place because any user got that he can sign with any permssion for cert
* new user create key and csr and send to admin
 $ openssl genrsa -out raghu.key 2048
 $ openssl req -new -key raghu.key -subj "/CN=raghu" -out raghu.csr
* create CertificateSigningRequest object,mention any one user csr with base64 encoded. Once object created,it automatically show new cert requests
  ![certsingingrequestobject](images/certsingingrequestobject.png)
* sign the cert requests using kubectl command
  $ kubectl get csr
  $ kubectl certificate approve <name-of-request>
* view certificate and share with user. it will be in base64 format.
  $ kubectl get csr <name-of-request> -o yaml
* All certficate related activities are managed by controller manager using CSR-APPROVING and CSR-SIGNING
* you will get where ca.crt and ca.key is places by checking the kube-controller-manager.yaml

**Kubeconfig**
* kubectl get po --kubeconfig config  --> passing custom config file while dealing with multiple clusters
* config file located in .kube directory of user home directory --> $HOME/.kube/config
* kubeconfig file have three sections clusters,contexts and users
* cluster will have various cluster details
* users are users account which will have access to these cluster
* contexts are used to define which user need to access which cluster
* kubectl will get to know which context to use using current-context set in config file
* kubectl config commands to work with config file
   kubectl config view 
   kubectl config use-context prod-user@production
   kubectl config -h --> show available commands, better to learn other kubectl config commands
* we can pass certificate by mentioning path as well as base64 encoded data 
  Ex: certificate-authority: /path/to/ca.crt
      certificate-authority-data: base64 encoded of cert content

![kubeconfig](images/kubeconfig.png) 

**Kubeconfig lab practice**

**Persistent key/value store**

**API Groups**

**Autherization**

Why autherization:
* As a admin,he can perform all activities in cluster, we can't give same set of permission to developers and service accounts
* To manage permission with developers and service accounts,autherization is required.

Autherization mechanisms:
Node authorizer: Any request coming from username system.node prefix and part of system.node group is autherized by node autherizer. The requests between kube-api server and kubelet is handled by node authorizer,kubelet will be part of system.nodes group

ABAC(Attribute based access control): In ABAC, create policy file for a user or group with set of permissions and pass this file to kube-api server. if any permission updated on policy, you have to restart the kubelet to reflected,so ABAC is difficult to manage. 

RBAC(Role based access control): In RBAC, instead of creatin user with permission, create role with set of permission and attach role to user or group. if we update permision in role,that will be reflected with associated user or group immidiately. this make easy to manage permission

webhook: what if want to handle autherization from external tool, open policy agent is third party tool which helps in admision control and autherization. Kube-api server make request to open policy agent with user details about access requirement,open policy agent decides if user is permited or not,based on response, allow or deny the request.

Always allow: this mode as it name tell always allow the request without performing any authorization

Always deny: this mode as it name tell always deny the request.

Where to we set this authorization modes:
* In kube-api server,we have option called --authorization-mode, we can update modes with comma seperated.
![authorization-mode-settings](images/authorization-mode-settings.png)

Do the authorization happpen when multiple modes are set:
* if authorization success in first mode,it will not go and check in next authorization mode.
* if authorization fails in first mode,it will go to next authorization mode.

**RBAC in details**
* create role by setting apigroup,resource and verb under rules section with kind as role
* we can pass multiple apigroups under rules section
* create rolebinding by mentioning user or service account with role to bind
![role-rolebinding-creation](images/role-rolebinding-creation.png)

$ kubectl get roles
$ kubectl describe role <name-of-role>
$ kubectl get rolebindings
$ kubectl describe rolebindings <name-of-rolebinding>

To check you have access to create particular k8s resource:
$ kubectl auth can-i create deployments --> if you have permission,it will give yes as output otherwise no

To check other user have access to create or check k8s resource
$ kubectl auth can-i create deployments --as dev-user --namespace test --> this will check dev-user have permission to create deployment in test namespace

We can give access to particular resource using Resource Names while rolw creation
![resource-naming-in-role](images/resource-naming-in-role.png)


Namespace Scoped: the resources create and access only from particular namespace. if don't mention namespace while creating,it will create in default namespace
   Ex: pod,deployment,job etc
    To know the namespace scoped resource,run the below command
       $ kubectl api-resources --namespace=true
 
Cluster Scoped: the resources can be created which are nor require the namespace and accessed cluster wide.
    Ex: node,pv, cluster role etc
    To know the cluster scoped resource,run the below command
       $ kubectl api-resources --namespace=false

Cluster Role: cluster role is like role except that are created for cluster scoped resources but we can use cluster role for access resource in any particular namespace, there is no hard rule like use cluster role for cluster scoped resources.
   Ex: cluster admin role can create,view and delete node

Service Accounts:
* service account is identity which is associated with token used to athenticate into the kubernetes api from other applications or services like Jenkins to deploy module or prometheus to collect matrics
* for internal usage of service account,every namespace have default service account and the default service account is automatically attaced to pod on creation. use serviceaccountname key to attach custom service account
* service account is mounted as projected volume, think projected volume as dynamic directory inside pod. it is mounted in /var/run/secrets/kubernetes.io/serviceaccount

Commands used to work with service account
   $ kubectl get serviceaccount
   $ kubectl describe serviceaccount <name-of-serviceaccount>
   $ kubectl create serviceaccount <name-of-serviceaccount>

   whenever pod is created,a default service account automatically attached to pod

**Creation of token**: we can create token and use in external tool like jenkins or monitoring tools
By default all tokens are valid for one hour and created token are not stored in any secret

$ kubectl create token <name-of-token>  --> created token and valid for next one hour
$ kubectl create token <name-of-token> --duration 2h --> created token and valid for next two hour

![token-usage](images/token-usage.png)

**Image Security**:
* if we mention just image name,kubernetes will pulll image from docker registry and use in pod
  Ex: image: nginx 
      kubernets will assume registry as docker.io and user/account as library and pull image from docker.io/library/nginx
* To pull image from private registry,need credential to login into private registry
  creating the secret with type as docker registry and store docker details
  $ kubectl create secret docker-registry regcred \
    --docker-server=private.registry.io \
    --docker-username=registry-user \
    --docker-password=registry-password \
    --docker-email=registry@email.com
* In pod.yaml,we can use below
    imagePullSecrets:
    - name: regcred

**Security in docker** --> this is for only knowledge purpose, not required for exam.

How container process are isolated on host system:
* containers and host share the same kernel. containers are isolated using namespaces in linux.
* all processes run by container are infact run on host system but in their own namespace.
* container can see only the processes of its namespace. 
* if you check inside of container,you can see the processed start with one,two 
  but if we list processed on host system it is just another process

Running process inside container using non-root user:
$ docker run --user=1001 ubuntu sleep 3600

Another way to enforce is using USER instruction on dockerfile.
FROM ubuntu
USER 1001

* the root user inside container is not same as root user in host system,
  docker introduced linux capabilities to handle this(giving permisison to perform root user related activity)
$ docker run --cap-add MAC_ADMIN ubuntu --> adding the linux capability 
$ docker run --cap-drop KILL ubuntu --> removing the linux capability
$ docker run --privileged ubuntu --> giving all linux capabilities

**Security Contexts**
* In kubernetes also,we can set security context at pod level as well as container level. if we give it at both level,
  security context given at container level overwrites
* if we give at pod level,that security context are apply to all container in pod.
* The capablities are applicable only at container level

![sc-at-pod](images/sc-at-pod.png)


![sc-at-container](images/sc-at-container.png)

**Network Policy** --> read the ckad notes

When we are working with large number of contexts and cluster,switching the context and namespace is difficult.

the below tools will help us to make this easy
kubectx --> this tool is particularly useful to switch context between clusters in multi-cluster environment.
  Installation:
     sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx
     sudo ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx

  $ kubectx --> list all contexts
  $ kubectx <context_name>

kubens --> this tool allow users to switch between namespaces quickly with simple command
    Installation:
       sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx
       sudo ln -s /opt/kubectx/kubens /usr/local/bin/kubens
   $ kubens <namespace>  --> switch to namespace
   $ kubens - --> switch back to previous namespace

**Custom Resource Definition(CRD)**
Resource: we can create,update and delete resource using kubectl,it is just a updating the resource details on etcd.
To have pods according to replicas,we need to controller to create,update and delete actual pods.

Controller: controller is a process,runs in background and its job is to monitor status of resource it suppose to manage.
when we change or delete,make nessessary changes in cluster to match state in etcd
  Ex: deployment controller managing the pods of deployment resource

![resource-and-controller](images/resource-and-controller.png)
 
we need to create custom resource and custom controller to have our own resource.
Ex: need resource to create the fight ticket. this resource should actually create fight ticket.

**Creation of flight ticker custom resource:**

```
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: flighttickets.flights.com
spec:
  scope: Namespaced
  group: flights.com
  names:
    kind: FlightTicket
    singular: flightticket
    plural: flighttickets
    shortNames:
      - ft
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                from:
                  type: string
                to:
                  type: string
                number:
                  type: integer
                  minimum: 1
```
$ kubectl create -f flightticket-custom-definition.yml

```
apiVersion: flights.com/v1
kind: FlightTicket
metadata:
  name: my-flight-ticket
spec:
  from: Mumbai
  to: London
  number: 2
```
$ kubectl create -f flightticket.yml
$ kubectl get flightticket
$ kubectl delete -f flightticket.yml

resource just create,update and delete from etcd database

flight controller required to actually create flight ticket.

**Creation of flight ticket custom controller**
* clone the sample-controller repo
* make sure to have go programing longuage installed on system
* go to sample-controller directory
* customize the controller.go with our custom logic
* build the controller
* install controller on cluster to do actual task
* you can package custom controller as image and choose to run as pod

![building-controller](images/building-controller.png)

* In exam, they will not ask to build custom controller as more coding knowledge is required.
* may be ask to build custom resource or work with custom resource definition file

**Operator Framework**
* Operator Framework is used to package custom resource and custom resource controller together to deploy as single entity.
* By deploying the operator,it internally deploy the custom resource and custom resource controller.
* Most popular operator is etcd operator,it deploy and manage etcd cluster and manage it by taking backup,restore
* All operator are available in 

# Section 8: Storage

Storage in docker:
There are two concepts in docker storage
  1) storage drivers
  2) volume drivers

we will talk about volume drivers
When you install docker,it create docker folder under /var/lib --> /var/lib/docker and create multiple folders under docker folder

Layered Architecture
Each line in dockerfile create new layer in docker image with just changes from previous later.
Each layer store only changes from previous layer,it is reflected in size as well
Docker don't build layer if layer already build in other image,it just re-use layer from cache and only create other layers

![layered_architechture](images/layered_architechture.png)

All layers in image are read-only, we can't modify,you have build new image if you want to update

when we create container,docker will create read-write container layer to store data generated by container and container layer exits as long as container exits. if we create another container from same image,container use same layers and create read write container 

![container_layer](images/container_layer.png)

don't we modify file which are in image?
we can modify file but docker will create copy of file on container layer once we modified,this is called copy-on-write mechanism

![copy_on_write](images/copy_on_write.png)

Volumes: 
volumes comes into picture when we want to store changes made on container and persist those changes
when we mount data while creatin container,docker automatically create volume directory under /var/lib/docker/volumes 

docker run -v data_volume:/var/lib/mysql mysql --> this is called volume mount as it mount data under volumes directory

docker run -v /data/mysql:/var/lib/mysql mysql --> this is called bind mount as we can't store on any location 

![volumes_and_binds](images/volumes_and_binds.png)

who will takecare on this layered arch,creating writable layer,moving file across writable layer?
 docker use storage drivers,one of commonly used storage drivers are below

![storage_drivers](images/storage_drivers.png)

volumes are handled by volume drivers plugin.
like creaing volume directory,mounting data from container to volume directory
default volume driver plugin is local,have many other volume drivers as third part like Azure file storage, convoy etc

![volume_drivers](images/volume_drivers.png)

Container Storage Interface:

Earlier kubernetes using container-d only as container run time and part of kubenetes source code.
As other container run times comes like cri-o,rocket it is impatant to expend and work with other container run times and not be depend on kubernetes source code.
Container runtime interface is standard that define how kubernetes communicate with container run time,in future if any new container runtime follow CRI standard,it can work with kubernetes

To extend support in different networking solution,come up with container networking interface
if company can develop their networking plugin based on CNI standard and make their solution work with kubenetes.

Container storage interface developed to support different storage solution,with CSI you can write your own driver to wor with kubernetes

![container_interfaces](images/container_interfaces.png)

Volumes in Kubernetes:
we can use volume section in pod to create volume and use volumeMount to mount data from container.
we can use different types of storage space in volume
hostPath is not recommented to use as cluster in multi node

![volumes_in_kubenetes](images/volumes_in_kubernetes.png)

![volume_types](images/volume_types.png)

![aws_volume_type](images/aws_volume_type.png)

Persistent Volumes:































 









