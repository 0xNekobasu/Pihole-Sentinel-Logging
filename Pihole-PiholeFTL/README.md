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

3. Setup Custom Table in Log Analytics using the following guide and the following templates linked:
https://learn.microsoft.com/en-us/azure/sentinel/connect-custom-logs-ama?tabs=portal

- Run the following powershell and login to the tenant which you wish to create the custom table, you will need Log Analytics Contributor at minimum to do this:
`az login`

- Set a variable which contains the table parameters as found in PiholeFTL_CL.json (See link above on how)
use:
```
$VARIABLENAME = @'
{
    "properties": {
        "schema": {
               "name": "PiholeFTL_CL",
               "columns": [
                    {
                        "name": "TimeGenerated",
                        "type": "DateTime"
                    }, 
                    {
                        "name": "ProcessAndThreadID",
                        "type": "string"
                    },
                    {
                        "name":"LogLevel",
                        "Type":"string"
                    },
                    {
                        "name": "Message",
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
| File Pattern | /var/log/pihole/FTL.log |
| Table Name | PiholeFTL_CL |
| Record Delimiter | Timestamp |
| Timestamp Format | YYYY-MM-DD HH:MM:SS |

This transformation query will pull all the logs though into the custom table and sort them into the columns you setup earlier.
```
source | project TimeGenerated, ProcessAndThreadID = extract(@'\[(.*?)\]', 1, RawData), LogLevel = extract(@'\] (\w+):', 1, RawData) , Message = extract(@'\w+: (.*)', 1, RawData)
```

7. Once deployed, the logs should appear in your custom table after about 5 minutes.
