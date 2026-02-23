#  Lab 7: Azure Function App – Serverless Compute

##  Lab Objectives

By the end of this lab, participants will be able to:

* Understand **Azure Functions** and **serverless** concepts
* Create a **Function App** with required storage account
* Develop and deploy **HTTP-triggered** and **timer-triggered** functions
* Configure **function bindings** (input/output)
* Test functions via **portal**, **browser**, and **Postman/curl**
* Monitor function execution using **Application Insights**
* Understand **triggers**, **bindings**, and **hosting plans**

All tasks are performed in Microsoft Azure.

---

##  Estimated Time

**~120 minutes**

---

##  Prerequisites

* Completed **Sessions 1–6 labs**
* Existing resources:

  * Resource Group: `rg-app-dev-centralindia`
  * Storage Account: `stappdev<uniqueid>` (or create new for Function App)
* Contributor access on subscription
* Azure Portal access
* (Optional) Visual Studio Code with Azure Functions extension for local development

---

##  Lab Scenario (Enterprise Context)

You are building **event-driven** and **API-based** serverless workloads:

* HTTP-triggered API for lightweight REST endpoints
* Timer-triggered function for scheduled tasks (e.g., cleanup, reports)
* No server management—pay only for execution time
* Automatic scaling based on demand

![Image](https://learn.microsoft.com/en-us/azure/azure-functions/media/functions-overview/azure-functions-overview.png)

![Image](https://learn.microsoft.com/en-us/azure/azure-functions/media/functions-triggers-bindings/function-trigger-bindings.png)

![Image](https://learn.microsoft.com/en-us/azure/azure-functions/media/functions-scale/consumption-plan.png)

![Image](https://learn.microsoft.com/en-us/azure/azure-functions/media/functions-bindings-http-request/function-app-run-http-trigger.png)

![Image](https://learn.microsoft.com/en-us/azure/azure-functions/media/functions-monitoring/application-insights.png)

---

##  Step 1: Review Azure Functions Concepts (Checkpoint)

Before implementation, confirm understanding:

* **Azure Functions** = Serverless compute; run code on events
* **Trigger** = What starts the function (HTTP, Timer, Blob, Queue, etc.)
* **Binding** = Connect to Azure services (Storage, Cosmos DB, etc.) without code
* **Function App** = Container for one or more functions
* **Consumption plan** = Pay per execution; auto-scale to zero

Trainer checkpoint ✔️

---

##  Step 2: Create a Storage Account for Function App

Azure Functions **requires** a storage account for:

* Managing triggers and execution state
* Logging
* Supporting scaling

1. If you have `stappdev<uniqueid>` from Lab 4, reuse it
2. Otherwise, create new:

   * Search **Storage accounts** → **Create**
   * **Resource Group:** `rg-app-dev-centralindia`
   * **Name:** `stfuncdev<uniqueid>` (globally unique)
   * **Region:** Central India
   * **Redundancy:** LRS
3. Create

---

##  Step 3: Create a Function App

1. In **Azure Portal**, search **Function App**
2. Click **Create**
3. Configure **Basics**:

   * **Subscription:** Active subscription
   * **Resource Group:** `rg-app-dev-centralindia`
   * **Function App name:** `func-app-dev-<uniqueid>` (globally unique)
   * **Runtime stack:** Node.js
   * **Version:** 20 LTS
   * **Region:** Central India
   * **Operating System:** Linux
4. Click **Next: Storage >**

---

##  Step 4: Configure Storage and Hosting

* **Storage account:** Select `stfuncdev<uniqueid>` or `stappdev<uniqueid>`
* **Hosting options:** Consumption (Serverless)

Click **Review + Create** → **Create**

⏳ Deployment may take 1–2 minutes.

---

##  Step 5: Create an HTTP-Triggered Function (Portal)

1. Open your **Function App**
2. Go to **Functions**
3. Click **+ Create**
4. Choose **Develop in portal**
5. Select template: **HTTP trigger**
6. Configure:

   * **New Function:** `HttpTrigger1` (or `GetHello`)
   * **Authorization level:** Function (requires function key)
7. Click **Create**

✅ **Expected Result:**
Function created with default code

---

##  Step 6: Review and Modify HTTP Function Code

1. Open the function → **Code + Test**
2. Review the default Node.js code:

```javascript
module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');
    const name = (req.query.name || req.body?.name) || 'World';
    context.res = {
        body: `Hello, ${name}!`
    };
};
```

3. Optionally customize:

```javascript
module.exports = async function (context, req) {
    context.log('HTTP trigger invoked');
    const name = (req.query.name || req.body?.name) || 'Azure Functions';
    context.res = {
        status: 200,
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
            message: `Hello, ${name}!`,
            timestamp: new Date().toISOString(),
            lab: 'Lab 07 - Azure Function App'
        })
    };
};
```

4. Click **Save**

---

##  Step 7: Get Function URL and Test

1. Open function → **Get Function URL**
2. Copy the full URL (includes `?code=<function-key>`)
3. Test in browser or curl:

```bash
curl "https://func-app-dev-xxx.azurewebsites.net/api/HttpTrigger1?name=Developer"
```

Or open URL in browser with `?name=YourName`

✅ **Expected Result:**
JSON response: `{"message":"Hello, Developer!","timestamp":"...","lab":"Lab 07 - Azure Function App"}`

---

##  Step 8: Test with Different HTTP Methods (Optional)

1. Use **Code + Test** → **Test/Run**
2. Configure:

   * **HTTP method:** POST
   * **Body:** `{"name": "Azure"}`
3. Click **Run**

✅ **Expected Result:**
Function returns response with `name` from request body

---

##  Step 9: Create a Timer-Triggered Function

1. Go to **Functions** → **+ Create**
2. Choose **Develop in portal**
3. Select template: **Timer trigger**
4. Configure:

   * **New Function:** `TimerTrigger1`
   * **Schedule:** `0 */5 * * * *` (every 5 minutes) or `0 0 * * * *` (hourly)
5. Click **Create**

---

##  Step 10: Review Timer Function Code

1. Open **TimerTrigger1** → **Code + Test**
2. Default code:

```javascript
module.exports = async function (context, myTimer) {
    const timeStamp = new Date().toISOString();
    context.log('Timer trigger function ran!', timeStamp);
};
```

3. Customize for lab:

```javascript
module.exports = async function (context, myTimer) {
    const timeStamp = new Date().toISOString();
    context.log('Lab 07 - Timer function executed at:', timeStamp);
    // Simulate scheduled task (e.g., cleanup, report)
    context.log('Scheduled task completed successfully');
};
```

4. Save

💡 **Learning Point:**
Timer uses **NCRONTAB** format: `{second} {minute} {hour} {day} {month} {day-of-week}`

---

##  Step 11: Manually Run Timer Function

1. Open **TimerTrigger1** → **Code + Test**
2. Click **Test/Run**
3. Provide empty input `{}`
4. Click **Run**

✅ **Expected Result:**
Function executes; check **Monitor** for execution logs

---

##  Step 12: Monitor Function Execution

1. Open any function → **Monitor**
2. Observe:

   * Invocation count
   * Success/failure
   * Execution duration
3. Click an invocation for details:

   * Logs
   * Input/output
   * Duration

💡 **Learning Point:**
Application Insights (if enabled) provides deeper analytics and traces.

---

##  Step 13: Enable Application Insights (Recommended)

1. Open **Function App** → **Application Insights**
2. Click **Turn on Application Insights**
3. Create new or use existing workspace
4. Apply

✅ Enables request tracking, failures, and performance metrics

---

##  Step 14: Configure Function App Settings

1. Open **Function App** → **Configuration**
2. Add **Application settings** (optional):

   * **Name:** `APP_ENV`
   * **Value:** `development`
3. Review **Function keys** (under **App keys**):

   * Default key used in function URL
   * Can create additional keys for different clients

---

##  Step 15: Validate via Azure CLI (Optional)

In **Cloud Shell**:

```bash
az functionapp list -g rg-app-dev-centralindia -o table
az functionapp function list -n func-app-dev-<uniqueid> -g rg-app-dev-centralindia -o table
az functionapp keys list -n func-app-dev-<uniqueid> -g rg-app-dev-centralindia
```

---

##  Step 16: Understand Triggers and Bindings (Conceptual)

Trainer explains:

| Trigger   | Use Case                    | Example                    |
| --------- | --------------------------- | -------------------------- |
| HTTP      | REST API, webhooks          | `api/GetHello`             |
| Timer     | Scheduled tasks             | Cleanup, reports           |
| Blob      | File upload/change          | Process new blobs          |
| Queue     | Message processing          | Process queue messages     |
| Event Grid| Event-driven                | React to Azure events      |

💡 **Binding** = Declarative connection to Azure services (Storage, Cosmos DB, etc.)

---

##  Lab Validation Checklist

✔ Storage account created/selected for Function App
✔ Function App created (Consumption plan)
✔ HTTP-triggered function created and tested
✔ Timer-triggered function created and executed
✔ Function URL accessible from browser/curl
✔ Monitor shows execution logs
✔ Application Insights enabled (optional)
✔ Triggers and bindings understood

---

##  Key Takeaways

* Azure Functions = serverless, event-driven compute
* Function App = container for functions; requires storage account
* Triggers start functions (HTTP, Timer, Blob, Queue, etc.)
* Bindings connect to Azure services without custom code
* Consumption plan = pay per execution; scales to zero
* Test via portal, browser, curl, or Postman
* Monitor and Application Insights provide observability
* Functions integrate with Azure ecosystem (Storage, Event Grid, etc.)
