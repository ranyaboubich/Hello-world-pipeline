pipeline {
    agent any
    
    tools {
        nodejs "nodejs"
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
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
                bat 'npm audit fix'
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

        stage('Scan') {
            steps {
                bat 'npm audit'
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
    }

    post {
        success {
            build job: 'CD-Pipeline'
        }
    }
}