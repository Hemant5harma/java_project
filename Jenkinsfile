pipeline {
    agent any

    tools {
        maven 'maven' 
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'eks-application-3', url: 'https://github.com/Hemant5harma/java_project.git'
            }
        }
        
        stage('Build Maven Project') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                echo 'Test done'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    withDockerRegistry([credentialsId: 'docker-cred']) {
                        sh '''
                            docker build -t ${JOB_NAME}:${BUILD_ID} .
                            docker tag ${JOB_NAME}:${BUILD_ID} hemanthub/${JOB_NAME}:${BUILD_ID}
                            docker tag ${JOB_NAME}:${BUILD_ID} hemanthub/${JOB_NAME}:latest
                        '''
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry([credentialsId: 'docker-cred']) {
                        sh '''
                            docker push hemanthub/${JOB_NAME}:${BUILD_ID}
                            docker push hemanthub/${JOB_NAME}:latest
                        '''
                    }
                }
            }
        }
        stage('Deploy to EKS') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yml'
                sh 'kubectl apply -f k8s/service.yml'
                sh 'kubectl rollout restart deployment/eks2 -n ingress-nm'
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs for more details.'
        }
    }
}
