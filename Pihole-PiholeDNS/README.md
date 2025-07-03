# Pihole DNS Custom Log Ingestion Setup

## Information
This guide only serves as my own interpretation of how the Pihole DNS logs should look in Sentinel. These logs are just DNSMASQ logs in a different location and filename at the
end of the day. 

### Pre requisites
You will need the ability to assign roles in Azure, modify Log analytics workspaces and create DCR's and DCE's.
Following the principle of least privilege you will need a concotion of:
- Log analytics Contributor 
- Monitoring Contributor
- Role Based Access Control Administrator (for assigning roles to the managed identities later)

For the things such as creating watchlists this would require:
- Sentinel Contributor


Or if not following the principle of least privilege, 

1. Enable "Full/Enhanced" logging on the Pihole by doing the following:
`sudo sh -c 'echo "log-queries=extra" > /etc/dnsmasq.d/99-pihole-log-facility.conf'`

2. Restart the pihole FTL service

3. Setup Custom Table in Log Analytics using the following guide and the following templates linked:
https://learn.microsoft.com/en-us/azure/sentinel/connect-custom-logs-ama?tabs=portal

- Run the following powershell and login to the tenant which you wish to create the custom table, you will need Log Analytics Contributor at minimum to do this:
`az login`

- Set a variable which contains the table parameters as found in PiholeDNS_CL.json (See link above on how)
use:
```
$VARIABLENAME = @'
{
    "properties": {
        "schema": {
               "name": "PiholeDNS_CL",
               "columns": [
                    {
                        "name": "TimeGenerated",
                        "type": "DateTime"
                    }, 
                    {
                        "name": "Service",
                        "type": "string"
                    },
                    {
                        "name":"Action",
                        "Type":"string"
                    },
                    {
                        "name": "Domain",
                        "type": "string"
                    },
                    {
                        "name": "StatusOrDirection",
                        "type": "string"
                    },
                    {
                        "name": "Value",
                        "type":"string"
                    },
                    {
                        "name": "Computer",
                        "type":"string"
                    }
              ]
        }
    }
}
'@
```
- Run the following but modify it to suit your Azure environment:
```
Invoke-AzRestMethod -Path "/subscriptions/{subscriptionID}/resourcegroups/{resourcegroup}/providers/microsoft.operationalinsights/workspaces/{Workspace}/tables/{TableName}_CL?api-version=2021-12-01-preview" -Method PUT -payload $VARIABLEFROM ABOVE
```

4. You will need a Data Collection Endpoint to ingest Custom text file logs. Create one of these if you don't have one already.

5. Create Data Collection Rule to collect these logs. You can only have 1 set of custom text logs per DCR so call this something like `<tenant Identifier>-DCR-PiholeFTL`
Ensure you select a Data Collection Endpoint on the creation of the DCR
Target any pihole machines you have.

6. When adding your datasource set the following:

| Setting | Value |
|---------|-------|
| Data Source type | Custom Text Logs |
| File Pattern | /var/log/pihole/pihole.log |
| Table Name | PiholeDNS_CL |
| Record Delimiter | Timestamp |
| Timestamp Format | MMM D hh:mm:ss|

The Transform query can be something like this:

This transformation query will pull all the logs though into the custom table.
```
source | extend d = split(RawData, " ") | project TimeGenerated ,Service=trim(':',tostring(d[3])),Action=tostring(d[4]), Domain = tostring(d[5]), StatusOrDirection=tostring(d[6]),Value = tostring(d[7])
```

7. Once deployed, the logs should appear in your custom table after about 5 minutes.

8. You may want specific information rather than the whole log. For this there are some functions which you can use and deploy which will filter the logs for you nicely at a glance.
See : [PiholeDNS-Functions](https://github.com/0xNekobasu/Pihole-Sentinel-Logging/tree/main/Pihole-PiholeDNS/PiholeDNS-Functions)