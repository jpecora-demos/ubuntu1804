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
    - name: buildkitd
      image: moby/buildkit:master
      args:
        - --addr
        - unix:///run/buildkit/buildkitd.sock
        - --addr
        - tcp://0.0.0.0:1234
        - --tlscacert
        - /certs/ca.pem
        - --tlscert
        - /certs/cert.pem
        - --tlskey
        - /certs/key.pem
      securityContext:
        privileged: true
'''
        }
    }
    environment {
        DOCKER_REGISTRY = "docker.io"
        APP_REPO = "${DOCKER_REGISTRY}/jpecora716"
        APP_NAME = "demo"
        DOCKER_HOST = "unix:///run/buildkit/buildkitd.sock"
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
        stage('Download_WizCLI') {
            steps {
                // Download_WizCLI
                sh 'echo "Downloading wizcli..."'
                sh 'curl -o wizcli https://downloads.wiz.io/wizcli/latest/wizcli-linux-amd64'
                sh 'chmod +x wizcli'
            }
        }
        stage('Auth_With_Wiz') {
            steps {
                // Auth with Wiz
                sh 'echo "Authenticating to the Wiz API..."'
                withCredentials([usernamePassword(credentialsId: 'wizcli', usernameVariable: 'ID', passwordVariable: 'SECRET')]) {
                sh './wizcli auth --id $ID --secret $SECRET'}
            }
        }
        stage('Scan') {
            steps {
                // Scanning the image
                sh 'ls -l /run/buildkit'
                sh 'echo "Scanning the image using wizcli..."'
                sh './wizcli docker scan --image ${APP_REPO}/${APP_NAME}:${GIT_COMMIT}'
            }
        }
    }
}
