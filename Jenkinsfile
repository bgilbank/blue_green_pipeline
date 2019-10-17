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
            withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
            sh '''
            docker build -t bgilbank/cloudcapstone:$BUILD_ID .
            '''
            }
          }
        }
        stage('Push image to Dockerhub') {
          dockerpath="bgilbank/cloudcapstone"
          docker login -u bgilbank
          docker tag cloudcapstone bgilbank/cloudcapstone:v1
          docker push bgilbank/cloudcapstone:v1
        }
        stage('Set kubectl context') {
            kubectl config use-context arn:aws:eks:us-west-2:cluster/cloudcapstonecluster
        }
        stage('Deploy Blue Container') {
          kubectl apply -f ./blue_controller.json
        }
        stage('Deploy Green Container') {
          kubectl apply -f ./green_controller.json
        }
        stage('Create A Service and redirect traffic to Blue Container') {
          kubectl apply -f ./blue_green_service.json
        }
        stage('Wait for user input') {
           steps {
                input "Does the staging environment look ok?"
            } 
        }
        stage('Update the service to redirect traffic to Green Container') {
          kubectl apply -f ./blue-controller.json
        }

      }
    }
