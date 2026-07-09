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

why we go with PV: --> for my knowledge purpose
Every configuration required to configure storage for volume goes within pod definition file when you have large env,deploying lot of pods, user has to configure storage every time for each pod. every time changes made into storage type,user need to update each pods.

PV is cluster wise pull of storage configured by adminstrator to be used by user by deploying application.
$ kubectl create -f pv.yml 
$ kubectl ge pv

![pv1](images/pv1.png)

Persist Volume Claims:
Administrator create set of pv and user create pvc to claim storage on cluster,once pvc created kubernetes binds the pv into pvc based requests and properties set on pv.
properties like sufficient capacity,access modes,volume modes,storage class 

if multiple pv are match,you stil use labels and selector to bind to right pv.
if no storage available, pvc remains in pendign state,once new pv available claim will be automatically bind into newly available pv.

$ kubectl create -f pvc.yml 
$ kubectl get pvc 

![pvc](images/pvc.png)

$ kubectl delete pvc <pvc-name> 

what happen into underying persistent volume?
persistentVolumeReclaimPolicy: Retail --> keep pv and its data, not available reuse for other pvc
                               Delete --> pv will be delete as soon as pvc delete
                               Recycle(Deprecated) --> data in pv scrubbed and pv is made available for other pvc again
 
![pv_pvc_pod](images/pv_pvc_pod.png)


Storage Classes:

why storage classes? --> my knowledge purpose
when we need to use storage on pv,first we have create storage on particular platform like google cloud or aws,then we can use in pv definition this is called static provisioning. To make use of dynamic provisioning,kubernetes come up with storage classes.


storage class:
In storage class, you can mention provisioner, which can automatically provision storage and attach that to pod when claim is made,this called dynamic provisioning of volumes

we can use storage class name in pvc,when pvc create, request go to storage class,it default  provisionor to provision the storage with required size on storage provider(google cloud,aws etc) and create pv, bind that pvc to pv and no need to create pv manually any more.

![sc_pv_pvc_pod](images/sc_pv_pvc_pod.png)


# Section 9: Networking

create network.md file in linux notes folder and update below notes.
well explained networking,DNS and Network namespaces concepts,which will help in interview

Networking Prerequisites:
* Switching and Routing
  - Switching
  - Routing
  - Default Gateway
* DNS
  - DNS configuration on linux
  - coreDNS introduction
* Network namespaces
* Docker Networking

Switching:
we have system A and system B, do we connect them?
we connect them by switch, switch create network containing two systems.
To connect systems to switch we need interface($ ip link, pyshical or virtual depending on host). we use $ ip link command to see the inteface(eth0 is interface) and that will using to connect to switch.


![switching1](images/switching1.png)

let assume it is network with address 192.168.1.0, we can assign system with same network.
we use command below command to assign ip
  $ ip addr add 192.168.1.10/24 dev eth0.
systems on same network can communicate with each other using switch.
if we are connecting system to multiple networks via interfaces,system will have multiple ip.
eth0,eth1 etc are interfaces which helps to connect to network,ip assgined from network.

![switching2](images/switching2.png)

switch can enable connection within a network. which means it receive package from host and delivery to other host within the same network.


Routing:
how do we connect system on different network?
Router is intelitent device which helps to connect two networks together,since it connect other network it get assigned two ip addresses.
we have router connected to two network, that can enable communication between networks

conneting two network process is called routing

![routing1](images/routing1.png)

When system B want to send a package to system C,how does it know where is router because router is another device on network?
we can configure the systems with gateway.if network is room,gateway is door, systems need to know where to go.

To see the exitsing routing configuration on system, use below command
$ route

configure route on system B to connect to system C through the gateway.
$ ip route add 192.168.2.0/24 via 192.168.1.1 --> reach the 192.168.2.0 network though gateway(192.168.1.1)

$ route --> show the route configured on system 

![routing2](images/routing2.png)

if system D want to send package to system D, need to add route configuration on system D
$ ip route add 192.168.1.0/24 via 192.168.2.1

suppose system need access into internet, connect router to internet and add new route on system
$ ip route add 172.217.194.0/24 via 192.168.2.1

There are so many different sites on internet, instead of adding same route table for each internet,we can simply say 
any network you don't know route,use default route as a default gateway
this way any request exits outsite of your network goes through this default gateway.

$ ip route add default via 192.168.2.1 

![routing3](images/routing3.png)

you can also use 0.0.0.0 instead of defualt
$ ip route add 0.0.0.0 via 192.169.2.1

if your facing issue to reach to internet, checking route table and gateway is best place start debugging.

how we can setup linux host as router?
Connect system B into both network using interfaces
Add route in system A on reaching other network via gateway
  $ ip route add 192.168.2.0 via 192.168.1.6
As system C send back response,add route to reach to network of system A
  $ ip route add 192.168.1.0 via 192.168.2.6
stil we can share packages between system A to system C

![routing4](images/routing4.png)

By default in linux packages are not forwarded from one interface(eth0) into other interface(eth1)
package recived by system B on eth0 not forwarded into system B eth1
for security reason,it is not allowed,if eth0 connected to public network and eth1 connected private network, we don't allow to send any message from public network to private network unless you explicitly allow that.

we can allow by setting 1 on /proc/sys/net/ipv4/ip_forword file. this changes does not persist after reboot,so update 
net.ipv4.ip_forward = 1 to persist changes after reboot also.

