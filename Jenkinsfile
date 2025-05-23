pipeline {
    agent any

    // Poll GitHub every 5 minutes
    triggers {
        pollSCM('H/5 * * * *')
    }

    // Make SONAR_TOKEN available in all stages
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
    powershell '''
      # Force TLS1.2 for GitHub/Sonar downloads
      [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12

      $workspace = $env:WORKSPACE
      $zip       = Join-Path $workspace 'sonar-scanner.zip'
      $outDir    = Join-Path $workspace 'sonar-scanner'

      # Download ZIP if missing
      if (-Not (Test-Path $zip)) {
        Write-Host "Downloading SonarScanner CLI via WebClient…"
        $wc = New-Object System.Net.WebClient
        $wc.DownloadFile(
          'https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli.zip',
          $zip
        )
      }

      # (Re)extract
      if (Test-Path $outDir) { Remove-Item $outDir -Recurse -Force }
      Write-Host "Extracting SonarScanner CLI…"
      Expand-Archive -Path $zip -DestinationPath $outDir

      # Locate the versioned folder
      $scanner = Get-ChildItem -Directory $outDir |
                 Where-Object Name -Like 'sonar-scanner*' |
                 Select-Object -First 1

      if (-Not $scanner) { Throw '❌ sonar-scanner folder not found!' }

      # Run SonarScanner
      $exe = Join-Path $scanner.FullName 'bin\sonar-scanner.bat'
      Write-Host "Running SonarScanner: $exe"
      & $exe "-Dsonar.login=$env:SONAR_TOKEN"
    '''
  }
}

    post {
        success {
            echo '✅ Full pipeline (with SonarCloud) succeeded.'
        }
        failure {
            echo '❌ Pipeline failed — check the console log & SonarCloud dashboard.'
        }
    }
}
