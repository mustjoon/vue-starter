pipeline {
  tools {nodejs "node"}
  agent any
    environment {
    registryCredential = 'dockerhub'
    dockerImage = ''
    PACKAGE_JSON = readJSON file: './package.json'
    HOST = "167.71.56.231"
    VERSION = "${PACKAGE_JSON['version']}"
    APP_NAME = "${PACKAGE_JSON['name']}"
    CONTAINER_NAME = "${APP_NAME}"
    HOST_PORT = "8090"
    CONTAINER_PORT = "80"
    GIT_URL = "${PACKAGE_JSON['repository']['url']}"
    registry = "mustjoon/$APP_NAME"
  }
 
  stages {

    stage('Setup') {
      steps {
        script {
          git "$GIT_URL"
        }
        script {
          sh ('npm install')
        }
       
      }
    }
     
    stage('Test') {
      steps {
         sh 'npm run test:unit'
      }
    }

      stage('Building image') {
      steps{
        script {
          dockerImage = docker.build registry + ":$VERSION"
        }
      }
    }
    stage('Deploy Image') {
      steps{
        script {
          docker.withRegistry( '', registryCredential ) {
            dockerImage.push()
          }
        }
      }
    }      
    stage('Deploy to Server') {
      steps{
        script {
          sh("docker network inspect home >/dev/null 2>&1 || \
              docker network create --driver bridge home")
          sh("docker pull $registry:$VERSION")
          sh("(docker stop $CONTAINER_NAME > /dev/null && echo Stopped container $CONTAINER_NAME && \
            docker rm $CONTAINER_NAME ) 2>/dev/null || true")
          sh("docker run -d --health-cmd='curl -f http://localhost:$CONTAINER_PORT/version.json'  --health-interval=5s  --network 'home' --publish $HOST_PORT:$CONTAINER_PORT --name='$CONTAINER_NAME'  $registry:$VERSION")
         
        }
      }
    }
     stage('Check that currently running version is correct') {
        steps{
           retry(5) {
              script {
                sleep(1)
                sh("bash ./bin/health-check.sh -v '$VERSION' -h '$HOST' -p '$HOST_PORT'")
              }
           }
        }
     }
  }
}