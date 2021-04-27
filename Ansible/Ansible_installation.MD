# Ansible Installation

Ansible is an open-source automation platform. It is very, very simple to set up and yet powerful. Ansible can help you with configuration management, application deployment, task automation.

### Pre-requisites

1. An AWS EC2 instance (on Control node) <br/>
   Leave everything as default, and just specify the following:  <br/>
   Tag: Name: Ansible_Server <br/>
   Select an existing securit group (previously created group): DevOps_Project_SG

### Installation steps:
#### on Amazon EC2 instance
SSH into created instance and install the following (impersonnate as root first: sudo su -

1. Install python and python-pip
   ```sh
   yum install python
   yum install python-pip
   ```
1. Install ansible using pip check for version
    ```sh
    pip install ansible
   ansible --version
   ```
   
1. Create Ansible directory
   ```sh
   mkdir /etc/ansible
   ```

1. Create a user called ansadmin (on Control node and Managed host)  
   ```sh
   useradd ansadmin
   passwd ansadmin
   ```
1. Below command grant sudo access to ansadmin user. But we strongly recommended using "visudo" command if you are aware vi or nano editor.  (on Control node and Managed host)
   ```sh
   echo "ansadmin ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
   ```
   or use visudo command as below:
   ```sh
   visudo
   # do shift G to go the end of the file. Then do o to go into insert mode. Add the following line:
   ansadmin ALL=(ALL)      NOPASSWD: ALL
   # So now it'll not ask for the password when you login as a root.
   ```
   
1. Ansible server used to create images and store on docker registry. Hence install docker, start docker services and add ansadmin to the docker group. 
   ```sh
   yum install docker
   
   # start docker services 
   service docker status
   service docker start 
   
   # add user to docker group 
   usermod -aG docker ansadmin
   ```   
   
1. By default user authentication is not enabled in AWS instance. So enable it:
   ```sh
   vi /etc/shh/sshd_config
   # Change PasswrodAuthentication to yes
   PasswordAuhthentication yes
   # Save the file, then reload it:
   service sshd reload
   ```

1. Log in as a ansadmin user on master and generate ssh key (on Control node)
   ```sh 
   sudo su - ansadmin
   ssh-keygen
   ```
   Keys are usually generated under .ssh folder:
   ```sh
   cd .ssh
   # You'll see files "id_rsa" --> private key,  and "id_rsa.pub" --> public key
   # We'll need to copy the public key and keep the private key secure. If someone has access to the private key, they'll be able to access your system.
   ```
   
1. Copy keys onto all ansible managed hosts (on Control node)
   ```sh 
   ssh-copy-id ansadmin@<target-server>
   ```
   
1. Switch back to root user. Change hostname to make it easier to identify:
   ```sh
   hostname ansible-control-node
   ```
   Exit and re-impersonnate as root to see the change.

1. Create a directory /etc/ansible (done in previous steps) and create an inventory file called "hosts" add control node and managed hosts IP addresses to it. 
 
### Validation test

   
1. Run ansible command as ansadmin user it should be successful (Master)
   ```sh 
   ansible all -m ping
   ```
