| Title                    | Process Dump via Comsvcs DLL       |
|:-------------------------|:------------------|
| **Description**          | Detects process memory dump via comsvcs.dll and rundll32 |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li><li>[TA0006: Credential Access](https://attack.mitre.org/tactics/TA0006)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1218.011: Rundll32](https://attack.mitre.org/techniques/T1218/011)</li><li>[T1003.001: LSASS Memory](https://attack.mitre.org/techniques/T1003/001)</li><li>[T1003: OS Credential Dumping](https://attack.mitre.org/techniques/T1003)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1218.011: Rundll32](../Triggers/T1218.011.md)</li><li>[T1003.001: LSASS Memory](../Triggers/T1003.001.md)</li><li>[T1003: OS Credential Dumping](../Triggers/T1003.md)</li></ul>  |
| **Severity Level**       | medium |
| **False Positives**      | <ul><li>unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://modexp.wordpress.com/2019/08/30/minidumpwritedump-via-com-services-dll/](https://modexp.wordpress.com/2019/08/30/minidumpwritedump-via-com-services-dll/)</li><li>[https://twitter.com/SBousseaden/status/1167417096374050817](https://twitter.com/SBousseaden/status/1167417096374050817)</li></ul>  |
| **Author**               | Modexp (idea) |


## Detection Rules

### Sigma rule

```
title: Process Dump via Comsvcs DLL
id: 09e6d5c0-05b8-4ff8-9eeb-043046ec774c
status: experimental
description: Detects process memory dump via comsvcs.dll and rundll32
references:
    - https://modexp.wordpress.com/2019/08/30/minidumpwritedump-via-com-services-dll/
    - https://twitter.com/SBousseaden/status/1167417096374050817
author: Modexp (idea)
date: 2019/09/02
modified: 2020/09/05
logsource:
    category: process_creation
    product: windows
detection:
    rundll_image:
        Image: '*\rundll32.exe'
    rundll_ofn:
        OriginalFileName: 'RUNDLL32.EXE'
    selection:
        CommandLine:
            - '*comsvcs*MiniDump*full*'
            - '*comsvcs*MiniDumpW*full*'
    condition: (rundll_image or rundll_ofn) and selection
fields:
    - CommandLine
    - ParentCommandLine
tags:
    - attack.defense_evasion
    - attack.t1218.011
    - attack.credential_access
    - attack.t1003.001
    - attack.t1003      # an old one
falsepositives:
    - unknown
level: medium

```





### powershell
    
```
Get-WinEvent | where {(($_.message -match "Image.*.*\\\\rundll32.exe" -or $_.message -match "OriginalFileName.*RUNDLL32.EXE") -and ($_.message -match "CommandLine.*.*comsvcs.*MiniDump.*full.*" -or $_.message -match "CommandLine.*.*comsvcs.*MiniDumpW.*full.*")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
((winlog.event_data.Image.keyword:*\\\\rundll32.exe OR OriginalFileName:"RUNDLL32.EXE") AND winlog.event_data.CommandLine.keyword:(*comsvcs*MiniDump*full* OR *comsvcs*MiniDumpW*full*))
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/09e6d5c0-05b8-4ff8-9eeb-043046ec774c <<EOF\n{\n  "metadata": {\n    "title": "Process Dump via Comsvcs DLL",\n    "description": "Detects process memory dump via comsvcs.dll and rundll32",\n    "tags": [\n      "attack.defense_evasion",\n      "attack.t1218.011",\n      "attack.credential_access",\n      "attack.t1003.001",\n      "attack.t1003"\n    ],\n    "query": "((winlog.event_data.Image.keyword:*\\\\\\\\rundll32.exe OR OriginalFileName:\\"RUNDLL32.EXE\\") AND winlog.event_data.CommandLine.keyword:(*comsvcs*MiniDump*full* OR *comsvcs*MiniDumpW*full*))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "((winlog.event_data.Image.keyword:*\\\\\\\\rundll32.exe OR OriginalFileName:\\"RUNDLL32.EXE\\") AND winlog.event_data.CommandLine.keyword:(*comsvcs*MiniDump*full* OR *comsvcs*MiniDumpW*full*))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Process Dump via Comsvcs DLL\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}Hit on {{_source.@timestamp}}:\\n      CommandLine = {{_source.CommandLine}}\\nParentCommandLine = {{_source.ParentCommandLine}}================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
((Image.keyword:*\\\\rundll32.exe OR OriginalFileName:"RUNDLL32.EXE") AND CommandLine.keyword:(*comsvcs*MiniDump*full* *comsvcs*MiniDumpW*full*))
```


### splunk
    
```
((Image="*\\\\rundll32.exe" OR OriginalFileName="RUNDLL32.EXE") (CommandLine="*comsvcs*MiniDump*full*" OR CommandLine="*comsvcs*MiniDumpW*full*")) | table CommandLine,ParentCommandLine
```


### logpoint
    
```
((Image="*\\\\rundll32.exe" OR OriginalFileName="RUNDLL32.EXE") CommandLine IN ["*comsvcs*MiniDump*full*", "*comsvcs*MiniDumpW*full*"])
```


### grep
    
```
grep -P '^(?:.*(?=.*(?:.*(?:.*.*\\rundll32\\.exe|.*RUNDLL32\\.EXE)))(?=.*(?:.*.*comsvcs.*MiniDump.*full.*|.*.*comsvcs.*MiniDumpW.*full.*)))'
```



