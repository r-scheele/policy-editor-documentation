Introduction
============

OPAL is the easiest way to keep your solution's authorization layer up-to-date in realtime. By writing policies and authorisation rules for your application, you can ensure synchronization between your policies and your live application, where each user interaction or API call may affect access-control decisions.

This project arise from the need to write authorisation rules using a client application e.g front-end application, and to keep them up-to-date with the application's code. OPAL Policy manager provides a user interface for non-technical users and a REST API to write and manage authorisation rules.

By converting JSON variables to REGO rules, the API ensures the written rules are pushed to a Github/Gitlab repository that houses the company's authorization solution using the Opal server.


![API Image](./img/api.png)