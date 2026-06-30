pipeline {
    agent { label 'red' }


    environment {
        REMOTE_HOST = "ec2-user@34.207.236.241"
        DEPLOY_DIR  = "/opt/myflaskapp"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/carina030308/master.git'
            }
        }

        stage('Package') {
            steps {
                sh 'tar -czf app.tar.gz app.py requirements.txt'
            }
        }

        stage('Deploy to RedHat') {
            steps {
                sshagent(['redhat-ssh-cred']) {
                    sh """
                        
                            sudo mkdir -p /opt/myflaskapp
                            sudo cp app.py requirements.txt /opt/myflaskapp/
                            sudo python3 -m venv ${DEPLOY_DIR}/venv || true &&
                            sudo /opt/myflaskapp/venv/bin/pip install -r /opt/myflaskapp/requirements.txt
                            sudo chown -R flaskapp:flaskapp /opt/myflaskapp
                            sudo systemctl restart myflaskapp
                            sudo systemctl reload nginx
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successfully!'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}