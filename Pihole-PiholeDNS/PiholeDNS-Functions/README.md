# Useful Functions
I found that I would like to enrich the dataset from the pihole logs to make it easier to understand. 
One thing I wanted to do was enrich the data with things such as hostnames if they were on a static IP. Things such as servers etc. 
I also dislike what some other functions do which is require you to edit the function directly to update your hosts etc. I prefer to use watchlists in Sentinel. 
The use of the watchlists allows for easier management in my eyes. You can also update them using logic apps.

# Overview of functions

## PiholeDNS_DNSRequests
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


