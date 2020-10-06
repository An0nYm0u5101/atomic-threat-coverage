| Title                    | LSASS Access Detected via Attack Surface Reduction       |
|:-------------------------|:------------------|
| **Description**          | Detects Access to LSASS Process |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0006: Credential Access](https://attack.mitre.org/tactics/TA0006)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1003: OS Credential Dumping](https://attack.mitre.org/techniques/T1003)</li><li>[T1003.001: LSASS Memory](https://attack.mitre.org/techniques/T1003/001)</li></ul>  |
| **Data Needed**          |  There is no documented Data Needed for this Detection Rule yet  |
| **Trigger**              | <ul><li>[T1003: OS Credential Dumping](../Triggers/T1003.md)</li><li>[T1003.001: LSASS Memory](../Triggers/T1003.001.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Google Chrome GoogleUpdate.exe</li><li>Some Taskmgr.exe related activity</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-exploit-guard/attack-surface-reduction-exploit-guard?WT.mc_id=twitter](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-exploit-guard/attack-surface-reduction-exploit-guard?WT.mc_id=twitter)</li></ul>  |
| **Author**               | Markus Neis |


## Detection Rules

### Sigma rule

```
title: LSASS Access Detected via Attack Surface Reduction
id: a0a278fe-2c0e-4de2-ac3c-c68b08a9ba98
description: Detects Access to LSASS Process
status: experimental
references:
    - https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-exploit-guard/attack-surface-reduction-exploit-guard?WT.mc_id=twitter
author: Markus Neis
date: 2018/08/26
tags:
    - attack.credential_access
    - attack.t1003          # an old one
# Defender Attack Surface Reduction
    - attack.t1003.001
logsource:
    product: windows_defender
    definition: 'Requirements:Enabled Block credential stealing from the Windows local security authority subsystem (lsass.exe) from Attack Surface Reduction (GUID: 9e6c4e1f-7d60-472f-ba1a-a39ef669e4b2)'
detection:
    selection:
        EventID: 1121
        Path: '*\lsass.exe'
    condition: selection
falsepositives:
    - Google Chrome GoogleUpdate.exe
    - Some Taskmgr.exe related activity
level: high

```





### powershell
    
```
Get-WinEvent | where {($_.ID -eq "1121" -and $_.message -match "Path.*.*\\\\lsass.exe") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.event_id:"1121" AND winlog.event_data.Path.keyword:*\\\\lsass.exe)
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/a0a278fe-2c0e-4de2-ac3c-c68b08a9ba98 <<EOF\n{\n  "metadata": {\n    "title": "LSASS Access Detected via Attack Surface Reduction",\n    "description": "Detects Access to LSASS Process",\n    "tags": [\n      "attack.credential_access",\n      "attack.t1003",\n      "attack.t1003.001"\n    ],\n    "query": "(winlog.event_id:\\"1121\\" AND winlog.event_data.Path.keyword:*\\\\\\\\lsass.exe)"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.event_id:\\"1121\\" AND winlog.event_data.Path.keyword:*\\\\\\\\lsass.exe)",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'LSASS Access Detected via Attack Surface Reduction\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
(EventID:"1121" AND Path.keyword:*\\\\lsass.exe)
```


### splunk
    
```
(EventCode="1121" Path="*\\\\lsass.exe")
```


### logpoint
    
```
(event_id="1121" Path="*\\\\lsass.exe")
```


### grep
    
```
grep -P '^(?:.*(?=.*1121)(?=.*.*\\lsass\\.exe))'
```



