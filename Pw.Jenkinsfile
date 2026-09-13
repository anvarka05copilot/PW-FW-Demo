pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Replace with your actual repository URL
                git url: 'https://your-git-repo.git', branch: 'main'
            }
        }
        stage('Install Dependencies') {
            steps {
                // Ensure Node.js dependencies are installed
                sh 'npm ci'
            }
        }
        stage('Run Playwright Tests') {
            agent {
                docker {
                    // Use the official Playwright image (check for the latest version)
                    image 'mcr.microsoft.com/playwright:v1.48.0-noble'
                    // Run as root to avoid permission issues in CI
                    args '-u root'
                }
            }
            steps {
                // Run the tests in headless mode, which is standard for CI
                sh 'npx playwright test'
            }
            post {
                always {
                    // Publish JUnit test results for clear visibility
                    junit allowEmptyResults: true, testResults: 'results/junit.xml'
                    // Archive the HTML report for debugging
                    archiveArtifacts artifacts: 'pw-report/**', allowEmptyArchive: true
                }
            }
        }
    }
}