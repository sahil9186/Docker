pipeline {
    agent any

    environment {
        DEV_SERVER = "65.2.131.51"
        STG_SERVER = "3.108.228.60"
        PRD_SERVER = "13.233.232.244"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t apache-demo .'
            }
        }

        stage('Deploy to DEV') {
            when {
                branch 'dev'
            }
            steps {
                sh '''
                ssh -o StrictHostKeyChecking=no ec2-user@$DEV_SERVER << EOF
                docker stop apache-demo || true
                docker rm apache-demo || true
                docker run -d -p 80:80 --name apache-demo apache-demo
                EOF
                '''
            }
        }

        stage('Deploy to STG') {
            when {
                branch 'stg'
            }
            steps {
                sh '''
                ssh -o StrictHostKeyChecking=no ec2-user@$STG_SERVER << EOF
                docker stop apache-demo || true
                docker rm apache-demo || true
                docker run -d -p 80:80 --name apache-demo apache-demo
                EOF
                '''
            }
        }

        stage('Deploy to PROD') {
            when {
                branch 'prd'
            }
            steps {
                sh '''
                ssh -o StrictHostKeyChecking=no ec2-user@$PRD_SERVER << EOF
                docker stop apache-demo || true
                docker rm apache-demo || true
                docker run -d -p 80:80 --name apache-demo apache-demo
                EOF
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment Successful'
        }

        failure {
            echo 'Deployment Failed'
        }
    }
}
