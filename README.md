# Kubernetes
Installations Guide

Kubernetes Architecture & Commands

In Kubernetes we have a Master controller that controls the Nodes. and each Nodes contains a Docker Engine & Pods. And If we go into deep that each Pod contains Number of containers.

Pods: It contains a Unique IP Address. So that the traffic can distribute in the containers. Can also share the volumes, disk utilizations, Finally the pod management is done through API or delegated through controller.
Labels: Clients can manage "Key value pairs". Used for Identification of configuration and management system.
Selectors: Represent quries that made againest labels. They resolve to the corresponding macthing objects. The labels and selectors are the primary way where grouping is done in kubernetes.
Controller: Used to manage Pods within the cluster and container within the pods with the help of Replication controler. With the help of this replication controller if any container fails in the pod, it automatically replaces the new controller. We do have a Job controler that controls pods to completion of the batch Jobs. Demaon set controler that enforces an 1 to 1 ratio of Pods to Nodes. Also manage the cluster. Each set of pods that any controller manages, is determined by the label selectors.
Service: In kubernetes each service can handle a routing with the static IP for each pod as well as load balancing(Round robin based) connection to that service among the pods that match the label selector indicated. By default, the service is only Exposed inside a cluster, it is also exposed outside the cluster as needed.
Introduction to YAML:

YAML(Yet another Markup language) (Human readable data serialization format).
Based on Idententation, also compiling & running complex applications.
Kubernetes setup and Configurations:

Note: while we are configuring with centos-master or centos-node if it is not properly configuring in the place of centos-master usse the IP Address of the centos-master & In the place of centos-nodes use the IP Address what we have given as per the requirement.

Packages and Dependencies:

First thing we have to download ntp service in all of our Master and Minions(Nodes).

  yum install -y ntp
We have to enable & start the ntpd services.

  systemctl enable ntpd && systemctl start ntpd && systemctl status ntpd
vim /etc/hosts

# Paste the IP address & name of the Nodes in order to communicate.
 
 173.31.24.15 centos-master1
 173.31.25.26 centos-Node1
 173.31.26.58 centos-Node2
 173.31.28.65 centos-Node3
We can also verify it is configured correctly or not by using the below commands:

  ping centos-Node1 in Master 
  ping centos-master1 in the node
vim /etc/yum.repos.d/virt7-docker-common-release.repo

  # Paste the below content in both Minions(Nodes) and Master for the etcd to communicate
  
  [virt7-docker-common-release]
  name=virt7-docker-common-release
  baseurl=http://cbs.centos.org/repos/virt7-docker-common-release/x86_64/os/
  gpgcheck=0
Update the Environment in both Master and Nodes.

  yum update 
We have to enable repo and install the etcd for kuberntes & also we have to install docker also by using the command:

  yum install --enablerepo=virt7-docker-common-release etcd kubernetes docker
Note: Everything relates to packages and depencies has should be done in both Master and Nodes

Install and configure Master controller: we has to be in the Root user.

Go the configuration file by using the below commands:

cd /etc/kubernetes
ll
we can see the configuration file here change and add the configuration file in the controller manager, schedular & proxy that kube master has to run in the centos-master node in 8080 and also add the following etcd_servers along kubemasters is shown below:

vim config

      # How the control manager, schedular & proxy and api server.
      
     KUBE_MASTER="--master=http://centos-master:8080"
     
     KUBE_ETCD_SERVERS="--etcd-servers=http://centos-master:2379"
Now go to the etcd configuration file:

cd /etc/etcd
ll
Change and edit the etcd configuration file:

vim etcd.conf

     # In the Member and cluster sections edit the etcd clients url and etcd advertise client urls from local host to 0.0.0.0
     
     ETCD_LISTEN_CLIENTS_URLS="http://0.0.0.0:2379"
     
     ETCD_ADVERTISE_CLIENTS_URLS="http://0.0.0.0:2379"
Go back to the kubernetes directory and edit the api-server:

     cd /etc/kubernetes/
     ll
Edit the api-server:

     vim apiserver
