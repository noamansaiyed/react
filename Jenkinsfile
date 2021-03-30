pipeline {
    environment {
        SERVER_IP_1 = "13.232.96.165"
        //SERVER_CREDENTIALSID = "abcd1234-abcd-abcd-abcd-abcd1234abcd"
        SERVER_DEPLOY_DIR = "$HOME"

        CACHE_DIR = "/tmp"

        GIT_URL = "git@github.com:noamansaiyed/react.git"
        GIT_BRANCH = "master"
        //GIT_CREDENTIALSID = "abcd1234-abcd-abcd-abcd-abcd1234abcd"
    }
    agent none
    stages {
        stage('Checkout code') {
            agent any
            steps {
                git (
                    branch: "${GIT_BRANCH}",
                    credentialsId: "${GIT_CREDENTIALSID}",
                    url: "${GIT_URL}",
                    changelog: true
                )
                sh '''
                    ls -al
                    cache_dir="${CACHE_DIR}"
                    cache_nm="${CACHE_DIR}node_modules"
                    cache_lock="${CACHE_DIR}yarn.lock"

                    if [ ! -d "$cache_dir" ]; then mkdir ${cache_dir}; fi
                    if [ ! -d "$cache_nm" ]; then mkdir ${cache_nm}; fi
                    if [ -d "$cache_nm" ]; then ln -sf ${cache_nm} ./; fi
                    if [ -f "$cache_lock" ]; then mv -n ${cache_lock} .; fi

                    ls -al
                '''
            }
        }
        stage('Build') {
            agent {
                docker {
                    image 'node:8-alpine'
                    args ''
                }
            }
            steps {
                sh '''
                    npm config set registry https://registry.npm.taobao.org
                    yarn install
                    yarn build
                    tar -cvf build.tar build

                    ls -al
                    mv ./yarn.lock ${CACHE_DIR}
                    rm -rf ./node_modules
                    ls -al
                '''
                archiveArtifacts artifacts: 'build.tar', fingerprint: true
            }
        }
        stage('Deploy') {
            agent any
            steps {
                unarchive mapping: ['build.tar': 'build.tar']
                echo '--- Deploy ---'

                sshagent(["${SERVER_CREDENTIALSID}"]) {
                  sh "scp -o StrictHostKeyChecking=no build.tar root@${SERVER_IP_1}:${SERVER_DEPLOY_DIR}"
                  sh "ssh -o StrictHostKeyChecking=no root@${SERVER_IP_1} \"rm -rf ${SERVER_DEPLOY_DIR}build; tar -xvf ${SERVER_DEPLOY_DIR}build.tar -C ${SERVER_DEPLOY_DIR}\""
                }

            }
        }
    }
}
