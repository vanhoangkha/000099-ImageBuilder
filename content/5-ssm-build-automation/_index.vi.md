---
title : "Tự động hóa với SSM"
date : "`r Sys.Date()`"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

**Triển khai Tự động hóa với SSM**

Sau khi đã hoàn thành **Builder Pipeline** ở phần trước, chúng ta sẽ tiến đến giai đoạn tự động hoá triển khai với dịch vụ System Manager. Với mục tiêu sẽ triển khai **AMI** mới đã hoàn thành **OS Patching** cho hạ tầng ứng dụng hiện tại, chúng ta sẽ sử dụng **Automation Document**.

Với **Automation Document**, những hoạt động sau sẽ lần lượt được thực thi:
- Tự động thực thi **Builder Pipeline**.
- Tạo **AMI** mới đã hoàn thành **OS Patching** sau khi **Builder Pipeline** hoàn tất.
- Cập nhật **CloudFormation Stack** với giá trị **AMI** mới.
- Tự động kích hoạt chính sách `AutoScalingReplacingUpdate` nhằm triển khai phương pháp **Blue/Green Deployment** cho **Auto-Scaling Group**.

{{% notice tip %}}
Với cách tiếp cận này, chúng ta sẽ giảm thiểu tối da tổng thời gian từ khởi tạo **AMI** cho đến cập nhật hạ tầng ứng dụng, điều này cực kỳ hữu ích và tránh gián đoạn đến trải nghiệm người dùng.
{{% /notice %}}

Bên cạnh đó, điểm mấu chốt của quá trình tự động hoá này chính là chính sách `AutoScalingReplacingUpdate` nhằm đơn giản hoá tính phức tạp nếu triển khai phương pháp **Blue/Green Deployment** một cách thủ công. Để dễ dàng hình dung hơn, chúng ta sẽ xem xét quá trình sau:
- Đầu tiên, CloudFormation sẽ phát hiện có bản cập nhật mới cho **LaunchConfiguration** với giá trị **AMI** mới.
- CloudFormation tiến hành tạo một **Auto-Scaling Group** mới cùng với **AMI** mới đã hoàn thành **OS Patching**.
- CloudFormation tiến hành đợi cho đến khi các EC2 instances được kiểm tra bởi **Load Balanacer** và trả về kết quả `Health Check` là `Healthy`.
  - Khi kết quả là `Healthy`, CloudFormation tiến hành thay thế và sử dụng **Auto-Scaling Group** mới.
  - Khi kết quả là `Un-Healthy`, CloudFormation tiến hành quay ngược quá trình cập nhật và giữ tài nguyên hiện tại.

> Chúng ta có thể tham khảo thêm về đặc tả quá trình trên thông qua **CloudFormation Template** ở nội dung kế tiếp.

![5-automate-architecture](/images/5-automate-architecture.png?featherlight=false&width=60pc)

