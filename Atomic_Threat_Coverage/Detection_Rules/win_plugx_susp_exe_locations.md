| Title                    | Executable Used by PlugX in Uncommon Location       |
|:-------------------------|:------------------|
| **Description**          | Detects the execution of an executable that is typically used by PlugX for DLL side loading started from an uncommon location |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1073: DLL Side-Loading](https://attack.mitre.org/techniques/T1073)</li><li>[T1574.002: DLL Side-Loading](https://attack.mitre.org/techniques/T1574/002)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0001_4688_windows_process_creation](../Data_Needed/DN_0001_4688_windows_process_creation.md)</li><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1574.002: DLL Side-Loading](../Triggers/T1574.002.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[http://www.hexacorn.com/blog/2016/03/10/beyond-good-ol-run-key-part-36/](http://www.hexacorn.com/blog/2016/03/10/beyond-good-ol-run-key-part-36/)</li><li>[https://countuponsecurity.com/2017/06/07/threat-hunting-in-the-enterprise-with-appcompatprocessor/](https://countuponsecurity.com/2017/06/07/threat-hunting-in-the-enterprise-with-appcompatprocessor/)</li></ul>  |
| **Author**               | Florian Roth |
| Other Tags           | <ul><li>attack.s0013</li></ul> | 

## Detection Rules

### Sigma rule

```
title: Executable Used by PlugX in Uncommon Location
id: aeab5ec5-be14-471a-80e8-e344418305c2
status: experimental
description: Detects the execution of an executable that is typically used by PlugX for DLL side loading started from an uncommon location
references:
    - http://www.hexacorn.com/blog/2016/03/10/beyond-good-ol-run-key-part-36/
    - https://countuponsecurity.com/2017/06/07/threat-hunting-in-the-enterprise-with-appcompatprocessor/
author: Florian Roth
date: 2017/06/12
tags:
    - attack.s0013
    - attack.defense_evasion
    - attack.t1073          # an old one
    - attack.t1574.002
logsource:
    category: process_creation
    product: windows
detection:
    selection_cammute:
        Image: '*\CamMute.exe'
    filter_cammute:
        Image: '*\Lenovo\Communication Utility\\*'
    selection_chrome_frame:
        Image: '*\chrome_frame_helper.exe'
    filter_chrome_frame:
        Image: '*\Google\Chrome\application\\*'
    selection_devemu:
        Image: '*\dvcemumanager.exe'
    filter_devemu:
        Image: '*\Microsoft Device Emulator\\*'
    selection_gadget:
        Image: '*\Gadget.exe'
    filter_gadget:
        Image: '*\Windows Media Player\\*'
    selection_hcc:
        Image: '*\hcc.exe'
    filter_hcc:
        Image: '*\HTML Help Workshop\\*'
    selection_hkcmd:
        Image: '*\hkcmd.exe'
    filter_hkcmd:
        Image:
            - '*\System32\\*'
            - '*\SysNative\\*'
            - '*\SysWowo64\\*'
    selection_mc:
        Image: '*\Mc.exe'
    filter_mc:
        Image:
            - '*\Microsoft Visual Studio*'
            - '*\Microsoft SDK*'
            - '*\Windows Kit*'
    selection_msmpeng:
        Image: '*\MsMpEng.exe'
    filter_msmpeng:
        Image:
            - '*\Microsoft Security Client\\*'
            - '*\Windows Defender\\*'
            - '*\AntiMalware\\*'
    selection_msseces:
        Image: '*\msseces.exe'
    filter_msseces:
        Image:
            - '*\Microsoft Security Center\\*'
            - '*\Microsoft Security Client\\*'
            - '*\Microsoft Security Essentials\\*'
    selection_oinfo:
        Image: '*\OInfoP11.exe'
    filter_oinfo:
        Image: '*\Common Files\Microsoft Shared\\*'
    selection_oleview:
        Image: '*\OleView.exe'
    filter_oleview:
        Image:
            - '*\Microsoft Visual Studio*'
            - '*\Microsoft SDK*'
            - '*\Windows Kit*'
            - '*\Windows Resource Kit\\*'
    selection_rc:
        Image: '*\rc.exe'
    filter_rc:
        Image:
            - '*\Microsoft Visual Studio*'
            - '*\Microsoft SDK*'
            - '*\Windows Kit*'
            - '*\Windows Resource Kit\\*'
            - '*\Microsoft.NET\\*'
    condition: ( selection_cammute and not filter_cammute ) or ( selection_chrome_frame and not filter_chrome_frame ) or ( selection_devemu and not filter_devemu ) or ( selection_gadget and not filter_gadget ) or ( selection_hcc and not filter_hcc ) or ( selection_hkcmd and not filter_hkcmd ) or ( selection_mc and not filter_mc ) or ( selection_msmpeng and not filter_msmpeng ) or ( selection_msseces and not filter_msseces ) or ( selection_oinfo and not filter_oinfo ) or ( selection_oleview and not filter_oleview ) or ( selection_rc and not filter_rc )
fields:
    - CommandLine
    - ParentCommandLine
falsepositives:
    - Unknown
level: high

```





### powershell
    
```
Get-WinEvent | where {(((((((((((($_.message -match "Image.*.*\\\\CamMute.exe" -and  -not ($_.message -match "Image.*.*\\\\Lenovo\\\\Communication Utility\\\\.*")) -or ($_.message -match "Image.*.*\\\\chrome_frame_helper.exe" -and  -not ($_.message -match "Image.*.*\\\\Google\\\\Chrome\\\\application\\\\.*"))) -or ($_.message -match "Image.*.*\\\\dvcemumanager.exe" -and  -not ($_.message -match "Image.*.*\\\\Microsoft Device Emulator\\\\.*"))) -or ($_.message -match "Image.*.*\\\\Gadget.exe" -and  -not ($_.message -match "Image.*.*\\\\Windows Media Player\\\\.*"))) -or ($_.message -match "Image.*.*\\\\hcc.exe" -and  -not ($_.message -match "Image.*.*\\\\HTML Help Workshop\\\\.*"))) -or ($_.message -match "Image.*.*\\\\hkcmd.exe" -and  -not (($_.message -match "Image.*.*\\\\System32\\\\.*" -or $_.message -match "Image.*.*\\\\SysNative\\\\.*" -or $_.message -match "Image.*.*\\\\SysWowo64\\\\.*")))) -or ($_.message -match "Image.*.*\\\\Mc.exe" -and  -not (($_.message -match "Image.*.*\\\\Microsoft Visual Studio.*" -or $_.message -match "Image.*.*\\\\Microsoft SDK.*" -or $_.message -match "Image.*.*\\\\Windows Kit.*")))) -or ($_.message -match "Image.*.*\\\\MsMpEng.exe" -and  -not (($_.message -match "Image.*.*\\\\Microsoft Security Client\\\\.*" -or $_.message -match "Image.*.*\\\\Windows Defender\\\\.*" -or $_.message -match "Image.*.*\\\\AntiMalware\\\\.*")))) -or ($_.message -match "Image.*.*\\\\msseces.exe" -and  -not (($_.message -match "Image.*.*\\\\Microsoft Security Center\\\\.*" -or $_.message -match "Image.*.*\\\\Microsoft Security Client\\\\.*" -or $_.message -match "Image.*.*\\\\Microsoft Security Essentials\\\\.*")))) -or ($_.message -match "Image.*.*\\\\OInfoP11.exe" -and  -not ($_.message -match "Image.*.*\\\\Common Files\\\\Microsoft Shared\\\\.*"))) -or ($_.message -match "Image.*.*\\\\OleView.exe" -and  -not (($_.message -match "Image.*.*\\\\Microsoft Visual Studio.*" -or $_.message -match "Image.*.*\\\\Microsoft SDK.*" -or $_.message -match "Image.*.*\\\\Windows Kit.*" -or $_.message -match "Image.*.*\\\\Windows Resource Kit\\\\.*")))) -or ($_.message -match "Image.*.*\\\\rc.exe" -and  -not (($_.message -match "Image.*.*\\\\Microsoft Visual Studio.*" -or $_.message -match "Image.*.*\\\\Microsoft SDK.*" -or $_.message -match "Image.*.*\\\\Windows Kit.*" -or $_.message -match "Image.*.*\\\\Windows Resource Kit\\\\.*" -or $_.message -match "Image.*.*\\\\Microsoft.NET\\\\.*")))) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
((((((((((((winlog.event_data.Image.keyword:*\\\\CamMute.exe AND (NOT (winlog.event_data.Image.keyword:*\\\\Lenovo\\\\Communication\\ Utility\\\\*))) OR (winlog.event_data.Image.keyword:*\\\\chrome_frame_helper.exe AND (NOT (winlog.event_data.Image.keyword:*\\\\Google\\\\Chrome\\\\application\\\\*)))) OR (winlog.event_data.Image.keyword:*\\\\dvcemumanager.exe AND (NOT (winlog.event_data.Image.keyword:*\\\\Microsoft\\ Device\\ Emulator\\\\*)))) OR (winlog.event_data.Image.keyword:*\\\\Gadget.exe AND (NOT (winlog.event_data.Image.keyword:*\\\\Windows\\ Media\\ Player\\\\*)))) OR (winlog.event_data.Image.keyword:*\\\\hcc.exe AND (NOT (winlog.event_data.Image.keyword:*\\\\HTML\\ Help\\ Workshop\\\\*)))) OR (winlog.event_data.Image.keyword:*\\\\hkcmd.exe AND (NOT (winlog.event_data.Image.keyword:(*\\\\System32\\\\* OR *\\\\SysNative\\\\* OR *\\\\SysWowo64\\\\*))))) OR (winlog.event_data.Image.keyword:*\\\\Mc.exe AND (NOT (winlog.event_data.Image.keyword:(*\\\\Microsoft\\ Visual\\ Studio* OR *\\\\Microsoft\\ SDK* OR *\\\\Windows\\ Kit*))))) OR (winlog.event_data.Image.keyword:*\\\\MsMpEng.exe AND (NOT (winlog.event_data.Image.keyword:(*\\\\Microsoft\\ Security\\ Client\\\\* OR *\\\\Windows\\ Defender\\\\* OR *\\\\AntiMalware\\\\*))))) OR (winlog.event_data.Image.keyword:*\\\\msseces.exe AND (NOT (winlog.event_data.Image.keyword:(*\\\\Microsoft\\ Security\\ Center\\\\* OR *\\\\Microsoft\\ Security\\ Client\\\\* OR *\\\\Microsoft\\ Security\\ Essentials\\\\*))))) OR (winlog.event_data.Image.keyword:*\\\\OInfoP11.exe AND (NOT (winlog.event_data.Image.keyword:*\\\\Common\\ Files\\\\Microsoft\\ Shared\\\\*)))) OR (winlog.event_data.Image.keyword:*\\\\OleView.exe AND (NOT (winlog.event_data.Image.keyword:(*\\\\Microsoft\\ Visual\\ Studio* OR *\\\\Microsoft\\ SDK* OR *\\\\Windows\\ Kit* OR *\\\\Windows\\ Resource\\ Kit\\\\*))))) OR (winlog.event_data.Image.keyword:*\\\\rc.exe AND (NOT (winlog.event_data.Image.keyword:(*\\\\Microsoft\\ Visual\\ Studio* OR *\\\\Microsoft\\ SDK* OR *\\\\Windows\\ Kit* OR *\\\\Windows\\ Resource\\ Kit\\\\* OR *\\\\Microsoft.NET\\\\*)))))
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/aeab5ec5-be14-471a-80e8-e344418305c2 <<EOF\n{\n  "metadata": {\n    "title": "Executable Used by PlugX in Uncommon Location",\n    "description": "Detects the execution of an executable that is typically used by PlugX for DLL side loading started from an uncommon location",\n    "tags": [\n      "attack.s0013",\n      "attack.defense_evasion",\n      "attack.t1073",\n      "attack.t1574.002"\n    ],\n    "query": "((((((((((((winlog.event_data.Image.keyword:*\\\\\\\\CamMute.exe AND (NOT (winlog.event_data.Image.keyword:*\\\\\\\\Lenovo\\\\\\\\Communication\\\\ Utility\\\\\\\\*))) OR (winlog.event_data.Image.keyword:*\\\\\\\\chrome_frame_helper.exe AND (NOT (winlog.event_data.Image.keyword:*\\\\\\\\Google\\\\\\\\Chrome\\\\\\\\application\\\\\\\\*)))) OR (winlog.event_data.Image.keyword:*\\\\\\\\dvcemumanager.exe AND (NOT (winlog.event_data.Image.keyword:*\\\\\\\\Microsoft\\\\ Device\\\\ Emulator\\\\\\\\*)))) OR (winlog.event_data.Image.keyword:*\\\\\\\\Gadget.exe AND (NOT (winlog.event_data.Image.keyword:*\\\\\\\\Windows\\\\ Media\\\\ Player\\\\\\\\*)))) OR (winlog.event_data.Image.keyword:*\\\\\\\\hcc.exe AND (NOT (winlog.event_data.Image.keyword:*\\\\\\\\HTML\\\\ Help\\\\ Workshop\\\\\\\\*)))) OR (winlog.event_data.Image.keyword:*\\\\\\\\hkcmd.exe AND (NOT (winlog.event_data.Image.keyword:(*\\\\\\\\System32\\\\\\\\* OR *\\\\\\\\SysNative\\\\\\\\* OR *\\\\\\\\SysWowo64\\\\\\\\*))))) OR (winlog.event_data.Image.keyword:*\\\\\\\\Mc.exe AND (NOT (winlog.event_data.Image.keyword:(*\\\\\\\\Microsoft\\\\ Visual\\\\ Studio* OR *\\\\\\\\Microsoft\\\\ SDK* OR *\\\\\\\\Windows\\\\ Kit*))))) OR (winlog.event_data.Image.keyword:*\\\\\\\\MsMpEng.exe AND (NOT (winlog.event_data.Image.keyword:(*\\\\\\\\Microsoft\\\\ Security\\\\ Client\\\\\\\\* OR *\\\\\\\\Windows\\\\ Defender\\\\\\\\* OR *\\\\\\\\AntiMalware\\\\\\\\*))))) OR (winlog.event_data.Image.keyword:*\\\\\\\\msseces.exe AND (NOT (winlog.event_data.Image.keyword:(*\\\\\\\\Microsoft\\\\ Security\\\\ Center\\\\\\\\* OR *\\\\\\\\Microsoft\\\\ Security\\\\ Client\\\\\\\\* OR *\\\\\\\\Microsoft\\\\ Security\\\\ Essentials\\\\\\\\*))))) OR (winlog.event_data.Image.keyword:*\\\\\\\\OInfoP11.exe AND (NOT (winlog.event_data.Image.keyword:*\\\\\\\\Common\\\\ Files\\\\\\\\Microsoft\\\\ Shared\\\\\\\\*)))) OR (winlog.event_data.Image.keyword:*\\\\\\\\OleView.exe AND (NOT (winlog.event_data.Image.keyword:(*\\\\\\\\Microsoft\\\\ Visual\\\\ Studio* OR *\\\\\\\\Microsoft\\\\ SDK* OR *\\\\\\\\Windows\\\\ Kit* OR *\\\\\\\\Windows\\\\ Resource\\\\ Kit\\\\\\\\*))))) OR (winlog.event_data.Image.keyword:*\\\\\\\\rc.exe AND (NOT (winlog.event_data.Image.keyword:(*\\\\\\\\Microsoft\\\\ Visual\\\\ Studio* OR *\\\\\\\\Microsoft\\\\ SDK* OR *\\\\\\\\Windows\\\\ Kit* OR *\\\\\\\\Windows\\\\ Resource\\\\ Kit\\\\\\\\* OR *\\\\\\\\Microsoft.NET\\\\\\\\*)))))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "((((((((((((winlog.event_data.Image.keyword:*\\\\\\\\CamMute.exe AND (NOT (winlog.event_data.Image.keyword:*\\\\\\\\Lenovo\\\\\\\\Communication\\\\ Utility\\\\\\\\*))) OR (winlog.event_data.Image.keyword:*\\\\\\\\chrome_frame_helper.exe AND (NOT (winlog.event_data.Image.keyword:*\\\\\\\\Google\\\\\\\\Chrome\\\\\\\\application\\\\\\\\*)))) OR (winlog.event_data.Image.keyword:*\\\\\\\\dvcemumanager.exe AND (NOT (winlog.event_data.Image.keyword:*\\\\\\\\Microsoft\\\\ Device\\\\ Emulator\\\\\\\\*)))) OR (winlog.event_data.Image.keyword:*\\\\\\\\Gadget.exe AND (NOT (winlog.event_data.Image.keyword:*\\\\\\\\Windows\\\\ Media\\\\ Player\\\\\\\\*)))) OR (winlog.event_data.Image.keyword:*\\\\\\\\hcc.exe AND (NOT (winlog.event_data.Image.keyword:*\\\\\\\\HTML\\\\ Help\\\\ Workshop\\\\\\\\*)))) OR (winlog.event_data.Image.keyword:*\\\\\\\\hkcmd.exe AND (NOT (winlog.event_data.Image.keyword:(*\\\\\\\\System32\\\\\\\\* OR *\\\\\\\\SysNative\\\\\\\\* OR *\\\\\\\\SysWowo64\\\\\\\\*))))) OR (winlog.event_data.Image.keyword:*\\\\\\\\Mc.exe AND (NOT (winlog.event_data.Image.keyword:(*\\\\\\\\Microsoft\\\\ Visual\\\\ Studio* OR *\\\\\\\\Microsoft\\\\ SDK* OR *\\\\\\\\Windows\\\\ Kit*))))) OR (winlog.event_data.Image.keyword:*\\\\\\\\MsMpEng.exe AND (NOT (winlog.event_data.Image.keyword:(*\\\\\\\\Microsoft\\\\ Security\\\\ Client\\\\\\\\* OR *\\\\\\\\Windows\\\\ Defender\\\\\\\\* OR *\\\\\\\\AntiMalware\\\\\\\\*))))) OR (winlog.event_data.Image.keyword:*\\\\\\\\msseces.exe AND (NOT (winlog.event_data.Image.keyword:(*\\\\\\\\Microsoft\\\\ Security\\\\ Center\\\\\\\\* OR *\\\\\\\\Microsoft\\\\ Security\\\\ Client\\\\\\\\* OR *\\\\\\\\Microsoft\\\\ Security\\\\ Essentials\\\\\\\\*))))) OR (winlog.event_data.Image.keyword:*\\\\\\\\OInfoP11.exe AND (NOT (winlog.event_data.Image.keyword:*\\\\\\\\Common\\\\ Files\\\\\\\\Microsoft\\\\ Shared\\\\\\\\*)))) OR (winlog.event_data.Image.keyword:*\\\\\\\\OleView.exe AND (NOT (winlog.event_data.Image.keyword:(*\\\\\\\\Microsoft\\\\ Visual\\\\ Studio* OR *\\\\\\\\Microsoft\\\\ SDK* OR *\\\\\\\\Windows\\\\ Kit* OR *\\\\\\\\Windows\\\\ Resource\\\\ Kit\\\\\\\\*))))) OR (winlog.event_data.Image.keyword:*\\\\\\\\rc.exe AND (NOT (winlog.event_data.Image.keyword:(*\\\\\\\\Microsoft\\\\ Visual\\\\ Studio* OR *\\\\\\\\Microsoft\\\\ SDK* OR *\\\\\\\\Windows\\\\ Kit* OR *\\\\\\\\Windows\\\\ Resource\\\\ Kit\\\\\\\\* OR *\\\\\\\\Microsoft.NET\\\\\\\\*)))))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Executable Used by PlugX in Uncommon Location\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}Hit on {{_source.@timestamp}}:\\n      CommandLine = {{_source.CommandLine}}\\nParentCommandLine = {{_source.ParentCommandLine}}================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
((((((((((((Image.keyword:*\\\\CamMute.exe AND (NOT (Image.keyword:*\\\\Lenovo\\\\Communication Utility\\\\*))) OR (Image.keyword:*\\\\chrome_frame_helper.exe AND (NOT (Image.keyword:*\\\\Google\\\\Chrome\\\\application\\\\*)))) OR (Image.keyword:*\\\\dvcemumanager.exe AND (NOT (Image.keyword:*\\\\Microsoft Device Emulator\\\\*)))) OR (Image.keyword:*\\\\Gadget.exe AND (NOT (Image.keyword:*\\\\Windows Media Player\\\\*)))) OR (Image.keyword:*\\\\hcc.exe AND (NOT (Image.keyword:*\\\\HTML Help Workshop\\\\*)))) OR (Image.keyword:*\\\\hkcmd.exe AND (NOT (Image.keyword:(*\\\\System32\\\\* *\\\\SysNative\\\\* *\\\\SysWowo64\\\\*))))) OR (Image.keyword:*\\\\Mc.exe AND (NOT (Image.keyword:(*\\\\Microsoft Visual Studio* *\\\\Microsoft SDK* *\\\\Windows Kit*))))) OR (Image.keyword:*\\\\MsMpEng.exe AND (NOT (Image.keyword:(*\\\\Microsoft Security Client\\\\* *\\\\Windows Defender\\\\* *\\\\AntiMalware\\\\*))))) OR (Image.keyword:*\\\\msseces.exe AND (NOT (Image.keyword:(*\\\\Microsoft Security Center\\\\* *\\\\Microsoft Security Client\\\\* *\\\\Microsoft Security Essentials\\\\*))))) OR (Image.keyword:*\\\\OInfoP11.exe AND (NOT (Image.keyword:*\\\\Common Files\\\\Microsoft Shared\\\\*)))) OR (Image.keyword:*\\\\OleView.exe AND (NOT (Image.keyword:(*\\\\Microsoft Visual Studio* *\\\\Microsoft SDK* *\\\\Windows Kit* *\\\\Windows Resource Kit\\\\*))))) OR (Image.keyword:*\\\\rc.exe AND (NOT (Image.keyword:(*\\\\Microsoft Visual Studio* *\\\\Microsoft SDK* *\\\\Windows Kit* *\\\\Windows Resource Kit\\\\* *\\\\Microsoft.NET\\\\*)))))
```


### splunk
    
```
((((((((((((Image="*\\\\CamMute.exe" NOT (Image="*\\\\Lenovo\\\\Communication Utility\\\\*")) OR (Image="*\\\\chrome_frame_helper.exe" NOT (Image="*\\\\Google\\\\Chrome\\\\application\\\\*"))) OR (Image="*\\\\dvcemumanager.exe" NOT (Image="*\\\\Microsoft Device Emulator\\\\*"))) OR (Image="*\\\\Gadget.exe" NOT (Image="*\\\\Windows Media Player\\\\*"))) OR (Image="*\\\\hcc.exe" NOT (Image="*\\\\HTML Help Workshop\\\\*"))) OR (Image="*\\\\hkcmd.exe" NOT ((Image="*\\\\System32\\\\*" OR Image="*\\\\SysNative\\\\*" OR Image="*\\\\SysWowo64\\\\*")))) OR (Image="*\\\\Mc.exe" NOT ((Image="*\\\\Microsoft Visual Studio*" OR Image="*\\\\Microsoft SDK*" OR Image="*\\\\Windows Kit*")))) OR (Image="*\\\\MsMpEng.exe" NOT ((Image="*\\\\Microsoft Security Client\\\\*" OR Image="*\\\\Windows Defender\\\\*" OR Image="*\\\\AntiMalware\\\\*")))) OR (Image="*\\\\msseces.exe" NOT ((Image="*\\\\Microsoft Security Center\\\\*" OR Image="*\\\\Microsoft Security Client\\\\*" OR Image="*\\\\Microsoft Security Essentials\\\\*")))) OR (Image="*\\\\OInfoP11.exe" NOT (Image="*\\\\Common Files\\\\Microsoft Shared\\\\*"))) OR (Image="*\\\\OleView.exe" NOT ((Image="*\\\\Microsoft Visual Studio*" OR Image="*\\\\Microsoft SDK*" OR Image="*\\\\Windows Kit*" OR Image="*\\\\Windows Resource Kit\\\\*")))) OR (Image="*\\\\rc.exe" NOT ((Image="*\\\\Microsoft Visual Studio*" OR Image="*\\\\Microsoft SDK*" OR Image="*\\\\Windows Kit*" OR Image="*\\\\Windows Resource Kit\\\\*" OR Image="*\\\\Microsoft.NET\\\\*")))) | table CommandLine,ParentCommandLine
```


### logpoint
    
```
((((((((((((Image="*\\\\CamMute.exe"  -(Image="*\\\\Lenovo\\\\Communication Utility\\\\*")) OR (Image="*\\\\chrome_frame_helper.exe"  -(Image="*\\\\Google\\\\Chrome\\\\application\\\\*"))) OR (Image="*\\\\dvcemumanager.exe"  -(Image="*\\\\Microsoft Device Emulator\\\\*"))) OR (Image="*\\\\Gadget.exe"  -(Image="*\\\\Windows Media Player\\\\*"))) OR (Image="*\\\\hcc.exe"  -(Image="*\\\\HTML Help Workshop\\\\*"))) OR (Image="*\\\\hkcmd.exe"  -(Image IN ["*\\\\System32\\\\*", "*\\\\SysNative\\\\*", "*\\\\SysWowo64\\\\*"]))) OR (Image="*\\\\Mc.exe"  -(Image IN ["*\\\\Microsoft Visual Studio*", "*\\\\Microsoft SDK*", "*\\\\Windows Kit*"]))) OR (Image="*\\\\MsMpEng.exe"  -(Image IN ["*\\\\Microsoft Security Client\\\\*", "*\\\\Windows Defender\\\\*", "*\\\\AntiMalware\\\\*"]))) OR (Image="*\\\\msseces.exe"  -(Image IN ["*\\\\Microsoft Security Center\\\\*", "*\\\\Microsoft Security Client\\\\*", "*\\\\Microsoft Security Essentials\\\\*"]))) OR (Image="*\\\\OInfoP11.exe"  -(Image="*\\\\Common Files\\\\Microsoft Shared\\\\*"))) OR (Image="*\\\\OleView.exe"  -(Image IN ["*\\\\Microsoft Visual Studio*", "*\\\\Microsoft SDK*", "*\\\\Windows Kit*", "*\\\\Windows Resource Kit\\\\*"]))) OR (Image="*\\\\rc.exe"  -(Image IN ["*\\\\Microsoft Visual Studio*", "*\\\\Microsoft SDK*", "*\\\\Windows Kit*", "*\\\\Windows Resource Kit\\\\*", "*\\\\Microsoft.NET\\\\*"])))
```


### grep
    
```
grep -P '^(?:.*(?:.*(?:.*(?:.*(?:.*(?:.*(?:.*(?:.*(?:.*(?:.*(?:.*(?:.*(?:.*(?:.*(?:.*(?:.*(?:.*(?:.*(?:.*(?:.*(?:.*(?:.*(?:.*(?=.*.*\\CamMute\\.exe)(?=.*(?!.*(?:.*(?=.*.*\\Lenovo\\Communication Utility\\\\.*)))))|.*(?:.*(?=.*.*\\chrome_frame_helper\\.exe)(?=.*(?!.*(?:.*(?=.*.*\\Google\\Chrome\\application\\\\.*)))))))|.*(?:.*(?=.*.*\\dvcemumanager\\.exe)(?=.*(?!.*(?:.*(?=.*.*\\Microsoft Device Emulator\\\\.*)))))))|.*(?:.*(?=.*.*\\Gadget\\.exe)(?=.*(?!.*(?:.*(?=.*.*\\Windows Media Player\\\\.*)))))))|.*(?:.*(?=.*.*\\hcc\\.exe)(?=.*(?!.*(?:.*(?=.*.*\\HTML Help Workshop\\\\.*)))))))|.*(?:.*(?=.*.*\\hkcmd\\.exe)(?=.*(?!.*(?:.*(?=.*(?:.*.*\\System32\\\\.*|.*.*\\SysNative\\\\.*|.*.*\\SysWowo64\\\\.*))))))))|.*(?:.*(?=.*.*\\Mc\\.exe)(?=.*(?!.*(?:.*(?=.*(?:.*.*\\Microsoft Visual Studio.*|.*.*\\Microsoft SDK.*|.*.*\\Windows Kit.*))))))))|.*(?:.*(?=.*.*\\MsMpEng\\.exe)(?=.*(?!.*(?:.*(?=.*(?:.*.*\\Microsoft Security Client\\\\.*|.*.*\\Windows Defender\\\\.*|.*.*\\AntiMalware\\\\.*))))))))|.*(?:.*(?=.*.*\\msseces\\.exe)(?=.*(?!.*(?:.*(?=.*(?:.*.*\\Microsoft Security Center\\\\.*|.*.*\\Microsoft Security Client\\\\.*|.*.*\\Microsoft Security Essentials\\\\.*))))))))|.*(?:.*(?=.*.*\\OInfoP11\\.exe)(?=.*(?!.*(?:.*(?=.*.*\\Common Files\\Microsoft Shared\\\\.*)))))))|.*(?:.*(?=.*.*\\OleView\\.exe)(?=.*(?!.*(?:.*(?=.*(?:.*.*\\Microsoft Visual Studio.*|.*.*\\Microsoft SDK.*|.*.*\\Windows Kit.*|.*.*\\Windows Resource Kit\\\\.*))))))))|.*(?:.*(?=.*.*\\rc\\.exe)(?=.*(?!.*(?:.*(?=.*(?:.*.*\\Microsoft Visual Studio.*|.*.*\\Microsoft SDK.*|.*.*\\Windows Kit.*|.*.*\\Windows Resource Kit\\\\.*|.*.*\\Microsoft\\.NET\\\\.*))))))))'
```



