| Title                    | Admin User Remote Logon       |
|:-------------------------|:------------------|
| **Description**          | Detect remote login by Administrator user depending on internal pattern |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0008: Lateral Movement](https://attack.mitre.org/tactics/TA0008)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1078: Valid Accounts](https://attack.mitre.org/techniques/T1078)</li><li>[T1078.001: Default Accounts](https://attack.mitre.org/techniques/T1078/001)</li><li>[T1078.002: Domain Accounts](https://attack.mitre.org/techniques/T1078/002)</li><li>[T1078.003: Local Accounts](https://attack.mitre.org/techniques/T1078/003)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0004_4624_windows_account_logon](../Data_Needed/DN_0004_4624_windows_account_logon.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1078.001: Default Accounts](../Triggers/T1078.001.md)</li></ul>  |
| **Severity Level**       | low |
| **False Positives**      | <ul><li>Legitimate administrative activity</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://car.mitre.org/wiki/CAR-2016-04-005](https://car.mitre.org/wiki/CAR-2016-04-005)</li></ul>  |
| **Author**               | juju4 |
| Other Tags           | <ul><li>car.2016-04-005</li></ul> | 

## Detection Rules

### Sigma rule

```
title: Admin User Remote Logon
id: 0f63e1ef-1eb9-4226-9d54-8927ca08520a
description: Detect remote login by Administrator user depending on internal pattern
references:
    - https://car.mitre.org/wiki/CAR-2016-04-005
tags:
    - attack.lateral_movement
    - attack.t1078          # an old one
    - attack.t1078.001
    - attack.t1078.002
    - attack.t1078.003
    - car.2016-04-005
status: experimental
author: juju4
date: 2017/10/29
modified: 2020/08/23
logsource:
    product: windows
    service: security
    definition: 'Requirements: Identifiable administrators usernames (pattern or special unique character. ex: "Admin-*"), internal policy mandating use only as secondary account'
detection:
    selection:
        EventID: 4624
        LogonType: 10
        AuthenticationPackageName: Negotiate
        AccountName: 'Admin-*'
    condition: selection
falsepositives:
    - Legitimate administrative activity
level: low

```





### powershell
    
```
Get-WinEvent -LogName Security | where {($_.ID -eq "4624" -and $_.message -match "LogonType.*10" -and $_.message -match "AuthenticationPackageName.*Negotiate" -and $_.message -match "AccountName.*Admin-.*") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.channel:"Security" AND winlog.event_id:"4624" AND winlog.event_data.LogonType:"10" AND winlog.event_data.AuthenticationPackageName:"Negotiate" AND winlog.event_data.AccountName.keyword:Admin\\-*)
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/0f63e1ef-1eb9-4226-9d54-8927ca08520a <<EOF\n{\n  "metadata": {\n    "title": "Admin User Remote Logon",\n    "description": "Detect remote login by Administrator user depending on internal pattern",\n    "tags": [\n      "attack.lateral_movement",\n      "attack.t1078",\n      "attack.t1078.001",\n      "attack.t1078.002",\n      "attack.t1078.003",\n      "car.2016-04-005"\n    ],\n    "query": "(winlog.channel:\\"Security\\" AND winlog.event_id:\\"4624\\" AND winlog.event_data.LogonType:\\"10\\" AND winlog.event_data.AuthenticationPackageName:\\"Negotiate\\" AND winlog.event_data.AccountName.keyword:Admin\\\\-*)"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.channel:\\"Security\\" AND winlog.event_id:\\"4624\\" AND winlog.event_data.LogonType:\\"10\\" AND winlog.event_data.AuthenticationPackageName:\\"Negotiate\\" AND winlog.event_data.AccountName.keyword:Admin\\\\-*)",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Admin User Remote Logon\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
(EventID:"4624" AND LogonType:"10" AND AuthenticationPackageName:"Negotiate" AND AccountName.keyword:Admin\\-*)
```


### splunk
    
```
(source="WinEventLog:Security" EventCode="4624" LogonType="10" AuthenticationPackageName="Negotiate" AccountName="Admin-*")
```


### logpoint
    
```
(event_source="Microsoft-Windows-Security-Auditing" event_id="4624" logon_type="10" AuthenticationPackageName="Negotiate" AccountName="Admin-*")
```


### grep
    
```
grep -P '^(?:.*(?=.*4624)(?=.*10)(?=.*Negotiate)(?=.*Admin-.*))'
```



