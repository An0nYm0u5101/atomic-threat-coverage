| Title                    | Execution of Renamed PaExec       |
|:-------------------------|:------------------|
| **Description**          | Detects execution of renamed paexec via imphash and executable product string |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1036: Masquerading](https://attack.mitre.org/techniques/T1036)</li><li>[T1036.003: Rename System Utilities](https://attack.mitre.org/techniques/T1036/003)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1036.003: Rename System Utilities](../Triggers/T1036.003.md)</li></ul>  |
| **Severity Level**       | medium |
| **False Positives**      | <ul><li>Unknown imphashes</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[sha256=01a461ad68d11b5b5096f45eb54df9ba62c5af413fa9eb544eacb598373a26bc](sha256=01a461ad68d11b5b5096f45eb54df9ba62c5af413fa9eb544eacb598373a26bc)</li><li>[https://summit.fireeye.com/content/dam/fireeye-www/summit/cds-2018/presentations/cds18-technical-s05-att&cking-fin7.pdf](https://summit.fireeye.com/content/dam/fireeye-www/summit/cds-2018/presentations/cds18-technical-s05-att&cking-fin7.pdf)</li></ul>  |
| **Author**               | Jason Lynch |
| Other Tags           | <ul><li>FIN7</li><li>car.2013-05-009</li></ul> | 

## Detection Rules

### Sigma rule

```
title: Execution of Renamed PaExec
id: 7b0666ad-3e38-4e3d-9bab-78b06de85f7b
status: experimental
description: Detects execution of renamed paexec via imphash and executable product string
references:
    - sha256=01a461ad68d11b5b5096f45eb54df9ba62c5af413fa9eb544eacb598373a26bc
    - https://summit.fireeye.com/content/dam/fireeye-www/summit/cds-2018/presentations/cds18-technical-s05-att&cking-fin7.pdf
tags:
    - attack.defense_evasion
    - attack.t1036 # an old one
    - attack.t1036.003
    - FIN7
    - car.2013-05-009
date: 2019/04/17
modified: 2020/09/06
author: Jason Lynch 
falsepositives:
    - Unknown imphashes
level: medium
logsource:
    category: process_creation
    product: windows
detection:
    selection1:
        Product:
            - '*PAExec*'
    selection2:
        Imphash:
            - 11D40A7B7876288F919AB819CC2D9802
            - 6444f8a34e99b8f7d9647de66aabe516
            - dfd6aa3f7b2b1035b76b718f1ddc689f
            - 1a6cca4d5460b1710a12dea39e4a592c
    filter1:
        Image: '*paexec*'
    condition: (selection1 and selection2) and not filter1

```





### powershell
    
```
Get-WinEvent | where {((($_.message -match "Product.*.*PAExec.*") -and ($_.message -match "11D40A7B7876288F919AB819CC2D9802" -or $_.message -match "6444f8a34e99b8f7d9647de66aabe516" -or $_.message -match "dfd6aa3f7b2b1035b76b718f1ddc689f" -or $_.message -match "1a6cca4d5460b1710a12dea39e4a592c")) -and  -not ($_.message -match "Image.*.*paexec.*")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
((Product.keyword:(*PAExec*) AND winlog.event_data.Imphash:("11d40a7b7876288f919ab819cc2d9802" OR "11D40A7B7876288F919AB819CC2D9802" OR "6444f8a34e99b8f7d9647de66aabe516" OR "6444F8A34E99B8F7D9647DE66AABE516" OR "dfd6aa3f7b2b1035b76b718f1ddc689f" OR "DFD6AA3F7B2B1035B76B718F1DDC689F" OR "1a6cca4d5460b1710a12dea39e4a592c" OR "1A6CCA4D5460B1710A12DEA39E4A592C")) AND (NOT (winlog.event_data.Image.keyword:*paexec*)))
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/7b0666ad-3e38-4e3d-9bab-78b06de85f7b <<EOF\n{\n  "metadata": {\n    "title": "Execution of Renamed PaExec",\n    "description": "Detects execution of renamed paexec via imphash and executable product string",\n    "tags": [\n      "attack.defense_evasion",\n      "attack.t1036",\n      "attack.t1036.003",\n      "FIN7",\n      "car.2013-05-009"\n    ],\n    "query": "((Product.keyword:(*PAExec*) AND winlog.event_data.Imphash:(\\"11d40a7b7876288f919ab819cc2d9802\\" OR \\"11D40A7B7876288F919AB819CC2D9802\\" OR \\"6444f8a34e99b8f7d9647de66aabe516\\" OR \\"6444F8A34E99B8F7D9647DE66AABE516\\" OR \\"dfd6aa3f7b2b1035b76b718f1ddc689f\\" OR \\"DFD6AA3F7B2B1035B76B718F1DDC689F\\" OR \\"1a6cca4d5460b1710a12dea39e4a592c\\" OR \\"1A6CCA4D5460B1710A12DEA39E4A592C\\")) AND (NOT (winlog.event_data.Image.keyword:*paexec*)))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "((Product.keyword:(*PAExec*) AND winlog.event_data.Imphash:(\\"11d40a7b7876288f919ab819cc2d9802\\" OR \\"11D40A7B7876288F919AB819CC2D9802\\" OR \\"6444f8a34e99b8f7d9647de66aabe516\\" OR \\"6444F8A34E99B8F7D9647DE66AABE516\\" OR \\"dfd6aa3f7b2b1035b76b718f1ddc689f\\" OR \\"DFD6AA3F7B2B1035B76B718F1DDC689F\\" OR \\"1a6cca4d5460b1710a12dea39e4a592c\\" OR \\"1A6CCA4D5460B1710A12DEA39E4A592C\\")) AND (NOT (winlog.event_data.Image.keyword:*paexec*)))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Execution of Renamed PaExec\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
((Product.keyword:(*PAExec*) AND Imphash:("11d40a7b7876288f919ab819cc2d9802" "11D40A7B7876288F919AB819CC2D9802" "6444f8a34e99b8f7d9647de66aabe516" "6444F8A34E99B8F7D9647DE66AABE516" "dfd6aa3f7b2b1035b76b718f1ddc689f" "DFD6AA3F7B2B1035B76B718F1DDC689F" "1a6cca4d5460b1710a12dea39e4a592c" "1A6CCA4D5460B1710A12DEA39E4A592C")) AND (NOT (Image.keyword:*paexec*)))
```


### splunk
    
```
(((Product="*PAExec*") (Imphash="11D40A7B7876288F919AB819CC2D9802" OR Imphash="6444f8a34e99b8f7d9647de66aabe516" OR Imphash="dfd6aa3f7b2b1035b76b718f1ddc689f" OR Imphash="1a6cca4d5460b1710a12dea39e4a592c")) NOT (Image="*paexec*"))
```


### logpoint
    
```
((Product IN ["*PAExec*"] Imphash IN ["11D40A7B7876288F919AB819CC2D9802", "6444f8a34e99b8f7d9647de66aabe516", "dfd6aa3f7b2b1035b76b718f1ddc689f", "1a6cca4d5460b1710a12dea39e4a592c"])  -(Image="*paexec*"))
```


### grep
    
```
grep -P '^(?:.*(?=.*(?:.*(?=.*(?:.*.*PAExec.*))(?=.*(?:.*11D40A7B7876288F919AB819CC2D9802|.*6444f8a34e99b8f7d9647de66aabe516|.*dfd6aa3f7b2b1035b76b718f1ddc689f|.*1a6cca4d5460b1710a12dea39e4a592c))))(?=.*(?!.*(?:.*(?=.*.*paexec.*)))))'
```



