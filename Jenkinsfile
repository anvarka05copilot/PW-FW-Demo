pipeline {
    agent any

    tools {
        // Must match the name in Global Tool Configuration
        nodejs 'NodeJS'
    }

    parameters {
        choice(
            name: 'Script',
            choices: ['regression', 'sanity', 'smoke-allure'],
            description: 'Select the npm script to execute'
        )
        booleanParam(
            name: 'INSTALL_BROWSERS',
            defaultValue: true,
            description: 'Run npx playwright install --with-deps before tests'
        )
    }

    environment {
        CI = 'true'
        // Cache npm packages inside the workspace to speed up builds
        npm_config_cache = "${WORKSPACE}/.npm"
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/anvarka05copilot/PW-FW-Demo.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                // npm ci is faster & deterministic when package-lock.json exists
                sh 'npm ci || npm install'
            }
        }

        stage('Install Playwright Browsers') {
            when {
                expression { params.INSTALL_BROWSERS }
            }
            steps {
                sh 'npx playwright install --with-deps'
            }
        }

        stage('Run Playwright Tests') {
            steps {
                // Pass the selected script from parameters
                sh "npm run ${params.Script}"
            }
        }
    }

    post {
        always {
            // Publish JUnit results if your Playwright config outputs them
            junit allowEmptyResults: true, testResults: 'results/junit.xml'
            
            // Default Playwright HTML report → downloadable artifact
            archiveArtifacts artifacts: 'playwright-report/**',
                             allowEmptyArchive: true

            // Archive Playwright HTML report and raw Allure results
            archiveArtifacts artifacts: 'playwright-report/**', allowEmptyArchive: true
            archiveArtifacts artifacts: 'allure-results/**', allowEmptyArchive: true

            // Generate Allure report using the Jenkins Allure plugin
            allure includeProperties: false, results: [[path: 'allure-results']]
        }
        failure {
            echo 'Pipeline failed — check the Playwright report and Allure results above.'
        }
        cleanup {
            // Clean workspace after build to free disk space
            cleanWs()
        }
    }
}