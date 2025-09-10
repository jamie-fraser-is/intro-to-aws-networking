# Introduction to AWS Networking

This repository contains AWS CloudFormation templates and resources to help you deploy and experiment with core AWS networking components, including VPCs, subnets, route tables, NAT gateways, VPC endpoints, security groups, NACLs, Lambda, and more.

## Repository Structure

- `templates/tgw-template.yaml`: Standalone Transit Gateway (TGW) stack. Deploys a TGW and exports its ID for use by other stacks.
- `templates/vpc-template.yaml`: Core network and VPC configuration. Deploys a VPC, subnets, NAT Gateway, endpoints, and attaches to the TGW using the output from `tgw-template.yaml`.
- `templates/alb-ecs-template.yaml`: Application Load Balancer and ECS Fargate service, referencing VPC and subnet outputs from `vpc-template.yaml`.

## Prerequisites

- An AWS account with permissions to create VPCs, subnets, IAM roles, Lambda functions, and related resources.
- AWS CLI installed and configured, or access to the AWS Management Console.
- **For GitHub Actions OIDC role-based deployments:**
  1. Create an AWS IAM role for GitHub Actions OIDC federation ("OIDC role") with trust policy for GitHub's OIDC provider and permissions to assume the CloudFormation deployment role.
  2. Create a separate AWS IAM role for CloudFormation deployments ("CloudFormation role") with permissions to create/update/delete the required AWS resources (including IAM roles, VPC, ECS, Lambda, etc). The OIDC role should be allowed to assume this role.
  3. Add the following GitHub repository secrets:
     - `AWS_OIDC_ROLE_ARN`: ARN of the OIDC role to be assumed by GitHub Actions.
     - `AWS_CLOUDFORMATION_ROLE_ARN`: ARN of the CloudFormation deployment role to be used by the AWS CLI `--role-arn` flag.

> **Note:** If you clone this repository for personal or organisational use, it is strongly recommended to add branch protection policies to your repository. This helps to safeguard your main branch and maintain code quality.

## OIDC & IAM Role Setup for GitHub Actions Deployment

Follow these steps to securely enable GitHub Actions deployments to AWS using OIDC and least-privilege IAM roles.

### 1. Create the GitHub OIDC Identity Provider in AWS

1. Go to the IAM console > Identity providers > Add provider.
2. Choose **Provider type**: `OpenID Connect`.
3. Set **Provider URL**: `https://token.actions.githubusercontent.com`
4. Set **Audience**: `sts.amazonaws.com`
5. Click **Add provider**.

### 2. Create the OIDC Role (GitHub Actions Assume Role)

This role is assumed by GitHub Actions via OIDC and can assume the CloudFormation deployment role.

**Trust policy JSON:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<YOUR_AWS_ACCOUNT_ID>:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
          "token.actions.githubusercontent.com:sub": "repo:<YOUR_GITHUB_ORG>/<YOUR_REPO>:ref:refs/heads/main"
        }
      }
    }
  ]
}
```
- Replace `<YOUR_AWS_ACCOUNT_ID>`, `<YOUR_GITHUB_ORG>`, and `<YOUR_REPO>` as appropriate.
- You can broaden the `sub` condition for more branches or workflows if needed.

**Permissions policy:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Resource": "arn:aws:iam::<YOUR_AWS_ACCOUNT_ID>:role/<CloudFormationDeploymentRoleName>"
    }
  ]
}
```

### 3. Create the CloudFormation Deployment Role

This role is assumed by the OIDC role and used for all AWS resource creation.

**Trust policy JSON:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<YOUR_AWS_ACCOUNT_ID>:role/<OIDCRoleName>"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

