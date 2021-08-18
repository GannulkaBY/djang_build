pipeline {
  environment {
    registry = "gannagp"
    registryCredential = 'docker_registry'
  }
  agent {
        node {
            label 'master'
        }
    }
  stages {
    stage('Cloning Git') {
      steps {
        git url: 'https://github.com/GannulkaBY/django',
        credentialsId: "github",
        branch: "main" 
      }
    }
    stage ("lint dockerfile") {
        agent {
            docker {
                image 'hadolint/hadolint:latest-debian'
                reuseNode true
            }
        }
        steps {
            sh 'hadolint */Dockerfile | tee -a hadolint_lint.txt'
        }
        post {
            always {
                archiveArtifacts 'hadolint_lint.txt'
            }
        }
    }
    stage('Building images') {
        parallel {
            stage('Building nginx') {
                steps {
                    script {
                        dockerImageNginx = docker.build("$registry/reverse:${env.BUILD_ID}", "-f nginx/Dockerfile ./nginx")
                    }
                }
            }
            stage('Building django') {
                steps {
                    script {
                        dockerImageDjango = docker.build("$registry/django:${env.BUILD_ID}", "-f web/Dockerfile ./web")
                    }
                }
            }
        }
        
    }
    stage('Deploy images') {
        parallel {
            stage('Deploy nginx') {
                steps {
                    script {
                        docker.withRegistry( '', registryCredential ) {
                            dockerImageNginx.push()
                        }
                    }
                }
            }
            stage('Deploy django') {
                steps {
                    script {
                        docker.withRegistry( '', registryCredential ) {
                            dockerImageDjango.push()
                        }
                    }
                }
            }
        }
        
    }
    stage('Remove Unused docker image') {
      steps{
        sh "docker rmi $registry/reverse:$BUILD_NUMBER"
        sh "docker rmi $registry/django:$BUILD_NUMBER"
      }
    }
  }
  post {
    always {
        archiveArtifacts 'hadolint_lint.txt'
        }
    success {
        slackSend (color: '#008000', message: "SUCCESS: Job '(${env.BUILD_URL})")
        }
    failure {
        slackSend (color: '#FF0000', message: "FAIL: Job '(${env.BUILD_URL})")
        }
    }
}