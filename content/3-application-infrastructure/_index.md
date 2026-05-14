---
title : "Application Infrastructure"
date: 2024-01-01
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

**Deploy the Application Infrastructure**

Tại phần này, chúng ta sẽ tiến hành tạo một **Application Stack** dựa trên **Base Infrastructure** ở phần trước, tài nguyên sẽ bao gồm như sau:
1. Applicaion Load Balancer
2. Auto-Scaling Group & Launch Configuration

![application-architecture](/images/3-application-architecture.png?featherlight=false&width=60pc)

Nếu chúng ta khởi tạo các tài nguyên trên một cách thủ công, tổng thời gian sẽ khá dài và không thể đạt được mục tiêu mà chúng ta đã đề ra với việc tự động hoá. Để tiết kiệm thời gian cũng như tự động hoá quá trình khởi tạo, **CloudFormation Stack** cần được triển khai như sau.

**Contents**
- [CloudFormation Stack](#cloudformation-stack)
- [AWS CLI](#aws-cli)
- [Review CloudFormation Stack Output](#review-cloudformation-stack-output)

#### CloudFormation Stack
Để tiến hành triển khai hạ tầng, chúng ta sẽ sử dụng dịch vụ **AWS CloudFormation** thông qua AWS Console hoặc AWS CLI.

| Thành Phần | Giá Trị (Bắt buộc) |
| ---------- | ------- |
| Stack Name | pattern3-app |
| Template URL | [pattern3-application.yml](https://www.wellarchitectedlabs.com/Security/300_Autonomous_Patching_With_EC2_Image_Builder_and_Systems_Manager/Code/templates/section2/pattern3-application.yml) |

Hoặc bạn có thể tải về template bên dưới:

{{%attachments title="Template" pattern=".*(yml)"/%}}

#### AWS CLI
Sau đây là các bước khởi tạo thông qua AWS CLI:
1. Khởi tạo CloudFormation Stack
   ```
   aws cloudformation create-stack --stack-name pattern3-app --template-body file://pattern3-application.yml --parameters ParameterKey=AmazonMachineImage,ParameterValue=ami-0f96495a064477ffb ParameterKey=BaselineVpcStack,ParameterValue=pattern3-base --capabilities CAPABILITY_IAM --region ap-southeast-2
   ```   
![cloudformation-cli-create-stack](/images/3-cloudformation-cli-create-stack.png?featherlight=false&width=90pc)
2. Xác nhận CloudFormation Stack đã khởi tạo hoàn tất với *StackStatus* là `CREATE_COMPLETE`.
   ```
   aws cloudformation describe-stacks --stack-name pattern3-app --region ap-southeast-2
   ```

3. Ghi chú lại các giá trị tại *Output*.

![cloudformation-cli-describe-stack](/images/3-cloudformation-cli-describe-stack.png?featherlight=false&width=90pc)

{{% notice tip %}}
Bài thực hành dựa trên *Golden AMI ID* của **Amazon Linux 2 AMI (HVM)** tại AWS Region `ap-southeast-2`, nếu bạn muốn sử dụng AWS Region khác, giá trị *Golden AMI ID* cần được thay thế một cách tương ứng.
{{% /notice %}}

#### Review CloudFormation Stack Output
Một khi CloudFormation Stack khởi tạo, chúng ta sẽ tiến hành xác minh liệu ứng dụng đã được triển khai thành công hay chưa?

1. Từ CloudFormation Stack *Output*, tìm giá trị của `OutputPattern3ALBDNSName`.
2. Thử truy cập trên trình duyệt web và kiểm tra kết quả có tiêu đề tương tự `‘Welcome to Re:Invent 2020 The Well Architected Way’` hay không?

![cloudformation-output-alb-dns-url-verification](/images/3-cloudformation-output-alb-dns-url-verification.png?featherlight=false&width=90pc)

3. Tại đường dẫn URL trên trình duyệt web, chúng ta tiến hành thêm đường dẫn sau - `/details.php`.
4. Kiểm tra xem kết quả liệu có phải là danh sách các phần mềm được cài đặt? Sao lưu lại các dữ liệu hiển thị bao gồm *Amazon Image Id* và *Installed Packages*.

![alb-dns-url-details-ami](/images/3-alb-dns-url-details-ami.png?featherlight=false&width=90pc)

5. Tiến hành di chuyển tới phần kế tiếp với **AMI Builder Pipeline**.