## Synopsis
AWS recently released a learning path documenting the creation of a Unicorn ride service known as Wild Rydes. While the learning path (found at https://aws.amazon.com/getting-started/serverless-web-app/) is great at helping users understand the high level concepts of serverless architecture and the nitty gritty details of provisiong API gateways, DynamoDB tables, etc. it wasn’t very ‘real world’ friendly. This document aims to take the learning path a step up utilizing a few bash scripts, and the Serverless Framework to emulate a real world workflow. As a note I would strongly recommend completing the learning path as intended, then come back and revisit this page. That will give you a greater appreciation for how powerful and easy the Serverless Framework truly is.

First things first. What is Serverless Framework and how do I use it? Visit https://serverless.com/learn/ to learn more about what it is, and https://serverless.com/framework/docs/providers/aws/guide/installation/ to walk through the installation and initial setup. Once you have your environment setup we can begin to walk through the serverless.yml and see how this correlates with learning module.

### Pre-Module
If you haven’t gone through this learning path as intended some of the things might seem out of order. I will be referencing lines in the serverless.yml that aren’t in sequential order. At some point every line in the .yml will be covered, it just may seem missequenced.

There are a few things that need to be specified before we begin to declare resources to be provisioned, IAM role statements, and the like. I will cover the entirety of the serverless.yml file used for this project. The first few lines in our serverless.yml don’t have any real bearing on the modules and are just informative:

```markdown
service: wild-rydes

frameworkVersion: ">=1.26.1 <2.0.0"

custom:
  stage: ${opt:stage, self:provider.stage}

provider:
  name: aws
  runtime: nodejs6.10
  region: us-east-1
  environment:
    sls_stage: ${self:custom.stage}
    db_name: {Ref: Rides}
```

As you can see from the above we’re defining our service name and the version of the Serverless framework. Nex there is a custom stage option define. This is so we can differentiate between dev, and prod, etc. Then follows all of the provider information. In this case it happens to be AWS; however, please note that the Serverless Framework is flexible and can be used to deploy services in Azure and other Cloud Providers as well. Lastly we have our environment definitions. These were both custom and not necessary for the deployment of this service; however, it will help keep things clean and organized.

### Module 1
The goal of this module is to essentially create an S3 bucket, upload static content, and then create a policy that makes it accessible to anonymous users (i.e. the world), and then enable website hosting on said bucket. This is all pretty easy but quite labor intensive normally. With the Serverless Framework the following lines are all we need to provision the S3 bucket and configure it as needed:

```markdown
resources:
  Resources:
    S3WildRydes:
      Type: AWS::S3::Bucket
      Properties:
        WebsiteConfiguration:
          ErrorDocument: index.html
          IndexDocument: index.html
```

This snippet of the serverless.yml is creating a WebSite enabled S3 bucket and defining the Error and Index documents on the fly!

While we will walk through deploying this project at the end of this document the following shell script is what I use to push my static files into the designated S3 bucket. As a reminder, please remember this script will not work if you have not deployed this service (as it does not yet exist):

```markdown
#!/bin/bash
set -eu

STAGE="${1:-dev}"
echo "Deploying static assets to ${STAGE}..."

BUCKET_NAME=$(aws \
    cloudformation describe-stacks \
    --stack-name "wild-rydes-${STAGE}" \
    --query "Stacks[0].Outputs[?OutputKey=='WebSiteBucket'] | [0].OutputValue" \
    --output text)

WEBSITE_URL=$(aws \
    cloudformation describe-stacks \
    --stack-name "wild-rydes-${STAGE}" \
    --query "Stacks[0].Outputs[?OutputKey=='WebSiteUrl'] | [0].OutputValue" \
    --output text)

aws s3 sync --acl 'public-read' --delete ./static/ "s3://${BUCKET_NAME}/"

echo "Bucket URL: ${WEBSITE_URL}"
```

As a note please be sure to update your stack-name to whatever your stack name is in cloud formation. Additionally, MAKE SURE TO USE THE FILES PROVIDED IN THIS REPO AND NOT THE AWS LEARNING PATH.
