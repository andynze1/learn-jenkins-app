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

        stage('Archive Build Artifacts') {
            when {
                expression { fileExists('dist') } // Assuming your build output is in /dist
            }
            steps {
                echo '📦 Archiving build artifacts...'
                archiveArtifacts artifacts: 'dist/**', allowEmptyArchive: true
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