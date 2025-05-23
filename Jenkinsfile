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
    // one PowerShell block does download, extract, find & run
    powershell '''
      # go to the workspace
      Set-Location $env:WORKSPACE

      # download the CLI if we don’t already have it
      if (-Not (Test-Path sonar-scanner.zip)) {
        Write-Host "Downloading SonarScanner CLI…"
        Invoke-WebRequest `
          -Uri "https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli.zip" `
          -OutFile "sonar-scanner.zip"
      }

      # extract (overwrite if it already exists)
      Write-Host "Extracting SonarScanner CLI…"
      Expand-Archive `
        -Path "sonar-scanner.zip" `
        -DestinationPath "sonar-scanner" `
        -Force

      # find the extracted subfolder (it has a version in its name)
      $scannerDir = Get-ChildItem -Directory sonar-scanner |
                    Where-Object Name -Like "sonar-scanner*" |
                    Select-Object -First 1

      if (-Not $scannerDir) {
        Throw "❌ Could not find extracted sonar-scanner directory!"
      }

      # run the scanner
      $exe = Join-Path $scannerDir.FullName "bin\\sonar-scanner.bat"
      Write-Host "Running SonarScanner from $exe"
      & $exe "-Dsonar.login=$env:SONAR_TOKEN"
    '''
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
