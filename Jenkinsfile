pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/atharvapachunde/Hello-Devops.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🛠 Building Docker image...'
                bat 'docker build -t hello-devops:11 .'
            }
        }

        stage('Run Docker Container') {
            steps {
                echo '🚀 Running Docker container...'
                bat '''
                for /F "tokens=*" %%i in ('docker ps -aq -f "name=hello-devops-11"') do (
                    docker stop %%i  
                    docker rm %%i
                )
                docker run -d -p 5000:5000 --name hello-devops-11 hello-devops:11
                REM Wait for container to start
                ping 127.0.0.1 -n 6 >nul
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo '🩺 Checking health...'
                script {
                    def result = bat(returnStatus: true, script: 'curl -s http://localhost:5000 >nul')
                    if (result != 0) {
                        error("❌ Health check failed! App not reachable.")
                    } else {
                        echo "✅ Flask app is up and reachable!"
                    }
                }
            }
        }
    }

    post {
        always {
            echo "📋 Showing container logs:"
            bat 'docker logs mystifying_kilby || echo No logs found'
        }
        success {
            echo "🎉 Build & container ran successfully!"
        }
        failure {
            echo "⚠ Build failed — check above logs."
        }
    }
}


