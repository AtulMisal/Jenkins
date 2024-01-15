# Project 2 : CI CD Pipeline to Deploy to Kubernetes Cluster Using Jenkins | AWS DevOps Projects For Beginners

# Step 1 : Install and Configure the Jenkins
            1. From Official website install Jenkins on EC2 server.


# Step 2 : Install and Configure the Maven
        1. On same server will install maven.
	    2. For this we need java 8 or above version.
	    3. cd opt/
	    4. wget https://dlcdn.apache.org/maven/maven-3/3.9.6/binaries/apache-maven-3.9.6-bin.tar.gz
	    5. tar -xvf apache-maven-3.9.6-bin.tar.gz
	    6. mv apache-maven-3.9.6-bin.tar.gz maven 
	    7. cd maven/bin/
	    8. ./mvn -v 
	    9. As this maven is not accessible from outside this folder, will add in bash_profile file as 
	   10. cd ~
	   11. ll -a 
	   12. vi .bash_profile
	   13. Add these below lines -
		  M2_HOME=/opt/maven 
		  M2=/opt/maven/bin
		  JAVA_HOME=**
	   14. To find which java version is installed 
                  find / -name java-11*
	   15. Copy this path to **		  
	   16. In below line in that file which starts from path 
                   PATH=$PATH:$HOME/bin:$JAVA_HOME:$M2_HOME:$M2
           17. source .bash_profile	


# Step 3 : Install Maven plugin & Integrate with Jenkins :		   
        1. Install maven plugin.
	    2. Create an Maven project in Jenkins and use the repo https://github.com/Ashfaque-9x/registration-app.git
	    3. By running this project an war file will be created.
	    4. By uploading this war file to server our application will run.
			

# Step 4 : Ansible Server Setup and Ansible Installation
         Use Train with Shubham Channel for Ansible & Node Installation.
	    1. Create an new EC2 server with free tier eligible.
	    2. useradd ansadmin
	    3. passwd ansadmin
	    4. Add this user to visudo file.
	    5. To go to last line in vi file use shift+G
	    6. ansadmin ALL=(ALL) NOPASSWD: All
	    7. vi etc/ssh/sshd_config  Uncomment two lines. Permit Root Login & Pass Authentication
	    8. service sshd reload 
	    9. sudo su - ansadmin
	    10. sudo apt update
        11. sudo apt install software-properties-common
        12. sudo add-apt-repository --yes --update ppa:ansible/ansible
        13. sudo apt install ansible
		   

# Step 5 : Integrating Ansible with Jenkins :
		        1. In Manage Jenkins - System - Publish over SSH - Add - Provide name - Hostname - Username (That you created)
		           step 2 - Check the box Use Password Authentication (Password Step 3) - click on Apply. 
		        2. On Ansible Server create an folder called Docker. mkdir Docker 
		        3. Change permissions as 
		            chown ansadmin:ansadmin Docker
		       4. Create an Jenkins Maven Project - Add git repo - In post build action - Send build 
		          artifact over ssh - source file - *.war - Remote Directory - //Opt//Docker  (// Necessory)
		       5. cd Docker 
           6. ls - The war file will be copied here.		   
		   
		  
# Step 6 : Install and Configure Docker on Ansible Server :
           1. apt Update && apt-get install docker.io -y 
           2. Add ansadmin to docker group 
                 sudo usermod -aG docker ansadmin
           3. sudo init 6
           4. All this configuration are done inside Docker folder created in step 5(2).
           5. Create an Dockerfile and add below content.
                  FROM tomcat:latest
                  RUN cp -R  /usr/local/tomcat/webapps.dist/*  /usr/local/tomcat/webapps
                  COPY ./*.war /usr/local/tomcat/webapps 	
           

# Step 7 : Create ansible playbook to create docker image & copy image to DockerHub :
           1. As we know we have created an Docker directory and created an Dockerfile there.
           2. Will add the ip address of this machine to ansible hosts file. vi /etc/ansible/hosts 
           3. Create an playbook and add data as below 
                ---
                - hosts :ansible
                  tasks :
                    - name: Create docker image 
                      command: docker build -t image_name:latest .
                      args:
					               chdir: /opt/Docker
                     
                    - name: Tag the given image 
                      command: docker tag image_name:latest dockerhubId:image_name:latest 

                    - name: push it to dockerhub 
                      command: docker push dockerhubId/image_name:latest 


# Step 8 : Update Jenkins job to use ansible playbook :
              In jenkins under post build section of the above job add under exec command - ansible-playbook /opt/Docker/playbook.yml


# Step 9 : Setup bootstrap server for eksctl :			  
          # Install AWS Cli on the above EC2 (Refer==https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
               1. curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
               2. unzip awscliv2.zip
               3. sudo ./aws/install
                   OR
               4. sudo yum remove -y aws-cli
               5. pip3 install --user awscli
               6. sudo ln -s $HOME/.local/bin/aws /usr/bin/aws
               7. aws --version	

          # Installing kubectl (Refer===https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)
               1. curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.1/2023-04-19/bin/linux/amd64/kubectl
               2. chmod +x ./kubectl 
               3. mv kubectl /bin  OR $ mv kubectl /usr/local/bin
               4. kubectl version --output=yaml

          # Installing or eksctl (Refer==https://github.com/eksctl-io/eksctl/blob/main/README.md#installation)
               1. curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
               2. cd /tmp
               3. sudo mv /tmp/eksctl /bin   OR  $ sudo mv /tmp/eksctl /usr/local/bin
               4. eksctl version
			   
			   
# Step 10 : Setup Kubernetes using eksctl :
          Setup Kubernetes using eksctl (Refer===https://github.com/aws-samples/eks-workshop/issues/734)
               1. eksctl create cluster --name virtualtechbox-cluster \
                  --region ap-south-1 \
                  --node-type t2.small \
               2. kubectl get nodes
               3. Create yaml files for deployment & service.


# Step 11 : Integrate bootstrap server with Ansible :
               1. vi /etc/sshd_config  Uncomment the Password line to yes 
               2. Add Bootstrap server ip address to ansible hosts file.


# Step 12 : Create ansible playbook to run deployment & service yml file :
			         1. passwd root
               2. nano /etc/ansible/hosts
                  [ansible]
                  localhost

                  [kubernetes]
                  BootStrap-Server-IP

               3. ssh-copy-id root@BootStrap-Server-IP

          Create Ansible Playbook to Run Deployment and Service Manifest files
               1. mv regapp.yml creat_image_regapp.yml 
               2. nano kube_deploy.yml
---
- hosts: kubernetes
  user: root

  tasks:
    - name: deploy regapp on kubernetes
      command: kubectl apply -f regapp-deployment.yml

    - name: create service for regapp
      command: kubectl apply -f regapp-service.yml

    - name: update deployment with new pods if image updated in docker hub
      command: kubectl rollout restart deployment.apps/virtualtechbox-regapp


# Step 13 :  Create Jenkins Deployment Job for Kubernetes :
        Here we need to edit the jenkins job and add some coomands in exec command section.
