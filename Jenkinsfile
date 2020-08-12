properties = null
def loadProperties() {
    node {
        echo "In loadProperties()"
        checkout scm
        properties = readProperties file: 'pipeline.properties'
        echo "Finished loading properties: ${properties}"        
    }
}
def readDockerRepoUrl() {
    echo "In readDockerRepoUrl"
    echo "properties value: ${properties}"
    loadProperties()
     echo "loaded properties: ${properties}"
    def repoUrl = "${properties.docker_repo_url}"
    echo "after repoUrl assignment"
    echo "reporUrl: ${repoUrl}"
    //return repoUrl
    def tag = sh script: 'git rev-parse HEAD', returnStdout: true
    return repoUrl
}
def readPropertyDynamically(pName) {
    echo "In readDockerRepoUrlDynamic(), property to get is  ${pName}"
    
    if(properties == null){
        echo "In if block as properties value is : ${properties}"
        loadProperties()
    }else{
        echo "In else block as properties are already loaded"
    }
    def repoUrl = "properties"+".${pName}"
    echo "property to fetched is reporUrl: ${repoUrl}"
    return repoUrl
}
pipeline {
  environment {
    //loadprop = loadProperties()
    //registry = "192.168.203.17:5000/justme/myweb"
    registry = readPropertyDynamically('docker_repo_url')
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
