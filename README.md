 # :warning: Repository Deprecation Notice :warning:
This repository, `awslabs/aws-sam-codepipeline-cd`, is scheduled for archival by the end of August 2024. After this date, the repository will become read-only, and no further updates or support will be provided.

We recommend that you migrate to the AWS Serverless Application Model (SAM) CLI's `sam pipelines` command, which provides a streamlined and integrated experience for automatically generating deployment pipelines for your serverless applications.

The `sam pipelines` command replaces the functionality provided by this repository and offers a more seamless and up-to-date solution. You can find detailed documentation and guidance on using sam pipelines at the following links:
- [Introducing AWS SAM Pipelines: Automatically Generate Deployment Pipelines for Serverless Applications](https://aws.amazon.com/blogs/compute/introducing-aws-sam-pipelines-automatically-generate-deployment-pipelines-for-serverless-applications/)
- [AWS SAM CLI Command Reference: sam pipeline bootstrap](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-pipeline-bootstrap.html)
- [AWS SAM CLI Command Reference: sam pipeline init](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-pipeline-init.html)

Please take the necessary steps to migrate your projects to the sam pipelines command before the end of August 2024. After this date, this repository will be archived, and no further updates or support will be provided. 

Thank you for your understanding and cooperation.

---

# AWS SAM CodePipeline CD ![Build Status](https://codebuild.us-east-1.amazonaws.com/badges?uuid=eyJlbmNyeXB0ZWREYXRhIjoiSlYvZlpjVEttS0FjbXFyUEpvWFpwdGJtbTlSbjRTaEsranM4QjFpUWxUSTB4ZUdPeDROSlNTMW14bnRxM1l6YTRpUDZsSXg2L3hyRXpWN0ZxM1BpeGFBPSIsIml2UGFyYW1ldGVyU3BlYyI6Ik9pWWV1MXhKbnR6alB2NTUiLCJtYXRlcmlhbFNldFNlcmlhbCI6MX0%3D&branch=master)

This serverless app sets up an AWS CodePipeline pipeline as a Continuous Deployment (CD) solution for a SAM project hosted on GitHub or AWS CodeCommit. Once setup, every time you push to the specified Git repository branch, the change will flow through the AWS CodePipeline pipeline.

## Pipeline Structure

When this application is deployed, it will create an AWS CodePipeline pipeline that has up to the following 5 stages:
1. **Source**: This stage is the entry point of the pipeline. It is triggered when you push a change to the specified Git repository branch.
1. **Build**: This stage builds the project using AWS CodeBuild.
1. **Test** (optional): This stage runs the integration tests of the project using CodeBuild. This stage will only be created if you provide the `IntegTestRoleName` parameter when setting up this module. See the "Parameters" section below.
1. **Deploy** (optional): This stage deploys the project using CloudFormation. This stage will only be created if you provide the `DeployRoleName` parameter when setting up this application. See the "Parameters" section below.
1. **Publish** (optional): This stage publishes the project to AWS Serverless Application Repository using the publish [app](https://serverlessrepo.aws.amazon.com/applications/arn:aws:serverlessrepo:us-east-1:077246666028:applications~aws-serverless-codepipeline-serverlessrepo-publish). This stage will only be created if you pass 'true' to the `PublishToSAR` parameter when setting up this module. See the "Parameters" section below.

Here is an example CodePipeline pipeline that has all 5 stages:
![aws-sam-codepipeline-cd-pipeline-example](https://github.com/awslabs/aws-sam-codepipeline-cd/raw/master/images/aws-sam-codepipeline-cd-pipeline-example.png)

## Installation

1. [Create an AWS account](https://portal.aws.amazon.com/gp/aws/developer/registration/index.html) if you do not already have one and login
1. If your source code repository is on GitHub, then create a GitHub OAuth token (see instructions below). 
1. Go to this app's page on the [Serverless Application Repository](https://serverlessrepo.aws.amazon.com/applications/arn:aws:serverlessrepo:us-east-1:646794253159:applications~aws-sam-codepipeline-cd) and click "Deploy"
1. Provide the required app parameters and click "Deploy"

### Creating a GitHub OAuth token

General instructions for creating a GitHub OAuth token can be found [here](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/). When you get to the scopes/permissions page, you should select the "repo" and "admin:repo_hook" scopes, which will automatically select all permissions under those two scopes.

![GitHub OAuth Token Permissions](https://github.com/awslabs/aws-sam-codepipeline-cd/raw/master/images/github-token-permissions.png)

## Parameters

The app has the following parameters:

| Parameter     | Required      | Description   |
| ------------- | ------------- | ------------- |
| SourceCodeProvider  | Optional  | Whether the Git repository is hosted on GitHub or CodeCommit. Allowed values: GitHub, CodeCommit. Default: GitHub |
| ComputeType  | Optional  | AWS CodeBuild project compute type. See [the documentation](https://docs.aws.amazon.com/codebuild/latest/userguide/create-project.html#create-project-cli) for details. Default: BUILD_GENERAL1_SMALL |
| EnvironmentType  | Optional  | Environment type used by AWS CodeBuild. See [the documentation](https://docs.aws.amazon.com/codebuild/latest/userguide/create-project.html#create-project-cli) for details. Default: LINUX_CONTAINER |
| BuildSpecFilePath  | Optional  | CodeBuild build spec file name for build stage. See [Build Specification Reference for CodeBuild](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html). Default: buildspec.yaml |
| IntegTestRoleName  | Optional  | IAM role name for test stage. This role needs to be configured to allow codebuild.amazonaws.com and cloudformation.amazonaws.com to assume it. Test stage will not be added if default value is used. Default: '' |
| IntegTestBuildSpecFilePath  | Optional  | CodeBuild build spec file name for test stage. This parameter is only used if you provide the `IntegTestRoleName` parameter. See [the documentation](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html). Default: integ-test-buildspec.yaml |
| DeployRoleName  | Optional  | IAM role name for deploy stage. This role needs to be configured to allow cloudformation.amazonaws.com to assume it. Deploy stage will not be added if default value is used. Default: '' |
| DeployStackName  | Optional  | CloudFormation stack name for deploy stage. Default: ''. This parameter is only used if you provide the `DeployRoleName` parameter. Note that if you provide the `DeployRoleName` but do not provide a `DeployStackName` then AWS CodePipeline will fail. |
| DeployParameterOverrides  | Optional  | CloudFormation parameter overrides for deploy stage in JSON string. For more information and an example, see the `ParameterOverrides` parameter of [AWS CloudFormation Configuration Properties Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/continuous-delivery-codepipeline-action-reference.html#w2ab1c13c13b9). Default: {} |
| PublishToSAR  | Optional  | Boolean to indicate whether or not include publish stage. Allowed values: true, false. Default: false |

The following parameters are only applicable if `SourceCodeProvider` is GitHub.

| Parameter     | Required      | Description   |
| ------------- | ------------- | ------------- |
| GitHubOwner  | Optional  | GitHub username owning the repo. |
| GitHubRepo  | Optional  | GitHub repo name (just the name, not the full URL). |
| GitHubOAuthToken  | Optional  | OAuth token used by AWS CodeBuild to connect to GitHub. |
| GitHubBranch  | Optional  | GitHub repo branch name. Default: master. |

The following parameters are only applicable if `SourceCodeProvider` is CodeCommit.

| Parameter     | Required      | Description   |
| ------------- | ------------- | ------------- |
| CodeCommitRepo  | Optional  | CodeCommit repository name (just the name, not the full URL). |
| CodeCommitBranch  | Optional  | CodeCommit repo branch name. Default: master. |

## Outputs

1. `ArtifactsBucketArn` - The S3 bucket ARN that stores artifacts for the pipeline such as input and output artifacts between stages.
1. `ArtifactsBucketName` - The S3 bucket name that stores artifacts for the pipeline such as input and output artifacts between stages.
1. `PipelineName` - The CodePipeline pipeline name.
1. `PipelineVersion` - The CodePipeline pipeline version.

## IAM Roles in Test and Deploy stages

You must provide IAM roles in order to create Test and Deploy stages. IAM policies will be attached to the IAM roles that you provide. The sections below describe what IAM policies are attached to the IAM role for each stage.

### Test stage

In Test stage, the tests are run in AWS CodeBuild. IAM policies are attached to the provided `IntegTestRole` to grant permissions to CodeBuild to:
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

In deploy stage, the application is deployed using AWS CloudFormation. IAM policies are attached to the provided `DeployRole` to grant permissions to CloudFormation to:
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
