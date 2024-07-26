# Equation Solver Deployment Pipeline
The idea is to create an AWS CodePipeline to deploy changes to the equation_solver and dockerize_equation_solver repositories. At a high level, we aim to create 
a CodePipeline that has the following resources:
- Integrate GitHub with AWS CodePipeline,
- AWS CodeBuild
    - To build the Docker image,
    - To run unit tests,
    - Push the image to AWS Elastic Container Registry,
    - ~Generate the AppSpec file and pipe it to CodeDeploy using output artifacts in the buildspec file~,
- AWS CodeDeploy
  - To deploy a pre-production Lambda function alias that uses the image created above,
  - ~~To deploy an API Gateway pre-production stage~~,
- AWS CodeBuild
  - To run integration tests,
  - Integrate with Postman,
  - Generate an AppSpec file and pipe it to CodeDeploy. This is a lesson from the first CodeBuild integration,
- AWS CodeDeploy
  - To deploy a production Lambda function using a canary deployment strategy,
  - To deploy an API Gateway production stage.


## Notes
- AWS CodeDeploy cannot read the AppSpec.yml file from the input artifact. The input artifact is the docker_equation_solver repository. We used a "hack" to read it from the host that builds our Docker image via the secondary artifact of the buildspec.yml file,
- We improved the process used to build our Docker image. We use a [multi-stage](https://docs.docker.com/language/java/run-tests/) process. This allows one stage to be used for running unit tests. We install every library we need at this stage. This includes git and OpenSSH, and our application, dependencies, and unit tests. The other stage is to build the Docker image that is eventually pushed to AWS ECR. The application is copied from the test stage into the production stage. This results in a much lighter image,
- ~~The Configuration property of the CodePipeline action is poorly documented~~,
- The integration between CodeBuild and CodeDeploy is not great for Lambda. There is really no need to specify the current and target versions of a Lambda function in the AppSpec file.       - Given an alias, we should be able to deduce the current version. Therefore, we only ever need to specify one of these,
    - Moreover, if the target version is not specified then it should be assumed that we want to create a new version from the artifacts and associate it with the given alias,
- We are using the CodeBuild [buildspec](https://github.com/MogomotsiFM/docker_equation_solver/commit/dc06c867eb99be264f520fcb1fbf7f16877f017a) file to update the function (lambda::UpdataFuntionCode) and publish a new version (lambda::PublishVersion). These functions could, we dare say, should be performed by CodeDeploy for Lambda deployments to improve integration.

## Triggering the pipeline using GitHub
- CodePipeline polls the repository for changes,
- Use Webhooks,
- Use CodeStarConnections.

## Integrating CodeDeploy

### Interlude
At this point, our configuration, which involves piping the AppSpec file from CodeBuild buildspec to CodePipeline, is working. We know this because the CodePipeline error message says that the specified Lambda function does not exist. The Lambda function and API Gateway are not central to CICD. As a result, we create them using the console to unblock ourselves. 

## Adding integration tests
- We use CodeBuild and Postman to run the integration tests,
- We have a couple of unit tests that can pass for integration tests. We copy them into a new integration tests folder in the equation solver repository.
