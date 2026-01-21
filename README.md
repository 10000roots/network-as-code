# Automation for Network Engineers: Creating AWS VPC with Python and Boto3

### Introduction
As a Senior Network Engineer with over 10 years of experience (CCIE), I've spent much of my career configuring CLI for hardware routers. Moving to the cloud doesn't mean networking fundamentals change, but how we deploy them does. Today, I want to share a simple Python script using the **Boto3** library to automate the creation of a standard VPC infrastructure.

### The Scenario
We will create a VPC with a `/16` CIDR block, a public subnet, and an Internet Gateway (IGW) to allow external communication. This mimics a basic "Public Subnet" architecture used for jump hosts or web servers.

### The Python Code
```python
import boto3

# Initialize EC2 resource
ec2 = boto3.resource('ec2', region_name='us-west-2')

# 1. Create VPC
vpc = ec2.create_vpc(CidrBlock='10.0.0.0/16')
vpc.create_tags(Tags=[{"Key": "Name", "Value": "CommunityBuilder-VPC"}])
vpc.wait_until_available()
print(f"VPC Created: {vpc.id}")

# 2. Create Internet Gateway & Attach
igw = ec2.create_internet_gateway()
vpc.attach_internet_gateway(InternetGatewayId=igw.id)
print(f"IGW Created and Attached: {igw.id}")

# 3. Create Subnet
subnet = vpc.create_subnet(CidrBlock='10.0.1.0/24')
print(f"Subnet Created: {subnet.id}")

# 4. Create Route Table and Public Route
route_table = vpc.create_route_table()
route = route_table.create_route(
    DestinationCidrBlock='0.0.0.0/0',
    GatewayId=igw.id
)
route_table.associate_with_subnet(SubnetId=subnet.id)
print(f"Route Table Created and Associated with IGW")
