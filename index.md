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
