pipeline {
    agent { label 'red' }

    environment {
        DEPLOY_DIR = "/opt/myflaskapp"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/carina030308/master.git'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    sudo mkdir -p ${DEPLOY_DIR}
                    sudo cp app.py requirements.txt ${DEPLOY_DIR}/
                    sudo /opt/myflaskapp/venv/bin/pip install -r ${DEPLOY_DIR}/requirements.txt
                    sudo chown -R flaskapp:flaskapp ${DEPLOY_DIR}
                    sudo systemctl restart myflaskapp
                    sudo systemctl reload nginx
                '''
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