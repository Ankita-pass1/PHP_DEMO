pipeline {
   agent none
  environment{
      BUILD_SERVER_IP='ec2-user@172.31.47.86'
      // IMAGE_NAME='devopstrainer/java-mvn-privaterepos:php$BUILD_NUMBER'
        IMAGE_NAME='newaxis.jfrog.io/mydocker-docker/PHP:$BUILD_NUMBER'
       DEPLOY_SERVER_IP='ec2-user@172.31.45.121'
   }
    stages {          
        stage('BUILD DOCKERIMAGE AND PUSH TO DOCKERHUB') {
            agent any            
            steps {
                script{
                sshagent(['slave2']) {
                withCredentials([usernamePassword(credentialsId: 'jfrog-cred', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                echo "Packaging the apps"
                sh "scp -o StrictHostKeyChecking=no -r devconfig ${BUILD_SERVER_IP}:/home/ec2-user"
                sh "ssh -o StrictHostKeyChecking=no ${BUILD_SERVER_IP} 'bash ~/devconfig/docker-script.sh'"
                sh "ssh ${BUILD_SERVER_IP} sudo docker build -t ${IMAGE_NAME} /home/ec2-user/devconfig/"
                sh "ssh ${BUILD_SERVER_IP} sudo docker login -u $USERNAME -p $PASSWORD newaxis.jfrog.io"
                sh "ssh ${BUILD_SERVER_IP} sudo docker push ${IMAGE_NAME}"
              }
            }
            }
        }
        }
      stage('DEPLOY DOCKER CONTAINER USING DOCKER_COMPOSE'){
           agent any
           steps{
               script{
                    sshagent(['slave2']){
                         withCredentials([usernamePassword(credentialsId: 'jfrog-cred', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                         sh "scp -o StrictHostKeyChecking=no -r testconfig ${DEPLOY_SERVER_IP}:/home/ec2-user"
                         sh "ssh -o StrictHostKeyChecking=no ${DEPLOY_SERVER_IP} 'bash ~/testconfig/docker-script.sh'"
                         sh "ssh ${DEPLOY_SERVER_IP} sudo docker login -u $USERNAME -p $PASSWORD newaxis.jfrog.io"
                         sh "ssh ${DEPLOY_SERVER_IP} bash /home/ec2-user/testconfig/docker-compose-script.sh ${IMAGE_NAME}"
                         }
                    }
               }
           }
       }
    }
}