$ ip link --> see the interfaces and modify
$ ip addr --> to see ip address assinged those interfaces.
$ ip addr add 192.168.1.10/24 dev eth0 --> set ip address on interface --> changes made using this command only valid still restart,to persist change you must set on hcnetwork interface file
$ ip route or route --> to seet the routing table or routes configured on system 
$ ip route add 192.168.1.0/24 via 192.168.2.1 --> to add route into routing table.
$ cat /proc/sys/net/ipv4/ip_forward --> to check forward between network is enabled or not
 
DNS:

we can use name instead of ip to communicate,to do this we have to update ip address followed by name on /etc/hosts file.
system will check for /etc/hosts file to resolve to ip address.
Even if hostname of system B is different and updated some name(ex:db) in /etc/hosts file, ping db will reach into system B.
Translating hostname into ip address is know as name resolution.

![dns1](images/dns1.png)

DNS server:
when network grow and have larger number of systems, managing them by updating /etc/hosts in each system is very hard.
so we decided to move all these entries into single server to manage centrally,we call that as DNS server.
we can tell systems to lookup DNS server when need to resolve hostname 

![dns2](images/dns2.png)

how to we point our systems to lookup DNS server?
update dns server details on /etc/resolv.conf file like below
 nameserver 192.168.1.100
system will look into dns server from /etc/resolv.conf file and resolve into respective ip address

![dns3](images/dns3.png)

you still can have entry in /etc/hosts file.
for example, you need to provision test server your own need and don't need others to resolve to test server so he can update /etc/hosts file instead of updating in DNS server

![dns4](images/dns4.png)

what is same host name have different ip address in /etc/hosts file and dns server?
system will first lookup into /etc/hosts file and resolve to ip in /etc/hosts file 

![dns5](images/dns5.png)

but the order can be changed on /etc/nsswitch.conf file to refer dns server first then /etc/hosts

![dns6](images/dns6.png)

what if we don't have entry in dsn server for some hostname ex: www.instagram.com?
we can update common well know dsn server 8.8.8.8 hosted by google that knows about all websites on internet.

![dns7](images/dns7.png)

Domain Names:
the reason they are in www. format,seperated by dot is to group like things together. the last partion of domain name , dot coms, dot net,dot edu etc are top level domains and they represents intent of website.

.com for commercial
.net for network related
.edu for educational 
.org for non-profitable organization

look at the google domain name 
see the below image to understand the level 
![domain_names](images/domain_names.png)

subdomain helps in further grouping things together under google 
www.maps.google.com 

when you try to reach any of domain name example www.apps.google.com?
your request first hit your organication DNS server,if it don't know who is www.apps.google.com it forward request into intrenet, on internet ip of serving app.google.com may be resolved by multiple dns servers like root dns server then .com dns server 

![domain_names2](images/domain_names2.png)

in order to speed up this resolution,org DNS server cache this ip for sometime 


we can talk about domain names in our organization?
we have different domain names to address different application so we have stardarized like 
www.web.mycompany.com, www.db.mycompany.com etc
so if we ping web it will not work as it is not complete domain name 

how to configure to resolve to www.web.mycompany.com when i say web?
we can add entry calles serach mycompany.com in /etc/resolv.conf so when you ping web you host is intelligent to exclude the search domain if you specify domain in query, you can also provide additinal search domain like below

![domain_names3](images/domain_names3.png)

how the records are stored in DNS server?
maping hostname with ip address is know A record
maping hostname with ipv6 address is know as quad record
maping one name into another name is called CNAME records

![record_types](images/record_types.png)

ping is not right tool to test resolution, use nslookup to query a hostname from DNS server
nslookup www.google.com but nslookup does not consider entries in /etc/hosts file

dig is another tool to test DNS resolution,it returns more details is stored on server
did www.google.com

how to configure a host as a DNS server?
We are given a server dedicated as the DNS server and a set of IPs to configure as entries in the server.
we will focus on a particular one – CoreDNS.
CoreDNS binaries can be downloaded from their Github releases page or as a docker image
$ curl -LO https://github.com/coredns/coredns/releases/download/v1.12.4/coredns_1.12.4_linux_amd64.tgz
$ tar -zxf coredns_1.12.4_linux_amd64.tgz 
Run the executable to start a DNS server. It, by default, listens on port 53, which is the default port for a DNS server.
we put all of the entries into the DNS servers /etc/hosts file. Then, we configure CoreDNS to use that file. CoreDNS loads its configuration from a file named Corefile.
Here is a simple configuration that instructs CoreDNS to fetch the IP to hostname mappings from the file /etc/hosts. When the DNS server is run, it now picks the IPs and names from the /etc/hosts file on the server.

.:53 {
    # Use /etc/hosts to resolve hostname
    hosts /etc/hosts {
        reload 1m
        fallthrough
    }
 
    # Forward unmatched queries to the host's resolver
    forward . /etc/resolv.conf {
       max_concurrent 1000
    }
    cache 30
    log
    errors
}

CoreDNS also supports other ways of configuring DNS entries through plugins. We will look at the plugin that it uses for Kubernetes in a later


Namespaces: As we know containers are seperated from underying host using namespaces.

what is namespaces?
if your host is house then namespaces are rooms within house that you assign to each of your children. 
room helps in providing privacy to each child and each child can only see within her room.
As a parent you have visibility to all rooms in a house.

