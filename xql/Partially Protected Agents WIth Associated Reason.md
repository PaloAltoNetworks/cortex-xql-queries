```shell
config case_sensitive = false
| dataset = endpoints
|fields operational_status_description as osd, endpoint_name, operational_status, operating_system 
|filter operational_status = PARTIALLY_PROTECTED

| alter reason = arraymap (
            json_extract_array (to_json_string(osd ),"$."),
            json_extract_scalar ("@element", "$.reason"))

|arrayexpand reason 

|comp count_distinct(endpoint_name) as Agents by reason
| view graph type = pie header = "Partially Protected" xaxis = reason yaxis = Agents font = "Arial" headerfontsize = 12 legendfontsize = 10 
```
