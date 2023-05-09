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

    stage('Run Coverity') {
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
            ENV_NAME = '\'clean package -Dmaven.test.skip=true\''
          }
          steps {
            synopsys_detect(detectProperties: '--blackduck.trust.cert=true  --detect.project.version.name=1 --detect.project.name=WebGoat --detect.cleanup=true --blackduck.offline.mode=false  --detect.blackduck.signature.scanner.snippet.matching=SNIPPET_MATCHING --detect.detector.search.depth=5  --detect.excluded.directories=idir --detect.maven.build.command="${ENV_NAME}"', returnStatus: true)
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