---
title : "Introduction"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

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