Edit and change as fallows i.e it has to be like below for the fallowing and the remaining all are same:

     # The address of the local server listen to
     KUBE_API_ADDRESS="--address=0.0.0.0"
     
     # The port on the local server to listen on.
     KUBE_API_PORT="--port=8080"
     
     # Port minions listen to
     KUBELET_PORT="--kubelet-port=10250"
     
     # Comma separated list of nodes in the etcd cluster
     KUBE_ETCD_SERVICES="--etcd-services=http://127.0.0.1:2379"
     
     # Address range to use for services
     KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"
     
     # default admission control polices
     # KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle.NamespaceExists.LimitRanger.SecurityContext.ServiceAccount.Restore.ResourceQuota"
Make sure after changing and saving all this we have to enable the four services as fallows:

     systemctl enable etcd kube-apiserver kube-controller-manager kube-scheduler
     
     systemctl start etcd kube-apiserver kube-controller-manager kube-scheduler
     
     systemctl status etcd kube-apiserver kube-controller-manager kube-scheduler | grep "(running)" | wc -l
Note: The above configuration as to be done only in the Master only.

Install and Configure Nodes(Minions) :

We has to do This configuration only in the Node1, Node2, Node3 ..n all the Nodes only depending on how many nodes you are taking.

Go the configuration file by using the below commands:

  cd /etc/kubernetes
  ll
we can see the configuration file here change and add the configuration file in the controller manager, schedular & proxy that kube master has to run in the centos-master node in 8080 and also add the following etcd_servers along kubemasters is shown below:

  vim config
  
        # How the control manager, schedular & proxy and api server.
        
       KUBE_MASTER="--master=http://centos-master:8080"
       
       KUBE_ETCD_SERVERS="--etcd-servers=http://centos-master:2379"
Go back to the kubernetes directory and edit the kubelet:

        cd /etc/kubernetes/
        ll
Edit the kubelet:

        vim kubelet
See the kubelet configuration file has to be as fallows in some places & remaining all are same:

        # The address for the info server to serve on (set to 0.0.0.0 or "" for all interfaces)
        KUBELET_ADDRESS="--address=0.0.0.0"
        
        # The port for the info server to serve on
        KUBELET_PORT="--port=10250"
        
        # You may leave this blank to use the actual hostname
        KUBELET_HOSTNAME="--hostname-override=centos-minion1"
        
        # location of the api-server
        KUBELET_API_SERVER="--api-server=http://centos-master:8080"
        
        # pod infrastructure container
        # KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=re.....................everything are same in the configfile file
Make sure after changing and saving all this we have to enable the three services as fallows:

        systemctl enable etcd kube-proxy kubelet docker
        
        systemctl start etcd kube-proxy kubelet docker
        
        systemctl status etcd kube-proxy kubelet docker | grep "(running)" | wc -l
we have to make sure that docker is running currently are not by using the below commands and also check by pulling the hello-world image.

        docker images
        docker version
        docker pull hello-world
        docker images
        docker run hello-world
        docker ps
        docker ps -a
4 . Kubectl: Exploring our Environment ( Main Untility):

Basically kubectl controlles & manages the cluster manager we can also see the kubectl man page by using below command:

        man kubectl
we can also check that what nodes are registerd & also can see the man page of kubectl-get by using the below command:

        kubectl get nodes
        
        man kubectl-get
If we want to describe the nodes & also describes the nodes in the JSON format & also says that they are ready=True:

        kubectl describe nodes
        
        kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address)'
        
        kubectl get nodes -o jsonpath='{range .items[*]}{@.metadata.name}:{range @.status.conditions[*]}{@.type}={@.status};{end}{end}'| tr ';' "\n"  | grep "Ready=True"
Finally we know that an Nodes consists of pods or services which they don't have Now, So If we use below command It shows Nothing:

        kubectl get pods
Creating Pods:

Note: Use only one centos-master & one centos-node1 and stop centos-node2 & centos-node3.

              systemctl stop kubelet kube-proxy ( in both centos-node2 & centos-node3) 
Below what we do exactly is that we create a YAML file and name it as nginx.yaml in a Directory called Build.In order to create a Pod contains an nginx container. So after creating Pod this can be deployed in one Node, So that we can see the same nginx container.

In this senario we use one centos-Master and one centos-Node. Below is the commands to start with:

        mkdir Builds
        ll
        cd Builds
        
        vim nginx.yaml              
Below is the YAML file that we has to paste the below content

        apiVersion: V1
        kind: Pod
        metadata:
          name: nginx
        spec:
          containers: 
          - name: nginx
            image: nginx:1.7.9
            ports:
            - containerPort: 80
