pipeline {
    agent any
    
    tools {
        nodejs "nodejs"
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        ARM_CLIENT_ID = credentials('azure-client-id')
        ARM_CLIENT_SECRET = credentials('azure-client-secret')
        ARM_SUBSCRIPTION_ID = credentials('azure-subscription-id')
        ARM_TENANT_ID = credentials('azure-tenant-id')
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/ranyaboubich/Hello-world-pipeline'
            }
        }
        
        stage('Identify Agent') {
            steps {
                echo "Running on agent: ${env.NODE_NAME}"
            }
        }
        
        stage('Install Dependencies') {
            steps {
                bat 'npm install'
            }
        }
        
        stage('Build') {
            steps {
                bat 'npm run build'
            }
        }
        
        stage('Test') {
            steps {
                bat 'npm run test'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building the docker image ...'
                    bat "docker build -t ranyaboubich/hello-world:latest ."
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        bat 'docker login -u %DOCKER_USERNAME% -p %DOCKER_PASSWORD%'
                        bat 'docker push ranyaboubich/hello-world:latest'
                    }
                }
            }
        }

        stage('Terraform') {
            steps {
                script {
                    echo 'Terraform ...'
                    bat 'terraform init'
                    bat 'terraform plan'
                    bat 'terraform apply -auto-approve'
                    echo 'Terraform done'
                }
            }
        }

        stage('configure kubeconfig'){
            steps {
                script {
                    def kubeconfig = sh(returnStdout: true, script: 'terraform output -raw kubeconfig')
                       writeFile file: 'kubeconfig.yaml', text: kubeconfig
                    sh 'export KUBECONFIG=$WORKSPACE/kubeconfig.yaml'
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo 'Deploying the application ...'
                    bat 'kubectl apply -f deployment.yaml'
                }
            }
        }
        
    }

}