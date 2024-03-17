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
#### Command : Copy Into Params

- HEADER = FALSE 
  - default it is set to TRUE, you can change it to FALSE if you do not want column names of a header on the output file.
- file_format = (type=CSV COMPRESSION = BZ2)
  -  default, it unloads a file into GZIP format
  -  AUTO | GZIP | BZ2 | BROTLI | ZSTD | DEFLATE | RAW_DEFLATE | NONE
-  file_format = (type=CSV COMPRESSION = NONE FIELD_DELIMITER='|')
  - default it used ‘,’ character   
- file_format = (type=CSV COMPRESSION = NONE RECORD_DELIMITER='\n\r')
- file_format = (type=CSV COMPRESSION = NONE FILE_EXTENSION='TXT');
- file_format = (type=CSV COMPRESSION = NONE NULL_IF=(''))
  - Converts a value to NULL
- file_format = (type=CSV COMPRESSION = NONE EMPTY_FIELD_AS_NULL='')
  - Changes empty field to null
- file_format = (type=CSV COMPRESSION = NONE DATE_FORMAT='MM-DD-YYYY')
  - Change Date to a specified format
  - By default date columns outputs in ‘YYYY-MM-DD‘,
- file_format = (type=CSV COMPRESSION = NONE TIME_FORMAT='HH24:MI:SS')
  - By default time columns outputs in ‘HH24:MI:SS ‘
- file_format = (type=CSV COMPRESSION = NONE TIMESTAMP_FORMAT='MM-DD-YYYY HH24:MI:SS.FF3 TZHTZM')
  - By default timestamp columns outputs in ‘YYYY-MM-DD HH24:MI:SS.FF3 TZHTZM‘,    

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



## References
### File format Type Options
|Format Type Option|Description|Notes
| :---:   | :--- | :--- |
|COMPRESSION|AUTO or GZIP or BZ2 or BROTLI or ZSTD or DEFLATE or RAW_DEFLATE or NONE|
|RECORD_DELIMITER |‘<character>’ or NONE|
|FIELD_DELIMITER|‘<character>’ or NONE|
|FILE_EXTENSION|‘<string>’|
|SKIP_HEADER|<integer>|
|SKIP_BLANK_LINES|TRUE or FALSE|
|DATE_FORMAT|‘<string>’ or AUTO|
|TIME_FORMAT|‘<string>’ or AUTO|
|TIMESTAMP_FORMAT|‘<string>’ or AUTO|
|BINARY_FORMAT|HEX or BASE64 or UTF8|
|ESCAPE|‘<character>’ or NONE|
|ESCAPE_UNENCLOSED_FIELD|‘<character>’ or NONE|
|TRIM_SPACE|TRUE or FALSE|
|FIELD_OPTIONALLY_ENCLOSED_BY|‘<character>’ or NONE|
|NULL_IF|( ‘<string>’ [ , ‘<string>’ … ] )|
|ERROR_ON_COLUMN_COUNT_MISMATCH|TRUE or FALSE|
|REPLACE_INVALID_CHARACTERS|TRUE or FALSE|
|VALIDATE_UTF8|TRUE or FALSE|
|EMPTY_FIELD_AS_NULL|TRUE or FALSE|
|SKIP_BYTE_ORDER_MARK|TRUE or FALSE|
|ENCODING|‘<string>’ or UTF8|


### Snowflake File format Support

|Format Type Option|Snowflake-Load|Snowflake-Unload
| :---:   | :---: | :---: |
|Avro|Yes|No
|Orc|Yes|No
|XML|Yes|No

### Alter
ALTER FILE FORMAT [ IF EXISTS ] <name> RENAME TO <new_name>

ALTER FILE FORMAT file_format_name  SET 
   FIELD_DELIMITER = ','
   SKIP_HEADER  = 1


### Show
SHOW FILE FORMATS
SHOW FILE FORMATS in database_name.schema_name

### Clone

CREATE [ OR REPLACE ] { STAGE | FILE FORMAT | SEQUENCE | TASK } [ IF NOT EXISTS ] <object_name>
  CLONE <source_object_name>

### Describe
desc file format file_format_name;

### Drop
DROP File Format File_format_name

