# Install Jenkins on AWS EC2
Jenkins is a self-contained Java-based program, ready to run out-of-the-box, with packages for Windows, Mac OS X and other Unix-like operating systems. As an extensible automation server, Jenkins can be used as a simple CI server or turned into the continuous delivery hub for any project.


### Prerequisites
1. EC2 Instance 
   - With Internet Access
   - Security Group with Port `8080` open for internet
1. Java v1.8.x 

## AWS EC2 Instance:
- On AWS launch an Amazon Linux instance.
- Keep everything as default, just edit the folowing:
```
Tag:  Name : Jenkins-Server
Security group: Create new security group, with the following Inbound rules:
Port range   Protocol   Source          Security groups
8080	     TCP	0.0.0.0/0	DevOps_Project_SG
8080	     TCP	::/0	        DevOps_Project_SG
22	     TCP	0.0.0.0/0	DevOps_Project_SG  --> Needed to SSH  to the VM

Outbound rules:
Port range   Protocol   Destination     Security groups
All	     All	0.0.0.0/0	DevOps_Project_SG
```

## Connect to EC2 Instance:
```
SSH to your Instance using putty or MobaXterm (pem key file needed for both).
IP address: Use public IP of EC2 instance.
AWS: User: ec2-user

Change hostname to jenkins, to make it easier to identify:
In linux shell, type: hostname jenkins
[root@ip-172-31-13-3 ~]# hostname jenkins
[root@ip-172-31-13-3 ~]#
[root@ip-172-31-13-3 ~]# exit
logout
[ec2-user@ip-172-31-13-3 ~]$ sudo su -
```

## Install Java
To become root: sudo su -  <br />
We need root priviledges to do installation <br />

To check java version: java -version  <br />
To remove java 1.7 that comes installed on AWS instance, use:  yum remove java-1.7.0*  <br />
To install java 1.8, do: yum install java-1.8*   <br />

1. We will be using open java for our demo, Get the latest version from http://openjdk.java.net/install/
   ```sh
   yum install java-1.8*
   #yum -y install java-1.8.0-openjdk-devel
   ```

2. Confirm Java Version and set the java home
   ```sh
   java -version
   # run this command to find where your java jar file is available on your VM
   find /usr/lib/jvm/java-1.8* | head -n 3
   # Output:
   # /usr/lib/jvm/java-1.8.0
   # /usr/lib/jvm/java-1.8.0-openjdk
   # /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.282.b08-1.amzn2.0.1.x86_64  --> this is the actual path of the jre
   
   # So we need to set the home path for the jre under the home directory to be:
   # /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.282.b08-1.amzn2.0.1.x86_64/jre
   JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-<Java version which seen in the above output>	
   export JAVA_HOME
   PATH=$PATH:$JAVA_HOME
   # To set it permanently update your .bash_profile
   vi ~/.bash_profile
   
   # Go to the home directory: cd ~
   # vi .bash_profile  --> under this we provide the java home path
   # Edit the file to add specified line:
    .bash_profile
     Get the aliases and functions
     if [ -f ~/.bashrc ]; then
        . ~/.bashrc
     fi
     # User specific environment and startup programs
     JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.282.b08-1.amzn2.0.1.x86_64  --> added line
     PATH=$PATH:$HOME/bin:$JAVA_HOME   --> we add :$JAVA_HOME to this line
     export PATH
    # save the file.
    
    # If you do now: echo $JAVA_HOME --> it will not give you the new path. You need to logout and login to get the latest path.
    # So do: exit
    # The log again as root: sudo su -
    # Now do: echo $JAVA_HOME to see the latest path.

   ```
   _The output should be something like this,_
    ```sh
   [root@~]# java -version
   openjdk version "1.8.0_151"
   OpenJDK Runtime Environment (build 1.8.0_151-b12)
   OpenJDK 64-Bit Server VM (build 25.151-b12, mixed mode)
   ```
 
## Install Jenkins
Now to install Jenkins, on the linux instance, if you do: yum install jenkins --> this will not work because it does not have the jenkins reporsitories. <br />
To download jenkins, in your browser search for "jenkins download". --> go to first download link --> You'll see "Long-term Support" (this is stable version)--> click on "Redhat/Fedora/CentOS"  <br />
You'll see than the commands to use to set the Jenkins repository in our server. <br />

You can install jenkins using the rpm or by setting up the repo. We will set up the repo so that we can update it easily in the future.
1. Get the latest version of jenkins from https://pkg.jenkins.io/redhat-stable/ and install
   ```sh
   yum -y install wget
   sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
   sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
   yum -y install jenkins
   ```

   ### Start Jenkins
   ```sh
   # Check status of jenkins instance, whether started or stopped
   service jenkins status 

   # Start jenkins service
   service jenkins start

   # Setup Jenkins to start at boot,
   chkconfig jenkins on
   ```

   ### Accessing Jenkins
   By default jenkins runs at port `8080`, You can access jenkins at
   ```sh
   http://YOUR-SERVER-PUBLIC-IP:8080
   ```   
   
  #### Configure Jenkins
- The default Username is `admin`
- Grab the default password
- Password Location:`/var/lib/jenkins/secrets/initialAdminPassword`
  copy the password by running: cat /var/lib/jenkins/secrets/initialAdminPassword
- `Skip` Plugin Installation; _We can do it later_
- Change admin password
   - `Admin` > `Configure` > `Password`
- Configure `java` path
  - `Manage Jenkins` > `Global Tool Configuration` > `JDK`  
     Name: JAVA_HOME
    /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.282.b08-1.amzn2.0.1.x86_64
- Create another admin user id

### Test Jenkins Jobs
1. Create “new item”
1. Enter an item name – `My-First-Project`
   - Chose `Freestyle` project
1. Under the Build section
	Execute shell: echo "Welcome to Jenkins Demo"
1. Save your job 
1. Build job
1. Check "console output"

