# Architecture

The API is written in Python and FastAPI framework, in a way that it is easy to use, extend, test and maintain.

![Screenshot 2022-06-13 at 09 55 43](https://user-images.githubusercontent.com/67229938/173343113-d51d72b4-84c8-4c3b-8555-af41e59cd2de.png)


## Translator Logic

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
```py3
List[Rule] == Allow {
...
}
```

The API supports the following commands; input_prop_equals, input_prop_in, input_prop_in_as, allow_full_access.



### input_props_equals

This command has different logic to handle series of equality checks.

- Handling '*' as the wildcard flag: <br /> 

This logic handles all the paths after a particular section. if `/collections/` is supplied as the option, all the routes after it will be allowed e.g allow `/collections/obs/`, allow `/collections/test-data/obs/`, allow `/collections/obs/` etc. <br /> 

Example:
```json     
{
   "command": "input_prop_equals",
   "properties": {
      "input_property": "request_path",
      "value": ["v1", "collections", "*"]
   }
}
```
In the json above, the `value` key holds an asterik in the values section, to indicate that the endpoint x is allowed to do y.

The REGO equivalent of the above rule object is: <br />
```rego
input.request_path[0] == "v1"
input.request_path[1] == "collections"
```

- Allowing all path parameter after a path section, except one: <br /> 

This logic handles cases where a particular path parameter is to be exempted, the command matches all other parameters aside the exempted one. e.g allow `/collections/obs/`, allow `/collections/test-data/obs/`, allow `/collections/obs/`. deny `/collections/lakes/`. <br /> 

Example:

```json
{
   "command": "input_prop_equals",
   "properties": {
      "input_property": "request_path",
      "value": ["v1", "collections", "*"],
      "exceptional_value": "obs",
   },
}
```
In the json above, the `value` key holds an asterik in the values section, to indicate that the endpoint x is allowed to do y. The `exceptional_value` key holds the value of the path parameter that is to be exempted.

The REGO equivalent of the above rule object is: <br />
```rego
input.request_path[0] == "v1"
input.request_path[1] == "collections"
input.request_path[2] != "obs"
```

-  Handling equality check between a particular property on the request object and a value: <br />
This logic handles cases where a particular property on the input object is to be checked for equality against a value. 
Example:

```json
{
   "command": "input_prop_equals",
   "properties": {
      "input_property": "company",
      "value": "Geobeyond srl"
  }
}
```
In the json above, the `value` key holds the value that is to be checked for equality, while the `input_property` key holds the property that is to be checked in the input object.

The REGO equivalent of the above rule object is: <br />
```rego
input.company == "Geobeyond srl"
```

-  Allow access to a particular path: <br />
This logic handles cases where a specific path is to granted access, if certain property is present on the input object.

Example:
```json
[
{
   "command": "input_prop_equals",
   "properties": {
      "input_property": "request_path",
      "value": ["v1", "collections", "lakes"],
   },
},
   {
   "command": "input_prop_equals",
   "properties": {
      "input_property": "request_path",
      "value": "admin",
   },
}
]
```
In the JSON above, the list of objects indicates a very special `allow` block, that combines two commands. The `input_property` key holds the property that is to be validated before the request to that path is passed. 

The REGO equivalent of the above rule object is: <br />
```rego
allow {
   input.request_path == ["v1", "collections", "lakes", ""]
   input.groupname == "admin"
}
```

### input_props_in

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

### allow_if_object_in_database

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


### allow_full_access

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
## Database integration

There are times where the authorization rules are dependent on data from a database. For example, a user can only access a resource if they are in a specific group. In this case, the group data is stored in a database. The API supports the use of databases by providing a datasource command that fetches the structure of the data present in the database and creates a REGO rule that can be used to access the data. 

```
SELECT DISTINCT groupname AS value FROM geostore.gs_usergroup
```

The above query is an example of a query that can be used to fetch the group names from the database. The query is written in SQL and is sent to the API as a string. The API then creates a rule that can be used to access the data, and these rules are then added to the REGO policy file. 

```json
{
  "geostore.usergroup": [
    "EDITOR_DPAU",
    "EDITOR_CPQ",
    "VIEWER",
    "everyone",
    "EDITOR_ATAC",
    "TestGeocity",
    "EDITOR_AMBIENTE",
    "GEOCITY_ADMINS",
    "EDITOR_SIZA",
    "ROLE_SYS_GWCS",
    "EDITOR_SITPAU",
    "EDITOR_DBGT",
    "ctr-AltimetriaLineeCTRN-VIEW",
    "ctr-AltimetriaLineeCTRN-EDIT",
    "EDITOR_NIC",
    "ambiente-AsparmRaccoltaFarmaci-EDIT",
    "EDITOR_URBANISTICA",
    "EDITOR_CITTA_PUBBLICA",
    "EDITOR_VINCOLI",
    "EDITOR_COMPLETO"
  ]
}
```

An example of rego policy that is dependent on a database is shown below. The `geostore.usergroup` is the name of the datasource and the `data` is the name of the variable that holds the data. The `data` variable is used to access the data in the datasource. 

```rego
allow {
  data.geostore.usergroup[_] == "EDITOR_DPAU"
}

```
This rule checks if the user belongs to the group `EDITOR_DPAU`