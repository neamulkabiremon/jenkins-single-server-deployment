pipeline {
    agent any

    environment {
        SERVER_IP = credentials('prod-server-ip')  // Jenkins Secret Text Credential
    }

    stages {
        stage('Setup') {
            steps {
                sh """
                set -e
                pip install -r requirements.txt
                """
            }
        }

        stage('Test') {
            steps {
                sh """
                set -e
                pytest
                """
            }
        }

        stage('Package Code') {
            steps {
                sh """
                set -e
                zip -r myapp.zip ./* -x '*.git*'
                ls -lart
                """
            }
        }

        stage('Deploy to Prod') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key', keyFileVariable: 'MY_SSH_KEY', usernameVariable: 'USERNAME')]) {
                    sh """
                    set -e
                    echo "🚀 Deploying app to production server: $SERVER_IP"

                    # Copy app package to the server
                    scp -i $MY_SSH_KEY -o StrictHostKeyChecking=no myapp.zip ${USERNAME}@${SERVER_IP}:/home/ec2-user/

                    # SSH into the server and deploy
                    ssh -i $MY_SSH_KEY -o StrictHostKeyChecking=no ${USERNAME}@${SERVER_IP} << 'EOF'
                        set -e
                        echo "📦 Extracting package..."
                        unzip -o /home/ec2-user/myapp.zip -d /home/ec2-user/app/
                        cd /home/ec2-user/app/

                        # Setup virtual environment
                        if [ ! -d 'venv' ]; then
                            python3 -m venv venv
                        fi
                        . venv/bin/activate  # Use '.' instead of 'source' for better compatibility

                        # Install dependencies
                        pip install -r requirements.txt

                        # Restart the Flask app service
                        sudo systemctl restart flaskapp.service
                        echo "✅ Deployment successful!"
EOF
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline executed successfully!"
        }
        failure {
            echo "❌ Pipeline failed! Check logs for errors."
        }
    }
}
