| Title                    | DPAPI Domain Backup Key Extraction       |
|:-------------------------|:------------------|
| **Description**          | Detects tools extracting LSA secret DPAPI domain backup key from Domain Controllers |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0006: Credential Access](https://attack.mitre.org/tactics/TA0006)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1003: OS Credential Dumping](https://attack.mitre.org/techniques/T1003)</li><li>[T1003.004: LSA Secrets](https://attack.mitre.org/techniques/T1003/004)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0030_4662_operation_was_performed_on_an_object](../Data_Needed/DN_0030_4662_operation_was_performed_on_an_object.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1003: OS Credential Dumping](../Triggers/T1003.md)</li><li>[T1003.004: LSA Secrets](../Triggers/T1003.004.md)</li></ul>  |
| **Severity Level**       | critical |
| **False Positives**      | <ul><li>Unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://github.com/Cyb3rWard0g/ThreatHunter-Playbook/tree/master/playbooks/windows/06_credential_access/T1003_credential_dumping/domain_dpapi_backupkey_extraction.md](https://github.com/Cyb3rWard0g/ThreatHunter-Playbook/tree/master/playbooks/windows/06_credential_access/T1003_credential_dumping/domain_dpapi_backupkey_extraction.md)</li></ul>  |
| **Author**               | Roberto Rodriguez @Cyb3rWard0g |


## Detection Rules

### Sigma rule

```
title: DPAPI Domain Backup Key Extraction
id: 4ac1f50b-3bd0-4968-902d-868b4647937e
description: Detects tools extracting LSA secret DPAPI domain backup key from Domain Controllers
status: experimental
date: 2019/06/20
author: Roberto Rodriguez @Cyb3rWard0g
references:
    - https://github.com/Cyb3rWard0g/ThreatHunter-Playbook/tree/master/playbooks/windows/06_credential_access/T1003_credential_dumping/domain_dpapi_backupkey_extraction.md
tags:
    - attack.credential_access
    - attack.t1003          # an old one
    - attack.t1003.004
logsource:
    product: windows
    service: security
detection:
    selection:
        EventID: 4662
        ObjectType: 'SecretObject'
        AccessMask: '0x2'
        ObjectName: 'BCKUPKEY'
    condition: selection
falsepositives:
    - Unknown
level: critical

```





### powershell
    
```
Get-WinEvent -LogName Security | where {($_.ID -eq "4662" -and $_.message -match "ObjectType.*SecretObject" -and $_.message -match "AccessMask.*0x2" -and $_.message -match "ObjectName.*BCKUPKEY") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.channel:"Security" AND winlog.event_id:"4662" AND winlog.event_data.ObjectType:"SecretObject" AND winlog.event_data.AccessMask:"0x2" AND winlog.event_data.ObjectName:"BCKUPKEY")
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/4ac1f50b-3bd0-4968-902d-868b4647937e <<EOF\n{\n  "metadata": {\n    "title": "DPAPI Domain Backup Key Extraction",\n    "description": "Detects tools extracting LSA secret DPAPI domain backup key from Domain Controllers",\n    "tags": [\n      "attack.credential_access",\n      "attack.t1003",\n      "attack.t1003.004"\n    ],\n    "query": "(winlog.channel:\\"Security\\" AND winlog.event_id:\\"4662\\" AND winlog.event_data.ObjectType:\\"SecretObject\\" AND winlog.event_data.AccessMask:\\"0x2\\" AND winlog.event_data.ObjectName:\\"BCKUPKEY\\")"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.channel:\\"Security\\" AND winlog.event_id:\\"4662\\" AND winlog.event_data.ObjectType:\\"SecretObject\\" AND winlog.event_data.AccessMask:\\"0x2\\" AND winlog.event_data.ObjectName:\\"BCKUPKEY\\")",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'DPAPI Domain Backup Key Extraction\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
(EventID:"4662" AND ObjectType:"SecretObject" AND AccessMask:"0x2" AND ObjectName:"BCKUPKEY")
```


### splunk
    
```
(source="WinEventLog:Security" EventCode="4662" ObjectType="SecretObject" AccessMask="0x2" ObjectName="BCKUPKEY")
```


### logpoint
    
```
(event_source="Microsoft-Windows-Security-Auditing" event_id="4662" ObjectType="SecretObject" AccessMask="0x2" ObjectName="BCKUPKEY")
```


### grep
    
```
grep -P '^(?:.*(?=.*4662)(?=.*SecretObject)(?=.*0x2)(?=.*BCKUPKEY))'
```



