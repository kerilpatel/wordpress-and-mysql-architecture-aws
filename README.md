# Deployment of WordPress and MySQL on AWS using Terraform
Deploying WordPress Website with the dedicated database server (MySQL) using Terraform (Infrastructure as Code)

## Table of Contents


## Objectives
- To deploy a WordPress Website with a dedicated MySQL Database Server
- To make sure that WordPress is publicly accessible to clients whereas, Database should not be accessible to outside world due to security reasons

## Pre-requisites 
1. IAM (Identity Access Management)
2. AWS CLI Configuration
3. Terraform 

## Some Basic Terminologies 
- **WordPress**: WordPress is a free and open-source content management system written in PHP and paired with a MySQL or MariaDB database. Features include a plugin architecture and a template system, referred to within WordPress as Themes.

- **MySQL**: MySQL is an open-source relational database management system. A relational database organizes data into one or more data tables in which data types may be related to each other; these relations help structure the data.

- **Terraform**: Terraform is an open-source infrastructure as code software tool created by HashiCorp. Users define and provide data center infrastructure using a declarative configuration language known as HashiCorp Configuration Language, or optionally JSON.

## Architecture
![Job1](/Images/Architecture%20Design.png)
