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
To understand Terraform Code please jump to Infrastructure code explanation.
![Terraform Code](/Images/Coding%20the%20Infrastructure.png)

4. Initialize a working directory containing Terraform configuration files using `terraform init`
 ![Terraform init](/Images/Terraform%20init.png)

5. Use `terraform apply --auto-approve` command to generate required Infrastructure from previously saved plan.
`teraform apply` : This command executes the actions proposed in a Terraform plan.
`--auto-approve` : Skips interactive approval of plan before applying. This option is ignored when you pass a previously-saved plan file, because Terraform considers you passing the plan file as the approval and so will never prompt in that case.
![Terraform Apply-1](/Images/Terraform%20Apply-1.png)
![Terraform Apply-2](/Images/Terraform%20Apply-2.png)

6. Accessing WordPress website using Public IPv4 address
![WordPress-1](/Images/Accessing%20WordPress-1.png)
![WordPress-2](/Images/Accessing%20WordPress-2.png)
