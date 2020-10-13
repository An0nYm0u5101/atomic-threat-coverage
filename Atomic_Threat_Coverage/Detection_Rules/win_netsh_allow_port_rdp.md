| Title                    | Netsh RDP Port Opening       |
|:-------------------------|:------------------|
| **Description**          | Detects netsh commands that opens the port 3389 used for RDP, used in Sarwent Malware |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1089: Disabling Security Tools](https://attack.mitre.org/techniques/T1089)</li><li>[T1562.004: Disable or Modify System Firewall](https://attack.mitre.org/techniques/T1562/004)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1562.004: Disable or Modify System Firewall](../Triggers/T1562.004.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Legitimate administration</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://labs.sentinelone.com/sarwent-malware-updates-command-detonation/](https://labs.sentinelone.com/sarwent-malware-updates-command-detonation/)</li></ul>  |
| **Author**               | Sander Wiebing |


## Detection Rules

### Sigma rule

```
title: Netsh RDP Port Opening
id: 01aeb693-138d-49d2-9403-c4f52d7d3d62
description: Detects netsh commands that opens the port 3389 used for RDP, used in Sarwent Malware
references:
    - https://labs.sentinelone.com/sarwent-malware-updates-command-detonation/
date: 2020/05/23
modified: 2020/09/01
tags:
    - attack.defense_evasion
    - attack.t1089          # an old one
    - attack.t1562.004
status: experimental
author: Sander Wiebing
logsource:
    category: process_creation
    product: windows
detection:
    selection1:
        CommandLine|contains|all:
            - netsh
            - firewall add portopening
            - tcp 3389
    selection2:
        CommandLine|contains|all:
            - netsh
            - advfirewall firewall add rule
            - action=allow
            - protocol=TCP
            - localport=3389
    condition: 1 of them
falsepositives:
    - Legitimate administration
level: high

```





### powershell
    
```
Get-WinEvent | where {($_.message -match "CommandLine.*.*netsh.*" -and (($_.message -match "CommandLine.*.*firewall add portopening.*" -and $_.message -match "CommandLine.*.*tcp 3389.*") -or ($_.message -match "CommandLine.*.*advfirewall firewall add rule.*" -and $_.message -match "CommandLine.*.*action=allow.*" -and $_.message -match "CommandLine.*.*protocol=TCP.*" -and $_.message -match "CommandLine.*.*localport=3389.*"))) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.event_data.CommandLine.keyword:*netsh* AND ((winlog.event_data.CommandLine.keyword:*firewall\ add\ portopening* AND winlog.event_data.CommandLine.keyword:*tcp\ 3389*) OR (winlog.event_data.CommandLine.keyword:*advfirewall\ firewall\ add\ rule* AND winlog.event_data.CommandLine.keyword:*action\=allow* AND winlog.event_data.CommandLine.keyword:*protocol\=TCP* AND winlog.event_data.CommandLine.keyword:*localport\=3389*)))
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/01aeb693-138d-49d2-9403-c4f52d7d3d62 <<EOF
{
  "metadata": {
    "title": "Netsh RDP Port Opening",
    "description": "Detects netsh commands that opens the port 3389 used for RDP, used in Sarwent Malware",
    "tags": [
      "attack.defense_evasion",
      "attack.t1089",
      "attack.t1562.004"
    ],
    "query": "(winlog.event_data.CommandLine.keyword:*netsh* AND ((winlog.event_data.CommandLine.keyword:*firewall\\ add\\ portopening* AND winlog.event_data.CommandLine.keyword:*tcp\\ 3389*) OR (winlog.event_data.CommandLine.keyword:*advfirewall\\ firewall\\ add\\ rule* AND winlog.event_data.CommandLine.keyword:*action\\=allow* AND winlog.event_data.CommandLine.keyword:*protocol\\=TCP* AND winlog.event_data.CommandLine.keyword:*localport\\=3389*)))"
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
                    "query": "(winlog.event_data.CommandLine.keyword:*netsh* AND ((winlog.event_data.CommandLine.keyword:*firewall\\ add\\ portopening* AND winlog.event_data.CommandLine.keyword:*tcp\\ 3389*) OR (winlog.event_data.CommandLine.keyword:*advfirewall\\ firewall\\ add\\ rule* AND winlog.event_data.CommandLine.keyword:*action\\=allow* AND winlog.event_data.CommandLine.keyword:*protocol\\=TCP* AND winlog.event_data.CommandLine.keyword:*localport\\=3389*)))",
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
      "throttle_period": "15m",
      "email": {
        "profile": "standard",
        "from": "root@localhost",
        "to": "root@localhost",
        "subject": "Sigma Rule 'Netsh RDP Port Opening'",
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
(CommandLine.keyword:*netsh* AND ((CommandLine.keyword:*firewall add portopening* AND CommandLine.keyword:*tcp 3389*) OR (CommandLine.keyword:*advfirewall firewall add rule* AND CommandLine.keyword:*action=allow* AND CommandLine.keyword:*protocol=TCP* AND CommandLine.keyword:*localport=3389*)))
```


### splunk
    
```
(CommandLine="*netsh*" ((CommandLine="*firewall add portopening*" CommandLine="*tcp 3389*") OR (CommandLine="*advfirewall firewall add rule*" CommandLine="*action=allow*" CommandLine="*protocol=TCP*" CommandLine="*localport=3389*")))
```


### logpoint
    
```
(CommandLine="*netsh*" ((CommandLine="*firewall add portopening*" CommandLine="*tcp 3389*") OR (CommandLine="*advfirewall firewall add rule*" CommandLine="*action=allow*" CommandLine="*protocol=TCP*" CommandLine="*localport=3389*")))
```


### grep
    
```
grep -P '^(?:.*(?=.*.*netsh.*)(?=.*(?:.*(?:.*(?:.*(?=.*.*firewall add portopening.*)(?=.*.*tcp 3389.*))|.*(?:.*(?=.*.*advfirewall firewall add rule.*)(?=.*.*action=allow.*)(?=.*.*protocol=TCP.*)(?=.*.*localport=3389.*))))))'
```



