# Deploy Wordpress with Amazon RDS

This repository contains what is needed to deploy Wordpress in AWS.

*Top level and first level hierarchy of nested stacks.*

![top-level](https://user-images.githubusercontent.com/53886913/219972549-17ba8b71-3e53-4282-bd7d-c0af7f8732e2.png)

*The following diagram represents high level overview of the infrastructure:*

![diagram](https://github.com/andres-pulecio/deploy-wordpress-with-amazon-rds/assets/53886913/72a6558b-6db6-42d2-a90b-0e35da19615d)

This template create:
* The root stack (which is also a parent stack for the first level stacks). This root stack will contain all the other stacks.
* The IAM instance role stack. This contains the IAM instance role template decoupled form your EC2 template.
## VPC stack: 
This AWS CloudFormation template creates a Virtual Private Cloud (VPC) stack with 2 Public Subnets, an Internet Gateway, and Route tables. It's designed for use in an AWS CloudFormation workshop to illustrate how to create a basic VPC infrastructure.

### Parameters

1. **AvailabilityZones** (List): The list of Availability Zones to use for the subnets in the VPC.
2. **VPCName** (String): The name of the VPC.
3. **VPCCidr** (String): The CIDR block for the VPC. Must be in the form x.x.x.x/16-28.
4. **PublicSubnet1Cidr** (String): The CIDR block for the public subnet located in Availability Zone 1.
5. **PublicSubnet2Cidr** (String): The CIDR block for the public subnet located in Availability Zone 2.

### Resources

#### VPC with Internet Gateway

- **VPC**: Creates the Virtual Private Cloud (VPC) with the specified CIDR block.
- **IGW**: Creates an Internet Gateway and attaches it to the VPC.
- **VPCtoIGWConnection**: Attaches the Internet Gateway to the VPC.

#### Public Route Table

- **PublicRouteTable**: Creates a Route Table for the public subnets.
- **PublicRoute**: Adds a default route to the Internet Gateway for the public subnets.

#### Private Route Tables

- **PrivateRouteTable**: Creates a Route Table for the private subnets.
- **PrivateRouteTable2**: Creates a second Route Table for the private subnets.

#### Public Subnet

- **PublicSubnet1**: Creates the first public subnet in Availability Zone 1.
- **PublicRouteTableAssociation1**: Associates the first public subnet with the public route table.

#### Private Subnets

- **PrivateSubnet1**: Creates the first private subnet in Availability Zone 1.
- **PrivateRouteTableAssociation1**: Associates the first private subnet with the private route table.
- **PrivateSubnet2**: Creates the second private subnet in Availability Zone 2.
- **PrivateRouteTableAssociation2**: Associates the second private subnet with the private route table.
### Outputs


- **VpcId**: The ID of the created VPC.
- **PrivateSubnet1**: The ID of the first private subnet.
- **PrivateSubnet2**: The ID of the second private subnet.
- **PublicSubnet1**: The ID of the first public subnet.

### Usage

1. Deploy this CloudFormation template in your AWS account.
2. Provide the necessary parameter values as input.
3. After successful deployment, you will have a VPC with public and private subnets, internet connectivity, and route tables.

### Note

- This template is for educational purposes and provides a basic VPC setup.
- Make sure to configure security groups, NACLs, and other resources as per your application's requirements.


## The RDS stack: 
This AWS CloudFormation template deploys a MySQL database using Amazon RDS in a specified Virtual Private Cloud (VPC). It also configures security groups to allow access to the database and defines the necessary parameters and resources.

### Parameters

1. **SubnetId1** (AWS::EC2::Subnet::Id): The ID of the first subnet where the RDS instance will be deployed.
2. **SubnetId2** (AWS::EC2::Subnet::Id): The ID of the second subnet where the RDS instance will be deployed.
3. **RDSMasterUsername** (String): The master username for the RDS database.
4. **RDSMasterUserPassword** (String): The master user password for the RDS database.
5. **VpcId** (AWS::EC2::VPC::Id): The ID of the Virtual Private Cloud (VPC) where the RDS instance will be deployed.

### Resources

#### RDSSecurityGroup

- **Type**: AWS::EC2::SecurityGroup
- **Description**: Creates a security group that allows HTTP, HTTPS, and MySQL (3306) access from anywhere.
- **Properties**: Specifies inbound and outbound rules for the security group.

#### DBSubnetGroup

- **Type**: AWS::RDS::DBSubnetGroup
- **Description**: Creates a DB subnet group for the RDS instance.
- **Properties**: Defines the name, description, and subnet IDs for the subnet group.

#### DBSecurityGroup

- **Type**: AWS::RDS::DBSecurityGroup
- **Description**: Creates a DB security group that allows access from the EC2 security group defined by RDSSecurityGroup.
- **Properties**: Specifies inbound rules to allow access from the specified EC2 security group.

#### DBInstance

- **Type**: AWS::RDS::DBInstance
- **Description**: Creates the RDS database instance with the specified settings.
- **Properties**: Specifies instance details, such as storage, instance class, engine, version, credentials, and security group.

### Outputs

#### RDSEndpointAddress

- **Description**: RDS Database Endpoint Address
- **Value**: The endpoint address of the RDS database instance.

### Usage

1. Deploy this CloudFormation template in your AWS account.
2. Provide the necessary parameter values as input.
3. After successful deployment, you will have an RDS MySQL database instance accessible within the specified VPC.

### Note

- Ensure that your security group settings and database configurations are in line with your security and application requirements.
- Modify the template as needed for production use cases, such as specifying production-ready instance types and enabling Multi-AZ deployments for high availability.

## EC2 stack: 

This AWS CloudFormation template creates an Amazon Elastic Compute Cloud (EC2) instance and configures it as a simple web server. It deploys the instance in a specified Virtual Private Cloud (VPC) and provides parameters for customizing the environment type, Amazon Machine Image (AMI) ID, and more.
### Parameters

- **EnvironmentType**: Specify the environment type of the stack (Dev, Test, or Prod).
- **AmiID**: The ID of the Amazon Machine Image (AMI) to use for the EC2 instance.
- **VpcId**: The ID of the VPC where the EC2 instance will be deployed.
- **SubnetId**: The ID of the subnet where the EC2 instance will be launched.
- **WebServerInstanceProfile**: Instance profile resource ID.
- **RDSEndpointAddress**: RDS Database Endpoint Address.
- **RDSMasterUsername**: RDS Database Username.
- **RDSMasterUserPassword**: RDS Database Password.

### Metadata

The metadata section defines parameter groups and labels for the CloudFormation interface.

### Mappings

The mappings section defines instance types based on the environment type (Dev, Test, or Prod).

### Resources

#### WebServerInstance

- **Type**: AWS::EC2::Instance
- **Description**: Creates an EC2 instance and configures it as a simple web server.
- **Properties**: Defines instance details, user data, security groups, and more.

#### WebServerSecurityGroup

- **Type**: AWS::EC2::SecurityGroup
- **Description**: Creates a security group that allows HTTP, HTTPS, and MySQL (3306) access.
- **Properties**: Specifies inbound and outbound rules for the security group.

#### WebServerEIP

- **Type**: AWS::EC2::EIP
- **Description**: Creates an Elastic IP (EIP) and associates it with the web server EC2 instance.
- **Properties**: Specifies the domain and instance ID.

### Outputs

- **WebServerPublicDNS**: Public DNS of the EC2 instance.
- **WebServerElasticIP**: Elastic IP associated with the web server EC2 instance.
- **WebsiteURL**: Application URL for accessing the web server.

### Usage

1. Deploy this CloudFormation template in your AWS account.
2. Provide the necessary parameter values as input, including the environment type, AMI ID, and VPC details.
3. After successful deployment, you will have an EC2 instance running as a web server accessible via the provided URL.

### Note

- This template creates a basic web server for demonstration purposes. Modify and enhance it according to your specific use case and security requirements.
- Ensure that security group settings are suitable for your application's security needs.
- Customize the user data script in the `WebServerInstance` resource section to install additional software or perform specific configurations.
- Always follow AWS best practices for security and resource management when deploying EC2 instances in a production environment.
- it is necessary to execute this command on the instance accessing the SSM:
  ```bash
  sudo /etc/cfn/set_user_db.sh
  ```
## Prerequisites
Whilst single templates can be deployed from your local machine, Nested Stacks require that the nested templates are stored in an S3 bucket.

You have created simple CloudFormation template which created S3 bucket. Please make a note of the bucket name.

For example:

Bucket name: cfn-workshop-s3-s3bucket-2cozhsniu50t

If you don't have S3 bucket, please go back to [S3simple.yaml](../../S3/S3simple.yaml) create one.
