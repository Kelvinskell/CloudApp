# CloudApp
An Serverless, Cloud-Native application that provisions pre-configured EC@ instances to authenticated users.

This application is a Cloud-Native app, built using a serverless three-tier architecture. It is also built on AWS Cloud Infrastructure and is not designed to be cloud agnostic.

## Presentation Tier
The frontend is a Python Flask Web Application deployed in an **ECS Cluster.**
### Technical design
- The Frontend accepts user credentials and supplies them to a RESTful **API Gateway**
- The API Gateway invokes an **Express Step Function.**
- The frontend also exposes an _Admin_ dashboard for viewing statistics and performimg administrative actions.

This layer can only access the logic tier but not the Data tier.

## Logic Tier - Phase 1
The logic tier is where the core of the application resides. Phase 1 of this tier can only be accessed by the Presentation tier.

This tier also is also the only tier that can directly access the database - which resides in the Data tier.
### Technical design
- The **Step Function** calls an "auth" **lambda function** to process the credentials passed to it by the Presentation tier.
- This lambda function connects with the Data tier and checks the credentials against a user database. Based on the checks, this function will either return _"True"_ or _"False"._
  - If _False_, the **Step Function** will return an "Error" message to the Flask App in the Presentation Tier and then exits.
  - The presentation Tier then displays an "Authentication Error" message to the user.
  - If _True_, a "Success" message is returned.
    - A lambda function sends an SNS message to the Admin, about a successful login. 
    - The flask application in the presentation tier then displays an authentication message to the user.
    - The **Step Function** exits.
    
 ## Logic Tier - Phase 2
 This is the second part of the logic tier and implements the core functionalities of this application.
 
 This phase is made up of an **EKS Cluster** which exposes three (3) services using a Microservices architecture.
 
 The Presentation tier connects to this tier if user authentication from the previous step is successful.
 ### technical design
 - Three microservices are implemented at this phase of the Logic tier.
   - **create-user service:** This service is responsible for creating users.
     - It interacts with the Data tier to create and store the user credentials.
     - This service can only be accessed through the admin page in the Presentation layer - hence, access is restricted only to Admins.
   - **provision-instance service:** This service provisons **EC2 instances**.
     - It will provision an **Ec2 instance** complete with all necessary installations and configurations.
     - It then return login details for the instance back to the Presentation layer - which will display this to the user.
     - Instance details and user-details will be uploaded to an **S3 bucket** where they will be analysed by another service.
     - It will also send an SNS Email notification to the Admin, about instance creation.
   - **view-stats service:** This service provides a dashboard for admins to 
     - 
       

