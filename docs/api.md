# Rest API

The application is a secured REST API, protected with an access token generated from either gitlab or github, depending on the repository you are using. Which means that you need to be authenticated to with Gitlab if you are using a Gitlab repository, and with Github if you are using a Github repository.

Using the API is pretty straightforward without a front-end, you just need to send a request to the API endpoint, with the access token in the header. The access token is generated from the Gitlab or Github account you are using to access the repository.

If you prefer to access the API using the swagger interface, you need to paste the access token in the top right corner of the swagger interface(see image below).

![Authorize button on the swagger ui](./img/authorize.png)

## Authentication

GET `/gitlab/token` - Get a Gitlab access token
GET `/github/token` - Get a Github access token

These endpoints are only needed if your application has a front-end. They will redirect you to the Gitlab or Github authentication page, and then redirect you back to the application with a valid access token.

## Policy CRUD Operations

Please visit the `/redoc` endpoint to see the API documentation.

## Repository Management

### Front-end use case

This section contains the endpoints for managing the repositories used by the application. Depending on the sign in method used by the user, repositories from the token provider can be accessed as a list.

GET `/user/repo/github` <br />

GET `/user/repo/gitlab`

This endpoint is used to select which repo the user wants to use. The response will be a list of all the repositories that the user has access to.

### Backend use case

If the API is used directly, the repository to use for writing the policy is supplied as a url, in the request body
For Gitlab provider, identifying the repository for writing the policy requires both the url and the repo id. The repo is retrieved from the response body of the GET request to the Gitlab API `/user/repo/gitlab`.

```json
{
  "repo_url": "https://gitlab.com/youngestdev1/policies/opal-server",
  "repo_id": 39020791,
   ...

}
```
## Database Operations

GET `/data`

This endpoint retrieves the groupnames stored as part of the usergroups needed in the rego policy creation, it is used to accurately populate the dropdown menu in the frontend. The response will be a list of all the groupnames that have been stored in the database. <br />


Example response body: <br />
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
