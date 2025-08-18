# Introduction to AWS Networking

This repository provides CloudFormation templates and supplementary materials to help you learn and deploy foundational AWS networking components, including VPCs, subnets, route tables, NAT gateways, VPC endpoints, security groups, NACLs, and more.

## Contents

- `examples/cloudformation.yaml`: Basic VPC setup with public/private subnets, IGW, NAT Gateway, route tables, and VPC endpoints for S3 and Lambda.
- `examples/vpc-template.yaml`: Extended template with security group and NACL examples, Lambda function in VPC mode, private hosted zone, and Transit Gateway attachment.

## Prerequisites

- An AWS account with permissions to create VPCs, subnets, IAM roles, Lambda functions, and related networking resources.
- AWS CLI or AWS Management Console access.

## Usage

1. **Clone this repository:**
   ```sh
   git clone https://github.com/<your-org>/intro-to-aws-networking.git
   cd intro-to-aws-networking/examples
   ```

2. **Review and customize parameters:**
   - Edit the parameter defaults in the YAML files as needed (e.g., CIDR blocks, subnet sizes).
   - For Lambda deployment, update the `Role` property in the Lambda function to use a valid IAM role ARN in your account.

3. **Deploy the template:**
   - Using AWS CLI:
     ```sh
     aws cloudformation deploy --template-file vpc-template.yaml --stack-name my-vpc-stack --capabilities CAPABILITY_NAMED_IAM
     ```
   - Or use the AWS Console to upload and launch the template.

4. **Outputs:**
   - The stack will output resource IDs for VPC, subnets, NAT Gateway, and more for reference and integration.

## Key Features

- Public and private subnets across two AZs
- Internet Gateway and NAT Gateway
- Public and private route tables
- VPC endpoints for S3 (Gateway) and Lambda (Interface)
- Example Security Groups and NACLs
- Lambda function deployed in VPC mode
- Private Route53 hosted zone
- Transit Gateway attachment (requires existing TGW)

## Notes

- The templates are for demonstration and learning. Adjust security group and NACL rules for production use.
- Ensure you have a valid IAM role for Lambda with VPC and logging permissions.
- For the Transit Gateway attachment, you must have a TGW already created and exported as `TransitGatewayId`.

## Cleanup

To avoid ongoing charges, delete the CloudFormation stack when finished:
```sh
aws cloudformation delete-stack --stack-name my-vpc-stack
```

## License

See [LICENSE](../LICENSE) for details.
