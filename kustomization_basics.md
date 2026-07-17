Kustomize basics:

why we need kustomize tool?
we can't deploy the same configuration into different environment like dev, stage, prod.
Ex: for dev we require 3 replicas,6 replicas for stage and 10 replicas for prod.
so we have to keep different 

kustomization.yaml:
kustomize will look into kustomization file to apply custome changes before deploying.
kustomization.yaml will have details about resources yaml and custmatization need to be done

Ex:
  k8s
   nginx-deploy.yaml
   service-deploy.yaml
   kustomization.yaml
 
kustomization.yaml file content:
 resources:
   nginx-deploy.yaml
   service-deploy.yaml
   

 commonLabels:
   company: kodekloud

![kustomize3](./images/kustomize3.png)  --> image

![kustomize4](./images/kustomize4.png)  --> image

![kustomize5](./images/kustomize5.png)  --> image


Apply the changes:

$ kustomize build k8s/ | kubectl apply -f -       --> redirecting the output of kustomize build command

or 

$ kubectl apply -k k8s/

Deleting:

$ kustomize build k8s/ | kubectl delete -f -   --> deleting the resources 

or
 
$ kubectl delete -k k8s/ 


Kustomization apiVersion and kind:
we can set the apiversion and kind into our resources by defining it in kustomization.yaml file

  apiVersion: kustomize.config.k8s.io/v1beta1
  kind: Kustomization

Managing the directories:

As we grow, we will have multiple types of application components like api, db , backend so we can't keep each file in common directory,so we can go with folder structure

k8s/
  kustomization.yaml
  api/
    api-deploy.yaml
    api-service.yaml
  db/
    db-deploy.yaml
    db-service.yaml

we have update all resouces yaml file path in kustomization.yaml like below
kustomization.yaml
  resources:
    api/api-deploy.yaml
    api/api-service.yaml
    db/db-deploy.yaml
    db/db-service.yaml


as multiple yaml files grow in each component, it is difficult to add each yaml files into resouces section
so create kustomization.yaml in each component folder and mention the component folder in kustomization.yaml in parent folder(k8s)

![kustomize5](./images/kustomize5.png)  --> image

$ kustomize build k8s/ | kubectl apply -f -   --> will work as same

$ kustomize build k8s/ | kubectl delete -f -   --> will work as same

Common Transformers:
* common transformers is applying common changes into all resouces yaml files like adding common labels, updating suffix name into resource name etc
* some of common transfermers are below

![common_transformers1](./images/common_transformers1.png)  --> image

![common_transformers2](./images/common_transformers2.png)  --> image

![common_transformers3](./images/common_transformers3.png)  --> image

![common_transformers4](./images/common_transformers4.png)  --> image


Image Transformers:
change the image before deploying resources

![image_transformers1](./images/image_transformers1.png)  --> image

![image_transformers2](./images/image_transformers2.png)  --> image

![image_transformers3](./images/image_transformers3.png)  --> image

Patches:

![patches1](./images/patches1.png)  --> image

![patches2](./images/patches2.png)  --> image

![patches3](./images/patches3.png)  --> image

Different ways of doing patches:

![patches4](./images/patches4.png)  --> image

Different types of patches:
inline vs seperate file

i will go with seperate file type,it look easier to me

![patches5](./images/patches5.png)  --> image


Patches dictionary:

![patches_dictionary1.png](./images/patches_dictionary1.png)  --> image

![patches_dictionary2.png](./images/patches_dictionary2.png)  --> image

![patches_dictionary3.png](./images/patches_dictionary3.png)  --> image


Patches list:

how to update when we have list on particular section
Ex: containers section can have multiple containers, how to update image into particular container

updating:

![patches_list1.png](./images/patches_list1.png)  --> image

adding:

![patches_list2.png](./images/patches_list2.png)  --> image

removing:

![patches_list3.png](./images/patches_list3.png)  --> image

Overlays:

![overlay1](./images/overlay1.png)  --> image

![overlay2](./images/overlay2.png)  --> image

we can also include the yaml which is specific that environment in env directory.
Ex: having prometheus yaml file which will be used only for prod env

![overlay3](./images/overlay3.png)  --> image

![overlay4](./images/overlay4.png)  --> image

Components:

componenst is reusable block of kubernertes configuration.
it is usefull when your application supports multiple feature and need to enable different features based on env.

Ex: you application support caching and external db feature but you need to enable based on env
envs are dev, premium and self hosted
   caching need to be enabled for premium and self hosted
   exteranl db need to be enable for dev and premium

![components1](./images/componets1.png)  --> image

so we can't keep it in base folder as based on env we need to enable featur and we can't keep in required multiple env as we need to do changes on both yaml.
components comes into picture

![components2](./images/componets2.png)  --> image

![components3](./images/componets3.png)  --> image

![components4](./images/componets4.png)  --> image

![components5](./images/componets5.png)  --> image








