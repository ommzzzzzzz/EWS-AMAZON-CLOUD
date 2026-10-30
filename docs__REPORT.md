# AWS VPC Peering Project Report

**Student name:** Om Prakash Behera  
**Project title:** Private Connectivity Between Two AWS VPCs Using VPC Peering  
**AWS Region:** [Fill in the region actually used]  
**Completion date:** [Fill in after completing the lab]

## 1. Objective

Build a private IPv4 connection between two non-overlapping Amazon VPCs using VPC peering. Keep App-A and App-B private, use one public bastion host for SSH administration, and verify routing and security controls through positive and negative tests.

## 2. Architecture

VPC A uses `10.10.0.0/16` and contains a public subnet (`10.10.1.0/24`) for the bastion and a private subnet (`10.10.2.0/24`) for App-A. VPC B uses `10.20.0.0/16` and contains a private subnet (`10.20.1.0/24`) for App-B. A VPC peering connection joins the VPCs. App-A and App-B do not receive public IPv4 addresses.

Insert your architecture screenshot or diagram here.

## 3. Resource inventory

| Resource | Actual ID / address | Evidence filename |
|---|---|---|
| VPC A | [Fill in] | [Fill in] |
| VPC B | [Fill in] | [Fill in] |
| Peering connection | [Fill in] | [Fill in] |
| Bastion instance / public IP | [Fill in] | [Fill in] |
| App-A instance / private IP | [Fill in] | [Fill in] |
| App-B instance / private IP | [Fill in] | [Fill in] |

## 4. Route design

- Public route table in VPC A: local VPC route, default route `0.0.0.0/0` to the internet gateway, and `10.20.0.0/16` to the peering connection.
- Private route table for App-A: local VPC route and `10.20.0.0/16` to the peering connection.
- Private route table for App-B: local VPC route and `10.10.0.0/16` to the peering connection.

The forward route and return route are both required. Peering alone does not automatically add routes or security permissions.

## 5. Security controls

- Bastion SSH ingress is limited to the student's public IPv4 `/32`.
- App-A and App-B SSH ingress is limited to the bastion private IP `10.10.1.10/32`.
- ICMP echo requests are limited to the other private application host's `/32`.
- App-A and App-B have no public IPv4 addresses.
- The lab uses default network ACLs and broad outbound rules; these should be reviewed for production use.

## 6. Test results

**Do not fill in expected outcomes as if they were real results. Replace each status with your observed result and attach evidence.**

| Test | Command / location | Actual result | Evidence |
|---|---|---|---|
| Baseline before peer routes | App-A: `ping -c 4 10.20.1.10` | [Observed result] | [Filename] |
| Forward private ping | App-A: `ping -c 4 10.20.1.10` | [Observed result] | [Filename] |
| Reverse private ping | App-B: `ping -c 4 10.10.2.10` | [Observed result] | [Filename] |
| Private SSH | Laptop: `ssh ewb-app-b` | [Observed result] | [Filename] |
| Isolation | EC2 details for App-A and App-B | [Confirm no public IPv4] | [Filename] |
| Controlled ICMP failure | Remove App-B ICMP ingress and ping from App-A | [Observed result] | [Filename] |
| Recovery | Restore App-B ICMP ingress and ping from App-A | [Observed result] | [Filename] |

## 7. Troubleshooting notes

Record any issue and how it was resolved: [Complete after testing]. Check peering status, route-table associations, routes in both directions, actual private IPs, security-group source addresses, key pair, and the bastion's current public IP.

## 8. What I learned

[Write a short reflection about non-overlapping CIDRs, route selection, return routes, security-group rules, bastion/ProxyJump access, and why a successful peering status alone does not prove connectivity.]

## 9. Cleanup

Record the date and evidence that the three EC2 instances, peering connection, lab route tables, subnets, security groups, internet gateway, and both VPCs were deleted. Check for leftover EBS volumes and public IPv4 allocations.

## 10. Conclusion

[Summarize the result based on tests actually completed.]
