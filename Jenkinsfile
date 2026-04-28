pipeline {
    agent any

    environment {
        DEV_SERVER = "13.126.231.48"
        STG_SERVER = "13.126.75.43"
        PRD_SERVER = "13.233.3.126"

        IMAGE_NAME = "apache-demo"
        CONTAINER_NAME = "apache-container"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Deploy to DEV') {
            when {
                branch 'dev'
            }
            steps {
                sshagent(['ec2-ssh']) {
                    sh '''
                    docker save $IMAGE_NAME | bzip2 | ssh -o StrictHostKeyChecking=no ec2-user@$DEV_SERVER 'bunzip2 | docker load'

                    ssh ec2-user@$DEV_SERVER "
                    docker stop $CONTAINER_NAME || true
                    docker rm $CONTAINER_NAME || true
                    docker run -d -p 80:80 --name $CONTAINER_NAME $IMAGE_NAME
                    "
                    '''
                }
            }
        }

        stage('Deploy to STG') {
            when {
                branch 'stg'
            }
            steps {
                sshagent(['ec2-ssh']) {
                    sh '''
                    docker save $IMAGE_NAME | bzip2 | ssh -o StrictHostKeyChecking=no ec2-user@$STG_SERVER 'bunzip2 | docker load'

                    ssh ec2-user@$STG_SERVER "
                    docker stop $CONTAINER_NAME || true
                    docker rm $CONTAINER_NAME || true
                    docker run -d -p 80:80 --name $CONTAINER_NAME $IMAGE_NAME
                    "
                    '''
                }
            }
        }

        stage('Deploy to PROD') {
            when {
                branch 'main'
            }
            steps {
                sshagent(['ec2-ssh']) {
                    sh '''
                    docker save $IMAGE_NAME | bzip2 | ssh -o StrictHostKeyChecking=no ec2-user@$PRD_SERVER 'bunzip2 | docker load'

                    ssh ec2-user@$PRD_SERVER "
                    docker stop $CONTAINER_NAME || true
                    docker rm $CONTAINER_NAME || true
                    docker run -d -p 80:80 --name $CONTAINER_NAME $IMAGE_NAME
                    "
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Deployment Successful"
        }
        failure {
            echo "Deployment Failed"
        }
    }
}