when we create container, you wanna make sure container is isolated.it doesn't see any other process on host.
we create seperate isolated space using namespace. Host have visibility to all of process including process running inside container.

![namespaces](images/namespaces.png)

![namespaces2](images/namespaces2.png)

Network namespace:
host have interface(eth0) to connect to network and it have its route table and ARP table with information of all network.

when container created,we create networkspace for it,that way it has no visibility to any network related information on host. within container it have its own virtual interface(veth0),route table and ARP table

![network_namespaces0](images/network_namespaces0.png)

to create network namespace on linux system, run below commands
$ ip netns add red
$ ip netns add blue

$ ip netns --> list network namespaces

To list interfaces inside red or blue network namespace
$ ip netns exec red ip link --> ip link command executed inside red network namespace and show the interfaces
or $ ip -n red link --> will do same

![network_namespaces1](images/network_namespaces1.png)

$ route --> list routes on routing table on host network namespace
$ ip netns exec red route --> list routesm on red network namespace

![network_namespaces2](images/network_namespaces2.png)

let connect two network namespaces:
we have to create virtual cable to connect two network namespaces
creating virtual cables and adding into network namespace 

$ ip link add veth-red type veth peer name veth-blue --> connecting virtual cables first

$ ip link set veth-red netns red --> connecting red virtual cable into red network namespace

$ ip link set veth-blue netns blue --> connecting red virtual cable into blue network namespace

assign the ip address into each network namespace 

up the each vitural interface(veth-red and veth-blue)

ping ip of blue network namespace on red, you will able to connect

![virtual_namespaces_connection](images/virtual_namespaces_connection.png)

![virtual_namespaces_connection2](images/virtual_namespaces_connection2.png)

what if we have many network namespaces and how to you enable to communicate each other?
we can use linux bridge or open vswitch tool to create switch and make connection between multiple network namespaces

create new interface on host for switch and up new interface 

![virtual_interface_on_host](images/virtual_interface_on_host.png)

delete the virual cable created earlier as it is no use 
$ ip -n red link  del veth-red --> as they paired,if we delete one, other one deleted automatically

creating virtual cables to connect network namespace into switch

![virtual_cables_for_switch](images/virtual_cables_for_switch.png)

attach one into network namespace and other end into switch 
assign ip address and up them

![virtual_cables_eshtablishment](images/virtual_cables_eshtablishment.png)

how to establish connection between my host to this network namespaces?
attach the ip address into virtual interface,then able to establish connection between network namespaces and host

![host_to_network_namespaces](images/host_to_network_namespaces.png)

how to reach system on other network from network namespace? from red or blue network namespace into other network outside of host system

our local host system interface is gateway to connect two networks(from private network namespace into outside network)

![network_namespace_to_outside_network](images/network_namespace_to_outside_network.png)

update ip table to get response back from outside network, so outside network will treat request coming from host instead of network namespace 

![ip_table_route](images/ip_table_route.png)

add internet gateway as default route to reach internet

![internet_route](images/internet_route.png)

how about connection from outside network into network namespace on host?
we add update ip table like any traffic coming on port 80 on host forwarded into particular network namespace

![outside_into_host](images/outside_into_host.png)

Docker networking --> i will skip this as of now, not required for cka course

Container Networking Interface --> i will skip this as of now, not required for cka course

Cluster Networking:
Each node have interface(eth0),connected to network,assigned with ip address and mac address.
The ports should be opered on master node and worknode as show in below image

![ports_on_master_and_worker](images/ports_on_master_and_worker.png)

when you have multiple master node, ports need to be opened on master nodes are shown in below image 

![two_master_node_ports](images/two_master_node_ports.png)

![commands_for_networking_setup](images/commands_for_networking_setup.png)

Question about deploying network plugin on cluster:
we have used weave-net as an example, please bear in mind that you can use any of the plugins which are described here:

In the CKA exam, for a question that requires you to deploy a network addon, unless specifically directed, you may use any of the solutions described in the link above.

https://kubernetes.io/docs/concepts/cluster-administration/addons/

https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-networking-model

The links above redirect to third-party/vendor sites or GitHub repositories, which cannot be used in the exam. 

NOTE: In the official exam, all essential CNI deployment details will be provided.

Pod Networking:

how pods communicate with each other?how you access service within cluster or outside of clustr?
kubernetes is not comeup with built-in solution to achieve this. it expect networking solution that solve this challenges.
kubernetes told requirements for pod networking so any networking solution achieve those requirements we can use in cluster
 
![networking_model](images/networking_model.png)

we have many netwoking solution available to achieve this by fulfilling kubenetes requirements
that are flannel,cilium,vmware NSX etc

we can use our network namespace knowledge such switching,routing,assigning ip, establishing connection between network namespaces

![commands_for_networking](images/commands_for_networking.png)

consider all nodes are system and impletement pod networking solution

when containers are created,kubernetes create network namespaces for them so we need to establish connection between pods, pod on one node into pod on another node 

create bridge network on each node and bring them up

![bridge_network](images/brigde_network.png)

![bridge_network2](images/brigde_network2.png)

assign subnet into each bridge network and set ip address for bridge interface 

![ip_address_into_bridge](images/ip_address_into_bridge.png)

connecting pods into bridge and establishing communication between pods

![establishing_communication](images/establishing_communication.png)

