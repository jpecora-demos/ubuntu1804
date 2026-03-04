pipeline {
    agent {
        kubernetes {yaml '''
apiVersion: v1
kind: Pod
metadata:
  name: wizagent
  namespace: jenkins
spec:
  containers:
    - name: podman
      image: quay.io/podman/stable
      tty: true
      securityContext:
        privileged: true
'''
        }
    }
    environment {
        DOCKER_REGISTRY = "docker.io"
        APP_REPO = "${DOCKER_REGISTRY}/jpecora716"
        APP_NAME = "demo"
        DOCKER_HOST = "unix:///run/podman/podman.sock"
    }
    stages {
        stage('Build CI image') {
            steps {
                container("podman") {
                    podman "build -t ${APP_REPO}/${APP_NAME}:${GIT_COMMIT} ."
                }
            }
        }
        stage('Download_WizCLI') {
            steps {
                container("podman") {
                    curl "-o wizcli https://downloads.wiz.io/v1/wizcli/latest/wizcli-linux-amd64"
                    chmod "+x wizcli"
                }
            }
        }
        stage('Scan') {
            steps {
                // Scanning the image
                container("podman") {
                    withCredentials([usernamePassword(credentialsId: 'wizcli', usernameVariable: 'WIZ_CLIENT_ID', passwordVariable: 'WIZ_CLIENT_SECRET')]) {
                        podman "system service --time 120 &"
                        ./wizcli "docker scan --image ${APP_REPO}/${APP_NAME}:${GIT_COMMIT} --driver mountWithLayers 2> /dev/null"
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                exit 0
            }
        }
    }
}
