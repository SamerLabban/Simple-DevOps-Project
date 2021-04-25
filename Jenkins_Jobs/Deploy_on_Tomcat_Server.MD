# Deploy on a Tomcat server
# *Jenkins Job name:* `Deploy_on_Tomcat_Server`

### Pre-requisites

1. Jenkins server 
2. Tomcat Server 

### Create Tomcat server
- On AWS launch an Amazon Linux instance.
- Keep everything as default, just edit the folowing:  <br/>
Tag:  Name : Tomcat_Server  <br/>
Security group: Create new security group (or use DevOps_Project_SG creating when setting up Jenkins server), with the following Inbound rules:  <br/>
|Port range  |Protocol  |Source         |Security groups |
|8080	       |TCP      	|0.0.0.0/0	    |DevOps_Project_SG |
|8080	       |TCP      	|::/0	          |DevOps_Project_SG |
|22	         |TCP	      |0.0.0.0/0	    |DevOps_Project_SG  --> Needed to SSH  to the VM  |

Outbound rules:  <br/>
|Port range   |Protocol  |Destination     |Security groups| 
|All	        |All	     |0.0.0.0/0	      |DevOps_Project_SG|

### Adding Deployment steps

1. Install 'deploy to container' plugin. This plugin needs to deploy on tomcat server. 

  - Install 'deploy to container' plugin without restart  
    - `Manage Jenkins` > `Jenkins Plugins` > `available` > `deploy to container`
 
2. Jenkins should need access to the tomcat server to deploy build artifacts. setup credentials to enable this process. use credentials option on Jenkins home page.

- setup credentials
  - `credentials` > `jenkins` > `Global credentials` > `add credentials`
    - Username	: `deployer`
    - Password : `deployer`
    - id      :  `deployer`
    - Description: `user to deploy on tomcat vm`

### Steps to create "Deploy_on_Tomcat_Server" Jenkin job
 #### From Jenkins home page select "New Item"
   - Enter an item name: `Deploy_on_Tomcat_Server`
     - Copy from: `My_First_Maven_Build`
     
   - *Source Code Management:*
      - Repository: `https://github.com/yankils/hello-world.git`
      - Branches to build : `*/master`  
   - *Poll SCM* :      - `* * * *`  <br/>
     You can have the job run everytime you commit to git repo. For that in your jenkins job, you need to check "Poll SCM" under "Build Triggers", and then you the specify schedule as ***** --> which means poll scm every minute every hour every day every week every month

   - *Build:*
     - Root POM:`pom.xml`
     - Goals and options: `clean install package`

 - *Post-build Actions*
   - Deploy war/ear to container
      - When deploying war file to tomcat container, you specify to deploy the war file, the file is located under folder having same name as the jenkins job. You can also specify "**/*.war", which will pick any war file under the job's workspace.
      - WAR/EAR files : `**/*.war`
      - Containers : `Tomcat 8.x`
         - Credentials: `deployer` (user created on above)
         - Tomcat URL : `http://<PUBLIC_IP>:8080`

Save and run the job now.

To deploy on tomcat server, tomcat should accept jenkins authentication.

The default directoy for tomcat app is "/opt/tomcat/webapps", so whenever we run the job, it'll copy war files to that webapp location. <br/>
```
[root@tomcat ~]# cd /opt/tomcat/webapps/
[root@tomcat webapps]# ll
total 8
drwxr-x--- 15 root root 4096 Apr 18 02:45 docs
drwxr-x---  7 root root   99 Apr 18 02:45 examples
drwxr-x---  6 root root   79 Apr 18 02:45 host-manager
drwxr-x---  6 root root  114 Apr 18 02:45 manager
drwxr-x---  3 root root  223 Apr 18 02:45 ROOT
drwxr-x---  4 root root   54 Apr 18 04:55 webapp
-rw-r-----  1 root root 2530 Apr 18 04:55 webapp.war
```

To access the tomcat application in the browser, you need to specify the name of the war file (which is webapp in our case): <br/>
http://15.223.121.206:8080/webapp

