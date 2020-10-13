| Title                    | Koadic Execution       |
|:-------------------------|:------------------|
| **Description**          | Detects command line parameters used by Koadic hack tool |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1059.003: Windows Command Shell](https://attack.mitre.org/techniques/T1059/003)</li><li>[T1059: Command and Scripting Interpreter](https://attack.mitre.org/techniques/T1059)</li><li>[T1059.005: Visual Basic](https://attack.mitre.org/techniques/T1059/005)</li><li>[T1059.007: JavaScript/JScript](https://attack.mitre.org/techniques/T1059/007)</li><li>[T1064: Scripting](https://attack.mitre.org/techniques/T1064)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1059.003: Windows Command Shell](../Triggers/T1059.003.md)</li><li>[T1059.005: Visual Basic](../Triggers/T1059.005.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Pentest</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://unit42.paloaltonetworks.com/unit42-sofacy-groups-parallel-attacks/](https://unit42.paloaltonetworks.com/unit42-sofacy-groups-parallel-attacks/)</li><li>[https://github.com/zerosum0x0/koadic/blob/master/data/stager/js/stdlib.js#L955](https://github.com/zerosum0x0/koadic/blob/master/data/stager/js/stdlib.js#L955)</li><li>[https://blog.f-secure.com/hunting-for-koadic-a-com-based-rootkit/](https://blog.f-secure.com/hunting-for-koadic-a-com-based-rootkit/)</li></ul>  |
| **Author**               | wagga |


## Detection Rules

### Sigma rule

```
title: Koadic Execution
id: 5cddf373-ef00-4112-ad72-960ac29bac34
status: experimental
description: Detects command line parameters used by Koadic hack tool
references:
    - https://unit42.paloaltonetworks.com/unit42-sofacy-groups-parallel-attacks/
    - https://github.com/zerosum0x0/koadic/blob/master/data/stager/js/stdlib.js#L955
    - https://blog.f-secure.com/hunting-for-koadic-a-com-based-rootkit/
tags:
    - attack.execution
    - attack.t1059.003
    - attack.t1059  # an old one
    - attack.t1059.005
    - attack.t1059.007
    - attack.t1064  # an old one
date: 2020/01/12
modified: 2020/09/01
author: wagga
logsource:
    category: process_creation
    product: windows
detection:
    selection1:
        CommandLine:
            - '*cmd.exe* /q /c chcp *'
    condition: selection1
fields:
    - CommandLine
    - ParentCommandLine
falsepositives:
    - Pentest
level: high

```





### powershell
    
```
Get-WinEvent | where {($_.message -match "CommandLine.*.*cmd.exe.* /q /c chcp .*") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
winlog.event_data.CommandLine.keyword:(*cmd.exe*\ \/q\ \/c\ chcp\ *)
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/5cddf373-ef00-4112-ad72-960ac29bac34 <<EOF
{
  "metadata": {
    "title": "Koadic Execution",
    "description": "Detects command line parameters used by Koadic hack tool",
    "tags": [
      "attack.execution",
      "attack.t1059.003",
      "attack.t1059",
      "attack.t1059.005",
      "attack.t1059.007",
      "attack.t1064"
    ],
    "query": "winlog.event_data.CommandLine.keyword:(*cmd.exe*\\ \\/q\\ \\/c\\ chcp\\ *)"
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
                    "query": "winlog.event_data.CommandLine.keyword:(*cmd.exe*\\ \\/q\\ \\/c\\ chcp\\ *)",
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
        "subject": "Sigma Rule 'Koadic Execution'",
        "body": "Hits:\n{{#ctx.payload.hits.hits}}Hit on {{_source.@timestamp}}:\n      CommandLine = {{_source.CommandLine}}\nParentCommandLine = {{_source.ParentCommandLine}}================================================================================\n{{/ctx.payload.hits.hits}}",
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
CommandLine.keyword:(*cmd.exe* \/q \/c chcp *)
```


### splunk
    
```
(CommandLine="*cmd.exe* /q /c chcp *") | table CommandLine,ParentCommandLine
```


### logpoint
    
```
CommandLine IN ["*cmd.exe* /q /c chcp *"]
```


### grep
    
```
grep -P '^(?:.*.*cmd\.exe.* /q /c chcp .*)'
```



