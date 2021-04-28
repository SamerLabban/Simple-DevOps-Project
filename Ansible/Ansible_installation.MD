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
   Troubleshooting: <br/>
   If you are unable to install ansible with "yum install ansible" or "pip install ansible", Please make sure you have installed python first.
   then try below command. <br/>
   sudo amazon-linux-extras install ansible2
   
1. Create Ansible directory
   ```sh
   mkdir /etc/ansible
   ```

1. Create a user called ansadmin (on Control node and Managed host)  
   ```sh
   useradd ansadmin
   passwd ansadmin
   ```
1. Below command grants sudo access to ansadmin user. But we strongly recommended using "visudo" command if you are aware vi or nano editor.  (on Control node and Managed host)
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
   
1. Ansible server is used to create images and store them on docker registry. Hence install docker, start docker services and add ansadmin to the docker group. 
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
   vi /etc/ssh/sshd_config
   # Change PasswrodAuthentication to yes
   PasswordAuhthentication yes
   # Save the file, then reload it:
   service sshd reload
   ```
   
1. Switch back to root user. Change hostname to make it easier to identify:
   ```sh
   hostname ansible-control-node
   ```
   Exit and re-impersonnate as root to see the change.
   
1. Log in as a ansadmin user on master and generate ssh key (on Control node)
   ```sh 
   sudo su - ansadmin
   ssh-keygen
   ```
   Keys are usually generated under .ssh folder:
   cd .ssh  <br/>
   You'll see files "id_rsa" --> private key,  and "id_rsa.pub" --> public key   <br/>
   We'll need to copy the public key and keep the private key secure. If someone has access to the private key, they'll be able to access your system.
   
1. Switch to docker-host (not ansible-host you are now on):
   Create user ansadmin and set password
   ```sh
   useradd ansadmin
   passwd ansadmin
   ```
   Password Authntication is already enabled on this host, so no issue to login. <br/>
   Copy the keys to our docker host system. <br/>
   Do "ip addr" and copy private key. You'll need this key when copying keys from ansible host in following step. <br/>
   Docker host will communicate with ansible host using private key, since both are in same VPC.

1. From Ansible Host, Copy keys onto all ansible managed hosts (on Control node)
   ```sh 
   su - ansadmin
   ssh-copy-id ansadmin@<target-server>
   ```
   You'll need to pass the password the first time you run the ssh copy command, to copy the private keys. <br/>
   But the second time you ssh to docker host from the ansible host, it won't ask for the password: <br/>
   ssh ansadmin@<target-server>

### Validation test

1. Test the connectivity by doing a ping test
   From the ansible host:
   ```sh
   su - ansadmin
   cd /etc/ansible/
   ls # you'll see that hosts file does not exist
   # create an inventory file called "hosts" add control node and managed hosts IP addresses to it.
   sudo vi hosts
   # Add the following lines
   <private ip of docker host> # example: 172.31.18.245
   localhost  # this is my local ansible host
   # Save the file
   ```
   Do a ping test by running the following command:  ansible all -m ping   <br/>
   Above command will run ping module on hosts file. <br/>
   You'll see that it is able to ping remoe host (docker host). It fails to ping local host because we did not copy keys to local host. <br/>
   So copy keys to local host by running: <br/>
   ssh-copy-id localhost  <br/>
   Run the ping command again:  ansible all -m ping   <br/>
   You'll see that it successfully pings both systems.

1. Run ansible command as ansadmin user it should be successful (Master)
   ```sh 
   ansible all -m ping
   ```
