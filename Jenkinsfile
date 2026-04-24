pipeline {
  agent {
    kubernetes {
      label 'jenkins-worker'
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: docker
    image: docker:24-dind
    securityContext:
      privileged: true
    volumeMounts:
      - name: docker-storage
        mountPath: /var/lib/docker
  - name: kubectl
    image: bitnami/kubectl:latest
    command: ['cat']
    tty: true
  volumes:
  - name: docker-storage
    emptyDir: {}
"""
    }
  }

  environment {
    REGISTRY = "registry.digitalocean.com"
    IMAGE = "meu-app"
    TAG = "${env.BUILD_NUMBER}"
    FULL_IMAGE = "${REGISTRY}/${IMAGE}:${TAG}"
  }

  options {
    disableConcurrentBuilds()
    timestamps()
  }

  triggers {
    githubPush()
  }

  stages {

    stage('Checkout') {
      steps {
        container('docker') {
          git branch: 'master',
              credentialsId: 'git-credentials',
              url: 'https://github.com/marcosribeiro15/meu-app.git'
        }
      }
    }

    stage('Build') {
      steps {
        container('docker') {
          sh """
            dockerd-entrypoint.sh &
            sleep 10
            docker build -t ${FULL_IMAGE} .
          """
        }
      }
    }

    stage('Push') {
      steps {
        container('docker') {
          withCredentials([usernamePassword(credentialsId: 'registry-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
            sh """
              docker login ${REGISTRY} -u $USER -p $PASS
              docker push ${FULL_IMAGE}
            """
          }
        }
      }
    }

    stage('Deploy') {
      steps {
        container('kubectl') {
          sh """
            kubectl set image deployment/meu-app meu-app=${FULL_IMAGE} -n default
            kubectl rollout status deployment/meu-app -n default
          """
        }
      }
    }
  }
}