adding route to enable communication between pods from one node into another node 

![establishment_on_different_nodes](images/establishment_on_different_nodes.png)

executing the script to make network changes is manually, it will not work when large env and pods are getting created on reach second.
when container created, container network interface(CNI) tell kubernetes to call the script

container runtime is resposible for creating container, when container are create,container runtime will look at CNI and run script with container name and namespace

![cni_script](images/cni_script.png)


Container Networking Interface in Kubernetes:
CNI defines the resposibilities of container run time 
As per CNI, container run time is resposible to 
  create network namespace
  identify and attaching those namespaces into right network by calling right network plugin 
  invoke network plugin(bridge) when container is added
  invoke network plugin(bridge) when container is deleted
  

where do we configure CNI plugin to use?
configure container runtime to use installed CNI plugin in /opt/cni/bin to create network namespace and configure netwrok settings. 

which plugin to use and how do use is configured in /etc/cni/net.d directory,there may be multiple configuration files on this directory to configure each plugin 

![configuring_cni](images/configuring_cni.png)

if you look at the /opt/cni/bin, you can see the all executable files of network plugin 

container run time will look into /etc/cni/net.d directory to know which network plugin to use
if directory have multiple file, choose one based on alphabetical order

![viewing_cni](images/viewing_cni.png)

![viewing_cni2](images/viewing_cni2.png)

Before going to the CNI weave lecture, we have an update for the Weave Net installation link. 
They have announced the end of service for Weave Cloud.

use the below latest link to install the weave net: -
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

CNI weave:
weave CNI deploy agent on each node , they communicate with each other to exchange information about nodes, networks and pods. each agent store the topology of entire network where they know pods and their ips on the other node.
single pod can attached to multiple bridge network,example pod attached to view bridge as well as docker bridge , which one to choose is depends on route configured on container 

when package send to one pod into another pod on another node, weave intercept the package and identify its on seperate network, encapsulate this package with new source and new destionation,sends it across network once on otherside retrieve the package and decapsulate it,route the package into right pod 

package send by the pod  
![package_send_by_pod](images/package_send_by_pod.png)

encapsulate package by weave network plugin
![encapsulate](images/encapsulate.png)

send encapsulated package into other node 
![send_to_other_node](images/send_to_other_node.png)

decapsulate by weave network plugin 
![decapsulate](images/decapsulate.png)

send to destionation pod 
![send_to_dest_pod](images/send_to_dest_pod.png)


deploy weave on kubernets:

![weave_deploy](images/weave_deploy.png)

![weave_deploy2](images/weave_deploy2.png)

IPAM(CNI) --> IP address management:

how the virtual briage get assigned ip and how pods will get assigned ip?
CNI plugin is resposible to assign ip to virtual briade and pods 

how to we manage these ips? we don't to assign duplicate ips?
create file which have list of ips and have script to call and assgine the ip into pod
![list_of_ips](images/list_of_ips.png)

instead of coding our self to achieve this, CNI comes with two built-in plugins 
  DHCP
  host-local
![invoke_built_in_plugin](images/invoke_built_in_plugin.png)

still our resposible to invoke that plugin in our script
get the built-in plugin details from CNI plugin configuration and invoke 
![get_built_in_plugin_details](images/get_built_in_plugin_details.png)

Service Networking:

you rarely configure pods to communicate each other.
when you want to access other pod, you would always use the service.

how the services getting ip address and how it make available across cluster? 
kubeproxy runs on each node,it watches for changes from kube-apiserver. when service get created
unlike container, service are not created on each node, service are cluster wide,there is no server or service
really listening on service. 

how it make available accross cluster?
when we create service,it get assigned ip address from pre-defined ip range.
kubeproxy on each node get that ip address and create forwarding rule on each node.
create forward like any traffic comming into service ip should go into ip of pod.

how are these rules created?
kube-proxy supports different ways userspace,ipvs and default one ip table
proxy mode option can be set by using kube-proxy configuration.

![rule_types](images/rule_types.png)

how the service getting ip address?
when create service,ip get assigned into service and the ip range is set on kube-api server using service-cluste-ip-range option 

![service_ip_range](images/service_ip_range.png)

search for name of service all routes crated by kube-proxy have comment with name of service on it.

![forwarding_rule](images/forwarding_rule.png)

in the kube-proxy.log we can see which type it uses,added entry when service created

![kube_proxy_log](images/kube_proxy_log.png)

DNS in kubernetes:
when kubernetes deployed, by default kube DNS will be deployed.

when service get creates, kube DNS create DNS record with service name to ip address of service.
Now within the cluster any pod can reach to service usign name of service.
As test pod and web pod service in same namespace,test pod can reach web pod using name of service 
![dns_design1](images/dns_design1.png)


if service in different namespace, you have mention name of namespace at end of service name to access service.
for each namespace kube DNS create sub domain with its name , all service pods are group together within subdomain in the name of namespace and all service pods are further group together into another subdomain called svc. finally all pods and services are group together into root domain for cluster which is set to cluster.local by default 

you can access service using http://<name-of-service>.<namespace>.svc.cluster.local, that is the fully qualified domain name for service.

![dns_design2](images/dns_design2.png)

for pods also DNS record created but create with ip address by replacing dot with hypen

![dns_design3](images/dns_design3.png)

core DNS in kubernetes:

how kubernetes implements DNS?

