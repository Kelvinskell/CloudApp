# CloudApp
An Serverless, Cloud-Native application that provisions pre-configured EC2 instances to authenticated users.

This application is a Cloud-Native app, built using a serverless three-tier architecture. It is also built on AWS Cloud Infrastructure and is not designed to be cloud agnostic.

The three-tier acrchitectuiire comprises of the **Presentation Tier, Logic Tier and Application Tier.**

## Presentation Tier
The frontend is a Python Flask Web Application deployed in an **ECS Cluster** on Fargate.

This layer can only access the Logic tier but not the Data Tier.

### Technical Design
- User creation, authetication and deletion is handled by **Amazon Cognito.**
- **Amazon Cognito** is integrated with a post-authentication trigger which calls an "auth" **Lambda Function** for further processing of the JSON Web Tokens (JWT) and sends an **SNS Email Notification** to the Admins - when a new user is created.
- The frontend exposes a _Regular user_ page and an _Admin user_ page.
- The _Admin user_ page requires additional authentication for access and is used for viewing application statistics and performing administrative actions.
- The admin page contains implementations such as users dashboard, instance configuration dashboard, application usage statistics dashboard and Stop-instance button.
     1. The _instance-configuration_ dashboard is used to define configuration details for EC2 Instances. 
       - This is achived by interacting with the _configure instance_ service in the Logic tier.
    2. The _users dashboard_ is used to view number of active users, to create new users who can authenticate and access the application or delete users from the database.
       - Interaction with **Amazon Cognito** helps to achieve this.
    3. The _application usage statistics_ dashboard interacts with the _view-stats_ service in the Logic Tier
       - The "Stop Instance" button is accessible from this dashboard and can be used by the Adminuser to force-stop any running instance.
- The _Regular user_ page contains a "Create Instance" button and a "Stop instance" button.
  - Both of these buttons invoke the _provision-instance service_ in the Logic Tier.
  - The service willl either create or destroy an **Ec2 Instance** depending on the arguments passed.
  
- The Presentation layer then performs a GET Operation to an API Gateway to determine the status of the execution.


## Logic Tier - Phase 1
The first phase of this tier contains a series of lambda functions that help to streamline application workflow.

This tier also is also the only tier that can directly access the database - which resides in the Data tier.
### Technical Design
- A **Lambda Function** is implemented here which polls the "login" **SQS Queue** in the phase 2 for execution status.
  - The message from the **SQS Queue** is then passed over to the **API Gateway** in the presentation Tier.
- This phase implements a "stop-ec2" **Lambda Function** that will be executed when the "Stop-instance" button in the Admin page is clicked.
  - This function will abruptly shutdown the specified EC2 instance.
- This phase also implements an "auth" **Lambda Function** which is used by **Amazon Cognito** to perform post authentication steps (Send **SNS Notification**).
    
 ## Logic Tier - Phase 2
 This is the second part of the logic tier and implements the core functionalities of this application.
 
 This phase is made up of an **EKS Cluster** on AWS Fargate which exposes three (3) services using a Microservices architecture.
 
 The Presentation tier connects to this tier if user authentication from the previous step is successful.
 ### Technical Design
 - Three microservices are implemented at this phase of the Logic tier.
   - **configure-instance service:** This service is responsible for setting instance configurations.
     - It defines configuration details for EC2 instances and persists them in the database residing in the Data Tier.
     - This service can only be accessed through the admin page in the Presentation layer - hence, access is restricted only to Admins.
     
   - **view-stats service:** This service provides a dashboard for admins to view and analyse application state.
     - Uses **Amazon Athena** to query the designated **S3 bucket** for data analysis and returns statistics to the Presentation tier.
     - This service can only be accessed through the Admin page in the presentation tier.
     
   - **provision-instance service:** This service provisons **EC2 instances**.
     - It will provision an **Ec2 instance** complete with all necessary installations and configurations.
     - It reads configuration instructions from the database in the Data Tier and provisions the instance accoridng to specified configurations.
     - Instance details and user-details will be uploaded to a designated **S3 bucket** where they will be analysed by the _view-stats_ service_ service.
     - It will also send an SNS Email notification to the Admin, about instance creation.
     - Instance login details are then sent to a "login" **SQS Queue** where they will be polled by the Presentation Tier - and displayed to the User.
    
   
   
  - The **EKS Cluster** uses an **AWS Load Balancer Controller** to expose the microservices and to perform path-based routing.
  - **Helm** is utilised to package the Kubernetes manifests.
  
  ## Data Tier
  **Amazon Aurora Serverless** is used as the user database for the application. 
  
  This database resides in the Data-tier and can only be directly accessed by the Logic tier.
  
  **All valid instance configuration instructions are stored here.**
  
  ## Extras
  - **Terraform scripts** for provisioning AWS Cloud Infrastructure.
   
       

