
# Deployment of WordPress and MySQL on AWS using Terraform
Deploying WordPress Website with the dedicated database server (MySQL) using Terraform (Infrastructure as Code)

## Table of Contents


## Objectives
- To deploy a WordPress Website with a dedicated MySQL Database Server
- To make sure that WordPress is publicly accessible to clients whereas, Database should not be accessible to outside world due to security reasons

## Pre-requisites 
1. IAM (Identity Access Management)
2. AWS CLI Configuration
3. Terraform CLI Installed

## Some Basic Terminologies 
- **WordPress**: WordPress is a free and open-source content management system written in PHP and paired with a MySQL or MariaDB database. Features include a plugin architecture and a template system, referred to within WordPress as Themes.

- **MySQL**: MySQL is an open-source relational database management system. A relational database organizes data into one or more data tables in which data types may be related to each other; these relations help structure the data.

- **Terraform**: Terraform is an open-source infrastructure as code software tool created by HashiCorp. Users define and provide data center infrastructure using a declarative configuration language known as HashiCorp Configuration Language, or optionally JSON.

## Architecture
![Architecture](/Images/Architecture%20Design.png)

## Project Steps
1. Creating a IAM User with Administrator Access.

![IAM User](/Images/Creating%20IAM%20User.png)


2. Configure the above IAM User by `aws configure` command.  Generate AWS Access Key ID and AWS Secret Access Key from IAM.

![AWS Configure](/Images/Configure%20User.png)


