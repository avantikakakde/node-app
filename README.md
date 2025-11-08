# 🚀 Automated CI/CD for a Node.js Application with Jenkins and GitHub Webhooks

This project sets up a **fully automated CI/CD pipeline** for a **Node.js application** using **Jenkins** and **GitHub Webhooks**.  
Every time new code is pushed to the repository, Jenkins automatically triggers the build, test, and deployment process — ensuring smooth, consistent delivery with zero manual steps.

---

## 🧩 Project Overview

The pipeline demonstrates the following DevOps practices:
- **Continuous Integration (CI):** Automatically builds and tests code changes.
- **Continuous Deployment (CD):** Deploys updates to the server after successful builds.
- **Automation:** Eliminates manual deployment steps using Jenkins automation.

---

## ⚙️ Tech Stack

- **Node.js** – Application runtime  
- **Jenkins** – Automation server  
- **GitHub Webhooks** – Triggers Jenkins jobs on code push  
- **PM2** – Process manager for Node.js apps  
- **Ubuntu (EC2)** – Deployment environment  

---

## 🔄 CI/CD Flow

1. Developer pushes code to the **GitHub repository**  
2. **GitHub Webhook** notifies **Jenkins**  
3. **Jenkins**:
   - Pulls the latest code  
   - Installs dependencies (`npm install`)  
   - Runs build/tests (if configured)  
   - Deploys the app using **PM2**



---

## 🧩 Step 1: Launch EC2 Instances

### Create Two EC2 Instances (in the same default VPC)

#### 1️⃣ Jenkins Server
- Add **port 8080** in the Security Group (for Jenkins access)

#### 2️⃣ Target Server
- Add **port 22** (SSH) and **port 3000** (app access) in the Security Group

### Install Node.js, npm, and PM2 on Target Server
```bash
sudo apt update
sudo apt install nodejs npm -y
sudo npm install -g pm2
```

![](./images/Screenshot%202025-11-08%20182023.png)

![](./images/Screenshot%202025-11-08%20180900.png)



![](./images/Screenshot%202025-11-08%20181204.png)




---

## 🧾 Step 2: Create GitHub Repository

* Create a new repository on GitHub
  **Name:** `node-app-deploy-cicd`
  **Branch:** `main`

### Add Webhook

* Go to **Settings → Webhooks → Add webhook**
* **Payload URL:**
  `http://<JENKINS-SERVER-PUBLIC-IP>:8080/github-webhook/`
* **Content Type:** `application/json`
* **Event:** “Just the push event”


![](./images/Screenshot%202025-11-08%20181540.png)

---

## 🔐 Step 3: Add Credentials in Jenkins

Navigate to:
**Manage Jenkins → Credentials → System → Global**

Create New Credential:

* **Scope:** Global
* **ID:** `node-app-key`
* **Description:** node-app-key
* **Username:** ubuntu


---

## ⚙️ Step 4: Create a Jenkins Pipeline Job

1. Go to Jenkins → **New Item → Pipeline**
2. **Name:** `node-app-deploy-cicd`
3. **Trigger:** ✅ *GitHub hook trigger for GITScm polling*
4. **Definition:** Pipeline script from SCM
5. **SCM:** Git
   **Repository URL:** 
   **Branch:** `main`
6. **Script Path:** *(your Jenkinsfile name, e.g. `Jenkinsfile`)*

---

## 🧠 Step 5: Jenkinsfile

```groovy
pipeline {
    agent any

    environment {
        SERVER_IP      = '13.233.116.153'
        SSH_CREDENTIAL = 'node-app-key'
        REPO_URL       = 'https://github.com/avantikakakde/node-app.git'
        BRANCH         = 'main'
        REMOTE_USER    = 'ubuntu'
        REMOTE_PATH    = '/home/ubuntu/node-app'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: "${BRANCH}", url: "${REPO_URL}"
            }
        }

        stage('Upload Files to EC2') {
            steps {
                sshagent([SSH_CREDENTIAL]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} 'mkdir -p ${REMOTE_PATH}'
                        scp -o StrictHostKeyChecking=no -r * ${REMOTE_USER}@${SERVER_IP}:${REMOTE_PATH}/
                    """
                }
            }
        }

        stage('Install Dependencies & Start App') {
            steps {
                sshagent([SSH_CREDENTIAL]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} '
                            cd ${REMOTE_PATH} &&
                            npm install &&
                            pm2 start app.js --name node-app || pm2 restart node-app
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Application deployed successfully!'
        }
        failure {
            echo '❌ Deployment failed.'
        }
    }
}


```

---

## 💾 Step 6: Push Code and Jenkinsfile to GitHub

```bash
git init
git add .
git commit -m "Initial commit "
git branch -M main
git remote add origin 
git push -u origin main
```

Once code is pushed, the **GitHub Webhook** automatically triggers Jenkins.
Jenkins will:

* Pull the latest code
* Install dependencies
* Build and deploy the app to the target EC2 instance

---

## 🌐 Step 7: Access the Application

Open a browser and visit:

```
http://<TARGET-SERVER-PUBLIC-IP>:3000
```
![](./images/Screenshot%202025-11-08%20181938.png)

---

## 🏁 Conclusion

Deploying a Node.js application using **Jenkins CI/CD** integrated with **GitHub Webhooks** provides a **fully automated, efficient, and reliable** deployment pipeline.

Every push to GitHub triggers the entire process — build → test → deploy — ensuring faster delivery, reduced manual effort, and better collaboration.

This setup enhances code quality, scalability, and automation — making the deployment process truly **DevOps-ready**. 🚀

---

## 📘 About

This project demonstrates:

* Continuous Integration and Continuous Deployment (CI/CD)
* Jenkins + GitHub Webhook integration
* Node.js application deployment using PM2 on AWS EC2
  It’s a great foundation for mastering **DevOps automation** and **pipeline workflows**.
