pipeline {
    agent any

    environment {
        APP_NAME = "aceest_fitness"
        DOCKER_REPO = "yourdockerhubusername/aceest_fitness"
        PYTHON = "python3"
        KUBE_CONFIG_ID = "kube-config"
        SONARQUBE_SERVER = "SonarQubeServer"
        DOCKER_CREDS = "dockerhub-creds"
        GIT_CREDENTIALS = "github-token"
    }

    triggers {
        // Poll SCM every 5 minutes for changes
        pollSCM('H/5 * * * *')
    }

    stages {

        stage('Checkout Source') {
            steps {
                echo "🔹 Checking out code from GitHub repository..."
                git branch: 'main',
                    credentialsId: "${GIT_CREDENTIALS}",
                    url: 'https://github.com/<your-github-username>/ACEest_Fitness.git'
            }
        }

        stage('Setup Python Environment') {
            steps {
                echo "🔹 Setting up virtual environment and installing dependencies..."
                sh """
                ${PYTHON} -m venv venv
                . venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt pytest pytest-flask
                """
            }
        }

        stage('Run Unit Tests') {
            steps {
                echo "🧪 Running Pytest test cases..."
                sh """
                . venv/bin/activate
                mkdir -p reports
                pytest -v --disable-warnings --junitxml=reports/pytest-results.xml
                """
            }
            post {
                always {
                    junit 'reports/pytest-results.xml'
                }
            }
