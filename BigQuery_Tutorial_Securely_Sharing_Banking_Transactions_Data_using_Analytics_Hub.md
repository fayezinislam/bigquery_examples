# BigQuery Tutorial: Securely Sharing Banking Transactions Data using Analytics Hub

## License
```
Copyright 2025 Google LLC
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

## Overview
This tutorial guides you through loading banking transaction data into Google Cloud's BigQuery, creating a customer-specific authorized view for enhanced security, and then sharing that authorized view with a specific customer using BigQuery Analytics Hub.

---

## Prerequisites

Before you begin, make sure you have:

* **A Google Cloud Project:** If you don't have one, create a new project in the Google Cloud Console.
* **Billing Enabled:** Ensure billing is enabled for your Google Cloud project.
* **BigQuery API Enabled:** The BigQuery API is usually enabled by default, but it's good to confirm.
* **Basic understanding of SQL:** Familiarity with SQL queries will be helpful.
* **Sample Banking Transactions Data:** For this tutorial, we'll assume you have a CSV file named `banking_transactions.csv` with the following columns (you can create a small sample file for demonstration):
    * `transaction_id` (STRING)
    * `customer_id` (STRING)
    * `transaction_date` (DATE)
    * `amount` (NUMERIC)
    * `transaction_type` (STRING)
    * `description` (STRING)
    * `bank_account_number` (STRING) - *Sensitive data*

**Sample `banking_transactions.csv` content:**

```csv
transaction_id,customer_id,transaction_date,amount,transaction_type,description,bank_account_number
T001,C101,2024-05-01,150.75,Debit,Groceries,ACC123456789
T002,C102,2024-05-02,250.00,Credit,Salary,ACC987654321
T003,C101,2024-05-03,45.20,Debit,Coffee,ACC123456789
T004,C103,2024-05-04,1000.00,Credit,Loan Disbursement,ACC112233445
T005,C102,2024-05-05,75.00,Debit,Online Purchase,ACC987654321
T006,C101,2024-05-06,200.00,Debit,Utilities,ACC123456789
```

## **Part 1: Loading Banking Transactions Data into a BigQuery Table**

### **Step 1: Create a BigQuery Dataset**

A dataset is a top-level container for tables and views in BigQuery.

1. **Go to the BigQuery console:**  
   * In the Google Cloud Console, navigate to **BigQuery** (you can search for it in the search bar).  
   * Click on **BigQuery Studio** in the left navigation.  
2. **Create a new dataset:**  
   * In the Explorer pane (left side), click on your project ID.  
   * Click on the **"..." (More actions)** icon next to your project ID and select **"Create dataset"**.  
   * For **Dataset ID**, enter banking\_data.  
   * Choose a **Data location** (e.g., us-central1).  
   * Leave other settings as default for now.  
   * Click **"Create dataset"**.

### **Step 2: Upload banking\_transactions.csv to Cloud Storage (Optional but Recommended)**

For large datasets, loading from Cloud Storage is generally more efficient and reliable.

1. **Go to Cloud Storage:**  
   * In the Google Cloud Console, search for and navigate to **Cloud Storage**.  
2. **Create a new bucket (if you don't have one):**  
   * Click **"Create bucket"**.  
   * Enter a unique **Name** for your bucket (e.g., your-project-id-banking-data).  
   * Choose a **Region** that matches your BigQuery dataset location (e.g., us-central1).  
   * Leave other settings as default and click **"Create"**.  
3. **Upload the CSV file:**  
   * Navigate into your newly created bucket.  
   * Click **"Upload files"** and select your banking\_transactions.csv file.

### **Step 3: Load Data into a BigQuery Table**

We will now load the data from your CSV file (either local or from Cloud Storage) into a new BigQuery table.

1. **Go back to BigQuery Studio.**  
2. **Create a new table:**  
   * In the Explorer pane, expand your banking\_data dataset.  
   * Click on the **"..." (More actions)** icon next to banking\_data and select **"Create table"**.  
3. **Configure table creation:**  
   * **Source:**  
     * If uploading from your local machine: Select **"Upload"**. Click **"Browse"** and select your banking\_transactions.csv file.  
     * If loading from Cloud Storage: Select **"Google Cloud Storage"**. For **Select file from Cloud Storage bucket**, click **"Browse"** and navigate to your banking\_transactions.csv file in your Cloud Storage bucket.  
   * **File format:** Select **"CSV"**.  
   * **Destination:**  
     * **Project name:** Your project ID will be pre-selected.  
     * **Dataset name:** banking\_data will be pre-selected.  
     * **Table name:** Enter transactions.  
   * **Schema:**  
     * Check **"Auto detect"**. BigQuery will try to infer the schema.  
     * *(Optional)* If auto-detection isn't perfect or you want to be explicit, you can manually define the schema. Click **"Edit as text"** and paste the following:  

     ```JSON  
     [  
        {"name": "transaction_id", "type": "STRING"},  
        {"name": "customer_id", "type": "STRING"},  
        {"name": "transaction_date", "type": "DATE"},  
        {"name": "amount", "type": "NUMERIC"},  
        {"name": "transaction_type", "type": "STRING"},  
        {"name": "description", "type": "STRING"},  
        {"name": "bank_account_number", "type": "STRING"}  
      ]
     ```

   * **Advanced options:**  
     * **Header rows to skip:** Enter 1 (since your CSV has a header).  
   * Click **"Create table"**.

BigQuery will initiate a load job. Once complete, you'll see the transactions table under your banking\_data dataset. You can click on the table and then the **"Preview"** tab to verify the data.

## **Part 2: Creating a Customer-Specific Authorized View**

We want to share transaction data with a specific customer (e.g., C101) but *without* revealing sensitive information like bank\_account\_number and only showing their own transactions.

### **Step 1: Create a Dataset for Views**

It's a best practice to create a separate dataset for authorized views. This helps with permission management.

1. **Go to BigQuery Studio.**  
2. **Create a new dataset:**  
   * In the Explorer pane, click on your project ID.  
   * Click on the **"..." (More actions)** icon next to your project ID and select **"Create dataset"**.  
   * For **Dataset ID**, enter customer\_views.  
   * Choose the same **Data location** as your banking\_data dataset.  
   * Click **"Create dataset"**.

### **Step 2: Create the Customer-Specific Authorized View**

This view will filter data for a specific customer and exclude sensitive columns.

1. **Go to BigQuery Studio.**  
2. **Open the query editor:** Click **"+ Compose new query"**.  
3. **Enter the SQL for the view:** Replace YOUR\_PROJECT\_ID with your actual Google Cloud project ID.  
   ```SQL  
   CREATE VIEW `YOUR_PROJECT_ID.customer_views.customer_C101_transactions` AS  
   SELECT  
       transaction_id,  
       customer_id,  
       transaction_date,  
       amount,  
       transaction_type,  
       description  
   FROM  
       `YOUR_PROJECT_ID.banking_data.transactions`  
   WHERE  
       customer_id = 'C101';
   ```
4. **Save the view:**  
   * Click **"Save"** \> **"Save view"**.  
   * For **Project**, your project ID should be selected.  
   * For **Dataset**, select customer\_views.  
   * For **Table**, enter customer\_C101\_transactions.  
   * Click **"Save"**.

You will now see customer\_C101\_transactions under the customer\_views dataset in the Explorer pane.

### **Step 3: Authorize the View to Access the Source Data**

For an authorized view to work, it needs permission to query its underlying tables. You grant this permission at the dataset level.

1. **Go to BigQuery Studio.**  
2. **Select the source dataset:** In the Explorer pane, expand your project and select the banking\_data dataset (the one containing the transactions table).  
3. **Authorize views:**  
   * In the right-hand panel, click **"Sharing"** \> **"Authorize views"**.  
   * In the **Authorized views** pane, click **"Add authorization"**.  
   * For **Authorized view**, click **"Browse"** and select YOUR\_PROJECT\_ID.customer\_views.customer\_C101\_transactions.  
   * Click **"Add authorization"**.  
   * Click **"Close"**.

Now, customer\_C101\_transactions can query data from banking\_data.transactions.

### **Step 4: Grant Access to the Customer (Individual User/Service Account)**

To allow the customer to query their specific view, you grant them BigQuery Data Viewer access to the customer\_views dataset.

1. **Identify the customer's identity:** This would typically be a Google Account email address or a service account email address. For this tutorial, let's assume customer-c101@example.com is the customer's email.  
2. **Go to BigQuery Studio.**  
3. **Select the views dataset:** In the Explorer pane, expand your project and select the customer\_views dataset.  
4. **Manage permissions:**  
   * In the right-hand panel, click **"Sharing"** \> **"Permissions"**.  
   * Click **"Add principal"**.  
   * For **New principals**, enter customer-c101@example.com.  
   * For **Select a role**, choose **"BigQuery"** \> **"BigQuery Data Viewer"**.  
   * Click **"Save"**.

Now, customer-c101@example.com can query customer\_C101\_transactions and only see data for C101 without access to bank\_account\_number or other customer's data. They do *not* need access to the underlying banking\_data dataset.

## **Part 3: Sharing the Authorized View using BigQuery Analytics Hub**

BigQuery Analytics Hub allows you to create data exchanges to share data products securely and at scale.

### **Step 1: Enable BigQuery Analytics Hub API**

1. **Go to the Google Cloud Console.**  
2. **Search for "BigQuery Analytics Hub API"** and ensure it's **"Enabled"**.

### **Step 2: Create a Data Exchange**

A data exchange is a container for your data listings.

1. **Go to BigQuery Analytics Hub:**  
   * In the Google Cloud Console, search for and navigate to **"Analytics Hub"**.  
   * Click on **"Data exchanges"**.  
2. **Create a new exchange:**  
   * Click **"Create exchange"**.  
   * **Project:** Select your current project.  
   * **Region:** Choose the same region as your BigQuery datasets (e.g., us-central1).  
   * **Display name:** Enter Banking Data Exchange.  
   * *(Optional)* Add a **Description** and **Primary contact**.  
   * You can choose to make it **Publicly discoverable** or keep it private. For this tutorial, we'll keep it private.  
   * Click **"Create Exchange"**.  
   * *(Optional)* You can add **Exchange Permissions** here to define administrators, publishers, and subscribers at the exchange level. For now, you can skip this as we'll manage access at the listing level for a specific customer.

### **Step 3: Create a Data Listing for the Authorized View**

A data listing represents a data product (your authorized view) that you want to share.

1. **Navigate to your data exchange:** From the Analytics Hub page, click on your Banking Data Exchange.  
2. **Create a new listing:**  
   * Click **"Create listing"**.  
   * **Resource type:** Select **"BigQuery dataset"**.  
   * **BigQuery dataset:** Click **"Browse"** and select customer\_views. (Analytics Hub shares datasets, and the authorized view is within this dataset).  
   * Click **"Next"**.  
   * **Display name:** Enter Customer C101 Transactions.  
   * *(Optional)* Add a **Description** and **Primary contact**.  
   * **Linkable tables and views:** Ensure that customer\_C101\_transactions is listed here. This confirms that the authorized view will be exposed through this listing.  
   * Click **"Next"**.  
   * **Permissions:** This is where you specify who can subscribe to this listing.  
     * Click **"Add principal"**.  
     * For **New principals**, enter the email address of your customer (e.g., customer-c101@example.com).  
     * For **Select a role**, choose **"Analytics Hub"** \> **"Analytics Hub Subscriber"**.  
     * Click **"Add principal"**.  
   * Click **"Create listing"**.

Now, your customer\_C101\_transactions view is available as a listing within your Banking Data Exchange in Analytics Hub, and customer-c101@example.com is authorized to subscribe to it.

## **Part 4: Listing for the Data (as the Customer) and Accessing it**

Now, let's switch hats and act as the customer (customer-c101@example.com) to subscribe to and query the shared data.

### **Step 1: Access Analytics Hub as the Customer**

The customer needs to be in their own Google Cloud Project. For this demonstration, you might need to use a different Google Cloud account or a service account with the appropriate permissions.

**Customer's Prerequisites:**

* A Google Cloud Project (e.g., customer-project-id).  
* Billing enabled for their project.  
* BigQuery API enabled in their project.  
* The customer user/service account must have the BigQuery User role (roles/bigquery.user) on their own project, which includes bigquery.datasets.create permission to create the linked dataset.  
1. **Log in as the customer:**  
   * Have customer-c101@example.com log into the Google Cloud Console.  
   * Ensure they are in their own Google Cloud Project (e.g., customer-project-id).  
2. **Go to Analytics Hub:**  
   * Search for and navigate to **"Analytics Hub"**.  
   * Click on **"Listings"**.

### **Step 2: Discover and Subscribe to the Listing**

1. **Find the listing:**  
   * The customer should see Customer C101 Transactions listed under the Banking Data Exchange (if they have the Analytics Hub Subscriber role on it).  
   * Click on the Customer C101 Transactions listing.  
2. **Subscribe to the listing:**  
   * Click **"Subscribe"**.  
   * **Project:** The customer's project ID will be pre-selected.  
   * **Linked dataset name:** Enter a name for the linked dataset in the customer's project (e.g., my\_bank\_transactions). This dataset will act as a pointer to the shared data.  
   * Click **"Save"**.

### **Step 3: Query the Linked Dataset**

Once subscribed, a linked dataset is created in the customer's BigQuery project.

1. **Go to BigQuery Studio (as the customer).**  
2. **Locate the linked dataset:** In the Explorer pane, under the customer's project, they will now see a new dataset named my\_bank\_transactions.  
3. **Query the data:**  
   * Expand my\_bank\_transactions and they will see the customer\_C101\_transactions view.  
   * Click on the customer\_C101\_transactions view, and then click **"Query"** \> **"In new tab"**.  
   * Enter a query, for example:

     ```SQL  
     SELECT
         *
     FROM
         `customer-project-id.my_bank_transactions.customer_C101_transactions`
     LIMIT 100;
     ```

     *(Remember to replace customer-project-id with the customer's actual project ID).*  
   * Click **"Run"**.

The customer will see only the banking transactions for customer\_id \= 'C101' and the bank\_account\_number column will *not* be present, demonstrating the effectiveness of the authorized view.

## **Summary and Key Takeaways**

This tutorial demonstrated a secure and scalable way to share specific subsets of data in BigQuery:

* **BigQuery Data Loading:** Efficiently load your raw data into BigQuery tables.  
* **Authorized Views:** Create views that filter rows (e.g., customer\_id) and project specific columns (excluding sensitive data). This provides a powerful layer of security and data governance.  
* **BigQuery Analytics Hub:** Establish a data exchange to catalog and share your authorized views as "listings." This simplifies data discovery and access for your data consumers (customers, partners, internal teams).  
* **Granular Permissions:** BigQuery's IAM and Analytics Hub's subscription model ensure that consumers only access the data they are authorized to see, without direct access to the underlying raw data.

By following these steps, you can confidently manage and share your banking transactions data while maintaining strict data privacy and security.
