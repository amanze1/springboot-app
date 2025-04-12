pipeline {
   tools {
        maven 'Maven3'
    }
    agent any
    environment {
        registry = "715841347012.dkr.ecr.us-east-2.amazonaws.com/devops/my-docker-repo"
    }
   
    stages {
        stage('Cloning Git') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/amanze1/springboot-app']])
            }
        }
      stage ('Build') {
          steps {
            sh 'mvn clean install'           
            }
      }
    // Building Docker images
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build registry 
          dockerImage.tag("$BUILD_NUMBER")
        }
      }
    }
   
    // Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
     steps{  
         script {
                sh 'aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 715841347012.dkr.ecr.us-east-2.amazonaws.com'
                sh 'docker push 715841347012.dkr.ecr.us-east-2.amazonaws.com/devops/my-docker-repo:$BUILD_NUMBER'
         }
        }
      }
    // Avoid latest tag image and pass build ID dynamically from Jenkins pipeline
       stage('K8S Deploy') {
        steps{   
            script {
                withKubeConfig([credentialsId: 'K8S', serverUrl: '']) {
                echo "Current build number is: ${env.BUILD_ID}"
               // Replace your aws acct ID in the eks-deploy-k8s.yaml file, line 19 
                sh """ 
                sed -i 's/\${BUILD_NUMBER}/${env.BUILD_ID}/g' eks-deploy-k8s.yaml
                """ 
                sh ('kubectl apply -f  eks-deploy-k8s.yaml -n springboot-app-ns')
                }
            }
        }
       }
    }
}
