# Environment Setting
##### _Procedure_
- setup env (dbx, adls, kv)
- code & notebook deploy
- data generator
- log analysis Query/Power BI 
- automation

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)
##### _Step one_
[Access Azure Data Lake Storage Gen2 using OAuth 2.0 with an Azure service principal](https://docs.microsoft.com/en-us/azure/databricks/data/data-sources/azure/adls-gen2/azure-datalake-gen2-sp-access)

Description:  allow access to storage account

Purpose:  decouple the application and data (for data protection)


✨Register an Azure Active Directory application

  ✨Create an Azure key vault instance and secret scope
  - *Should save the value when creating Azure Key Vault*
  - *Create Azure Databricks instance*
 - *https://<per-workspace-url>/#secrets/createScope*
  - *Manage Principal----Creator(Premium tier for ADB)/All Users(Standard)*
- *Save the client secret in the Azure key vault*

✨Assign roles
  
  ✨Create a container
  
✨Mount ADLS Gen2 storage
```
# Configure authentication for mounting
configs = {"fs.azure.account.auth.type": "OAuth",
          "fs.azure.account.oauth.provider.type": "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",
          "fs.azure.account.oauth2.client.id": "<application-id>",
          "fs.azure.account.oauth2.client.secret": dbutils.secrets.get(scope="<scope-name>",key="<service-credential-key-name>"),
          "fs.azure.account.oauth2.client.endpoint": "https://login.microsoftonline.com/<directory-id>/oauth2/token"}
```
`Issue: Enable soft delete for blobs/containers @ Storage--> DataProtection`
```
# Mount filesystem
dbutils.fs.mount(
  source = "abfss://<container-name>@<storage-account-name>.dfs.core.windows.net/",
  mount_point = "/mnt/<mount-name>",
  extra_configs = configs)
```
##### _Step two_
✨Create Cluster
- *Add Wheel files and Pypi Package in Libraries*
  
 `Issue: Cluster terminated when starting for a while. 
  Reason: Azure Operation Not Allowed Exception -- checking event logs
  Error Message: Operation could not be completed as it results in exceeding approved Total Regional Cores quota.
  Solution: Change region when creating Azure DataBricks`

✨Create Jobs
  
 `Issue: Change path (storing rz/pz/cz) in config.json  `
  
✨Store  master data  to
- *testcontainer/config/hon_edm_dynamo_customers/customer_psr/master_data*
  
##### _Step three_
✨Azure Databricks monitoring with Azure Log Analytics
  

**Yeah!**
