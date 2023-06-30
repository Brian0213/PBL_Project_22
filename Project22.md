In this project, you will build upon your knowledge of Kubernetes architecture, and begin to deploy applications on a K8s cluster. Kubernetes has a lot of moving parts; it operates with several layers of abstraction between your application and host machines where it runs. So many terms, and capabilities that is not realistic to learn it all at once. Hence, you will be introduced to as many concepts as possible, but gradually.

Within this project we are going to learn and see in action following:

Deployment of software applications using YAML manifest files with following K8s objects:
Pods
ReplicaSets
Deployments
StatefulSets
Services (ClusterIP, NodeIP, Loadbalancer)
Configmaps
Volumes
PersistentVolumes
PersistentVolumeClaims
…and many more

Difference between stateful and stateless applications

Deploy MySQL as a StatefulSet and explain why
Limitations of using manifests directly to deploy on K8s

Working with Helm templates, its components and the most important parts – semantic versioning
Converting all the .yaml templates into a helm chart
Deploying more tools with Helm charts on AWS Elastic Kubernetes Service (EKS)

Jenkins
-MySQL
-Ingress Controllers (Nginx)
Cert-Manager
Ingress for Jenkins
Ingress for the actual application
Deploy Monitoring Tools

Prometheus
Grafana
Hybrid CI/CD by combining different tools such as: Gitlab CICD, Jenkins. And, you will also be introduced to concepts around GitOps using Weaveworks Flux.

Instructions On How To Submit Your Work For Review And Feedback
To submit your work for review and feedback – follow this instruction.

Choosing the right Kubernetes cluster set up
When it comes to using a Kubernetes cluster, there is a number of options available depending on the ultimate use of it. For example, if you just need a cluster for development or learning, you can use lightweight tools like Minikube, or k3s. These tools can run on your workstation without heavy system requirements. Obviously, there is limit to the amount of workload you can deploy there for testing purposes, but it works exactly like any other Kubernetes cluster.

On the other hand, if you need something more robust, suitable for a production workload and with more advanced capabilities such as horizontal scaling of the worker nodes, then you can consider building own Kubernetes cluster from scratch just as you did in Project 21. If you have been able to automate the entire bootstrap using Ansible, you can easily spin up your nodes with Terraform, and configure the cluster with your Ansible configuration scripts.

It it a great combination of tools responsible for different parts of your applications:

Terraform for infrastructure provisioning
Ansible for cluster master and worker nodes configuration
Kubernetes for deploying your containerized application and orchestrating the deployment


Other options will be to leverage a Managed Service Kubernetes cluster from public cloud providers such as: AWS EKS, Microsoft AKS, or Google Cloud Platform GKE. There are so many more options out there. Regardless of whichever one you choose, the experience is usually very similar.

Most organisations choose Managed Service options for obvious reasons such as:

Less administrative overheads
Reduced cost of ownership
Improved Security
Seamless support
Periodical updates to a stable and well-tested version
Faster cluster spin up
… and many more

However, there is usually strong reasons why organisations with very strict compliance and security concerns choose to build their own Kubernetes clusters. Most of the companies that go this route will mostly use on-premises data centres. When there is need to store data privately due to its sensitive nature, companies will rather not use a public cloud provider. Because, if they do, they have no idea of the physical location of the data centre in which their data is being persisted. Banks and Governments are typical examples of this.

Some setup options can combine both public and private cloud together. For example, the master nodes, etcd clusters, and some worker nodes that run stateful applications can be configured in private datacentres, while worker nodes that require heavy computations and stateless applications can run in public clouds. This kind of hybrid architecture is ideal to satisfy compliance, while also benefiting from other public cloud capabilities.



Deploying the Tooling app using Kubernetes objects
In this section, you will begin to write configuration files for Kubernetes objects (they are usually referred as manifests) in the form of files with yaml syntax and deploy them using kubectl console. But first, let us understand what a Kubernetes object is.

Kubernetes objects are persistent entities in the Kubernetes system. Kubernetes uses these entities to represent the state of your cluster. Specifically, they can describe:

What containerized applications are running (and on which nodes)
The resources available to those applications
The policies around how those applications behave, such as restart policies, upgrades, and fault-tolerance
These objects are "record of intent" – once you create the object, the Kubernetes system will constantly work to ensure that the object exists. By creating an object, you are effectively telling the Kubernetes system what you want your cluster’s workload to look like; this is your cluster’s desired state.

To work with Kubernetes objects – whether to create, modify, or delete them – you will need to use the Kubernetes API. When you use the kubectl command-line interface, for example, the CLI makes the necessary Kubernetes API calls for you. It is also possible to use curl to directly interact with the Kubernetes API, or it can be as part of developing a program in different programming languages. That will require some advance knowledge. You can read more about client libraries to get an idea on how that works.

Before creating the EKS cluster ensure the version of the Kubectl matches the cluster.

