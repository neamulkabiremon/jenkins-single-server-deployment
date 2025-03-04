pipeline {
    agent any

    environment {
        SERVER_IP = credentials('prod-server-ip')  // Jenkins Secret Text Credential
        VENV_DIR = "venv"
    }

    stages {
        stage('Setup') {
            steps {
                sh '''
                set -e
                echo "🔧 Checking Python and Pip installation..."
                which python3 || { echo "❌ Python3 not found!"; exit 1; }
                which pip || { echo "❌ Pip not found!"; exit 1; }

                echo "🐍 Setting up virtual environment..."
                if [ ! -d "$VENV_DIR" ]; then
                    python3 -m venv $VENV_DIR
                fi

                echo "📦 Activating virtual environment and installing dependencies..."
                source $VENV_DIR/bin/activate
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
                source $VENV_DIR/bin/activate
                pip install pytest  # Ensure pytest is installed
                pytest || { echo "❌ Tests failed!"; exit 1; }
                '''
            }
        }

        stage('Package Code') {
            steps {
                sh '''
                set -e
                echo "📦 Packaging application..."
                zip -r myapp.zip ./* -x '*.git*' "$VENV_DIR/*"
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
                        source venv/bin/activate

                        # Ensure dependencies are installed
                        pip install --upgrade pip
                        pip install -r requirements.txt

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