In order to create the Pod Just Run the below command: This creation will create the pod that contains an nginx container in both Master and one Node we are using.

        kubectl create -f ./nginx.yaml 
Note: If we get any Error in creating a POD of nginx.yaml, then add a new key as fallows

Error from server (ServerTimeout): error when creating "./nginx.yaml": No API token found for service account "default", retry after the token is automatically created and added to the service account.

Resolution:

Generate RSA key
[root@kamb1021 Builds]# openssl genrsa -out /etc/kubernetes/serviceaccount.key 2048

Generating RSA private key, 2048 bit long modulus

..........+++

.............+++

e is 65537 (0x10001)

Add the new key in /etc/kubernetes/apiserver
[root@kamb1021 Builds]# vim /etc/kubernetes/apiserver

KUBE_API_ARGS="--service_account_key_file=/etc/kubernetes/serviceaccount.key"

Add the new key in /etc/kubernetes/controller-manager
[root@kamb1021 Builds]# vim /etc/kubernetes/controller-manager

KUBE_CONTROLLER_MANAGER_ARGS="--service_account_private_key_file=/etc/kubernetes/serviceaccount.key"

Restart services
systemctl restart etcd kube-apiserver kube-controller-manager kube-scheduler

systemctl status etcd kube-apiserver kube-controller-manager kube-scheduler

Source:

https://github.com/kubernetes/kubernetes/issues/11355#issuecomment-127378691

Also we can check is the container is running in the Minion(Node) as well by using the below command, by going to the Node:

              docker ps
If we want to Know what the Pods are running or Particular pod by using the below command in the Master :

              kubectl describe pods
              kubectl describe pod nginx
Note: Here we can see the IP address, So in order to connect to that Particular IP address externaly by using the command "ping 172.17.0.2" we thrown an error, because we don't have route externally to that Pod, in order to rectify that we will create an other resources with our pod in our environment so that we can ping to that IP Address. So the name of the resource is busybox & run this busybox image within the environment, By using the below commands:

              kubectl run busybox --image=busybox --restart=Never --tty -i --generator=run-pod/v1
we will be with the busybox container So paste the below command:

              wget -qO- http://172.17.0.2
  Here we get the access for this Pod internally, we can see the html page of nginx.

Next we can also delate the both image of Busybox & also Particular Pod in the master so that it will delate automatically in the Nodes(Minions) as well by using the below commands:

              kubectl delete pod busybox
              kubectl delete pod nginx
In order to get nginx access externally, we has to first do the port forward, in order to do that create the nginx pod again and do port farword:

              kubectl get pods
              kubectl create -f ./nginx.yaml
              
              kubectl port-forward nginx :80 &
                     (& - represents runit in the background)
                     (we can also do 8080:80 if we need)
                    
              wget -qO- http://localhost:34853 
  Note: Now we get the nginx service externally by using the above wget command.

Tags, Labels and Selectors:

Note: Use only one centos-master & one centos-node1 and stop centos-node2 & centos-node3.

              systemctl stop kubelet kube-proxy ( in both centos-node2 & centos-node3) 
Why this tags,labels comes into picture is that we have "n" Number of Pods in an organization, In order communicate with one pod to another pod there should an label. So, this label has to be mention in the Pod. So inorder to do all this first of all we have to create an YAML file that contains Pods which has label in that Pod. Please see below the YAML file. And name of the YAML file is nginx-pod-label.yaml file:

              apiVersion: V1
              kind: Pod
              metadata:
                name: nginx
                labels:
                  app: nginx
              spec:
                containers: 
                - name: nginx
                  image: nginx:1.7.9
                  ports:
                  - containerPort: 80  
Then create that Pod that contains Label by using the below command:

             kubectl create -f ./nginx-pod-label.yaml
If we want to describe the label particularly, we can use the below command:

             kubectl get pods -l app=nginx ( here we can see the particular label nginx )
             kubectl describe pods -l app=nginx  ( So that we can describe only the label of nginx that we need )
Note: This will be used if we have "n" number of pods contains "n" number of labels, by using the above commands we can describe or get labels of the particular Pod.

Deployment State:

Note: Use only one centos-master & one centos-node1 and stop centos-node2 & centos-node3.

              systemctl stop kubelet kube-proxy ( in both centos-node2 & centos-node3) 
