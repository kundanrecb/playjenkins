properties = null
def loadProperties() {
    node {
        checkout scm
        properties = readProperties file: 'pipeline.properties'
        echo "Loaded properties file: ${properties.docker_repo_url}"        
    }
}
def readDockerRepoUrl() {
    echo "in readDockerRepoUrl"
    echo "properties value: ${properties}"
    loadProperties()
     echo "loaded properties: ${properties}"
    def repoUrl = "${properties.docker_repo_url}"
    echo "after repoUrl assignment"
    echo "reporUrl: ${repoUrl}"
    //return repoUrl
    def tag = sh script: 'git rev-parse HEAD', returnStdout: true
    return reporUrl
}
pipeline {
  environment {
    //loadprop = loadProperties()
    //registry = "192.168.203.17:5000/justme/myweb"
    registry = readDockerRepoUrl()
    dockerImage = ""
  }

  agent any

  stages {

    stage('Build image') {
      steps{
        script {
            echo "Reading properties for Docker Registry project ${registry}"
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
