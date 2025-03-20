## Jenkins, SonarQube, and Docker CI/CD Pipeline on AWS

![jjjjjnks](https://github.com/user-attachments/assets/7dee6da4-6eae-4d4b-83ef-be188eb4f270)

## Architecture

- **GitHub**: Source code repository.
- **Jenkins**: Automation server for building, testing, and deploying code.
- **SonarQube**: Static code analysis tool.
- **Docker**: Container platform for consistent deployment.
- **AWS**: Hosting environment for Jenkins, SonarQube, and Docker servers.

## Prerequisites

1. **AWS Account**: To create EC2 instances.
2. **GitHub Repository**: To host your source code.
3. **SSH Key Pair**: For accessing EC2 instances.

## Setup Steps

### 1. Launch EC2 Instances

1. Log into the AWS management console.
2. Navigate to the EC2 dashboard and spin up three EC2 Instances (one each for Jenkins, SonarQube, and Docker).

### 2. Configure Jenkins Server

1. SSH into your Jenkins server:
   ```bash
   ssh -i web-SSHkeys.pem ubuntu@<Jenkins-Server-IP>
   ```
2. Set hostname:
   ```bash
   sudo hostnamectl set-hostname Jenkins-server
   /bin/bash
   ```
3. Update and install Java:
   ```bash
   sudo apt update
   sudo apt install openjdk-17-jre
   ```
4. Install Jenkins:
   ```bash
   sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
   echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
   sudo apt-get update
   sudo apt-get install jenkins
   ```
5. Check Jenkins status and copy the initial admin password:
   ```bash
   sudo systemctl status jenkins
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```

### 3. Configure Security Group for Jenkins

1. Open port 8080 for Jenkins in your EC2 instance's security group.

### 4. Access Jenkins

1. Open Jenkins in your browser:
   ```
   http://<Jenkins-Server-IP>:8080
   ```
2. Enter the initial admin password and install suggested plugins.
3. Create the first admin user and start using Jenkins.

### 5. Set Up GitHub Webhook

1. Create a new Jenkins job and configure source code management to point to your GitHub repository.
2. Add a GitHub webhook in your repository settings with the URL:
   ```
   http://<Jenkins-Server-IP>:8080/github-webhook/
   ```

### 6. Configure SonarQube Server

1. SSH into your SonarQube server:
   ```bash
   ssh -i web-SSHkeys.pem ubuntu@<SonarQube-Server-IP>
   ```
2. Set hostname:
   ```bash
   sudo hostnamectl set-hostname Sonarqube-server
   /bin/bash
   ```
3. Update and install Java:
   ```bash
   sudo apt update
   sudo apt upgrade -y
   sudo apt install openjdk-17-jre
   ```
4. Install and unzip SonarQube:
   ```bash
   wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.0.64466.zip
   sudo apt install unzip
   sudo unzip sonarqube-9.9.0.64466.zip
   cd sonarqube-9.9.0.64466/bin/linux-x86-64
   ./sonar.sh console
   ```
5. Open port 9000 for SonarQube in your EC2 instance's security group.

### 7. Access SonarQube

1. Open SonarQube in your browser:
   ```
   http://<SonarQube-Server-IP>:9000
   ```
2. Log in with admin/admin and update the credentials.

### 8. Integrate SonarQube with Jenkins

1. Install SonarQube Scanner and SSH plugins in Jenkins.
2. Configure SonarQube Scanner in Jenkins:
   - Go to `Manage Jenkins` > `Global Tool Configuration` > `SonarQube Scanner`.
3. Add SonarQube server in Jenkins:
   - Go to `Manage Jenkins` > `Configure System` > `SonarQube servers`.

### 9. Configure Docker Server

1. SSH into your Docker server:
   ```bash
   ssh -i web-SSHkeys.pem ubuntu@<Docker-Server-IP>
   ```
2. Set hostname:
   ```bash
   sudo hostnamectl set-hostname Docker-server
   ```
3. Update, upgrade, and install Docker:
   ```bash
   sudo apt update
   sudo apt install ca-certificates curl
   sudo install -m 0755 -d /etc/apt/keyrings
   sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
   sudo chmod a+r /etc/apt/keyrings/docker.asc
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   sudo apt-get update
   sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   sudo docker run hello-world
   ```

### 10. Enable Passwordless SSH Authentication

1. Adjust SSH configurations:
   ```bash
   sudo vi /etc/ssh/sshd_config
   # Ensure the following lines are uncommented and correctly configured
   PubkeyAuthentication yes
   PasswordAuthentication yes
   ```
2. Restart SSH service:
   ```bash
   sudo systemctl restart ssh
   ```
3. Set password for the `ubuntu` user:
   ```bash
   sudo su -
   passwd ubuntu
   exit
   ```
4. Generate and copy SSH keys from Jenkins server to Docker server:
   ```bash
   ssh-keygen -t rsa
   ssh-copy-id ubuntu@<Docker-Server-IP>
   ```

### 11. Configure Jenkins for Docker Deployment

1. Add Docker server in Jenkins:
   - Go to `Manage Jenkins` > `Configure System` > `Server group center`.

### 12. Adjust Docker Server Security Group

1. Open port 8085 for Docker in your EC2 instance's security group.

### 13. Access Your Deployed Application

1. Open your application in the browser:
   ```
   http://<Docker-Server-IP>:8085
   ```

## Conclusion

This CI/CD pipeline automates the entire process from code commit to deployment. Code changes pushed to a GitHub branch are pulled by Jenkins, analyzed by SonarQube, and then deployed to a Docker container, making the application accessible to end users through their browsers.

## Cleanup
