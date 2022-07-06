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
## Features



Markdown is a lightweight markup language based on the formatting conventions
that people naturally use in email.
As [John Gruber] writes on the [Markdown site][df1]

> The overriding design goal for Markdown's
> formatting syntax is to make it as readable
> as possible. The idea is that a
> Markdown-formatted document should be
> publishable as-is, as plain text, without
> looking like it's been marked up with tags
> or formatting instructions.

This text you see here is *actually- written in Markdown! To get a feel
for Markdown's syntax, type some text into the left window and
watch the results in the right.



## Installation

Dillinger requires [Node.js](https://nodejs.org/) v10+ to run.

Install the dependencies and devDependencies and start the server.

```sh
cd dillinger
npm i
node app
```

For production environments...

```sh
npm install --production
NODE_ENV=production node app
```

## Plugins

Dillinger is currently extended with the following plugins.
Instructions on how to use them in your own application are linked below.

| Plugin | README |
| ------ | ------ |
| Dropbox | [plugins/dropbox/README.md][PlDb] |
| GitHub | [plugins/github/README.md][PlGh] |
| Google Drive | [plugins/googledrive/README.md][PlGd] |
| OneDrive | [plugins/onedrive/README.md][PlOd] |
| Medium | [plugins/medium/README.md][PlMe] |
| Google Analytics | [plugins/googleanalytics/README.md][PlGa] |

## Development

Want to contribute? Great!

Dillinger uses Gulp + Webpack for fast developing.
Make a change in your file and instantaneously see your updates!

Open your favorite Terminal and run these commands.

First Tab:

```sh
node app
```

Second Tab:

```sh
gulp watch
```

(optional) Third:

```sh
karma test
```

#### Building for source

For production release:

```sh
cd dillinger
docker build -t <youruser>/dillinger:${package.json.version} .
```

This will create the dillinger image and pull in the necessary dependencies.
Be sure to swap out `${package.json.version}` with the actual
version of Dillinger.


```sh
docker run -d -p 8000:8080 --restart=always --cap-add=SYS_ADMIN --name=dillinger <youruser>/dillinger:${package.json.version}
```

> Note: `--capt-add=SYS-ADMIN` is required for PDF rendering.

Verify the deployment by navigating to your server address in
your preferred browser.

```sh
127.0.0.1:8000
```

## License

MIT

**Free Software, Hell Yeah!**
