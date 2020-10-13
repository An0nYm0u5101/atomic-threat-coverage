| Title                    | Suspicious Debugger Registration Cmdline       |
|:-------------------------|:------------------|
| **Description**          | Detects the registration of a debugger for a program that is available in the logon screen (sticky key backdoor). |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0003: Persistence](https://attack.mitre.org/tactics/TA0003)</li><li>[TA0004: Privilege Escalation](https://attack.mitre.org/tactics/TA0004)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1546.008: Accessibility Features](https://attack.mitre.org/techniques/T1546/008)</li><li>[T1015: Accessibility Features](https://attack.mitre.org/techniques/T1015)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1546.008: Accessibility Features](../Triggers/T1546.008.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Penetration Tests</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://blogs.technet.microsoft.com/jonathantrull/2016/10/03/detecting-sticky-key-backdoors/](https://blogs.technet.microsoft.com/jonathantrull/2016/10/03/detecting-sticky-key-backdoors/)</li></ul>  |
| **Author**               | Florian Roth |


## Detection Rules

### Sigma rule

```
title: Suspicious Debugger Registration Cmdline
id: ae215552-081e-44c7-805f-be16f975c8a2
status: experimental
description: Detects the registration of a debugger for a program that is available in the logon screen (sticky key backdoor).
references:
    - https://blogs.technet.microsoft.com/jonathantrull/2016/10/03/detecting-sticky-key-backdoors/
tags:
    - attack.persistence
    - attack.privilege_escalation
    - attack.t1546.008
    - attack.t1015  # an old one
author: Florian Roth
date: 2019/09/06
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        CommandLine:
            - '*\CurrentVersion\Image File Execution Options\sethc.exe*'
            - '*\CurrentVersion\Image File Execution Options\utilman.exe*'
            - '*\CurrentVersion\Image File Execution Options\osk.exe*'
            - '*\CurrentVersion\Image File Execution Options\magnify.exe*'
            - '*\CurrentVersion\Image File Execution Options\narrator.exe*'
            - '*\CurrentVersion\Image File Execution Options\displayswitch.exe*'
            - '*\CurrentVersion\Image File Execution Options\atbroker.exe*'
    condition: selection
falsepositives:
    - Penetration Tests
level: high


```





### powershell
    
```
Get-WinEvent | where {($_.message -match "CommandLine.*.*\\CurrentVersion\\Image File Execution Options\\sethc.exe.*" -or $_.message -match "CommandLine.*.*\\CurrentVersion\\Image File Execution Options\\utilman.exe.*" -or $_.message -match "CommandLine.*.*\\CurrentVersion\\Image File Execution Options\\osk.exe.*" -or $_.message -match "CommandLine.*.*\\CurrentVersion\\Image File Execution Options\\magnify.exe.*" -or $_.message -match "CommandLine.*.*\\CurrentVersion\\Image File Execution Options\\narrator.exe.*" -or $_.message -match "CommandLine.*.*\\CurrentVersion\\Image File Execution Options\\displayswitch.exe.*" -or $_.message -match "CommandLine.*.*\\CurrentVersion\\Image File Execution Options\\atbroker.exe.*") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
winlog.event_data.CommandLine.keyword:(*\\CurrentVersion\\Image\ File\ Execution\ Options\\sethc.exe* OR *\\CurrentVersion\\Image\ File\ Execution\ Options\\utilman.exe* OR *\\CurrentVersion\\Image\ File\ Execution\ Options\\osk.exe* OR *\\CurrentVersion\\Image\ File\ Execution\ Options\\magnify.exe* OR *\\CurrentVersion\\Image\ File\ Execution\ Options\\narrator.exe* OR *\\CurrentVersion\\Image\ File\ Execution\ Options\\displayswitch.exe* OR *\\CurrentVersion\\Image\ File\ Execution\ Options\\atbroker.exe*)
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/ae215552-081e-44c7-805f-be16f975c8a2 <<EOF
{
  "metadata": {
    "title": "Suspicious Debugger Registration Cmdline",
    "description": "Detects the registration of a debugger for a program that is available in the logon screen (sticky key backdoor).",
    "tags": [
      "attack.persistence",
      "attack.privilege_escalation",
      "attack.t1546.008",
      "attack.t1015"
    ],
    "query": "winlog.event_data.CommandLine.keyword:(*\\\\CurrentVersion\\\\Image\\ File\\ Execution\\ Options\\\\sethc.exe* OR *\\\\CurrentVersion\\\\Image\\ File\\ Execution\\ Options\\\\utilman.exe* OR *\\\\CurrentVersion\\\\Image\\ File\\ Execution\\ Options\\\\osk.exe* OR *\\\\CurrentVersion\\\\Image\\ File\\ Execution\\ Options\\\\magnify.exe* OR *\\\\CurrentVersion\\\\Image\\ File\\ Execution\\ Options\\\\narrator.exe* OR *\\\\CurrentVersion\\\\Image\\ File\\ Execution\\ Options\\\\displayswitch.exe* OR *\\\\CurrentVersion\\\\Image\\ File\\ Execution\\ Options\\\\atbroker.exe*)"
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
                    "query": "winlog.event_data.CommandLine.keyword:(*\\\\CurrentVersion\\\\Image\\ File\\ Execution\\ Options\\\\sethc.exe* OR *\\\\CurrentVersion\\\\Image\\ File\\ Execution\\ Options\\\\utilman.exe* OR *\\\\CurrentVersion\\\\Image\\ File\\ Execution\\ Options\\\\osk.exe* OR *\\\\CurrentVersion\\\\Image\\ File\\ Execution\\ Options\\\\magnify.exe* OR *\\\\CurrentVersion\\\\Image\\ File\\ Execution\\ Options\\\\narrator.exe* OR *\\\\CurrentVersion\\\\Image\\ File\\ Execution\\ Options\\\\displayswitch.exe* OR *\\\\CurrentVersion\\\\Image\\ File\\ Execution\\ Options\\\\atbroker.exe*)",
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
        "subject": "Sigma Rule 'Suspicious Debugger Registration Cmdline'",
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
CommandLine.keyword:(*\\CurrentVersion\\Image File Execution Options\\sethc.exe* *\\CurrentVersion\\Image File Execution Options\\utilman.exe* *\\CurrentVersion\\Image File Execution Options\\osk.exe* *\\CurrentVersion\\Image File Execution Options\\magnify.exe* *\\CurrentVersion\\Image File Execution Options\\narrator.exe* *\\CurrentVersion\\Image File Execution Options\\displayswitch.exe* *\\CurrentVersion\\Image File Execution Options\\atbroker.exe*)
```


### splunk
    
```
(CommandLine="*\\CurrentVersion\\Image File Execution Options\\sethc.exe*" OR CommandLine="*\\CurrentVersion\\Image File Execution Options\\utilman.exe*" OR CommandLine="*\\CurrentVersion\\Image File Execution Options\\osk.exe*" OR CommandLine="*\\CurrentVersion\\Image File Execution Options\\magnify.exe*" OR CommandLine="*\\CurrentVersion\\Image File Execution Options\\narrator.exe*" OR CommandLine="*\\CurrentVersion\\Image File Execution Options\\displayswitch.exe*" OR CommandLine="*\\CurrentVersion\\Image File Execution Options\\atbroker.exe*")
```


### logpoint
    
```
CommandLine IN ["*\\CurrentVersion\\Image File Execution Options\\sethc.exe*", "*\\CurrentVersion\\Image File Execution Options\\utilman.exe*", "*\\CurrentVersion\\Image File Execution Options\\osk.exe*", "*\\CurrentVersion\\Image File Execution Options\\magnify.exe*", "*\\CurrentVersion\\Image File Execution Options\\narrator.exe*", "*\\CurrentVersion\\Image File Execution Options\\displayswitch.exe*", "*\\CurrentVersion\\Image File Execution Options\\atbroker.exe*"]
```


### grep
    
```
grep -P '^(?:.*.*\CurrentVersion\Image File Execution Options\sethc\.exe.*|.*.*\CurrentVersion\Image File Execution Options\utilman\.exe.*|.*.*\CurrentVersion\Image File Execution Options\osk\.exe.*|.*.*\CurrentVersion\Image File Execution Options\magnify\.exe.*|.*.*\CurrentVersion\Image File Execution Options\narrator\.exe.*|.*.*\CurrentVersion\Image File Execution Options\displayswitch\.exe.*|.*.*\CurrentVersion\Image File Execution Options\atbroker\.exe.*)'
```



