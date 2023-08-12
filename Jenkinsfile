pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps {
        sh 'echo passed'
        git branch: 'main', url: 'https://github.com/rishipollai/devops-automation.git'
      }
    }
    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        // build the project and create a JAR file
        sh 'mvn clean package'
      }
    }
    stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://13.126.98.242:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
   stage('Dockerfile build image') {
            steps {
                 sh 'docker image build -t $JOB_NAME:V1.$BUILD_ID .'
            }
         }

         stage('Docker tag images') {
            steps {

                 sh 'docker image tag $JOB_NAME:V1.$BUILD_ID rishipollai/$JOB_NAME:V1.$BUILD_ID'
                 sh 'docker image tag $JOB_NAME:V1.$BUILD_ID rishipollai/$JOB_NAME:latest'
               }
         }

         stage('Push Docker images to dockerhub') {
            steps {
                  withCredentials([string(credentialsId: 'dockerhub_password', variable: 'dockerhub_password')]) {
                     sh "docker login -u rishipollai -p ${dockerhub_password}"
                     sh 'docker image push rishipollai/$JOB_NAME:V1.$BUILD_ID'
                     sh 'docker image push rishipollai/$JOB_NAME:latest'
                     
                     sh 'docker image rm rishipollai/$JOB_NAME:V1.$BUILD_ID rishipollai/$JOB_NAME:latest $JOB_NAME:V1.$BUILD_ID'
                
               }
            }
         }

         stage('Docker Deploy') {
            steps {
              ithCredentials([string(credentialsId: 'dockerhub_password', variable: 'dockerhub_password')]) {
                     sh "docker login -u rishipollai -p ${dockerhub_password}"
                     sh 'docker run -d -p 8011:8080 --name doct rishipollai/$JOB_NAME:latest'
               }       
            }
         }


    // stage('Update Deployment File') {
    //     environment {
    //         GIT_REPO_NAME = "Jenkins-Zero-To-Hero"
    //         GIT_USER_NAME = "iam-veeramalla"
    //     }
    //     steps {
    //         withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
    //             sh '''
    //                 git config user.email "abhishek.xyz@gmail.com"
    //                 git config user.name "Abhishek Veeramalla"
    //                 BUILD_NUMBER=${BUILD_NUMBER}
    //                 sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests/deployment.yml
    //                 git add java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests/deployment.yml
    //                 git commit -m "Update deployment image to version ${BUILD_NUMBER}"
    //                 git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
    //             '''
    //         }
    //     }
    // }
  }
}
