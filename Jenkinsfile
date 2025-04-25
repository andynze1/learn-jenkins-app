pipeline {
    agent any

    stages {
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
                    echo "📁 Listing files before build"
                    ls -la

                    echo "🧾 Node version:"
                    node --version

                    echo "🧾 NPM version:"
                    npm --version

                    echo "📦 Installing dependencies..."
                    npm ci

                    echo "🏗️ Building the app..."
                    npm run build
                    
                    echo "📁 Listing files after build"
                    ls -la
                '''
            }
        }

        stage ('Tests') {
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
                            test -f build/index.html
                            echo "✅ Running unit tests..."
                            npm test || echo "⚠️ Tests failed or not configured."
                        '''
                    }
                    post {
                        always {
                            // ✅ Collect Playwright JUnit test results
                            junit 'jest-results/*.xml'
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
                            ls -la test-results/
                        '''
                    }
                    post {
                        always {
                            // ✅ Collect Playwright JUnit test results.
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
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
   
    }

    post {
       /* always {
            // ✅ Collect Playwright JUnit test results
            junit 'jest-results/*.xml'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        } */  

        failure {
            echo '❌ Build failed. Please check the logs.'
        }
        success {
            echo '✅ Build completed successfully.'
        }
    }
}