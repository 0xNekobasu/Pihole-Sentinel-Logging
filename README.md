# Pihole-Sentinel-Logging
A guide on how to configure the Azure monitor agent to pull Pihole DNS logs from text file.

This in a sense has done the "hard" part of defining the JSON schema's for the custom logging but this is just essentially following Microsofts
own guide on how to ingest custom text logs via the Azure Monitor Agent. 
https://learn.microsoft.com/en-us/azure/azure-monitor/vm/data-collection-log-text#delimited-log-files

This also serves as beginner friendly guide to hopefully make the use of the Azure Monitor agent easier for those who are just starting to use it for ingesting custom log sources.

## Contents
This repo contains the following:

- Guide to setup Pihole DNS logs, Pihole FTL logs and the Webserver logs to Log Analytics

- Useful functions for analysing some of the Pihole Logs including a watchlist which enriches the data you ingest to make the logs slightly more useful. 


## Setup PiholeDNS Logs to Sentinel:
Link: [PiholeDNS_CL Setup Guide](https://github.com/0xNekobasu/Pihole-Sentinel-Logging/tree/main/Pihole-PiholeDNS)


## Setup PiholeFTL Logs to Sentinel:
Link: [PiholeFTL_CL Setup Guide](https://github.com/0xNekobasu/Pihole-Sentinel-Logging/tree/main/Pihole-PiholeFTL)
