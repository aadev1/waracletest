# Summary 
This solution creates resources and pipelines required to host a static web app and function app in Azure.  The resource are built using ARM templatea and are listed below

- Storage account to host static website
- CDN to sit infront of the storage account
- A CDN Endpoint with the static website configured as it's backend
- App Service Plan to host the funtion app
- Function App
- Application Insights
- Storage account for the Function App

The pipelines consist of 4 stages
- Publish Artifacts
- CI
- Dev
- Prod 

The Production environment has an approval gate on it, so the relevant people can review the change before deploying to Production.

# How to set up locally
To run the Static Website from a local machine in your own dev subscription.

```

# Create the infrastructure
$rgname = "<resource group>"
New-AzResourceGroupDeployment -ResourceGroupName $rgname -TemplateFile "Source\ARM\azuredeploy.json" -TemplateParameterFile "Source\ARM\azuredeploy.dev.parameters.json" -Verbose

# Enable the static website in the Storage account
$storageaccountName = 'stwrcltestdev'
az storage blob service-properties update --account-name $storageaccountName --static-website --404-document error.html --index-document index.html

# Upload your html file to the storage account 
$accountKey = az storage account keys list --account-name $storageaccountName --resource-group $rgname --query [0].value -o tsv
az storage blob upload --account-name $storageaccountName --container-name '$web' --name 'index.html' --file 'Source\StaticContent\index.html' --account-key $accountKey --debug --verbose

```
# Build and Test
To build and test the Static Website from a local machine in your own dev subscription run the below command changing the variables accordingly.

```

# Create the infrastructure and upload the static content
$rgname = "<resource group>"
New-AzResourceGroupDeployment -ResourceGroupName $rgname -TemplateFile "Source\ARM\azuredeploy.json" -TemplateParameterFile "Source\ARM\azuredeploy.dev.parameters.json" -Verbose

# Enable the static website in the Storage account
$storageaccountName = 'stwrcltestdev'
az storage blob service-properties update --account-name $storageaccountName --static-website --404-document error.html --index-document index.html

# Upload your html file to the storage account 
$accountKey = az storage account keys list --account-name $storageaccountName --resource-group $rgname --query [0].value -o tsv
az storage blob upload --account-name $storageaccountName --container-name '$web' --name 'index.html' --file 'Source\StaticContent\index.html' --account-key $accountKey --debug --verbose

```

To Test the CDN, Storage Account and static website are all connected run the below command changing the variables accordingly

```
$cdnEndpoint = "https://epwrcltestdev.azureedge.net"
$statuscode = Invoke-WebRequest -Uri $cdnEndpoint | Select-Object -Expand StatusCode
if ($statuscode -eq 200)
{
  Write-Host "SUCCESS: CDN test retured a status $statuscode"
} else {
  Write-Host "FAILED: CDN test retured a status $statuscode"
}

```

To build and test the function app locally follow the instructions in the link below:
[Develop Azure Functions by using Visual Studio Code](https://docs.microsoft.com/en-us/azure/azure-functions/functions-develop-vs-code?tabs=powershell) 

# Deploy
The deployment process for the Static Website deployment is convered above in the Build and Test section.

To deploy the function app, change the variables below accordingly and deploy to your Dev subscription.

```

$rgname = "rg-wrcltest-dev"
New-AzResourceGroupDeployment -ResourceGroupName $rgname -TemplateFile "api-layer\arm\azuredeploy.json" -TemplateParameterFile "api-layer\arm\azuredeploy.parameters.dev.json" -Verbose

```

To publish the function app follow the publish section in the link below:
[Develop Azure Functions by using Visual Studio Code](https://docs.microsoft.com/en-us/azure/azure-functions/functions-develop-vs-code?tabs=powershell) 

To test the API after deployment run the below powershell, changing the variables appropriately

```

$FunctionAppName = "func-wrcltest-dev"
$GroupName = "kk3157b"
$TriggerName = "HttpTrigger1"
$FunctionApp = Get-AzWebApp -ResourceGroupName $GroupName -Name $FunctionAppName
$Id = $FunctionApp.Id
$DefaultHostName = $FunctionApp.DefaultHostName
$FunctionKey = (Invoke-AzResourceAction -ResourceId "$Id/functions/$TriggerName" -Action listkeys -Force).default
$FunctionURL = "https://" + $DefaultHostName + "/api/" + $TriggerName + "?code=" + $FunctionKey
Invoke-WebRequest -Uri $FunctionURL | Select-Object -Expand StatusCode

```
