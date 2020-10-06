| Title                    | CobaltStrike Process Injection       |
|:-------------------------|:------------------|
| **Description**          | Detects a possible remote threat creation with certain characteristics which are typical for Cobalt Strike beacons |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1055: Process Injection](https://attack.mitre.org/techniques/T1055)</li><li>[T1055.001: Dynamic-link Library Injection](https://attack.mitre.org/techniques/T1055/001)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0012_8_windows_sysmon_CreateRemoteThread](../Data_Needed/DN_0012_8_windows_sysmon_CreateRemoteThread.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1055: Process Injection](../Triggers/T1055.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://medium.com/@olafhartong/cobalt-strike-remote-threads-detection-206372d11d0f](https://medium.com/@olafhartong/cobalt-strike-remote-threads-detection-206372d11d0f)</li><li>[https://blog.cobaltstrike.com/2018/04/09/cobalt-strike-3-11-the-snake-that-eats-its-tail/](https://blog.cobaltstrike.com/2018/04/09/cobalt-strike-3-11-the-snake-that-eats-its-tail/)</li></ul>  |
| **Author**               | Olaf Hartong, Florian Roth, Aleksey Potapov, oscd.community |


## Detection Rules

### Sigma rule

```
title: CobaltStrike Process Injection
id: 6309645e-122d-4c5b-bb2b-22e4f9c2fa42
description: Detects a possible remote threat creation with certain characteristics which are typical for Cobalt Strike beacons
references:
    - https://medium.com/@olafhartong/cobalt-strike-remote-threads-detection-206372d11d0f
    - https://blog.cobaltstrike.com/2018/04/09/cobalt-strike-3-11-the-snake-that-eats-its-tail/
tags:
    - attack.defense_evasion
    - attack.t1055          # an old one
    - attack.t1055.001
status: experimental
author: Olaf Hartong, Florian Roth, Aleksey Potapov, oscd.community
date: 2018/11/30
modified: 2020/08/28
logsource:
    product: windows
    service: sysmon
detection:
    selection:
        EventID: 8
        TargetProcessAddress|endswith: 
            - '0B80'
            - '0C7C'
            - '0C88'
    condition: selection
falsepositives:
    - unknown
level: high


```





### powershell
    
```
Get-WinEvent -LogName Microsoft-Windows-Sysmon/Operational | where {($_.ID -eq "8" -and ($_.message -match "TargetProcessAddress.*.*0B80" -or $_.message -match "TargetProcessAddress.*.*0C7C" -or $_.message -match "TargetProcessAddress.*.*0C88")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.channel:"Microsoft\\-Windows\\-Sysmon\\/Operational" AND winlog.event_id:"8" AND TargetProcessAddress.keyword:(*0B80 OR *0C7C OR *0C88))
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/6309645e-122d-4c5b-bb2b-22e4f9c2fa42 <<EOF\n{\n  "metadata": {\n    "title": "CobaltStrike Process Injection",\n    "description": "Detects a possible remote threat creation with certain characteristics which are typical for Cobalt Strike beacons",\n    "tags": [\n      "attack.defense_evasion",\n      "attack.t1055",\n      "attack.t1055.001"\n    ],\n    "query": "(winlog.channel:\\"Microsoft\\\\-Windows\\\\-Sysmon\\\\/Operational\\" AND winlog.event_id:\\"8\\" AND TargetProcessAddress.keyword:(*0B80 OR *0C7C OR *0C88))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.channel:\\"Microsoft\\\\-Windows\\\\-Sysmon\\\\/Operational\\" AND winlog.event_id:\\"8\\" AND TargetProcessAddress.keyword:(*0B80 OR *0C7C OR *0C88))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'CobaltStrike Process Injection\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
(EventID:"8" AND TargetProcessAddress.keyword:(*0B80 *0C7C *0C88))
```


### splunk
    
```
(source="WinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode="8" (TargetProcessAddress="*0B80" OR TargetProcessAddress="*0C7C" OR TargetProcessAddress="*0C88"))
```


### logpoint
    
```
(event_id="8" TargetProcessAddress IN ["*0B80", "*0C7C", "*0C88"])
```


### grep
    
```
grep -P '^(?:.*(?=.*8)(?=.*(?:.*.*0B80|.*.*0C7C|.*.*0C88)))'
```