Why this deployment kind came into picture is that, As an Example we can do with a kind deployment and this consists of one Pod (Mentioned in an Replica) and also this Pod (Replica) contain an label associated with it and contains one nginx-container.As per why this deployment kind came into picture is that mainly for the updates. what it exactly mean is that for example we have nginx-deployment-prod.yaml & nginx-deployment-dev.yaml, if we want to update the nginx-container version from 1.7.9 to 1.8 of the nginx-deployment-prod.yaml first we have to test in the dev.

So in order to do that first copy the nginx-deployment-dev.yaml to nginx-deployment-dev-update.yaml and change the version of nginx(container) to 1.8 in the nginx-deployment-dev-update.yaml file and apply those changes. So it automatically update the nginx(container) version to 1.8 in the nginx-deployment-dev.yaml.

Below are the steps, Procedure from creating YAML file. In the Builds Directory we have to create an two YAML files naming it as nginx-deployment-prod.yaml & nginx-deployment-dev.yaml contains the below data:

For nginx-deployment-prod.yaml:

              apiVersion: extensions/v1beta1
              kind: Deployment
              metadata:
                name: nginx-deployment-prod
              spec:
                replicas: 1
                template: 
                  metadata:
                    labels:
                      app: nginx-deployment-prod
                  spec:
                    containers:
                    - name: nginx-deployment-prod
                      image: nginx:1.7.9
                      ports:
                      - containerPort: 80
For nginx-deployment-dev.yaml:

              apiVersion: extensions/v1beta1
              kind: Deployment
              metadata:
                name: nginx-deployment-dev
              spec:
                replicas: 1
                template: 
                  metadata:
                    labels:
                      app: nginx-deployment-dev
                  spec:
                    containers:
                    - name: nginx-deployment-dev
                      image: nginx:1.7.9
                      ports:
                      - containerPort: 80
Create the deployments for Prod & dev by using the below commands:

              kubectl create -f nginx-deployment-prod.yaml
              kubectl create -f nginx-deployment-dev.yaml
we can also check those deployments & pods that we have by using the below commands:

              kubectl get pods ( we can see pods )
              kubectl get deployments ( we can see the both deployments )
As per the senario of the kind deployment we have to do update Now, So in order to do that first copy the file from nginx-deployment-dev.yaml to nginx-deployment-dev-update.yaml by using the below command:

              cp nginx-deployment-dev.yaml nginx-deployment-dev-update.yaml
Change to nginx (container) version in the nginx-deployment-dev-update.yaml file from 1.7.9 to 1.8 as fallows:

              apiVersion: extensions/v1beta1
              kind: Deployment
              metadata:
                name: nginx-deployment-dev
              spec:
                replicas: 1
                template: 
                  metadata:
                    labels:
                      app: nginx-deployment-dev
                  spec:
                    containers:
                    - name: nginx-deployment-dev
                      image: nginx:1.8
                      ports:
                      - containerPort: 80
Run the below commands in order to make this update in nginx-deployment-dev.yaml (But are not creating this nginx-deployment-dev-update.yaml file we are Just updating)

              kubectl apply -f nginx-deployment-dev-update.yaml
Note: what we see here is it will update the nginx-deployment-dev.yaml file with the new version 1.8, we can also check by using "kubectl get pods" and also we can see in the Node(Minion) by using "docker ps". Also we can describe and see what update it is curretly running by using below command:

              kubectl describe deployments -l app=nginx-deployment-dev
Multi-Pod( Container ) Replication Controller:

Note: Use all centos-master & centos-node1, centos-node2 & centos-node3. You can also start the both centos-node2 & centos-node3 by using the command:

             systemctl start kubelet kube-proxy ( in both centos-node2 & centos-node3) 
why,this replication controller came, is that because we can deploy Multiple Pods, in Multiple Nodes at the same time as far we written the YAML File. what it does is that even if we delate the Pod as far we mentioned in the replica, it automatically create a new Pod. And also this each pod conatines container in it, after we create Replication Controller, we can see in the Nodes for the command "docker ps" we can see an container in the Pod accordingly in each Node. How the YAML file looks like is as fallows:

              mkdir Builds
              cd Builds
