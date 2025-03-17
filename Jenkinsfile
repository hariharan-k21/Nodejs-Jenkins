pipeline {
    agent any
    
    environment {
        // Set environment variables
        DEPLOY_SERVER = 'ubuntu@3.110.75.239' // Replace with EC2 username and IP address
        DEPLOY_DIR = '/home/ubuntu/Nodejs-Jenkins/app.js' // Path to your app on EC2
        NODE_VERSION = '18' // Node.js version to use
        JAVA_OPTS = "-Dorg.jenkinsci.plugins.durabletask.BourneShellScript.HEARTBEAT_CHECK_INTERVAL=86400"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/hariharan-k21/Nodejs-Jenkins.git' // Replace with your Git repository
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Skipping installation, Node.js already installed on the system.'   
            }
        }

        stage('Build') {
            steps {
                script {
                    // If your app has a build step (e.g., TypeScript or Webpack)
                    sh 'npm run build' // Optional, remove if not needed
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    // Deploy the Node.js application to EC2
                    sshagent(['2144284a-1a0b-4e11-b0aa-282e4c97cac7']) { // Use the credentials ID set in Jenkins
                        sh '''
                        # Copy files to EC2
                        scp -r * ${DEPLOY_SERVER}:${DEPLOY_DIR}

                        # SSH into EC2 and restart the application (ensure PM2 or similar is installed on EC2)
                        ssh ${DEPLOY_SERVER} <<EOF
                        cd ${DEPLOY_DIR}
                        npm install
                        pm2 stop app || true  # Stop app if it's running
                        pm2 start app.js --name app  # Start app using pm2 (or your preferred process manager)
                        EOF
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment Successful!'
        }
        failure {
            echo 'Deployment Failed!'
        }
    }
}
