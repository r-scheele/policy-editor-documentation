
# JSON to REGO conversion Logic

The REGO policies are created by a set of functions called **commands** that serves as the translation logic. Each of these commands is responsible for writing a specific REGO rule to a `.rego` file. However, the JSON to REGO conversion must follow certain conditions as defined by a pydantic model for effectiveness. 
The following is how a rule is defined. The  Policy object which is the request body, as well as the Rule object is pydantic models.

```py3
class Policy:
name: str
rules: List[List[Rule]]
```

Rule Object: <br />

```py3
class Rule(BaseModel):
command: str
properties: Dict[str, Union[str, List[str], Dict[str, Union[str, List[str]]]]]
```

A rule is defined by two keys: command and properties. The command key holds one of the recognized commands and the properties key, holds another dictionary containing the input to the command function e.g `input_property` and `value`. in special cases, the `datasource_item` items are also included in the properties key.

```json     
{
   "command": "input_prop_equals",
   "properties": {
      "input_property": "request_method",
      "value": "GET"
   }
}
```

In the example above, the command key represents the operation to be performed and the properties key represents the properties that are being used in the operation.

`input_prop_equals` is the command in the example that initiates the appropriate operation, on the properties object.
The above rule translates to an equality check between the input property `input_property` and the value `value`. <br />

The REGO equivalent of the above rule object is: <br />

```rego
`input.request_method == "GET"`
```

Each Rule object forms a specific rule in a Allow block, and a list of Rules forms a Allow block. <br />


The API supports the following commands; input_prop_equals, input_prop_in, input_prop_in_as, allow_full_access.



## input_props_equals

Path authorization is the most important feature of the API

   1. Allow users the access to all the paths, except a particular path variable. 

      Request body:

      ```json
      {
         "command": "input_prop_equals",
         "properties": {
            "input_property": "request_path",
            "value": "/users/*",
            "exceptional_value": "payment",
         }
      }
      ```

      Result:

      ```rego

      input.request_path[0] = "users
      input.request_path[1] != "payment"
      ```

      With the above rule, all the request paths, that starts with `/users` will be allowed, except the payment.

      Example:

      `/users/feed`  - returns Authorised
      `/users/posts/comment`  - returns Authorised

      `/users/payment`  - returns Unauthorised


   2. Allow only access to request path prefixed with`/users` only
      Request body:
      ```json
      {
         "command": "input_prop_equals",
         "properties": {
               "input_property": "request_path",
               "value": "/users/*",
         },
      }
      ```

      Result:
      ```rego
      input.request_path[0] = "users

      ```

      Example:

      `/users/followers`  - returns Authorised
      `/users/feeds/posts`  - returns Authorised

      `/payment`  - returns Unauthorised

   3. Allow only users access to their own routes. A log in with the username `Habeeb`, should have access to request path prefixed with`/users/Habeeb` and not `/users/John`

      Request body:
      ```json
      {
         "command": "input_prop_equals",
         "properties": {
            "input_property": "request_path",
            "value": "v1/collections/{username}/*",
         },
      }
      ```

      Result:
      ```rego

      some username
      input.request_path[0] = "users
      input.request_path[1] = "username

      ```
      With the above rule, all the request paths, that starts with `/users/{username}` will be allowed

      Example:
      Say the username from an identity provider is habeeb

      `/users/habeeb/profile`  - returns Authorised
      `/users/habeeb/posts`  - returns Authorised

      `/users/john/posts`  - returns Unauthorised


   4. These rules targets a specific path. Only a certain path will be allowed, while every other path is rejected.

      Request body:

      ```json
      {
         "command": "input_prop_equals",
         "properties": {
               "input_property": "request_path",
               "value": "users/payment",
         },
      }
      ```

      Result:

      ```rego
      input.request_path = ["users", "payment"]

      ```

      or 

      ```json
      {
            "command": "input_prop_equals",
            "properties": {
               "input_property": "request_path",
               "value": "users/{username}",
            },
      }
      ```

      Result:

      ```rego

      some username
      input.request_path = ["users", username]
      input.preferred_username = username

      ```

      Example:
      Say the username from an identity provider is habeeb
      `/users/habeeb`  - returns Authorised
      `/users/habeeb/profile`  - returns Unauthorised

      `/users/payment`  - returns Authorised
      `/users/feeds`  - returns Unauthorised

   5. Allow users the access to all the paths prefixed with the username, except a particular path variable. 

      Request body:

      ```json
         {
            "command": "input_prop_equals",
            "properties": {
                  "input_property": "request_path",
                  "value": "/users/{username}/*",
                  "exceptional_value": "chat",
            }
         }
      ```

      Result:

      ```rego
      some username
      input.preferred_username = username
      input.request_path[0] = "users
      input.request_path[1] = username
      input.request_path[2] = "chat"
      ```

      With the above rule, all the request paths, that starts with `/users/{username}` will be allowed, except `/users/{username}/chat`


      Example:
      `/users/habeeb/feed`  - returns Authorised
      `/users/habeeb/posts`  - returns Authorised

      `/users/habeeb/payment`  - returns Unauthorised

      Note that a couple of these rules can be combined to achieve your authorization rules, and it is not limited to example given here.

## input_props_in

This logic checks if a particular property on the input object is present in a list of values from the database. <br />

Example:

```json
{
   "command": "input_prop_in",
   "properties": {
      "input_property": "groupname",
      "datasource_name": "usergroups",
      "datasource_loop_variable": "name"
   }
}
```
In the json above, the `input_property` key holds the property that is to be validated before the request to that path is passed. The `datasource_name` key holds the name of the datasource(a list) from the database. The `datasource_loop_variable` key holds the name of the key on each object in the datasource.

The rego rules combine data from the database with the input object, to work out certain conditions. The datasource is the list of values from the database, and the `datasource_loop_variable` is the name of the key on each object in the datasource.

The REGO equivalent of the above rule object is: <br />

```rego
input.groupname == data.usergroups[i].name
```

## allow_if_object_in_database

This logic checks if the value of two properties on the input object is present on one object in the database <br />

Example:
```json
{
   "command": "allow_if_object_in_database",
   "properties": {
      "datasource_name": "usergroups",
      "datasource_variables": ["name", "groupname"],
   }
}
```

In the JSON above, the resulting REGO code loops over the datasource twice, checking for equality between the values of the input properties, and the values of the datasource loop variables.

The REGO equivalent of the above rule object is: <br />
```rego
{ name: input.name, groupname: input.groupname} == data.usergroups[_]
```


## allow_full_access

This logic allows full access to the resources defined. If the value of the property on the input object has a particular value<br />

Example:
```json
[
   {
      "command": "allow_full_access",
      "properties": {
         "input_property": "groupname",
         "value": "EDITOR_ATAC",
      },
   }
]
```
In the JSON above, the result is an allow block that allows access to the resources defined if the value of the property on the input object has a particular value.

```rego
allow {
   input.groupname == "EDITOR_ATAC"
}
```