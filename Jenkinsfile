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
echo ${cov-idir}
echo "start Cpature ....."
cov-build --dir ${cov-idir} mvn clean package -Dmaven.test.skip=true
echo "list capture ....."
coverity list
echo "start analyze ....."
cov-analyze --dir ${cov-idir} --strip-path `pwd`
echo ${COV_URL}
cov-commit-defects --dir ${cov-idir} --url ${COV_URL} --stream ${COV_STREAM} --auth-key-file ${COV_AUTH_KEY_PATH}'''
            }

          }
        }

        stage('BlackDuck') {
          environment {
            ENV_NAME = 'clean package -Dmaven.test.skip=true'
            DETECT_PLUGIN_ESCAPING = 'false'
          }
          steps {
            synopsys_detect(detectProperties: '--blackduck.trust.cert=true  --detect.project.version.name=2 --detect.project.name=WebGoat --detect.cleanup=true --blackduck.offline.mode=true  --detect.blackduck.signature.scanner.snippet.matching=NONE --detect.detector.search.depth=5 --detect.maven.build.command=${ENV_NAME} --detect.excluded.directories=idir ', returnStatus: true)
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
        step([$class: 'CodeDxPublisher', analysisName: 'Build #${BUILD_NUMBER}', analysisResultConfiguration: [failureOnlyNew: false, failureSeverity: 'None', numBuildsInGraph: 0, policyBreakBuildBehavior: 'NoAction', unstableOnlyNew: false, unstableSeverity: 'None'], key: 'api-key:hkZpD1V2WEpsMjC-cGjr5C24B2HDtvbp9FR4HQ5f', projectId: '5', selfSignedCertificateFingerprint: '', sourceAndBinaryFiles: '**', url: 'http://10.107.85.95:81/codedx'])
      }
    }

  }
  tools {
    maven 'maven3.9'
    jdk 'java17'
  }
}