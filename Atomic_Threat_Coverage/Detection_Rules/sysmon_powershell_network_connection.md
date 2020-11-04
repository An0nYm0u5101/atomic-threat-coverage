| Title                    | PowerShell Network Connections       |
|:-------------------------|:------------------|
| **Description**          | Detects a Powershell process that opens network connections - check for suspicious target ports and target systems - adjust to your environment (e.g. extend filters with company's ip range') |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1086: PowerShell](https://attack.mitre.org/techniques/T1086)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0007_3_windows_sysmon_network_connection](../Data_Needed/DN_0007_3_windows_sysmon_network_connection.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1086: PowerShell](../Triggers/T1086.md)</li></ul>  |
| **Severity Level**       | low |
| **False Positives**      | <ul><li>Administrative scripts</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://www.youtube.com/watch?v=DLtJTxMWZ2o](https://www.youtube.com/watch?v=DLtJTxMWZ2o)</li></ul>  |
| **Author**               | Florian Roth |


## Detection Rules

### Sigma rule

```
title: PowerShell Network Connections
id: 1f21ec3f-810d-4b0e-8045-322202e22b4b
status: experimental
description: Detects a Powershell process that opens network connections - check for suspicious target ports and target systems - adjust to your environment (e.g.
    extend filters with company's ip range')
author: Florian Roth
date: 2017/03/13
references:
    - https://www.youtube.com/watch?v=DLtJTxMWZ2o
tags:
    - attack.execution
    - attack.t1086
logsource:
    product: windows
    service: sysmon
detection:
    selection:
        EventID: 3
        Image: '*\powershell.exe'
        Initiated: 'true'
    filter:
        DestinationIp:
            - '10.*'
            - '192.168.*'
            - '172.16.*'
            - '172.17.*'
            - '172.18.*'
            - '172.19.*'
            - '172.20.*'
            - '172.21.*'
            - '172.22.*'
            - '172.23.*'
            - '172.24.*'
            - '172.25.*'
            - '172.26.*'
            - '172.27.*'
            - '172.28.*'
            - '172.29.*'
            - '172.30.*'
            - '172.31.*'
            - '127.0.0.1'
        DestinationIsIpv6: 'false'
        User: 'NT AUTHORITY\SYSTEM'
    condition: selection and not filter
falsepositives:
    - Administrative scripts
level: low

```





### powershell
    
```
Get-WinEvent -LogName Microsoft-Windows-Sysmon/Operational | where {(($_.ID -eq "3" -and $_.message -match "Image.*.*\\powershell.exe" -and $_.message -match "Initiated.*true") -and  -not (($_.message -match "DestinationIp.*10..*" -or $_.message -match "DestinationIp.*192.168..*" -or $_.message -match "DestinationIp.*172.16..*" -or $_.message -match "DestinationIp.*172.17..*" -or $_.message -match "DestinationIp.*172.18..*" -or $_.message -match "DestinationIp.*172.19..*" -or $_.message -match "DestinationIp.*172.20..*" -or $_.message -match "DestinationIp.*172.21..*" -or $_.message -match "DestinationIp.*172.22..*" -or $_.message -match "DestinationIp.*172.23..*" -or $_.message -match "DestinationIp.*172.24..*" -or $_.message -match "DestinationIp.*172.25..*" -or $_.message -match "DestinationIp.*172.26..*" -or $_.message -match "DestinationIp.*172.27..*" -or $_.message -match "DestinationIp.*172.28..*" -or $_.message -match "DestinationIp.*172.29..*" -or $_.message -match "DestinationIp.*172.30..*" -or $_.message -match "DestinationIp.*172.31..*" -or $_.message -match "127.0.0.1") -and $_.message -match "DestinationIsIpv6.*false" -and $_.message -match "User.*NT AUTHORITY\\SYSTEM")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.channel:"Microsoft\-Windows\-Sysmon\/Operational" AND (winlog.event_id:"3" AND winlog.event_data.Image.keyword:*\\powershell.exe AND Initiated:"true") AND (NOT (winlog.event_data.DestinationIp.keyword:(10.* OR 192.168.* OR 172.16.* OR 172.17.* OR 172.18.* OR 172.19.* OR 172.20.* OR 172.21.* OR 172.22.* OR 172.23.* OR 172.24.* OR 172.25.* OR 172.26.* OR 172.27.* OR 172.28.* OR 172.29.* OR 172.30.* OR 172.31.* OR 127.0.0.1) AND winlog.event_data.DestinationIsIpv6:"false" AND winlog.event_data.User:"NT\ AUTHORITY\\SYSTEM")))
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/1f21ec3f-810d-4b0e-8045-322202e22b4b <<EOF
{
  "metadata": {
    "title": "PowerShell Network Connections",
    "description": "Detects a Powershell process that opens network connections - check for suspicious target ports and target systems - adjust to your environment (e.g. extend filters with company's ip range')",
    "tags": [
      "attack.execution",
      "attack.t1086"
    ],
    "query": "(winlog.channel:\"Microsoft\\-Windows\\-Sysmon\\/Operational\" AND (winlog.event_id:\"3\" AND winlog.event_data.Image.keyword:*\\\\powershell.exe AND Initiated:\"true\") AND (NOT (winlog.event_data.DestinationIp.keyword:(10.* OR 192.168.* OR 172.16.* OR 172.17.* OR 172.18.* OR 172.19.* OR 172.20.* OR 172.21.* OR 172.22.* OR 172.23.* OR 172.24.* OR 172.25.* OR 172.26.* OR 172.27.* OR 172.28.* OR 172.29.* OR 172.30.* OR 172.31.* OR 127.0.0.1) AND winlog.event_data.DestinationIsIpv6:\"false\" AND winlog.event_data.User:\"NT\\ AUTHORITY\\\\SYSTEM\")))"
  },
  "trigger": {
    "schedule": {
      "interval": "30m"
    }
  },
  "input": {
    "search": {
      "request": {
        "body": {
          "size": 0,
          "query": {
            "bool": {
              "must": [
                {
                  "query_string": {
                    "query": "(winlog.channel:\"Microsoft\\-Windows\\-Sysmon\\/Operational\" AND (winlog.event_id:\"3\" AND winlog.event_data.Image.keyword:*\\\\powershell.exe AND Initiated:\"true\") AND (NOT (winlog.event_data.DestinationIp.keyword:(10.* OR 192.168.* OR 172.16.* OR 172.17.* OR 172.18.* OR 172.19.* OR 172.20.* OR 172.21.* OR 172.22.* OR 172.23.* OR 172.24.* OR 172.25.* OR 172.26.* OR 172.27.* OR 172.28.* OR 172.29.* OR 172.30.* OR 172.31.* OR 127.0.0.1) AND winlog.event_data.DestinationIsIpv6:\"false\" AND winlog.event_data.User:\"NT\\ AUTHORITY\\\\SYSTEM\")))",
                    "analyze_wildcard": true
                  }
                }
              ],
              "filter": {
                "range": {
                  "timestamp": {
                    "gte": "now-30m/m"
                  }
                }
              }
            }
          }
        },
        "indices": [
          "winlogbeat-*"
        ]
      }
    }
  },
  "condition": {
    "compare": {
      "ctx.payload.hits.total": {
        "not_eq": 0
      }
    }
  },
  "actions": {
    "send_email": {
      "email": {
        "to": "root@localhost",
        "subject": "Sigma Rule 'PowerShell Network Connections'",
        "body": "Hits:\n{{#ctx.payload.hits.hits}}{{_source}}\n================================================================================\n{{/ctx.payload.hits.hits}}",
        "attachments": {
          "data.json": {
            "data": {
              "format": "json"
            }
          }
        }
      }
    }
  }
}
EOF

```


### graylog
    
```
((EventID:"3" AND Image.keyword:*\\powershell.exe AND Initiated:"true") AND (NOT (DestinationIp.keyword:(10.* 192.168.* 172.16.* 172.17.* 172.18.* 172.19.* 172.20.* 172.21.* 172.22.* 172.23.* 172.24.* 172.25.* 172.26.* 172.27.* 172.28.* 172.29.* 172.30.* 172.31.* 127.0.0.1) AND DestinationIsIpv6:"false" AND User:"NT AUTHORITY\\SYSTEM")))
```


### splunk
    
```
(source="WinEventLog:Microsoft-Windows-Sysmon/Operational" (EventCode="3" Image="*\\powershell.exe" Initiated="true") NOT ((DestinationIp="10.*" OR DestinationIp="192.168.*" OR DestinationIp="172.16.*" OR DestinationIp="172.17.*" OR DestinationIp="172.18.*" OR DestinationIp="172.19.*" OR DestinationIp="172.20.*" OR DestinationIp="172.21.*" OR DestinationIp="172.22.*" OR DestinationIp="172.23.*" OR DestinationIp="172.24.*" OR DestinationIp="172.25.*" OR DestinationIp="172.26.*" OR DestinationIp="172.27.*" OR DestinationIp="172.28.*" OR DestinationIp="172.29.*" OR DestinationIp="172.30.*" OR DestinationIp="172.31.*" OR DestinationIp="127.0.0.1") DestinationIsIpv6="false" User="NT AUTHORITY\\SYSTEM"))
```


### logpoint
    
```
((event_id="3" Image="*\\powershell.exe" Initiated="true")  -(DestinationIp IN ["10.*", "192.168.*", "172.16.*", "172.17.*", "172.18.*", "172.19.*", "172.20.*", "172.21.*", "172.22.*", "172.23.*", "172.24.*", "172.25.*", "172.26.*", "172.27.*", "172.28.*", "172.29.*", "172.30.*", "172.31.*", "127.0.0.1"] DestinationIsIpv6="false" User="NT AUTHORITY\\SYSTEM"))
```


### grep
    
```
grep -P '^(?:.*(?=.*(?:.*(?=.*3)(?=.*.*\powershell\.exe)(?=.*true)))(?=.*(?!.*(?:.*(?=.*(?:.*10\..*|.*192\.168\..*|.*172\.16\..*|.*172\.17\..*|.*172\.18\..*|.*172\.19\..*|.*172\.20\..*|.*172\.21\..*|.*172\.22\..*|.*172\.23\..*|.*172\.24\..*|.*172\.25\..*|.*172\.26\..*|.*172\.27\..*|.*172\.28\..*|.*172\.29\..*|.*172\.30\..*|.*172\.31\..*|.*127\.0\.0\.1))(?=.*false)(?=.*NT AUTHORITY\SYSTEM)))))'
```



