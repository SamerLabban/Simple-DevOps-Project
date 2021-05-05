## Create Deployment and Service using kubectl commands

- Connect to your kubernetes ec2 instance.
- Impersonate as root: sudo - u
- Create yml file valaxy-deploy.yml and copy the content from "Simple-DevOps-Project/Kubernetes/valaxy-deploy.yml"
- Create yml file valaxy-service.yml and copy the content from "Simple-DevOps-Project/Kubernetes/valaxy-service.yml"
- Create the deployment by running (this will apply any instructions in the file "-f valaxy-deploy.yml"): kubectl apply -f valaxy-deploy.yml
- Check the deployments: kubectl get deployments.
- If an old deployment exists and you wish to delete it, you can do that by running: 
- ```sh 
  kubectl delete deployment <deployment name>
  ```
-
