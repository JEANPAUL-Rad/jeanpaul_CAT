pipeline {
  agent any

  environment {
    DOCKERHUB_REPOSITORY = 'jeanpaul60001/jeanpaul-cat'
    IMAGE_TAG = 'v6'
    COMPOSE_FILE = 'compose.yml'
  }

  options {
    timestamps()
    disableConcurrentBuilds()
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Validate') {
      steps {
        sh 'docker compose -f "${COMPOSE_FILE}" config --quiet'
      }
    }

    stage('Build') {
      steps {
        sh 'docker build -t "${DOCKERHUB_REPOSITORY}:${IMAGE_TAG}" -t "${DOCKERHUB_REPOSITORY}:latest" .'
      }
    }

    stage('Test') {
      steps {
        sh 'docker run --rm --name jeanpaul-cat-test -d -p 127.0.0.1:3000:3000 "${DOCKERHUB_REPOSITORY}:${IMAGE_TAG}"'
        sh 'for i in $(seq 1 30); do curl --fail --silent --show-error http://127.0.0.1:3000/ >/dev/null && exit 0; sleep 2; done; exit 1'
        sh 'curl --fail --silent --show-error http://127.0.0.1:3000/metrics | grep -q juiceshop_version_info'
      }
      post {
        always {
          sh 'docker rm -f jeanpaul-cat-test >/dev/null 2>&1 || true'
        }
      }
    }

    stage('Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_TOKEN')]) {
          sh 'echo "$DOCKERHUB_TOKEN" | docker login -u "$DOCKERHUB_USER" --password-stdin'
          sh 'docker push "${DOCKERHUB_REPOSITORY}:${IMAGE_TAG}"'
          sh 'docker push "${DOCKERHUB_REPOSITORY}:latest"'
        }
      }
    }

    stage('Deploy') {
      steps {
        sh 'docker compose -f "${COMPOSE_FILE}" pull juice-shop'
        sh 'docker compose -f "${COMPOSE_FILE}" up -d --force-recreate juice-shop nginx prometheus grafana'
      }
    }
  }

  post {
    always {
      sh 'docker logout >/dev/null 2>&1 || true'
    }
  }
}
