pipeline {
    agent any

    environment {
        VENV_DIR = "venv"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/neamulkabiremon/jenkins-single-server-deployment.git'
            }
        }

        stage('Setup') {
            steps {
                sh '''
                # Ensure Python and pip are available
                which python3 || exit 1
                which pip || exit 1
                
                # Create and activate virtual environment if not exists
                if [ ! -d "$VENV_DIR" ]; then
                    python3 -m venv $VENV_DIR
                fi
                source $VENV_DIR/bin/activate
                
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
                # Activate virtual environment and run pytest
                source $VENV_DIR/bin/activate
                python3 -m pytest
                '''
            }
        }

        stage('Package Code') {
            steps {
                sh '''
                # Create a ZIP archive of the application
                zip -r myapp.zip . -x "*.git*" "$VENV_DIR/*"
                '''
            }
        }

        stage('Deploy to Prod') {
            steps {
                sh '''
                echo "Deploying application..."
                # Add deployment commands here, such as copying files, restarting services, etc.
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs for errors."
        }
    }
}
