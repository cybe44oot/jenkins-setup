# DevOps Project — Jenkins Master & Agent Setup

This guide walks through setting up Jenkins Master and Agent on Ec2 Instances.

---

---

## 1. Create EC2 Jenkins Master

1. Copy the public IP and SSH into the instance:
   ```bash
   ssh -i your-key.pem ubuntu@<MASTER_PUBLIC_IP>
   ```
2. Update and upgrade the system:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
3. In AWS Console → Security Groups → Edit inbound rules:  
   - Add **Port 8080** → **Anywhere (IPv4)**.
4. Install Java (OpenJDK):
   ```bash
   sudo apt install -y openjdk-11-jdk
   ```
5. Install Jenkins from official website.
6. sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins

7. Enable Jenkins:
   ```bash
   sudo systemctl enable jenkins
   ```
8. Reboot system:
   ```bash
   sudo reboot
   ```
9. Edit SSH config:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```
   Uncomment or add:
   ```
   PubKeyAuthentication yes
   AuthorizedKeysFile .ssh/authorized_keys
   ```
10. Reload SSH:
   ```bash
   sudo service sshd reload
   ```

---

## 2. Create EC2 Jenkins Agent

1. SSH into the EC2 instance:
   ```bash
   ssh -i your-key.pem ubuntu@<AGENT_PUBLIC_IP>
   ```
2. Update and upgrade the system:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
3. Install Java:
   ```bash
   sudo apt install -y openjdk-11-jdk
   ```
4. Install Docker:
   ```bash
   sudo apt-get install -y docker.io
   ```
5. Add current user to Docker group:
   ```bash
   sudo usermod -aG docker $USER
   ```
6. Reboot:
   ```bash
   sudo reboot
   ```
7. Edit SSH config:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```
   Uncomment or add:
   ```
   PubKeyAuthentication yes
   AuthorizedKeysFile .ssh/authorized_keys
   ```
8. Reload SSH:
   ```bash
   sudo service sshd reload
   ```

---

## 3. Configure SSH Keys

- On Jenkins Master:
  ```bash
  cat ~/.ssh/id_rsa.pub
  ```
- On Jenkins Agent:
  Paste into `~/.ssh/authorized_keys` and save.

---

## 4. Access Jenkins UI

1. In browser:  
   ```
   http://<MASTER_PUBLIC_IP>:8080
   ```
2. Find initial admin password:
   ```bash
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```

---

## 5. Configure Master Node

- Go to **Manage Jenkins → Nodes → Built-in Node → Configure**  
- Set **Number of executors** = `0`  
- Save.

---

## 6. Add Jenkins Agent Node

1. **Manage Jenkins → Nodes → New Node**  
2. Name: `jenkins-Agent`, type: Permanent  
3. Settings:  
   - **Number of executors** = 2  
   - **Remote root directory** = `/home/ubuntu`  
   - **Launch method** = SSH  
   - **Host** = Agent's private IP  
   - **Credentials** = SSH with private key from Master `~/.ssh/id_rsa`  
   - **Host verification strategy** = Non-verifying strategy  
4. Save.

---

## 7. Test Pipeline

- Create a pipeline job using "Hello World" template.  
- Run it and confirm it executes on the Agent.



# PART 2

## Install Plugins:

1. Pipeline Maven Integration and Maven Integration and Eclipse

2. Manage Jenkins > Tools > Maven Installation  
   - Name: `maven3`  
   - Install automatically  
   - Apply

3. Manage Jenkins > JDK Installation  
   - Name: `java17`  
   - Install automatically  
   - Add installer > Install from adoptium.net  
   - Version: `jdk17.0.5+8`  
   - Apply

4. Add GitHub Credentials  
   - Manage Jenkins > Credentials > Domain (Global)  
   - Add: Username with password  
     - Username: *Your GitHub username*  
     - Password: *Your access token*  
     - ID and Description: `GitHub`

5. Assuming you already have a project in your GitHub, create a `Jenkinsfile` at the root of the project.

6. Create a pipeline project in Jenkins and configure the script to pull from SCM.