Prior to kubernetes vesion 1.12,the DNS server implemented by kubernetes was know as kube DNS.
from kubernetes version 1.12 version,kubernetes recommend to use core DNS as DNS server in cluster.

Core DNS server deployed as pod in a kube system namespace in kubernetes cluster.
They are deployed as two pods for redundancy as part of replica set.
the core DNS pod execute coredns executable, which require configuration file(/etc/coredns/Corefile).
within this file, number of plugins are configured, plugins are configured to handing error,reporting monitoring
metrics cache etc. core dns works with kubenetes plugin and cluster.local is a top level domain name of the cluster,
so every record in core dns server fall under this domain.
pods option inside kubernetes is responsible for creating record for pods in cluster.

![coredns1](images/coredns1.png)

For example , any pod try to reach to www.google.com,it forwarded to nameserver specifiec on /etc/resolv.conf file.
/etc/resolv.conf file is set to use nameserver from kubernetes node

this configuration passed as configmap object into coredns

![coredns2](images/coredns2.png)

when pod got created,coredsn create record in dns nameserver. when one pod want to talk to another pod by going into dns nameserver.

when we deploy DNS solution, it also create a service and make it available to other components on cluster.
the ip of service mentioned on /etc/resolv.conf as nameserver. Adding ip of coredns service into pod done by kubelet automatically when the pods are created.

![coredns3](images/coredns3.png)

![coredns4](images/coredns4.png)


Ingress: you already know what is ingress and why we use the ingress,same explained in video.

ingress helps your users to access your application using single externally accessible URL.
configure to route traffic to different service based on url path and also implement SSL solution

ingress deployment:
 deploy ingress controller provided third party like nginx ingress controller,HAproxy traefik etc
 once ingress controller deployed, then only we can create ingress and use it.
 By default no ingress controller deployed on cluster.

![ingress_deployment](images/ingress_deployment.png)

![nginx_ingress_controller](images/nginx_ingress_controller.png)

![nginx_ingress_service](images/nginx_ingress_service.png)

![nginx_ingress_sa](images/nginx_ingress_sa.png)

![ingress_resource1](images/ingress_resource1.png)

![ingress_resource2](images/ingress_resource2.png)

Rewrite option in ingress:
The target applications are not configured with /watch or /wear paths. They are different applications built specifically for their purpose, so they don't expect /watch or /wear in the URLs. And as such the requests would fail and throw a 404 not found error.

To fix that we want to "ReWrite" the URL when the request is passed on to the watch or wear applications. We don't want to pass in the same path that user typed in.

For example: replace(path, rewrite-target)

In our case: replace("/path","/")

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - http:
        paths:
          - path: /pay
            pathType: Prefix
            backend:
              service:
                name: pay-service
                port:
                  number: 8282

In another example given here, this could also be:
replace("/something(/|$)(.*)", "/$2")


Gateway API:

ingress limitation:
earlier when we talk about ingress,single ingress handing two service to distribute based on path or host.

what if each service are completely manage by different team or even different company?
* the underlying ingress resource is single and only one team managed by at a time.
  ingress face challenge in multi-tenancy enviroment like they have to co-ordinate to handle single ingress resource and might lead to conflics 

![ingress_limitation1](images/ingress_limitation1.png)

* ingress only support http based rule such host matching, path matching, other like TCP , UDP based rules are not supported
 
![ingress_limitation2](images/ingress_limitation2.png)

Gateway:
Infrastructure provider provides the gateway class which defines what is underlying network such as nginx,traffic or other local load balancers. 
Cluster operator install the gateway(gateway require gateway controller) then we have http route,tcp route and GRPC route created by application developer.

![gateway_api1](images/gateway_api1.png)

![gateway_api2](images/gateway_api2.png)

![routes_on_gateway_api](images/routes_on_gateway_api.png)

we can talk about below points when someone asked about 
  difference between ingress and gateway api? or
  advantages of gateway api? or 
  disadvantages of ingress? or 
  why we move into gateway api?

In gateway api, we no need to mention annotations to apply ssl certs, all configuration will go under spec.
unlike in ingress, we have use annotation specific to ingress provider like nginx ingress have different annotation which will not work for other ingress controller.

![gateway_api3](images/gateway_api3.png)

In gateway api, we can define how much % of traffic goes to which rules under spec, no extra annotations required.

![gateway_api4](images/gateway_api4.png)

In gateway api,we can mention rewrite rule configuration centrally under responseHeaderModifier.

![gateway_api5](images/gateway_api5.png)

Different companies are implementing the gateway controller and below is status

![gateway_implementation_status](images/gateway_implementation_status.png)

Section 10:
Design and Install a Kubernetes Cluster

Designing a Kubernetes Cluster:

Before implement kubernete cluster,we must ask below question first

![k8s_design1](images/k8s_desgin1.png)

![k8s_design2](images/k8s_desgin2.png)

![k8s_design3](images/k8s_desgin3.png)

![k8s_design4](images/k8s_desgin4.png)

![k8s_design5](images/k8s_desgin5.png)

![k8s_design6](images/k8s_desgin6.png)

In the exam perspective, no need to remember these numbers.

Choosing infrastructure:

we have two types of soluation for setting up cluster

Turnkey solution --> you will provision vms,configure vms,use your script to deploy cluster and you maintain vms yourself.
Eg: kubenetes on aws using KOPS

