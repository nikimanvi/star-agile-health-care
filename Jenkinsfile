pipeline {
    agent { label 'Jenkins_Slave' }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker_CI')
    }

    stages {
        stage('SCM Checkout') {
            steps {
                echo "Perform SCM Checkout"
                git 'https://github.com/Deepakd106/star-agile-health-care.git'
            }
        }

        stage('Application Build') {
            steps {
                echo "Perform Application Build"
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker version'
                sh "docker build -t niki/healthcare:${BUILD_NUMBER} ."
                sh "docker image list"
                sh "docker tag niki/healthcare:${BUILD_NUMBER} niki/healthcare:latest"
            }
        }

        stage('Login to DockerHub') {
            steps {
                sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
            }
        }

        stage('Publish to Docker Registry') {
            steps {
                sh "docker push niki/healthcare:${BUILD_NUMBER}"
                sh "docker push niki/healthcare:latest"
            }
        }

        stage('Deploy to Kubernetes Cluster') {
            steps {
                script {
                    // Optionally use sshPublisher here if deploying remotely
                    sh '''
                        kubectl apply -f kubedeploy_Healthcare_Project.yaml
                        kubectl set image deployment/myproject-healthcare medicure=niki/healthcare:${BUILD_NUMBER}
                        kubectl rollout status deployment/myproject-healthcare
                    '''
                }
            }
        }
    }
}

