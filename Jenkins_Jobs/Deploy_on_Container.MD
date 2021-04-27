# Deploy on a docker container using Jenkins 
# `Deploy_on_Container`

### Pre-requisites

1. Jenkins server 
1. Docker-host server  <br/>
   Start docker service on docker host: <br/>
       Impersonnate as root: sudo su -u  <br/>
       Start docker service: service docker start
3. `Dockerfile` under *`/home/dockeradmin`* in user home directory on docker host <br/>
    Impersonnate as dockeradmin first before creating the dockerfile under path *`/home/dockeradmin`*: su - dockeradmin 
   ```sh 
   # Pull tomcat latest image from dockerhub 
   From tomcat
   # Maintainer
   MAINTAINER "AR Shankar" 

   # copy war file on to container 
   COPY ./webapp.war /usr/local/tomcat/webapps
### Integration between Docker-host and Jenkins

Install "publish Over SSH"
 - `Manage Jenkins` > `Manage Plugins` > `Available` > `Publish over SSH`

Enable connection between Docker-host and Jenkins

- `Manage Jenkins` > `Configure System` > `Publish Over SSH` > `SSH Servers` 

	- SSH Servers:
                - Name: `docker-host`
		- Hostname:`<ServerIP>`
		- username: `dockeradmin`
               
       -  `Advanced` > chose `Use password authentication, or use a different key`
		 - password: `*******`
 
### Steps to create "Deploy_on_Container" Jenkin job
#### From Jenkins home page select "New Item"
   - Enter an item name: `Deploy_on_Container`
     - Copy from: `Deploy_on_Docker_Host`
     
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
      - SSH Server Name: `docker-host`
       - `Transfers` >  `Transfer set`
            - Source files: `webapp/target/*.war`
	       - Remove prefix: `webapp/target`
	       - Remote directory: `//home//ansadmin` or `.`
	       - Exec command: 
                ```sh 
                cd /home/dockeradmin;
                docker build -t simple-devops-image .; 
                docker run -d --name simple-devops-container -p 8080:8080 simple-devops-image;
                ```

Save and run the job now. <br/>
Open webpage page in browser by going to: <Public_IP>:8080/webapp  <br/>
The following will access the apache default page: <Public_IP>:8080

### Note
If you make change to the code and want to make another deployment, you'll get a message saying that container name already in use. So you cannot deploy again using same container name. <br/>
Jenkins cannot manage deployments properly. For that we need to use available deployment tools, such as Ansible. <br/>
Another issue with Jenkins, is that in our case we deployed to a dev host that we already registered. But imagine you have 100 systems where you want to deploy to, so you need to register all these with Jenkins which is hard to achieve. Therefore, we'll use Jenkins only as a build tool and we'll use Jenkins as a deployment tool.

### Troubleshooting: Unable to push docker image
If you are unable to push docker image it means that you haven't logged into dockerhub as a user which you are trying to push. <br/>
During my course, I ran playbook as a "ansadmin" user (because there is a typo in the "become: true" so it didn't execute as root user). If you are executing as a root user then you should log in to DockerHub as a root user. <br/>
In short, you can do either one of the below  <br/>
1. remove or comment "become: true" in the playbook  or <br/>
2. Login to dockerhub as a root user
