# AWS VPC Infrastructure with Terraform

## Project Overview

This project uses Terraform to provision AWS networking infrastructure
in the US East (N. Virginia) region (`us-east-1`).

## Architecture

- One Amazon VPC with CIDR `10.0.0.0/16`
- One public subnet (`10.0.1.0/24`)
- One private subnet (`10.0.2.0/24`)
- One Internet Gateway
- One NAT Gateway with an Elastic IP
- Public and private route tables and subnet associations
- Amazon S3 remote Terraform state with versioning,
  server-side encryption, and public access blocked

## Tools Used

- Terraform
- AWS CLI
- Amazon VPC
- Amazon S3
- AWS IAM

## Validation

- `terraform fmt`
- `terraform validate`
- `terraform plan`
- `terraform state list`
- Verified NAT Gateway status using AWS CLI

## Cost Management

The NAT Gateway incurs ongoing hourly and data-processing charges.
Destroy the development networking resources when they are no longer
needed to avoid unnecessary costs.

## Environment

- Environment: dev
- AWS Region: us-east-1
- Terraform state key: `environments/dev/terraform.tfstate`
