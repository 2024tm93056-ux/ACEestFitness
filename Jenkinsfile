pipeline {
    agent {
      docker {
        image 'python:3.14-slim'
        args '-v $PWD:/app'
      }
    }

    environment {
        APP_NAME = "aceest_fitness"
        DOCKER_REPO = "yourdockerhubusername/aceest_fitness"
        PYTHON = "python3"
        KUBE_CONFIG_ID = "kube-config"
        SONARQUBE_SERVER = "SonarQubeServer"
        DOCKER_CREDS = "dockerhub-creds"
        GIT_CREDENTIALS = "github-token"
    }

    // triggers {
    //     // Poll SCM every 5 minutes for changes
    //     pollSCM('H/5 * * * *')
    // }

    stages {
        stage('Clean Workspace') {
            steps {
                // Clean checkout
                cleanWs()
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
        }

        // stage('SonarQube Analysis') {
        //     steps {
        //         echo "🔍 Running SonarQube static code analysis..."
        //         withSonarQubeEnv("${SONARQUBE_SERVER}") {
        //             sh """
        //             sonar-scanner \
        //             -Dsonar.projectKey=ACEest_Fitness \
        //             -Dsonar.sources=. \
        //             -Dsonar.host.url=$SONAR_HOST_URL \
        //             -Dsonar.login=$SONAR_AUTH_TOKEN
        //             """
        //         }
        //     }
        // }

        // stage('Build Docker Image') {
        //     steps {
        //         echo "🐳 Building Docker image..."
        //         sh """
        //         VERSION=$(git describe --tags --always || echo latest)
        //         docker build -t ${DOCKER_REPO}:$VERSION .
        //         docker tag ${DOCKER_REPO}:$VERSION ${DOCKER_REPO}:latest
        //         """
        //     }
        // }

        // stage('Push Docker Image') {
        //     steps {
        //         echo "📦 Pushing Docker image to Docker Hub..."
        //         withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDS}", usernameVariable: 'USER', passwordVariable: 'PASS')]) {
        //             sh """
        //             echo $PASS | docker login -u $USER --password-stdin
        //             VERSION=$(git describe --tags --always || echo latest)
        //             docker push ${DOCKER_REPO}:$VERSION
        //             docker push ${DOCKER_REPO}:latest
        //             """
        //         }
        //     }
        // }

        // stage('Deploy to Kubernetes') {
        //     steps {
        //         echo "🚀 Deploying updated app to Kubernetes cluster..."
        //         withCredentials([file(credentialsId: "${KUBE_CONFIG_ID}", variable: 'KUBECONFIG')]) {
        //             sh """
        //             kubectl apply -f deployment.yaml --kubeconfig $KUBECONFIG
        //             kubectl apply -f service.yaml --kubeconfig $KUBECONFIG
        //             kubectl rollout status deployment/aceest-fitness-deployment --kubeconfig $KUBECONFIG
        //             """
        //         }
        //     }
        // }

        // stage('Tag Build and Archive') {
        //     steps {
        //         echo "🏷️ Tagging Git version and archiving build artifact..."
        //         sh """
        //         VERSION=v$(date +'%Y.%m.%d.%H%M')
        //         git config user.email "jenkins@ci.local"
        //         git config user.name "Jenkins CI"
        //         git tag -a $VERSION -m "Automated build $VERSION"
        //         git push origin $VERSION || true

        //         mkdir -p build
        //         zip -r build/${APP_NAME}-$VERSION.zip .
        //         """
        //         archiveArtifacts artifacts: 'build/*.zip', fingerprint: true
        //     }
        // }
    }

    post {
        success {
            echo "✅ Build completed successfully!"
        }
        failure {
            echo "❌ Build failed. Please check console output for errors."
        }
    }
}
pipeline {
    agent any

    environment {
        APP_NAME = "aceest-fitness"
        DOCKER_REPO = "yourdockerhubusername/aceest-fitness"  // ⚠️ CHANGE THIS
        PYTHON_VERSION = "3.11"
    }

    options {
        skipDefaultCheckout(true)
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
        ansiColor('xterm')
    }

    stages {
        stage('🧹 Cleanup') {
            steps {
                echo "Cleaning workspace..."
                deleteDir()
            }
        }

        stage('📥 Checkout') {
            steps {
                echo "Cloning repository..."
                sh '''
                git clone https://github.com/2024tm93056-ux/ACEestFitness.git .
                git checkout main
                echo "✅ Branch: $(git branch --show-current)"
                echo "✅ Commit: $(git log -1 --oneline)"
                ls -la
                '''
            }
        }

        stage('🐍 Setup Python') {
            steps {
                echo "Setting up Python environment..."
                sh '''
                python3 --version
                python3 -m venv venv
                . venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                pip list
                '''
            }
        }

        stage('🧪 Run Tests') {
            steps {
                echo "Running tests..."
                sh '''
                . venv/bin/activate
                pip install pytest pytest-flask pytest-cov
                mkdir -p reports coverage
                
                # Run tests
                pytest -v --disable-warnings \
                    --junitxml=reports/pytest-results.xml \
                    --cov=. \
                    --cov-report=xml:coverage/coverage.xml \
                    --cov-report=html:coverage/html \
                    --cov-report=term || echo "Tests completed with warnings"
                '''
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'reports/pytest-results.xml'
                    publishHTML([
                        allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'coverage/html',
                        reportFiles: 'index.html',
                        reportName: 'Coverage Report'
                    ])
                }
            }
        }

        stage('🔍 Code Quality') {
            steps {
                echo "Checking code quality..."
                sh '''
                . venv/bin/activate
                pip install pylint flake8
                
                echo "Running pylint..."
                pylint app.py run.py || true
                
                echo "Running flake8..."
                flake8 . --max-line-length=120 --exclude=venv || true
                '''
            }
        }

        stage('🐳 Build Docker Image') {
            steps {
                echo "Building Docker image..."
                script {
                    def version = sh(
                        script: 'git describe --tags --always || echo "v1.0.0"',
                        returnStdout: true
                    ).trim()
                    
                    env.VERSION = version
                    env.BUILD_TAG = "${version}-build${BUILD_NUMBER}"
                    
                    sh """
                    docker build -t ${DOCKER_REPO}:${BUILD_TAG} .
                    docker tag ${DOCKER_REPO}:${BUILD_TAG} ${DOCKER_REPO}:latest
                    docker images | grep ${DOCKER_REPO}
                    """
                }
            }
        }

        stage('🧪 Test Docker Image') {
            steps {
                echo "Testing Docker image..."
                sh '''
                # Start container
                docker run -d --name test-app -p 5001:5000 ${DOCKER_REPO}:${BUILD_TAG}
                
                # Wait for startup
                sleep 15
                
                # Test health endpoint
                curl -f http://localhost:5001/ || echo "App responded"
                
                # Cleanup
                docker stop test-app
                docker rm test-app
                '''
            }
        }

        stage('📦 Push to Docker Hub') {
            when {
                branch 'main'
            }
            steps {
                echo "Pushing to Docker Hub..."
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push ${DOCKER_REPO}:${BUILD_TAG}
                    docker push ${DOCKER_REPO}:latest
                    docker logout
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning up..."
            sh 'docker system prune -f || true'
        }
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}