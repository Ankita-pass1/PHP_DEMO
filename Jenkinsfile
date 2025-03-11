pipeline {
   agent none
  environment{
      BUILD_SERVER_IP='ec2-user@172.31.27.112'
       IMAGE_NAME='ankita2025/docker:php$BUILD_NUMBER'
       DEPLOY_SERVER_IP='ec2-user@172.31.27.28'
   }
    stages {          
        stage('BUILD DOCKERIMAGE AND PUSH TO DOCKERHUB') {
            agent any            
            steps {
                script{
                sshagent(['slave2']) {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                echo "Packaging the apps"
                sh "scp -o StrictHostKeyChecking=no -r docker-files ${BUILD_SERVER_IP}:/home/ec2-user"
                sh "ssh -o StrictHostKeyChecking=no ${BUILD_SERVER_IP} 'bash ~/docker-files/docker-script.sh'"
                sh "ssh ${BUILD_SERVER_IP} sudo docker build -t ${IMAGE_NAME} /home/ec2-user/docker-files/"
                sh "ssh ${BUILD_SERVER_IP} sudo docker login -u $USERNAME -p $PASSWORD"
                sh "ssh ${BUILD_SERVER_IP} sudo docker push ${IMAGE_NAME}"
              }
            }
            }
        }
        }
       stage('DEPLOY DOCKER CONATINER USING DOCKER_COMPOSE'){
           agent any
           steps{
               script{
                    sshagent(['slave2']){
                         withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                         sh "scp -o StrictHostKeyChecking=no -r docker-files ${DEPLOY_SERVER_IP}:/home/ec2-user"
                         sh "ssh -o StrictHostKeyChecking=no ${DEPLOY_SERVER_IP} 'bash ~/docker-files/docker-script.sh'"
                         sh "ssh ${DEPLOY_SERVER_IP} sudo docker login -u $USERNAME -p $PASSWORD"
                         sh "ssh ${DEPLOY_SERVER_IP} bash /home/ec2-user/docker-files/docker-compose-script.sh ${IMAGE_NAME}"
                         }
                    }
               }
           }
       }
    }
}
