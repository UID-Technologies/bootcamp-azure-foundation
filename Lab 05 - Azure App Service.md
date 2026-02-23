#  Lab 6: Azure App Service – Deploy a Sample Web Application

##  Lab Objectives

By the end of this lab, participants will be able to:

* Understand **Azure App Service** and its capabilities
* Create an **App Service Plan** and **Web App**
* Deploy a **sample web application** using multiple methods
* Configure **application settings** and **connection strings**
* Enable **deployment slots** for staging
* Validate **scaling options** and **monitoring**
* Access and test the deployed application

All tasks are performed in Microsoft Azure.

---

##  Estimated Time

**~120 minutes**

---

##  Prerequisites

* Completed **Sessions 1–5 labs**
* Existing resources:

  * Resource Group: `rg-app-dev-centralindia`
  * Log Analytics Workspace: `law-app-dev` (optional, for monitoring)
* Contributor access on subscription
* Azure Portal access
* (Optional) Git installed locally for deployment methods

---

##  Lab Scenario (Enterprise Context)

You are deploying a **web application** for your organization that requires:

* Managed hosting without VM management
* Automatic scaling based on demand
* Support for multiple deployment methods (ZIP, Git, GitHub)
* Staging environment for safe releases
* Integration with existing monitoring and governance

![Image](https://learn.microsoft.com/en-us/azure/architecture/web-apps/app-service/media/basic-web-app/basic-web-app.png)

![Image](https://learn.microsoft.com/en-us/azure/app-service/media/overview/app-service-value-proposition.png)

![Image](https://learn.microsoft.com/en-us/azure/app-service/media/overview/choose-deployment-method.png)

![Image](https://learn.microsoft.com/en-us/azure/app-service/media/overview/scale-app-service.png)

![Image](https://learn.microsoft.com/en-us/azure/app-service/media/overview/deployment-slots.png)

---

##  Step 1: Review Azure App Service Concepts (Checkpoint)

Before implementation, confirm understanding:

* **App Service** = PaaS for web apps, APIs, mobile backends
* **App Service Plan** = Compute resources (CPU, memory) for your apps
* **Web App** = The actual application running on the plan
* Supports: .NET, Java, Node.js, Python, PHP, containers
* Built-in: scaling, SSL, deployment slots, CI/CD

Trainer checkpoint ✔️

---

##  Step 2: Create an App Service Plan

1. In **Azure Portal**, search **App Service plans**
2. Click **Create**
3. Configure **Basics**:

   * **Subscription:** Active subscription
   * **Resource Group:** `rg-app-dev-centralindia`
   * **Name:** `asp-app-dev`
   * **Operating System:** Linux
   * **Region:** Central India
4. Click **Next: Compute >**

---

##  Step 3: Select Compute Tier

* **Pricing tier:** Free F1 (for lab) or Basic B1 (recommended for slots)
* **Sku and size:** Free F1 or Basic B1

💡 **Note:** Deployment slots require **Standard** tier or higher. For lab, Free/Basic is sufficient for core deployment.

Click **Review + Create** → **Create**

---

##  Step 4: Create a Web App

1. Search **App Services**
2. Click **Create**
3. Configure **Basics**:

   * **Subscription:** Active subscription
   * **Resource Group:** `rg-app-dev-centralindia`
   * **Name:** `webapp-app-dev-<uniqueid>` (must be globally unique)
   * **Publish:** Code
   * **Runtime stack:** Node 20 LTS (or Python 3.11)
   * **Operating System:** Linux
   * **Region:** Central India
   * **App Service Plan:** `asp-app-dev`
4. Click **Review + Create** → **Create**

✅ **Expected Result:**
Web App created and running

---

##  Step 5: Create a Simple Sample Application (Node.js)

Create a minimal Node.js app for deployment.

### Option A: Using Cloud Shell

1. Open **Cloud Shell** (Bash)
2. Create project directory:

```bash
mkdir -p ~/sample-webapp && cd ~/sample-webapp
```

3. Create `package.json`:

```json
{
  "name": "sample-webapp",
  "version": "1.0.0",
  "description": "Sample Azure App Service app",
  "main": "server.js"
}
```

4. Create `server.js`:

```javascript
const http = require('http');
const port = process.env.PORT || 8080;
const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/html' });
  res.end('<h1>Welcome to Azure App Service!</h1><p>Lab 06 - Sample Web Application</p>');
});
server.listen(port, () => console.log('Server running on port ' + port));
```

5. Zip the application:

```bash
zip -r app.zip package.json server.js
```

---

##  Step 6: Deploy Using ZIP Deploy

1. Open your **Web App** in Azure Portal
2. Go to **Deployment Center**
3. Select **Zip Deploy** or use **Advanced Tools (Kudu)**

### Using Azure CLI (from Cloud Shell):

```bash
az webapp deployment source config-zip \
  --resource-group rg-app-dev-centralindia \
  --name <your-webapp-name> \
  --src ~/sample-webapp/app.zip
```

Replace `<your-webapp-name>` with your actual Web App name.

✅ **Expected Result:**
Deployment succeeds; app is live

---

##  Step 7: Configure Application Settings

1. Open Web App → **Configuration**
2. Click **+ New application setting**
3. Add:

   * **Name:** `APP_ENV`
   * **Value:** `development`
4. Add another:

   * **Name:** `WEBSITE_NODE_DEFAULT_VERSION`
   * **Value:** `~20`
5. Click **Save** → **Continue**

💡 **Learning Point:**
App settings are injected as environment variables—never hardcode config in code.

---

##  Step 8: Enable Always On (Optional – Basic+ Tier)

1. Go to **Configuration → General settings**
2. Set **Always On:** On (if available on your tier)
3. Save

💡 **Why:** Prevents cold starts; keeps app warm for faster response.

---

##  Step 9: Access and Test the Application

1. Open Web App → **Overview**
2. Click the **Default domain** URL (e.g., `https://webapp-app-dev-xxx.azurewebsites.net`)
3. Verify:

   * Page loads successfully
   * "Welcome to Azure App Service!" message displayed

✅ **Expected Result:**
Sample application is accessible and responding

---

##  Step 10: Explore Deployment Slots (Standard Tier or Higher)

If using **Standard** tier or above:

1. Go to **Deployment slots**
2. Click **+ Add Slot**
3. Name: `staging`
4. Create

💡 **Use Case:**
Deploy to staging first, validate, then **swap** to production with zero downtime.

---

##  Step 11: Deploy via GitHub (Optional – Alternative Method)

1. Go to **Deployment Center**
2. Select **GitHub** as source
3. Authorize Azure to access your GitHub
4. Choose:

   * Organization, Repository, Branch
5. Save

✅ **Expected Result:**
Automatic deployment on every push to the selected branch

---

##  Step 12: Configure Scaling (Read-Only)

1. Open Web App → **Scale out (App Service plan)**
2. Review options:

   * **Manual scale:** Set instance count
   * **Custom autoscale:** Scale based on CPU, HTTP queue, etc.

💡 **Learning Point:**
Scale out = more instances; Scale up = bigger VM (more CPU/RAM).

---

##  Step 13: Enable Application Insights (Optional)

1. Open Web App → **Application Insights**
2. Click **Turn on Application Insights**
3. Create new or use existing: `law-app-dev` (Log Analytics)
4. Apply

💡 Enables request tracking, performance monitoring, and failure analysis.

---

##  Step 14: Validate via Azure CLI (Optional)

In **Cloud Shell**:

```bash
az webapp list -g rg-app-dev-centralindia -o table
az webapp show -n <your-webapp-name> -g rg-app-dev-centralindia
az webapp deployment list-publishing-credentials -n <your-webapp-name> -g rg-app-dev-centralindia
```

---

##  Lab Validation Checklist

✔ App Service Plan created
✔ Web App created with correct runtime
✔ Sample application deployed successfully
✔ Application settings configured
✔ Default URL accessible
✔ Deployment method validated (ZIP or GitHub)
✔ (Optional) Deployment slot created
✔ (Optional) Application Insights enabled

---

##  Key Takeaways

* Azure App Service is a fully managed PaaS for web applications
* App Service Plan defines compute capacity; Web App hosts your code
* Multiple deployment options: ZIP, Git, GitHub, containers
* Application settings keep configuration out of code
* Deployment slots enable zero-downtime releases
* Scaling (out/up) supports growth without code changes
* App Service integrates with monitoring and CI/CD pipelines
