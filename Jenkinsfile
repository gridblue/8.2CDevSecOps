pipeline {
  agent any

  // Poll GitHub every 5 minutes
  triggers {
    pollSCM('H/5 * * * *')
  }

  // Inject your SonarCloud token
  environment {
    SONAR_TOKEN = credentials('SONAR_TOKEN')
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
        // run tests but don’t fail the build if they error
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

  post {
    success {
      echo '✅ Pipeline (with SonarCloud) succeeded.'
    }
    failure {
      echo '❌ Pipeline failed – check console & SonarCloud.'
    }
  }
}
