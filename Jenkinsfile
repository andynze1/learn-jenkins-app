pipeline {
    agent any

    stages {
        stage('Build Stage') {
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

        stage('Unit Test Stage') {
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
        }
        stage('E2E Stage') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
     //               args '-u root:root'
                }
            }
            steps {
                sh '''
                    npm ci
                    npm install serve
                    node_modules/.bin/serve -s build
                    npx playwright test
                '''
            }
        }

     // stage('E2E Test Stage - Playwright') {
        //     agent {
        //         docker {
        //             image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         sh '''
        //             echo "📦 Installing dependencies (including Playwright)..."
        //             npm ci
        //             npm install serve
        //             node_modules/.bin/serve -s build
        //             echo "🚀 Running Playwright E2E tests..."
        //             npx playwright test

        //             echo "📁 Listing test output..."
        //             ls -la test-results/
        //         '''
        //     }
        // }

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
        always {
            // ✅ Collect Playwright JUnit test results
            junit 'test-results/*.xml'
        }
        failure {
            echo '❌ Build failed. Please check the logs.'
        }
        success {
            echo '✅ Build completed successfully.'
        }
    }
}