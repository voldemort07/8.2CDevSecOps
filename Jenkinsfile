// SIT223 7.1C - Part 1, Task 2 (DevSecOps Basics)
// Runs npm security tests over the snyk-labs/nodejs-goof project.
// Windows agent: steps use "bat", and failures are swallowed with "|| exit /b 0".
// Maaheer Pandya

pipeline {
    agent any

    triggers {
        pollSCM('H/2 * * * *')
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/voldemort07/8.2CDevSecOps.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                // nodejs-goof pins deliberately old, vulnerable packages, so modern npm
                // needs --legacy-peer-deps to resolve the tree at all.
                bat 'npm install --legacy-peer-deps || exit /b 0'
            }
        }

        stage('Run Tests') {
            steps {
                // Allows the pipeline to continue despite test failures
                bat 'npm test || exit /b 0'
            }
        }

        stage('Generate Coverage Report') {
            steps {
                // Ensure coverage report exists
                bat 'npm run coverage || exit /b 0'
            }
        }

        stage('NPM Audit (Security Scan)') {
            steps {
                // This will show known CVEs in the output
                bat 'npm audit || exit /b 0'
                // Keep a machine-readable copy of the scan as a build artefact.
                bat 'npm audit --json > npm-audit-report.json || exit /b 0'
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'npm-audit-report.json', allowEmptyArchive: true
        }
    }
}