**Permissions policy (least privilege example):**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "cloudformation:*",
        "ec2:*",
        "iam:PassRole",
        "iam:GetRole",
        "iam:CreateRole",
        "iam:DeleteRole",
        "iam:AttachRolePolicy",
        "iam:DetachRolePolicy",
        "iam:PutRolePolicy",
        "iam:DeleteRolePolicy",
        "ecs:*",
        "elasticloadbalancing:*",
        "logs:*",
        "lambda:*",
        "route53:*",
        "acm:*"
      ],
      "Resource": "*"
    }
  ]
}
```
- Restrict resources further for production use.

### 4. Add Role ARNs to GitHub Secrets

- `AWS_OIDC_ROLE_ARN`: ARN of the OIDC role (step 2)
- `AWS_CLOUDFORMATION_ROLE_ARN`: ARN of the CloudFormation deployment role (step 3)

Add these as repository secrets in GitHub Settings > Secrets and variables > Actions.

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


4. **(Recommended) Deploy using GitHub Actions (OIDC role-based):**
   - This repository includes a GitHub Actions workflow to automate CloudFormation deployments in the correct order using OIDC and role-based access.
   - The workflow will:
     - Lint all templates.
     - Configure AWS credentials using the OIDC role (`AWS_OIDC_ROLE_ARN`).
     - Deploy the TGW stack (`tgw-stack`), then the VPC stack (`network-stack`), then the ALB/ECS stack (`alb-ecs-stack`), each using the CloudFormation deployment role (`AWS_CLOUDFORMATION_ROLE_ARN`).
     - Use fixed stack names so all cross-stack references work automatically.
   - To use this pipeline:
     1. Set up the required OIDC and CloudFormation roles in AWS (see Prerequisites above).
     2. Add the role ARNs as repository secrets (`AWS_OIDC_ROLE_ARN`, `AWS_CLOUDFORMATION_ROLE_ARN`).
     3. Push your changes to trigger the workflow.

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

## Troubleshooting: Misconfigured Services Stack

The `services-misconfigured.yaml` template is designed for learning and troubleshooting. All resources use the correct subnet assignments, so you do not need to redeploy any resources. Misconfigurations are limited to security groups, route tables, and NACLs. Below are the key misconfigurations and how to fix them using the AWS Console so they match the working `services-template.yaml`.

## 1. ALB Security Group: No Ingress Rule
**Misconfiguration:** The ALB security group does not allow inbound HTTP traffic (port 80).

**How to Fix in Console:**
1. Go to **EC2 > Security Groups**.
2. Find the security group named `ALBSecurityGroup`.
3. Edit **Inbound rules** and add:
   - Type: HTTP
   - Protocol: TCP
   - Port range: 80
   - Source: 0.0.0.0/0 (or restrict as needed)
4. Save the rule.

## 2. ECS Security Group: Allows traffic from ALB SG (but ALB SG has no ingress)
**Misconfiguration:** The ECS security group allows traffic from the ALB security group, but the ALB security group itself does not allow inbound traffic.

**How to Fix in Console:**
1. After fixing the ALB security group ingress (see #1), this rule will work as intended.
2. No change needed if ALB SG is fixed.

## 3. Lambda Security Group: Outbound Only
**Misconfiguration:** The Lambda security group only allows outbound HTTPS, but you may want to restrict or expand rules for your use case.

**How to Fix in Console:**
1. Go to **EC2 > Security Groups**.
2. Find the security group named `LambdaSG`.
3. Edit **Inbound/Outbound rules** as needed for your scenario.

## 4. Route Table: Missing NAT Gateway Route for Private Subnets
**Misconfiguration:** The route table for private subnets is missing a route to the NAT Gateway, so resources (like Lambda) cannot access the internet.

**How to Fix in Console:**
1. Go to **VPC > Route Tables**.
2. Find the route table named `MisconfiguredRouteTable` (or the one associated with your private subnets).
3. Edit routes and add:
   - Destination: `0.0.0.0/0`
   - Target: NAT Gateway (select the correct NAT Gateway for your VPC)
4. Save the route.

## 5. NACL: Overly Restrictive, Blocks All Traffic
**Misconfiguration:** The NACL associated with subnets is set to deny all inbound and outbound traffic.

**How to Fix in Console:**
1. Go to **VPC > Network ACLs**.
2. Find the NACL named `MisconfiguredNACL`.
3. Edit inbound and outbound rules:
   - Remove or modify the rules that deny all traffic.
   - Add rules to allow required traffic (e.g., allow HTTP/HTTPS, ephemeral ports, etc.)
4. Save the changes.

---

**Tip:** For a fully working deployment, compare your changes to the `services-template.yaml` and ensure all networking and security group settings match. Use the AWS Console to inspect and update resources as described above.

## Licence

See [LICENSE](../LICENSE) for details.
