| Title                    | Detection of PowerShell Execution via DLL       |
|:-------------------------|:------------------|
| **Description**          | Detects PowerShell Strings applied to rundll as seen in PowerShdll.dll |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1085: Rundll32](https://attack.mitre.org/techniques/T1085)</li><li>[T1218.011: Rundll32](https://attack.mitre.org/techniques/T1218/011)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1218.011: Rundll32](../Triggers/T1218.011.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://github.com/p3nt4/PowerShdll/blob/master/README.md](https://github.com/p3nt4/PowerShdll/blob/master/README.md)</li></ul>  |
| **Author**               | Markus Neis |


## Detection Rules

### Sigma rule

```
title: Detection of PowerShell Execution via DLL
id: 6812a10b-60ea-420c-832f-dfcc33b646ba
status: experimental
description: Detects PowerShell Strings applied to rundll as seen in PowerShdll.dll
references:
    - https://github.com/p3nt4/PowerShdll/blob/master/README.md
tags:
    - attack.defense_evasion
    - attack.t1085          # an old one
    - attack.t1218.011
author: Markus Neis
date: 2018/08/25
modified: 2020/09/01
logsource:
    category: process_creation
    product: windows
detection:
    selection1:
        Image:
            - '*\rundll32.exe'
    selection2:
        Description:
            - '*Windows-Hostprozess (Rundll32)*'
    selection3:
        CommandLine:
            - '*Default.GetString*'
            - '*FromBase64String*'
    condition: (selection1 or selection2) and selection3
falsepositives:
    - Unknown
level: high

```





### powershell
    
```
Get-WinEvent | where {((($_.message -match "Image.*.*\\\\rundll32.exe") -or ($_.message -match "Description.*.*Windows-Hostprozess (Rundll32).*")) -and ($_.message -match "CommandLine.*.*Default.GetString.*" -or $_.message -match "CommandLine.*.*FromBase64String.*")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
((winlog.event_data.Image.keyword:(*\\\\rundll32.exe) OR winlog.event_data.Description.keyword:(*Windows\\-Hostprozess\\ \\(Rundll32\\)*)) AND winlog.event_data.CommandLine.keyword:(*Default.GetString* OR *FromBase64String*))
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/6812a10b-60ea-420c-832f-dfcc33b646ba <<EOF\n{\n  "metadata": {\n    "title": "Detection of PowerShell Execution via DLL",\n    "description": "Detects PowerShell Strings applied to rundll as seen in PowerShdll.dll",\n    "tags": [\n      "attack.defense_evasion",\n      "attack.t1085",\n      "attack.t1218.011"\n    ],\n    "query": "((winlog.event_data.Image.keyword:(*\\\\\\\\rundll32.exe) OR winlog.event_data.Description.keyword:(*Windows\\\\-Hostprozess\\\\ \\\\(Rundll32\\\\)*)) AND winlog.event_data.CommandLine.keyword:(*Default.GetString* OR *FromBase64String*))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "((winlog.event_data.Image.keyword:(*\\\\\\\\rundll32.exe) OR winlog.event_data.Description.keyword:(*Windows\\\\-Hostprozess\\\\ \\\\(Rundll32\\\\)*)) AND winlog.event_data.CommandLine.keyword:(*Default.GetString* OR *FromBase64String*))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Detection of PowerShell Execution via DLL\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
((Image.keyword:(*\\\\rundll32.exe) OR Description.keyword:(*Windows\\-Hostprozess \\(Rundll32\\)*)) AND CommandLine.keyword:(*Default.GetString* *FromBase64String*))
```


### splunk
    
```
(((Image="*\\\\rundll32.exe") OR (Description="*Windows-Hostprozess (Rundll32)*")) (CommandLine="*Default.GetString*" OR CommandLine="*FromBase64String*"))
```


### logpoint
    
```
((Image IN ["*\\\\rundll32.exe"] OR Description IN ["*Windows-Hostprozess (Rundll32)*"]) CommandLine IN ["*Default.GetString*", "*FromBase64String*"])
```


### grep
    
```
grep -P '^(?:.*(?=.*(?:.*(?:.*(?:.*.*\\rundll32\\.exe)|.*(?:.*.*Windows-Hostprozess \\(Rundll32\\).*))))(?=.*(?:.*.*Default\\.GetString.*|.*.*FromBase64String.*)))'
```



