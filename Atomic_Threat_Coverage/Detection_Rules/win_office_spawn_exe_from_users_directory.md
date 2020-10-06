| Title                    | MS Office Product Spawning Exe in User Dir       |
|:-------------------------|:------------------|
| **Description**          | Detects an executable in the users directory started from Microsoft Word, Excel, Powerpoint, Publisher or Visio |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1204: User Execution](https://attack.mitre.org/techniques/T1204)</li><li>[T1204.002: Malicious File](https://attack.mitre.org/techniques/T1204/002)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1204.002: Malicious File](../Triggers/T1204.002.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[sha256=23160972c6ae07f740800fa28e421a81d7c0ca5d5cab95bc082b4a986fbac57c](sha256=23160972c6ae07f740800fa28e421a81d7c0ca5d5cab95bc082b4a986fbac57c)</li><li>[https://blog.morphisec.com/fin7-not-finished-morphisec-spots-new-campaign](https://blog.morphisec.com/fin7-not-finished-morphisec-spots-new-campaign)</li></ul>  |
| **Author**               | Jason Lynch |
| Other Tags           | <ul><li>FIN7</li><li>car.2013-05-002</li></ul> | 

## Detection Rules

### Sigma rule

```
title: MS Office Product Spawning Exe in User Dir
id: aa3a6f94-890e-4e22-b634-ffdfd54792cc
status: experimental
description: Detects an executable in the users directory started from Microsoft Word, Excel, Powerpoint, Publisher or Visio
references:
    - sha256=23160972c6ae07f740800fa28e421a81d7c0ca5d5cab95bc082b4a986fbac57c
    - https://blog.morphisec.com/fin7-not-finished-morphisec-spots-new-campaign
tags:
    - attack.execution
    - attack.t1204          # an old one
    - attack.t1204.002
    - FIN7
    - car.2013-05-002
author: Jason Lynch 
date: 2019/04/02
modified: 2020/09/01
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        ParentImage:
            - '*\WINWORD.EXE'
            - '*\EXCEL.EXE'
            - '*\POWERPNT.exe'
            - '*\MSPUB.exe'
            - '*\VISIO.exe'
            - '*\OUTLOOK.EXE'
        Image:
            - 'C:\users\\*.exe'
    condition: selection
fields:
    - CommandLine
    - ParentCommandLine
falsepositives:
    - unknown
level: high

```





### powershell
    
```
Get-WinEvent | where {(($_.message -match "ParentImage.*.*\\\\WINWORD.EXE" -or $_.message -match "ParentImage.*.*\\\\EXCEL.EXE" -or $_.message -match "ParentImage.*.*\\\\POWERPNT.exe" -or $_.message -match "ParentImage.*.*\\\\MSPUB.exe" -or $_.message -match "ParentImage.*.*\\\\VISIO.exe" -or $_.message -match "ParentImage.*.*\\\\OUTLOOK.EXE") -and ($_.message -match "Image.*C:\\\\users\\\\.*.exe")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.event_data.ParentImage.keyword:(*\\\\WINWORD.EXE OR *\\\\EXCEL.EXE OR *\\\\POWERPNT.exe OR *\\\\MSPUB.exe OR *\\\\VISIO.exe OR *\\\\OUTLOOK.EXE) AND winlog.event_data.Image.keyword:(C\\:\\\\users\\\\*.exe))
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/aa3a6f94-890e-4e22-b634-ffdfd54792cc <<EOF\n{\n  "metadata": {\n    "title": "MS Office Product Spawning Exe in User Dir",\n    "description": "Detects an executable in the users directory started from Microsoft Word, Excel, Powerpoint, Publisher or Visio",\n    "tags": [\n      "attack.execution",\n      "attack.t1204",\n      "attack.t1204.002",\n      "FIN7",\n      "car.2013-05-002"\n    ],\n    "query": "(winlog.event_data.ParentImage.keyword:(*\\\\\\\\WINWORD.EXE OR *\\\\\\\\EXCEL.EXE OR *\\\\\\\\POWERPNT.exe OR *\\\\\\\\MSPUB.exe OR *\\\\\\\\VISIO.exe OR *\\\\\\\\OUTLOOK.EXE) AND winlog.event_data.Image.keyword:(C\\\\:\\\\\\\\users\\\\\\\\*.exe))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.event_data.ParentImage.keyword:(*\\\\\\\\WINWORD.EXE OR *\\\\\\\\EXCEL.EXE OR *\\\\\\\\POWERPNT.exe OR *\\\\\\\\MSPUB.exe OR *\\\\\\\\VISIO.exe OR *\\\\\\\\OUTLOOK.EXE) AND winlog.event_data.Image.keyword:(C\\\\:\\\\\\\\users\\\\\\\\*.exe))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'MS Office Product Spawning Exe in User Dir\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}Hit on {{_source.@timestamp}}:\\n      CommandLine = {{_source.CommandLine}}\\nParentCommandLine = {{_source.ParentCommandLine}}================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
(ParentImage.keyword:(*\\\\WINWORD.EXE *\\\\EXCEL.EXE *\\\\POWERPNT.exe *\\\\MSPUB.exe *\\\\VISIO.exe *\\\\OUTLOOK.EXE) AND Image.keyword:(C\\:\\\\users\\\\*.exe))
```


### splunk
    
```
((ParentImage="*\\\\WINWORD.EXE" OR ParentImage="*\\\\EXCEL.EXE" OR ParentImage="*\\\\POWERPNT.exe" OR ParentImage="*\\\\MSPUB.exe" OR ParentImage="*\\\\VISIO.exe" OR ParentImage="*\\\\OUTLOOK.EXE") (Image="C:\\\\users\\\\*.exe")) | table CommandLine,ParentCommandLine
```


### logpoint
    
```
(ParentImage IN ["*\\\\WINWORD.EXE", "*\\\\EXCEL.EXE", "*\\\\POWERPNT.exe", "*\\\\MSPUB.exe", "*\\\\VISIO.exe", "*\\\\OUTLOOK.EXE"] Image IN ["C:\\\\users\\\\*.exe"])
```


### grep
    
```
grep -P '^(?:.*(?=.*(?:.*.*\\WINWORD\\.EXE|.*.*\\EXCEL\\.EXE|.*.*\\POWERPNT\\.exe|.*.*\\MSPUB\\.exe|.*.*\\VISIO\\.exe|.*.*\\OUTLOOK\\.EXE))(?=.*(?:.*C:\\users\\\\.*\\.exe)))'
```



