pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = '3e4b59b5-5c89-4f0e-acc1-016e330a2746'
        NETLIFY_AUTH_TOKEN = credentials('Netlify-Token')
    }

    stages {
        stage('Checkout') {
            steps {
                echo '🔄 Checking out source code...'
                checkout scm
            }
        }

        stage('Static Code Analysis') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "🔍 Running ESLint for static analysis..."
                    npm ci
                    if [ -f .eslintrc.js ] || [ -f .eslintrc.json ]; then
                      npx eslint . || echo "⚠️ Lint warnings found."
                    else
                      echo "⚠️ ESLint config not found. Skipping linting."
                    fi
                '''
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                script {
                    if (!fileExists('package.json')) {
                        error("package.json not found. Cannot proceed with npm build.")
                    }
                }
                sh '''
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo "✅ Running unit tests..."
                            npm test || echo "⚠️ Unit tests failed, but continuing to post results..."
                        '''
                    }
                    post {
                        always {
                            junit allowEmptyResults: true, testResults: 'jest-results/*.xml'
                        }
                    }
                }

                stage('E2E Playwright') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo "📦 Installing dependencies (including Playwright)..."
                            npm ci
                            npm install serve
                            node_modules/.bin/serve -s build  & 
                            sleep 10

                            echo "🚀 Running Playwright E2E tests..."
                            npx playwright test --reporter=html

                            echo "📁 Listing test output..."
                            ls -la playwright-report/
                        '''
                    }
                    post {
                        always {
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: false,
                                icon: '',
                                keepAll: false,
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Playwright HTML Report',
                                useWrapperFileDirectly: true
                            ])
                        }
                    }
                }
            }
        }
        stage('Archive Build Artifacts') {
            steps {
                script {
                    if (fileExists('build')) {
                        echo '📦 Archiving build artifacts...'
                        archiveArtifacts artifacts: 'build/**', allowEmptyArchive: true
                    } else {
                        echo '⚠️ Build directory not found. Skipping archive.'
                    }
                }
            }
        }
        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                script {
                    if (!fileExists('package.json')) {
                        error("package.json not found. Cannot proceed with npm build.")
                    }
                }
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to Production Site ID $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
    }

    post {
        failure {
            echo '❌ Build failed. Please check the logs.'
        }
        success {
            echo '✅ Build completed successfully.'
        }
    }
}