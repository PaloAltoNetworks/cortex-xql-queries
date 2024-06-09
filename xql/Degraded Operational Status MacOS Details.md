```shell
config case_sensitive = false
| dataset = endpoints
|fields operational_status_description as osd, endpoint_name, ip_address, operational_status, operating_system, agent_version, last_seen, content_version, endpoint_status, last_content_update_time, manual_protection_pause, linux_operation_mode, endpoint_type, kernel_version, platform
|filter operational_status != PROTECTED and platform = ENUM.MACOS  

| alter reason = arraymap (
            json_extract_array (to_json_string(osd ),"$."),
            json_extract_scalar ("@element", "$.reason"))

|arrayexpand reason 

|dedup endpoint_name, reason 

 
|fields endpoint_name, ip_address, endpoint_type, endpoint_status, last_seen,operational_status, reason,agent_version, content_version, last_content_update_time, manual_protection_pause
|sort desc reason, desc agent_version, desc content_version 
```
