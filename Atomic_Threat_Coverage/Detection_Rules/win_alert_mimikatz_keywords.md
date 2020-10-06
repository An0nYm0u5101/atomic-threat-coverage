| Title                    | Mimikatz Use       |
|:-------------------------|:------------------|
| **Description**          | This method detects mimikatz keywords in different Eventlogs (some of them only appear in older Mimikatz version that are however still used by different threat groups) |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0008: Lateral Movement](https://attack.mitre.org/tactics/TA0008)</li><li>[TA0006: Credential Access](https://attack.mitre.org/tactics/TA0006)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1003: OS Credential Dumping](https://attack.mitre.org/techniques/T1003)</li><li>[T1003.002: Security Account Manager](https://attack.mitre.org/techniques/T1003/002)</li><li>[T1003.004: LSA Secrets](https://attack.mitre.org/techniques/T1003/004)</li><li>[T1003.001: LSASS Memory](https://attack.mitre.org/techniques/T1003/001)</li><li>[T1003.006: DCSync](https://attack.mitre.org/techniques/T1003/006)</li></ul>  |
| **Data Needed**          |  There is no documented Data Needed for this Detection Rule yet  |
| **Trigger**              | <ul><li>[T1003: OS Credential Dumping](../Triggers/T1003.md)</li><li>[T1003.002: Security Account Manager](../Triggers/T1003.002.md)</li><li>[T1003.004: LSA Secrets](../Triggers/T1003.004.md)</li><li>[T1003.001: LSASS Memory](../Triggers/T1003.001.md)</li></ul>  |
| **Severity Level**       | critical |
| **False Positives**      | <ul><li>Naughty administrators</li><li>Penetration test</li></ul>  |
| **Development Status**   |  Development Status wasn't defined for this Detection Rule yet  |
| **References**           |  There are no documented References for this Detection Rule yet  |
| **Author**               | Florian Roth |
| Other Tags           | <ul><li>attack.s0002</li><li>car.2013-07-001</li><li>car.2019-04-004</li></ul> | 

## Detection Rules

### Sigma rule

```
title: Mimikatz Use
id: 06d71506-7beb-4f22-8888-e2e5e2ca7fd8
description: This method detects mimikatz keywords in different Eventlogs (some of them only appear in older Mimikatz version that are however still used by different threat groups)
author: Florian Roth
date: 2017/01/10
modified: 2019/10/11
tags:
    - attack.s0002
    - attack.t1003          # an old one
    - attack.lateral_movement
    - attack.credential_access
    - car.2013-07-001
    - car.2019-04-004
    - attack.t1003.002
    - attack.t1003.004
    - attack.t1003.001
    - attack.t1003.006
logsource:
    product: windows
detection:
    keywords:
        Message:
            - "* mimikatz *"
            - "* mimilib *"
            - "* <3 eo.oe *"
            - "* eo.oe.kiwi *"
            - "* privilege::debug *"
            - "* sekurlsa::logonpasswords *"
            - "* lsadump::sam *"
            - "* mimidrv.sys *"
            - "* p::d *"
            - "* s::l *"
    condition: keywords
falsepositives:
    - Naughty administrators
    - Penetration test
level: critical

```





### powershell
    
```
Get-WinEvent | where {($_.message -match ".* mimikatz .*" -or $_.message -match ".* mimilib .*" -or $_.message -match ".* <3 eo.oe .*" -or $_.message -match ".* eo.oe.kiwi .*" -or $_.message -match ".* privilege::debug .*" -or $_.message -match ".* sekurlsa::logonpasswords .*" -or $_.message -match ".* lsadump::sam .*" -or $_.message -match ".* mimidrv.sys .*" -or $_.message -match ".* p::d .*" -or $_.message -match ".* s::l .*") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
Message.keyword:(*\\ mimikatz\\ * OR *\\ mimilib\\ * OR *\\ <3\\ eo.oe\\ * OR *\\ eo.oe.kiwi\\ * OR *\\ privilege\\:\\:debug\\ * OR *\\ sekurlsa\\:\\:logonpasswords\\ * OR *\\ lsadump\\:\\:sam\\ * OR *\\ mimidrv.sys\\ * OR *\\ p\\:\\:d\\ * OR *\\ s\\:\\:l\\ *)
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/06d71506-7beb-4f22-8888-e2e5e2ca7fd8 <<EOF\n{\n  "metadata": {\n    "title": "Mimikatz Use",\n    "description": "This method detects mimikatz keywords in different Eventlogs (some of them only appear in older Mimikatz version that are however still used by different threat groups)",\n    "tags": [\n      "attack.s0002",\n      "attack.t1003",\n      "attack.lateral_movement",\n      "attack.credential_access",\n      "car.2013-07-001",\n      "car.2019-04-004",\n      "attack.t1003.002",\n      "attack.t1003.004",\n      "attack.t1003.001",\n      "attack.t1003.006"\n    ],\n    "query": "Message.keyword:(*\\\\ mimikatz\\\\ * OR *\\\\ mimilib\\\\ * OR *\\\\ <3\\\\ eo.oe\\\\ * OR *\\\\ eo.oe.kiwi\\\\ * OR *\\\\ privilege\\\\:\\\\:debug\\\\ * OR *\\\\ sekurlsa\\\\:\\\\:logonpasswords\\\\ * OR *\\\\ lsadump\\\\:\\\\:sam\\\\ * OR *\\\\ mimidrv.sys\\\\ * OR *\\\\ p\\\\:\\\\:d\\\\ * OR *\\\\ s\\\\:\\\\:l\\\\ *)"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "Message.keyword:(*\\\\ mimikatz\\\\ * OR *\\\\ mimilib\\\\ * OR *\\\\ <3\\\\ eo.oe\\\\ * OR *\\\\ eo.oe.kiwi\\\\ * OR *\\\\ privilege\\\\:\\\\:debug\\\\ * OR *\\\\ sekurlsa\\\\:\\\\:logonpasswords\\\\ * OR *\\\\ lsadump\\\\:\\\\:sam\\\\ * OR *\\\\ mimidrv.sys\\\\ * OR *\\\\ p\\\\:\\\\:d\\\\ * OR *\\\\ s\\\\:\\\\:l\\\\ *)",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Mimikatz Use\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
Message.keyword:(* mimikatz * * mimilib * * <3 eo.oe * * eo.oe.kiwi * * privilege\\:\\:debug * * sekurlsa\\:\\:logonpasswords * * lsadump\\:\\:sam * * mimidrv.sys * * p\\:\\:d * * s\\:\\:l *)
```


### splunk
    
```
(Message="* mimikatz *" OR Message="* mimilib *" OR Message="* <3 eo.oe *" OR Message="* eo.oe.kiwi *" OR Message="* privilege::debug *" OR Message="* sekurlsa::logonpasswords *" OR Message="* lsadump::sam *" OR Message="* mimidrv.sys *" OR Message="* p::d *" OR Message="* s::l *")
```


### logpoint
    
```
Message IN ["* mimikatz *", "* mimilib *", "* <3 eo.oe *", "* eo.oe.kiwi *", "* privilege::debug *", "* sekurlsa::logonpasswords *", "* lsadump::sam *", "* mimidrv.sys *", "* p::d *", "* s::l *"]
```


### grep
    
```
grep -P '^(?:.*.* mimikatz .*|.*.* mimilib .*|.*.* <3 eo\\.oe .*|.*.* eo\\.oe\\.kiwi .*|.*.* privilege::debug .*|.*.* sekurlsa::logonpasswords .*|.*.* lsadump::sam .*|.*.* mimidrv\\.sys .*|.*.* p::d .*|.*.* s::l .*)'
```



