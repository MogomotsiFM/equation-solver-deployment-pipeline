# Equation Solver Deployment Pipeline
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


## Notes
- AWS CodeDeploy cannot read the AppSpec.yml file from the input artifact. The input artifact is the docker_equation_solver repository. We used the hack to read it from the host used to build our Docker image via the secondary artifact of the buildspec.yml file,
- We improved the process used to build our Docker image. We use a multi-stage process. This allows one stage to be used for running unit tests. We install every library we need on this stage. This includes git and OpenSSH, and our application and its unit tests. The other stage is to build the Docker image that is eventually pushed to AWS ECR. The application is copied from the test stage into the production stage,
- The Configuration property of the CodePipeline action is poorly documented,
- 
