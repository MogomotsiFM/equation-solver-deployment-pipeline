# Equation Solver Deployment Pipeline
The idea is to create an AWS CodePipeline to deploy changes to the [equation solver](https://github.com/MogomotsiFM/equation_solver) and [dockerize equation solver](https://github.com/MogomotsiFM/docker_equation_solver) repositories. The first repository contains the actual linear equation solver code. The second one includes a Docker file that packages the equation solver code as an image. The image is meant to be used with an AWS Lambda function. At a high level, we aim to create 
a CodePipeline that has the following resources:
- Integrate GitHub with AWS CodePipeline,
- AWS CodeBuild
    - To build the Docker image,
    - To run unit tests,
    - Push the image to AWS Elastic Container Registry,
    - *Generate the AppSpec file and pipe it to CodeDeploy using output artifacts in the buildspec file*,
- AWS CodeDeploy
  - To deploy a pre-production Lambda function alias that uses the image created above,
  - ~~To deploy an API Gateway pre-production stage~~,
- AWS CodeBuild
  - To run integration tests using Postman,
  - *Generate an AppSpec file and pipe it to CodeDeploy. This is a lesson from the first CodeBuild integration*,
- AWS CodeDeploy
  - To deploy a production Lambda function using a canary deployment strategy,
  - ~~To deploy an API Gateway production stage~~,
  - Add alarms that can trigger rollback in case of failures.


## Notes
- AWS CodeDeploy cannot read the AppSpec.yml file from the input artifact [[1]](https://www.reddit.com/r/aws/comments/12f51k3/an_appspec_file_is_required_but_could_not_be/). The input artifact is the docker_equation_solver repository. We used a "hack" to read it from the host that builds our Docker image via the secondary artifact of the buildspec.yml file,
- We improved the process used to build our Docker image. We use a [multi-stage](https://docs.docker.com/language/java/run-tests/) process. This allows one stage to be used for running unit tests. We install every library we need at this stage. This includes git and OpenSSH, and our application, dependencies, and unit tests. The other stage is to build the Docker image that is eventually pushed to AWS ECR. The application is copied from the test stage into the production stage. This results in a much lighter image,
- ~~The Configuration property of the CodePipeline action is poorly documented~~,
- The integration between CodeBuild and CodeDeploy is not great for Lambda. There is really no need to specify the current and target versions of a Lambda function in the AppSpec file.       - Given an alias, we should be able to deduce the current version. Therefore, we only ever need to specify one of these,
    - Moreover, if the target version is not specified then it should be assumed that we want to create a new version from the artifacts and associate it with the given alias,
- We are using the CodeBuild [buildspec](https://github.com/MogomotsiFM/docker_equation_solver/commit/dc06c867eb99be264f520fcb1fbf7f16877f017a) file to update the function (lambda::UpdataFuntionCode) and publish a new version (lambda::PublishVersion). These functions could, we dare say, should be performed by CodeDeploy for Lambda deployments to improve integration.

## Triggering the pipeline using GitHub
The following are strategies to trigger our code pipeline:
- CodePipeline polls the repository for changes,
- Use Webhooks,
- Use CodeStarConnections.

## Integrating CodeDeploy

## Interlude
At this point, our configuration, which involves piping the AppSpec file from CodeBuild buildspec to CodePipeline, is working. We know this because the CodePipeline error message says that the specified Lambda function does not exist. The Lambda function and API Gateway are not central to CICD. As a result, we create them using the console to unblock ourselves. The following figure shows the state of the pipeline:

![Screenshot 2024-07-26 201139](https://github.com/user-attachments/assets/aedeacbb-3dd1-41ba-970f-b3b97e5f2ba9)


## Adding integration tests
- We use CodeBuild and Postman to run the integration tests,
- We have a couple of unit tests in the equation solver repository that can pass for integration tests. We present them in a format that Postman requires.

  ### Using Postman
  - We created test data in a JSON file [[2]](https://github.com/MogomotsiFM/equation_solver/commit/74f64faa47e1fee5b5d717a47329da00ee381a08),
  - We created a collection using the Postman GUI. This included configuring the API endpoint and keys to access that endpoint. You could also add methods
    to retrieve tokens if your API requires them. We exported this collection so it may be used from the CLI in the CodeBuild host [[3]](https://github.com/MogomotsiFM/docker_equation_solver/blob/main/LinearEquationSolverIntegrationRequireAPIKey.postman_collection.json),
  - We created a CodeBuild [buildspec.yaml](https://github.com/MogomotsiFM/docker_equation_solver/blob/main/buildspec.yml) file that runs the integration tests. It also retrieves the required API key from the AWS System Manager Parameter Store.
 



 
