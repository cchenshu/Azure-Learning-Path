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
- *App registration*
- *Go to Certificates & secrets, create client secret, should save the secret id and value*

✨Create an Azure key vault instance and secret scope
- *Creating Azure Key Vault, go to Secrets, create new secret, set secret value equal to client secret value in AAD*
- *Create Azure Databricks instance*
  - *https://<per-workspace-url>/#secrets/createScope*
  - *Manage Principal----Creator(Premium tier for ADB)/All Users(Standard)*
- *Save the client secret in the Azure key vault*
  -*Go to storage account, Access Control--Role Assignment--Add aad app as Owner*

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
`Issue: Should disable soft delete for blobs/containers @ Storage--> DataProtection`
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
  ###### _About Spark driver logs_

Spark driver logs are composed of Standard output, Standard error and Log4j output logs.
And all key Spark jobs processing logs could be found in Log4j output logs, which could be sent to Azure Log Analytics workspace with the help of [spark-monitoring](https://github.com/mspnp/spark-monitoring) library.   
  
_Tip : make sure the name of the cluster does not contain ' single quotes_
  
###### _Setup spark-monitoring in Databricks Spark cluster_
Install Databricks CLI, following [this guide](https://docs.microsoft.com/en-us/azure/databricks/dev-tools/cli/).

Install JDK of version 1.8 or higher.

###### _Building spark-monitoring library jars_
Clone repository spark-monitoring repo.
```sh
  git clone https://github.com/mspnp/spark-monitoring
```
Build jars following either Docker or Maven option described in the [README](https://github.com/mspnp/spark-monitoring/blob/main/README.md#build-the-azure-databricks-monitoring-library) markdown.

###### _Upload shell and jars to DBFS of Databricks_
  ```sh
    # Skip below two commands if it's been done previously
    # run in git bash
    export DATABRICKS_TOKEN=<Databricks PAT token>
    export DATABRICKS_HOST=<Databricks workspace URL>
  ```
Edit /spark-monitoring/src/spark-listeners/scripts/spark-monitoring.sh file, to save the target [_Log Analytics workspace ID and access key_](https://docs.microsoft.com/en-us/azure/azure-monitor/agents/agent-windows#workspace-id-and-key) to below entries, and those two values could be found here.
  ```sh
    export LOG_ANALYTICS_WORKSPACE_ID={workspace ID}
    export LOG_ANALYTICS_WORKSPACE_KEY={access key}
  ```
  ```sh
    # MAKE DIRECTORY AND UPLOAD .SHELL FILE
    dbfs mkdirs dbfs:/databricks/spark-monitoring 
    dbfs cp <local path to spark-monitoring.sh> dbfs:/databricks/spark-monitoring/spark-monitoring.sh
  ```
  ```sh
    # UPLOAD .JAR FILES
    #.sh and jar files are in the same directory: /databricks/spark-monitoring/ 
    dbfs cp --overwrite --recursive <local path to target folder> dbfs:/databricks/spark-monitoring/ 
    dbfs cp --overwrite --recursive ~/Downloads/target dbfs:/databricks/spark-monitoring/
  ```
###### _Configure Databrick Spark cluster_
Add the copied spark-monitoring shell as _Init Scripts_ of the target Databricks Spark cluster. 
```  
Type          DBFS 
File path     dbfs:/databricks/spark-monitoring/spark-monitoring.sh    
``` 
  
###### _Check Spark cluster Log4j output logs in Log Analytics workspace_
    SparkListenerEvent_CL https://github.com/mspnp/spark-monitoring/blob/main/README.md#sparklistenerevent_cl
    SparkMetric_CL https://github.com/mspnp/spark-monitoring/blob/main/README.md#sparkmetric_cl
Go to Log Analytics workspace and check relevant Log4j output logs by querying _SparkLoggingEventCL_ table from _Custom Logs_ category.
```sh
  # Kusto queries
  # Query all Log4j output logs
  SparkLoggingEventCL
  # Query error logs
  SparkLoggingEventCL
  | where Level == "ERROR"
  ```

