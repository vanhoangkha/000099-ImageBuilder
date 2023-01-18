---
title : "Autonomous Patching with EC2 Image Builder and System Manager"
date :  "`r Sys.Date()`" 
weight : 1
chapter : false
---

# Autonomous Patching with EC2 Image Builder and System Manager

{{% notice note %}}
For simplicity, we have used Sydney ‘ap-southeast-2’ as the default region for this lab. Please ensure all lab interaction is completed from this region.
{{% /notice %}}

#### Authors
- **Tim Robinson**, Well-Architected Geo Solutions Architect
- **Stephen Salim**, Well-Architected Geo Solutions Architect

#### Overview
Patching is a vital component to any security strategy which ensures that your compute environments are operating with the latest code revisions available. This in turn means that you are running with the latest security updates for the system, which reduces the potential attack surface of your workload.

The majority of compliance frameworks require evidence of patching strategy or some sort. This means that patching needs to be performed on a regular basis. Depending on the criticality of the workload, the operational overhead will need to be managed in a way that poses minimal impact to the workload’s availability.

Ensuring that you have an automated patching solution, will contribute to building a good security posture, while at the same time reducing the operational overhead, together with allowing traceability that can potentially be useful for future compliance audits.

There are multiple different approaches available to automate operating system patching using a combination of AWS services.

One approach is to utilize a blue/green deployment methodology to build an entirely new Amazon Machine Image (AMI) that contains the latest operating system patch, which can be deployed into the application cluster. This lab will walk you through this approach, utilizing a combination of the following services and features:
- EC2 Image Builder to automate creation of the AMI
- Systems Manager Automated Document to orchestrate the execution.
- CloudFormation with [AutoScalingReplacingUpdate](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-attribute-updatepolicy.html) update policy, to gracefully deploy the newly created AMI into the workload with minimal interruption to the application availability.

#### Goals
1. **EC2 Image Builder** configuration experience.
2. **Systems Manager Automated Document** experience.

#### Prerequisites
1. An AWS account that you are able to use for **testing**, that is not used for production or other purposes.

{{% notice note %}}
You will be billed for any applicable AWS resources used if you complete this lab that are not covered in the AWS Free Tier.
{{% /notice %}}

#### Steps
1. Base Infrastructure
2. Application Infrastructure
3. AMI Builder Pipeline
4. SSM Build Automation


#### Contents
Bài thực hành sẽ bao gồm các phần như sau:

| Thứ tự | Tên |
| ------ | --- |
| 1 | [Base Infrastructure](1-base-infrastructure/) |
| 2 | [Application Infrastructure](2-application-infrastructure/) |
| 3 | [AMI Builder Pipeline](3-ami-builder-pipeline/) |
| 4 | [SSM Build Automation](4-ssm-build-automation/) |
| 5 | [Teardown](5-teardown/) |