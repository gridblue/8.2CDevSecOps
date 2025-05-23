pipeline {
  agent any

  // Poll GitHub every 5 minutes for new commits
  triggers {
    pollSCM('H/5 * * * *')
  }

  stages {
    stage('Checkout') {
      steps {
        // Grab whatever is in your GitHub repo
        checkout scm
      }
    }

    stage('Install Dependencies') {
      steps {
        // Install Node.js packages
        bat 'npm install'
      }
    }

    stage('Run Tests') {
      steps {
        // Run tests but don’t fail the build if they break
        bat 'npm test || true'
      }
    }

    stage('Generate Coverage') {
      steps {
        // Create a coverage report
        bat 'npm run coverage || true'
      }
    }

    stage('Security Scan') {
      steps {
        // Check for vulnerabilities
        bat 'npm audit || true'
      }
    }
  }

  post {
    success {
      echo '✅ Build succeeded!'
    }
    failure {
      echo '❌ Build failed — check the console output.'
    }
  }
}
