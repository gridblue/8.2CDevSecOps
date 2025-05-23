pipeline {
  agent any

  triggers {
    githubPush()          // on every GitHub push

  }

  environment {
    RECIPIENT = 'tejussrao@gmail.com'
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Install Dependencies') {
      steps { bat 'npm install' }
    }

    stage('Run Tests') {
      steps {
        // run tests and tee output into test.log (Windows)
        bat script: 'npm test > test.log 2>&1', returnStatus: true
      }
      post {
        always {
          emailext(
            subject: "Stage ‘Run Tests’ ${currentBuild.currentResult}",
            body: """
              Stage **Run Tests** has finished with status: ${currentBuild.currentResult}  
              See attached `test.log` for details.
            """,
            to: "${env.RECIPIENT}",
            attachmentsPattern: 'test.log',
            mimeType: 'text/plain'
          )
        }
      }
    }

    stage('Generate Coverage') {
      steps {
        bat script: 'npm run coverage > coverage.log 2>&1', returnStatus: true
      }
    }

    stage('Security Scan') {
      steps {
        // audit output into security.log
        bat script: 'npm audit > security.log 2>&1', returnStatus: true
      }
      post {
        always {
          emailext(
            subject: "Stage ‘Security Scan’ ${currentBuild.currentResult}",
            body: """
              Stage **Security Scan** has finished with status: ${currentBuild.currentResult}  
              See attached `security.log` for details.
            """,
            to: "${env.RECIPIENT}",
            attachmentsPattern: 'security.log',
            mimeType: 'text/plain'
          )
        }
      }
    }
  }

  post {
    // you can still send a final summary here if you like
    always {
      echo "Pipeline finished with ${currentBuild.currentResult}"
    }
  }
}
