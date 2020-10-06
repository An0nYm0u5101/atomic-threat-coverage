| Title                    | Possible Shim Database Persistence via sdbinst.exe       |
|:-------------------------|:------------------|
| **Description**          | Detects installation of a new shim using sdbinst.exe. A shim can be used to load malicious DLLs into applications. |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0003: Persistence](https://attack.mitre.org/tactics/TA0003)</li><li>[TA0004: Privilege Escalation](https://attack.mitre.org/tactics/TA0004)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1546.011: Application Shimming](https://attack.mitre.org/techniques/T1546/011)</li><li>[T1138: Application Shimming](https://attack.mitre.org/techniques/T1138)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1546.011: Application Shimming](../Triggers/T1546.011.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://www.fireeye.com/blog/threat-research/2017/05/fin7-shim-databases-persistence.html](https://www.fireeye.com/blog/threat-research/2017/05/fin7-shim-databases-persistence.html)</li></ul>  |
| **Author**               | Markus Neis |


## Detection Rules

### Sigma rule

```
title: Possible Shim Database Persistence via sdbinst.exe
id: 517490a7-115a-48c6-8862-1a481504d5a8
status: experimental
description: Detects installation of a new shim using sdbinst.exe. A shim can be used to load malicious DLLs into applications.
references:
    - https://www.fireeye.com/blog/threat-research/2017/05/fin7-shim-databases-persistence.html
tags:
    - attack.persistence
    - attack.privilege_escalation
    - attack.t1546.011
    - attack.t1138 # an old one
author: Markus Neis
date: 2019/01/16
modified: 2020/09/06
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        Image:
            - '*\sdbinst.exe'
        CommandLine:
            - '*.sdb*'
    condition: selection
falsepositives:
    - Unknown
level: high

```





### powershell
    
```
Get-WinEvent | where {(($_.message -match "Image.*.*\\\\sdbinst.exe") -and ($_.message -match "CommandLine.*.*.sdb.*")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.event_data.Image.keyword:(*\\\\sdbinst.exe) AND winlog.event_data.CommandLine.keyword:(*.sdb*))
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/517490a7-115a-48c6-8862-1a481504d5a8 <<EOF\n{\n  "metadata": {\n    "title": "Possible Shim Database Persistence via sdbinst.exe",\n    "description": "Detects installation of a new shim using sdbinst.exe. A shim can be used to load malicious DLLs into applications.",\n    "tags": [\n      "attack.persistence",\n      "attack.privilege_escalation",\n      "attack.t1546.011",\n      "attack.t1138"\n    ],\n    "query": "(winlog.event_data.Image.keyword:(*\\\\\\\\sdbinst.exe) AND winlog.event_data.CommandLine.keyword:(*.sdb*))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.event_data.Image.keyword:(*\\\\\\\\sdbinst.exe) AND winlog.event_data.CommandLine.keyword:(*.sdb*))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Possible Shim Database Persistence via sdbinst.exe\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
(Image.keyword:(*\\\\sdbinst.exe) AND CommandLine.keyword:(*.sdb*))
```


### splunk
    
```
((Image="*\\\\sdbinst.exe") (CommandLine="*.sdb*"))
```


### logpoint
    
```
(Image IN ["*\\\\sdbinst.exe"] CommandLine IN ["*.sdb*"])
```


### grep
    
```
grep -P '^(?:.*(?=.*(?:.*.*\\sdbinst\\.exe))(?=.*(?:.*.*\\.sdb.*)))'
```



