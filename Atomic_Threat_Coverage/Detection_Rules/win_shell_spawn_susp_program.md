| Title                    | Windows Shell Spawning Suspicious Program       |
|:-------------------------|:------------------|
| **Description**          | Detects a suspicious child process of a Windows shell |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1064: Scripting](https://attack.mitre.org/techniques/T1064)</li><li>[T1059.005: Visual Basic](https://attack.mitre.org/techniques/T1059/005)</li><li>[T1059.001: PowerShell](https://attack.mitre.org/techniques/T1059/001)</li><li>[T1218: Signed Binary Proxy Execution](https://attack.mitre.org/techniques/T1218)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1059.005: Visual Basic](../Triggers/T1059.005.md)</li><li>[T1059.001: PowerShell](../Triggers/T1059.001.md)</li><li>[T1218: Signed Binary Proxy Execution](../Triggers/T1218.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Administrative scripts</li><li>Microsoft SCCM</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://mgreen27.github.io/posts/2018/04/02/DownloadCradle.html](https://mgreen27.github.io/posts/2018/04/02/DownloadCradle.html)</li></ul>  |
| **Author**               | Florian Roth |


## Detection Rules

### Sigma rule

```
title: Windows Shell Spawning Suspicious Program
id: 3a6586ad-127a-4d3b-a677-1e6eacdf8fde
status: experimental
description: Detects a suspicious child process of a Windows shell
references:
    - https://mgreen27.github.io/posts/2018/04/02/DownloadCradle.html
author: Florian Roth
date: 2018/04/06
modified: 2020/09/06
tags:
    - attack.execution
    - attack.defense_evasion
    - attack.t1064 # an old one
    - attack.t1059.005
    - attack.t1059.001
    - attack.t1218    
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        ParentImage:
            - '*\mshta.exe'
            - '*\powershell.exe'
            # - '*\cmd.exe'  # too many false positives
            - '*\rundll32.exe'
            - '*\cscript.exe'
            - '*\wscript.exe'
            - '*\wmiprvse.exe'
        Image:
            - '*\schtasks.exe'
            - '*\nslookup.exe'
            - '*\certutil.exe'
            - '*\bitsadmin.exe'
            - '*\mshta.exe'
    falsepositives:
        CurrentDirectory: '*\ccmcache\\*'
    condition: selection and not falsepositives
fields:
    - CommandLine
    - ParentCommandLine
falsepositives:
    - Administrative scripts
    - Microsoft SCCM
level: high

```





### powershell
    
```
Get-WinEvent | where {((($_.message -match "ParentImage.*.*\\\\mshta.exe" -or $_.message -match "ParentImage.*.*\\\\powershell.exe" -or $_.message -match "ParentImage.*.*\\\\rundll32.exe" -or $_.message -match "ParentImage.*.*\\\\cscript.exe" -or $_.message -match "ParentImage.*.*\\\\wscript.exe" -or $_.message -match "ParentImage.*.*\\\\wmiprvse.exe") -and ($_.message -match "Image.*.*\\\\schtasks.exe" -or $_.message -match "Image.*.*\\\\nslookup.exe" -or $_.message -match "Image.*.*\\\\certutil.exe" -or $_.message -match "Image.*.*\\\\bitsadmin.exe" -or $_.message -match "Image.*.*\\\\mshta.exe")) -and  -not ($_.message -match "CurrentDirectory.*.*\\\\ccmcache\\\\.*")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
((winlog.event_data.ParentImage.keyword:(*\\\\mshta.exe OR *\\\\powershell.exe OR *\\\\rundll32.exe OR *\\\\cscript.exe OR *\\\\wscript.exe OR *\\\\wmiprvse.exe) AND winlog.event_data.Image.keyword:(*\\\\schtasks.exe OR *\\\\nslookup.exe OR *\\\\certutil.exe OR *\\\\bitsadmin.exe OR *\\\\mshta.exe)) AND (NOT (winlog.event_data.CurrentDirectory.keyword:*\\\\ccmcache\\\\*)))
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/3a6586ad-127a-4d3b-a677-1e6eacdf8fde <<EOF\n{\n  "metadata": {\n    "title": "Windows Shell Spawning Suspicious Program",\n    "description": "Detects a suspicious child process of a Windows shell",\n    "tags": [\n      "attack.execution",\n      "attack.defense_evasion",\n      "attack.t1064",\n      "attack.t1059.005",\n      "attack.t1059.001",\n      "attack.t1218"\n    ],\n    "query": "((winlog.event_data.ParentImage.keyword:(*\\\\\\\\mshta.exe OR *\\\\\\\\powershell.exe OR *\\\\\\\\rundll32.exe OR *\\\\\\\\cscript.exe OR *\\\\\\\\wscript.exe OR *\\\\\\\\wmiprvse.exe) AND winlog.event_data.Image.keyword:(*\\\\\\\\schtasks.exe OR *\\\\\\\\nslookup.exe OR *\\\\\\\\certutil.exe OR *\\\\\\\\bitsadmin.exe OR *\\\\\\\\mshta.exe)) AND (NOT (winlog.event_data.CurrentDirectory.keyword:*\\\\\\\\ccmcache\\\\\\\\*)))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "((winlog.event_data.ParentImage.keyword:(*\\\\\\\\mshta.exe OR *\\\\\\\\powershell.exe OR *\\\\\\\\rundll32.exe OR *\\\\\\\\cscript.exe OR *\\\\\\\\wscript.exe OR *\\\\\\\\wmiprvse.exe) AND winlog.event_data.Image.keyword:(*\\\\\\\\schtasks.exe OR *\\\\\\\\nslookup.exe OR *\\\\\\\\certutil.exe OR *\\\\\\\\bitsadmin.exe OR *\\\\\\\\mshta.exe)) AND (NOT (winlog.event_data.CurrentDirectory.keyword:*\\\\\\\\ccmcache\\\\\\\\*)))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Windows Shell Spawning Suspicious Program\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}Hit on {{_source.@timestamp}}:\\n      CommandLine = {{_source.CommandLine}}\\nParentCommandLine = {{_source.ParentCommandLine}}================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
((ParentImage.keyword:(*\\\\mshta.exe *\\\\powershell.exe *\\\\rundll32.exe *\\\\cscript.exe *\\\\wscript.exe *\\\\wmiprvse.exe) AND Image.keyword:(*\\\\schtasks.exe *\\\\nslookup.exe *\\\\certutil.exe *\\\\bitsadmin.exe *\\\\mshta.exe)) AND (NOT (CurrentDirectory.keyword:*\\\\ccmcache\\\\*)))
```


### splunk
    
```
(((ParentImage="*\\\\mshta.exe" OR ParentImage="*\\\\powershell.exe" OR ParentImage="*\\\\rundll32.exe" OR ParentImage="*\\\\cscript.exe" OR ParentImage="*\\\\wscript.exe" OR ParentImage="*\\\\wmiprvse.exe") (Image="*\\\\schtasks.exe" OR Image="*\\\\nslookup.exe" OR Image="*\\\\certutil.exe" OR Image="*\\\\bitsadmin.exe" OR Image="*\\\\mshta.exe")) NOT (CurrentDirectory="*\\\\ccmcache\\\\*")) | table CommandLine,ParentCommandLine
```


### logpoint
    
```
((ParentImage IN ["*\\\\mshta.exe", "*\\\\powershell.exe", "*\\\\rundll32.exe", "*\\\\cscript.exe", "*\\\\wscript.exe", "*\\\\wmiprvse.exe"] Image IN ["*\\\\schtasks.exe", "*\\\\nslookup.exe", "*\\\\certutil.exe", "*\\\\bitsadmin.exe", "*\\\\mshta.exe"])  -(CurrentDirectory="*\\\\ccmcache\\\\*"))
```


### grep
    
```
grep -P '^(?:.*(?=.*(?:.*(?=.*(?:.*.*\\mshta\\.exe|.*.*\\powershell\\.exe|.*.*\\rundll32\\.exe|.*.*\\cscript\\.exe|.*.*\\wscript\\.exe|.*.*\\wmiprvse\\.exe))(?=.*(?:.*.*\\schtasks\\.exe|.*.*\\nslookup\\.exe|.*.*\\certutil\\.exe|.*.*\\bitsadmin\\.exe|.*.*\\mshta\\.exe))))(?=.*(?!.*(?:.*(?=.*.*\\ccmcache\\\\.*)))))'
```



