# Jenkins CI/CD Pipeline – Apache Static Website Deployment
 
---
 
## Project Overview
 
This project demonstrates a complete CI/CD pipeline implemented on a local machine using Jenkins to automate deployment of a static website to an Apache web server.
 
The pipeline pulls code from a GitHub repository and deploys the website to Apache’s document root directory, making it accessible via a browser.
 
---
 
## Technologies Used
 
* Jenkins (CI/CD Automation)
* Apache HTTP Server
* Git & GitHub
* Linux (SLES / Local System)
 
---
 
##  Step-by-Step Implementation
 
---
 
###  Step 1: Install and Start Apache Server
 
```bash
sudo zypper install apache2 -y
sudo systemctl start apache2
sudo systemctl enable apache2
```
 
![apache_install](Images/apache_install.png)
 
Verification:
 
```
http://localhost
```
 
![apache_ui.png](Images/apache_ui.png)
 
---
 
### Step 2: Install Git
 
Git is required for Jenkins to pull code from GitHub.
 
```bash
sudo zypper install git -y
```
 
Verify:
 
```bash
git --version
```
 
 
![git_install](Images/git_install.png)
 
---
 
### Step 3: Install and Setup Jenkins
 
```bash
sudo zypper install jenkins -y
sudo systemctl start jenkins
sudo systemctl enable jenkins
```
 
![jenkins_install](Images/jenkins_install.png)
 
Access Jenkins:
 
```
http://localhost:8080
```
 
Initial Setup:
 
* Enter admin password
* Install suggested plugins
 
![jenkins_UI](Images/jenkins_UI.png)
 
---
 
### Step 4: Add Jenkins Credentials
 
* Go to **Manage Jenkins → Credentials**
* Add SSH / Git credentials
* Use credentials in pipeline
 
![credentials_add.png](Images/credentials_add.png)
 
 
---
 
### Step 5: Configure SSH Access (Local Deployment)
 
```bash
ssh-keygen -t rsa
ssh-copy-id root@localhost
```
 
Test:
 
```bash
ssh root@localhost
```
 
![SSH_Setup](Images/SSH_login.png)
 
---
 
### Step 6: Add Jenkins Node (Agent Configuration)
 
* Go to **Manage Jenkins → Nodes**
* Click **New Node**
* Configure node details
* Connect node to Jenkins master
 
 
![Node_online](Images/Node_online.png)
 
---
 
### Step 7: Create Jenkins Pipeline Job
 
* Click **New Item**
* Select **Pipeline**
* Configure job
 
![Branch_created1](Images/Branch_created1.png)
![Branch_created1](Images/Branch_created2.png)
 
---
 
### Step 8: Configure GitHub Repository
 
* Add repository URL
* Select branch (main/master)
* Configure webhook for auto trigger
 
![webhook](Images/webhook.png)
 
---
 
### Step 9: Configure Pipeline
 
* Select **Pipeline script from SCM**
* SCM: Git
* Script Path: `Jenkinsfile`
 
![configuration](Images/configuration.png)
 
 
---
 
 
### Step 10: Push Code to GitHub using VS Code
 
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/gowda-m/jenkins-ci-cd-new.git
git push -u origin main
 
![push_code](Images/push_code.png)
 
---
 
### Step 11: Trigger Build and Verify Console Output
 
* auto **Build Now**
* Check logs for success
 
![auto_deploy_console.png](Images/auto_deploy_console.png)
 
---
 
### Step 12: Verify Deployment
 
Files deployed to:
 
```
/var/www/html/
```
 
![deployment](Images/deployment_index.png)
 
---
 
### Step 13: Verify Website
 
```
http://localhost
```
 
Website is live
 
 
---
 
## CI/CD Workflow
 
1. Code pushed to GitHub
2. Webhook triggers Jenkins
3. Jenkins pulls code
4. Pipeline executes
5. Deployment to Apache
6. Website goes live
 
---
 
## Final Result
 
* CI/CD pipeline successfully implemented
* Automated deployment achieved
* Website deployed on Apache
 
---
 
## Notes
 
* Local environment setup
* Can extend to AWS / Production
* Jenkinsfile handles automation
 
---
