pipeline {
    agent any

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
                        scp -o StrictHostKeyChecking=no app.tar.gz ${REMOTE_HOST}:/tmp/
                        ssh -o StrictHostKeyChecking=no ${REMOTE_HOST} '
                            sudo mkdir -p ${DEPLOY_DIR} &&
                            sudo tar -xzf /tmp/app.tar.gz -C ${DEPLOY_DIR} &&
                            sudo python3 -m venv ${DEPLOY_DIR}/venv || true &&
                            sudo ${DEPLOY_DIR}/venv/bin/pip install -r ${DEPLOY_DIR}/requirements.txt &&
                            sudo chown -R flaskapp:flaskapp ${DEPLOY_DIR} &&
                            sudo systemctl restart myflaskapp &&
                            sudo systemctl reload nginx
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}