pipeline {
    agent {
        kubernetes {yaml '''
apiVersion: v1
kind: Pod
metadata:
  name: wizagent
  namespace: jenkins
spec:
  serviceAccountName: k8s-builder
  volumes:
  - name: buildkit
    emptyDir: {}
  - name: optbin
    emptyDir: {}
  - name: cache
    emptyDir: {}
  containers:
    - name: jnlp
      image: 'jenkins/inbound-agent'
      args: ['\$(JENKINS_SECRET)', '\$(JENKINS_NAME)']
      volumeMounts:
      - name: optbin
        mountPath: /opt/bin

    - name: docker
      image: docker:23.0.4-cli
      tty: true
      volumeMounts:
      - name: buildkit
        mountPath: /run/buildkit
      - name: cache
        mountPath: /var/lib/docker

    - name: buildkitd
      image: moby/buildkit:stable
      args:
        - --addr
        - unix:///run/buildkit/buildkitd.sock
        - --addr
        - tcp://0.0.0.0:1234
      volumeMounts:
      - name: buildkit
        mountPath: /run/buildkit
      - name: cache
        mountPath: /var/lib/buildkit
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
                    sh "apk add --no-cache ca-certificates"
                    // NEW AND IMPROVED WAIT LOOP
                    sh '''
                        echo "Waiting for buildkitd daemon to be responsive..."
                        until docker buildx version > /dev/null 2>&1; do
                          echo -n "."
                          sleep 1
                        done
                        echo "\nBuildkitd is responsive!"
                    '''
                    sh "docker buildx build -t ${APP_REPO}/${APP_NAME}:${GIT_COMMIT} ."
                }
            }
        }
        stage('Download_WizCLI') {
            steps {
                sh 'curl -o /opt/bin/wizcli https://downloads.wiz.io/wizcli/latest/wizcli-linux-amd64'
                sh 'chmod +x /opt/bin/wizcli'
            }
        }
        stage('Auth_With_Wiz') {
            steps {
                container("docker") {
                    withCredentials([usernamePassword(credentialsId: 'wizcli', usernameVariable: 'ID', passwordVariable: 'SECRET')]) {
                    sh '/opt/bin/wizcli auth --id $ID --secret $SECRET'}
                }
            }
        }
        stage('Scan') {
            steps {
                // Scanning the image
                container("docker") {
                    sh '/opt/bin/wizcli docker scan --image ${APP_REPO}/${APP_NAME}:${GIT_COMMIT}'
                }
            }
        }
    }
}