**Nội dung**
- [CloudFormation Stack](#cloudformation-stack)
- [AWS CLI](#aws-cli)
- [Chuẩn bị Automation Document](#chuẩn-bị-automation-document)
- [Chuẩn bị Monitoring Script](#chuẩn-bị-monitoring-script)
- [Thực thi Automation Document](#thực-thi-automation-document)
- [Xác Minh AMI ID](#xác-minh-ami-id)

#### CloudFormation Stack
Để tiến hành triển khai hạ tầng, chúng ta sẽ sử dụng dịch vụ **AWS CloudFormation** thông qua AWS Console hoặc AWS CLI.

| Thành Phần | Giá Trị (Bắt buộc) |
| ---------- | ------- |
| Stack Name | pattern3-automate |
| Template URL | [pattern3-automate.yml](https://www.wellarchitectedlabs.com/Security/300_Autonomous_Patching_With_EC2_Image_Builder_and_Systems_Manager/Code/templates/section4/pattern3-automate.yml) |

Hoặc bạn có thể tải về template bên dưới:

{{%attachments title="Template" pattern=".*(yml)"/%}}

#### AWS CLI
Sau đây là các bước khởi tạo thông qua AWS CLI:
1. Khởi tạo **CloudFormation Stack**
  ```
  aws cloudformation create-stack --stack-name pattern3-automate --template-body file://pattern3-automate.yml --parameters  ParameterKey=ApplicationStack,ParameterValue=pattern3-app ParameterKey=ImageBuilderPipelineStack,ParameterValue=pattern3-pipeline --capabilities CAPABILITY_IAM --region ap-southeast-2
  ```

![cloudformation-create-stack](/images/5-cloudformation-create-stack.png?featherlight=false&width=90pc)

2. Xác nhận CloudFormation Stack đã khởi tạo hoàn tất với *StackStatus* là `CREATE_COMPLETE`.
  ```
  aws cloudformation desribe-stacks --stack-name pattern3-automate --region ap-southeast-2
  ```
3. Ghi chú lại các giá trị tại *Output*.

![cloudformation-describe-stack](/images/5-cloudformation-describe-stack.png?featherlight=false&width=90pc)

#### Chuẩn bị Automation Document

---
**Các Bước Khởi Tạo**

1. Truy cập vào dịch vụ [System Manager](https://ap-southeast-2.console.aws.amazon.com/systems-manager/home?region=ap-southeast-2).
2. Ở thanh điều hướng bên tay trái, ở phần *Shared Resources*, chọn **Documents**.

![5-ssm-documents](/images/5-ssm-documents.png?featherlight=false&width=90pc)

3. Nhấn nút `Create Automation`.
4. Điền tên và chọn chế độ Editor để tiến hành điền đặc tả từ Console.
5. Nhấn nút `Create Automation` để tiến hành khởi tạo.
6. Trước khi triển khai **Automation Document** này, chúng ta cần phải chuẩn bị **Monitoring Script** ở phần kế tiếp.

![ssm-documents-create-automation-editor](/images/5-ssm-documents-create-automation-editor.png?featherlight=false&width=90pc)

![ssm-documents-automation](/images/5-ssm-documents-automation.png?featherlight=false&width=90pc)

{{% notice tip %}}
Chúng ta có thể tải về chi tiết đặc tả tại [Automation Document Script URL](https://www.wellarchitectedlabs.com/Security/300_Autonomous_Patching_With_EC2_Image_Builder_and_Systems_Manager/Code/scripts/section4_ssm_automation_document.yml) sau.
{{% /notice %}}

---
**Chi Tiết Đặc Tả**

Dưới đây là chi tiết đặc tả mà chúng ta sẽ điền vào:
1. Đầu tiên chúng ta sẽ xác định `schemaVersion` và định nghĩa các `parameters`.
   1. `ImageBuilderPipelineARN`: tham khảo **Builder Pipeline** mà chúng ta đã tạo ở phần trước.
   2. `ApplicationStack`: tham khảo tên CloudFormation Stack mà chúng ta đã tạo ở phần trước - `pattern3-app`.
2. Kế đến là đặc tả `mainSteps`, bằng việc sử dụng các giá trị của `parameters`, chúng ta có thể tạo hành động tên là *ExecuteImageCreation* thông qua AWS API `aws:executeAwsApi`.
3. Sau đó, chúng ta cần đợi cho đến khi *ExecuteImageCreation* hoàn tất với AWS API `aws:waitForAwsResourceProperty`.
4. Một khi *ExecuteImageCreation* hoàn tất, chúng ta tiến hành tạo hành động tên là *GetBuiltImage* nhắm lấy được các thông tin cần thiết là **AMI ID** và truyền giá trị này đến bước kế tiếp.
5. Sau khi có được **AMI ID**, chúng ta sẽ tạo hành động tiến hành cập nhật CloudFormation Stack tên là *UpdateCluster*.
6. Tiến hành đợi một khi quá trình cập nhật hoàn tất với trạng thái `UPDATE_COMPLETE`.

```yml
description: CreateImage
schemaVersion: '0.3'
parameters:
  ImageBuilderPipelineARN:
    description: (Required) Corresponding EC2 Image Builder Pipeline to execute.
    type: String
  ApplicationStack:
    description: (Required) Corresponding Application Stack to Deploy the Image to.
    type: String
mainSteps:
  - name: ExecuteImageCreation
    action: aws:executeAwsApi
    maxAttempts: 10
    timeoutSeconds: 3600
    onFailure: Abort
    inputs:
      Service: imagebuilder
      Api: StartImagePipelineExecution
      imagePipelineArn: '{{ ImageBuilderPipelineARN }}'
    outputs:
    - Name: imageBuildVersionArn
      Selector: $.imageBuildVersionArn
      Type: String
  - name: WaitImageComplete
    action: aws:waitForAwsResourceProperty
    maxAttempts: 10
    timeoutSeconds: 3600
    onFailure: Abort
    inputs:
      Service: imagebuilder
      Api: GetImage
      imageBuildVersionArn: '{{ ExecuteImageCreation.imageBuildVersionArn }}'
      PropertySelector: image.state.status
      DesiredValues: 
        - AVAILABLE
- name: GetBuiltImage
  action: aws:executeAwsApi
  maxAttempts: 10
  timeoutSeconds: 3600
  onFailure: Abort
  inputs:
    Service: imagebuilder
    Api: GetImage         
    imageBuildVersionArn: '{{ ExecuteImageCreation.imageBuildVersionArn }}'
  outputs:
  - Name: image
    Selector: $.image.outputResources.amis[0].image
    Type: String
- name: UpdateCluster
  action: aws:executeAwsApi
  maxAttempts: 10
  timeoutSeconds: 3600
  onFailure: Abort
  inputs:
    Service: cloudformation
    Api: UpdateStack
    StackName: '{{ ApplicationStack }}'
    UsePreviousTemplate: true
    Parameters:
      - ParameterKey: BaselineVpcStack
        UsePreviousValue: true
      - ParameterKey: AmazonMachineImage
        ParameterValue: '{{ GetBuiltImage.image }}'
    Capabilities:
      - CAPABILITY_IAM
- name: WaitDeploymentComplete
  action: aws:waitForAwsResourceProperty
  maxAttempts: 10
  timeoutSeconds: 3600
  onFailure: Abort
  inputs:
    Service: cloudformation
    Api: DescribeStacks
    StackName: '{{ ApplicationStack }}'
    PropertySelector: Stacks[0].StackStatus
    DesiredValues: 
      - UPDATE_COMPLETE
```

#### Chuẩn bị Monitoring Script
Chúng ta tiến hành chuẩn bị một quá trình theo dõi cơ bản nhằm liên tục gửi tin nhắn đến đường dẫn URL của **Application Load Balancer** trước khi thực thi **Automation Document**. Khi đó, chúng ta có thể theo dõi hoàn toàn quá trình trước và sau khi thay đổi.

{{% notice tip %}}
Chúng ta có thể tải về [Monitoring Script URL](https://www.wellarchitectedlabs.com/Security/300_Autonomous_Patching_With_EC2_Image_Builder_and_Systems_Manager/Code/scripts/watchscript.sh) sau.
{{% /notice %}}

```bash
./watchscript.sh http://<ALB_DNS_URL>
```

![monitoring-script](/images/5-monitoring-script.png?featherlight=false&width=90pc)

Đoạn mã sẽ liên tục gửi và lấy kết quả trả về cùng với `StatusCode`.

#### Thực thi Automation Document
1. Một khi **Monitoring Script** được chuẩn bị, chúng ta tiến hành thực thi **Automation Document**.
  ```
  aws ssm start-automation-execution --document-name <SSM_DOCUMENT_AUTOMATION_NAME> --parameters "ApplicationStack=pattern3-app,ImageBuilderPipeline=<BUILDER_PIPELINE_ARN>" --region ap-southeast-2
  ```

![ssm-cli-execute-document](/images/5-ssm-cli-execute-document.png?featherlight=false&width=90pc)

2. Kiểm tra trạng thái của **Automation Document**.
  ```
  aws ssm describe-automation-executions --filter "Key=ExecutionId,Values=<EXECUTION_ID>" --region ap-southeast-2
  ```

![ssm-cli-describe-automation](/images/5-ssm-cli-describe-automation.png?featherlight=false&width=90pc)

![ssm-automation-executions](/images/5-ssm-automation-executions.png?featherlight=false&width=90pc)

Từ AWS Console, chúng ta có thể theo dõi từng bước và trạng thái:

![ssm-automation-document-execution-detail](/images/5-ssm-automation-document-execution-detail.png?featherlight=false&width=90pc)

#### Xác Minh AMI ID
Chúng ta sẽ tiến hành xác minh **AMI ID** mới liệu đã được cập nhật hay chưa bằng cách truy cập vào đường dẫn **DNS URL** của **Application Load Balancer**.

![ami-id-verification](/images/5-ami-id-verification.png?featherlight=false&width=90pc)

Tại bước `Step 1: ExecuteImageCreation`, ở phần *Outputs*, chúng ta lấy thông tin của **Pipeline Execution ARN**.
![ssm-automation-execution-detail-step1](/images/5-ssm-automation-execution-detail-step1.png?featherlight=false&width=90pc)

Sau đó, chúng ta có thể đối chiếu với giá trị ở dịch vụ **EC2 Image Builder**.
![ec2-image-builder-pipeline-execution-output](/images/5-ec2-image-builder-pipeline-execution-output.png?featherlight=false&width=90pc)
