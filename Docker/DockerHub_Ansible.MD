## Using Ansible, create docker image, push to docker hub, pull and create container

We need two ansible playbooks: <br/>
1- An ansible playbook that will run on ansible host, and pushes an image to docker hub  <br/>
2- A deployment ansible playbook that will run on all target systems and creates the docker container on docker system.

### Playbook to create docker image <br/>
- On Ansible host impersonate as ansadmin  <br/>
- Create ansible playbook: vi create-simple-ansible-image.yml  <br/>

```sh
---
- hosts: all
  become: ture
  
  tasks:
  - name: create docker image using war file
    command: docker build -t simple-devops-image:latest .
    args:
      chdir: /opt/docker
      
  - name: create tag to image
    command: docker tag simple-devops-image <dockerhub_account_username>/simple-devops-image
    
  - name: push image on to dockerhub
    command: docker push <dockerhub_account_username>/simple-devops-image
    
  - name: remove docker images from ansible server
    command: docker rmi simple-devops-image:latest <dockerhub_account_username>/simple-devops-image
    ignore_errors: yes
```    
    
- Run the ansible playbook: <br/>
  ansible-playbook -i hosts create-simple-devops-image.yml  <br/>
- If a conatiner is running from the image you want to create, stop and remove the container before running the ansible playbook, else image creation will fail. <br/>
  docker rm -f <container id>   This will forcefully remove a running container. 

So after running the playbook, you'll see that it creates an image from war file locally, pushes it to dockerhub, and then deletes the local image.

### Playbook to create docker container <br/>
- On ansible host or target host, create ansible playbook: <br/>
  vi create-simple-devops-project.yml
  ```sh
  ---
  - hosts: all
    become: true
    
    tasks:
    
    - name: stop current running container
      command: docker stop simple-devops-container
      ignore_errors: yes
      
    - name: remove stopped container
      command: docker rm simple-devops-container
      ignore_error: yes
      
    - name: remove docker image
      command: docker rmi <dockerhub_account_username>/simple-devops-image:latest
      ignore_errors: yes
      
    - name: pull docker image from dockerhub
      command: docker pull <dockerhub_account_username>/simple-devops-image:latest
      
    - name: create container using simple-devops-image
      command: docker run -d --name simple-devops-container -p 8080:8080 <dockerhub_account_username>/simple-devops-image:latest
    ```
    
- Run ansible playbook:
  ansible-playbook -i hosts create-simple-devops-project.yml
- If you do "docker images" you see pulled image. If you do "docker ps", you'll see created container.

## Trigger Above playbooks from Jenkins

- First modify hosts file on ansible host to include both "localhost" and private ip address of docker host. Both will be our targets.
- On target host (docker host):
  ```sh
  # Switch to ansadmin to run operations as that user
  su - ansadmin
  # check if ansadmin is part of docker group
  id
  # Add ansadmin to docker group
  exit  # to go back to root user
  usermod -aG docker ansadmin
  su - ansadmin
  id
  ```
- You can limit ansible playbook to run on a specific host, rather than running on all hosts in hosts file. To do that, use the "--limit" option, example: <br/>
ansible-playbook -i hosts create-simple-devops-image.yml --limit localhost  <br/>
You can also restrict to run on a different target host: <br/>
ansible-playbook -i hosts create-simple-devops-image.yml --limit <private_ip_addr_of_target_host>