hosted solutions(managed solutions):
use kubernetes as server from provider like aws, azure,gcp
Provider provisions vms
Provider install kubernetes
Provider maintains VMs
Eg: Google container engine

Turnkey solutions:
Openshift: openshift is a popular on prem kubenetes  platform by Redhat.
openshift is open source container platfrom and it built on top of kubernetes.
It providers a set of additional tools and nice GUI to create and manage kubernetes constructs and easily integrate with ci/cd pipelines.

Cloud Fondry container runtime: it is open source project from cloud fondry that helps in deploying and managing highly available  kubernetes cluster using their open source tool called bosh.

VMware cloud PKS is another solution

vagrant provides a different scripts to deploy kubernetes on different cloud service providers.

All of these solution helps to deploy and manage kubernetes privately within your organition 

Hosted solutions:
Google kubernetes engine: is very popular kubernetes as a service offering on google cloud platform

openshift online: is offering from redhat where you gain access to a fully functional kubernetes.

Azure kubernetes service:  it is from azure for kubernetes

Amazon elastic kubernetes service: kubernetes solution from aws

Configuring High Availability:
Configure two master nodes on production to have high availability.

when communicate with kubeapi server using kubctl utility,to which master node request will go?
we need to configure load balancer infront of master nodes, lb will forward request into any one of master node

when we have controller and scheduler on both master nodes, which controller or scheduler will be used to schedule or create pod?
master nodes run by active and standby mode.

who decide which one is active and standby?
this is decide by leader election process.

when controller manager configured, we set the leader elect option to true,with when controller manager process start,it try to gain the lock on endpoint object on kubernetes named as kube-controller-manager endpoint.
whichever controller process first update the endpoint with its information,gain the lock and become active and other one is standy. it holds the lock for the lease duration specified using lease duration option,which by default set to 15 seconds.the active process then renew the lease every 10s which is default value for the option.
Both the processes try to become the leader every two seconds set by leader elect retry period option.

![ha_configuration1](images/ha_configuration1.png)

![ha_configuration2](images/ha_configuration2.png)

similar setup for scheduler also.

ETCD:
for etcd, there are two topologies that can be configured in kubernetes.

etcd is part of master node and this topology called as stacked topology.

![ha_configuration3](images/ha_configuration3.png)


etcd is seperated from master node, deploy on external server and update etcd endpoints on kubeapi to communicate.
this is topology called external etcd topology.

![ha_configuration4](images/ha_configuration4.png)

![ha_configuration5](images/ha_configuration5.png)


ETCD in HA:

you have three servers,etcd running on all three,how does etcd ensure data on all nodes are consistent?
with read, it is easy as same data available on all nodes.

what if two write requests coming on two different instances?
only one of the instance is resposible to write data and send the update data into other two servers.
Internally two nodes elect the leader among them,on total instances one node become leader and other nodes becomes followers. if write come into leader node,leader node processes data and send a copy of updated data into other nodes.
if write come into non-leader node,non-leader node will redirect request into leader node and leader node repeate same.

![etcd_ha1](images/etcd_ha1.png)

how do they elect the leader?
ETCD implement distributed consensus using RAFT protocol,
RAFT algorithm uses random timer for initiating requests.
random timer kicked off into all three node,the first one finish the timer sends out a request to other nodes requesting permission to be the leader,the other managers on receiving the request responds with their vote and node assumes the leader.
leader node sends out notification at regular interval to other master nodes informing them that is continuing to assume the role of leader. In case other nodes don't recieve notification from leader at some point in time,which could either be due to leader going down or network connectivity,the nodes initiate a relection process among themselves and new leader identifed 

if leader not able to write into any one of node, is leader node mask write as completed?
Yes leader mask as write completed as data write into majority nodes.
In case of three nodes, majority is two so if data can be written on two nodes then the write is considered as completed.
when third node come online,data to be copied into that node as well.

what is majority when we have more nodes?
kubernetes use quorum to calculate majority
quorum=N/2+1 
quorum is the minimum number of nodes that must be available for cluster to fuction propertly or make successfull write.

3 nodes
3/2+1 --> 2.5 so consider whole number that is 2

![quorum](images/quorum.png)

having two instance does not mean it doesn't offer any real value as quorum can't be met,which is why it is recommended to have a minimum of three instances in an etcd cluster. that way it offers a fault tolerance at least one node goes down.

fult tolerance column give number of nodes that you can offer to lose while keeping the cluster alive.

![quorum2](images/quorum2.png)

when deciding on the number of master nodes, it is recommended to select an odd number as highlighted in table like 3 5 7

if we select even number of nodes and make split to configure equal number of nodes on different network segment,chances of failing is more as per quorum calcutation

![quorum3](images/quorum3.png)

odd number of nodes will give good fault tolarence nodes

![quorum4](images/quorum4.png)


it is required to configure initial cluste peer to know it is part of cluster and where its peers are there

![etcd_conf](images/etcd_conf.png)

use etcdctl to write and retrieve data
 etcdctl put name john --> writing data
 etcdctl get name --> getting value of name key
 etcdctl get / --prefix --key-only --> getting all keys on etcd.

# Section 11: Install kubernetes the kubeadm way

