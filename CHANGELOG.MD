## 1.3
- Added support to ingest ".csv" and ".json" files apart from CloudTrail Events

## 1.2
- To avoid additional billing and duplication added additional environment variables CoreFieldsAllTable and SplitAWSResourceTypeTables to turn on/off LogAnalyticsTables
- Added CHANGELOG.MD to track code changes

## 1.1
- Added Secrets Manager to store Log Analytics Workspace (LAW) Id and Log Analytics Workspace (LAW) Key
- Added Environment Variable Secret Name to retrieve secrets from Secrets Manager

## 1.0
- Created AWS Lamba using PowerShell to ingest CloudTrail Logs to Azure Log Analytics Workspace
- Splitting the data if it is more than 25MB
- Using Environment variables for Log Analytics Workspace Id, Key and LogAnalyticsTableName
- Takes the core fields of the record. i.e. all fields except for the Request and Response associated fields and puts them in a LogAnalyticsTableName_ALL. Providing a single table with all records with core event information.	
- Looks at each event and puts it into a table with an extension <AWSREsourceType> i.e. LogAnalyticsTableName_S3 
- Exception to 2 above is for EC2 events, the volume of fields for EC2 Request and Response parameters exceeds 500 columns. EC2 data is split into 3 tables, Header, Request & Response. 
  Ex: LogAnalyticsTableName_EC2_Header
- Data being ingested into 2 or more different tables and increase the log ingestion metrics\billing