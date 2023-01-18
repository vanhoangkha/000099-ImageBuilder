---
title : "Hạ tầng Cơ bản"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---


**Triển khai Hạ tầng Cơ bản**

Ở phần 1, chúng ta sẽ tiến hành triển khai các thành phần hạ tầng cơ bản như sau. Các thành phần này sẽ là kiến trúc cơ sở để có thể triển khai ứng dụng và thực hiện tự động hoá ở các phần kế tiếp.
- 1 VPC
- Public & Private Subnets across 2 AZs
- Internet Gateway
- NAT Gateway
- Route Tables

Khi hạ tầng được chuẩn bị hoàn tất, kiến trúc triển khai sẽ tương đương với biểu đồ sau:

![infrastructure-base-architecture](/images/2-infrastructure-base-architecture.png?featherlight=false&width=60pc)

**Nội dung**
- [CloudFormation Stack](#cloudformation-stack)
- [AWS CLI](#aws-cli)

#### CloudFormation Stack
Để tiến hành triển khai hạ tầng, chúng ta sẽ sử dụng dịch vụ **AWS CloudFormation** thông qua AWS Console hoặc AWS CLI.

| Thành Phần | Giá Trị (Bắt buộc) |
| ---------- | ------- |
| Stack Name | pattern3-base |
| Template URL | [pattern3-base.yml](https://www.wellarchitectedlabs.com/Security/300_Autonomous_Patching_With_EC2_Image_Builder_and_Systems_Manager/Code/templates/section1/pattern3-base.yml) |

Hoặc bạn có thể tải về template bên dưới:

{{%attachments title="Template" pattern=".*(yml)"/%}}

#### AWS CLI
Sau đây là các bước khởi tạo thông qua AWS CLI:
1. Khởi tạo CloudFormation Stack
   ```
   aws cloudformation create-stack --stack-name pattern3-base --template-body file://pattern3-base.yml --region ap-southeast-2
   ```

![cloudformation-cli-create-stack](/images/2-cloudformation-cli-create-stack.png?featherlight=false&width=90pc)

2. Xác nhận CloudFormation Stack đã khởi tạo hoàn tất với *StackStatus* là `CREATE_COMPLETE`.
   ```
   aws cloudformation describe-stacks --stack-name pattern3-base --region ap-southeast-2
   ```

3. Ghi chú lại các giá trị tại *Output*.

![cloudformation-cli-describe-stack](/images/2-cloudformation-cli-describe-stack.png?featherlight=false&width=90pc)