Deployment with kubeadm --> introduction
steps:
* creation of vms,choose one as master and remaining as worker nodes.
* install containerd on all nodes
* install kubeadm tool on all nodes --> kubeadm tool helps us to bootstrap the kubernetes solution by installing and configuring all required components on right node and right order.
* initialize master node,once the master node initialized and before joining the worker nodes to mastert, you must ensure pod network are meet.
* join the worker nodes into master node

![kubeadm_steps](images/kubeadm_steps.png)

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

Deployment with kubeadm -- Provision VMs with vagrant
they have vagrant file and use vagrant command to provision the vms

* install kubeadm using below document,also install kubelet,kubectl on all nodes
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
* install containerd on all nodes
https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd
https://github.com/containerd/containerd/blob/main/docs/getting-started.md 
if we are using apt to install containerd then we have to follow below link and metion only containerd.io package name
  https://docs.docker.com/engine/install/debian/
* using driver ---> cgroup driver or systemd driver, any one we can use but ensure both kubelet and containerd must use same driver
* check your init system on your linux distribution 
  ps -p 1 --> if it is systemd , then we are going to use systemd
* from kubernetes 1.21 onwards, by default kubelet will use systemd as driver, we have set only for containerd 
* use below document to make containerd to use systemd as driver(on all nodes)
  create /etc/containerd directory
  generate default configuration by using below command
   $ containerd config default and make required changes using below link
  https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd-systemd
  $ cat /etc/containerd/config.toml | grep -i SystemCgroup -B 50 --> checking configuration after update
* restart containerd
  $ systemctl restart containerd
* initializing controll plan --> it install all control plan components
  https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#initializing-your-control-plane-node 
  $ sudo kubeadm init --apiserver-advertise-address <master-node-ip> --pod-network-cidr "10.244.0.0/16" --upload-certs

  the ip to pods are assigned from 10.244.0.0 cidr block, we can set to other cidr block also,this one is default
  this will go and generate the certs for kubeapi server,etcd etc
  at the end of output, tell to copy admin.config file to communicate with cluster
* Deploy pod network to the cluster to make master node ready,then you can join any number of worker nodes using kubadm join command showed at end of console output
* install pod network add-on 
  https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network
  use any one of pod network add-on from support network solution
  https://kubernetes.io/docs/concepts/cluster-administration/addons/#networking-and-network-policy
  we are going to use Flannel and click on link to take into documentation.
  https://github.com/flannel-io/flannel#deploying-flannel-manually
  update your pod network cidr in net-conf.json section to make sure to use same pod network cidr 
  $ kubectl apply -f kube-flannel.yml
  $ kubectl get pods -n kube-flannel 
  $ kubectl get nodes --> now master node must be come up to ready state
* use the kubeadm join command given from master node console and join the worker nodes.
  $ kubectl get nodes
  deploy any pod to test to check our cluster is ready
  $ kubectl run nginx --image=nginx
  $ kubectl get po 

# Section 14: Troubleshooting --> this section helps to clear interview
 it covers 
  Application failure 
  Control plan failure
  Worker node failure
  Networking

Application failure 
 * Understanding the arch is important, then we can debug how traffic is going on and how many compontents on our application
* ex: user reported not able to access application.
  application have web server and db
 
  check the applicaiton runnning on service port 
   curl http://web-service-ip:node-port

  if not able to connect or get response,check the endpoint configured on service,
  check is application running on endpoint.

  compare the selectors configured on service

  check the status of pods, logs, events
  $ kubectl get pod
  $ kubectl descrip po <name-of-pod> --> checking events of pod
  $ kubectl logs <name-of-pod>
 
  check the db service like how we checked for web service, selectors are matching, enpoint are correct
  
  check the status,logs and events of db pod
  
  https://kubernetes.io/docs/tasks/debug/debug-application/  --> this have links to other debug documents 

* Control plan failure
  
  check the pods of control plan on kube-system namespace

  $ kubectl get pods -n kube-system

  if control plan components are deployed on system level, check the status of services
  $ service kube-apiserver status

  $ service kube-control-manager status
 
  $ service kube-scheduler status

  $ service kubelet status
  
  $ service kube-proxy status

  $ kubectl logs kube-apiserver-master -n kube-system

  same way check the logs for other control plan components 

  if case of service configured on node
  $ sudo journalctl -u  kube-apiserver --> checking the logs of kube-apiserver 

  $ kubectl get nodes

  $ kubectl cluster-info dump --> to get detailed information about the overall health of cluster

  $ kubectl describe node <name-of-node>

  $ kubectl get node <name-of-node> -o yaml

 https://kubernetes.io/docs/tasks/debug/debug-cluster/ 

* worker node failure

  $ kubectl get nodes
  $ kubectl describe node <name-of-node>
  
  each node have conditions section, it show why my node is failed, check for which it become true 
  if node is running out of disk space, OutOfDisk flag set to true
  if node is out of memory, MemoryPressure flag set to true
  if disk capacity is low, DiskPressure flag set to true
  
  when worker node stop communicate with master node,may be due to status of one these flags set unknown state

  checking status of node itself helps 
   $ top 

  check disk space 
   $ df -h 

  check the status of kubelet 
  $ service kubelet status 

  check the kubelet logs
  $ sudo journalctl -u kubelet --> checking logs of kubelet service 

  check the kubelet certificates 
  $ openssl x509 -in /var/lib/kubelet/worker-1.crt -text 
  check certs are issued by right CA,certs are not expired,check the subject for organization(O) 

