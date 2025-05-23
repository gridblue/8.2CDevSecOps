pipeline {
  agent any

  // Poll GitHub every 5 minutes
  triggers {
    pollSCM('H/5 * * * *')
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    stage('Install Dependencies') {
      steps {
        bat 'npm install'
      }
    }
    stage('Run Tests') {
      steps {
        bat script: 'npm test', returnStatus: true
      }
    }
    stage('Generate Coverage') {
      steps {
        bat script: 'npm run coverage', returnStatus: true
      }
    }
    stage('Security Scan') {
      steps {
        bat script: 'npm audit', returnStatus: true
      }
    }
    // ← SonarCloud stage removed
  }

  post {
    success {
      echo '✅ Build succeeded — sending success email.'
      emailext(
        subject: "✔️ BUILD SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """\
          Hi,

          The Jenkins job '${env.JOB_NAME}' (#${env.BUILD_NUMBER}) succeeded!

          • URL: ${env.BUILD_URL}
          • Commit: ${env.GIT_COMMIT ?: 'N/A'}

          Cheers,
          Jenkins
        """,
        to: 'your.email@domain.com'
      )
    }
    failure {
      echo '❌ Build failed — sending failure email.'
      emailext(
        subject: "❌ BUILD FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """\
          Hi,

          The Jenkins job '${env.JOB_NAME}' (#${env.BUILD_NUMBER}) has failed.

          • URL: ${env.BUILD_URL}
          • Check console output for details.

          Please investigate.
        """,
        to: 'your.email@domain.com'
      )
    }
  }
}
