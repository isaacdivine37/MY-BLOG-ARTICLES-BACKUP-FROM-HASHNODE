---
title: "Deploy Your MERN Stack App to Azure with GitHub Actions: A Step-by-Step Guide"
datePublished: Mon Sep 08 2025 07:08:28 GMT+0000 (Coordinated Universal Time)
cuid: cmfas5u2d000802l9f59zbk0b
slug: deploy-your-mern-stack-app-to-azure-with-github-actions-a-step-by-step-guide
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1757265768122/f7e25a36-57d6-4455-ae71-1b0b696f8e27.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1757314775085/665648b0-72ba-4cf6-bdb0-fac44e768e0d.png
tags: programming, azure, cloud-computing, devops, azure-app-service, github-actions-1, devops-articles

---

# Introduction

The **MERN stack** — MongoDB, Express.js, React, and Node.js has become one of the most popular frameworks for building modern, full-stack web applications. While MERN makes development seamless, deploying a production-ready application can be challenging without the right automation practices.

That’s where **Microsoft Azure** and **GitHub Actions** come in. By combining the scalability of Azure with the power of GitHub’s CI/CD pipelines, you can automate the entire deployment process from pushing code to GitHub, to building, testing, and deploying your MERN app to the cloud.

In this guide, I’ll Walk you through how to deploy a full-stack MERN application to Azure using GitHub Actions. By the end of this tutorial, you will have:

* A live MERN application running on Azure
    
* A fully automated CI/CD pipeline that deploys every new code push
    
* Secure handling of environment variables and database credentials
    
* A clear understanding of how to integrate MongoDB Atlas with Azure services
    

Whether you’re a beginner exploring cloud deployments or a developer aiming to streamline production workflows, this step-by-step approach will give you the confidence to run and scale MERN applications in the cloud.

## Pre-requisites

Before we begin, make sure you have the following set up:

1. **Microsoft Azure Account**
    
    * Sign up here: Azure Free Account
        
    * [S](https://github.com?utm_source=chatgpt.com)tudents can get $100 free credits with their student email ID → Azure for Students
        
    * Access the Azure Portal: portal.azure.com
        
2. **Git & GitHub Account**
    
3. **Node.js & npm(for local testing)**
    
    * Download and install: N[o](https://git-scm.com/downloads?utm_source=chatgpt.com)de.js
        
4. **MongoDB Atlas Account (for cloud database hosting)**
    
    * Sign up here: MongoDB Atlas
        
    * Make sure to whitelist your IPs under Network Access Settings.
        
    * (For production, restrict access to only trusted services. For this demo, you can allow all IPs `0.0.0.0/0` for easier setup.)
        
5. **Azure CLI (for managing Azure resources from your terminal)**
    
    * Install here: Azure CLI Installation Guide
        
6. **Basic Knowledg**[**e**](https://git-scm.com/downloads?utm_source=chatgpt.com) **of:**
    
    * React (Frontend)
        
    * Express.js & Node.js (Backend)
        
    * Git & GitHub Actions (Version Control + CI/CD)
        

---

## Step-by-Step Deploymen[t](https://portal.azure.com?utm_source=chatgpt.com)

### STEP 1) Clone the Repository

First, clone your MERN project (or a demo repo):

```plaintext
git clone <your-repo-url>
cd <your-project-folder>
```

### STEP 2) Understand the Project Structure

A typical MERN project looks like this:

```plaintext
/app
 ├── /client   (React frontend)
 ├── /server   (Express backend)
 └── README.md
```

* **Frontend:** React (served via Azure Static Web Apps)
    
* **Backend:** Express.js (deployed as Azure Web App)
    
* **Database:** MongoDB Atlas.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756974787370/2dbf7110-684e-4cf5-8eef-d56accb59de8.png align="center")
    

---

### STEP 3) Setup MongoDB Atlas

1. Create an account at MongoDB Atlas.
    
2. Create a new **Cluster**.
    
3. Under **Network Access**, add `0.0.0.0/0` for testing (allows all IPs).  
    *(In production, restrict to trusted IPs only* )
    
4. Get your connection string from **Database → Connect → Drivers**.
    

Example:

```plaintext
mongodb+srv://<username>:<password>@cluster0.mongodb.net/myDB
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756975114925/cb1913e5-a7ed-4a0b-999c-7b98783bfb34.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757141238160/942c7964-7c9c-4b58-b3fc-cce81c10dbb9.png align="center")

---

### STEP 4) Configure Backend Locally

Inside `/server`, create a `config.env`:

```plaintext
ATLAS_URI=<your-mongodb-ATLAS-connection-string>
PORT=5050
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756975371621/4d25ffc5-806a-45b5-8166-083ece2a24e8.png align="center")

Run your backend:

```plaintext
cd server
npm install
npm run dev
```

Your backend should be live at → `http://localhost:5050`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756975724810/09eabb5f-81b0-4adb-8d47-14fc38b2b664.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756975859541/42b25f29-db08-444f-88b0-3e16a6b732bc.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756976038839/9d073d73-172a-40bb-8b07-57df04f8d4f6.png align="center")

---

### STEP 6) Configure Frontend Locally

Inside `/client`, create a `.env` file:

```plaintext
VITE_API_URL=http://localhost:5050
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756976276348/c2129666-20ba-4d27-85dd-f0a05e8cb77a.png align="center")

Run your frontend:

```plaintext
cd client
npm install
npm run dev
```

Your frontend should be live at → `http://localhost:5173`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756976625030/c1857def-5087-44a4-b88d-ab268e7bb5c4.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756976714535/d313fb56-052b-426c-80db-e5466759da2c.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756976896482/7b3d3d02-76cf-4025-bd01-5e5d237d2f21.png align="center")

