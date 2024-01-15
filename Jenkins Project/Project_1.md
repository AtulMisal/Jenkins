# Real Time DevOps Project | Deploy to Kubernetes Using Jenkins | End to End DevOps Project | CICD

1. Virtual TechBox (Playlist 8 Videos) :
   Real Time DevOps Project | Deploy to Kubernetes Using Jenkins | End to End DevOps Project | CICD  
	 
Step 1 : Install and Configure the Jenkins-Master and Jenkins-Agent :
  1. Create an EC2 instance free tier eligible & install jenkins on that. This is Jenkins-Master.
  2. Create an EC2 instance free tier eligible & install docker on that. This is Jenkins-Agent.
  3. On Jenkins-Agent edit vi /etc/ssh/sshd_config file. Uncomment two lines- PubkeyAuthontication & AuthorizedKeyFile.
  4. service sshd reload.
  5. On Jenkins-Agent after installing docker do this - usermod -aG docker $USER 
  6. On Jenkins-Agent do this - sudo init 6. This is same as reboot.
  7. On Jenkins-Master perform ssh-keygen & Copy the public key data.
  8. On Jenkins-Agent In .ssh/ path an authorized_keys file is there open it and paste the data that you copied in above step.
  9. Open Jenkins on Browser & in manage jenkins add Node as Jenkins-Agent.
 10. Add Nodes & it's Credentials using SSH via Private key. Copy & Paste data of Private key here.
 

Step 2 : Integrate Maven to Jenkins and Add GitHub Credentials to Jenkins :
  1. Install some plugins as maven integration & pipeline maven integration.
  2. In manage jenkins under configur tools add the tools like maven & jdk.
  3. Add GitHub Credentials in Jenkins. For password use Token.
  
  
Step 3 : Create Pipeline Script(Jenkinsfile) for Build & Test Artifacts and Create CI Job on Jenkins :
         # Step 4 also added here. I have divided the pipeline according to steps.		 

      pipeline {
        agent { label: 'Jenkins-Agent'}
        tools {
           jdk 'Java'       # The name given in manage jenkins - configuration tools 
           maven 'Maven3'   # The name given in manage jenkins - configuration tools 
         }
        environment {
           NAME = 'Gloabl_Variable'
           APP_NAME = "register-app-pipeline"
           RELEASE = "1.0.0"
           DOCKER_USER = "ashfaque9x"
           DOCKER_PASS = 'dockerhub'
           IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
           IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	   JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
         }
  
        stages {
          stage ("CleanUp Stage"){
	   steps {
	     cleanWs()
	  }
	 }
	
	  stage ("Git Repo") {
	    steps {
	      git url: "", credentialsId: "github", branch: "main"    # credentialsId is the id that we have added in Credentials in jenkins.
	  }
	  }

          stage ("Build Application") {
	    steps {
	     sh 'mvn clean package'
	  }
	}
	
	  stage ("Test Application") {
	    steps {
	      sh 'mvn test'
	  }
	  }
          //	  
          }
          }   

 
Step 4 : Install and Configure the SonarQube :
         # Add these stages in above pipeline from //
		 
     1. For SonarQube we need t3.medium instance type.
     2. Along with SonarQube we will install Postgresql in this instance.
     3. Need 15 GB storage. While creating this instance we have to do this.
     4. Add Postgresql repo as :
           sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
           wget -qO- https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo tee /etc/apt/trusted.gpg.d/pgdg.asc &>/dev/null
     5. Install Postgresql :
           sudo apt update
           sudo apt-get -y install postgresql postgresql-contrib
           sudo systemctl enable postgresql	 
	   6. Create Database for Sonarqube :
           sudo passwd postgres
           su - postgres
           createuser sonar
           psql
           ALTER USER sonar WITH ENCRYPTED password 'sonar';
           CREATE DATABASE sonarqube OWNER sonar;
           grant all privileges on DATABASE sonarqube to sonar;
           \q
		       exit		  
     7. Add Adoptium repository :
           sudo bash
           wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
           echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) 
           main" | tee /etc/apt/sources.list.d/adoptium.list
     8. Install Java 17 :
           apt update
           apt install temurin-17-jdk
           update-alternatives --config java
           /usr/bin/java --version
           exit
     7. Increase Limits
           sudo vim /etc/security/limits.conf
           # Paste the below values at the bottom of the file
            sonarqube   -   nofile   65536
            sonarqube   -   nproc    4096		   
     8. Increase Mapped Memory Regions
           sudo vim /etc/sysctl.conf
           # Paste the below values at the bottom of the file
           vm.max_map_count = 262144
     9. Download and Extract
           sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.0.65466.zip
           sudo apt install unzip
           sudo unzip sonarqube-9.9.0.65466.zip -d /opt
           sudo mv /opt/sonarqube-9.9.0.65466 /opt/sonarqube
      # Create user and set permissions
           sudo groupadd sonar
           sudo useradd -c "user to run SonarQube" -d /opt/sonarqube -g sonar sonar
           sudo chown sonar:sonar /opt/sonarqube -R
      # Update Sonarqube properties with DB credentials
           sudo vim /opt/sonarqube/conf/sonar.properties
      # Find and replace the below values, you might need to add the sonar.jdbc.url
           sonar.jdbc.username=sonar
           sonar.jdbc.password=sonar
           sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
      # Create service for Sonarqube
           sudo vim /etc/systemd/system/sonar.service
      # Paste the below into the file
           [Unit]
           Description=SonarQube service
           After=syslog.target network.target

           [Service]
           Type=forking

           ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
           ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

           User=sonar
           Group=sonar
           Restart=always

           LimitNOFILE=65536
           LimitNPROC=4096

           [Install]
           WantedBy=multi-user.target

    10. Start Sonarqube and Enable service
           sudo systemctl start sonar
           sudo systemctl enable sonar
           sudo systemctl status sonar

    11. Watch log files and monitor for startup
           sudo tail -f /opt/sonarqube/logs/sonar.log

