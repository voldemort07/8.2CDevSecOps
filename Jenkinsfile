// SIT223 7.1C - Part 2, Task 1 (DevSecOps with SonarCloud.io)
// Extends the Part 1 Task 2 pipeline with a SonarCloud Analysis stage.
// Windows agent: steps use "bat", and failures are swallowed with "|| exit /b 0".
// Maaheer Pandya

pipeline {
    agent any

    triggers {
        pollSCM('H/2 * * * *')
    }

    environment {
        SCANNER_URL = 'https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-7.0.2.4839-windows-x64.zip'
        SCANNER_ZIP = 'sonar-scanner.zip'
        SCANNER_DIR = 'sonar-scanner-7.0.2.4839-windows-x64'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/voldemort07/8.2CDevSecOps.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'npm install --legacy-peer-deps || exit /b 0'
            }
        }

        stage('Run Tests') {
            steps {
                bat 'npm test || exit /b 0'
            }
        }

        stage('Generate Coverage Report') {
            steps {
                // Produces coverage/lcov.info, which SonarCloud reads for coverage metrics.
                bat 'npm run coverage || exit /b 0'
            }
        }

        stage('NPM Audit (Security Scan)') {
            steps {
                bat 'npm audit || exit /b 0'
                bat 'npm audit --json > npm-audit-report.json || exit /b 0'
            }
        }

        stage('SonarCloud Analysis') {
            steps {
                // The token never appears in the console - Jenkins masks it, and it is
                // passed on the command line rather than committed to the properties file.
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    bat '''
                        @echo off
                        if not exist "%SCANNER_DIR%\\bin\\sonar-scanner.bat" (
                            echo Downloading SonarScanner CLI...
                            powershell -NoProfile -Command "Invoke-WebRequest -Uri '%SCANNER_URL%' -OutFile '%SCANNER_ZIP%'"
                            echo Extracting SonarScanner CLI...
                            powershell -NoProfile -Command "Expand-Archive -Path '%SCANNER_ZIP%' -DestinationPath '.' -Force"
                        )
                        echo Running SonarCloud analysis...
                        "%SCANNER_DIR%\\bin\\sonar-scanner.bat" -Dsonar.token=%SONAR_TOKEN%
                    '''
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'npm-audit-report.json', allowEmptyArchive: true
        }
        success {
            echo 'Analysis uploaded. Open the SonarCloud dashboard to review the findings.'
        }
    }
}