**TESTING THE MERN APP LOCALLY**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756977071261/5b545db7-0446-4227-b5da-fc807d61cb16.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756977215359/a18c26d9-574f-4849-bb9b-aa5ad376cb89.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756977393537/ff8a819b-0226-429a-86ee-76e5afcf1d24.png align="center")

---

### STEP 6) Deploy Backend to Azure Web App

1. Go to Azure Portal → **Create a Web App**
    
    * Runtime: Node.js
        
    * Region: Choose closest to you
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757056669063/44b6a2ff-90b8-4f60-9991-64374a556076.png align="center")
        
2. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757056782235/f29ef643-7934-4120-91fa-b1283c828ecc.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757056963659/dfbdb327-6e73-49de-aaee-58b5f38619ea.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757057392092/0b800a1a-f6fe-4c26-89c9-9fbd27901e43.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757057799187/2eba11b4-7500-4967-84b5-eec57e299d5b.png align="center")
    
    Connect it to **GitHub Actions** under Deployment Center.
    
3. Add **environment variables**:
    
    * `ATLAS_URI` → Your MongoDB connection string
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757057609914/27042657-eb6e-46a8-a962-787b8fd9b2ca.png align="center")
        
4. Push code → GitHub Actions will build & deploy your backend.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757057946908/d685df45-4ff4-49d2-bbd5-59aa3116f2a5.png align="center")
    
    NB; Don’t forget to create federated credentials under app registration for authentication of GitHub actions and azure credentials. (Follow these steps in the Azure portal to create the necessary federated credential.
    
    1. **Find Your App Registration**: In the Azure portal, navigate to **Azure Active Directory** &gt; **App registrations**. Search for the application registration associated with your service principal, in this case, it appears to be named `mern-deployment-sp`.
        
    2. **Go to Certificates & secrets**: From the left-hand menu of your app registration, select **Certificates & secrets**.
        
    3. **Add a Federated Credential**: Click on the **Federated credentials** tab, then click **Add credential**.
        
    4. **Configure the Credential**: Fill out the form with the following details from your error log:
        
        * **Scenario**: Choose **GitHub Actions deploying to Azure resources**.
            
        * **Organization**: Your GitHub organization name (`isaacdivine37`).
            
        * **Repository**: The name of your repository (`MERN-App-Deployment-Automation-main`).
            
        * **Entity type**: `Environment`.
            
        * **GitHub environment name**: `Production` (this must match the environment name in your workflow).
            
        * **Name**: Give it a descriptive name like `mern-app-production-deployment`.)
            
            ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757058277489/daefb95f-be24-4532-9e98-deee0e046ab2.png align="center")
            
            ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757058361581/bf6d6e75-65fb-4c03-80e0-130ba7a193ba.png align="center")
            

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757137539779/7defa62a-2112-4808-8781-1afffbac8f55.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757139212778/4372cad6-2f3d-4722-a3e9-a9f72a2a73b8.png align="center")

---

### STEP 7) Deploy Frontend to Azure Static Web Apps

1. Go to Azure Portal → **Create Static Web App**.
    
2. Connect GitHub repo → choose branch.
    
3. Configure build settings:
    
    * **App Location:** `app/client`
        
    * **Output Location:** `dist`
        
4. Add **GitHub Secret**:
    
    * `VITE_API_URL` → Backend Web App URL (e.g., `https://your-api.azurewebsites.net`)
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757142259576/bb586434-b66c-41ae-b12c-174933e3672b.png align="center")
        
        Access the Github actions and edit the yaml and include the environment variable for it access the VITE\_API\_URL
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757140150732/5f4ada4c-bd71-4b32-a6e2-e83bff768352.png align="center")
        
        Testing the workflow after including the vite variable
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757140052743/45c8bd8b-db2f-4524-8714-d806a18720c3.png align="center")
        
5. Push code → GitHub Actions will build & deploy your frontend.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757140309748/448bd8ea-da7c-40e0-8e61-06ee0477fc47.png align="center")
    

---

### STEP 8) Verify Deployment

* Frontend → `https://your-frontend.azurestaticapps.net`
    

They should now communicate via **VITE\_API\_URL**.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757140420077/dcd5a808-a2d2-420f-bba6-07458061ec4d.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757140473766/2e985de0-6cc2-4382-b33d-4134d59d4046.png align="center")

Backend → `https://your-api.azurewebsites.net/record`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757140607670/fbf72f84-b5df-4263-af71-d2d390d746ff.png align="center")

Verifying our profile at MongoDb Atlas

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757140844498/c346e7bb-7c5b-4945-80bf-e38c1ec5381c.png align="center")

---

## Why Use Azure Static Web Apps for MERN Frontend?

**Azure Static Web Apps** is better than regular App Service for React frontends because:

* **Performance**: Static files served via global CDN load faster than server-hosted React
    
* **Cost**: Free tier covers most apps vs $13+ monthly for App Service
    
* **Auto-scaling**: Handles traffic spikes automatically without configuration
    
* **Independent deployment**: Frontend and backend can be updated separately
    
* **Built-in CI/CD**: Automatic GitHub integration with preview deployments
    

**Use regular App Service only if you need:**

* Server-side rendering (SSR)
    
* Real-time WebSocket features
    
* Tightly coupled frontend/backend
    

Resources

* [Azure Web App Docs](https://learn.microsoft.com/en-us/azure/app-service/?utm_source=chatgpt.com)
    
* [Azure Static Web Apps Docs](https://learn.microsoft.com/en-us/azure/static-web-apps/?utm_source=chatgpt.com)
    
* [GitHub Actions Docs](https://docs.github.com/en/actions?utm_source=chatgpt.com)
    
* [MongoDB Atlas Docs](https://www.mongodb.com/docs/atlas/?utm_source=chatgpt.com)