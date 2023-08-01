```shell
//Setting case sensitivity to false for this query.
config case_sensitive = false

//defining the data source to run the query against.
|dataset = endpoints

//Limiting results to only the necessary fields.
|fields operational_status_description as osd, endpoint_name, operational_status, operating_system 

//Filtering to only include unprotected endpoints in the results.
|filter operational_status = UNPROTECTED

/*The "arraymap" function applies a specified function to every element of an array. The syntax for this function is to first define the array, then define the function to apply separated by a comma.
In this example the field "operational_status_description" has been given the alias "osd", so that is how it will be referenced for the duration of this query. The osd field is a string, but is structured like a json array. In order for the arraymap function to work, this field must be converted to an XQL native array. First, the "to_json_string" function is applied to the osd field which returns a json formatted string. Next, the "json_extract_array" function is applied to the output of the to_json_string function, which returns an XQL native array. Now that the input for the arraymap function is in the correct format, the "json_extract_scalar" is defined as the function to apply to every element in the array, using the "@element" syntax to apply the function to the field name "reason". The result is a new field created by the alter stage named "reason" which contains an array of all of the extracted reasons from the osd field.
*/
| alter reason = arraymap (
            json_extract_array (to_json_string(osd ),"$."),
            json_extract_scalar ("@element", "$.reason"))


//Expanding the array to create individual log rows.
|arrayexpand reason 

//Counting number of distinct endpoints with a common value in the "reason" field.
|comp count_distinct(endpoint_name) as Agents by reason

//Graph parameters.
| view graph type = pie header = "Unprotected" xaxis = reason yaxis = Agents font = "Arial" headerfontsize = 12 legendfontsize = 10 
```
