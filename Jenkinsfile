pipeline {
    agent {
        kubernetes {yaml '''
apiVersion: v1
kind: Pod
metadata:
  namespace: jenkins
spec:
  serviceAccountName: k8s-builder
  containers:
    - name: docker
      image: docker:23.0.4-cli
      tty: true
'''
        }
    }
    environment {
        DOCKER_REGISTRY = "docker.io"
        APP_REPO = "${DOCKER_REGISTRY}/jpecora716"
        APP_NAME = "demo"
    }
    stages {

        stage('Setup') {
            steps {
                container("docker") {
                    // Configure the kubernetes builder
                    sh "docker buildx create --name k8s-builder --driver kubernetes --driver-opt replicas=1 --use"
                    // Registry login
                    //sh 'echo ${DOCKER_PASSWORD} | docker login ${DOCKER_REGISTRY} --username ${DOCKER_USERNAME} --password-stdin'
                }
            }
        }

        stage('Build CI image') {
            when {
                allOf {
                    branch "main"
                    not {
                        buildingTag()
                    }
                }
                
            }
            steps {
                container("docker") {
                    sh "docker buildx build -t ${APP_REPO}/${APP_NAME}:${GIT_COMMIT} ."
                }
            }
        }
    }
}
