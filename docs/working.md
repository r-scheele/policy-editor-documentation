
# Working principles

fastapi-opa is an extension to FastAPI that allows you to add a login flow to your application within minutes using open policy agent and your favourite identity provider.

With this extension, you can authenticate your users using your favourite identity provider and then use the user information to make authorization decisions using OPA.

To show the power of this extension, we will use the following example, where we use the rego equivalent of the following JSON policy, to authorize user access to some endpoints in an application.



JSON request:

```json
    {
        "command": "input_prop_equals",
        "properties": {
            "input_property": "request_path",
            "value": "v1/collections/{username}",
        },
    }

```

Our API response:

```rego

package httpapi.authz
import input
default allow = false



allow {
  some username  
  input.request_path = ["user", username] 
  input.preferred_username = username  
  
}

```

Consider a simple FastAPI application, that uses Keycloak identity provider and Opa to make authorization decisions. 

We'll use the example rego result above to show how to use fastapi-opa to add a login flow to your application.

The policy decision in `policy/auth.rego` authorizes the user to access the endpoint if they're logged as the username in the path.

Start by cloning the repository and installing the dependencies:
  
```bash
  git clone https://github.com/r-scheele/test-api.git
```

```bash
  cd test-api
```

```bash
  poetry shell && poetry install
```

Start all the services (Keycloak and Opa) using docker-compose:

```bash
  docker-compose up -d
```

Start the application:

```bash
  uvicorn main:app --port 8000 --reload
```

The application is now running on http://localhost:8000.

### Configure postman for testing the rules

1. Import the already configured postman collection and environment variables to test the application:
`test-api.postman_collection.json` and `test-api.postman_environment.json`

2. In the login request, open the test tab and paste the following code:

```js

var response = JSON.parse(responseBody);
postman.setEnvironmentVariable("refresh_token", response.refresh_token);
postman.setEnvironmentVariable("access_token", response.access_token);
postman.setEnvironmentVariable("session_state", response.session_state);

```

### Send request to keycloak to get access token

POST `{{keycloak_server}}/auth/realms/{{realm}}/protocol/openid-connect/token`

if you're using the `test-api.postman_environment.json`, keycloak_server and realm is already set.

returns - access token 


### Send request to the application test endpoints in the REGO rules.

GET `/user/habeeb` - Accessible

response;

```json
{
    "message": "Hello World from test1"
}
```

GET `/user/john` - Inaccessible

response;

```json 
{
    "message": "Unauthorized"
}
```


GET `/user/{username}/test` - Inaccessible 

response;

```json 
{
    "message": "Unauthorized"
}
```

