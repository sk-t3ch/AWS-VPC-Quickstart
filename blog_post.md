# Virtual Private Cloud on AWS — Quickstart with CloudFormation

A Virtual Private Cloud is the foundation from which to build a new system. In this article, I demonstrate how to create one using CloudFormation.

![Photo by [Pero Kalimero](https://unsplash.com/@pericakalimerica?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/7008/0*kmVpoS3oecZICPjc)*

Photo by [Pero Kalimero](https://unsplash.com/@pericakalimerica?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

Virtual Private Clouds (VPCs) are networks in the cloud. A VPC contains a range of IP addresses that can be subdivided into smaller ranges of IPs known as subnets. Communication across these subnets can be controlled using route tables, security groups and access control lists. Communication to and from the internet is enabled by two types of gateway:

* NAT Gateways which enable access **to** the internet.

* Internet Gateways which enable access **to** and **from** the internet.

A subnet is considered private if it uses a NAT instead of an Internet Gateway. Route tables determine where network traffic from your subnet or gateway is directed. A route is required from the subnet to the gateway to enable internet access.

VPCs are important because they are environments which can be configured to reduce security risks and improve networks speed between components.

### Architecture

For this example, I am going to create a place in Amazon’s Cloud. The VPC comprises three public and three private (hybrid) subnets.

![](https://cdn-images-1.medium.com/max/2000/1*0xJI-GnE10TDeDZvQGvQrQ.png)

### Deployment

To deploy this system, I am using the AWS CLI with CloudFormation. It is as simple as:

    `aws cloudformation create-stack --stack-name service --template-body file://template.yml --capabilities CAPABILITY_NAMED_IAM`

The full template can be found [here](https://github.com/sk-t3ch/AWS-VPC).

### VPC

To begin, the simplest VPC:

    VPC:
        Type: AWS::EC2::VPC
        Properties:
        CidrBlock: "10.0.0.0/16"


The only property we are required to provide is the `CidrBlock`, a range of IP addresses that you declare available for use in the network.

The next step is to add subnets to the VPC. These occupy sections of the range of IPs in the VPC. You are required to configure `CidrBlock` which in this case is `10.0.1.0/24` meaning that we can use the addresses in the range `10.0.1.0 — 10.0.1.255` .

    Subnet:
        Type: AWS::EC2::Subnet
        Properties:
        VpcId: !Ref VPC
        CidrBlock: "10.0.1.0/24"

The component required to enable access from a subnet to the internet is an Internet Gateway. This sits at the edge of the VPC and orchestrates any traffic routed from or to them with the internet.

The following template configures a public subnet:

    PublicSubA:
        Type: AWS::EC2::Subnet
        Properties:
        VpcId: !Ref VPC
        CidrBlock: "10.0.1.0/24"
        AvailabilityZone: !Select
            - 0
            - Fn::GetAZs: !Ref AWS::Region
            
    InternetGateway:
        Type: AWS::EC2::InternetGateway
        
    GatewayAttachment:
        Type: AWS::EC2::VPCGatewayAttachment
        Properties:
        VpcId: !Ref VPC
        InternetGatewayId: !Ref InternetGateway
        
    RouteTablePublic:
        Type: AWS::EC2::RouteTable
        Properties:
        VpcId: !Ref VPC
        
    RoutePublic:
        Type: AWS::EC2::Route
        Properties:
        DestinationCidrBlock: "0.0.0.0/0"
        GatewayId: !Ref InternetGateway
        RouteTableId: !Ref RouteTablePublic

    RouteTableAssocSubnetPublicA:
        Type: AWS::EC2::SubnetRouteTableAssociation
        Properties:
        RouteTableId: !Ref RouteTablePublic
        SubnetId: !Ref PublicSubA

A private subnet is not directly accessible from the internet. However, it is possible to access the internet from a private subnet using a NAT Gateway. An Elastic IP is required because the NatGateway is not automatically assigned a public IP.

    PrivateSubA:
        Type: AWS::EC2::Subnet
        Properties:
        VpcId: !Ref VPC
        CidrBlock: "10.0.4.0/24"
        AvailabilityZone: !Select
            - 0
            - Fn::GetAZs: !Ref AWS::Region
            
    ElasticIPNatGatewayPrivateA:
        Type: AWS::EC2::EIP
        
    NatGatewayPrivateA:
        Type: AWS::EC2::NatGateway
        Properties:
        SubnetId: !Ref PrivateSubA
        AllocationId: !Ref ElasticIPNatGatewayPrivateA
        
    RouteTablePrivateA:
        Type: AWS::EC2::RouteTable
        Properties:
        VpcId: !Ref VPC
        
    RoutePrivate:
        Type: AWS::EC2::Route
        Properties:
        DestinationCidrBlock: "0.0.0.0/0"
        InstanceId: !Ref NatGatewayPrivateA
        RouteTableId: !Ref RouteTablePrivateA
        
    RouteTableAssocSubnetPrivateA:
        Type: AWS::EC2::SubnetRouteTableAssociation
        Properties:
        RouteTableId: !Ref RouteTablePrivateA
        SubnetId: !Ref PrivateSubA


Availability Zones (AZs) are groups of data centres inside a Region that are isolated from the failure of another AZ. Each subnet created in a VPC must be placed in a single AZ and a system can be protected from an AZ failure by utilising multiple AZ architecture — you’re recommended to have three public and three private subnets spanning different AZ zones. A VPC template for this can be found [here](https://github.com/sk-t3ch/AWS-VPC).

A VPC is pretty useless without any services running inside it. For that, check out the following articles:
[**Cheaper than API Gateway — ALB with Lambda using CloudFormation**
*An alternative to API gateway is Application Load Balancer. ALB can be connected with Lambda to produce a highly…*medium.com](https://medium.com/@t3chflicks/cheaper-than-api-gateway-alb-with-lambda-using-cloudformation-b32b126bbddc)
[**AWS Auto Scaling Spot Fleet Cluster — Quickstart with CloudFormation**
*Running applications on separate machines quickly becomes an inefficient use of compute and therefore money. In this…*medium.com](https://medium.com/@t3chflicks/aws-auto-scaling-spot-fleet-cluster-quickstart-with-cloudformation-6504a61f7aab)

### Thanks For Reading

I hope you have enjoyed this article. If you like the style, check out [T3chFlicks.org](https://t3chflicks.org/Projects/aws-quickstart-series) for more tech focused educational content ([YouTube](https://www.youtube.com/channel/UC0eSD-tdiJMI5GQTkMmZ-6w), [Instagram](https://www.instagram.com/t3chflicks/), [Facebook](https://www.facebook.com/t3chflicks), [Twitter](https://twitter.com/t3chflicks)).



Resources:

* [https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)

* [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html)