# Snowflake - Stages

## Types of Stages

| # | Type    | Notes   |
| :---:   | :--- | :--- |
| 1 | Internal Stage   | <ul><li>Internal to Snowflake</li><li>3 types are available, user stage, table stage, named stage</li></ul>    |
| 2 | External Stage   | <ul><li>Microsoft Azure Blob storage, Amazon AWS S3, and Google Cloud Storage</li></ul>   |


## External Stage

### Unload 2 ADF

| # |App| Type    | Notes   |
| :---:   | :--- | :--- |:--- |
|1|Azure|Create a storage account in Azure|
|2|Snowflake|Create Storage Integration| Refer "Command : STORAGE INTEGRATION" below
|3|Snowflake|Run Describe command to Authorize| Refer Command : DESCRIBE INTEGRATION" below


#### Command : STORAGE INTEGRATION

- Note
  - Capture the tenant id
  - Capture the storage account name
  - Capture the container name

- Syntax
```
CREATE STORAGE INTEGRATION <storage-integration-name>
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = 'AZURE'
  ENABLED = TRUE
  AZURE_TENANT_ID = '<tenant_id>'
  STORAGE_ALLOWED_LOCATIONS = ('azure://myaccount.blob.core.windows.net/mycontainer/path1/', 'azure://myaccount.blob.core.windows.net/mycontainer/path2/');
```

- Example
```
CREATE STORAGE INTEGRATION sf_unload_2_adf
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = 'AZURE'
  ENABLED = TRUE
  AZURE_TENANT_ID = 'Get the tenand from Azure AAD'
  STORAGE_ALLOWED_LOCATIONS = ('azure://<ur-storage-account-name>.blob.core.windows.net/<your-container-name>/raw1/');

```



#### Command : DESCRIBE INTEGRATION
- Note
  -  Click on the AZURE_CONSENT_URL" and clik on the Accept
  -  Login into Azure AD and Click on the "Enterprise applications" and verify the "snowflake" account


- Syntax
```
DESC STORAGE INTEGRATION <storage-integration-name> ;
```

- Example
  
```
DESC STORAGE INTEGRATION sf_unload_2_adf;
```
