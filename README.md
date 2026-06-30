# Hello project with github codespace

1. Create a repo in github and chose github codespace.

crate file "app.py" and paste the content:

-----------------------

from flask import Flask
app = Flask(__name__)

@app.route("/")
def home():
    return "Hello from Flask on RedHat via Jenkins!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)

--------------------------

 Create "requirements.txt" file and paste content:

--------------------------

flask
gunicorn

--------------------------
Run:
pip install -r requirements.txt

python app.py


Now go to jenkins and create a job :

2. RedHat  ( Install manually )

sudo dnf install -y python3 python3-pip nginx
sudo systemctl enable --now nginx

Python virtualenv setup:

sudo dnf install -y python3-virtualenv


Create a systemd service file.
--------------------------

[Unit]
Description=Gunicorn instance for Flask app
After=network.target

[Service]
User=ec2-user
Group=ec2-user
WorkingDirectory=/opt/myflaskapp
Environment="PATH=/opt/myflaskapp/venv/bin"
ExecStart=/opt/myflaskapp/venv/bin/gunicorn --workers 3 --bind 127.0.0.1:8000 app:app
Restart=always

[Install]
WantedBy=multi-user.target

--------------------------

sudo systemctl daemon-reload
sudo systemctl enable myflaskapp


Nginx Reverse Proxy

/etc/nginx/conf.d/myflaskapp.conf

server {
    listen 80;
    server_name <ip-public>;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}



sudo nginx -t
sudo systemctl reload nginx


Pay attention to SELinux and the firewall (they are enabled by default on RedHat):

sudo setsebool -P httpd_can_network_connect 1
getsebool httpd_can_network_connect


Jenkinsfile 

pipeline {
    agent any

    environment {
        REMOTE_HOST = "your_user@redhat_server_ip"
        DEPLOY_DIR  = "/opt/myflaskapp"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/yourname/myflaskapp.git'
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



curl http://your_server_ip

