# CloudApp
An Serverless, Cloud-Native application that provisions pre-configured EC2 instances to authenticated users.

This application is a Cloud-Native app, built using a serverless three-tier architecture. It is also built on AWS Cloud Infrastructure and is not designed to be cloud agnostic.

The three-tier acrchitectuiire comprises of the **Presentation Tier, Logic Tier and Application Tier.**

## Presentation Tier
The frontend is a Python Flask Web Application deployed in an **ECS Cluster** on Fargate.
### Technical Design
- User creation, authetication and deletion is handled by **Amazon Cognito.**
- **Amazon Cognito** is integrated with a post-authentication trigger which calls an "auth" **Lambda Function** from the Logic tier for further processing of the JSON Web Tokens (JWT) and persisting the data in the Data Tier if a new user is created.
- If authentication is successful, a **Step Function** is invoked and a workflow begins in the Logic Tier.
- The frontend also exposes an _Admin_ dashboard for viewing statistics and performimg administrative actions.
  - The admin page contains other implementations such as users dashboard, instance configuration dashboard, application usage statistics dashboard and Stop-instance button.
     1. The _instance-configuration_ dashboard is used to define configuration details for EC2 Instances. 
       - Instructions are sent to a "config" **SQS Queue** from where they are read from a "config" **Lambda Function** in the Logic tier and persisted in the config database.
    2. The _users dashboard_ is used to view number of active users, to create new users who can authenticate and access the application or delete users from the database.
       - Interaction with **Amazon Cognito** helps to achieve this.
    3. The _application usage statistics_ dashboard interacts with the _view-stats_ service in the Logic Tier.

This layer can only access the Logic tier but not the Data tier.

## Logic Tier - Phase 1
The logic tier is where the core of the application resides. Phase 1 of this tier can only be accessed by the Presentation tier.

This tier also is also the only tier that can directly access the database - which resides in the Data tier.
### Technical Design
- The **Step Function** calls an "auth" **Lambda Function** to process the credentials passed to it by the Presentation tier.
  - The **Step Function** uses a Choice state to detr 
    - A "config" **Lambda Function** will check the  "config" **SQS Queue** for any new messages.
      - If present, the function updates the "config" database with the latest configuration instructions.
    - The **Step Function** exits.
  - This phase also implements a **lambda Function** that will be executed when the "Stop-instance" button in the Admin page is clicked.
    - This function will abruptly shutdown the specified EC2 instance.
    
 ## Logic Tier - Phase 2
 This is the second part of the logic tier and implements the core functionalities of this application.
 
 This phase is made up of an **EKS Cluster** on AWS Fargate which exposes three (3) services using a Microservices architecture.
 
 The Presentation tier connects to this tier if user authentication from the previous step is successful.
 ### Technical Design
 - Three microservices are implemented at this phase of the Logic tier.
   - **user-service:** This service is responsible for creating users.
     - It interacts with the Data tier to create and store the user credentials.
     - Displays the total number of users and their details.
     - Can also delete users from the database.
     - This service can only be accessed through the admin page in the Presentation layer - hence, access is restricted only to Admins.
   - **provision-instance service:** This service provisons **EC2 instances**.
     - It will provision an **Ec2 instance** complete with all necessary installations and configurations.
     - It reads configuration instructions from the database in the Data Tier and provisions the instance accoridng to specified configurations.
     - Instance details and user-details will be uploaded to a designated **S3 bucket** where they will be analysed by another service.
     - It will also send an SNS Email notification to the Admin, about instance creation.
     - Instance login details are then returned to the Presentation Tier - where they are displayed to the User.
   - **view-stats service:** This service provides a dashboard for admins to view and analyse application state.
     - Uses **Amazon Athena** to query the designated **S3 bucket** for data analysis and returns statistics to the Presentation tier.
     - This service can only be accessed through the Admin page in the presentation tier.
   
  - The **EKS Cluster** uses an **AWS Load Balancer Controller** to expose the microservices and to perform path-based routing.
  - **Helm** is utilised to package the Kubernetes manifests.
  
  ## Data Tier
  **Amazon Aurora Serverless** is used as the user database for the application. This database resides in the Data-tier and can only be directly accessed by the Logic tier. Consists of two tables:
  - **User Table:** All valid user credentails are stored here.
  - **Config Table:** All valid instance configuration instructions are stored here.
  
  ## Extras
  - **Terraform scripts** for provisioning AWS Cloud Infrastructure.
   
       

