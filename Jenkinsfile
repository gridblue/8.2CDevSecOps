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

pipeline {
  /* … your stages … */

  post {
    success {
      echo '✅ Build succeeded — sending success email w/ log.'
      emailext(
        subject: "✔️ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """\
          Hi,

          Your job '${env.JOB_NAME}' (#${env.BUILD_NUMBER}) succeeded.
          See attached console log for details.

          URL: ${env.BUILD_URL}
        """,
        to: 'you@domain.com',
        attachLog: true,                 // <-- attaches the entire console log
        mimeType: 'text/plain'
      )
    }

    failure {
      echo '❌ Build failed — sending failure email w/ log.'
      emailext(
        subject: "❌ FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """\
          Hi,

          Your job '${env.JOB_NAME}' (#${env.BUILD_NUMBER}) failed.
          See attached console log for the error stack.

          URL: ${env.BUILD_URL}
        """,
        to: 'you@domain.com',
        attachLog: true,                 // <-- same here
        mimeType: 'text/plain'
      )
    }
  }
}

