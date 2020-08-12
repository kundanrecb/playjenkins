properties = null
def loadProperties() {
    node {
        checkout scm
        properties = readProperties file: 'pipeline.properties'
        echo "Loaded properties file: ${properties.docker_repo_url}"        
    }
}
def readProperty(propName) {
    node {
        echo " readProperties called, propName: ${propName} & properties:  ${properties}"
        if (properties == null) {
            echo "in if"
            loadProperties()
            echo " Now properties:  ${properties}"
        }
        echo "after if: ${properties.docker_repo_url}"
        return "${properties.docker_repo_url}"
    }
}
pipeline {
  environment {
    loadprop = readProperty("docker_repo_url")
    //registry = "192.168.203.17:5000/justme/myweb"
    //registry = "${properties.docker_repo_url}"
    dockerImage = ""
  }

  agent any

  stages {

    stage('Build image') {
      steps{
        script {
            echo "Reading properties for Docker Registry project ${loadprop}"
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
