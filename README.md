# AWS Lambda Function to import CloudTrail Logs to Azure Sentinel
This Lambda function is desgined to read SNS Files and send them to Azure Sentinel using the Log Analytics API.

Flow:
CloudTrail Logs --> AWS S3 --> AWS SNS Topic --> AWS Lambda --> Azure Log Analytics
![Picture9](./Graphics/Picture9.png)

## Deployment
### Option 1
### Machine Setup
To deploy this, you will need a machine prepared with the following:
 - PowerShell Core – I recommend PowerShell 7 [found here](https://github.com/PowerShell/PowerShell/releases)
 - .Net Core 3.1 SDK [found here](https://dotnet.microsoft.com/download) 
 - AWSLambdaPSCore module – You can install this either from the [PowerShell Gallery](https://www.powershellgallery.com/packages?q=AWSLambdaPSCore), or you can install it by using the following PowerShell Core shell command:  
```powershell
Install-Module AWSLambdaPSCore -Scope CurrentUser
```
See the documentation here https://docs.aws.amazon.com/lambda/latest/dg/powershell-devenv.html 

I recommend you review https://docs.aws.amazon.com/lambda/latest/dg/powershell-package.html to review the cmdlets that are part of AWSLambdaPSCore.

Note: If the environment uses a proxy, you may need to add the following to VSCode profile
```powershell
Added to VS Code profile:
$webclient=New-Object System.Net.WebClient
$webclient.Proxy.Credentials = [System.Net.CredentialCache]::DefaultNetworkCredentials
```

### Create the Lambda Function
To deploy the PowerShell script, you can create a Package (zip file) to upload to the AWS console or you can use the Publish-AWSPowerShell cmdlet.
```powershell
Publish-AWSPowerShellLambda -Name YourLambdaNameHere -ScriptPath <path>/SNStoAzSentinel.ps1 -Region <region> -IAMRoleArn <arn of role created earlier> -ProfileName <profile>
```
You might need –ProfileName if your configuration of .aws/credentials file doesn't contain a default.  See this document for information on setting up your AWS credentials. 

### Option 2
1.	Create a new AWS Lambda and select "Author from scratch"
2.	Give Function Name and select Runtime ".NET Core 2.1 (C#/PowerShell)" and click Create function
3.	After successful creation, now you can change its code and configuration 
4.	Under Function code, click on Actions --> Upload a .zip file (/aws-data-connector-az-sentinel/blob/main/SNStoAzSentinel.zip)
5.	Follow the steps in "### Lambda Configuration" from step 2

### Note: Either you choose Option 1/Option 2, the following configuration steps are mandatory.

### Lambda Configuration
1. Once created, login to the AWS console.   In Find services, search for Lambda.  Click on Lambda.
![Picture1](./Graphics/Picture1.png)
2. Click on the lambda function name you used with the cmdlet.  Click Environment Variables and add the following
```
WorkspaceId
WorkspaceKey
LogAnalyticsTableName
```
![Picture4](./Graphics/Picture4.png)
3. Click on the lambda function name you used with the cmdlet. Click Add Trigger 
![Picture2](./Graphics/Picture2.png)
4. Select SNS.  Select the SNS Name.  Click Add. 
![Picture3](./Graphics/Picture3.png)
5. Create AWS Role : The Lambda function will need an execution role defined that grants access to the S3 bucket and CloudWatch logs.  To create an execution role: 
	1. Open the [roles](https://console.aws.amazon.com/iam/home#/roles) page in the IAM console. 
	2. Choose Create role. 
	3. Create a role with the following properties. 
		 - Trusted entity – AWS Lambda. 
		 - Permissions – AWSLambdaBasicExecutionRole &  AmazonS3ReadOnlyAccess . 
		 - Role name – AWSSNStoAzureSentinel. 

	The AWSLambdaExecute policy has the permissions that the function needs to manage objects in Amazon S3 and write logs to CloudWatch Logs.  Copy the arn of the role created as you will need it for the next step. 

6. Your lambda function is ready to send data to Log Analytics.

### Test the function
1. To test your function, Perform some actions like Start EC2, Stop EC2, Login into EC2, etc.,. 
2. To see the logs, go the Lambda function.  Click Monitoring tab.  Click view logs in CloudWatch. 
![Pciture5](./Graphics/Picture5.png)
3. In CloudWatch, you will see each log stream from the runs.  Select the latest.   
![Picture6](./Graphics/Picture6.png)
4. Here you can see anything from the script from the Write-Host cmdlet. 
![Picture7](./Graphics/Picture7.png)
5. Go to portal.azure.com and verify your data is in the custom log. 
![Picture8](./Graphics/Picture8.png)