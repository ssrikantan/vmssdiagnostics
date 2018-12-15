# vmss diagnostics extension
describes how azure diagnostics can be enabled on a Virtual Machine Scale Set. The first part covers the steps to enable this for a VMSS running Windows VMs. The steps to enable these for a VMSS running Linux VMs follow it.

## Using Azure CLI - Windows VM based Scale Sets
These steps can be performed on an existing Virtual Machine Scale Set running Windows VMs. In the example here, a VMSS with the following properties pre-exist in Azure before enabling the Diagnostics extension.

VMSS Name - palvmss
VMSS Resource Group Name: palvmssrg

### Create a Storage Account to store the Diagnostics from the Scale Set
A Storage Account with the name palvmssstr is created in Azure

### Create the Protected and Public Configuration Settings for the Diagnostics Extension

#### Public Settings Configuration

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
