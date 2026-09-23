---
title: "Cleanup"
date: 2024-01-01
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

After recording all screenshots and demo videos for the capstone, I tore the **CloudNote** stack down in the correct dependency order to keep the account tidy and avoid charges. For each AWS resource I opened the relevant console page and selected delete/terminate.

**Part B first (highest priority — the ALB is billed by the hour):**

1. **EC2 → Auto Scaling Groups** → select `cloudnote-asg` → **Delete** (instances are terminated automatically).
2. **EC2 → Load Balancers** → select `cloudnote-alb` → **Actions → Delete**.
3. **EC2 → Target Groups** → select `cloudnote-tg` → **Delete**.
4. **EC2 → Launch Templates** → select `cloudnote-web-template` → **Delete**.
5. **EC2 → Instances** → terminate any leftover independent instance.
6. **VPC → Your VPCs** → select `cloudnote-vpc` → **Actions → Delete VPC** (wizard removes subnets, route tables, IGW).
7. **EC2 → Key Pairs** → delete `cloudnote-key` if no longer needed.

**Part A:**

8. **CloudTrail → Trails** → `cloudnote-audit-trail` → **Delete** (may keep it — first trail's management events are free).
9. **CloudWatch** → delete alarm `notes-api-error-alarm`; delete dashboard `CloudNote-Dashboard`.
10. **API Gateway** → `notes-http-api` → **Delete**.
11. **Lambda** → `notes-api` → **Delete**.
12. **IAM → Roles** → delete `LambdaNotesExecutionRole`.
13. **DynamoDB** → table `Notes` → **Delete table**.
14. **S3** → empty bucket `cloudnote-app-0205568-2026` (required before deletion) → **Delete bucket**; repeat for the CloudTrail log bucket if the trail was deleted.
15. (Optional) **IAM** → delete practice user `cloudnote-dev` or keep it for later workshops.

**Final check:** open **AWS Billing → Bills / Cost Explorer** and confirm no running resource generates charges (especially EC2 and ELB).