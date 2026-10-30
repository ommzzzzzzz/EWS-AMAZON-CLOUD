# AWS VPC Peering Project

**Prepared for:** Om Prakash Behera  
**Project:** Private Connectivity Between Two AWS VPCs Using VPC Peering  
**Suggested AWS Region:** `ap-south-1` (Mumbai)

## Project overview

This project implements the AWS VPC Peering student workbook architecture: two non-overlapping VPCs, two private application instances, and one public bastion host in VPC A. The application instances have no public IPv4 addresses. VPC peering provides private IPv4 connectivity between the VPCs; explicit routes and security-group rules control which traffic is allowed.

> **Important:** This repository contains infrastructure code and a report template. It does not prove that the resources have been deployed. Deploy it in your own AWS account, run the tests, and add your own screenshots/real output to `evidence/` before submitting. Do not claim test results that you have not observed.

## Architecture and IP plan

```text
Your laptop
   | SSH TCP/22 to your public IPv4 /32
   v
Internet Gateway
   |
   v
VPC A: 10.10.0.0/16
  Public subnet: 10.10.1.0/24
    Bastion: 10.10.1.10 + public IPv4
       |\
       | \ SSH via ProxyJump to App-B (after peering)
       v  \
  Private subnet: 10.10.2.0/24      VPC Peering      VPC B: 10.20.0.0/16
    App-A: 10.10.2.10  <============================> Private subnet: 10.20.1.0/24
                                                        App-B: 10.20.1.10
```

| Resource | CIDR / private IP | Public IPv4? |
|---|---|---|
| `ewb-vpc-a` | `10.10.0.0/16` | N/A |
| `ewb-a-public` | `10.10.1.0/24` | N/A |
| `ewb-a-private` | `10.10.2.0/24` | N/A |
| `ewb-vpc-b` | `10.20.0.0/16` | N/A |
| `ewb-b-private` | `10.20.1.0/24` | N/A |
| `ewb-bastion` | `10.10.1.10` | Yes, dynamically assigned |
| `ewb-app-a` | `10.10.2.10` | No |
| `ewb-app-b` | `10.20.1.10` | No |

## Prerequisites

- An AWS account permitted to create VPC, EC2, security groups, routes, and key pairs.
- Terraform 1.5 or later and AWS CLI configured with credentials.
- An EC2 key pair already created in the selected region. Keep its private key on your own computer; never commit it to GitHub.
- Your current public IPv4 address in CIDR notation, such as `203.0.113.10/32` (replace this example with your actual address).

## Deploy

1. Open a terminal in `terraform/`.
2. Copy `terraform.tfvars.example` to `terraform.tfvars`.
3. Set `key_name` to an existing EC2 key pair name and `admin_cidr` to your current public IPv4 `/32`.
4. Run:

   ```bash
   terraform init
   terraform fmt -check
   terraform validate
   terraform plan
   terraform apply
   ```

5. Review the plan and estimated costs before approving `terraform apply`. The project can incur EC2, EBS, and public IPv4 charges.
6. Save the outputs. Connect to the bastion first, then use SSH ProxyJump for App-A/App-B. Example commands are in `docs/TESTING.md`.

## Validation requirements

Record actual results for:

1. Before peering routes are installed, a ping from App-A to App-B should fail (the Terraform configuration creates the final routes at apply time, so to demonstrate the baseline you must temporarily remove/withhold the peer routes in a controlled lab, or document the workbook baseline before adding them).
2. After routes and rules are active, App-A should ping App-B and App-B should ping App-A using private IPs.
3. SSH to App-B should work through the bastion using ProxyJump.
4. App-A and App-B must have no public IPv4 addresses.
5. Remove App-B's inbound ICMP rule for App-A temporarily; the ping should fail. Restore the rule and confirm recovery.

Store screenshots and terminal logs in `evidence/`. The expected results in the source workbook are acceptance criteria, not evidence that the tests have already run.

## Cleanup

After saving evidence, run:

```bash
cd terraform
terraform destroy
```

Review the plan carefully before confirming. Also check the AWS console for leftover resources and billing.

## Repository contents

- `terraform/` — Terraform infrastructure-as-code.
- `docs/REPORT.md` — project report template to complete with your actual resource IDs, results, and screenshots.
- `docs/TESTING.md` — SSH and connectivity test procedure.
- `evidence/README.md` — evidence checklist.

## Source basis

This project follows the uploaded *EWB Cloud Labs — AWS VPC Peering Student Project Workbook*, especially the architecture/IP plan (pages 3–4), network/security rules (page 6), routes (page 10), validation (page 11), deliverables (page 13), and cleanup (page 14).
