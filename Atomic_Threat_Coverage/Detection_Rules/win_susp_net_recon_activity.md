| Title                    | Reconnaissance Activity       |
|:-------------------------|:------------------|
| **Description**          | Detects activity as "net user administrator /domain" and "net group domain admins /domain" |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0007: Discovery](https://attack.mitre.org/tactics/TA0007)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1087: Account Discovery](https://attack.mitre.org/techniques/T1087)</li><li>[T1087.002: Domain Account](https://attack.mitre.org/techniques/T1087/002)</li><li>[T1069: Permission Groups Discovery](https://attack.mitre.org/techniques/T1069)</li><li>[T1069.002: Domain Groups](https://attack.mitre.org/techniques/T1069/002)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0029_4661_handle_to_an_object_was_requested](../Data_Needed/DN_0029_4661_handle_to_an_object_was_requested.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1087.002: Domain Account](../Triggers/T1087.002.md)</li><li>[T1069.002: Domain Groups](../Triggers/T1069.002.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Administrator activity</li><li>Penetration tests</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://findingbad.blogspot.de/2017/01/hunting-what-does-it-look-like.html](https://findingbad.blogspot.de/2017/01/hunting-what-does-it-look-like.html)</li></ul>  |
| **Author**               | Florian Roth (rule), Jack Croock (method) |
| Other Tags           | <ul><li>attack.s0039</li></ul> | 

## Detection Rules

### Sigma rule

```
title: Reconnaissance Activity
id: 968eef52-9cff-4454-8992-1e74b9cbad6c
status: experimental
description: Detects activity as "net user administrator /domain" and "net group domain admins /domain"
references:
    - https://findingbad.blogspot.de/2017/01/hunting-what-does-it-look-like.html
author: Florian Roth (rule), Jack Croock (method)
date: 2017/03/07
modified: 2020/08/23
tags:
    - attack.discovery
    - attack.t1087           # an old one
    - attack.t1087.002
    - attack.t1069           # an old one
    - attack.t1069.002
    - attack.s0039
logsource:
    product: windows
    service: security
    definition: The volume of Event ID 4661 is high on Domain Controllers and therefore "Audit SAM" and "Audit Kernel Object" advanced audit policy settings are not configured in the recommendations for server systems
detection:
    selection:
        - EventID: 4661
          ObjectType: 'SAM_USER'
          ObjectName: 'S-1-5-21-*-500'
          AccessMask: '0x2d'
        - EventID: 4661
          ObjectType: 'SAM_GROUP'
          ObjectName: 'S-1-5-21-*-512'
          AccessMask: '0x2d'
    condition: selection
falsepositives:
    - Administrator activity
    - Penetration tests
level: high

```





### powershell
    
```
Get-WinEvent -LogName Security | where {($_.ID -eq "4661" -and $_.message -match "AccessMask.*0x2d" -and (($_.message -match "ObjectType.*SAM_USER" -and $_.message -match "ObjectName.*S-1-5-21-.*-500") -or ($_.message -match "ObjectType.*SAM_GROUP" -and $_.message -match "ObjectName.*S-1-5-21-.*-512"))) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.channel:"Security" AND winlog.event_id:"4661" AND winlog.event_data.AccessMask:"0x2d" AND ((winlog.event_data.ObjectType:"SAM_USER" AND winlog.event_data.ObjectName.keyword:S\\-1\\-5\\-21\\-*\\-500) OR (winlog.event_data.ObjectType:"SAM_GROUP" AND winlog.event_data.ObjectName.keyword:S\\-1\\-5\\-21\\-*\\-512)))
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/968eef52-9cff-4454-8992-1e74b9cbad6c <<EOF\n{\n  "metadata": {\n    "title": "Reconnaissance Activity",\n    "description": "Detects activity as \\"net user administrator /domain\\" and \\"net group domain admins /domain\\"",\n    "tags": [\n      "attack.discovery",\n      "attack.t1087",\n      "attack.t1087.002",\n      "attack.t1069",\n      "attack.t1069.002",\n      "attack.s0039"\n    ],\n    "query": "(winlog.channel:\\"Security\\" AND winlog.event_id:\\"4661\\" AND winlog.event_data.AccessMask:\\"0x2d\\" AND ((winlog.event_data.ObjectType:\\"SAM_USER\\" AND winlog.event_data.ObjectName.keyword:S\\\\-1\\\\-5\\\\-21\\\\-*\\\\-500) OR (winlog.event_data.ObjectType:\\"SAM_GROUP\\" AND winlog.event_data.ObjectName.keyword:S\\\\-1\\\\-5\\\\-21\\\\-*\\\\-512)))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.channel:\\"Security\\" AND winlog.event_id:\\"4661\\" AND winlog.event_data.AccessMask:\\"0x2d\\" AND ((winlog.event_data.ObjectType:\\"SAM_USER\\" AND winlog.event_data.ObjectName.keyword:S\\\\-1\\\\-5\\\\-21\\\\-*\\\\-500) OR (winlog.event_data.ObjectType:\\"SAM_GROUP\\" AND winlog.event_data.ObjectName.keyword:S\\\\-1\\\\-5\\\\-21\\\\-*\\\\-512)))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Reconnaissance Activity\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
(EventID:"4661" AND AccessMask:"0x2d" AND ((ObjectType:"SAM_USER" AND ObjectName.keyword:S\\-1\\-5\\-21\\-*\\-500) OR (ObjectType:"SAM_GROUP" AND ObjectName.keyword:S\\-1\\-5\\-21\\-*\\-512)))
```


### splunk
    
```
(source="WinEventLog:Security" EventCode="4661" AccessMask="0x2d" ((ObjectType="SAM_USER" ObjectName="S-1-5-21-*-500") OR (ObjectType="SAM_GROUP" ObjectName="S-1-5-21-*-512")))
```


### logpoint
    
```
(event_source="Microsoft-Windows-Security-Auditing" event_id="4661" AccessMask="0x2d" ((ObjectType="SAM_USER" ObjectName="S-1-5-21-*-500") OR (ObjectType="SAM_GROUP" ObjectName="S-1-5-21-*-512")))
```


### grep
    
```
grep -P '^(?:.*(?=.*4661)(?=.*0x2d)(?=.*(?:.*(?:.*(?:.*(?=.*SAM_USER)(?=.*S-1-5-21-.*-500))|.*(?:.*(?=.*SAM_GROUP)(?=.*S-1-5-21-.*-512))))))'
```