- ISSUE

Unable to locate kubecconfig after running the command:

`kubectl get service nginx-service -o wide`

FIX

Create a cluster.yaml file `sudo vi cluster.yaml`.

[Cluster Yaml File](./images/clust-yml.PNG)

Ran `eksctl create cluster -f cluster.yaml`

Error `unable to use kubectl with the EKS cluster (check 'kubectl version'): WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.` occcured

Fix

Ran the command below to update kubectl:

`sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl`

`sudo curl -LO https://dl.k8s.io/release/v1.27.3/bin/linux/amd64/kubectl`

Check the home folder

`ls`

Ran the commands below to manage kubectl:

`sudo chmod +x kubectl
sudo mkdir -p ~/.local/bin
sudo mv ./kubectl ~/.local/bin/kubectl`

Ran the command to confirm kubectl version

`kubectl version --client --output=yaml`

[Kubectl Latest Version](./Screenshots/late-kubectl-vers.PNG)

Ran `eksctl create cluster -f cluster.yaml` to create the cluster.yaml file

Command to locate kubeconfig `sudo vi ~/.kube/config`

To create cluster without a yaml file:

`eksctl create cluster --name devtest --version 1.24 --region eu-west-1 --nodegroup-name linux-nodes --node-type t2.medium --nodes 2`

To delete cluster `eksctl delete cluster devtest`

- TO CREATE NGINX-SERVICE:

1. Create nginx-pod.yaml file in you home:

[Nginx Pod Yaml](./Screenshots/nginx-pod.PNG)

2. Run this command:

`kubectl apply -f nginx-pod.yaml`

[Kubectl Nginx Pod](./Screenshots/nginx-pod-output.PNG)

3. Create nginx-service.yaml file in you home:

[Nginx Service Yaml](./Screenshots/nginx-service.PNG)

4. Run this command:

`kubectl apply -f nginx-service.yaml`

[Kubectl Nginx Service](./Screenshots/nginx-service-output.PNG) 

5. Run the command below to confirm the status of the service:

`kubectl get services nginx-service`

[Kubectl Nginx Service Status](./Screenshots/nginx-service-status.PNG)

Step 1:

Run this command:

`kubectl get service nginx-service -o wide`

[Kubectl Get Nginx Service](./Screenshots/get-nginx-service.PNG)

As you already know, the service’s type is ClusterIP, and in the above output, it has the IP address of 10.100.71.130 – This IP works just like an internal loadbalancer. It accepts requests and forwards it to an IP address of any Pod that has the respective selector label. In this case, it is app=nginx-pod. If there is more than one Pod with that label, service will distribute the traffic to all these pods in a Round Robin fashion.

Now, let us have a look at what the Pod looks like:

`kubectl get pod nginx-pod --show-labels`

[Kubectl Get Nginx Show Labels](./Screenshots/get-nginx-show-lab.PNG)

Notice that the IP address of the Pod, is NOT the IP address of the server it is running on. Kubernetes, through the implementation of network plugins assigns virtual IP adrresses to each Pod.

`kubectl get pod nginx-pod -o wide`

[Kubectl Get Nginx Pod](./Screenshots/get-nginx-pod.PNG)

Therefore, Service with IP 10.100.71.130 takes request and forwards to Pod with IP 172.50.197.236

Self Side Task:

1. Build the Tooling app Dockerfile and push it to Dockerhub registry.

2. Write a Pod and a Service manifests, ensure that you can access the Tooling app’s frontend using port-forwarding feature.

Expose a Service on a server’s public IP address & static port

Sometimes, it may be needed to directly access the application using the public IP of the server (when we speak of a K8s cluster we can replace ‘server’ with ‘node’) the Pod is running on. This is when the NodePort service type comes in handy.

A Node port service type exposes the service on a static port on the node’s IP address. NodePorts are in the 30000-32767 range by default, which means a NodePort is unlikely to match a service’s intended port (for example, 80 may be exposed as 30080).

Update the nginx-service yaml to use a NodePort Service:

[Kubectl Get Nginx Pod](./Screenshots/nginx-service-update.PNG)

Running the following commands to inspect the setup:

`kubectl get pods`

`kubectl describe pod nginx-pod`

[Kubectl Get Describe Nginx Pod](./Screenshots/kube-get-desc-pods1.PNG)

[Kubectl Get Describe Nginx Pod](./Screenshots/kube-get-desc-pods2.PNG)

STEP 2: Accessing The Nginx Application Through A Browser

First of all, Let's try accessing the Nginx Pod through its IP address from within the Kubernetes cluster. To do this an image that already has curl software installed is needed.

Running the kubectl command to run the container that has curl software in it as a pod:

`kubectl run curl --image=dareyregistry/curl -i --tty`

Running curl command and pointing it to the IP address of the Nginx Pod:

`curl -v 192.168.48.219`

[Curl](./Screenshots/curl-output.PNG)