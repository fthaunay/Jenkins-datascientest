pipeline {
    agent any
    environment { 
      DOCKER_ID = "dstdockerhub"
      DOCKER_IMAGE = "datascientestapi"
      DOCKER_TAG = "v.${BUILD_ID}.0" 
    }
    stages {
        stage('Building') {
          steps {
                sh 'pip install -r requirements.txt'
          }
        }
        stage('Testing') {
          steps {
                sh 'python3 -m unittest'
          }
        }
          stage('Deploying') {
          steps{
            script {
              sh '''
              # docker rm -f jenkins
              sudo systemctl status docker
              ls -l /var/run/docker.sock
              sudo chown root:docker /var/run/docker.sock
              sudo chmod 660 /var/run/docker.sock
              groups $USER
              sudo docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
              sudo docker run -d -p 8001:8001 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
              '''
            }
          }
        }
          stage('User Acceptance') {
            steps{
                input (message: "Proceed to push to main", ok: "Yes") 
            }
          }
          stage('Pushing and Merging'){
            parallel {
                stage('Pushing Image') {
                  environment {
                      DOCKERHUB_CREDENTIALS = credentials('docker_jenkins')
                  }
                  steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                      sh 'docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG'
                  }
                }
            stage('Merging') {
              steps {
                echo 'Merging done'
              }
            }
          }
        }
    }
    post {
      always {
        sh 'docker logout'
      }
    }
}