# Deploy on a docker container using Ansible
# *Jenkins Job name:* `Deploy_on_Container_using_ansible`

### Pre-requisites

1. Jenkins server 
1. Docker-host server 
1. Ansible server
1. `Dockerfile` under *`/opt/docker`* on Ansible server **[Get Help Here]()**
   ```sh 
   # Pull tomcat latest image from dockerhub 
   From tomcat
   # Maintainer
   MAINTAINER "AR Shankar" 

   # copy war file on to container 
   COPY ./webapp.war /usr/local/tomcat/webapps
   
1. Create Ansible Playbook to create docker image using dockerfile and webapp.war file
   Create `create-docker-image.yml` unser *`/opt/docker`* on Ansible server **[Get Help Here]()**  <br/>
   ```sh
   ---
   - hosts: all
     #ansadmin doesn't need root access to create an image, but you can give it root access with the command below:
     become: true 

     tasks:
     - name: build docker image using war file
       command: "docker build -t simple-devops-image ." 
       # specify on which directory you want to execute this command. chdir is change directory
       args:
         chdir: /opt/docker
   ```
   You can also create above file as shown in option 2 in file: Jenkins_Jobs/create-docker-image.yml 

1. Execute above Ansible playbook
   Before executing the playbook, you need to create "hosts" file, and add into it the hosts you want to run the playbook on (where the image will be created). <br/>
   cat /etc/ansible/hosts  <br/>
   This will shows the hosts we added in the file we creted before. <br/>
   Create a new file in /opt/docker folder, where you created the ansible playbook:  <br/>
   vi hosts   <br/>
   Add the follwoing line into it to create the image on ansible host only for now (later we specify other target hosts):  localhost   <br/>
   Execute ansible playbook: <br/>
   ansible-playbook -i hosts simple-devops-image.yml --check   <br/>
   Note: "-check" only shows you how the playbook is going to run. It does not actually execute it or create the image. It just checks if everything is fine. <br/>
   Rerun the command again without the check: <br/>
   ansible-playbook -i hosts simple-devops-image.yml   <br/>
   Now if you do "docker images", you'll see the image created. <br/>
   However we need to run the above playbook using Jenkins.

1. Add the task below to the above Ansible playbook. This task is to create the container. (Only copy the part where you ahve name and command)
   Create `create-docker-image.yml` under *`/opt/docker`* on Ansible server **[Get Help Here]()**
   ```ssh
   ---
   - hosts: all
     become: ture

     tasks:
     - name: create container using simple-devops-image
       command: docker run -d --name simple-devops-container -p 8080:8080 simple-devops-image
   ```
   Rename the file create-docker-image.yml to create-docker-project.yml, since now we are not just creating the image, but we are running it also: <br/>
   mv create-docker-image.yml create-docker-project.yml <br/>
   Run the ansible playbook: <br/>
   ansible-playbook -i hosts simple-devops-project.yml   <br/>
   Now if you do "docker images", you'll see the image created. And if you do "docker container ps", you'll see the container created and running.

### Integration between Ansible-control-node and Jenkins

Install "publish Over SSH"
 - `Manage Jenkins` > `Manage Plugins` > `Available` > `Publish over SSH`

Enable connection between Ansible-control-node and Jenkins

- `Manage Jenkins` > `Configure System` > `Publish Over SSH` > `SSH Servers` 

	- SSH Servers:
                - Name: `ansible-server`
		- Hostname:`<ServerIP>`
		- username: `ansadmin`
               
       -  `Advanced` > chose `Use password authentication, or use a different key`
		 - password: `*******`
 
### Steps to create "Deploy_on_Container_using_ansible" Jenkin job
#### From Jenkins home page select "New Item"
   - Enter an item name: `Deploy_on_Container_using_ansible`
     - Copy from: `Deploy_on_Container`
     
   - *Source Code Management:*
      - Repository: `https://github.com/yankils/hello-world.git`
      - Branches to build : `*/master`  
   - *Poll SCM* :      - `* * * *`

   - *Build:*
     - Root POM:`pom.xml`
     - Goals and options: `clean install package`

 - *Post-build Actions*
   - Send build artifacts over SSH
     - *SSH Publishers*
      - SSH Server Name: `ansible-server`
       - `Transfers` >  `Transfer set`
            - Source files: `webapp/target/*.war`
	       - Remove prefix: `webapp/target`
	       - Instead of using home directory of user, we'll create a new directory: <br/>
		 On Ansible host: sudo su -   (either impersonate as root, or run sudo before below commands) <br/>
		                  cd /opt   <br/>
		                  sudo mkdir docker  <br/>
				  # Give sudo access for ansadmin on this directory, so if we are copying anything from Jenkins it does not give permission issues:  <br/>
				  sudo chown -R ansadmin:ansadmin /opt/docker    <br/>
				  # So now the folder "docker" is owned by ansadmin  <br/> 
				  # We'll use this directory to store our artifacts.  <br/>
	         Remote directory: `//opt//docker`   <br/>
	         We put double slash above because it does not take special characters.   <br/>
	       - After createing this job and before adding exec command, you can build the job and see if it is copying the artifacts to docker directory on ansible host.
	       - Exec command: 
                ```sh 
                ansible-playbook -i /opt/docker/hosts /opt/docker/create-docker-image.yml;
                ```

Save and run the job now.
