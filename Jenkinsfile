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

    stage('SonarCloud Analysis') {
      steps {
        powershell """
          # Force TLS1.2
          [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12

          \$ws     = \$env:WORKSPACE
          \$zip    = Join-Path \$ws 'sonar-scanner.zip'
          \$outDir = Join-Path \$ws 'sonar-scanner'

          if (-Not (Test-Path \$zip)) {
            Write-Host 'Downloading SonarScanner CLI…'
            \$wc = New-Object System.Net.WebClient
            \$wc.DownloadFile(
              'https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli.zip',
              \$zip
            )
          }

          if (Test-Path \$outDir) { Remove-Item \$outDir -Recurse -Force }
          Write-Host 'Extracting SonarScanner CLI…'
          Expand-Archive -Path \$zip -DestinationPath \$outDir

          \$scanner = Get-ChildItem -Directory \$outDir |
                     Where-Object Name -Like 'sonar-scanner*' |
                     Select-Object -First 1

          if (-Not \$scanner) { Throw 'sonar-scanner folder not found' }

          \$exe = Join-Path (Join-Path \$scanner.FullName 'bin') 'sonar-scanner.bat'
          Write-Host "Running SonarScanner at \$exe"
          & \$exe "-Dsonar.login=\$env:SONAR_TOKEN"
        """
      }
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
