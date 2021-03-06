# [T3chFlicks](https://t3chflicks.org): Quickstart VPC with AWS 
> Quickstart for running a VPC on AWS using CloudFormation
![thumbnail](./thumbnail.png)

---

## `tutorials/`

[![Generic badge](https://img.shields.io/badge/Blog_Post-Github-orange.svg)](./blog_post.md)

[![Generic badge](https://img.shields.io/badge/Blog_Post-Medium-blue.svg)](https://medium.com/@t3chflicks/virtual-private-cloud-on-aws-quickstart-with-cloudformation-4583109b2433)


## Architecture
>![architecture](./architecture.png)

# Deployment
```
aws cloudformation create-stack --stack-name vpc --template-body file://template.yml --capabilities CAPABILITY_NAMED_IAM
````

---

This project was created by [T3chFlicks](https://t3chflicks.org) - A tech focused education and services company.

---
