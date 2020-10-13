| Title                    | Firewall Disabled via Netsh       |
|:-------------------------|:------------------|
| **Description**          | Detects netsh commands that turns off the Windows firewall |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1562.004: Disable or Modify System Firewall](https://attack.mitre.org/techniques/T1562/004)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1562.004: Disable or Modify System Firewall](../Triggers/T1562.004.md)</li></ul>  |
| **Severity Level**       | medium |
| **False Positives**      | <ul><li>Legitimate administration</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://www.winhelponline.com/blog/enable-and-disable-windows-firewall-quickly-using-command-line/](https://www.winhelponline.com/blog/enable-and-disable-windows-firewall-quickly-using-command-line/)</li><li>[https://app.any.run/tasks/210244b9-0b6b-4a2c-83a3-04bd3175d017/](https://app.any.run/tasks/210244b9-0b6b-4a2c-83a3-04bd3175d017/)</li></ul>  |
| **Author**               | Fatih Sirin |
| Other Tags           | <ul><li>attack.s0108</li></ul> | 

## Detection Rules

### Sigma rule

```
title: Firewall Disabled via Netsh
id: 57c4bf16-227f-4394-8ec7-1b745ee061c3
description: Detects netsh commands that turns off the Windows firewall
references:
    - https://www.winhelponline.com/blog/enable-and-disable-windows-firewall-quickly-using-command-line/
    - https://app.any.run/tasks/210244b9-0b6b-4a2c-83a3-04bd3175d017/
date: 2019/11/01
modified: 2020/08/30
status: experimental
author: Fatih Sirin
tags:
    - attack.defense_evasion
    - attack.t1562.004
    - attack.s0108
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        CommandLine:
            - netsh firewall set opmode mode=disable
            - netsh advfirewall set * state off
    condition: selection
falsepositives:
    - Legitimate administration
level: medium

```





### powershell
    
```
Get-WinEvent | where {($_.message -match "netsh firewall set opmode mode=disable" -or $_.message -match "CommandLine.*netsh advfirewall set .* state off") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
winlog.event_data.CommandLine.keyword:(netsh\ firewall\ set\ opmode\ mode\=disable OR netsh\ advfirewall\ set\ *\ state\ off)
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/57c4bf16-227f-4394-8ec7-1b745ee061c3 <<EOF
{
  "metadata": {
    "title": "Firewall Disabled via Netsh",
    "description": "Detects netsh commands that turns off the Windows firewall",
    "tags": [
      "attack.defense_evasion",
      "attack.t1562.004",
      "attack.s0108"
    ],
    "query": "winlog.event_data.CommandLine.keyword:(netsh\\ firewall\\ set\\ opmode\\ mode\\=disable OR netsh\\ advfirewall\\ set\\ *\\ state\\ off)"
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
                    "query": "winlog.event_data.CommandLine.keyword:(netsh\\ firewall\\ set\\ opmode\\ mode\\=disable OR netsh\\ advfirewall\\ set\\ *\\ state\\ off)",
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
        "subject": "Sigma Rule 'Firewall Disabled via Netsh'",
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
CommandLine.keyword:(netsh firewall set opmode mode=disable netsh advfirewall set * state off)
```


### splunk
    
```
(CommandLine="netsh firewall set opmode mode=disable" OR CommandLine="netsh advfirewall set * state off")
```


### logpoint
    
```
CommandLine IN ["netsh firewall set opmode mode=disable", "netsh advfirewall set * state off"]
```


### grep
    
```
grep -P '^(?:.*netsh firewall set opmode mode=disable|.*netsh advfirewall set .* state off)'
```



