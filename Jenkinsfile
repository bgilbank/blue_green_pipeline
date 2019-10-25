pipeline {
    agent any

    stages {
        stage('Lint HMTL') {
          steps {
              sh 'tidy -q -e *.html'
            }
        }
        stage('Build local Docker Image') {
          steps {
            withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: '75b4bc4a-52c3-4647-9cd7-2b50d6f368dc', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
            sh '''
             docker build --tag=cloudcapstone .
            '''
            }
          }
        }
        stage('Push image to Dockerhub') {
          steps {
            withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: '75b4bc4a-52c3-4647-9cd7-2b50d6f368dc', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
            sh '''
              dockerpath="bgilbank/cloudcapstone"
              docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
              docker tag cloudcapstone bgilbank/cloudcapstone:v1
              docker push bgilbank/cloudcapstone:v1
            '''
            }
          }
        }
        stage('Set kubectl context') {
          steps {
            sh 'kubectl config use-context arn:aws:eks:us-west-2:430555665756:cluster/cloudcapstone'
          }
        }
        stage('Deploy Blue Container') {
          steps {
            sh 'kubectl apply -f ./blue_controller.json'
          }
        }
        stage('Deploy Green Container') {
          steps {
            sh 'kubectl apply -f ./green_controller.json'
          }
        }
        stage('Create A Service and redirect traffic to Blue Container') {
          steps {
            sh 'kubectl apply -f ./blue_green_service.json'
          }
        }
        stage('Wait for user input') {
           steps {
                input "Does the staging environment look ok?"
           }
        }
        stage('Update the service to redirect traffic to Green Container') {
          steps {
            sh 'kubectl apply -f ./blue_green_service.json'
          }
        }

      }
    }
