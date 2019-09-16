# AWS SAM CodePipeline CD

This serverless app sets up an AWS CodePipeline Pipeline as a CD solution for a GitHub-based SAM project. Once setup, every time the specified GitHub repository branch is updated, the change will flow through the CodePipeline pipeline.

## Architecture

The pipeline has the following 5 stages:
1. **Source**: This stage is the entry point of the pipeline. It is triggered by changes from a GitHub respository branch.
1. **Build**: This stage builds the project using CodeBuild.
1. **Test** (optional): This stage runs the integration tests of the project using CodeBuild.
1. **Deploy** (optional): This stage deploys the project using CloudFormation.
1. **Publish** (optional): This stage publishes the project to AWS Serverless Application Repository using the publish [app](https://serverlessrepo.aws.amazon.com/applications/arn:aws:serverlessrepo:us-east-1:077246666028:applications~aws-serverless-codepipeline-serverlessrepo-publish)

Here is an example CodePipeline pipeline that has all 5 stages:
![aws-sam-codepipeline-cd-pipeline-example](https://github.com/awslabs/aws-sam-codepipeline-cd/raw/master/images/aws-sam-codepipeline-cd-pipeline-example.png)

## Installation

1. [Create an AWS account](https://portal.aws.amazon.com/gp/aws/developer/registration/index.html) if you do not already have one and login
1. Create a GitHub OAuth token (see instructions below).
1. Go to this app's page on the [Serverless Application Repository](https://serverlessrepo.aws.amazon.com/applications/arn:aws:serverlessrepo:us-east-1:646794253159:applications~aws-sam-codepipeline-cd) and click "Deploy"
1. Provide the required app parameters and click "Deploy"

### Creating a GitHub OAuth token

General instructions for creating a GitHub OAuth token can be found [here](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/). When you get to the scopes/permissions page, you should select the "repo" and "admin:repo_hook" scopes, which will automatically select all permissions under those two scopes.

![GitHub OAuth Token Permissions](https://github.com/awslabs/aws-sam-codepipeline-cd/raw/master/images/github-token-permissions.png)

## Parameters

The app has the following parameters:

1. `GitHubOwner` (required) - GitHub username owning the repo.
1. `GitHubRepo` (required) - GitHub repo name (just the name, not the full URL).
1. `GitHubOAuthToken` (required) - OAuth token used by AWS CodeBuild to connect to GitHub.
1. `GitHubBranch` (optional) - GitHub repo branch name. Default: master
1. `ComputeType` (optional) - AWS CodeBuild project compute type to use. See [the documentation] (https://docs.aws.amazon.com/codebuild/latest/userguide/create-project.html#create-project-cli) for details. Default: BUILD_GENERAL1_SMALL
1. `EnvironmentType` (optional) - Environment type used by AWS CodeBuild. See [the documentation] (https://docs.aws.amazon.com/codebuild/latest/userguide/create-project.html#create-project-cli) for details. Default: LINUX_CONTAINER
1. `BuildSpecFilePath` (optional) - CodeBuild build spec file name for build stage. See [the documentation](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html). Default: buildspec.yaml
1. `IntegTestRoleName` (optional) - IAM role name for test stage. This role needs to be configured to allow codebuild.amazonaws.com and cloudformation.amazonaws.com to assume it. Test stage will not be added if default value is used. Default: ''
1. `IntegTestBuildSpecFilePath` (optional) - CodeBuild build spec file name for test stage. See [the documentation](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html). Default: integ-test-buildspec.yaml
1. `DeployRoleName` (optional) - IAM role name for deploy stage. This role needs to be configured to allow cloudformation.amazonaws.com to assume it. Deploy stage will not be added if default value is used. Default: ''
1. `DeployStackName` (optional) - CloudFormation stack name for deploy stage.
1. `DeployParameterOverrides` (optional) - CloudFormation parameter overrides for deploy stage in JSON string. Default: {}
1. `PublishToSAR` (optional) - Boolean to indicate whether or not include publish stage. Allowed values: true, false. Default: false

## Outputs

1. `ArtifactsBucketArn` - The S3 bucket ARN that stores artifacts for the pipeline such as input and output artifacts between stages.
1. `ArtifactsBucketName` - The S3 bucket name that stores artifacts for the pipeline such as input and output artifacts between stages.
1. `PipelineName` - The CodePipeline pipeline name.
1. `PipelineVersion` - The CodePipeline pipeline version.

## IAM Roles in Test and Deploy stages

IAM roles are required to provide in Test and Deploy stages. IAM policies will be attached to the provided IAM roles.

### Test stage

In test stage, the tests are run in CodeBuild. IAM policies are attached to the provided `IntegTestRole` to grant permissions to CodeBuild to:
- Write logs to CloudWatch logs
- Read artifacts from previous stage in S3 artifacts bucket.
- Write artifacts to be used by later stage in S3 artifacts bucket.

Here is the IAM policy that will be attached to the provided `IntegTestRole`:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": [
                "arn:aws:logs:<region>:<account>:log-group:/aws/codebuild/*"
            ],
            "Effect": "Allow"
        },
        {
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "s3:GetObjectVersion"
            ],
            "Resource": [
                "arn:aws:s3:::<artifacts-bucket>/*"
            ],
            "Effect": "Allow"
        },
        {
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::<artifacts-bucket>"
            ],
            "Effect": "Allow"
        }
    ]
}
```

### Deploy stage

In deploy stage, the application is deployed via CloudFormation. IAM policies are attached to the provided `DeployRole` to grant permissions to CloudFormation to:
- Read artifacts from previous stage in S3 artifacts bucket.

Here is the IAM policy that will be attached to the provided `DeployRole`:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::<artifacts-bucket>/*"
            ],
            "Effect": "Allow"
        }
    ]
}
```

## License Summary

This sample code is made available under the MIT-0 license. See the LICENSE file.
