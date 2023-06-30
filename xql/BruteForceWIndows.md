

```shell
dataset = microsoft_windows_raw 
| filter event_id = 4625
| alter TargetUserName = json_extract_scalar(event_data , "$.TargetUserName"), WorkstationName = json_extract_scalar(event_data , "$.WorkstationName"),LogonType = json_extract_scalar(event_data , "$.LogonType"),IpAddress = json_extract_scalar(event_data , "$.IpAddress")
| filter TargetUserName !~= ".*\$"
| comp count(), max(_time) as mt by TargetUserName , WorkstationName
| join conflict_strategy = both  
(dataset = microsoft_windows_raw 
| filter event_id = 4624
| alter TargetUserName = json_extract_scalar(event_data , "$.TargetUserName"), WorkstationName = json_extract_scalar(event_data , "$.WorkstationName"),LogonType = json_extract_scalar(event_data , "$.LogonType"),IpAddress = json_extract_scalar(event_data , "$.IpAddress")
| filter TargetUserName !~= ".*\$" |alter success_time = _time) as succesfull_login succesfull_login.TargetUserName =TargetUserName and succesfull_login.WorkstationName =WorkstationName
| filter timestamp_diff(success_time ,mt, "second") >0 and timestamp_diff(success_time ,mt, "second") < 6000
```