3. Code the required infrastructure using terraform and save the file with terraform (.tf) extension. 
<br>To understand Infrastructure Code please jump to [Terraform code explanation](https://github.com/kerilpatel/wordpress-and-mysql-architecture-aws#understanding-terraform-code).

![Terraform Code](/Images/Coding%20the%20Infrastructure.png)


4. Initialize a working directory containing Terraform configuration files using `terraform init`

![Terraform init](/Images/Terraform%20init.png)

5. Use `terraform apply --auto-approve` command to generate required Infrastructure from previously saved plan.
<br>`teraform apply` : This command executes the actions proposed in a Terraform plan.
<br>`--auto-approve` : Skips interactive approval of plan before applying. This option is ignored when you pass a previously-saved plan file, because Terraform considers you passing the plan file as the approval and so will never prompt in that case.

![Terraform Apply-1](/Images/Terraform%20Apply-1.png)
![Terraform Apply-2](/Images/Terraform%20Apply-2.png)

6. Accessing WordPress website using Public IPv4 address

![WordPress-1](/Images/Accessing%20WordPress-1.png)
![WordPress-2](/Images/Accessing%20WordPress-2.png)


## Understanding Terraform Code

### Providing Information about the provider , the profile and the availability zone.
``` 
provider  "aws"{
region = "ap-south-1"
profile = "default"
}
```
### VPC 
Amazon Virtual Private Cloud is a commercial cloud computing service that provides users a virtual private cloud, by provisioning a logically isolated section of Amazon Web Services Cloud". Enterprise customers are able to access the Amazon Elastic Compute Cloud over an IPsec based virtual private network.
```
resource  "aws_vpc"  "main" {
cidr_block = "192.168.0.0/16"
enable_dns_hostnames = "true"
instance_tenancy = "default"
tags = {
	Name = "test-vpc"
	}
}
```

![VPC](/Images/VPC.png)

### Internet Gateway
An internet gateway is a horizontally scaled, redundant, and highly available VPC component that allows communication between your VPC and the internet.
```
resource  "aws_internet_gateway"  "igw" {
vpc_id = "${aws_vpc.main.id}"
tags = {
	Name = "test_gw"
	}
}
```

![Internet Gateway](/Images/Internet%20Gateway.png)

### Subnet
A subnet is a range of IP addresses in your VPC. You can launch AWS resources, such as EC2 instances, into a specific subnet. When you create a subnet, you specify the IPv4 CIDR block for the subnet, which is a subset of the VPC CIDR block.
Here there are two types of Subnets:
- **Public Subnet** : The subnet's IPv4 or IPv6 traffic is routed to an internet gateway or an egress-only internet gateway and can reach the public internet.
- **Private Subnet** : The subnet’s IPv4 or IPv6 traffic is not routed to an internet gateway or egress-only internet gateway and cannot reach the public internet.
```
resource  "aws_subnet"  "public" {
vpc_id = "${aws_vpc.main.id}"
cidr_block = "192.168.0.0/24"
availability_zone = "ap-south-1a"
map_public_ip_on_launch = "true"
tags = {
	Name = "public"
	}
}
```

![Public Subnet](/Images/Public%20Subnet.png)
```
resource  "aws_subnet"  "private" {
vpc_id = "${aws_vpc.main.id}"
cidr_block = "192.168.1.0/24"
availability_zone = "ap-south-1b"
tags = {
	Name = "private"
	}
}
```

![Private Subnet](/Images/Private%20Subnet.png)

### Route Table
A route table contains a set of rules, called routes, that are used to determine where network traffic from your subnet or gateway is directed.
```
resource  "aws_route_table"  "public_route" {
vpc_id = "${aws_vpc.main.id}"
route {
	cidr_block = "0.0.0.0/0"
	gateway_id = "${aws_internet_gateway.igw.id}"
}

tags = {
	Name = "pubic_routetable"
	}
}

resource  "aws_route_table_association"  "public_subnet_asso" {
	subnet_id = "${aws_subnet.public.id}"
	route_table_id = "${aws_route_table.public_route.id}"
	depends_on = [aws_route_table.public_route , aws_subnet.public]
}

resource  "aws_eip"  "lb" {
vpc = true
depends_on = [aws_internet_gateway.igw]
}
```

### NAT Gateway
A NAT gateway is a Network Address Translation (NAT) service. You can use a NAT gateway so that instances in a private subnet can connect to services outside your VPC but external services cannot initiate a connection with those instances.
```
resource  "aws_nat_gateway"  "gw" {
allocation_id = "${aws_eip.lb.id}"
subnet_id = "${aws_subnet.public.id}"
depends_on = [aws_internet_gateway.igw]
tags = {
	Name = "gw NAT"
	}
}

resource  "aws_route_table"  "nat_route" {
vpc_id = "${aws_vpc.main.id}"
route {
	cidr_block = "0.0.0.0/0"
	gateway_id = "${aws_internet_gateway.igw.id}"
	}

tags = {
	Name = "nat_routetable"
	}
}

```

![NAT Gateway](/Images/NAT%20Gateway.png)

### Security Groups 
A security group acts as a virtual firewall for your EC2 instances to control incoming and outgoing traffic. Inbound rules control the incoming traffic to your instance, and outbound rules control the outgoing traffic from your instance. When you launch an instance, you can specify one or more security groups.

***For WordPress Instance*** 
```
resource "aws_security_group" "sg_public" {
  name        = "vpc_sg"
  description = "Allow HTTP , SSH and ICMP"
  vpc_id      = "${aws_vpc.main.id}"

  ingress {
    description = "ssh"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    description = "http"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    description = "icmp"
    from_port   = 0
    to_port     = 0
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    description = "mysql"
    from_port   = 3306
    to_port     = 3306
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "sg_public"
  }
}
	
```

![WordPress SG](/Images/WordPress-SG.png)

***For MySQL Instance*** 
```
resource "aws_security_group" "sg_private" {

  name        = "sg_private"
  description = "Allow wordpress inbound traffic"
  vpc_id      = "${aws_vpc.main.id}"

  ingress {

    description = "Allow only wordpress"
    from_port   = 3306
    to_port     = 3306
    protocol    = "tcp"
    security_groups = [aws_security_group.sg_public.id]
}
  
    ingress {

    description = "Allow wordpress ping"
    from_port   = -1
    to_port     = -1
    protocol    = "icmp"
    security_groups = [aws_security_group.sg_public.id]
}

    egress {
      
      from_port=0
      to_port=0
      protocol="-1"
      cidr_blocks=["0.0.0.0/0"]
      ipv6_cidr_blocks =  ["::/0"]
}

  tags = {
    Name = "sg_private"
  }
}
```

![MySQL SG](/Images/MySQL-SG.png)

### EC2 Instances

EC2 provides a wide selection of instance types optimized to fit different use cases. Instance types comprise varying combinations of CPU, memory, storage, and networking capacity and give you the flexibility to choose the appropriate mix of resources for your applications. Each instance type includes one or more instance sizes, allowing you to scale your resources to the requirements of your target workload.

***For WordPress***
```
resource "aws_instance" "wordpress" {
  ami           = "ami-ff82f990"
  instance_type = "t2.micro"
  associate_public_ip_address = true
  subnet_id = "${aws_subnet.public.id}"
  vpc_security_group_ids = [aws_security_group.sg_public.id]
  key_name = "aws-key"

  tags = {
    Name = "wordpress"
    }
  } 
```

![WordPress EC2](/Images/WordPress-EC2.png)

***For MySQL***
```
resource "aws_instance" "mysql-private" {
  ami           = "ami-08706cb5f68222d09"
  instance_type = "t2.micro"
  associate_public_ip_address = true
  vpc_security_group_ids = [aws_security_group.sg_private.id]
  key_name = "aws-key"
  subnet_id = "${aws_subnet.private.id}"

  tags = {
    Name = "mysql"
    }
}
```

![MySQL EC2](/Images/MySQL-EC2.png)

## References
- [Terraform Documentation](https://www.terraform.io/docs/)
- [AWS Documentation](https://docs.aws.amazon.com/)
