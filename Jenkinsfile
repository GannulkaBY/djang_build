pipeline {
  agent {
    kubernetes {
        containerTemplate {
        name 'hadolint'
        image 'hadolint/hadolint:latest-debian'
        ttyEnabled true
        command 'cat'
        }
    }
  }
  stages {
    stage('Clone repository') { 
        steps { 
            git url: 'https://github.com/GannulkaBY/django',
            credentialsId: "github",
            branch: "main"         
        }
    }
    stage('Lint dockerfiles') {
        steps {
            container('hadolint') {
                sh 'hadolint */Dockerfile | tee -a hadolint_lint.txt'
            }
        }
           
    }
  }
  post {
    always {
        archiveArtifacts 'hadolint_lint.txt'
        }
    success {
        slackSend (color: '#008000', message: "SUCCESSFUL: Job '(${env.BUILD_URL})")
        }
    failure {
        slackSend (color: '#FF0000', message: "FAILED: Job '(${env.BUILD_URL})")
        }
    }
}
