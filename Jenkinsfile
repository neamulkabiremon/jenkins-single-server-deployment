pipeline {
    agent any

    environment {
        SERVER_IP = credentials('prod-server-ip')  // Jenkins Secret Text Credential
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    checkout([$class: 'GitSCM',
                        branches: [[name: '*/main']],
                        userRemoteConfigs: [[url: 'https://github.com/neamulkabiremon/jenkins-single-server-deployment.git']]
                    ])
                }
            }
        }

        stage('Setup') {
            steps {
                sh '''
                set -e
                echo "🔧 Checking Python and Pip installation..."
                which python3 || exit 1
                which pip || exit 1

                echo "🐍 Setting up virtual environment..."
                if [ ! -d "venv" ]; then
                    python3 -m venv venv
                fi

                echo "📦 Activating virtual environment and installing dependencies..."
                . venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                set -e
                echo "🧪 Running tests..."
                . venv/bin/activate
                pip install pytest  # Ensure pytest is installed
                pytest
                '''
            }
        }

        stage('Package Code') {
            steps {
                sh '''
                set -e
                echo "📦 Packaging application..."
                zip -r myapp.zip . -x "*.git*" "venv/*"
                ls -lart
                '''
            }
        }

        stage('Deploy to Prod') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key', keyFileVariable: 'MY_SSH_KEY', usernameVariable: 'USERNAME')]) {
                    sh '''
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
                        if [ ! -d "venv" ]; then
                            python3 -m venv venv
                        fi
                        . venv/bin/activate
                        pip install --upgrade pip
                        pip install -r requirements.txt

                        # Ensure systemd service exists
                        SERVICE_FILE="/etc/systemd/system/flaskapp.service"
                        if [ ! -f "$SERVICE_FILE" ]; then
                            echo "⚠️ Service file not found. Creating flaskapp.service..."
                            sudo bash -c 'cat > /etc/systemd/system/flaskapp.service << EOL
[Unit]
Description=Flask Application
After=network.target

[Service]
User=ec2-user
WorkingDirectory=/home/ec2-user/app
ExecStart=/home/ec2-user/app/venv/bin/python /home/ec2-user/app/app.py
Restart=always

[Install]
WantedBy=multi-user.target
EOL'
                            sudo systemctl daemon-reload
                            sudo systemctl enable flaskapp.service
                        fi

                        # Restart the Flask app service
                        sudo systemctl restart flaskapp.service
                        echo "✅ Deployment successful!"
EOF
                    '''
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
