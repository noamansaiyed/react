pipeline {
    environment {
        SERVER_IP = "13.232.96.165"
        SERVER_CREDENTIALSID = "test_agent"
        SERVER_DEPLOY_DIR = "/var/www/html/"

    }
    agent {
        docker {
            image 'node:12'
        }
    }
    stages {
        stage ('build') {
        steps{
         sh 'npm install'
         sh 'npm run build'
          sh "tar -cvf build.tar build"
         archiveArtifacts artifacts: 'build.tar', fingerprint: true
        }
        }
        stage ('deploy') {
        steps {
         unarchive mapping: ['build.tar': 'build.tar']
        

         
        sshagent(["ubuntu"]) {
         sh "scp -o StrictHostKeyChecking=no build.tar ubuntu@${SERVER_IP}:${SERVER_DEPLOY_DIR}"
         sh """
         ssh -o StrictHostKeyChecking=no ubuntu@${SERVER_IP} "sudo mkdir -p ${SERVER_DEPLOY_DIR}"
         """
         sh """
         ssh -o StrictHostKeyChecking=no ubuntu@${SERVER_IP} "sudo rm -rf ${SERVER_DEPLOY_DIR}build;sudo tar -xvf ${SERVER_DEPLOY_DIR}build.tar -C ${SERVER_DEPLOY_DIR}"
         """
        }
        }
        }

    }
}
