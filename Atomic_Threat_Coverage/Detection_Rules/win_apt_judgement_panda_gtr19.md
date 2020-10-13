| Title                    | Judgement Panda Exfil Activity       |
|:-------------------------|:------------------|
| **Description**          | Detects Judgement Panda activity as described in Global Threat Report 2019 by Crowdstrike |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0008: Lateral Movement](https://attack.mitre.org/tactics/TA0008)</li><li>[TA0006: Credential Access](https://attack.mitre.org/tactics/TA0006)</li><li>[TA0010: Exfiltration](https://attack.mitre.org/tactics/TA0010)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1003: OS Credential Dumping](https://attack.mitre.org/techniques/T1003)</li><li>[T1003.001: LSASS Memory](https://attack.mitre.org/techniques/T1003/001)</li><li>[T1002: Data Compressed](https://attack.mitre.org/techniques/T1002)</li><li>[T1560.001: Archive via Utility](https://attack.mitre.org/techniques/T1560/001)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1003: OS Credential Dumping](../Triggers/T1003.md)</li><li>[T1003.001: LSASS Memory](../Triggers/T1003.001.md)</li><li>[T1560.001: Archive via Utility](../Triggers/T1560.001.md)</li></ul>  |
| **Severity Level**       | critical |
| **False Positives**      | <ul><li>unknown</li></ul>  |
| **Development Status**   |  Development Status wasn't defined for this Detection Rule yet  |
| **References**           | <ul><li>[https://www.crowdstrike.com/resources/reports/2019-crowdstrike-global-threat-report/](https://www.crowdstrike.com/resources/reports/2019-crowdstrike-global-threat-report/)</li></ul>  |
| **Author**               | Florian Roth |
| Other Tags           | <ul><li>attack.g0010</li></ul> | 

## Detection Rules

### Sigma rule

```
title: Judgement Panda Exfil Activity
id: 03e2746e-2b31-42f1-ab7a-eb39365b2422
description: Detects Judgement Panda activity as described in Global Threat Report 2019 by Crowdstrike
references:
    - https://www.crowdstrike.com/resources/reports/2019-crowdstrike-global-threat-report/
author: Florian Roth
date: 2019/02/21
modified: 2020/08/27
tags:
    - attack.lateral_movement
    - attack.g0010
    - attack.credential_access
    - attack.t1003 # an old one
    - attack.t1003.001
    - attack.exfiltration
    - attack.t1002 # an old one
    - attack.t1560.001
logsource:
    category: process_creation
    product: windows
detection:
    selection1:
        CommandLine:
            - '*\ldifde.exe -f -n *'
            - '*\7za.exe a 1.7z *'
            - '* eprod.ldf'
            - '*\aaaa\procdump64.exe*'
            - '*\aaaa\netsess.exe*'
            - '*\aaaa\7za.exe*'
            - '*copy .\1.7z \\*'
            - '*copy \\client\c$\aaaa\\*'
    selection2:
        Image: C:\Users\Public\7za.exe
    condition: selection1 or selection2
falsepositives:
    - unknown
level: critical

```





### powershell
    
```
Get-WinEvent | where {(($_.message -match "CommandLine.*.*\\ldifde.exe -f -n .*" -or $_.message -match "CommandLine.*.*\\7za.exe a 1.7z .*" -or $_.message -match "CommandLine.*.* eprod.ldf" -or $_.message -match "CommandLine.*.*\\aaaa\\procdump64.exe.*" -or $_.message -match "CommandLine.*.*\\aaaa\\netsess.exe.*" -or $_.message -match "CommandLine.*.*\\aaaa\\7za.exe.*" -or $_.message -match "CommandLine.*.*copy .\\1.7z \\.*" -or $_.message -match "CommandLine.*.*copy \\client\\c$\\aaaa\\.*") -or $_.message -match "Image.*C:\\Users\\Public\\7za.exe") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.event_data.CommandLine.keyword:(*\\ldifde.exe\ \-f\ \-n\ * OR *\\7za.exe\ a\ 1.7z\ * OR *\ eprod.ldf OR *\\aaaa\\procdump64.exe* OR *\\aaaa\\netsess.exe* OR *\\aaaa\\7za.exe* OR *copy\ .\\1.7z\ \\* OR *copy\ \\client\\c$\\aaaa\\*) OR winlog.event_data.Image:"C\:\\Users\\Public\\7za.exe")
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/03e2746e-2b31-42f1-ab7a-eb39365b2422 <<EOF
{
  "metadata": {
    "title": "Judgement Panda Exfil Activity",
    "description": "Detects Judgement Panda activity as described in Global Threat Report 2019 by Crowdstrike",
    "tags": [
      "attack.lateral_movement",
      "attack.g0010",
      "attack.credential_access",
      "attack.t1003",
      "attack.t1003.001",
      "attack.exfiltration",
      "attack.t1002",
      "attack.t1560.001"
    ],
    "query": "(winlog.event_data.CommandLine.keyword:(*\\\\ldifde.exe\\ \\-f\\ \\-n\\ * OR *\\\\7za.exe\\ a\\ 1.7z\\ * OR *\\ eprod.ldf OR *\\\\aaaa\\\\procdump64.exe* OR *\\\\aaaa\\\\netsess.exe* OR *\\\\aaaa\\\\7za.exe* OR *copy\\ .\\\\1.7z\\ \\\\* OR *copy\\ \\\\client\\\\c$\\\\aaaa\\\\*) OR winlog.event_data.Image:\"C\\:\\\\Users\\\\Public\\\\7za.exe\")"
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
                    "query": "(winlog.event_data.CommandLine.keyword:(*\\\\ldifde.exe\\ \\-f\\ \\-n\\ * OR *\\\\7za.exe\\ a\\ 1.7z\\ * OR *\\ eprod.ldf OR *\\\\aaaa\\\\procdump64.exe* OR *\\\\aaaa\\\\netsess.exe* OR *\\\\aaaa\\\\7za.exe* OR *copy\\ .\\\\1.7z\\ \\\\* OR *copy\\ \\\\client\\\\c$\\\\aaaa\\\\*) OR winlog.event_data.Image:\"C\\:\\\\Users\\\\Public\\\\7za.exe\")",
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
        "subject": "Sigma Rule 'Judgement Panda Exfil Activity'",
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
(CommandLine.keyword:(*\\ldifde.exe \-f \-n * *\\7za.exe a 1.7z * * eprod.ldf *\\aaaa\\procdump64.exe* *\\aaaa\\netsess.exe* *\\aaaa\\7za.exe* *copy .\\1.7z \\* *copy \\client\\c$\\aaaa\\*) OR Image:"C\:\\Users\\Public\\7za.exe")
```


### splunk
    
```
((CommandLine="*\\ldifde.exe -f -n *" OR CommandLine="*\\7za.exe a 1.7z *" OR CommandLine="* eprod.ldf" OR CommandLine="*\\aaaa\\procdump64.exe*" OR CommandLine="*\\aaaa\\netsess.exe*" OR CommandLine="*\\aaaa\\7za.exe*" OR CommandLine="*copy .\\1.7z \\*" OR CommandLine="*copy \\client\\c$\\aaaa\\*") OR Image="C:\\Users\\Public\\7za.exe")
```


### logpoint
    
```
(CommandLine IN ["*\\ldifde.exe -f -n *", "*\\7za.exe a 1.7z *", "* eprod.ldf", "*\\aaaa\\procdump64.exe*", "*\\aaaa\\netsess.exe*", "*\\aaaa\\7za.exe*", "*copy .\\1.7z \\*", "*copy \\client\\c$\\aaaa\\*"] OR Image="C:\\Users\\Public\\7za.exe")
```


### grep
    
```
grep -P '^(?:.*(?:.*(?:.*.*\ldifde\.exe -f -n .*|.*.*\7za\.exe a 1\.7z .*|.*.* eprod\.ldf|.*.*\aaaa\procdump64\.exe.*|.*.*\aaaa\netsess\.exe.*|.*.*\aaaa\7za\.exe.*|.*.*copy \.\1\.7z \\.*|.*.*copy \\client\c\$\aaaa\\.*)|.*C:\Users\Public\7za\.exe))'
```



