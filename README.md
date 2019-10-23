# ml-agents-build-use-codebuild

## Usage

```console
> aws cloudformation create-stack \
  --stack-name <YOUR Stack Name> \
  --template-body file://template.yaml \
  --capabilities CAPABILITY_IAM \
  --region <YOUR Region> \
  --parameters '[
      {
        "ParameterKey": "ProjectName",
        "ParameterValue": "<YOUR Codebuild Name>"
      },
      {
        "ParameterKey": "InputBucketName",
        "ParameterValue": "<YOUR Input Bucket Name>"
      },
      {
        "ParameterKey": "OutputBucketName",
        "ParameterValue": "<YOUR Output Bucket Name>"
      },
      {
        "ParameterKey": "ECRRepositoryName",
        "ParameterValue": "<YOUR Repository Name>"
      }
  ]'


> aws cloudformation describe-stacks \
  --region <YOUR Region> \
  --stack-name <YOUR Stack Name> \
  --query "Stacks[0].StackStatus"

"CREATE_COMPLETE"


# Upload dummy file
> touch dummy.txt
> zip dummy.zip dummy.txt
  adding: dummy.txt (stored 0%)

> aws s3 cp dummy.zip s3://<YOUR Input Bucket Name>/
upload: ./dummy.zip to s3://<YOUR Input Bucket Name>/dummy.zip


> aws codebuild start-build \
  --region <YOUR Region> \
  --project-name <YOUR Codebuild Project Name> \
  --buildspec-override file://buildspec.yaml \
  --timeout-in-minutes-override 20 \
  --environment-variables-override '[
    {
      "name": "ECR_REPOSITORY_NAME",
      "value": "<YOUR Repository Name>",
      "type": "PLAINTEXT"
    },
    {
      "name": "IMAGE_TAG",
      "value": "0.9.1",
      "type": "PLAINTEXT"
    }
  ]'


# Check Status
> aws codebuild batch-get-builds \
  --region <YOUR Region> \
  --ids <YOUR Codebuild Project Name>:17bfb5c6-8c8e-480c-a445-9e5dbb4b6004 \
  --query "builds[0].phases"


> aws ecr list-images \
  --repository-name <YOUR Repository Name> \
  --region <YOUR Region>

> $(aws ecr get-login --no-include-email --region <YOUR Region>)

WARNING! Using --password via the CLI is insecure. Use --password-stdin.
Login Succeeded


> docker pull <AWS_ACCOUNT_ID>.dkr.ecr.<YOUR Region>.amazonaws.com/<YOUR Repository Name>:0.9.1

...

1b38598abf23: Pull complete
Digest: sha256:xxx
Status: Downloaded newer image for <AWS_ACCOUNT_ID>.dkr.ecr.<YOUR Region>.amazonaws.com/<YOUR Repository Name>:0.9.1
<AWS_ACCOUNT_ID>.dkr.ecr.<YOUR Region>.amazonaws.com/<YOUR Repository Name>:0.9.1


> docker run -it --rm \
  <AWS_ACCOUNT_ID>.dkr.ecr.<YOUR Region>.amazonaws.com/<YOUR Repository Name>:0.9.1
```