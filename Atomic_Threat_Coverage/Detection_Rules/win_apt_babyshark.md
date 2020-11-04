| Title                    | Baby Shark Activity       |
|:-------------------------|:------------------|
| **Description**          | Detects activity that could be related to Baby Shark malware |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li><li>[TA0007: Discovery](https://attack.mitre.org/tactics/TA0007)</li><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1059: Command and Scripting Interpreter](https://attack.mitre.org/techniques/T1059)</li><li>[T1086: PowerShell](https://attack.mitre.org/techniques/T1086)</li><li>[T1012: Query Registry](https://attack.mitre.org/techniques/T1012)</li><li>[T1170: Mshta](https://attack.mitre.org/techniques/T1170)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1059: Command and Scripting Interpreter](../Triggers/T1059.md)</li><li>[T1086: PowerShell](../Triggers/T1086.md)</li><li>[T1012: Query Registry](../Triggers/T1012.md)</li><li>[T1170: Mshta](../Triggers/T1170.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://unit42.paloaltonetworks.com/new-babyshark-malware-targets-u-s-national-security-think-tanks/](https://unit42.paloaltonetworks.com/new-babyshark-malware-targets-u-s-national-security-think-tanks/)</li></ul>  |
| **Author**               | Florian Roth |


## Detection Rules

### Sigma rule

```
title: Baby Shark Activity
id: 2b30fa36-3a18-402f-a22d-bf4ce2189f35
status: experimental
description: Detects activity that could be related to Baby Shark malware
references:
    - https://unit42.paloaltonetworks.com/new-babyshark-malware-targets-u-s-national-security-think-tanks/
tags:
    - attack.execution
    - attack.t1059
    - attack.t1086
    - attack.discovery
    - attack.t1012
    - attack.defense_evasion
    - attack.t1170
logsource:
    category: process_creation
    product: windows
author: Florian Roth
date: 2019/02/24
detection:
    selection:
        CommandLine:
            - reg query "HKEY_CURRENT_USER\Software\Microsoft\Terminal Server Client\Default"
            - powershell.exe mshta.exe http*
            - cmd.exe /c taskkill /im cmd.exe
    condition: selection
falsepositives:
    - unknown
level: high

```





### powershell
    
```
Get-WinEvent | where {($_.message -match "reg query \"HKEY_CURRENT_USER\\Software\\Microsoft\\Terminal Server Client\\Default\"" -or $_.message -match "CommandLine.*powershell.exe mshta.exe http.*" -or $_.message -match "cmd.exe /c taskkill /im cmd.exe") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
winlog.event_data.CommandLine.keyword:(reg\ query\ \"HKEY_CURRENT_USER\\Software\\Microsoft\\Terminal\ Server\ Client\\Default\" OR powershell.exe\ mshta.exe\ http* OR cmd.exe\ \/c\ taskkill\ \/im\ cmd.exe)
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/2b30fa36-3a18-402f-a22d-bf4ce2189f35 <<EOF
{
  "metadata": {
    "title": "Baby Shark Activity",
    "description": "Detects activity that could be related to Baby Shark malware",
    "tags": [
      "attack.execution",
      "attack.t1059",
      "attack.t1086",
      "attack.discovery",
      "attack.t1012",
      "attack.defense_evasion",
      "attack.t1170"
    ],
    "query": "winlog.event_data.CommandLine.keyword:(reg\\ query\\ \\\"HKEY_CURRENT_USER\\\\Software\\\\Microsoft\\\\Terminal\\ Server\\ Client\\\\Default\\\" OR powershell.exe\\ mshta.exe\\ http* OR cmd.exe\\ \\/c\\ taskkill\\ \\/im\\ cmd.exe)"
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
                    "query": "winlog.event_data.CommandLine.keyword:(reg\\ query\\ \\\"HKEY_CURRENT_USER\\\\Software\\\\Microsoft\\\\Terminal\\ Server\\ Client\\\\Default\\\" OR powershell.exe\\ mshta.exe\\ http* OR cmd.exe\\ \\/c\\ taskkill\\ \\/im\\ cmd.exe)",
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
        "subject": "Sigma Rule 'Baby Shark Activity'",
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
CommandLine.keyword:(reg query \"HKEY_CURRENT_USER\\Software\\Microsoft\\Terminal Server Client\\Default\" powershell.exe mshta.exe http* cmd.exe \/c taskkill \/im cmd.exe)
```


### splunk
    
```
(CommandLine="reg query \"HKEY_CURRENT_USER\\Software\\Microsoft\\Terminal Server Client\\Default\"" OR CommandLine="powershell.exe mshta.exe http*" OR CommandLine="cmd.exe /c taskkill /im cmd.exe")
```


### logpoint
    
```
CommandLine IN ["reg query \"HKEY_CURRENT_USER\\Software\\Microsoft\\Terminal Server Client\\Default\"", "powershell.exe mshta.exe http*", "cmd.exe /c taskkill /im cmd.exe"]
```


### grep
    
```
grep -P '^(?:.*reg query "HKEY_CURRENT_USER\Software\Microsoft\Terminal Server Client\Default"|.*powershell\.exe mshta\.exe http.*|.*cmd\.exe /c taskkill /im cmd\.exe)'
```



