
# Introduction to AWS Networking

This repository contains AWS CloudFormation templates and resources to help you deploy and experiment with core AWS networking components, including VPCs, subnets, route tables, NAT gateways, VPC endpoints, security groups, NACLs, Lambda, and more.


## Repository Structure

- `examples/vpc-template.yaml`: Core network and VPC configuration.
- `examples/tgw-template.yaml`: Template for deploying a Transit Gateway (TGW) and related resources. Deploy this first if you want to use the TGW attachment in the VPC template.


## Prerequisites

- An AWS account with permissions to create VPCs, subnets, IAM roles, Lambda functions, and related resources.
- AWS CLI installed and configured, or access to the AWS Management Console.
- If using the GitHub Actions pipeline, you must add your AWS credentials as GitHub repository secrets:
  - `AWS_ACCESS_KEY_ID`
  - `AWS_SECRET_ACCESS_KEY`
  - (Optional) `AWS_SESSION_TOKEN` if using temporary credentials

> **Note:** If you clone this repository for personal or organisational use, it is strongly recommended to add branch protection policies to your repository. This helps to safeguard your main branch and maintain code quality.



## Deployment Steps

1. **Clone this repository:**
   ```sh
   git clone https://github.com/<your-org-or-username>/intro-to-aws-networking.git
   cd intro-to-aws-networking/examples
   ```

2. **Review and edit parameters:**
   - Open the template YAML files and adjust parameter defaults (e.g., CIDR blocks, subnet sizes) as needed for your environment.
   - For Lambda deployment, update the `Role` property in the Lambda function resource to use a valid IAM role ARN from your AWS account.
   - If using the Transit Gateway attachment, ensure you have a TGW created and exported as `TransitGatewayId`.


3. **Deploy the template:**
   - Using AWS CLI:
     ```sh
     aws cloudformation deploy --template-file vpc-template.yaml --stack-name my-vpc-stack --capabilities CAPABILITY_NAMED_IAM
     ```
   - Or use the AWS Console to upload and launch the template.
   - If you want to use the Transit Gateway attachment, deploy `tgw-template.yaml` first, then deploy `vpc-template.yaml`.

4. **(Optional) Deploy using GitHub Actions:**
   - This repository includes a GitHub Actions workflow to automate CloudFormation deployments.
   - The workflow will:
     - Check out your code on push or pull request.
     - Set up AWS credentials from repository secrets.
     - Validate the CloudFormation templates.
     - Deploy the specified template (e.g., `vpc-template.yaml`) to your AWS account using the AWS CLI.
   - To use this pipeline:
     1. Add your AWS credentials as repository secrets (see Prerequisites above).
     2. Edit the workflow file (in `.github/workflows/`) to specify the correct template and stack name if needed.
     3. Push your changes to trigger the workflow.

5. **Check outputs:**
   - The stack will output resource IDs for VPC, subnets, NAT Gateway, Lambda, etc. Use these for integration or further automation.


## Customisation

- **IAM Roles:**
  - Replace the placeholder Lambda execution role ARN with one from your account that has the necessary permissions (VPC access, logging, etc).
- **Security Groups & NACLs:**
  - Adjust the example rules to fit your security requirements. The provided rules are for demonstration and may be overly permissive for production.
- **Parameters:**
  - Change CIDR blocks, subnet sizes, and naming to fit your network plan.

## Clean up

To avoid ongoing AWS charges, delete the CloudFormation stack when finished:
```sh
aws cloudformation delete-stack --stack-name my-vpc-stack
```

## Licence

See [LICENSE](../LICENSE) for details.
