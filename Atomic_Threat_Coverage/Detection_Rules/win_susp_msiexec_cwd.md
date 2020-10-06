| Title                    | Suspicious MsiExec Directory       |
|:-------------------------|:------------------|
| **Description**          | Detects suspicious msiexec process starts in an uncommon directory |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1036.005: Match Legitimate Name or Location](https://attack.mitre.org/techniques/T1036/005)</li><li>[T1036: Masquerading](https://attack.mitre.org/techniques/T1036)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0001_4688_windows_process_creation](../Data_Needed/DN_0001_4688_windows_process_creation.md)</li><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              |  There is no documented Trigger for this Detection Rule yet  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://twitter.com/200_okay_/status/1194765831911215104](https://twitter.com/200_okay_/status/1194765831911215104)</li></ul>  |
| **Author**               | Florian Roth |


## Detection Rules

### Sigma rule

```
title: Suspicious MsiExec Directory
id: e22a6eb2-f8a5-44b5-8b44-a2dbd47b1144
status: experimental
description: Detects suspicious msiexec process starts in an uncommon directory
references:
    - https://twitter.com/200_okay_/status/1194765831911215104
tags:
    - attack.defense_evasion
    - attack.t1036.005
    - attack.t1036      # an old one
author: Florian Roth
date: 2019/11/14
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        Image: '*\msiexec.exe'
    filter:
        Image: 
            - 'C:\Windows\System32\\*'
            - 'C:\Windows\SysWOW64\\*'
            - 'C:\Windows\WinSxS\\*' 
    condition: selection and not filter
falsepositives:
    - Unknown
level: high

```





### powershell
    
```
Get-WinEvent | where {($_.message -match "Image.*.*\\\\msiexec.exe" -and  -not (($_.message -match "Image.*C:\\\\Windows\\\\System32\\\\.*" -or $_.message -match "Image.*C:\\\\Windows\\\\SysWOW64\\\\.*" -or $_.message -match "Image.*C:\\\\Windows\\\\WinSxS\\\\.*"))) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.event_data.Image.keyword:*\\\\msiexec.exe AND (NOT (winlog.event_data.Image.keyword:(C\\:\\\\Windows\\\\System32\\\\* OR C\\:\\\\Windows\\\\SysWOW64\\\\* OR C\\:\\\\Windows\\\\WinSxS\\\\*))))
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/e22a6eb2-f8a5-44b5-8b44-a2dbd47b1144 <<EOF\n{\n  "metadata": {\n    "title": "Suspicious MsiExec Directory",\n    "description": "Detects suspicious msiexec process starts in an uncommon directory",\n    "tags": [\n      "attack.defense_evasion",\n      "attack.t1036.005",\n      "attack.t1036"\n    ],\n    "query": "(winlog.event_data.Image.keyword:*\\\\\\\\msiexec.exe AND (NOT (winlog.event_data.Image.keyword:(C\\\\:\\\\\\\\Windows\\\\\\\\System32\\\\\\\\* OR C\\\\:\\\\\\\\Windows\\\\\\\\SysWOW64\\\\\\\\* OR C\\\\:\\\\\\\\Windows\\\\\\\\WinSxS\\\\\\\\*))))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.event_data.Image.keyword:*\\\\\\\\msiexec.exe AND (NOT (winlog.event_data.Image.keyword:(C\\\\:\\\\\\\\Windows\\\\\\\\System32\\\\\\\\* OR C\\\\:\\\\\\\\Windows\\\\\\\\SysWOW64\\\\\\\\* OR C\\\\:\\\\\\\\Windows\\\\\\\\WinSxS\\\\\\\\*))))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Suspicious MsiExec Directory\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
(Image.keyword:*\\\\msiexec.exe AND (NOT (Image.keyword:(C\\:\\\\Windows\\\\System32\\\\* C\\:\\\\Windows\\\\SysWOW64\\\\* C\\:\\\\Windows\\\\WinSxS\\\\*))))
```


### splunk
    
```
(Image="*\\\\msiexec.exe" NOT ((Image="C:\\\\Windows\\\\System32\\\\*" OR Image="C:\\\\Windows\\\\SysWOW64\\\\*" OR Image="C:\\\\Windows\\\\WinSxS\\\\*")))
```


### logpoint
    
```
(Image="*\\\\msiexec.exe"  -(Image IN ["C:\\\\Windows\\\\System32\\\\*", "C:\\\\Windows\\\\SysWOW64\\\\*", "C:\\\\Windows\\\\WinSxS\\\\*"]))
```


### grep
    
```
grep -P '^(?:.*(?=.*.*\\msiexec\\.exe)(?=.*(?!.*(?:.*(?=.*(?:.*C:\\Windows\\System32\\\\.*|.*C:\\Windows\\SysWOW64\\\\.*|.*C:\\Windows\\WinSxS\\\\.*))))))'
```