Step 5 : Integrate Sonarqube with Jenkins :

            1. First generate the sonarqube token as :
                   My Account -> Security -> Generate Token
            2. Create an Credentials i Jenkins for Sonarqube with private text.
               Use above token as private text.
            3. Install Sonarqube plugin.
            4. Configure Sonarqube in Jenkins. 
                  Manage Jenkins -> System -> Sonarqube
            5. Configure Sonarqube Scanner.     

Step 6 : Build & Push Docker Image using Pipeline Script :

            1. Install docker plugins in jenkins - docker, docker commons, docker pipeline, docker api, docker build step, 
               cloud bees docker build & publish.
            2. Here will upload the docker image on docker hub. Using pipeline commands.


Step 7 : Setup Bootstrap Server for eksctl and Setup Kubernetes using eksctl :
          
	  1. Install AWS Cli on the above EC2
             Refer--https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
          2. sudo su
          3. curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
          4. apt install unzip,   $ unzip awscliv2.zip
          5. sudo ./aws/install
               OR
          6. sudo yum remove -y aws-cli
          7. pip3 install --user awscli
          8. sudo ln -s $HOME/.local/bin/aws /usr/bin/aws
          9. aws --version		       

Step 8 : ArgoCD Installation on EKS Cluster and Add EKS Cluster to ArgoCD :
         
	 1. Installing kubectl
            Refer--https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html
         2. sudo su
         3. curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.1/2023-04-19/bin/linux/amd64/kubectl
         4. ll , $ chmod +x ./kubectl  //Gave executable permisions
         5. mv kubectl /bin   //Because all our executable files are in /bin
         6. kubectl version --output=yaml

         7. Installing  eksctl
            Refer---https://github.com/eksctl-io/eksctl/blob/main/README.md#installation
         8. curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
         9. cd /tmp
        10. ll
        11. sudo mv /tmp/eksctl /bin
        12. eksctl version

        13. Setup Kubernetes using eksctl
            Refer--https://github.com/aws-samples/eks-workshop/issues/734
        14. eksctl create cluster --name virtualtechbox-cluster \
            --region ap-south-1 \
            --node-type t2.small \
            --nodes 3 \

        15. kubectl get nodes


Step 9 : ArgoCD Installation on EKS Cluster and Add EKS Cluster to ArgoCD :

          1) First, create a namespace
                kubectl create namespace argocd
          2) Next, let's apply the yaml configuration files for ArgoCd
                kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
          3) Now we can view the pods created in the ArgoCD namespace.
                kubectl get pods -n argocd
          4) To interact with the API Server we need to deploy the CLI:
                curl --silent --location -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v2.4.7/argocd-linux-amd64
                chmod +x /usr/local/bin/argocd
          5) Expose argocd-server
                kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
          6) Wait about 2 minutes for the LoadBalancer creation
                kubectl get svc -n argocd
          7) Get pasword and decode it.
                kubectl get secret argocd-initial-admin-secret -n argocd -o yaml
                echo WXVpLUg2LWxoWjRkSHFmSA== | base64 --decode
       Add EKS Cluster to ArgoCD
         9) login to ArgoCD from CLI
                argocd login a2255bb2bb33f438d9addf8840d294c5-785887595.ap-south-1.elb.amazonaws.com --username admin
        10) argocd cluster list
        11) Below command will show the EKS cluster
               kubectl config get-contexts
        12) Add above EKS cluster to ArgoCD with below command
               argocd cluster add i-08b9d0ff0409f48e7@virtualtechbox-cluster.ap-south-1.eksctl.io --name virtualtechbox-eks-cluster
        13) kubectl get svc	


 Step 10 : Clean Up Cluster :
            kubectl get all
            kubectl delete deployment.apps/virtualtechbox-regapp       //it will delete the deployment
            kubectl delete service/virtualtechbox-service              //it will delete the service
            eksctl delete cluster virtualtechbox --region ap-south-1     OR    eksctl delete cluster --region=ap-south-1 --name=virtualtechbox-cluster      
	    //it will delete the EKS 
            cluster	

