# React Todo App
## Part 1: Create a sudo user in Ubuntu
### Step-1:Create a new user called "DevOps":
```bash
sudo adduser DevOps
```
### Step 2: Add the DevOps user to the sudo group
```bash
sudo usermod -aG sudo DevOps
```
### Verify the user is added to the sudo group:
```bash
sudo su - DevOps
sudo -l
```
## Part 2: Checkout, Build, and Deploy a React Project
### Step 1: Install NodeJS & npm
### Update the package list and install NodeJS & npm:

```bash
sudo apt update
sudo apt install nodejs npm -y
```

### Verify the installation:
```bash
node -v
npm -v
```
### Step 2: Clone the project from GitHub
### Create the necessary directories:
```bash
sudo mkdir -p /opt/checkout/react-todo-app
sudo chown -R DevOps:DevOps /opt/checkout/react-todo-app
```

### Clone the repository:
```bash
git clone https://github.com/kabirbaidhya/react-todo-app /opt/checkout/react-todo-app
```
### Step 3: Build the React project

### Navigate to the project directory and install dependencies:
```bash
cd /opt/checkout/react-todo-app
npm install

```
### Build the project:
```bash
npm run build
```
### Step 4: Move the build files
### 1.Create the deployment directory and move the build files:
```bash
sudo mkdir -p /opt/deployment/react
sudo mv build/* /opt/deployment/react/
```
### Step 5: Deploy the project using pm2
```bash
Install pm2 globally:
sudo npm install -g pm2
```

### 2.Start the project with pm2:
```bash
cd /opt/deployment/react
pm2 serve /opt/deployment/react 3000 --name react-todo-app
```
## Part 3: Serve the Project
### Step 1: Setup Nginx proxy

### 1.Install Nginx:
```bash
sudo apt install nginx -y
```

### 2.Configure Nginx:
```bash
sudo nano /etc/nginx/sites-available/default
```
### Add the following configuration:
```groovy
server {
    listen 80;
    server_name <your-server-ip>;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```
### 3.Restart Nginx:
```bash
sudo systemctl restart nginx
```
### Step 2: Access the application
Open your browser and navigate to:
http://<your-server-ip>



### Step 3: Block unnecessary ports using UFW

### Allow necessary ports:
```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 3000/tcp
sudo ufw allow 22/tcp
```

### Enable UFW:
```bash
sudo ufw enable
sudo ufw status
```
## Part 4: CICD
### Step 1: Setup Jenkins
### Install Jenkins:
```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
```
```bash
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ |  sudo  tee /etc/apt/sources.list.d/jenkins.list > /dev/null
```
```bash
sudo apt install jenkins
```
```bash
sudo systemctl start jenkins
sudo systemctl status jenkins
```
```bash
sudo ufw allow 8080/tcp
```
```groovy
pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'ap-south-1'  // e.g., 'us-west-2'
    }

    stages {
        stage('Stop Deployment') {
            steps {
                script {
                    sshagent(credentials: ['ssh-cred']) {
                        sh "ssh ubuntu@15.206.90.153 'pm2 stop react-todo-app'"
                    }
                }
            }
        }
        stage('Checkout Code') {
            steps {
                git 'https://github.com/kabirbaidhya/react-todo-app'
            }
        }
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
         stage('Deploy using PM2') {
            steps {
                script {
                    sshagent(credentials: ['ssh-cred']) {
                        sh "scp -r build/* ubuntu@15.206.90.153:/opt/deployment/react"
                        sh "ssh ubuntu@15.206.90.153 'pm2 restart react-todo-app'"
                    }
                }
            }
        }

        stage('Upload Build to S3') {
            steps {
                script {
                    withCredentials([aws(accessKeyVariable: 'aws-access-key', credentialsId: 'aws-cred', secretKeyVariable: 'aws-secret-key')]) {
                        s3Upload(bucket: 'react-todo-app-project-1', path: 'react-todo-app/', file: 'build/**', workingDir: 'build')
                    }
                }
            }
        }
    }
}
        
```
## Part 5: Shell Scripting
### Create a Bash script to install Java

### 1.Create the script file:
```bash
sudo mkdir -p /opt/scripts
sudo nano /opt/scripts/install_java.sh
```

### 2.Add the following script content:
```bash
#!/bin/bash
LOG_FILE="/opt/logs/script_logs.log"
exec > >(tee -i $LOG_FILE)
exec 2>&1

echo "$(date) - Starting Java Installation"

echo "$(date) - Downloading OpenJDK 1.8"
sudo apt update
sudo apt install openjdk-8-jdk -y

echo "$(date) - Setting up Java environment variables"
echo "export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64" >> ~/.bashrc
echo "export PATH=\$PATH:\$JAVA_HOME/bin" >> ~/.bashrc
source ~/.bashrc

echo "$(date) - Java Installation Completed"
```
### 3.Make the script executable and run it:
```bash
sudo chmod +x /opt/scripts/install_java.sh
/opt/scripts/install_java.sh
```
## Part 6: Docker
### Dockerize the React app
### 1.Create a Dockerfile:
```bash
sudo nano /opt/checkout/react-todo-app/Dockerfile
```
### 2.Add the following content to the Dockerfile:

Dockerfile

```bash
FROM nginx:alpine
COPY build /usr/share/nginx/html
EXPOSE 3000
Create a docker-compose.yml:
```

```bash
sudo nano /opt/checkout/react-todo-app/docker-compose.yml
```
### 3.Add the following content to the docker-compose.yml:

```yaml
yaml
Copy code
version: '3'
services:
  react-todo-app:
    build: .
    ports:
      - "3000:80"
```
### 4.Build and run the Docker container:
```bash
cd /opt/checkout/react-todo-app
```