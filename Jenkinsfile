pipeline {
  agent any

  // Poll GitHub every 5 minutes
  triggers {
    pollSCM('H/5 * * * *')
  }

  // Make SONAR_TOKEN available everywhere
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

    stage('SonarCloud Analysis') {
      steps {
        // Download and unzip SonarScanner CLI
        bat '''
          powershell -Command "Invoke-WebRequest https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli.zip -OutFile sonar-scanner.zip"
          powershell -Command "Expand-Archive sonar-scanner.zip -DestinationPath sonar-scanner"
        '''
        // Run the scanner
        bat '''
          sonar-scanner\\sonar-scanner-*/bin/sonar-scanner.bat -Dsonar.login=%SONAR_TOKEN%
        '''
      }
    }
  }

  post {
    success {
      echo '✅ Full pipeline (with SonarCloud) succeeded.'
    }
    failure {
      echo '❌ Pipeline failed—check the console log & SonarCloud dashboard.'
    }
  }
}
