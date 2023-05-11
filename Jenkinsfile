pipeline {
  agent {
    node {
      label 'master'
    }

  }
  stages {
    stage('echo') {
      parallel {
        stage('echo') {
          steps {
            sh '''echo \'start\'
java --version'''
          }
        }

        stage('Check pom.xml') {
          steps {
            fileExists 'pom.xml'
          }
        }

      }
    }

    stage('Run Scan') {
      parallel {
        stage('Run Coverity') {
          steps {
            withCoverityEnvironment(coverityInstanceUrl: 'http://10.107.85.94:8080', createMissingProjectsAndStreams: true, credentialsId: 'Coverity94', projectName: 'WebGoat', streamName: 'WebGoat', viewName: 'Outstanding Issues') {
              sh '''mvn --version
'''
            }

          }
        }

        stage('BlackDuck') {
          environment {
            ENV_NAME = 'clean package -Dmaven.test.skip=true'
            DETECT_PLUGIN_ESCAPING = 'false'
          }
          steps {
            synopsys_detect(detectProperties: '--blackduck.trust.cert=true  --detect.project.version.name=v0.1 --detect.project.name=WebGoat --detect.cleanup=true --blackduck.offline.mode=false  --detect.blackduck.signature.scanner.snippet.matching=NONE --detect.detector.search.depth=0 --detect.maven.build.command=${ENV_NAME} --detect.excluded.directories=idir ', returnStatus: true)
          }
        }

      }
    }

    stage('Coverity results') {
      steps {
        coverityIssueCheck(coverityInstanceUrl: 'http://10.107.85.94:8080', credentialsId: 'Coverity94', markUnstable: true, viewName: 'Outstanding Issues', returnIssueCount: true, projectName: 'WebGoat')
      }
    }

    stage('CodeDX') {
      steps {
        step([$class: 'CodeDxPublisher', analysisName: 'Build #${BUILD_NUMBER}', analysisResultConfiguration: [failureOnlyNew: false, failureSeverity: 'None', numBuildsInGraph: 0, policyBreakBuildBehavior: 'NoAction', unstableOnlyNew: false, unstableSeverity: 'None'],baseBranchName: 'main', key: 'api-key:hkZpD1V2WEpsMjC-cGjr5C24B2HDtvbp9FR4HQ5f', projectId: '5', selfSignedCertificateFingerprint: '', sourceAndBinaryFiles: '**',targetBranchName: 'blueocean', url: 'http://10.107.85.95:81/codedx'])
      }
    }

    stage('Nexus') {
      steps {
        nexusArtifactUploader(artifacts: [[artifactId: 'webgoat', classifier: '', file: 'target/webgoat-2023.5-SNAPSHOT.jar', type: 'jar']], credentialsId: 'jenkins-user', groupId: 'org.owasp.webgoat', nexusUrl: '10.107.72.6:31368', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-central-repo', version: '2023.5-SNAPSHOT')
      }
    }

    stage('Docker Build') {
      steps {
        sh 'docker build -t age68573/webgoat .'
      }
    }

    stage('Docker Push') {
      steps {
        script {
          withCredentials([string(credentialsId: 'dockerhub', variable: 'dockerhub')]) {
            sh "docker login -u age68573 -p ${dockerhub}"
            sh "docker push age68573/webgoat"
            docker rmi $(docker images -f "dangling=true" -q)
          }
        }

      }
    }

  }
  tools {
    maven 'maven3.9'
    jdk 'java17'
  }
}
