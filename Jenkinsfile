pipeline {
    agent any

    environment {
        NODE_VERSION = '18.x'  // Specify the Node.js version your Angular app needs
        DEPLOY_SERVER = 'your-remote-server.com'  // Replace with your server address
        DEPLOY_PATH = '/var/www/html'  // Replace with your deployment path
        DEV_PORT = '4201'  // Custom port for ng serve
    }

    stages {
        stage('Setup') {
            steps {
                // Install Node.js
                script {
                    def nodeHome = tool name: 'NodeJS', type: 'jenkins.plugins.nodejs.tools.NodeJSInstallation'
                    env.PATH = "${nodeHome}/bin:${env.PATH}"
                }
                
                // Install dependencies
                sh 'npm install'
            }
        }

        stage('Build') {
            steps {
                // Build the Angular application for production
                sh 'npm run build'
            }
        }

        stage('Test') {
            steps {
                // Run unit tests
                sh 'npm run test -- --watch=false --browsers=ChromeHeadless'
            }
        }

        stage('Serve (Dev)') {
            steps {
                // Start Angular dev server on custom port for SSH tunnel testing
                sh 'nohup npx ng serve --port $DEV_PORT --disable-host-check --host 0.0.0.0 > dev-server.log 2>&1 &'
                echo "Angular dev server started on port $DEV_PORT. Use your SSH tunnel to access it."
            }
        }

        stage('Deploy') {
            steps {
                // Create the dist directory if it doesn't exist on the remote server
                sh """
                    ssh user@\${DEPLOY_SERVER} "mkdir -p \${DEPLOY_PATH}"
                """
                
                // Copy the built files to the remote server
                sh """
                    scp -r dist/angular-web-app-test/* user@\${DEPLOY_SERVER}:\${DEPLOY_PATH}/
                """
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Pipeline failed! Check the logs for details.'
        }
    }
}
