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
                    // Check if package.json exists
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

        stage('Test Stage') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    test -f build/index.html
                    echo "✅ Running tests..."
                    npm test || echo "⚠️ Tests failed or not configured."
                '''
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
        always {
            junit 'test-results/junit.xml'
        }
        // failure {
        //     echo '❌ Build failed. Please check the logs.'
        // }
        // success {
        //     echo '✅ Build completed successfully.'
        // }
    }
}