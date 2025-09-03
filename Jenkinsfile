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
                    sh "podman build -t ${APP_REPO}/${APP_NAME}:${GIT_COMMIT} ."
                }
            }
        }
        stage('Download_WizCLI') {
            steps {
                container("podman") {
                    sh 'curl -o wizcli https://downloads.wiz.io/wizcli/latest/wizcli-linux-amd64'
                    sh 'chmod +x wizcli'
                }
            }
        }
        stage('Auth_With_Wiz') {
            steps {
                container("podman") {
                    withCredentials([usernamePassword(credentialsId: 'wizcli', usernameVariable: 'ID', passwordVariable: 'SECRET')]) {
                    sh './wizcli auth --id $ID --secret $SECRET'}
                }
            }
        }
        stage('Scan') {
            steps {
                // Scanning the image
                container("podman") {
                    catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE', message: 'Wiz found critical findings, review before deployment') {
                        sh 'podman system service --time 120 &'
                        sh './wizcli docker scan --image ${APP_REPO}/${APP_NAME}:${GIT_COMMIT} --driver mountWithLayers 2> /dev/null && exit 1'
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                sh 'exit 0'
            }
        }
    }
}
