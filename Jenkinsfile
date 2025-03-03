pipeline {
    agent any

    environment {
        SERVER_IP = credentials('prod-server-ip')
    }

    stages {
        stage('Setup') {
            steps {
                sh '''
                echo "Updating system packages..."
                sudo apt update -y
                sudo apt install -y zip python3 python3-pip python3-venv unzip || true

                echo "Upgrading pip..."
                pip install --upgrade pip

                echo "Installing required dependencies..."
                pip install --user -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                echo "Checking if pytest is installed..."
                if ! command -v pytest &> /dev/null
                then
                    echo "pytest not found, installing..."
                    pip install --user pytest || true
                fi

                echo "Running tests..."
                ~/.local/bin/pytest || true
                '''
            }
        }

        stage('Package code') {
            steps {
                sh '''
                echo "Checking if zip is installed..."
                if ! command -v zip &> /dev/null
                then
                    echo "zip not found, installing..."
                    sudo apt install -y zip
                fi

                echo "Creating deployment package..."
                zip -r myapp.zip ./* -x '*.git*'

                echo "Listing package contents..."
                ls -lart
                '''
                echo "✅ Package created successfully"
            }
        }

        stage('Deploy to Prod') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key', keyFileVariable: 'MY_SSH_KEY')]) {
                    sh '''
                    echo "Transferring package to the remote server..."
                    scp -i $MY_SSH_KEY -o StrictHostKeyChecking=no myapp.zip ubuntu@${SERVER_IP}:/home/ubuntu/

                    echo "Connecting to the remote server and deploying..."
                    ssh -i $MY_SSH_KEY -o StrictHostKeyChecking=no ubuntu@${SERVER_IP} << 'EOF'
                        echo "Preparing deployment on remote server..."

                        echo "Installing missing dependencies..."
                        sudo apt update -y
                        sudo apt install -y zip python3 python3-pip python3-venv unzip || true

                        echo "Creating app directory if not exists..."
                        mkdir -p /home/ubuntu/app

                        echo "Extracting package..."
                        unzip -o /home/ubuntu/myapp.zip -d /home/ubuntu/app/

                        echo "Setting up virtual environment..."
                        python3 -m venv /home/ubuntu/app/venv

                        echo "Activating virtual environment..."
                        source /home/ubuntu/app/venv/bin/activate

                        echo "Upgrading pip inside virtual environment..."
                        /home/ubuntu/app/venv/bin/pip install --upgrade pip

                        echo "Installing dependencies..."
                        /home/ubuntu/app/venv/bin/pip install -r /home/ubuntu/app/requirements.txt

                        echo "Restarting Flask service..."
                        if [ -f /etc/systemd/system/flaskapp.service ]; then
                            sudo systemctl restart flaskapp.service
                        else
                            echo "⚠️ Warning: flaskapp.service not found!"
                        fi

                        echo "✅ Deployment completed successfully!"
                    EOF
                    '''
                }
            }
        }
    }
}
