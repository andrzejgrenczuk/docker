pipeline {
  agent any

  environment {
    GIT_HOST = 'git-server.example.com:8082' // zamień na host/IP widoczny z agenta
    REPO_URL = "http://${GIT_HOST}/git/root/docker"
    DOCKER_IMAGE = "ruwanthilakshika/nodeapp-cuban"
  }

  stages {
    stage('SCM Checkout') {
      steps {
        retry(3) {
          checkout([$class: 'GitSCM',
            branches: [[name: '*/main']],
            userRemoteConfigs: [[url: env.REPO_URL, credentialsId: 'GIT_CRED_ID']]
          ])
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          def buildTag = env.BUILD_NUMBER
          bat "docker build --no-cache -t ${DOCKER_IMAGE}:${buildTag} ."
        }
      }
    }

    stage('Login to Docker Hub') {
      steps {
        withCredentials([string(credentialsId: 'test-dockerhubpass', variable: 'DOCKER_HUB_PASS')]) {
          script {
            def loginSuccess = bat(returnStatus: true, script: "docker login -u ruwanthilakshika -p ${DOCKER_HUB_PASS}")
            if (loginSuccess != 0) {
              error("Docker login failed! Check credentials.")
            }
          }
        }
      }
    }

    stage('Push Image') {
      steps {
        script {
          def buildTag = env.BUILD_NUMBER
          bat "docker push ${DOCKER_IMAGE}:${buildTag}"
        }
      }
    }
  }

  post {
    always {
      bat 'docker logout'
    }
    failure {
      echo "Build failed. Check logs for details."
    }
  }
}

