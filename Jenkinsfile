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
        // Run tests but don't fail the build if they error
        bat script: 'npm test', returnStatus: true
      }
    }

    stage('Generate Coverage') {
      steps {
        // Generate coverage report, but continue even if it errors
        bat script: 'npm run coverage', returnStatus: true
      }
    }

    stage('Security Scan') {
      steps {
        // Audit for vulnerabilities, but continue even if it errors
        bat script: 'npm audit', returnStatus: true
      }
    }
  }

  post {
    success {
      echo '✅ Build succeeded — emailing console log...'
      emailext(
        subject: "✔️ BUILD SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """\
          Hi,

          Your Jenkins job '${env.JOB_NAME}' (#${env.BUILD_NUMBER}) succeeded!
          
          View details at: ${env.BUILD_URL}

          Cheers,
          Jenkins
        """,
        to: 'you@domain.com',
        attachLog: true,
        mimeType: 'text/plain'
      )
    }
    failure {
      echo '❌ Build failed — emailing console log...'
      emailext(
        subject: "❌ BUILD FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """\
          Hi,

          Your Jenkins job '${env.JOB_NAME}' (#${env.BUILD_NUMBER}) failed.
          
          View details at: ${env.BUILD_URL}

          Please investigate the attached log.
        """,
        to: 'tejussrao@gmail.com',
        attachLog: true,
        mimeType: 'text/plain'
      )
    }
  }
}
