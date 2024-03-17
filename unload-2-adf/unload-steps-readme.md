# Snowflake - Unload data from Snowflake 2 Azure

## Steps

### Unload 2 ADF

| # |App| Type    | Notes   |
| :---:   | :--- | :--- |:--- |
|1|Azure|Create a storage account in Azure|
|2|Snowflake|Create Storage Integration| Refer "Command : STORAGE INTEGRATION" below
|3|Snowflake|Run Describe command to Authorize Snowflake in Azure| Refer Command : DESCRIBE INTEGRATION" below
|4|Azure|Grant Access in Azure to Snowflake for storage | Refer to "Command : Grant Access to Azure Storage for Snowflake"
|5|Snowflake|Test the above steps|



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

#### Command : Grant Access to Azure Storage for Snowflake

- Grant the Azure Service Principal the Desired Access to Each Storage Container
- Steps
  - Add a new role assignment under the Access Control (IAM) pane from within the storage container
 
  
| # | App | Type | Notes |
| :---:   | :--- | :--- |:--- |
|1|Azure|Login to Azure||
|2|Azure|Go to Storage account||
|3|Azure|Click on Access Control (IAM)||
|4|Azure|Click on Add -> Add Role Assignment||
|5|Azure|Add Contributor role|Storage Blob Data Contributor|
|6|Azure|grant access to service principal |<ul><li>get the snowflake service principal name from Azure AD -> Enterprise |
|7|Snowflake|Create File Format||
|8|Snowflake|Create Stage||

#### Command : Create File Format

```
create or replace file format sf_unload_2_adf_csv_file_format
type = csv field_delimiter=',' record_delimiter='\n'
field_optionally_enclosed_by='"' compression='auto'
skip_header=0 null_if=('')
date_format='auto' timestamp_format='auto';
```

#### Command : Create Stage

```
create or replace stage "<database_name>"."<schema_name>"."<stage_name>"
 STORAGE_INTEGRATION = <storage-integration-name>
 URL = 'azure://<strg-name>.blob.core.windows.net/<cntnr-name>/raw1/'
 FILE_FORMAT = sf_unload_2_adf_csv_file_format;
```

#### Command : Copy Into CSV

- This command will copy the data from snowflake to Azure in compressed format

```
COPY INTO @<stage-name>
from (select top 10 * from users) 
FILE_FORMAT = (FORMAT_NAME = sf_unload_2_adf_csv_file_format)
OVERWRITE=TRUE;
```

#### Command : Copy Into Parquet

```
COPY INTO @sf_unload_2_adf_stage 
from (select top 10 * from users)
file_format = (type = 'parquet')
header = true overwrite = true;
```

