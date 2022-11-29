
# Why do we need this?

When a user sends a request to an endpoint, He/She is authenticated by an identity provider e.g Keycloak, AWS IAM, or Okta. The identity provider provides the user with an access token that contains information about the user. The access token is then sent to the application and the application verifies the token and extracts the user information. 

The application simply provides an API to write the most common policy decisions for your application, by Transforming the JSON policy to REGO. The developer can focus on writing the policy and not on the technical details of REGO.

![API Image](./img/fastapi-opa.png)

The application then uses the user information to decide whether to allow or deny the request. Writing these decisions requires knowledge of REGO policy language, which is a purpose-built declarative policy language that supports Open Policy Agent (OPA). It is used to write authorization policy allowing OPA to make access control decisions.

