# vmss diagnostics extension
describes how azure diagnostics can be enabled on a Virtual Machine Scale Set. The first part covers the steps to enable this for a VMSS running Windows VMs. The steps to enable these for a VMSS running Linux VMs follow it.

## Using Azure CLI - Windows VM based Scale Sets
These steps can be performed on an existing Virtual Machine Scale Set running Windows VMs. In the example here, a VMSS with the following properties pre-exist in Azure before enabling the Diagnostics extension.

VMSS Name - palvmss
VMSS Resource Group Name: palvmssrg

### Pre-requisites:
- Azure CLI on the local Computer. Details on this [here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)

### Create a Storage Account to store the Diagnostics from the Scale Set
A Storage Account with the name palvmssstr is created in Azure

### Create the Protected and Public Configuration Settings for the Diagnostics Extension

#### Public Settings Configuration
The Storage Account name needs to be entered in this Json Document. Save this file to the Working folder on your Computer (PublicSettings.json).
````
{
    "WadCfg": {
        "DiagnosticMonitorConfiguration": {
            "overallQuotaInMB": 10000,
            "DiagnosticInfrastructureLogs": {
                "scheduledTransferLogLevelFilter": "Verbose"
            },
            "PerformanceCounters": {
                "scheduledTransferPeriod": "PT1M",
                "sinks": "AzureMonitorSink",
                "PerformanceCounterConfiguration": [
                    {
                        "counterSpecifier": "\\Processor(_Total)\\% Processor Time",
                        "sampleRate": "PT1M",
                        "unit": "percent"
                    },
                    {
                        "counterSpecifier":"\\Memory\\Available Bytes",
                        "sampleRate":"PT15S",
                        "unit": "Bytes"
                    }
                ]
            },
            "WindowsEventLog": {
                "scheduledTransferPeriod": "PT1M",
                "DataSource": [
                    {
                        "name": "Application!*[System[(Level=1 or Level=2 or Level=3)]]"
                    },
                    {
                        "name": "System!*[System[(Level=1 or Level=2 or Level=3)]]"
                    },
                    {
                        "name": "Security!*[System[(band(Keywords,4503599627370496))]]"
                    }
                ]
            },
            "Logs": {
                "scheduledTransferPeriod": "PT1M",
                "scheduledTransferLogLevelFilter": "Verbose"
            }
        },
        "SinksConfig": {
            "Sink": [
                {
                    "name": "AzureMonitorSink",
                    "AzureMonitor":
                    {
                       
                    }
                }]
        }
    },
    "StorageAccount": "palvmssstr",
    "StorageType": "TableAndBlob"
}
````
Add other Performance counters and metrics to the configuration above, as necessary. Shown in this example are just a few of them. More details on these [here](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/diagnostics-extension-schema-1dot3)

### Protected Settings Configuration
This configuration contains the keys to access the Storage Account. Generate a SAS Key for the Storage Account, from the Azure portal. The '?' preceding the SAS key has to be removed before use. Save this Json document to the working folder on the local Computer(PrivateSettings.json)

````
{
    "storageAccountName": "palvmssstr",
    "storageAccountKey": "<Access Key to the Storage Account>",
    "storageAccountEndPoint": "https://core.windows.net",
    "storageAccountSasToken": "sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-06-29T13:57:55Z&st=2018-12-15T05:57:55Z&spr=https&sig=<generate a SAS Key from the Storage Account>"
}
````


