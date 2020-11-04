| Title                    | Mimikatz Use       |
|:-------------------------|:------------------|
| **Description**          | This method detects mimikatz keywords in different Eventlogs (some of them only appear in older Mimikatz version that are however still used by different threat groups) |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0008: Lateral Movement](https://attack.mitre.org/tactics/TA0008)</li><li>[TA0006: Credential Access](https://attack.mitre.org/tactics/TA0006)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1003: OS Credential Dumping](https://attack.mitre.org/techniques/T1003)</li></ul>  |
| **Data Needed**          |  There is no documented Data Needed for this Detection Rule yet  |
| **Trigger**              | <ul><li>[T1003: OS Credential Dumping](../Triggers/T1003.md)</li></ul>  |
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
description: This method detects mimikatz keywords in different Eventlogs (some of them only appear in older Mimikatz version that are however still used by different
    threat groups)
author: Florian Roth
date: 2017/01/10
modified: 2019/10/11
tags:
    - attack.s0002
    - attack.t1003
    - attack.lateral_movement
    - attack.credential_access
    - car.2013-07-001
    - car.2019-04-004
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
Get-WinEvent | where {($_.message -match "Message.*.* mimikatz .*" -or $_.message -match "Message.*.* mimilib .*" -or $_.message -match "Message.*.* <3 eo.oe .*" -or $_.message -match "Message.*.* eo.oe.kiwi .*" -or $_.message -match "Message.*.* privilege::debug .*" -or $_.message -match "Message.*.* sekurlsa::logonpasswords .*" -or $_.message -match "Message.*.* lsadump::sam .*" -or $_.message -match "Message.*.* mimidrv.sys .*" -or $_.message -match "Message.*.* p::d .*" -or $_.message -match "Message.*.* s::l .*") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
winlog.event_data.Message.keyword:(*\ mimikatz\ * OR *\ mimilib\ * OR *\ <3\ eo.oe\ * OR *\ eo.oe.kiwi\ * OR *\ privilege\:\:debug\ * OR *\ sekurlsa\:\:logonpasswords\ * OR *\ lsadump\:\:sam\ * OR *\ mimidrv.sys\ * OR *\ p\:\:d\ * OR *\ s\:\:l\ *)
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/06d71506-7beb-4f22-8888-e2e5e2ca7fd8 <<EOF
{
  "metadata": {
    "title": "Mimikatz Use",
    "description": "This method detects mimikatz keywords in different Eventlogs (some of them only appear in older Mimikatz version that are however still used by different threat groups)",
    "tags": [
      "attack.s0002",
      "attack.t1003",
      "attack.lateral_movement",
      "attack.credential_access",
      "car.2013-07-001",
      "car.2019-04-004"
    ],
    "query": "winlog.event_data.Message.keyword:(*\\ mimikatz\\ * OR *\\ mimilib\\ * OR *\\ <3\\ eo.oe\\ * OR *\\ eo.oe.kiwi\\ * OR *\\ privilege\\:\\:debug\\ * OR *\\ sekurlsa\\:\\:logonpasswords\\ * OR *\\ lsadump\\:\\:sam\\ * OR *\\ mimidrv.sys\\ * OR *\\ p\\:\\:d\\ * OR *\\ s\\:\\:l\\ *)"
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
                    "query": "winlog.event_data.Message.keyword:(*\\ mimikatz\\ * OR *\\ mimilib\\ * OR *\\ <3\\ eo.oe\\ * OR *\\ eo.oe.kiwi\\ * OR *\\ privilege\\:\\:debug\\ * OR *\\ sekurlsa\\:\\:logonpasswords\\ * OR *\\ lsadump\\:\\:sam\\ * OR *\\ mimidrv.sys\\ * OR *\\ p\\:\\:d\\ * OR *\\ s\\:\\:l\\ *)",
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
        "subject": "Sigma Rule 'Mimikatz Use'",
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
Message.keyword:(* mimikatz * * mimilib * * <3 eo.oe * * eo.oe.kiwi * * privilege\:\:debug * * sekurlsa\:\:logonpasswords * * lsadump\:\:sam * * mimidrv.sys * * p\:\:d * * s\:\:l *)
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
grep -P '^(?:.*.* mimikatz .*|.*.* mimilib .*|.*.* <3 eo\.oe .*|.*.* eo\.oe\.kiwi .*|.*.* privilege::debug .*|.*.* sekurlsa::logonpasswords .*|.*.* lsadump::sam .*|.*.* mimidrv\.sys .*|.*.* p::d .*|.*.* s::l .*)'
```



