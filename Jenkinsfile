pipeline {
  agent any
  environment {
     image_name = 'my-note_app'
  }
  stages {
    stage ('cloning the public repo') {
      steps {
      git url: "https://github.com/LondheShubham153/django-notes-app.git", branch: "main"
      sh 'echo "!!! Repo cloned successfully !!!"'
    }
    }
    stage ('Build the docker image') {
      steps {
        sh 'docker build -t ${image_name} .'
    }
    }
    stage("Push to Docker Hub"){
      steps {
        echo "Pushing the image to docker hub"
        withCredentials([usernamePassword(credentialsId:"docker_hub",passwordVariable:"dockerHubPass",usernameVariable:"dockerHubUser")]){
        sh "docker tag ${image_name} ${env.dockerHubUser}/${image_name}"
        sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
        sh "docker push ${env.dockerHubUser}/${image_name}:latest"
                }
            }
        }
    stage ('Deploy to Prod') {
      steps {
        sh 'apt-get install docker-compose -y'
        sh 'docker-compose down && docker-compose up -d'
  }
  } 
}
 post {
   always {
     echo "Successfully Completed"
   }
   failure {
     echo "Failure Occurs"
   }
   unsuccessful {
    echo "unsuccessful"
   }
 }
}
