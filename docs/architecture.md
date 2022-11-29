# Architecture

## Application structure

The API is written in Python and FastAPI framework, in a way that it is easy to use, extend, test and maintain.

![Screenshot 2022-06-13 at 09 55 43](https://user-images.githubusercontent.com/67229938/173343113-d51d72b4-84c8-4c3b-8555-af41e59cd2de.png)


## Application components

The application is composed of the following components:

- **SERVER**: The SERVER is the main component of the application. It is responsible for handling network requests and responses between the application and other components of the application(API component and services). It is also responsible for handling the interaction between the application and other third party APIs such as GitHub, Gitlab, etc.

The application uses Github/Gitlab to authenticate users and to get the user's repositories. The application also uses Github/Gitlab to commit and push the changes to the user's repository.

- **Utility Functions**: The utility functions are responsible for the logic of the application. They are responsible for handling the conversion from JSON to REGO with command mapping, writing the converted rules into a file, and getting the file ready for commit and push to the user's repository.

- **Schemas**: The schemas are responsible for the data validation of the application. They are responsible for validating the data that is sent to the application and the data that is received from the application. They ensure that the JSON is in the correct format and that it is valid, in order to avoid any errors in the REGO conversion.

- **Databases**: The application uses two different databases.

The first database stores the state of the REGO rules for future CRUD operations, after the initial write. It uses TinyDB to store the state of the rules.
The second database is a POSTGRES database. There are times where the authorization rules are dependent on data from a database. For example, a user can only access a resource if they are in a specific group in the database. The API supports the use of databases by providing a datasource command that fetches the structure of the data present in the database and creates a REGO rules that depends on data from the postgres. 

The SQL query that fetches this group information is below;

```sql
SELECT DISTINCT groupname AS value FROM geostore.gs_usergroup
```

This query is executed by the API as a string and the API creates rules that depends on its result.

result;

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

An example of rego policy that is dependent on the group information in the postgres database is shown below. 
The `geostore.usergroup` is the name of the datasource and the `data` is the name of the variable that holds the data. To access the data from the database, the schema name + the table name must be prefixed with `data.`.

`data.geostore.usergroup` = `data.schema_name.table_name`

```rego
allow {
  data.geostore.usergroup[_] == "EDITOR_DPAU"
}

```

This rule checks if the user in the request belongs to a group called `EDITOR_DPAU`. If the user belongs to the group, the rule returns true and the user is allowed to access the resource. If the user does not belong to the group, the rule returns false and the user is not allowed to access the resource.