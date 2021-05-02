# Setup Kubernetes (K8s) Cluster on AWS


1. Create Ubuntu EC2 instance <br/>
   Add tag: Name: k8s-management-server  <br/>
   When SSH'ing to the AWS instance from putty or mobaxterm, use username "ubuntu" since we are using ubuntu system.  <br/>
   Select existing secruity group created with the following rules: <br/>
   Custom TCP Rule  TCP   8080  0.0.0.0/0   <br/>
   Custom TCP Rule  TCP   8080  ::/0   <br/>
   SSH              TCP   22    0.0.0.0/0   <br/>
   After you SSH, impersonnate as root "sudo su -" and run the commands below.
   
1. install AWSCLI
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

1. Install kubectl on ubuntu instance
   ```sh
   
   curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin/kubectl
   ```

1. Install kops on ubuntu instance
   ```sh
    curl -LO  https://github.com/kubernetes/kops/releases/download/1.15.0/kops-linux-amd64
    chmod +x kops-linux-amd64
    sudo mv kops-linux-amd64 /usr/local/bin/kops
    kops version (it should be 1.15.0)
    Note: use below command if you wish to use latest version. For now we could see latest version of kops. So ignore it until further update. 
    # curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64

    ```
1. Create an IAM user/role  with Route53, EC2, IAM and S3 full access <br/>
   EC2 IAM role: Allows EC2 instances to call AWS services on your behalf. <br/>
   Services --> IAM --> Roles --> Create Role --> EC2 --> search for and add these 4 policies: EC2Fullaccess, S3FullAccess, Route53FullAccess, and IAMFull Access. <br/>
   Add tags:  <br/>
      Key: Name   <br/>
      Value: k8-role   <br/>
   Next  <br/>
   Role name: k8s-role  <br/>
   Create role.

1. Attach IAM role to ubuntu instance  <br/>
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

1. Create a Route53 private hosted zone (you can create Public hosted zone if you have a domain)
   ```sh
   On AWS console go to Services --> Route53 (it is under netwroking) --> hosted zones --> created hosted zone  (click on Get started now)
   Domain Name: valaxy.net
   Type: Private hosted zone for Amazon VPC. Make sure you are chosing right VPC if you have multiple
   VPC ID: choose the region we are using which is ca-central-1 (you can see the region on AWS console, top-right corner)
   The click on Create
   ```

1. create an S3 bucket
   ```sh
    # On AWS go to Services --> S3  (present under Storage) --> Create bucket, or instead of creating the bucket from AWS you can create it from the CLI using the command below:
    aws s3 mb s3://demo.k8s.valaxy.net
    # If you refresh your AWS console, you'll see the s3 bucket created.
   ```
1. Expose environment variable:
   ```sh
    # We need to export the variable so that kps can use it.
    # Run the command below from the CLI:
    export KOPS_STATE_STORE=s3://demo.k8s.valaxy.net
   ```

1. Create sshkeys before creating cluster
   ```sh
   # This key pair is needed to login to our kubernetes cluster.
   # From CLI run the command below:
    ssh-keygen
    # The key is generated under .ssh directory:
    cd .ssh/
    ls -l
    # You'll see key files: "id_rsa" and "id_rsa.pub"
   ```

1. Create kubernetes cluster definitions on S3 bucket
   ```sh
   kops create cluster --cloud=aws --zones=ca-central-1 --name=demo.k8s.valaxy.net --dns-zone=valaxy.net --dns private 
   # --name is the name of our cluser. We'll give it the same name as the name of the bucket we created.
   # Note: we do not store our devops environemnt and target environment in same VPCs, due to security reasons. So if you see once the cluser is created, in the config (output on the screen), it'll setup a new cliuster. 
    ```

1. Commands
   ```sh
   To list clusters: kops get cluster
   To edit this cluster: kops edit cluster demo.k8s.valaxy.net
   To edit your node instance group: kops edit ig --name=demo.k8s.valaxy.net nodes
   To edit your master instance group: kops edit ig --name=demo.k8s.valaxy.net master-ca-central-1b
   ```
   
1. Create kubernetes cluser
    ```sh
    kops update cluster demo.k8s.valaxy.net --yes
    ```
    Once the cluster is created, you can see in the AWS console: new files get created under the S3 bucket, <br/>
    2 new IAM roles get created: "masters.demo.k8s.valaxy.net" and "nodes.demo.k8s.valaxy.net"  <br/>
    Additional routes get created under Route53 <br/>
    If you got to EC2 Instances: you see 2 new node.demo instances got created and 1 master instance  <br/>
    Autoscaling: Launch Configuration --> you'll notice 2 new configurations created (nodes and master), this is in case if any system goes down, it'll spin up a new one automatically. <br/>
    Autoscaling: Autoscaling --> autoscaling group also gets created.

1. Commands
   ```sh
   Validate cluster: kops validate cluster   --> this command will fail until the cluster is created (it takes 5-10 min for it to be created)
   List nodes: kubectl get nodes --show-labels
   SSH to the master: ssh -i ~/.ssh/id_rsa admin@api.demo.k8s.valaxy.net
   This "api.demo.k8s.valaxy.net" is our DNS name. We can also use instead the public ip of our master server (under EC2 instances, check the public of the master instance created. ALways login as admin user, never login as root user.
   You can manage your cluster from the CLI because it is an ubuntu system and we installed kubctl commands. On CLI type "kubctl" to see the list of commands.
   ```
   Rather than managing our system from the ubuntu system we created "k8s-management-server", we'll connect to the master EC2 instance, but we'll need to install kubctl on that instance aswell.
   
1. To change the kubernetes master and worker instance sizes 
   ```sh 
   kops edit ig --name=<cluster_name> nodes
   #kops edit ig --name=demo.k8s.valaxy.net nodes 
   kops edit ig --name=<cluster_name> master-<zone_name>
   #kops edit ig --name=demo.k8s.valaxy.net master-ap-south-1b
   ```
1. to Delete cluster (try once your lab is done)
   ```sh 
   kops delete cluster <cluster_name> --yes
   ```
1. Validate your cluster
     ```sh
      kops validate cluster
    ```

1. To list nodes
   ```sh
   kubectl get nodes
   ```

   
#### Deploying Nginx pods on Kubernetes
1. Deploying Nginx Container
    ```sh
    kubectl run --generator=run-pod/v1 sample-nginx --image=nginx --replicas=2 --port=80
    #kubectl run sample-nginx --image=nginx --replicas=2 --port=80
    # kubectl run simple-devops-project --image=yankils/simple-devops-image --replicas=2 --port=8080
    kubectl get pods
    kubectl get deployments
   ```

1. Expose the deployment as service. This will create an ELB in front of those 2 containers and allow us to publicly access them.
   ```sh
   kubectl expose deployment sample-nginx --port=80 --type=LoadBalancer
   # kubectl expose deployment simple-devops-project --port=8080 --type=LoadBalancer
   kubectl get services -o wide
   ```
