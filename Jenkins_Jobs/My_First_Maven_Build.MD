# Create a First Maven Jenkins job to build hello-world project 
# *Jenkins Job name:* `My_First_Maven_Build`

We know how to use work with each and Git, Jenkins independently. What if you want to collaborate these two? that is where Simple DevOps project helps you. Follow the below steps if you are a new guy to DevOps. You love it. 


#### Pre-requisites

1. Jenkins server 


### Steps to create "My_First_Maven_Build" Jenkin job
1. Login to Jenkins console
1. Create *Jenkins job*, Fill the following details,
   - *Source Code Management:*
      - Repository: `https://github.com/yankils/hello-world.git`
      - Branches to build : `*/master`  
   - *Build:*
     - Root POM:`pom.xml`
     - Goals and options: `clean install package`

### Note
When the maven job runs in jenkins, you can check the Workspace folder and see the files from the git repo copied there, and the generated files will be in the workspace in folder webapp/target.  <br />
You can also access the files on the VM, the jenkins files will be located at: cd /var/lib/jenkins/workspace/   <br />

To find file context.xml in linux you do:  find / -name context.xml
