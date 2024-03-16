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

```
CREATE STORAGE INTEGRATION <storage-integration-name>
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = 'AZURE'
  ENABLED = TRUE
  AZURE_TENANT_ID = '<tenant_id>'
  STORAGE_ALLOWED_LOCATIONS = ('azure://myaccount.blob.core.windows.net/mycontainer/path1/', 'azure://myaccount.blob.core.windows.net/mycontainer/path2/');
```

#### Command : DESCRIBE INTEGRATION

```
DESC STORAGE INTEGRATION <storage-integration-name> ;
```


