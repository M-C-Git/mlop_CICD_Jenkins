pipeline {
  agent any

  environment {
    DOCKERHUB_NS = 'mchenai'
    IMAGE_NAME   = 'fraud-detector'
    IMAGE_TAG    = 'latest'
    FULL_IMAGE   = "${DOCKERHUB_NS}/${IMAGE_NAME}:${IMAGE_TAG}"
  }

  stages {
    stage('Clone Repository') {
      steps {
        git url: 'https://github.com/mohamadsolouki/MLOps.git', branch: 'main'
      }
    }

    stage('Build Docker Image') {
      steps {
        bat "docker build -t %IMAGE_NAME% ."
      }
    }

    stage('Run Unit Tests') {
      steps {
        // Mount workspace so tests run against your repo
        bat "docker run --rm -v %CD%:/app -w /app %IMAGE_NAME% pytest -q tests/"
      }
    }

    stage('Push to Docker Registry') {
      steps {
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
            bat "docker tag %IMAGE_NAME% %FULL_IMAGE%"
            bat "docker push %FULL_IMAGE%"
          }
        }
      }
    }

    stage('Deploy Model') {
      steps {
        bat "docker rm -f fraud-detector-app 2>nul || ver>nul"
        bat "docker run -d --name fraud-detector-app -p 8000:8080 %FULL_IMAGE%"
      }
    }
  }
}
