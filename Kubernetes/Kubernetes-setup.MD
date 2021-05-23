# Setup Kubernetes (K8s) Cluster on AWS


1. Create Ubuntu EC2 instance <br/>
   Add tag: Name: k8s-management-server  <br/>
   When SSH'ing to the AWS instance from putty or mobaxterm, use username "ubuntu" since we are using ubuntu system.  <br/>
   Select existing secruity group created with the following rules: <br/>
   Custom TCP Rule  TCP   8080  0.0.0.0/0   <br/>
   Custom TCP Rule  TCP   8080  ::/0   <br/>
   SSH              TCP   22    0.0.0.0/0   <br/>
   After you SSH, impersonnate as root "sudo su -" and run the commands below.
   
2. install AWSCLI
   ```sh
    curl https://s3.amazonaws.com/aws-cli/awscli-bundle.zip -o awscli-bundle.zip
    apt install unzip python
    unzip awscli-bundle.zip
    #sudo apt-get install unzip - if you dont have unzip in your system
    # run the command below to copy our files under /usr/local/bin
    ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
    ```
    Note: <br/>
    If you get the following error when running last command: "/usr/bin/env: ‘python’: No such file or directory"  <br/>
    Possible Solution #1  <br/>
    If Python 3 is not installed, install it: apt-get install python3  <br/>
    Possible Solution #2  <br/>
    If Python 3 has been installed, run these commands: whereis python3  <br/>
    Then we create a symlink to it: sudo ln -s /usr/bin/python3 /usr/bin/python   <br/>
    apt-get update  <br/>
    apt-get install python3-venv
    
    To confirm awscli has been installed, run the command: aws --version 

3. Install kubectl on ubuntu instance
   ```sh   
    curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin/kubectl
   ```

4. Install kops on ubuntu instance
   ```sh
    curl -LO  https://github.com/kubernetes/kops/releases/download/1.15.0/kops-linux-amd64
    chmod +x kops-linux-amd64
    sudo mv kops-linux-amd64 /usr/local/bin/kops
    kops version (it should be 1.15.0)
    Note: use below command if you wish to use latest version. For now we could see latest version of kops. So ignore it until further update. 
    # curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
    ```

5. Create an IAM user/role  with Route53, EC2, IAM and S3 full access <br/>
   EC2 IAM role: Allows EC2 instances to call AWS services on your behalf. <br/>
   Services --> IAM --> Roles --> Create Role --> EC2 --> search for and add these 4 policies: EC2Fullaccess, S3FullAccess, Route53FullAccess, and IAMFull Access. <br/>
   Add tags:  <br/>
      Key: Name   <br/>
      Value: k8-role   <br/>
   Next  <br/>
   Role name: k8s-role  <br/>
   Create role.

6. Attach IAM role to ubuntu instance  <br/>
   Go to your kubernetes EC2 instance, to attach the role to EC3 instance: <br/>
   EC2 Instance --> k8s-management-server (created kubernetes EC2 instance) --> select the instance --> Actions --> Instance Settings --> Attach/Replace IAM Role --> IAM role: choose the created kubernetes role (k8s-role) --> Apply <br/>
   If IAM Role attachment, is not present under "Instance Settigs", it will be under Security --> Modify IAM Role
   ```sh
   # Note: If you create IAM user with programmatic access then provide Access keys. Otherwise region information is enough
   aws configure
   # AWS Access Key ID: In our case no need to provide "Access Key" since we have attached code
   # AWS Seceret Access Key: We do not need also secret access code.
   # Default region name: ca-central-1  (this is central canada region)
   # Default output format: keep it as it is. By default it is JSON.
   ```
   So we have setup our ubuntu system to create kubernetes cluster on our AWS account.

7. Create a Route53 private hosted zone (you can create Public hosted zone if you have a domain) <br/>
   A hosted zone is a container that holds information about how you want to route traffic for a domain, such as example.com, and its subdomains.  <br/>
   ```sh
   On AWS console go to Services --> Route53 (it is under netwroking) --> hosted zones --> created hosted zone  (click on Get started now)
   Domain Name: valaxy.net
   Type: Private hosted zone for Amazon VPC. Make sure you are chosing right VPC if you have multiple
   VPC ID: choose the region we are using which is ca-central-1 (you can see the region on AWS console, top-right corner)
   The click on Create
   ```
   
