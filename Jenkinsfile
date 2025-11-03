pipeline {
  agent any
  environment {
    IMAGE = "hello-devops:${BUILD_NUMBER}"
    CONTAINER = "hello-devops-${BUILD_NUMBER}"
    APP_PORT = "5000"
    HOST_PORT = "5000"
  }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Docker image') {
      steps {
        script {
          echo "Building Docker image ${IMAGE}"
          // Use "bat" on Windows
          bat "docker build -t %IMAGE% ."
        }
      }
    }

    stage('Run container') {
      steps {
        script {
          echo "Cleaning up any existing container named %CONTAINER%"
          // remove any old container with same name, ignore errors
          bat """
            @echo off
            for /f "tokens=*" %%i in ('docker ps -a --filter "name=%CONTAINER%" --format "%%{{.Names}}"') do (
              if "%%i"=="%CONTAINER%" docker rm -f %CONTAINER% || echo failed-to-remove
            )
            docker run -d --name %CONTAINER% -p %HOST_PORT%:%APP_PORT% %IMAGE%
          """
        }
      }
    }

    stage('Smoke test') {
      steps {
        script {
          echo "Waiting for app to start..."
          bat "timeout /t 3 /nobreak >nul"
          echo "Requesting homepage..."
          // Use curl builtin on modern Windows or use PowerShell Invoke-WebRequest
          bat "curl -f http://localhost:%HOST_PORT% -o output.html"
          bat "type output.html"
        }
      }
    }
  }

  post {
    success {
      echo "Pipeline succeeded. Container is running: %CONTAINER%"
      bat "docker ps --filter name=%CONTAINER% --format \"Name: {{.Names}} Image: {{.Image}} Status: {{.Status}}\" || echo no-container"
      archiveArtifacts artifacts: 'output.html', allowEmptyArchive: true
    }
    failure {
      echo "Pipeline failed. Showing container logs (if any)..."
      bat "docker ps -a --filter name=%CONTAINER% --format \"{{.Names}}\" || echo no-container"
      bat "docker logs %CONTAINER% || echo no-logs"
    }
  }
}
