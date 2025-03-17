pipeline {
    agent any
    
    environment {
        // Set environment variables
        DEPLOY_SERVER = 'ubuntu@3.110.75.239' // Replace with EC2 username and IP address
        DEPLOY_DIR = '/home/ubuntu/Nodejs-Jenkins/app.js' // Path to your app on EC2
        NODE_VERSION = '18' // Node.js version to use
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    echo "Cloning repository from GitHub..."
                    git branch: 'main', url: 'https://github.com/hariharan-k21/Nodejs-Jenkins.git' // Replace with your Git repository
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    echo "Installing Node.js and dependencies..."
                    try {
                        sh '''
                        set -e  # Exit on error
                        curl -sL https://deb.nodesource.com/setup_${NODE_VERSION}.x | sudo -E bash -
                        sudo apt-get install -y nodejs
                        npm install
                        '''
                    } catch (Exception e) {
                        echo "Error in Install Dependencies: ${e.getMessage()}"
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    echo "Running build step (if needed)..."
                    try {
                        sh 'npm run build' // Optional, remove if not needed
                    } catch (Exception e) {
                        echo "Build failed: ${e.getMessage()}"
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    echo "Deploying to EC2..."
                    sshagent(['2144284a-1a0b-4e11-b0aa-282e4c97cac7']) { // Use the credentials ID set in Jenkins
                        try {
                            sh '''
                            set -e  # Exit on error

                            # Copy files to EC2
                            echo "Copying files to EC2..."
                            scp -r * ${DEPLOY_SERVER}:${DEPLOY_DIR} || { echo 'SCP failed'; exit 1; }

                            # SSH into EC2 and restart the application (ensure PM2 or similar is installed on EC2)
                            echo "Connecting to EC2 and deploying..."
                            ssh ${DEPLOY_SERVER} <<EOF
                            set -e  # Exit on error

                            # Navigate to deployment directory
                            cd ${DEPLOY_DIR}

                            # Install npm dependencies
                            npm install || { echo 'npm install failed'; exit 1; }

                            # Check if pm2 is installed, if not, install it
                            if ! command -v pm2 &> /dev/null; then
                                echo "PM2 not found. Installing PM2..."
                                sudo npm install pm2 -g
                            fi

                            # Stop app if it's running, then start it
                            pm2 stop app || true  # Stop app if it's running
                            pm2 start app.js --name app  # Start app using pm2 (or your preferred process manager)
                            EOF
                            '''
                        } catch (Exception e) {
                            echo "Deployment failed: ${e.getMessage()}"
                            currentBuild.result = 'FAILURE'
                            throw e
                        }
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
