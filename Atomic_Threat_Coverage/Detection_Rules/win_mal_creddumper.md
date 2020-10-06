| Title                    | Credential Dumping Tools Service Execution       |
|:-------------------------|:------------------|
| **Description**          | Detects well-known credential dumping tools execution via service execution events |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0006: Credential Access](https://attack.mitre.org/tactics/TA0006)</li><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1003: OS Credential Dumping](https://attack.mitre.org/techniques/T1003)</li><li>[T1003.001: LSASS Memory](https://attack.mitre.org/techniques/T1003/001)</li><li>[T1003.002: Security Account Manager](https://attack.mitre.org/techniques/T1003/002)</li><li>[T1003.004: LSA Secrets](https://attack.mitre.org/techniques/T1003/004)</li><li>[T1003.005: Cached Domain Credentials](https://attack.mitre.org/techniques/T1003/005)</li><li>[T1003.006: DCSync](https://attack.mitre.org/techniques/T1003/006)</li><li>[T1035: Service Execution](https://attack.mitre.org/techniques/T1035)</li><li>[T1569.002: Service Execution](https://attack.mitre.org/techniques/T1569/002)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0005_7045_windows_service_insatalled](../Data_Needed/DN_0005_7045_windows_service_insatalled.md)</li><li>[DN_0010_6_windows_sysmon_driver_loaded](../Data_Needed/DN_0010_6_windows_sysmon_driver_loaded.md)</li><li>[DN_0063_4697_service_was_installed_in_the_system](../Data_Needed/DN_0063_4697_service_was_installed_in_the_system.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1003: OS Credential Dumping](../Triggers/T1003.md)</li><li>[T1003.001: LSASS Memory](../Triggers/T1003.001.md)</li><li>[T1003.002: Security Account Manager](../Triggers/T1003.002.md)</li><li>[T1003.004: LSA Secrets](../Triggers/T1003.004.md)</li><li>[T1569.002: Service Execution](../Triggers/T1569.002.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Legitimate Administrator using credential dumping tool for password recovery</li></ul>  |
| **Development Status**   |  Development Status wasn't defined for this Detection Rule yet  |
| **References**           | <ul><li>[https://www.slideshare.net/heirhabarov/hunting-for-credentials-dumping-in-windows-environment](https://www.slideshare.net/heirhabarov/hunting-for-credentials-dumping-in-windows-environment)</li></ul>  |
| **Author**               | Florian Roth, Teymur Kheirkhabarov, Daniil Yugoslavskiy, oscd.community |
| Other Tags           | <ul><li>attack.s0005</li></ul> | 

## Detection Rules

### Sigma rule

```
---
action: global
title: Credential Dumping Tools Service Execution
description: Detects well-known credential dumping tools execution via service execution events
author: Florian Roth, Teymur Kheirkhabarov, Daniil Yugoslavskiy, oscd.community
id: 4976aa50-8f41-45c6-8b15-ab3fc10e79ed
date: 2017/03/05
modified: 2020/08/23
references:
    - https://www.slideshare.net/heirhabarov/hunting-for-credentials-dumping-in-windows-environment
tags:
    - attack.credential_access
    - attack.execution
    - attack.t1003          # an old one
    - attack.t1003.001
    - attack.t1003.002
    - attack.t1003.004
    - attack.t1003.005
    - attack.t1003.006
    - attack.t1035          # an old one
    - attack.t1569.002
    - attack.s0005
detection:
    selection_1:
        - ServiceName|contains:
            - 'fgexec'
            - 'wceservice'
            - 'wce service'
            - 'pwdump'
            - 'gsecdump'
            - 'cachedump'
            - 'mimikatz'
            - 'mimidrv'
        - ImagePath|contains:
            - 'fgexec'
            - 'dumpsvc'
            - 'cachedump'
            - 'mimidrv'
            - 'gsecdump'
            - 'servpw'
            - 'pwdump'
        - ImagePath|re: '((\\\\.*\\.*|.*\\)([{]?[0-9A-Fa-f]{8}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{12}[}])?\.(exe|scr|cpl|bat|js|cmd|vbs).*)'
    condition: selection and selection_1
falsepositives:
    - Legitimate Administrator using credential dumping tool for password recovery
level: high
---
logsource:
    product: windows
    service: system
detection:
    selection:
        EventID: 7045
---
logsource:
    product: windows
    service: sysmon
detection:
    selection:
        EventID: 6
---
logsource:
    product: windows
    service: security
detection:
    selection:
        EventID: 4697

```





### powershell
    
```

```


### es-qs
    
```
(winlog.event_id:"7045" AND (winlog.event_data.ServiceName.keyword:(*fgexec* OR *wceservice* OR *wce\\ service* OR *pwdump* OR *gsecdump* OR *cachedump* OR *mimikatz* OR *mimidrv*) OR winlog.event_data.ImagePath.keyword:(*fgexec* OR *dumpsvc* OR *cachedump* OR *mimidrv* OR *gsecdump* OR *servpw* OR *pwdump*) OR winlog.event_data.ImagePath:/((\\\\\\\\.*\\\\.*|.*\\\\)([{]?[0-9A-Fa-f]{8}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{12}[}])?\\.(exe|scr|cpl|bat|js|cmd|vbs).*)/))\n(winlog.channel:"Microsoft\\-Windows\\-Sysmon\\/Operational" AND winlog.event_id:"6" AND (winlog.event_data.ServiceName.keyword:(*fgexec* OR *wceservice* OR *wce\\ service* OR *pwdump* OR *gsecdump* OR *cachedump* OR *mimikatz* OR *mimidrv*) OR winlog.event_data.ImagePath.keyword:(*fgexec* OR *dumpsvc* OR *cachedump* OR *mimidrv* OR *gsecdump* OR *servpw* OR *pwdump*) OR winlog.event_data.ImagePath:/((\\\\\\\\.*\\\\.*|.*\\\\)([{]?[0-9A-Fa-f]{8}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{12}[}])?\\.(exe|scr|cpl|bat|js|cmd|vbs).*)/))\n(winlog.channel:"Security" AND winlog.event_id:"4697" AND (winlog.event_data.ServiceName.keyword:(*fgexec* OR *wceservice* OR *wce\\ service* OR *pwdump* OR *gsecdump* OR *cachedump* OR *mimikatz* OR *mimidrv*) OR winlog.event_data.ImagePath.keyword:(*fgexec* OR *dumpsvc* OR *cachedump* OR *mimidrv* OR *gsecdump* OR *servpw* OR *pwdump*) OR winlog.event_data.ImagePath:/((\\\\\\\\.*\\\\.*|.*\\\\)([{]?[0-9A-Fa-f]{8}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{12}[}])?\\.(exe|scr|cpl|bat|js|cmd|vbs).*)/))
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/4976aa50-8f41-45c6-8b15-ab3fc10e79ed <<EOF\n{\n  "metadata": {\n    "title": "Credential Dumping Tools Service Execution",\n    "description": "Detects well-known credential dumping tools execution via service execution events",\n    "tags": [\n      "attack.credential_access",\n      "attack.execution",\n      "attack.t1003",\n      "attack.t1003.001",\n      "attack.t1003.002",\n      "attack.t1003.004",\n      "attack.t1003.005",\n      "attack.t1003.006",\n      "attack.t1035",\n      "attack.t1569.002",\n      "attack.s0005"\n    ],\n    "query": "(winlog.event_id:\\"7045\\" AND (winlog.event_data.ServiceName.keyword:(*fgexec* OR *wceservice* OR *wce\\\\ service* OR *pwdump* OR *gsecdump* OR *cachedump* OR *mimikatz* OR *mimidrv*) OR winlog.event_data.ImagePath.keyword:(*fgexec* OR *dumpsvc* OR *cachedump* OR *mimidrv* OR *gsecdump* OR *servpw* OR *pwdump*) OR winlog.event_data.ImagePath:/((\\\\\\\\\\\\\\\\.*\\\\\\\\.*|.*\\\\\\\\)([{]?[0-9A-Fa-f]{8}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{12}[}])?\\\\.(exe|scr|cpl|bat|js|cmd|vbs).*)/))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.event_id:\\"7045\\" AND (winlog.event_data.ServiceName.keyword:(*fgexec* OR *wceservice* OR *wce\\\\ service* OR *pwdump* OR *gsecdump* OR *cachedump* OR *mimikatz* OR *mimidrv*) OR winlog.event_data.ImagePath.keyword:(*fgexec* OR *dumpsvc* OR *cachedump* OR *mimidrv* OR *gsecdump* OR *servpw* OR *pwdump*) OR winlog.event_data.ImagePath:/((\\\\\\\\\\\\\\\\.*\\\\\\\\.*|.*\\\\\\\\)([{]?[0-9A-Fa-f]{8}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{12}[}])?\\\\.(exe|scr|cpl|bat|js|cmd|vbs).*)/))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Credential Dumping Tools Service Execution\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\ncurl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/4976aa50-8f41-45c6-8b15-ab3fc10e79ed-2 <<EOF\n{\n  "metadata": {\n    "title": "Credential Dumping Tools Service Execution",\n    "description": "Detects well-known credential dumping tools execution via service execution events",\n    "tags": [\n      "attack.credential_access",\n      "attack.execution",\n      "attack.t1003",\n      "attack.t1003.001",\n      "attack.t1003.002",\n      "attack.t1003.004",\n      "attack.t1003.005",\n      "attack.t1003.006",\n      "attack.t1035",\n      "attack.t1569.002",\n      "attack.s0005"\n    ],\n    "query": "(winlog.channel:\\"Microsoft\\\\-Windows\\\\-Sysmon\\\\/Operational\\" AND winlog.event_id:\\"6\\" AND (winlog.event_data.ServiceName.keyword:(*fgexec* OR *wceservice* OR *wce\\\\ service* OR *pwdump* OR *gsecdump* OR *cachedump* OR *mimikatz* OR *mimidrv*) OR winlog.event_data.ImagePath.keyword:(*fgexec* OR *dumpsvc* OR *cachedump* OR *mimidrv* OR *gsecdump* OR *servpw* OR *pwdump*) OR winlog.event_data.ImagePath:/((\\\\\\\\\\\\\\\\.*\\\\\\\\.*|.*\\\\\\\\)([{]?[0-9A-Fa-f]{8}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{12}[}])?\\\\.(exe|scr|cpl|bat|js|cmd|vbs).*)/))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.channel:\\"Microsoft\\\\-Windows\\\\-Sysmon\\\\/Operational\\" AND winlog.event_id:\\"6\\" AND (winlog.event_data.ServiceName.keyword:(*fgexec* OR *wceservice* OR *wce\\\\ service* OR *pwdump* OR *gsecdump* OR *cachedump* OR *mimikatz* OR *mimidrv*) OR winlog.event_data.ImagePath.keyword:(*fgexec* OR *dumpsvc* OR *cachedump* OR *mimidrv* OR *gsecdump* OR *servpw* OR *pwdump*) OR winlog.event_data.ImagePath:/((\\\\\\\\\\\\\\\\.*\\\\\\\\.*|.*\\\\\\\\)([{]?[0-9A-Fa-f]{8}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{12}[}])?\\\\.(exe|scr|cpl|bat|js|cmd|vbs).*)/))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Credential Dumping Tools Service Execution\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\ncurl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/4976aa50-8f41-45c6-8b15-ab3fc10e79ed-3 <<EOF\n{\n  "metadata": {\n    "title": "Credential Dumping Tools Service Execution",\n    "description": "Detects well-known credential dumping tools execution via service execution events",\n    "tags": [\n      "attack.credential_access",\n      "attack.execution",\n      "attack.t1003",\n      "attack.t1003.001",\n      "attack.t1003.002",\n      "attack.t1003.004",\n      "attack.t1003.005",\n      "attack.t1003.006",\n      "attack.t1035",\n      "attack.t1569.002",\n      "attack.s0005"\n    ],\n    "query": "(winlog.channel:\\"Security\\" AND winlog.event_id:\\"4697\\" AND (winlog.event_data.ServiceName.keyword:(*fgexec* OR *wceservice* OR *wce\\\\ service* OR *pwdump* OR *gsecdump* OR *cachedump* OR *mimikatz* OR *mimidrv*) OR winlog.event_data.ImagePath.keyword:(*fgexec* OR *dumpsvc* OR *cachedump* OR *mimidrv* OR *gsecdump* OR *servpw* OR *pwdump*) OR winlog.event_data.ImagePath:/((\\\\\\\\\\\\\\\\.*\\\\\\\\.*|.*\\\\\\\\)([{]?[0-9A-Fa-f]{8}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{12}[}])?\\\\.(exe|scr|cpl|bat|js|cmd|vbs).*)/))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.channel:\\"Security\\" AND winlog.event_id:\\"4697\\" AND (winlog.event_data.ServiceName.keyword:(*fgexec* OR *wceservice* OR *wce\\\\ service* OR *pwdump* OR *gsecdump* OR *cachedump* OR *mimikatz* OR *mimidrv*) OR winlog.event_data.ImagePath.keyword:(*fgexec* OR *dumpsvc* OR *cachedump* OR *mimidrv* OR *gsecdump* OR *servpw* OR *pwdump*) OR winlog.event_data.ImagePath:/((\\\\\\\\\\\\\\\\.*\\\\\\\\.*|.*\\\\\\\\)([{]?[0-9A-Fa-f]{8}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{12}[}])?\\\\.(exe|scr|cpl|bat|js|cmd|vbs).*)/))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Credential Dumping Tools Service Execution\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
(EventID:"7045" AND (ServiceName.keyword:(*fgexec* *wceservice* *wce service* *pwdump* *gsecdump* *cachedump* *mimikatz* *mimidrv*) OR ImagePath.keyword:(*fgexec* *dumpsvc* *cachedump* *mimidrv* *gsecdump* *servpw* *pwdump*) OR ImagePath:/((\\\\\\\\.*\\\\.*|.*\\\\)([{]?[0-9A-Fa-f]{8}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{12}[}])?\\.(exe|scr|cpl|bat|js|cmd|vbs).*)/))\n(EventID:"6" AND (ServiceName.keyword:(*fgexec* *wceservice* *wce service* *pwdump* *gsecdump* *cachedump* *mimikatz* *mimidrv*) OR ImagePath.keyword:(*fgexec* *dumpsvc* *cachedump* *mimidrv* *gsecdump* *servpw* *pwdump*) OR ImagePath:/((\\\\\\\\.*\\\\.*|.*\\\\)([{]?[0-9A-Fa-f]{8}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{12}[}])?\\.(exe|scr|cpl|bat|js|cmd|vbs).*)/))\n(EventID:"4697" AND (ServiceName.keyword:(*fgexec* *wceservice* *wce service* *pwdump* *gsecdump* *cachedump* *mimikatz* *mimidrv*) OR ImagePath.keyword:(*fgexec* *dumpsvc* *cachedump* *mimidrv* *gsecdump* *servpw* *pwdump*) OR ImagePath:/((\\\\\\\\.*\\\\.*|.*\\\\)([{]?[0-9A-Fa-f]{8}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{12}[}])?\\.(exe|scr|cpl|bat|js|cmd|vbs).*)/))
```


### splunk
    
```

```


### logpoint
    
```

```


### grep
    
```

```



