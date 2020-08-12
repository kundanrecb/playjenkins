properties = null
def loadProperties() {
    node {
        checkout scm
        properties = readProperties file: 'pipeline.properties'
        echo "Reading properties for Docker Registry project ${properties.docker_repo_url}"
    }
}
pipeline {
  environment {
    loadprop = loadProperties()
    //registry = "192.168.203.17:5000/justme/myweb"
    registry = "${properties.docker_repo_url}"
    dockerImage = ""
  }

  agent any

  stages {

    stage('Build image') {
      steps{
        script {
          dockerImage = docker.build registry + ":$BUILD_NUMBER"
        }
      }
    }

    stage('Push Image') {
      steps{
        script {
          docker.withRegistry( "" ) {
            dockerImage.push()
          }
        }
      }
    }

    stage('Deploy App') {
      agent { label 'kubepod' }
      steps {
        script {
          kubernetesDeploy(configs: "myweb.yaml", kubeconfigId: "mykubeconfig")
        }
      }
    }

  }

}
