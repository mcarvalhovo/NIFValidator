#!groovy

pipeline {
    agent any
    environment {
        HOME = "$env.WORKSPACE"
    }
    stages {

        stage('docker environment') {
            agent {
                docker {
                    image 'python:3.11-slim'
                    reuseNode true
                }
            }
            steps {
                sh"""
                    pip install -r requirements.txt
                """
            }            
        }
        stage ('build') {
            steps {
                sh"""
                    docker build -t th3o4k/nif-validator .
                """
            }
        }
        stage ('Deliver') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId:'docker-id',
                    passwordVariable: 'passwd', 
                    usernameVariable: 'username')]) {
                    sh"""
                        docker build -t ${username}/${JOB_BASE_NAME} .
                        docker login -u ${username} -p ${passwd}
                        docker push ${username}/${JOB_BASE_NAME}
                    """
                    }
            }
        }
                stage('Deploy') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-id',
                    passwordVariable: 'passwd', 
                    usernameVariable: 'username')]) {
                    sshagent(credentials: ['cluster-crendentials']) {
                        sh"""
                        ssh -o StrictHostKeyChecking=no redhat@172.31.33.100 docker rm -f ${JOB_BASE_NAME} || true
                        ssh -o StrictHostKeyChecking=no redhat@172.31.33.100 docker run -d --name ${JOB_BASE_NAME} -p 8080:9046 ${username}/${JOB_BASE_NAME}
                        """
                    }
                }
            }
        }            
    }
}