create an YAML file by naming it as nginx-replication-controller.yaml, and paste the content below:

              apiVersion: v1
              kind: ReplicationController
              metadata:
                name: nginx-www
              spec:
                replicas: 3
                selector:
                  app: nginx
                template:
                  metadata:
                    name: nginx
                    labels:
                      app: nginx
                  spec:
                    containers: 
                    - name: nginx
                      image: nginx
                      ports:
                      - containerPort: 80
Fallow the below steps as fallows first create, delate the Pods, and see the New Pods is created or not by using the below commands:

              kubectl create -f nginx-replication controller.yaml
              
              kubectl get pods
Check that we have containers(Pods) in the centos-Node1 & centos-Node2 & centos-Node3 by using the command:

              docker ps
Now just to make sure that our replication-controller is configured currently are not by delating a Pod. So if we delate a Pod it should automatically spin up a New Pod.

              kubectl delete Pod <Pod Name>
              kubectl get pods
Here if we check we have another Pod created. In order to describe the Pod use the below command

              kubectl describe pod <Pod Name>
In order to delate this whole Replication-controller environment in centos-master Just use the below command:

              kubectl delete ReplicationController 
Create and Deploy services:

Note: Use all centos-master & centos-node1, centos-node2 & centos-node3. You can also start the both centos-node2 & centos-node3 by using the command:

             systemctl start kubelet kube-proxy ( in both centos-node2 & centos-node3)
We has to first create the Replication-contoller as i.e nginx-replicaition-controller.yaml file in other to deploy Services. For creating use the command below:

              kubectl create -f nginx-replication controller.yaml
First, what this exactly says is that,we need to create a service defination, a service defination starts to tie together, we got this three pods across this three minion(Nodes) and they are not Exposed except to other containers in pods and Pods in containers on the same host that they exit on. However, when we define a service we are actually creating a cluster that refers to resource that can exist on any of our minions(Nodes). What do I exactly mean by that, we are abstracting whats running behind the seens in our cluster and we are providing a mechanism for kubernetes simply to asign a single IP Address to those multiple pods to through refered to by Name or by Label or selector so it can connect to them and do something. And can have a Unique IP Address that doesn't matter where we are, if we refered to the IP Address, in the subsequent assigned port from any of the host we can use the cluster IP & Port combination, to work with the entire cluster of pods that meat the selector for the Label or the Name of the application or However we are refering to it.

First Let's build our YAML File for service, by naming the file nginx-service.yaml, and paste the below content:

              apiVersion: v1
              kind: Service
              metadata:
                name: nginx-service
              spec:
                ports:
                - port: 8080
                  targetPort: 80
                  protocol: TCP
                selector:
                  app: nginx
Now we can create a service & see the services, here if we observe a New-service will be created with an IP Address for the nginx port 8000/TCP. Use the below commands:

              kubectl create -f nginx-service.yaml
              
              kubectl get services
For the below command is to describe the service,endpoints, clusterIP address:

              kubectl describe service nginx-service
Note: What we are doing Now is that we are doing a round-robin Interface between this Endd-points, on Port 8000 for this Particular IP address. So how do I connect to it by using the below command:

              kubectl run busybox --generator=run-pod/v1 --image=busybox --restart=Never --tty -i
Here we will login to the container, So use the below command in order to get nginx-html page internally.

              wget -qO- http://10.254.197.123:8000 ( Now we can see the html page )
If we want to delete the busybox again, use the below command:

              kubectl delete pod busybox
