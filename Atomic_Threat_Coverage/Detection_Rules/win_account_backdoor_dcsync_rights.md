| Title                    | Powerview Add-DomainObjectAcl DCSync AD Extend Right       |
|:-------------------------|:------------------|
| **Description**          | backdooring domain object to grant the rights associated with DCSync to a regular user or machine account using Powerview\Add-DomainObjectAcl DCSync Extended Right cmdlet, will allow to re-obtain the pwd hashes of any user/computer |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0006: Credential Access](https://attack.mitre.org/tactics/TA0006)</li><li>[TA0003: Persistence](https://attack.mitre.org/tactics/TA0003)</li></ul>  |
| **ATT&amp;CK Technique** |  This Detection Rule wasn't mapped to ATT&amp;CK Technique yet  |
| **Data Needed**          | <ul><li>[DN_0026_5136_windows_directory_service_object_was_modified](../Data_Needed/DN_0026_5136_windows_directory_service_object_was_modified.md)</li></ul>  |
| **Trigger**              |  There is no documented Trigger for this Detection Rule yet  |
| **Severity Level**       | critical |
| **False Positives**      | <ul><li>New Domain Controller computer account, check user SIDs witin the value attribute of event 5136 and verify if it's a regular user or DC computer account.</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://twitter.com/menasec1/status/1111556090137903104](https://twitter.com/menasec1/status/1111556090137903104)</li><li>[https://www.specterops.io/assets/resources/an_ace_up_the_sleeve.pdf](https://www.specterops.io/assets/resources/an_ace_up_the_sleeve.pdf)</li></ul>  |
| **Author**               | Samir Bousseaden; Roberto Rodriguez @Cyb3rWard0g; oscd.community |


## Detection Rules

### Sigma rule

```
title: Powerview Add-DomainObjectAcl DCSync AD Extend Right
id: 2c99737c-585d-4431-b61a-c911d86ff32f
description: backdooring domain object to grant the rights associated with DCSync to a regular user or machine account using Powerview\Add-DomainObjectAcl DCSync
    Extended Right cmdlet, will allow to re-obtain the pwd hashes of any user/computer
status: experimental
date: 2019/04/03
author: Samir Bousseaden; Roberto Rodriguez @Cyb3rWard0g; oscd.community
references:
    - https://twitter.com/menasec1/status/1111556090137903104
    - https://www.specterops.io/assets/resources/an_ace_up_the_sleeve.pdf
tags:
    - attack.credential_access
    - attack.persistence
logsource:
    product: windows
    service: security
detection:
    selection:
        EventID: 5136
        LDAPDisplayName: 'ntSecurityDescriptor'
        Value|contains: 
         - '1131f6ad-9c07-11d1-f79f-00c04fc2dcd2'
         - '1131f6aa-9c07-11d1-f79f-00c04fc2dcd2'
         - '89e95b76-444d-4c62-991a-0facbeda640c'
    condition: selection
falsepositives:
    - New Domain Controller computer account, check user SIDs witin the value attribute of event 5136 and verify if it's a regular user or DC computer account.
level: critical

```





### powershell
    
```
Get-WinEvent -LogName Security | where {($_.ID -eq "5136" -and $_.message -match "LDAPDisplayName.*ntSecurityDescriptor" -and ($_.message -match "Value.*.*1131f6ad-9c07-11d1-f79f-00c04fc2dcd2.*" -or $_.message -match "Value.*.*1131f6aa-9c07-11d1-f79f-00c04fc2dcd2.*" -or $_.message -match "Value.*.*89e95b76-444d-4c62-991a-0facbeda640c.*")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.channel:"Security" AND winlog.event_id:"5136" AND LDAPDisplayName:"ntSecurityDescriptor" AND Value.keyword:(*1131f6ad\-9c07\-11d1\-f79f\-00c04fc2dcd2* OR *1131f6aa\-9c07\-11d1\-f79f\-00c04fc2dcd2* OR *89e95b76\-444d\-4c62\-991a\-0facbeda640c*))
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/2c99737c-585d-4431-b61a-c911d86ff32f <<EOF
{
  "metadata": {
    "title": "Powerview Add-DomainObjectAcl DCSync AD Extend Right",
    "description": "backdooring domain object to grant the rights associated with DCSync to a regular user or machine account using Powerview\\Add-DomainObjectAcl DCSync Extended Right cmdlet, will allow to re-obtain the pwd hashes of any user/computer",
    "tags": [
      "attack.credential_access",
      "attack.persistence"
    ],
    "query": "(winlog.channel:\"Security\" AND winlog.event_id:\"5136\" AND LDAPDisplayName:\"ntSecurityDescriptor\" AND Value.keyword:(*1131f6ad\\-9c07\\-11d1\\-f79f\\-00c04fc2dcd2* OR *1131f6aa\\-9c07\\-11d1\\-f79f\\-00c04fc2dcd2* OR *89e95b76\\-444d\\-4c62\\-991a\\-0facbeda640c*))"
  },
  "trigger": {
    "schedule": {
      "interval": "30m"
    }
  },
  "input": {
    "search": {
      "request": {
        "body": {
          "size": 0,
          "query": {
            "bool": {
              "must": [
                {
                  "query_string": {
                    "query": "(winlog.channel:\"Security\" AND winlog.event_id:\"5136\" AND LDAPDisplayName:\"ntSecurityDescriptor\" AND Value.keyword:(*1131f6ad\\-9c07\\-11d1\\-f79f\\-00c04fc2dcd2* OR *1131f6aa\\-9c07\\-11d1\\-f79f\\-00c04fc2dcd2* OR *89e95b76\\-444d\\-4c62\\-991a\\-0facbeda640c*))",
                    "analyze_wildcard": true
                  }
                }
              ],
              "filter": {
                "range": {
                  "timestamp": {
                    "gte": "now-30m/m"
                  }
                }
              }
            }
          }
        },
        "indices": [
          "winlogbeat-*"
        ]
      }
    }
  },
  "condition": {
    "compare": {
      "ctx.payload.hits.total": {
        "not_eq": 0
      }
    }
  },
  "actions": {
    "send_email": {
      "email": {
        "to": "root@localhost",
        "subject": "Sigma Rule 'Powerview Add-DomainObjectAcl DCSync AD Extend Right'",
        "body": "Hits:\n{{#ctx.payload.hits.hits}}{{_source}}\n================================================================================\n{{/ctx.payload.hits.hits}}",
        "attachments": {
          "data.json": {
            "data": {
              "format": "json"
            }
          }
        }
      }
    }
  }
}
EOF

```


### graylog
    
```
(EventID:"5136" AND LDAPDisplayName:"ntSecurityDescriptor" AND Value.keyword:(*1131f6ad\-9c07\-11d1\-f79f\-00c04fc2dcd2* *1131f6aa\-9c07\-11d1\-f79f\-00c04fc2dcd2* *89e95b76\-444d\-4c62\-991a\-0facbeda640c*))
```


### splunk
    
```
(source="WinEventLog:Security" EventCode="5136" LDAPDisplayName="ntSecurityDescriptor" (Value="*1131f6ad-9c07-11d1-f79f-00c04fc2dcd2*" OR Value="*1131f6aa-9c07-11d1-f79f-00c04fc2dcd2*" OR Value="*89e95b76-444d-4c62-991a-0facbeda640c*"))
```


### logpoint
    
```
(event_source="Microsoft-Windows-Security-Auditing" event_id="5136" LDAPDisplayName="ntSecurityDescriptor" Value IN ["*1131f6ad-9c07-11d1-f79f-00c04fc2dcd2*", "*1131f6aa-9c07-11d1-f79f-00c04fc2dcd2*", "*89e95b76-444d-4c62-991a-0facbeda640c*"])
```


### grep
    
```
grep -P '^(?:.*(?=.*5136)(?=.*ntSecurityDescriptor)(?=.*(?:.*.*1131f6ad-9c07-11d1-f79f-00c04fc2dcd2.*|.*.*1131f6aa-9c07-11d1-f79f-00c04fc2dcd2.*|.*.*89e95b76-444d-4c62-991a-0facbeda640c.*)))'
```



