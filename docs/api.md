# Rest API

POST `/policies/`  Create new policy

The POST route takes a request body containing the rules defined according to the schema. It is then, used to build a new policy. The response will be a REGO file written and pushed to GitHub as a newly established remote repository, with the request body conforming to a specified syntax described by the pydantic model `Policy`. <br />

An example request body is: <br />

```json
{
"name": "Example",
"rules": [
   [
      {
      "command": "input_prop_equals",
      "properties": {
         "input_property": "request_path",
         "value": [
            "v1",
            "collections",
            "*"
         ],
         "exceptional_value": "obs"
      }
      },
      {
         "command": "input_prop_in",
         "properties": {
            "input_property": "company",
            "datasource_name": "items",
            "datasource_loop_variable": "name"
         }
      },
      {
         "command": "input_prop_equals",
         "properties": {
            "input_property": "request_method",
            "value": "GET"
         }
      }
   ],
   [
         {
         "command": "allow_full_access",
         "properties": {
            "input_property": "groupname",
            "value": "EDITOR_ATAC"
         }
      }
   ]
]}

```

```bash
curl -X POST localhost:8000/policies
   -H 'Content-Type: application/json'
   -d $request_body

```

An example response body is: <br />
```json
{"status": 200, "message": "Policy created successfully"}
```

GET `/policies/{policy_name}` 

This route gets a specific policy by it's name. The response will be a policy object conforming to the pydantic model `Policy`. <br />
Example response body: <br />


```bash
curl localhost:8000/policies/{policy_name}
```
```json
{
"name": "Example",
"rules": [
   [
      {
         "command": "input_prop_equals",
         "properties": {
            "input_property": "request_path",
            "value": [
               "v1",
               "collections",
               "*"
            ],
            "exceptional_value": "obs"
      }
      },
      {
         "command": "input_prop_in",
         "properties": {
            "input_property": "company",
            "datasource_name": "items",
            "datasource_loop_variable": "name"
         }
      },
      {
         "command": "input_prop_equals",
         "properties": {
            "input_property": "request_method",
            "value": "GET"
         }
      }
   ],
   [
         {
         "command": "allow_full_access",
         "properties": {
            "input_property": "groupname",
            "value": "EDITOR_ATAC"
         }
      }
   ]
]}
```





GET `/policies/` Read all policies  

The GET route get all policies that have been created. The response will be a list of all policies that have been created by a certain user, and contains all the associating rules with the policy <br />

```bash
curl localhost:8000/policies
```

An example response body is: <br />

```json
[
   {
      "name": "Example",
      "rules": [
         [
            {
            "command": "input_prop_equals",
            "properties": {
               "input_property": "request_path",
               "value": [
                  "v1",
                  "collections",
                  "*"
               ],
               "exceptional_value": "obs"
            }
            },
            {
               "command": "input_prop_in",
               "properties": {
                  "input_property": "company",
                  "datasource_name": "items",
                  "datasource_loop_variable": "name"
               }
            },
            {
               "command": "input_prop_equals",
               "properties": {
                  "input_property": "request_method",
                  "value": "GET"
               }
            }
         ],
         [
            {
            "command": "allow_full_access",
            "properties": {
               "input_property": "groupname",
               "value": "EDITOR_ATAC"
            }
         }
      ]
   }
]
```


PUT `/policies/{policy_name}` Update existing policy by name

```bash
curl -X PUT localhost:8000/policies/{policy_name}
   -d $request_body
```

This request method is used to update a specific policy by name. The response will be a policy object conforming to the pydantic model `Policy`. <br />
Example response body: <br />
```json
{"status": 200, "message": "Updated successfully"}
```

DELETE `/policies/policy_name}` Delete existing policy by name


```bash
curl -X DELETE localhost:8000/policies/{policy_name}
```

This request method is used to delete a specific policy by name
Example response body: <br />

```json
{"status": 200, "message": "Policy deleted successfully"}  
```



