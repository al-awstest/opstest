This is minimalistic set of scripts to provision multi-region load-balanced and auto-scaled infrastructure
for deploying sample application

1. First you will need to access your AWS console and get Access key

2. install AWS CLI and configure it with Access key from previous step and initial region

3. Create IAM roles for CloudFormation Stack Sets and user for GitHub, Admin account ID needs to be provided
```bash
aws cloudformation create-stack --stack-name iam-stack --template-body file://iam-stack.json --parameters ParameterKey=AdministratorAccountId,ParameterValue=322223322223 --capabilities CAPABILITY_NAMED_IAM
```

4. Create HostedZone on Route53, domain name needs to be provided
```bash
aws cloudformation create-stack --stack-name route53-stack --template-body file://route53-stack.json --parameters ParameterKey=DomainName,ParameterValue=alawstest.me
```

5. Create Stack Set, need to provide email for notifications, domain name, Admin account ID
```bash
aws cloudformation create-stack-set --stack-set-name elb-regional-stack-set --template-body file://elb-regional-stack-set.json --parameters ParameterKey=OperatorEMail,ParameterValue=al.awstest@gmail.com ParameterKey=DomainName,ParameterValue=alawstest.me ParameterKey=AdministratorAccountId,ParameterValue=322223322223 --capabilities CAPABILITY_IAM
```

6. Now you can deploy infrastructure and application using specified account on regions of choice
```bash
aws cloudformation create-stack-instances --stack-set-name elb-regional-stack-set --accounts 322223322223 --regions "eu-west-1" "eu-central-1"
```

7. Open AWS Console, CloudFormation->StackSets->elb-regional-stack-set, sit back and relax, AWS will do everything for you in approx 30 minutes

8. After all stacks being created, you can get NS servers for your domain, which will return LoadBalancer IPs of fastest reachable region
```bash
aws cloudformation describe-stacks --stack-name=route53-stack --query='Stacks[0].Outputs[0].OutputValue'
```

9. You can use those NSes for your domain, or simple test
```bash
host alawstest.me <ONE ON NSes FROM POINT 8>
curl <ANY OF RETURNED IP ADDRESSES>/hello # this should return some text and AZ id
```

9. Now you can deploy app manually by specifying new commitId. As GitHub recently dropped Services integration, my "cool" auto-deploy idea failed, need time to get over webhooks.
```bash
aws cloudformation update-stack-set --stack-set-name elb-regional-stack-set --template-body file://elb-regional-stack-set.json --parameters ParameterKey=OperatorEMail,ParameterValue=al.awstest@gmail.com ParameterKey=DomainName,ParameterValue=alawstest.me ParameterKey=AdministratorAccountId,ParameterValue=322223322223 ParameterKey=GitCommitId,ParameterValue=6801f7305ce82d99be2c5be957640c35fabd4731 --capabilities CAPABILITY_IAM
```

please note that this was tested with public GitHub repo, 
for private repository you will need to skip initial deployment, access AWS Console. initiate deployment by hand and connect your GitHub account to AWS console