8. Create an S3 bucket  <br/>
   Buckets are containers for data stored in S3.  <br/>
   ```sh
   # On AWS go to Services --> S3  (present under Storage) --> Create bucket, or instead of creating the bucket from AWS you can create it from the CLI using the command below:
   aws s3 mb s3://demo.k8s.valaxy.net
   # If you refresh your AWS console, you'll see the s3 bucket created.
   ```
9. Expose environment variable:
   ```sh
    # We need to export the variable so that kps can use it.
    # Run the command below from the CLI:
    export KOPS_STATE_STORE=s3://demo.k8s.valaxy.net
   ```

10. Create sshkeys before creating cluster
    ```sh
    # This key pair is needed to login to our kubernetes cluster.
    # From CLI run the command below:
    ssh-keygen
    # The key is generated under .ssh directory:
    cd .ssh/
    ls -l
    # You'll see key files: "id_rsa" and "id_rsa.pub"
    ```

11. Create kubernetes cluster definitions on S3 bucket
    ```sh
    kops create cluster --cloud=aws --zones=ca-central-1a --name=demo.k8s.valaxy.net --dns-zone=valaxy.net --dns private 
    # --name is the name of our cluster. We'll give it the same name as the name of the bucket we created. If name exists, use a different unique name.
    # --zones is different from region. Forexample, we have region as "ca-central-1", and this has 2 availability zones "ca-central-1a" and "ca-central-1b". You can see the availability zone, by selecting an EC2 instance --> Networking --> Availibility Zone 
    # Note: we do not store our devops environment and target environment in same VPCs, due to security reasons. So if you see once the cluser is created, in the config (output on the screen), it'll setup a new cliuster. 
     ```

12. Commands
    ```sh
    To list clusters: kops get cluster
    To edit this cluster: kops edit cluster demo.samere.k8s.valaxy.net
    To edit your node instance group: kops edit ig --name=demo.samere.k8s.valaxy.net nodes
    To edit your master instance group: kops edit ig --name=demo.samere.k8s.valaxy.net master-ca-central-1a
    ```
   
13. Create kubernetes cluser
    ```sh
    kops update cluster demo.k8s.valaxy.net --yes
    ```
    Once the cluster is created, you can see in the AWS console: new files get created under the S3 bucket, <br/>
    2 new IAM roles get created: "masters.demo.k8s.valaxy.net" and "nodes.demo.k8s.valaxy.net"  <br/>
    Additional routes get created under Route53 <br/>
    If you go to EC2 Instances: you see 2 new node.demo instances got created and 1 master instance  <br/>
    Autoscaling: Autoscaling --> autoscaling group also gets created.

14. Commands
    ```sh
    Validate cluster: kops validate cluster   --> this command will fail until the cluster is created (it takes 5-10 min for it to be created)
    List nodes: kubectl get nodes --show-labels
    SSH to the master: ssh -i ~/.ssh/id_rsa admin@api.demo.k8s.valaxy.net
    This "api.demo.k8s.valaxy.net" is our DNS name. We can also use instead the public ip of our master server (under EC2 instances, check the public of the master instance created. Always login as admin user (admin@api.demo.k8s.valaxy.net), never login as root user. Before you SSH impersonate on ansible host as root (sudo su -))
    You can manage your cluster from the CLI because it is an ubuntu system and we installed kubctl commands. On CLI type "kubctl" to see the list of commands.
    ```
    Rather than managing our system from the ubuntu system that we created ("k8s-management-server"), we'll connect to the master EC2 instance, but we'll need to install kubctl on that instance aswell.
   
15. To change the kubernetes master and worker instance sizes 
    ```sh 
    kops edit ig --name=<cluster_name> nodes
    #kops edit ig --name=demo.k8s.valaxy.net nodes 
    kops edit ig --name=<cluster_name> master-<zone_name>
    #kops edit ig --name=demo.k8s.valaxy.net master-ap-south-1b
    ```
   
