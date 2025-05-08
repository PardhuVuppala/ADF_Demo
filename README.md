## 📚 Overview
This document provides detailed information about the HubSpot data source ingestion connector, including the required credentials, supported data points for ingestion, supported tools and destinations, and any known limitations or considerations

## 🔑 Prerequisites
### 🛠️ Step 1: Create a HubSpot Private App

1. Go to your HubSpot dashboard.
2. Navigate to:  
   **Settings → Integrations → Private Apps**
3. Click **“Create a private app”**
4. Fill in:
   - App name
   - (Optional) Description
   - Choose the required **Scopes** 
   
| **Scope**                     | **Description**           |
| ----------------------------- | ------------------------- |
| `crm.objects.deals.read`      | Read access to deals      |
| `crm.objects.companies.read`  | Read access to companies  |
| `crm.objects.contacts.read`   | Read access to contacts   |
| `crm.objects.quotes.read`     | Read access to quotes     |
| `crm.objects.line_items.read` | Read access to line items |


5. Click **"Create app"**

---

### 📋 Step 2: Copy Your Access Token

- After creation, HubSpot will generate an **Access Token**
- **Copy this token** immediately – it is shown only once
- Use this token to authenticate API requests:

```http
Authorization: Bearer YOUR_ACCESS_TOKEN
```

## Data Source URL:
For the endpoint or URL refer the **hubspot_job_master** 

## 🧰 Supported Tools
- ✅ ADF (Azure Data Factory)
- ✅ Python (PySpark)

## Supported Destination
### Files
- ❌ JSON
- ✅ CSV
- ✅ Parquet
### Lakehouse
- ✅ Databricks: Delta Lake
- ✅ Fabric - Lakehouse
### **Data Warehouse / Database**
- ❌ Snowflake
- ✅ Fabric - Warehouse
- ✅ Azure SQL

### ⚠️ Limitations
   **Rate Limiting**:
    HubSpot enforces strict rate limits:
      Private apps (OAuth):
         - 100 requests every 10 seconds per app per account.
         - 250,000 daily requests per app per account.

   **Data Volume**:
      Large datasets, such as extensive contact lists or activity logs, may require batching and pagination (using HubSpot’s after or offset parameters) to ensure successful data retrieval without timeouts or rate-limit violations.

   **Data Types**:
      HubSpot APIs may return complex nested structures, especially within custom properties or engagement data. These structures may need flattening or transformation before ingestion, as some connectors cannot handle deeply nested JSON objects.

   **Time Zone Handling**:
      Timestamps from HubSpot are typically in UTC and represented in milliseconds since epoch. Manual conversion may be necessary to align with your local or business time zone requirements.

   **API Changes and Versioning**:
      HubSpot periodically deprecates and updates its API endpoints. Always monitor HubSpot's API changelog to stay informed and update your connector accordingly to prevent integration failures.
