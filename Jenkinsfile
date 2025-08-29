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
        DOCKER_HOST = "unix:///usr/lib/systemd/user/podman.socket"
    }
    stages {
PP_REPO}/${APP_NAME}:${GIT_COMMIT}
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
                container("podman") {
                    //sh "apk add --no-cache ca-certificates"
                    // NEW AND IMPROVED WAIT LOOP
                    //sh "docker buildx build -t ${APP_REPO}/${APP_NAME}:${GIT_COMMIT} ."
                    sh "ls -l"
                    sh "podman build -t ${APP_REPO}/${APP_NAME}:${GIT_COMMIT} ."
                }
            }
        }
        stage('Download_WizCLI') {
            steps {
                container("podman") {
                    sh 'curl -o /opt/bin/wizcli https://downloads.wiz.io/wizcli/latest/wizcli-linux-amd64'
                    sh 'chmod +x /opt/bin/wizcli'
                }
            }
        }
        stage('Auth_With_Wiz') {
            steps {
                container("podman") {
                    withCredentials([usernamePassword(credentialsId: 'wizcli', usernameVariable: 'ID', passwordVariable: 'SECRET')]) {
                    sh '/opt/bin/wizcli auth --id $ID --secret $SECRET'}
                }
            }
        }
        stage('Scan') {
            steps {
                // Scanning the image
                container("podman") {
                    sh 'podman images'
                    sh '/opt/bin/wizcli docker scan --image ${APP_REPO}/${APP_NAME}:${GIT_COMMIT}'
                }
            }
        }
    }
}
