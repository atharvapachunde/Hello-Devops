pipeline {
  agent any

  environment {
    IMAGE = "hello-devops:${BUILD_NUMBER}"
    CONTAINER = "hello-devops-${BUILD_NUMBER}"
    HOST_PORT = "5000"
    APP_PORT = "5000"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build image') {
      steps {
        echo "Building image ${env.IMAGE}"
        bat "docker build -t ${env.IMAGE} ."
      }
    }

    stage('Stop any old container & Run') {
      steps {
        script {
          echo "Removing old container (if exists)"
          // remove by name if present; ignore errors
          bat '''
            @echo off
            for /f "tokens=*" %%C in ('docker ps -a --filter "name=%CONTAINER%" --format "%%{{.Names}}"') do (
              if "%%C"=="%CONTAINER%" docker rm -f %CONTAINER% || echo remove-failed
            )
          '''
          echo "Also remove any container using host port %HOST_PORT% (to avoid bind errors)"
          bat '''
            @echo off
            for /f "tokens=1" %%I in ('docker ps -a --format "%%{{.ID}} %%{{.Names}} %%{{.Ports}}" ^| findstr /C::%HOST_PORT%->') do (
              echo removing %%I
              docker rm -f %%I || echo remove-failed-%%I
            )
          '''
          echo "Starting new container"
          bat "docker run -d --name %CONTAINER% -p %HOST_PORT%:%APP_PORT% %IMAGE%"
          echo "Waiting a few seconds for the app to start..."
          bat "timeout /t 6 /nobreak >nul"
        }
      }
    }

    stage('Smoke test') {
      steps {
        script {
          echo "Trying to reach http://localhost:%HOST_PORT% (retries)..." 
          def ok = false
          for (int i = 1; i <= 6; i++) {
            echo "Attempt ${i}"
            // Use PowerShell to request the page and save to output.html (works even without curl)
            def rc = bat(
              script: 'powershell -Command "try { (Invoke-WebRequest -UseBasicParsing -Uri http://localhost:%HOST_PORT% -TimeoutSec 5).Content | Out-File output.html; exit 0 } catch { exit 1 }"',
              returnStatus: true
            )
            if (rc == 0) { ok = true; break }
            bat "timeout /t 3 /nobreak >nul"
          }

          if (ok) {
            echo "Smoke test PASSED — showing head of output.html"
            bat "type output.html | more"
            archiveArtifacts artifacts: 'output.html', allowEmptyArchive: true
          } else {
            echo "Smoke test FAILED — printing container status and logs"
            bat "docker ps -a --filter name=%CONTAINER% --format \"table {{.ID}}\\t{{.Names}}\\t{{.Status}}\\t{{.Ports}}\" || echo no-container"
            bat "docker logs %CONTAINER% --tail 200 || echo no-logs"
            archiveArtifacts artifacts: 'output.html', allowEmptyArchive: true
            error("Smoke test failed: could not reach http://localhost:%HOST_PORT%")
          }
        }
      }
    }
  }

  post {
    always { echo "Pipeline finished." }
  }
}
