## Create Deployment and Service using kubectl commands

### Deployments
- Connect to your master server.
- Impersonate as root: sudo - u
- Under /root, create the 2 files below.
- Create yml file valaxy-deploy.yml and copy the content from "Simple-DevOps-Project/Kubernetes/valaxy-deploy.yml"
- Create yml file valaxy-service.yml and copy the content from "Simple-DevOps-Project/Kubernetes/valaxy-service.yml"
- Create the deployment by running (this will apply any instructions in the file "-f valaxy-deploy.yml"): kubectl apply -f valaxy-deploy.yml
- Check the deployments: kubectl get deployments.
- If an old deployment exists and you wish to delete it, you can do that by running: 
- ```sh 
  kubectl delete deployment <deployment name>
  ```

### Services
- To check existing services, do: kubectl get services
- To delete services we created before, you do: 
  ```sh
  kubectl delete service <service name>
  Ex: kubectl delete service sample-nginx
  ```
- To create service:
  ```sh
  # kubectl apply -f <service name>
  kubectl apply -f valaxy-service.yml
  ```
- Check if the service is created, do: kubectl get services
- The 2 pods should be created, check by running: kubectl get pods   (the 2 pods should be running on 2 different systems to make the application highly available).

### Accessing the application
- Open the port defined in the services file (in our case it is 31200) in the security group in AWS. <br/>
  Add rule: Custom TCP Rule - Port: 32100 - Source: 0.0.0.0/0
- On the browser enter: <master_instance_public_or_dns_name>:32100  --> you'll be able to access the default tomcat page  <br/>
                        <master_instance_public_or_dns_name>:32100/webapp --> you'll access the html file.

Note: Whenever there is an update in the code, it should create a new image, and with the new image we should create our new deployment. FOr that we'll write our ansible playbook.
