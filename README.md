# Example VPC with AWS 
> Quickstart for running a VPC on AWS using CloudFormation

### Read The Blog Post ([github](./blog_post.md)/[medium](https://medium.com/@t3chflicks/virtual-private-cloud-on-aws-quickstart-with-cloudformation-4583109b2433)).

---

## Architecture
>![architecture](./architecture.png)

# Deployment
```
aws cloudformation create-stack --stack-name vpc --template-body file://tempalte.yml --capabilities CAPABILITY_NAMED_IAM
````