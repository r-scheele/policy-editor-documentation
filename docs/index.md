# Introduction


This project aims to abstract the technical details of writing REGO and provide a simple interface to write policies for your application in a Github/Gitlab repository. By transforming the familiar JSON format to REGO, the developer can focus on writing the policy and not on the technical details of REGO.

![API Image](./img/api.png)

## Example request and response to the policy editor.

As an example, the API transforms the following JSON policy into REGO rules:

```json
{
    "example": {
        "name": "Example",
        "repo_url": "https://github.com/geobeyond/test-rego",
        "repo_id": 12345,
        "rules": [
            [
                {
                    "command": "allow_if_object_in_database",
                    "properties": {
                        "datasource_name": "usergroups",
                        "datasource_variables": ["name", "groupname"],
                    },
                },
                {
                    "command": "input_prop_in",
                    "properties": {
                        "input_property": "company",
                        "datasource_name": "items",
                        "datasource_loop_variable": "name",
                    },
                },
                {
                    "command": "input_prop_equals",
                    "properties": {
                        "input_property": "request_method",
                        "value": "GET",
                    },
                },
            ],
            [
                {
                    "command": "input_prop_equals",
                    "properties": {
                        "input_property": "company",
                        "value": "geobeyond",
                    },
                },
            ],
            [
                {
                    "command": "allow_full_access",
                    "properties": {
                        "input_property": "groupname",
                        "value": "EDITOR_ATAC",
                    },
                },
            ],
            [
                {
                    "command": "input_prop_equals",
                    "properties": {
                        "input_property": "request_path",
                        "value": "v1/collections/*",
                        "exceptional_value": "obs",
                    },
                }
            ],
            [
                {
                    "command": "input_prop_equals",
                    "properties": {
                        "input_property": "request_path",
                        "value": "v1/collections/{username}/*",
                    },
                }
            ],
            [
                {
                    "command": "input_prop_equals",
                    "properties": {
                        "input_property": "request_path",
                        "value": "v1/collections/*",
                    },
                }
            ],
            [
                {
                    "command": "input_prop_equals",
                    "properties": {
                        "input_property": "request_path",
                        "value": "v1/collections",
                    },
                },
            ],
            [
                {
                    "command": "input_prop_equals",
                    "properties": {
                        "input_property": "request_path",
                        "value": "v1/collections/{username}",
                    },
                }
            ],
            [
                {
                    "command": "input_prop_equals",
                    "properties": {
                        "input_property": "request_path",
                        "value": "v1/collections/{username}/*",
                        "exceptional_value": "obs",
                    },
                }
            ],
        ],
    }
}
```

result: 

```rego

package httpapi.authz
import input
default allow = false



allow {
  {"name": input.name,"groupname": input.groupname} = data.usergroups[_]
  input.company = data.items[_].company
  input.request_method = "GET"
}

allow {
  input.company = "geobeyond"
}

allow {
  input.groupname = "EDITOR_ATAC"
}

allow {
  input.request_path[0] = "v1" 
  input.request_path[1] = "collections" 
  input.request_path[1] != "obs"
}

allow {
  some username  
  input.request_path = ["user", username] 
  input.preferred_username = username  
  
}

allow {
  input.request_path[0] = "v1" 
  input.request_path[1] = "collections" 
  
}

allow {
  input.request_path = ["v1"]
}

allow {
  some username 
  input.preferred_username = username  
  input.request_path[0] = "v1" 
  input.request_path[1] = "collections" 
  input.request_path[2] != "obs"
}

```
