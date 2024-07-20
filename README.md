# Equation Dolver Deployment Pipeline
The idea is to create an AWS CodePipeline to deploy changes to the equation_solver and dockerize_equation_solver repositories. At a high level, we aim to create 
a CodePipeline that has the following resources:
- Integrate GitHub with AWS CodePipeline,
- AWS CodeBuild
    - To build the Docker image,
    - To run unit tests,
    - Push the image to AWS Elastic Container Registry,
- AWS CodeDeploy
  - To deploy a pre-production Lambda function that uses the image created above,
  - To deploy an API Gateway pre-production stage,
  - To run integration tests,
- AWS CodeDeploy
  - To deploy a production Lambda function using a canary deployment strategy,
  - To deploy an API Gateway production stage.


