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
stage('Smoke test') {
  steps {
    script {
      echo "Smoke test: retrying curl up to 8 times with 3s wait between attempts..."
      // try curl first; if curl not present use PowerShell Invoke-WebRequest fallback
      def tryCurl = true
      try {
        bat 'where curl'
      } catch (err) {
        tryCurl = false
      }

      def success = false
      if (tryCurl) {
        for (int i=1; i<=8; i++) {
          echo "curl attempt ${i}..."
          def rc = bat(script: "curl -s -f http://localhost:${env.HOST_PORT} -o output.html", returnStatus: true)
          if (rc == 0) { success = true; break }
          bat 'timeout /t 3 /nobreak >nul'
        }
      } else {
        // PowerShell fallback
        for (int i=1; i<=8; i++) {
          echo "PowerShell Invoke-WebRequest attempt ${i}..."
          def rc = bat(script: 'powershell -Command "try { (Invoke-WebRequest -UseBasicParsing -Uri http://localhost:${env.HOST_PORT} -TimeoutSec 5).Content | Out-File output.html; exit 0 } catch { exit 1 }"', returnStatus: true)
          if (rc == 0) { success = true; break }
          bat 'timeout /t 3 /nobreak >nul'
        }
      }

      if (success) {
        echo "Smoke test passed — showing head of page:"
        bat 'type output.html | more'
      } else {
        echo "Smoke test failed after retries — collecting diagnostics..."
        // show container status and logs to help debugging
        bat "docker ps -a --filter name=${env.CONTAINER} --format \"table {{.ID}}\\t{{.Names}}\\t{{.Status}}\\t{{.Ports}}\" || echo no-container"
        bat "docker logs ${env.CONTAINER} --tail 300 || echo no-logs"
        archiveArtifacts artifacts: 'output.html', allowEmptyArchive: true
        error("Smoke test failed: could not reach http://localhost:${env.HOST_PORT}")
      }
    }
  }
}