If we want to delete the service use the below command:

              kubectl delete service nginx-service ( It won't deletes the Pods it deletes only services )
Creating Temporary Pods at the Command Line:

Note: we are going to use one centos-master & centos-node1, centos-node2 only. You can also stop centos-node3 by using the command below:
systemctl stop kubelet kube-proxy

Here we are going to see without writting an YAML file, how we can create, run a POD with a single container in it by a single command:

              kubectl run mysample --image=latest123/apache 
For the above command it create a deployment for My-sample created (apache), we can also see the Pods, Deployments, describe deployments, describe pods, delete deployment using the below commands:

              kubectl get pods
              kubectl get deployments
              kubectl describe deployments
              kubectl describe pod <pod name>
              kubectl delete deployment <deployment name>
If for Example, if we want to run multiple pods, multiple containers and can run in each Node, we can use replicas here. By using the below command:

              kubectl run myreplicas --image=latest123/apache --replicas=2 --labels=app=myapache,version=1.0
Here for the above command, deployment myreplicas will be created, we can also describe myreplicas, get Pods, check in minions(Nodes), describe with labels by using the below commands:

              kubectl describe deployment myreplicas 
              kubectl get pods
              docker ps ( check in Nodes, one will be running )
              kubectl describe pods -l version=1.0
Interacting with POD containers:

Before creating POD, first write an YAML file by name apache.yaml as fallows:

              apiVersion: v1
              kind: Pod
              metadata:
                name: myapache
              spec:
                containers:
                - name: myapache
                  image: latest123/apache
                  ports:
                  - containerPort: 80
In order to interact with the POD, first of all we has to have an write an YAML file for the POD creation, after writting an YAML File, if we Just create the POD. Then we can interact with that POD how its possible is shown in the below steps:

              kubectl create -f myapache.yaml ( we are creating an apache )
              
              kubectl get pods (we can see that myapache pod is running)
              
              kubectl describe pod myapache ( describes the pod <pod name> )
              
              kubectl exec myapache date ( shows the date )
If we want to attach to that pod (myapache) for the root user, use the below command:

              kubectl exec myapache -i -t -- /bin/bash 
              ps aux | grep apache ( can see the FOREGROUND )
              
              cd /var/www/html ( we won't see nothing here )
              ls -al (we see the index.html )
              
              export TERM=xterm  ( Enivronment variable for our terminal will be set up for our container)
              apt-get update
              apt-get install lynx ( Its a text based web browser, can see want apache within the container)
              lynx http://localhost ( can see apache debian webpage )
              mv index.html index.backup
              echo "This is a test webpage" > index.html 
              lynx http://localhost ( Now can see the index.html page with the content )
Note : If we exit the container (POD) and attach again we can see the xterm because the container within the pod is running.

LOGS: (IMPORTANT):

How to pull logs from our PODS, without having to attach them to run a command or execuate a command on it. We can see it by an kubectl utility. Shows all the system logs how the pod is behaving, also shows the system logs of the container running in our pods in our cluster as shown below:

              kubectl logs <pod name> 
              kubectl logs tail=1 <pod name> ( can get logs for the last log for the pod )
              kubectl logs --since=24h <pod name> ( can see the logs for a period of 24h, we can also see mins & sec also ) 
              kubectl logs -f <pod name> ( we can fallow the logs for that particular pod)
Auto Scaling & Scaling our Pods:

              kubectl run myautoscale --image=latest123/apache --port=80 --labels=app=myautoscale
              
              kubectl get deployments
If we want to autoscale the deployments, by keeping the Min & Max numbers:

              kubectl autoscale deployment myautoscale --min=2 --max=6
Immediatly after the autoscaling has runned, we can get atleast a minimum of two pods running can verify by using " kubectl get pods " & " Kubectl get deployments". If we want to make further changes to our current environment by using the above command again:

              kubectl autoscale deployment myautoscale --min=4 --max=6 
Note: It throws an Error that we have autoscale assigned already, what if we want to change our existing environment that we autoscaled currently and want to change again, using the below command:

              kubectl scale --current-replicas=2 --replicas=4 deployment/myautoscale ( we see that myautoscale is scaled )
              docker ps ( 4 instances in the Nodes )
              kubectl get pods ( can see the 4 pods Running )
Can we scale down Now for the existing Autoscaling Environment ? Yes as the above command

              kubectl scale --current-replicas=4 --replicas=2 deployment/myautoscale ( scale down from 4 to 2 )
              
              kubectl get pods ( can see only 2 pods)
Note: Here we can scale down but can' scale down less then what we have written in the autoscaling policy.(I.E first we created that is the one, we can't go less then that minimum number)

FAILURE & RECOVERY :

Why, this faliure & recovery mean, after we created a POD with one container in it as Replicas, and said to deploy this POD on one centos-Node1 each. i.e one POD in centos-Node1 & one POD in centos-Node2, even if any case the centos-Node1 fails, the POD will not be failed in the centos-Master i.e we can see that 2 PODS available in centos-master. However, if the centos-Node1 is started again or recoverd again we can see that one POD we deployed, by uding the command "docker ps".

For example, there is a command for creation.

              kubectl myrecovery --image=latest123/apache --port=80 --replicas=2 --labels=app=myrecovery 
Note: A myrecovery POD will be created and deployed, then later on fallow the above procedure accordingly.
