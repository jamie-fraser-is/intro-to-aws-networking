
# Introduction to AWS Networking

This repository contains AWS CloudFormation templates and resources to help you deploy and experiment with core AWS networking components, including VPCs, subnets, route tables, NAT gateways, VPC endpoints, security groups, NACLs, Lambda, and more.


## Repository Structure

- `templates/tgw-template.yaml`: Standalone Transit Gateway (TGW) stack. Deploys a TGW and exports its ID for use by other stacks.
- `templates/vpc-template.yaml`: Core network and VPC configuration. Deploys a VPC, subnets, NAT Gateway, endpoints, and attaches to the TGW using the output from `tgw-template.yaml`.
- `templates/alb-ecs-template.yaml`: Application Load Balancer and ECS Fargate service, referencing VPC and subnet outputs from `vpc-template.yaml`.


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
   - The templates are designed for automated deployment and cross-stack referencing. No manual parameter input is required for stack dependencies if you use the provided workflow and stack names.


3. **Deploy the templates in order:**
   - Using AWS CLI (recommended stack names):
     ```sh
     aws cloudformation deploy --template-file templates/tgw-template.yaml --stack-name tgw-stack --capabilities CAPABILITY_NAMED_IAM
     aws cloudformation deploy --template-file templates/vpc-template.yaml --stack-name network-stack --capabilities CAPABILITY_NAMED_IAM
     aws cloudformation deploy --template-file templates/alb-ecs-template.yaml --stack-name alb-ecs-stack --capabilities CAPABILITY_NAMED_IAM
     ```
   - Or use the AWS Console to upload and launch the templates in the same order.

4. **(Recommended) Deploy using GitHub Actions:**
   - This repository includes a GitHub Actions workflow to automate CloudFormation deployments in the correct order.
   - The workflow will:
     - Lint all templates.
     - Deploy the TGW stack (`tgw-stack`), then the VPC stack (`network-stack`), then the ALB/ECS stack (`alb-ecs-stack`).
     - Use fixed stack names so all cross-stack references work automatically.
   - To use this pipeline:
     1. Add your AWS credentials as repository secrets (see Prerequisites above).
     2. Push your changes to trigger the workflow.

5. **Check outputs:**
   - Each stack will output resource IDs (e.g., TGW ID, VPC ID, subnet IDs, ALB DNS name, ECS service name) for integration or further automation. Outputs are exported for cross-stack referencing.


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
