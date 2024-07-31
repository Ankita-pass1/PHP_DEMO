pipeline{
    agent any

    environment{
        IMAGE_NAME='devopstrainer/java-mvn-privaterepos:php$BUILD_NUMBER'
        DEV_SERVER_IP='ec2-user@172.31.39.97'
       // DEPLOY_SERVER_IP='ec2-user@172.31.42.128'
        ACM_IP='ec2-user@172.31.35.35'
        AWS_ACCESS_KEY_ID =credentials("AWS_ACCESS_KEY")
        AWS_SECRET_ACCESS_KEY=credentials("AWS_SECRET_ACCESS_KEY")
        //created a new credential of type secret text to store docker pwd
        DOCKER_REG_PASSWORD=credentials("DOCKER_REG_PASSWORD")
    }

    stages{
        stage("Build the dockerimage for php and push to private registry"){
            steps{
            script{
                 sshagent(['slave2']) {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                    sh "scp -o strictHostKeyChecking=no -r devserverconfig ${DEV_SERVER_IP}:/home/ec2-user"
                    sh "ssh -o strictHostKeyChecking=no ${DEV_SERVER_IP} 'bash ~/devserverconfig/docker-script.sh'"
                    sh "ssh  ${DEV_SERVER_IP} sudo docker build -t ${IMAGE_NAME} /home/ec2-user/devserverconfig"
                    sh "ssh  ${DEV_SERVER_IP} sudo docker login -u $USERNAME -p $PASSWORD"
                    sh "ssh  ${DEV_SERVER_IP} sudo docker push ${IMAGE_NAME}"
                }
            }
            }
        }
        }
        // stage("Deploy the microsvc app with docker compose"){
        //     steps{
        //     script{
        //          sshagent(['slave2']) {
        //             withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
        //             sh "scp -o strictHostKeyChecking=no -r deployserverconfig ${DEPLOY_SERVER_IP}:/home/ec2-user"
        //             sh "ssh -o strictHostKeyChecking=no ${DEPLOY_SERVER_IP} 'bash ~/deployserverconfig/docker-script.sh'"
        //             //sh "ssh  ${DEV_SERVER_IP} sudo docker build -t ${IMAGE_NAME} ~/devserverconfig"
        //             sh "ssh  ${DEPLOY_SERVER_IP} sudo docker login -u $USERNAME -p $PASSWORD"
        //             sh "ssh  ${DEPLOY_SERVER_IP} bash /home/ec2-user/deployserverconfig/compose-script.sh ${IMAGE_NAME}"
        //         }
        //     }
        // }
        // }
        // }
         stage("Provision anisble target server with TF"){
            agent any
                   steps{
                       script{
                           dir('terraform'){
                           sh "terraform init"
                           sh "terraform apply --auto-approve"
                           ANSIBLE_TARGET_PUBLIC_IP = sh(
                            script: "terraform output ec2_ip",
                            returnStdout: true
                           ).trim()
                         echo "${ANSIBLE_TARGET_PUBLIC_IP}"   
                       }
                       }
                   }
        }

       stage("RUN ansible playbook on ACM"){
            agent any
            steps{
                script{
                    echo "copy ansible files on ACM and run the playbook"
                    sshagent(['slave2']) {
            //sh "ssh -o StrictHostKeyChecking=no ${ACM_IP} envsubst < ansible/docker-compose-var.yml > ansible/docker-compose.yml" 
            sh "scp -o StrictHostKeyChecking=no ansible/* ${ACM_IP}:/home/ec2-user"
            
            //copy the ansible target key on ACM as private key file
            withCredentials([sshUserPrivateKey(credentialsId: 'Ansible_target',keyFileVariable: 'keyfile',usernameVariable: 'user')]){ 
            sh "scp  $keyfile ${ACM_IP}:/home/ec2-user/.ssh/id_rsa"    
            }
            sh "ssh  ${ACM_IP} bash /home/ec2-user/ansible-config.sh ${AWS_ACCESS_KEY_ID} ${AWS_SECRET_ACCESS_KEY} ${DOCKER_REG_PASSWORD} ${IMAGE_NAME}"
      }
        }
        }    
    }
    }
}
