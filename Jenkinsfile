pipeline {
  agent {
    kubernetes {
      yamlFile 'build-agent.yaml'
      defaultContainer 'maven'
      idleMinutes 1
    }
  }
  stages {
    stage('Build') {
      parallel {
        stage('Compile') {
          steps {
            container(name: 'maven') {
              sh 'mvn compile'
            }
          }
        }
      }
    }

    stage('StaticAnalysis') { // Corrigido as aspas simples para aspas duplas
      parallel {
        stage('SCA') {
          steps {
            container(name: 'maven') {
              catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                sh 'mvn org.owasp:dependency-check-maven:check'
              }
            }
          }
          post {
            always {
              archiveArtifacts allowEmptyArchive: true, artifacts: 'target/dependency-check-report.html',
              fingerprint: true, onlyIfSuccessful: true
            }
          }
        }
      }
    }

    stage('Package') {
      parallel {
        stage('Create Jarfile') {
          steps {
            container(name: 'maven') {
              sh 'mvn package -DskipTests'
            }
          }
        }
      }
    }

    stage('OCI Image BnP') {
      steps {
        container(name: 'kaniko') {
          sh '/kaniko/executor -f `pwd`/Dockerfile -c `pwd` --insecure --skip-tls-verify --cache=true --destination=docker.io/danielmagevski/dso-demo'
        }
      }
    }

    stage('Deploy to Dev') {
      steps {
        sh 'echo done'
      }
    }
  }
}
