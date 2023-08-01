```shell
//setting case sensitivity to false for this query
config case_sensitive = false

//defining the data source to run the query against
| dataset = endpoints

//Limiting results to only the necessary fields.
|fields operational_status_description as osd, endpoint_name, operational_status, operating_system, kernel_version, platform

//Filtering to only include endpoints that are Linux and not fully protected in the results.
|filter operational_status != PROTECTED and platform = ENUM.LINUX 

| alter reason = arraymap (
            json_extract_array (to_json_string(osd ),"$."),
            json_extract_scalar ("@element", "$.reason"))

|arrayexpand reason 

//Deduplicating the results based on the endpoint name and the underlying reason for the agent being in a degraded state.
|dedup endpoint_name, reason 

//Filtering for reasons that contain the string "kernel".
|filter reason contains "kernel"

|comp count_distinct(endpoint_name) as endpoints by kernel_version, operating_system 
```
