# Integrating Kubernetes cluster with Ansible

1. Login to ansible server using putty or mobaxterm, and copy public key onto kubernetes cluseter master account. We need to do this so that the ansible can do the passwordless authentication with the kubernetes master.
   ```sh
   # On ansible host, impersonate as ansadmin user
   sudo su - ansadmin
   cd .ssh/
   ls
   cat id_rsa.pub
   # Copy the output of the above command
   # Switch to the Kubernetes host
   # Impersonate as root
   sudo su -
   cd ~/.ssh
   ls
   # You'll see file "authorized_keys"
   vi authorized_keys
   # do shift + g + o  to go the end of the file. Then add the copied key at the end.
   # If the copy/paste does not work, just use the append command:
   cat >> authorized_keys
   # Press enter and on the new line paste the copied file. 
   # Verify that it copied by doing cat authorized_keys.
   # We can do all the above copying aswell by an SSH command from ansible host. 
   ```
   So now from anisble system, I can login to kubernetes master as root user without using password.
   ```sh
   ssh -i ~/.ssh/id_rsa root@<kubernetes_host_public_ip>
   ```
   You can not use private ip to ssh because ansible and kubernetes hosts are on different VPCs. Else you need to enable vpc tunneling.

1. Update hosts file with the new group called kubernetes by adding the kubernetes master ip address in that. 
   ```sh
   # On asnible host, impersonate as ansadmin
   sudo su - ansadmin
   cd /opt
   ls  # docker file you see there is the file we were working on before.
   sudo mkdir kubernetes
   cd kubernetes
   sudo vi hosts
   # Add the following into the file.
   [ansible-server]   --> we are doing grouping here. Grouping the ips here into group ansible-server. All ips under one group can be accessed using the group name.
   localhost --> this is local ansible server
   
   [kubernetes]
   <kubernetes_public_ip>  --> copy this from aws instance
   # save the hosts file.
   ```

1. Create ansible playbooks to create **[deployment](https://github.com/SamerLabban/Simple-DevOps-Project/blob/master/Kubernetes/kubernetes-valaxy-deployment.yml)** and **[services](https://github.com/SamerLabban/Simple-DevOps-Project/blob/master/Kubernetes/kubernetes-valaxy-service.yml)** 
   ```sh
   # On ansible host, impersonate as ansadmin user.
   cd /opt/kubernetes
   sudo vi kubernetes-valaxy-deployment.yml   --> paste the content from the above link.
   sudo vi kubernetes-valaxy-service.yml   --> paste the content from the above link.
   ```
    
1.  Check for pods, deployments and services on kubernetes master
    ```sh 
    kubectl get pods -o wide 
    kubectl get deploy -o wide
    kubectl get service -o wide
    ```
	
1. Access application suing service IP
   ```sh
   wget <kubernetes-Master-IP>:31200
   ```
