# CloudApp
An Serverless, Cloud-Native application that provisions pre-configured EC@ instances to authenticated users.

This application is a Cloud-Native app, built using a serverless three-tier architecture. It is also built on AWS Cloud Infrastructure and is not designed to be cloud agnostic.

## Presentation Tier
The frontend is a Python Flask Web Application deployed in an **ECS Cluster.**
### Technical implementation
- The Frontend accepts user credentials and supplies them to a RESTful **API Gateway**
- The API Gateway invokes an **Express Step Function.**
- The frontend also exposes an _Admin_ dashboard for viewing statistics and performimg administrative actions.

This application can only access the logic tier but not the Data tier.

## Logic Tier - Phase1
The logic tier is where the core of the application resides. This tier can only be accessed by the Presentation tier.

This tier also is also the only tier that can directly access the database - which resides in the Data tier.
### Technical implementation
- The **Step Function** calls an "auth" **lambda function** to process the credentials passed to it by the Presentation tier.
- This lambda function connects with the Data tier and checks the credentials against a user database. Based on the checks, this function will either return _"True"_ or _"False"._
  - If _False_, the **Step Function** will return an "Error" message to the Flask App in the Presentation Tier and then exits.
  - The presentation Tier then displays an "Authentication Error" message to the user.
  - If _True_, a "Success" message is returned and the **Step Function** exits.
    - The flask application in the presentation tier then displays an authentication message to the user.