16. To Delete cluster (try once your lab is done)
    ```sh 
    kops delete cluster <cluster_name> --yes
    ```
   
17. Validate your cluster
    ```sh
    kops validate cluster
    ```

18. To list nodes
    ```sh
    kubectl get nodes
    ```
   
#### Deploying Nginx pods on Kubernetes
1. Notes  <br/>
   SSH to the master: ssh -i ~/.ssh/id_rsa admin@api.demo.k8s.valaxy.net  <br/>
   Impersonnate as root user: sudo su -  <br/>
   Run "kubectl" command to confirm that it is working. <br/>
   kubectl get nodes  --> this will show you the nodes (1 master and 2 client nodes).  <br/>
   If you get an error running the above command, saying "connection to the server localhost:8080 was refused - did you specify the right host or port" --> then on the local host delete and rectreate the file .ssh/known_hosts, and add into it localhost   <br/>
   kubectl get pods --> once we create pods, you can see them by running this command.  Once you deploy and run this command, you'll see under "READY" column something like 1/1 --> the first "1" is for pods and the second "1" is for containers. Pods and containers are different. Within a pod our containers run. <br/>
   kubectl get nodes -o wide --> this gives more details about the pods, like the internal IP used to access the application, and the node IPs (each pod is created a different node to make it highly available). <br/>
   kubectl get deploy  or kubectl get deployments --> this will show you deployments done.   <br/>
   kubectl get service --> you'll see services there once created. "kubrnetes" service you see is by default created with the cluster. <br/>
   We will create a deployment, and whenever we create a deploy, in the background it'll create pods. Deploy is going to create replica set and the replica set are going to make sure your pods are always available. <br/>
   The application can be accessible only by creating a service. Service is needed to expose the application to the outside.
   
1. Deploying Nginx Container   
   ```sh
    kubectl run --generator=run-pod/v1 sample-nginx --image=nginx --replicas=2 --port=80
    # "sample-nginx" is the deployment name.
    # The image is pulled from docker hub.
    # "replicas" specifies how many pods do you need. If nothing is specified, it'll create 1 pod. Better to have replicas so that your application is highly available, if 1 replica goes down.
    # "port": ngnix runs on port 80. This is the port we are exposing to the external network.
    #kubectl run sample-nginx --image=nginx --replicas=2 --port=80
    # kubectl run simple-devops-project --image=yankils/simple-devops-image --replicas=2 --port=8080
    kubectl get pods
    kubectl get deployments
    # So far our application is accessible internally: 
    wget <internal IP>:80
    # Port 80 is the default port, so even if you do not specify it, you can still access your application: 
    wget <internal IP>
    # The index.html file should be accessible from the browser now. (if you do "ls" you see the index.html we deployed before.)
    ```

1. Expose the deployment as service. This will create an ELB in front of those 2 containers and allow us to publicly access them.
   ```sh
   kubectl expose deployment sample-nginx --port=80 --type=LoadBalancer
   # "LoadBalancer" is the service type. We have different types of services, depending on the use case you can choose the right one.
   # kubectl expose deployment simple-devops-project --port=8080 --type=LoadBalancer
   kubectl get services -o wide
   # By running above command , you'll see the LoadBalancer service we created. (You'll not get an External IP, you'll see it "pending"). Also you'll see a field Ports: 80/30001:TCP --> 80 is the port accessible within the cluster, 30001 is the port used to access the application from outside the cluster, so we'll use port 30001 from the browser <master_server_ip>:<port>.
   ```
   Not you'll need to expose port 30001 used above in the security group used in AWS. So add to the security group the following ruke: <br/>
   Type: Custom TCP  /  Protocol: TCP / Port range: 30001 / Source: 0.0.0.0/0
   
   To delete a pod you do:  kubectl delete pod <pod_id>   <br/>
   Once you delete the pods, the application won't be accessible, but our deployment detects that the pods are not running and re0-creates new pods. <br/>
   In real world, we do not create services using commands, but we'll use yaml file that has all the definitions. And whenever there is a new deployment we just need to remove the old containers, and it'll remove the old image and create add the new one.
   
   
