# CloudApp
An Serverless, Cloud-Native application that provisions pre-configured EC@ instances to authenticated users.
This application is a Cloud-Native app, built using a serverless three-tier architecture.

## Presentation Tier
The frontend is a Python Flask Web Application deployed in an **ECS Cluster.**
### Technical implementation
- The Frontend accepts user credentials and supplies them to a RESTful **API Gateway**
- The API Gateway invokes an **Express Step Function.**
- The frontend also exposes an _Admin_ dashboard for viewing statistics and performimg administrative actions.

