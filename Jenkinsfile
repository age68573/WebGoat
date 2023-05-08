pipeline {
  agent {
    node {
      label 'master'
    }

  }
  tools {
    maven 'Apache Maven 3.8.7'
  }
  stages {
    stage('echo') {
      steps {
        sh 'echo \'123456\''
      }
    }

  }
}
