# K8s-by-Loi
This project will show all about K8s: such as theory, how to deploy, and so on.

## Table of Contents
[Overview](#overview)

[Definition](#definition)
  1. Kubernetes provides
  2. Kubernetes's architecture
     - Master node (or Control plane)
         * Kube API server
         * etcd storage
         * Scheduler
         * Controller Manager
     - Worker node
         * Kubelet
         * Kube Proxy
         * Container Runtime
         * Addons
  3. Let's overview the Kubernetes's architecture once again
  4. In a Pod, you can have one or many containers?
  5. How will Pod 1 and Pod 6 (these pods are in different node) interact?

[Practice](#practice)
  1. Deploy K8s using Minikube

### Overview 
K8s stands for Kubernetes.
### Definition
Kubernetes is really not a replacement for docker engine. Kubernetes manages the cluster of docker engine. And not only docker engine, it can manage cluster of other container runtime environment like _rocket_.

(_rkt_ stands for _rocket_ is an application container engine developed for modern production cloud-native environments. It features a pod-native approach, a pluggable excution environment, and a well-defined surface area that makes it ideal for itergration with other systems. The core excution unit of _rkt_ is the pod, a collection of one or more applications excuting in a shared context (rkt's pod are synonymous with the concept in the K8s orchestration system). _rkt_ allows users to apply different configurations (like isolation parameters) at both pod-level and at the more granular per-application level. rkt's architecture means that each pod excutes directly in the classic Unix process model (i.e. there is no central daemon), in a self-contained, isolated environment. _rkt_ implements a modern, open, standard container format, the App container (appc) spec, but can also excute other container images, like those created with _Docker_).
#### 1. Kubernetes provides
- Service discovery and load balancing: You create a container which gets automatically dicovered by the load balancer and it gets updated in the load balancer.
- Storage orchestration: K8s provide intergration with lots of storage, like SAN, NAV, EBS volume, CEPH storage.
- Automated rollouts and rollbacks: easy to roll out a new image version and also roll back very easily if it's not working.
- Automatic bin packing: It's going to place your container on the right node where it gets the right resources based on the requirement.
- Self-healing: If the node goes down, it brings your containers to life on the live node. Apart from that, you containers are also monitored. You can set that just like **_Auto Scaling group_**. When an instance goes down, **_Auto Scaling group_** will launch a replacement like that. The **_Self-healing_** capability, it's much faster than **_Auto Scaling_**.
- Secret and configuration management: You can manage the configuration in form of variables and volumes and also secrets which are encoded values.

#### 2. Kubernetes's architecture
![image](https://github.com/loihoang1411/K8s-by-Loi/assets/126635820/45cd63ce-a8e0-4ae8-869b-ffb4b6e78fb4)
Two main components: Master node and Worker node. 
##### Master node (or Control plane):
It is the one that managing these worker nodes. So you don't log into the worker node and run the containers, you tell it to the master node. You don't even log into the master node, you connect by using some client. You give information to the master node (.yaml file) that you want to run about the containers. And it is going to take the action based on the requirement. There are four primary services: API server, Scheduler, Controller Manager and etcd.
###### Kube API server: 
- It handles all the incoming and out going communcation. When you want to send instructions to Kubernetes, Kube API server is going to recieve that. And then it's going to pass the information to other services like Scheduler, etcd, and Worker nodes.
- Component on the Master node that exposes the Kubernetes API. If you want, you can build your own tool that gets intergrated with Kubernetes API. There are so many third party tools to be available to intergrate with your Kubernetes API which you can use like monitoring agent, logging agent, web dashboards.
- It is the fornt-end of the control plane or the whole Kubernetes cluster.
- Admins connects to Kube API server using Kubectl CLI.
- Web dashboard can be intergrated with this API.
###### etcd storage:
- Stores all the information of your Kubernetes cluster.
- The Kube API service is going to store or retrieve information from this.
- It will have all the runtime information.
- It should be backed up regularly because if this fails, you lose the current data.
- Stores current state of everything in the cluster.
###### Scheduler:
- Scheduler is going to schedule the container on the right node.
- watches newly created pods that have no node assigned, and selects a node for them to run on.
- Factors taken into account for scheduling decisions include:
    - individual and collective resource requirements.
    - hardware/software/policy contraints.
    - affinity and anti-affinity specifications.
    - data locality.
    - inter-workload interference and deadlines.
###### Controller Manager: 
- Logically, each **_controller_** is a separate process.
- These controllers include:
    - Node Controller: Responsible for noticing and responding when nodes go down.
    - Replicaion Controller: Responsible for maintaining the correct number of pods for every replication controller object in the system.
    - Endpoints Controller: Populates the Endpoints object (that is, joins Service & Pods).
    - Service Account & Token Controllers: manages the authentication and authorization, create default accounts and API access tokens for new namespace.

##### Worker node:
Worker node is the one where docker engine are running. Worker node, we have: Kubelet, Proxy, Docker engine.
###### Kubelet
- An agent that runs on each node in the cluster. It makes sure that containers are running in a pod. It's going to listen to your kubernetes master node requests or commands.
- When the Scheduler decides that this Worker node is going to run the container, it's going to assign the responsiblity to Kubelet. Now Kubelet is going to fetch your image and run the container from it.
###### Kube Proxy
- Kube Proxy actually is a network proxy that is going to run on every node in the cluster.
- You can set network rules also like security groups. (E.g: rules allow network communication to your Pods inside or outside of your cluster.)
###### Container Runtime
- Docker
- containerd
- cri-o, rktlet
- Kubernetes CRI (Container Runtime Interface)
###### Add ons
These components are taken by some third party vendors who have some specialization in that area, like better logging tool, better monitoring tool, or webUI, or DNS server. 
- DNS
- webUI
- Container Resource Monitoring
- Cluster Level Logging

#### 3. Let's overview the Kubernetes's architecture once again:
![image](https://github.com/loihoang1411/K8s-by-Loi/assets/126635820/945fb4d3-3b91-46da-80c5-ce2c1831444e)

**kubectl** is a tool, which we are going to connect to the **Kubernetes master node**. Master node has **API server**: enables the communication; **Scheduler**: decides where your container will be running on which node; **Controller Manager**: responsible for monitoring **Worker nodes**, your containers, and also the authentication, authorization; and **etcd**: stores the current information. In Worker node, you have **kubelet**, which is the agent, will do all the heavy lifting on the containers: it's going to fetch the image, run the containers, map the volumes, do all those stuff. **kube proxy** is a network proxy. If you want to expose a Pod to the outside world, you can do it through Kube proxy or you can even set the network rules. And then, the **docker engine**, where your containers will be running, as you can see, containers enclosed in Pod. So what is Pod?
![image](https://github.com/loihoang1411/K8s-by-Loi/assets/126635820/823b15e0-07ad-4954-bb24-18f3b06cf0bc)
- What is the relation between Pod and the container? It's the same relation as the VM and the process running inside it (the VM is going to provide all the resource to the process, which is running RAM, network, CPU, storage, everything and the process just uses it). Similarly, Pod is going to provide all the resource to a container. The container will be running inside the Pod, so container will be like the process, and Pod will be like the VM. But, there is no virtualization, it's isolation.
- Why does Kubernetes uses Pod, why not directly run containers? Because Kubernetes can use different container runtime environment like Docker, Rocker, CRI. If you don't have the Pod, there will be no abstraction. Now we have Pod, it's a standard set of commands, standard set of configuration that we do, it doesn't matter what technology we are using behind the scene. So if you're running a "process A" in the Pod, the "process A" will be the container, which will be running on port 8080 (for example), and the Pod will give the IP address. You can access it by giving the Pod IP and the port number of the container.

#### 4. In a Pod, you can have one or many containers?
![image](https://github.com/loihoang1411/K8s-by-Loi/assets/126635820/9841afb6-e28b-4694-ad62-4d372314dfef)
Pod gives the resources to the containers. It really depends.
Ideally, you see one container inside the Pod, named **main**, the other containers will be the helper containers.
- In node 1, you have a Pod and a main container running inside that. One Pod - One container.
- In node 2, you have 3 containers in a Pod. **init** container will be short lived container. It's going to start does some commands excutions and then it will be dead. Then, **main** container will start with **sidecar** container. If you have **sidecar** container, its work will be helping **main** container (for example: streaming the log, it could be a logging agent or a monitoring agent to help **main** container). But at any given point of time, you should have one main container only running in the Pod. The other containers will be helper containers. If you have MySQL and Tomcat, you are not going to run both in the same Pod. You'll run it on the different pods. -> Pod will be distributed across multiple Worker nodes.

#### 5. How will Pod 1 and Pod 6 (these pods are in different node) interact?
![image](https://github.com/loihoang1411/K8s-by-Loi/assets/126635820/11776650-8237-4f88-8b4a-61785ec16aab)
On every node, you'll have a **bridge 00** (or a subnet, like Local Area Network - LAN). **bridge 0** acts like a switch, so all the Pods running in the node will be able to communicate with each other. If you want to connect a Pod in a different node, **bridge 0** is going to forward the request to the **wg0**, **wg0** acts like a router. **wg0** is going to route it to the right node router by looking at the IP addres. Then the destination router recieves the request and forward to the destination switch and switch send to the destination Pod. Anyway, every node will have a small private network, and all these private networks will be connected in one bigger network. This is **Overlay network**.

### Practice
#### Deploy K8s using Minikube (Windows OS)
  1. Install Git on your computer. (follow the link: https://git-scm.com/download/win)
  2. Open Windows PowerShell as an Aministrator
  3. Clone repository github (follow the link: https://github.com/devopshydclub/vprofile-project):
     ```
     cd ../..
     cd Downloads
     git clone https://github.com/devopshydclub/vprofile-project.git
     ```
#### Deploy K8s using Kubespray:
  1. Requirement: 3 nodes (2 CPUs and 4GB RAM, or higher). This is up to you, but anw, you should have at least 3 nodes, with 1 Master node and 2 Worker nodes. And you need another node to deploy K8s to 3 nodes above.
  2. I'll use CentOS 7.9 operating system for 4 nodes and I rented these virtual machines from a cloud service provider. My nodes default to using the **_root_** user.  Run this command first: ```yum update -y```. Nodes's information are shown below:

     **Deploy node:** 1 vCPU and 1GB RAM
     ```
      [root@deploy-node ~]# ip a
      1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
          link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
          inet 127.0.0.1/8 scope host lo
             valid_lft forever preferred_lft forever
          inet6 ::1/128 scope host
             valid_lft forever preferred_lft forever
      2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
          link/ether fa:16:3e:80:86:c5 brd ff:ff:ff:ff:ff:ff
          inet 192.168.0.19/24 brd 192.168.0.255 scope global dynamic eth0
             valid_lft 86384sec preferred_lft 86384sec
          inet6 fe80::f816:3eff:fe80:86c5/64 scope link
             valid_lft forever preferred_lft forever
     ```
     **Target nodes:**

     Master node (2vCPUs and 8GB RAM):
     ```
      [root@master-node ~]# ip a
      1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
          link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
          inet 127.0.0.1/8 scope host lo
             valid_lft forever preferred_lft forever
          inet6 ::1/128 scope host
             valid_lft forever preferred_lft forever
      2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
          link/ether fa:16:3e:6e:e7:14 brd ff:ff:ff:ff:ff:ff
          inet 192.168.0.48/24 brd 192.168.0.255 scope global dynamic eth0
             valid_lft 84533sec preferred_lft 84533sec
          inet6 fe80::f816:3eff:fe6e:e714/64 scope link
             valid_lft forever preferred_lft forever
     ```

     Worker node 1 (2vCPUs and 8GB RAM):
     ```
      [root@worker-node-1 ~]# ip a
      1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
          link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
          inet 127.0.0.1/8 scope host lo
             valid_lft forever preferred_lft forever
          inet6 ::1/128 scope host
             valid_lft forever preferred_lft forever
      2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
          link/ether fa:16:3e:53:84:4a brd ff:ff:ff:ff:ff:ff
          inet 192.168.0.211/24 brd 192.168.0.255 scope global dynamic eth0
             valid_lft 84640sec preferred_lft 84640sec
          inet6 fe80::f816:3eff:fe53:844a/64 scope link
             valid_lft forever preferred_lft forever
     ```
     
     Worker node 2 (2vCPUs and 8GB RAM):
     ```
      [root@worker-node-2 ~]# ip a
      1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
          link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
          inet 127.0.0.1/8 scope host lo
             valid_lft forever preferred_lft forever
          inet6 ::1/128 scope host
             valid_lft forever preferred_lft forever
      2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
          link/ether fa:16:3e:db:9a:1c brd ff:ff:ff:ff:ff:ff
          inet 192.168.0.54/24 brd 192.168.0.255 scope global dynamic eth0
             valid_lft 85057sec preferred_lft 85057sec
          inet6 fe80::f816:3eff:fedb:9a1c/64 scope link
             valid_lft forever preferred_lft forever
      ```
4. Enable SSH connection between Deploy node to others and between Master node to Worker nodes.
     - **At all nodes, do the following (Deploy node, for example):**

     Generate SSH key:
     ```ssh-keygen```. Results:
     ```
      [root@deploy-node ~]# ssh-keygen
      Generating public/private rsa key pair.
      Enter file in which to save the key (/root/.ssh/id_rsa):
      Enter passphrase (empty for no passphrase):
      Enter same passphrase again:
      Your identification has been saved in /root/.ssh/id_rsa.
      Your public key has been saved in /root/.ssh/id_rsa.pub.
      The key fingerprint is:
      SHA256:xxxabc... root@deploy-node.abcxyz
      The key's randomart image is:
      +---[RSA 2048]----+
      |  12  o.=+=.     |
      |  3  . . O *   . |
      |  4     * 4 B *..|
      |  5    * B o X oo|
      |  6   6 S + 4 1 .|
      |  7    1 = 2 +   |
      |      o + E 2    |
      |       3     2   |
      |             21  |
      +----[SHA256]-----+
     ```
     - **At Deploy node, SSH to the remaining nodes:**

    To Master node:
     ```ssh-copy-id root@192.168.0.48```. Results:
     ```
      [root@deploy-node ~]# ssh-copy-id root@192.168.0.48
      /usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
      The authenticity of host '192.168.0.48 (192.168.0.48)' can't be established.
      ECDSA key fingerprint is SHA256:t9/DL+62/z1I9knNiMTI7suM6cdrB6L28unXYRbkZQ4.
      ECDSA key fingerprint is MD5:61:8c:87:d9:28:1e:96:e6:82:f6:60:3c:7e:8d:70:71.
      Are you sure you want to continue connecting (yes/no)? yes
      /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
      /usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
      root@192.168.0.48's password:
      
      Number of key(s) added: 1
      
      Now try logging into the machine, with:   "ssh 'root@192.168.0.48'"
      and check to make sure that only the key(s) you wanted were added.
     ```


   To Worker node 1:
     ```ssh-copy-id root@192.168.0.211```

   To Worker node 2:
     ```ssh-copy-id root@192.168.0.54```
   
     - **At Master node, SSH to the Worker noes:**

   To Worker node 1:
     ```ssh-copy-id root@192.168.0.211```

   To Worker node 2:
     ```ssh-copy-id root@192.168.0.54```
5. Deploy

   At Deploy nodes, do the following:
   - Git clone Kubespray, version 2.16:
     ```
     yum install git -y
     git clone https://github.com/kubernetes-sigs/kubespray.git --branch release-2.16
     ```
   - Go to the folder:
     ```
     cd kubespray/inventory/sample/
     ```
   - Create and write ```host.yaml``` file:
     ```
     vi host.yaml
     ```
     Content:
     ```
      [all]
      master ansible_host=192.168.0.48 ip=192.168.0.48
      worker1 ansible_host=192.168.0.211 ip=192.168.0.211
      worker2 ansible_host=192.168.0.54 ip=192.168.0.54
      
      [kube_control_plane]
      master
      
      [etcd]
      master
      
      [kube_node]
      worker1
      worker2
      
      [k8s_cluster:children]
      kube_node
      kube_control_plane
     ```
   - Install docker:
     ```
     curl -fsSL https://get.docker.com/ | sh
     systemctl start docker
     ```
   - Docker run:
     ```
     docker run --rm -it --mount type=bind,source=/root/kubespray/inventory/sample/,dst=/kubespray/inventory quay.io/kubespray/kubespray:v2.16.0 bash 
     ```
     After running the above command, we have already executed it inside the container. Please note that the prompt will now be "root@a3247643f8e2:/kubespray#".

     Now, you'll enable SSH connection between the container to other nodes (except Deploy node):
     ```
     ssh-keygen
     ssh-copy-id root@192.168.0.48
     ssh-copy-id root@192.168.0.211
     ssh-copy-id root@192.168.0.54
     ```

     Run the final command:
     ```ansible-playbook -i inventory/host.yaml cluster.yml```
     
