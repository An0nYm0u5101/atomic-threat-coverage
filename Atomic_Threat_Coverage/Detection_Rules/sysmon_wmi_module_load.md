| Title                    | WMI Modules Loaded       |
|:-------------------------|:------------------|
| **Description**          | Detects non wmiprvse loading WMI modules |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1047: Windows Management Instrumentation](https://attack.mitre.org/techniques/T1047)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0011_7_windows_sysmon_image_loaded](../Data_Needed/DN_0011_7_windows_sysmon_image_loaded.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1047: Windows Management Instrumentation](../Triggers/T1047.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://github.com/Cyb3rWard0g/ThreatHunter-Playbook/tree/master/playbooks/windows/02_execution/T1047_windows_management_instrumentation/wmi_wmi_module_load.md](https://github.com/Cyb3rWard0g/ThreatHunter-Playbook/tree/master/playbooks/windows/02_execution/T1047_windows_management_instrumentation/wmi_wmi_module_load.md)</li></ul>  |
| **Author**               | Roberto Rodriguez @Cyb3rWard0g |


## Detection Rules

### Sigma rule

```
title: WMI Modules Loaded
id: 671bb7e3-a020-4824-a00e-2ee5b55f385e
description: Detects non wmiprvse loading WMI modules
status: experimental
date: 2019/08/10
modified: 2019/11/10
author: Roberto Rodriguez @Cyb3rWard0g
references:
    - https://github.com/Cyb3rWard0g/ThreatHunter-Playbook/tree/master/playbooks/windows/02_execution/T1047_windows_management_instrumentation/wmi_wmi_module_load.md
tags:
    - attack.execution
    - attack.t1047
logsource:
    product: windows
    service: sysmon
detection:
    selection: 
        EventID: 7
        ImageLoaded|endswith:
            - '\wmiclnt.dll'
            - '\WmiApRpl.dll'
            - '\wmiprov.dll'
            - '\wmiutils.dll'
            - '\wbemcomn.dll'
            - '\wbemprox.dll'
            - '\WMINet_Utils.dll'
            - '\wbemsvc.dll'
            - '\fastprox.dll'
    filter:
        Image|endswith:
            - '\WmiPrvSe.exe'
            - '\WmiPrvSE.exe'
            - '\WmiAPsrv.exe'
            - '\svchost.exe'
    condition: selection and not filter
fields:
    - ComputerName
    - User
    - Image
    - ImageLoaded
falsepositives:
    - Unknown
level: high

```





### powershell
    
```
Get-WinEvent -LogName Microsoft-Windows-Sysmon/Operational | where {(($_.ID -eq "7" -and ($_.message -match "ImageLoaded.*.*\\wmiclnt.dll" -or $_.message -match "ImageLoaded.*.*\\WmiApRpl.dll" -or $_.message -match "ImageLoaded.*.*\\wmiprov.dll" -or $_.message -match "ImageLoaded.*.*\\wmiutils.dll" -or $_.message -match "ImageLoaded.*.*\\wbemcomn.dll" -or $_.message -match "ImageLoaded.*.*\\wbemprox.dll" -or $_.message -match "ImageLoaded.*.*\\WMINet_Utils.dll" -or $_.message -match "ImageLoaded.*.*\\wbemsvc.dll" -or $_.message -match "ImageLoaded.*.*\\fastprox.dll")) -and  -not (($_.message -match "Image.*.*\\WmiPrvSe.exe" -or $_.message -match "Image.*.*\\WmiPrvSE.exe" -or $_.message -match "Image.*.*\\WmiAPsrv.exe" -or $_.message -match "Image.*.*\\svchost.exe"))) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.channel:"Microsoft\-Windows\-Sysmon\/Operational" AND (winlog.event_id:"7" AND winlog.event_data.ImageLoaded.keyword:(*\\wmiclnt.dll OR *\\WmiApRpl.dll OR *\\wmiprov.dll OR *\\wmiutils.dll OR *\\wbemcomn.dll OR *\\wbemprox.dll OR *\\WMINet_Utils.dll OR *\\wbemsvc.dll OR *\\fastprox.dll)) AND (NOT (winlog.event_data.Image.keyword:(*\\WmiPrvSe.exe OR *\\WmiPrvSE.exe OR *\\WmiAPsrv.exe OR *\\svchost.exe))))
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/671bb7e3-a020-4824-a00e-2ee5b55f385e <<EOF
{
  "metadata": {
    "title": "WMI Modules Loaded",
    "description": "Detects non wmiprvse loading WMI modules",
    "tags": [
      "attack.execution",
      "attack.t1047"
    ],
    "query": "(winlog.channel:\"Microsoft\\-Windows\\-Sysmon\\/Operational\" AND (winlog.event_id:\"7\" AND winlog.event_data.ImageLoaded.keyword:(*\\\\wmiclnt.dll OR *\\\\WmiApRpl.dll OR *\\\\wmiprov.dll OR *\\\\wmiutils.dll OR *\\\\wbemcomn.dll OR *\\\\wbemprox.dll OR *\\\\WMINet_Utils.dll OR *\\\\wbemsvc.dll OR *\\\\fastprox.dll)) AND (NOT (winlog.event_data.Image.keyword:(*\\\\WmiPrvSe.exe OR *\\\\WmiPrvSE.exe OR *\\\\WmiAPsrv.exe OR *\\\\svchost.exe))))"
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
                    "query": "(winlog.channel:\"Microsoft\\-Windows\\-Sysmon\\/Operational\" AND (winlog.event_id:\"7\" AND winlog.event_data.ImageLoaded.keyword:(*\\\\wmiclnt.dll OR *\\\\WmiApRpl.dll OR *\\\\wmiprov.dll OR *\\\\wmiutils.dll OR *\\\\wbemcomn.dll OR *\\\\wbemprox.dll OR *\\\\WMINet_Utils.dll OR *\\\\wbemsvc.dll OR *\\\\fastprox.dll)) AND (NOT (winlog.event_data.Image.keyword:(*\\\\WmiPrvSe.exe OR *\\\\WmiPrvSE.exe OR *\\\\WmiAPsrv.exe OR *\\\\svchost.exe))))",
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
        "subject": "Sigma Rule 'WMI Modules Loaded'",
        "body": "Hits:\n{{#ctx.payload.hits.hits}}Hit on {{_source.@timestamp}}:\nComputerName = {{_source.ComputerName}}\n        User = {{_source.User}}\n       Image = {{_source.Image}}\n ImageLoaded = {{_source.ImageLoaded}}================================================================================\n{{/ctx.payload.hits.hits}}",
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
((EventID:"7" AND ImageLoaded.keyword:(*\\wmiclnt.dll *\\WmiApRpl.dll *\\wmiprov.dll *\\wmiutils.dll *\\wbemcomn.dll *\\wbemprox.dll *\\WMINet_Utils.dll *\\wbemsvc.dll *\\fastprox.dll)) AND (NOT (Image.keyword:(*\\WmiPrvSe.exe *\\WmiPrvSE.exe *\\WmiAPsrv.exe *\\svchost.exe))))
```


### splunk
    
```
(source="WinEventLog:Microsoft-Windows-Sysmon/Operational" (EventCode="7" (ImageLoaded="*\\wmiclnt.dll" OR ImageLoaded="*\\WmiApRpl.dll" OR ImageLoaded="*\\wmiprov.dll" OR ImageLoaded="*\\wmiutils.dll" OR ImageLoaded="*\\wbemcomn.dll" OR ImageLoaded="*\\wbemprox.dll" OR ImageLoaded="*\\WMINet_Utils.dll" OR ImageLoaded="*\\wbemsvc.dll" OR ImageLoaded="*\\fastprox.dll")) NOT ((Image="*\\WmiPrvSe.exe" OR Image="*\\WmiPrvSE.exe" OR Image="*\\WmiAPsrv.exe" OR Image="*\\svchost.exe"))) | table ComputerName,User,Image,ImageLoaded
```


### logpoint
    
```
((event_id="7" ImageLoaded IN ["*\\wmiclnt.dll", "*\\WmiApRpl.dll", "*\\wmiprov.dll", "*\\wmiutils.dll", "*\\wbemcomn.dll", "*\\wbemprox.dll", "*\\WMINet_Utils.dll", "*\\wbemsvc.dll", "*\\fastprox.dll"])  -(Image IN ["*\\WmiPrvSe.exe", "*\\WmiPrvSE.exe", "*\\WmiAPsrv.exe", "*\\svchost.exe"]))
```


### grep
    
```
grep -P '^(?:.*(?=.*(?:.*(?=.*7)(?=.*(?:.*.*\wmiclnt\.dll|.*.*\WmiApRpl\.dll|.*.*\wmiprov\.dll|.*.*\wmiutils\.dll|.*.*\wbemcomn\.dll|.*.*\wbemprox\.dll|.*.*\WMINet_Utils\.dll|.*.*\wbemsvc\.dll|.*.*\fastprox\.dll))))(?=.*(?!.*(?:.*(?=.*(?:.*.*\WmiPrvSe\.exe|.*.*\WmiPrvSE\.exe|.*.*\WmiAPsrv\.exe|.*.*\svchost\.exe))))))'
```



