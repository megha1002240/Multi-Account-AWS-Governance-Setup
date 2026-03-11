# Multi-Account-AWS-Governance-Setup using AWS Organizations & SCP
# Project Overview

This project demonstrates how to implement centralized governance across multiple AWS accounts using AWS Organizations, Organizational Units (OUs), and Service Control Policies (SCPs).
The goal is to enforce security, cost control, and compliance policies across Development, Testing, and Production environments.

# Scenario

An enterprise organization manages multiple AWS accounts for different teams:

1. Development Team
2. Testing Team
3. Production Team
However, there are no centralized restrictions. Developers are launching expensive resources, security policies are inconsistent, and compliance requirements are not enforced.
To solve this, centralized governance is implemented using AWS native services.

# Architecture
<img width="1154" height="837" alt="image" src="https://github.com/user-attachments/assets/7a507da5-2120-4bb2-82d8-8d9a45cf3bd7" />


Organization Structure:
```
   Root
│
├── Dev OU
│   └── Dev Account
│
├── Test OU
│   └── Test Account
│
└── Prod OU
    └── Prod Account
```
<img width="1600" height="840" alt="image" src="https://github.com/user-attachments/assets/897111a2-d01a-4bcd-a2e4-e61d240bc8fc" />
---

# Technologies Used

1. AWS Organizations
2. Service Control Policies (SCP)
3. IAM
4. CloudTrail

# Step 1 – Create AWS Organization

1. Login to the AWS Management Account.
2. Open AWS Organizations.
3. Enable All Features.
# Create Organizational Units:
* Dev OU
* Test OU
* Prod OU

<img width="1600" height="840" alt="image" src="https://github.com/user-attachments/assets/ae056600-188b-4281-987e-63610bdb38c9" />

# Step 2 - Create aws account
1. Go to AWS Organizations.
2. click on Add aws account
3. create a Account
* Devaccount
   <img width="1920" height="1008" alt="image" src="https://github.com/user-attachments/assets/1c72a747-0d1f-4692-81d7-d74e7a0ffee4" />

* ProdAccount
   <img width="1600" height="840" alt="image" src="https://github.com/user-attachments/assets/02a276dd-04d3-478c-8762-cec2d418e49c" />

* TestAccount
   <img width="1600" height="840" alt="image" src="https://github.com/user-attachments/assets/e0a4b0e0-cec6-4465-9da1-a9f4352b35e4" />

4. Move member accounts under the respective OUs.
   * DevAccount---->Dev-OU
   * ProdAccount---->Prod-OU
   * TestAccount----->Test-OU

# Step 3 – Create Service Control Policies (SCP)
# 1. Deny Large EC2 Instances in Dev Environment
   * Open AWS Organizations.
   * Select policy
   * Open Service Control Policies
   * Click on create Service Control Policies 
   * Add a Policies
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowOnlyMicroInstances",
      "Effect": "Deny",
      "Action": "ec2:RunInstances",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "ec2:InstanceType": [
            "t2.micro",
            "t3.micro"
          ]
        }
      }
    }
  ]
}
```
<img width="1600" height="840" alt="image" src="https://github.com/user-attachments/assets/6b40ce81-4990-4ff2-af35-827cf665dc15" />

Purpose: Prevent developers from launching expensive EC2 instances.
Attached to:
Dev-OU

# 2. Prevent Disabling CloudTrail
   * Open AWS Organizations.
   * Select policy
   * Open Service Control Policies
   * Click on create Service Control Policies
   * Add a Policies
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PreventCloudTrailDisable",
      "Effect": "Deny",
      "Action": [
        "cloudtrail:StopLogging",
        "cloudtrail:DeleteTrail"
      ],
      "Resource": "*"
    }
  ]
}
```
<img width="1600" height="840" alt="image" src="https://github.com/user-attachments/assets/a7aa4b53-7c84-418a-8341-db69b6fb4fc7" />

Purpose: Ensure audit logs cannot be disabled.
Attached to:
root

# 3. Restrict AWS Regions
  * Open AWS Organizations.
  * Select policy
  * Open Service Control Policies
  * Click on create Service Control Policies
  * Add a Policies
Allowed Regions:
ap-south-1 (Mumbai)
us-east-1 (N. Virginia)
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyUnapprovedRegions",
      "Effect": "Deny",
      "NotAction": [
        "iam:*",
        "organizations:*",
        "route53:*",
        "cloudfront:*",
        "support:*"
      ],
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": [
            "ap-south-1",
            "us-east-1"
          ]
        }
      }
    }
  ]
}
```
<img width="1600" height="840" alt="image" src="https://github.com/user-attachments/assets/e8303452-8814-46a2-877e-ec6f984a7809" />
Purpose: Allow resource creation only in approved regions.
Attached to:
root

# Final Governance Model
```
Root
 ├── SCP
 │    ├── Protect-CloudTrail
 │    └── Restrict-Regions
 │
 ├── Dev-OU
 │    ├── SCP
 │    │    └── Deny-Large-EC2-Dev
 │    └── DevAccount
 │
 ├── Test-OU
 │    └── TestAccount
 │
 └── Prod-OU
      └── ProdAccount
```
# All Policies are created
<img width="1920" height="1008" alt="image" src="https://github.com/user-attachments/assets/04be67d5-b7c3-402f-afdf-646e86c102a6" />

# Validation

To validate governance policies:

# Test 1 – EC2 Restriction

1. Open Swich role
2. Login to the Dev Account.
<img width="1920" height="1008" alt="image" src="https://github.com/user-attachments/assets/150f58ed-727b-4017-a16a-c803dd5b5b50" />

3. Try launching an EC2 instance with instance type:
t2.large
<img width="1600" height="840" alt="image" src="https://github.com/user-attachments/assets/d1b74fdf-51f5-4379-9e03-46cf7c3f0d2b" />
4. Result
<img width="1600" height="840" alt="image" src="https://github.com/user-attachments/assets/633ca197-1f7f-40fb-935d-8df45ab58d47" />

# Test 2 – CloudTrail Protection

Attempt to stop CloudTrail logging.


Result:

Access Denied
<img width="1600" height="840" alt="image" src="https://github.com/user-attachments/assets/4cac2882-a260-44a4-8105-d8f12b77749c" />


# Test 3 – Region Restriction

Attempt to launch resources in an unapproved region such as:
ap-sountheast-2

Result:
<img width="1600" height="840" alt="image" src="https://github.com/user-attachments/assets/cb79d9e9-1dea-4ea1-a82b-f8db1ca78aa8" />

---
Author

Megha Patil
Cloud & DevOps Engineer
Pune, India

---