Network troubleshooting:

  first we have pods where you application is running
  second service --> provide stable way to access group of pods
  coreDNS --> translate service name into actual ip address of service
  CNI(container network interface) --> assigning ip address and configuring networking interfaces
  kube-proxy --> runs on every node and it is resposible for keeping network rules updated
  

  when your application is not reachable, service is first place to look into it.
  Step 1:
  check the pods are up and running because service will forward request only running pods
  $ kubectl get po -l app=hostnames --> checking the pods status which have app:hostnames label

  if your seeing issue in number of restart or showing as pending or failure, use below command to debug more

  $ kubectl logs <name-of-pod> -n <namespace>
  
  $ kubectl describe pod <name-of-pod>

  Step 2: try reaching pods directly by taking ip 
  $ kubectl get po -l apps=hostnames -o wide --> you will get ip address
  
  create temp pod and try to reach them 
  $ kubectl run -it --rm --restart=Never busybox --image=busybox -- sh
    for ip in ip1 ip2 ip3;do
       wget -qO $ip:port;done

 Step 3: check the serice assosicate with that application 
 selector must be match with labels of pods to forward the request
 check port of pod and port configured on service

 $ kubectl get service <name-of-service> -o yaml --> give attention to selector and compare with lables of pod

 check the port details configured on service and check pod running application on same port 

 check the endpoint slices associated with your service
 $ kubectl get endpointslices -l kubernet.io/service-name=<name-of-servie> -n <name-of-namespace>

 Step 4: troubleshoot the coreDNS problems 
 if dns are not working your application won't be able to each other by names

 coreDNS runs as pod in kube-system namespace.
 kube-dns service exposes these pods on port 53. pods-resolve.conf point into kube-dns service ip
 
 coredsn file is supper impart which come from configmap,it defines how exactly coreDNS should behave

 ![coredns_problems](images/coredns_problems.png)

  /etc/resolv.conf in coredsn config file connect the external dns server if it not able to find records on itself

  $ kubectl get pods -n kube-system -l k8s-app=kube-dns --> checking status of pod, even if you don't known the label also is fine, we will get list of pods running in kube-system namespace

  check the endpoints on kube-dsn service
  $ kubectl get endpointslices -l k8s.io/service-name=kube-dns -n kube-system 
  if don't see the endpoint then this service is not finding coreDNS pod

  check the resolv.conf file in your application pod as dns server(ip of kube-dsn service are set) and search paths also impartant
  $ kubect exec -it <pod-name> -- cat /etc/resolv.conf
  
  test dns resolution directly from the pod 
  $ kubectl exec -it busybox -- nslookup kubernetes.default.svc.cluster.local

  also test actual application service dns 
  $ kubectl exec -it busybox  -- nslookup my-app-service.default.svc.cluster.local

  CNI plugin is very impartant for networking, without these your pods will not get the ips
  Flannel is very popular and simply to install 
  $ kubectl apply -f <path or url of defition file>

  calico is more advanced option and widely recognized for its robust network policy features and strong performance 
  download yml and apply
  kubectl apply  -f calico.yml 

 check the network plugin Daemonset pods are running in their respective namespace
  $ kubectl get pod -n kube-flannel 
 
  $ kubectl logs <pod-name> -n kube-flannel 

  $ kubectl describe pod <pod-name> -n kube-flannel


  step 5: check about kube-proxy

  check the status of kube-proxy pod
  $ kubectl get pods -n kube-system
  
  $ kubectl logs <pod-name> -n kube-system 
 
  check the config map which is used by kube-proxy
  $ kubectl get cm kube-proxy -n kube-system -o yaml
  
 check the iptable rules
  if your using ipvs mode(configured on config map),use ipvsadm -ln 
  $ ipvsadm -ln --> shows how the service load balacing across the pods 

 ![trobleshootin_points](images/trobleshooting_points.png)

JSON PATH in kubernetes:

we use kubectl utilty to read and write object into cluster.
 when we are reading object details, kubeclt interact with kube-apiserver, kube-apiserver send response in json format,kubectl show only impartant details on readable format

$ kubectl get node  --> give high level status of node
$ kubectl get node -o wide --> some more information 

using jsonpath in kubernets
* identify the kubectl command like kubectl get nodes or kubectl get pods
* familier with json output
  $ kubectl get pods <name-of-pod> -o json
* form the json path query,getting image details of pod
   .items[0].spec.containers[0].image 
* use the jsonpath query with kubectl command
  $ kubectl get pods -o=jsonpath='{.items[0].spec.containers[0].image}'

 ![jsonpath_in_kubectl](images/jsonpath_in_kubectl.png)

we can add multiple json queries into single command and get output
 $ kubectl get nodes -o=jsonpath='{.items[*].metadata.name} {.items[*].status.capacity.cpu}'

  getting the name of node and cpu capacity details at a time
  
 $ kubectl get nodes -o=jsonpath='{.items[*].metadata.name} {"\n"} {.items[*].status.capacity.cpu}' --> to print in new line

we can use custom column option to print details with our own column name
 $ kubectl get nodes -o=custom-columns=NODE:.metadata.name,CPU:.status.capacity.cpu  --> no need to mention the items,by default custom-columns will go through the each items

we can also sort the output based on josn query
$ kubectl get nodes --sort-by=.metadata.name 
$ kubectl get nodes --sort-by=.status.capacity.cpu 


 
























