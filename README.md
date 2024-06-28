<<<<<<< HEAD
            # React Todo App
=======
# React Todo App
>>>>>>> 7a1a096c1d1d56df20ee336be40d146435e58743
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
![Alt Text](https://github.com/arfath29/Devops-Challlenge-Project/blob/master/screenshots/Screenshot_19-6-2024_194148_ap-south-1.console.aws.amazon.com.jpeg)
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
![Alt Text](https://github.com/arfath29/Devops-Challlenge-Project/blob/master/screenshots/Screenshot_21-6-2024_18314_13.232.189.51.jpeg)
### Step 3: Block unnecessary ports using UFW

### Allow necessary ports:
```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 3000/tcp
sudo ufw allow 22/tcp
```
![Alt Text](https://github.com/arfath29/Devops-Challlenge-Project/blob/master/screenshots/Screenshot_21-6-2024_183230_ap-south-1.console.aws.amazon.com.jpeg)
### Enable UFW:
```bash
sudo ufw enable
sudo ufw status
```
## Part 4: CICD
### Step 1: Setup Jenkins
### Install Jenkins:
```bash
sudo apt install openjdk-17-jdk -y
```
```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
```
```bash
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ |  sudo  tee /etc/apt/sources.list.d/jenkins.list > /dev/null
```
```bash
sudo apt update
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
echo "$(date) - Downloading OpenJDK 1.8" | tee -a /opt/logs/script_logs.log
sudo apt update | tee -a /opt/logs/script_logs.log
sudo apt install openjdk-8-jdk -y | tee -a /opt/logs/script_logs.log

echo "$(date) - Setting JAVA_HOME and updating PATH" | tee -a /opt/logs/script_logs.log
echo "export JAVA_HOME=$(dirname $(dirname $(readlink -f $(which javac))))" | sudo tee -a /etc/profile
echo "export PATH=$PATH:$JAVA_HOME/bin" | sudo tee -a /etc/profile

echo "$(date) - Java installation and setup complete" | tee -a /opt/logs/script_logs.log

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

### Dockerfile

```bash
# Use an official Node.js runtime as a parent image
FROM node:14-alpine

# Set the working directory in the container
WORKDIR /usr/src/app

# Copy package.json and package-lock.json to the working directory
COPY /opt/checkout/react-todo-app/package*.json ./

# Install the dependencies
RUN npm install

# Copy the rest of the application code to the working directory
COPY /opt/checkout/react-todo-app .

# Build the React app
RUN npm run build

# Install serve to serve the build folder
RUN npm install -g serve

# Expose the port the app runs on
EXPOSE 3000

# Serve the build folder on port 3000
CMD ["serve", "-s", "build"]
```

### Create a docker-compose.yml:
```bash
sudo nano /opt/checkout/react-todo-app/docker-compose.yml
```
### 3.Add the following content to the docker-compose.yml:

```yaml
version: '3'
services:
  react-todo-app:
    build: .
    ports:
      - "3000:3000"
```
### 4.Build and run the Docker container:
```bash
cd /opt/checkout/react-todo-app
```
### Build the Docker image:
```bash
docker build -t react-todo-app .
```
![Alt Text](https://github.com/arfath29/Devops-Challlenge-Project/blob/master/screenshots/Screenshot_27-6-2024_142555_ap-south-1.console.aws.amazon.com.jpeg)
### Run the Docker container:
```bash
docker run -p 3000:3000 react-todo-app
```
### Build and run the containers using Docker Compose:
```bash
sudo docker-compose build
sudo docker-compose up -d
```
![Alt Text](https://github.com/arfath29/Devops-Challlenge-Project/blob/master/screenshots/Screenshot_27-6-2024_143443_ap-south-1.console.aws.amazon.com.jpeg)
