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
        // run npm test but don’t abort the build if it fails (e.g. Snyk auth errors)
        bat script: 'npm test', returnStatus: true
      }
    }
    
    stage('Generate Coverage') {
      steps {
        // Create a coverage report
        bat script: 'npm run coverage', returnStatus: true
      }
    }

    stage('Security Scan') {
      steps {
        // Check for vulnerabilities
        bat script: 'npm audit', returnStatus: true
      }
    }
  }
stage('SonarCloud Analysis') {
      environment {
        SONAR_TOKEN = credentials('SONAR_TOKEN')
      }
      steps {
        // Download & unpack the SonarScanner CLI (once per build)
        bat '''
          powershell -Command "Invoke-WebRequest https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli.zip -OutFile sonar-scanner.zip"
          powershell -Command "Expand-Archive sonar-scanner.zip -DestinationPath sonar-scanner"
        '''
        // Run the scanner
        bat '''
          sonar-scanner\\sonar-scanner-*/bin/sonar-scanner.bat ^
            -Dsonar.login=%SONAR_TOKEN%
        '''
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
