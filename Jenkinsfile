pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/<your-username>/hello-devops.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build('hello-devops')
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    // Stop existing container if running
                    sh 'docker rm -f hello-devops-container || true'
                    // Run new container
                    sh 'docker run -d -p 5000:5000 --name hello-devops-container hello-devops'
                }
            }
        }
    }
}