pipeline {
    agent any

    environment {
        SERVER_IP = credentials('prod-server-ip')
        VENV_DIR = "venv"
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
                which python3 || exit 1
                which pip || exit 1

                # Create virtual environment if not exists
                if [ ! -d "$VENV_DIR" ]; then
                    python3 -m venv $VENV_DIR
                fi

                # Activate virtual environment
                . $VENV_DIR/bin/activate

                # Upgrade pip
                pip install --upgrade pip

                # Install dependencies
                pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                set -e
                . $VENV_DIR/bin/activate
                pip install pytest  # Ensure pytest is installed
                python3 -m pytest  # Run tests
                '''
            }
        }

        stage('Package Code') {
            steps {
                sh '''
                set -e
                zip -r myapp.zip . -x "*.git*" "$VENV_DIR/*"
                ls -lart
                '''
            }
        }

        stage('Deploy to Prod') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key', keyFileVariable: 'MY_SSH_KEY', usernameVariable: 'username')]) {
                    sh '''
                    set -e
                    scp -i $MY_SSH_KEY -o StrictHostKeyChecking=no myapp.zip ${username}@${SERVER_IP}:/home/ec2-user/
                    ssh -i $MY_SSH_KEY -o StrictHostKeyChecking=no ${username}@${SERVER_IP} << EOF
                        set -e
                        unzip -o /home/ec2-user/myapp.zip -d /home/ec2-user/app/
                        cd /home/ec2-user/app/

   
