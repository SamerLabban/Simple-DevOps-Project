## Integration Kubernetes with Jenkins

# *Jenkins CI Job:* `Deploy_on_Kubernetes-CI`

### Pre-requisites

1. Jenkins server 
1. Ansible server
1. Kubernetes cluster
 
### Steps to create "Deploy_on_Kubernetes_CI" Jenkin job
#### From Jenkins home page select "New Item"
   - Enter an item name: `Deploy_on_Kubernetes_CI`
     - Copy from: `Deploy_on_Docker_Container_using_Ansible_playbooks`
     
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
	       - # Remote directory: `//opt//docker`
	         Remote directory: `//opt//kubernetes`  --> change to this to copy the war file to kubernetes folder
	       - Exec command: 
                ```sh 
                # ansible-playbook -i /opt/docker/hosts /opt/docker/create-simple-devops-image.yml --limit localhost;
		ansible-playbook -i /opt/docker/hosts /opt/kubernetes/create-simple-devops-image.yml; --> "--limit localhost is not needed anymore, because we already specified that in our hosts file as explained below.
                ```

Save and run the job. <br/>
Note: The difference between job "Deploy_on_Docker_Container_using_Ansible_playbooks" and the new job "Deploy_on_Kubernetes_CI", is the before we used to created the image and run it in docker container. But now we'll just use this job to create the image and push it to docker hub, and then the job below "Deploy_on_Kubernetes_CD" will deploy and run the image on kubernetes cluser. <br/>
So before you run the job, you need to copy the "Dockerfile" and the create image yaml file, from "/opt/docker" to "opt/kubernetes".
```sh
Impersonnate as ansadmin user on ansible host.
cd opt/docker/
sudo mv Dockerfile create-simple-devops-image.yml /opt/kubernetes
cd /opt/kubernetes/
vi create-simple-devops-image.yml
# Edit the following line "chdir: /opt/docker" to create the image under kubernetes folder.
# chdir: /opt/kubernetes

# If you run, you'll see that the kubernetes folder is owned by root user.
ls -ld /opt/kubernetes/
# But from Jenkins we are trying to login to ansible server as "ansadmin" user.
# You can see that from Jenkins --> Manage Jenkins --> Configure System" --> SSH Servers: ansible-server --> Name: ansadmin
# So we'll need to give the priviledge for /opt/kuberneets" to ansadmin user
# So we do:
sudo chown -R ansadmin:ansadmin /opt/kubernetes/
# Now if you do the following you'll see that the folder is owned by ansadmin:
ls -ld /opt/kubernetes/
# And the files and folders under it, are owned by answadmin
ls -l /opt/kubernetes/
# Also in the image file, you can change "hosts: all" to "hosts: ansible-server"
# So edit the docker file create-simple-devops-image.yml and chnage the hosts. This way it'll only refer to the group "ansible-server" in the hosts file, whick has "localhost" under it.
```

# *Jenkins CD Job:* `Deploy_on_Kubernetes-CD`

### Steps to create "Deploy_on_Kubernetes_CD" Jenkin job
#### From Jenkins home page select "New Item"
   - Enter an item name: `Deploy_on_Kubernetes_CD`
     - Freestyle Project
	 
  - *Post-build Actions*  
    - Send build artifacts over SSH  
      - *SSH Publishers*  
	       - Exec command: 
                ```sh 
                ansible-playbook -i /opt/docker/hosts /opt/docker/kubernetes-valaxy-deployment.yml;
                ansible-playbook -i /opt/docker/hosts /opt/docker/kubernetes-valaxy-service.yml;
                ```
Save and run the job.
