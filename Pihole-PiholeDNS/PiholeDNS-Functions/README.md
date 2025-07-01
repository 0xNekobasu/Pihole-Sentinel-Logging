# Useful Functions
I found that I would like to enrich the dataset from the pihole logs to make it easier to understand. 
One thing I wanted to do was enrich the data with things such as hostnames if they were on a static IP. Things such as servers etc. 
I also dislike what some other functions do which is require you to edit the function directly to update your hosts etc. I prefer to use watchlists in Sentinel. 
The use of the watchlists allows for easier management in my eyes. You can also update them using logic apps.

## Watchlist Setup 
1. To setup the `Pihole-KnownHosts` watchlist take the `Pihole-KnownHosts.csv` file and save this to your machine.
2. Go to the Sentinel instance which is sitting ontop of your Log Analytics Workspace and under `Configuration` select `Watchlists`
3. Create a new Watchlist with the following:

| Field | Value |
| ----- | ----- |
| Name  | Pihole-KnownHosts | 
| Description | <whatever you want> |
| Alias | Pihole-KnownHosts |

4. Move to the next page

5. Select the following settings

| Field               |  Value                 |
| ------------------- |------------------------|
| Source Type         |             Local file | 
| File Type           | CSV file with a header |
| Number of lines before row with headings | 0 |

6. Upload the `Pihole-KnownHosts.csv`

7. Select the SearchKey to be `HostIP`

8. Review and create. You can modify and add your own rows to the watchlist by going back to the watchlist menu in sentinel.

## Overview of functions

### PiholeDNS_DNSRequests
This function simply just gathers all the queries that have been made to the Pihole, it does not show any responses or if the query was cached etc. Simply X Host made this request.

Initalisation of the watchlist which contains your known hosts
```
let Knownhosts = _GetWatchlist("Pihole-KnownHosts")|project HostIP=SearchKey, HostName, HostDescription;
PiholeDNS_CL
| where Action has "query"
```
You typically wouldn't need to have a "Replace" here to remove any hidden characters but i found that in my case the Value column contained hidden characters. This caused an issue so I filtered them out so the join statement would work correctly
```
|extend vIP= replace(@'\r|\n|\t', '', tostring(Value))
```
Join to the PiholeDNS_CL data. 
```
|join kind=leftouter (Knownhosts) on $left.vIP == $right.HostIP`
|project-away Value`
|project TimeGenerated, Service, Action,Domain,StatusOrDirection,HostIP=vIP, HostName,HostDescription`
```


