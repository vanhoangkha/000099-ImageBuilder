---
title : "AMI Builder Pipeline"
date : "`r Sys.Date()`"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

**Triển khai AMI Builder Pipeline**

Ở phần này, chúng ta sẽ tiến hành xây dựng một **AMI Pipeline** thông qua **EC2 Image Builder**.

**EC2 Image Build** là một dịch vụ giúp chúng ta đơn giản hoá các tác vụ từ khởi tạo, bảo trì, đánh giá và phát triển các **Linux/Windows Images**.
- Có thể sử dụng cho dịch vụ *EC2* cũng như môi trường *on-premises*.
- Giảm thiểu thời gian thực hiện các tác vụ cần thiết.

![ami-pipeline-architecture](/images/4-ami-pipeline-architecture.png?featherlight=false&width=60pc)

Sau khi hoàn thành phần này, chúng ta sẽ có thể
- Tự xây dựng được một **AMI Pipeline**.
- Tự tạo ra các **AMI** mới chứa các bản vá lỗi mới nhất.
- Sẵn sàng thay thế các **AMI** cũ và triển khai cho môi trường ứng dụng hiện tại.

**Nội dung**
- [CloudFormation Stack](#cloudformation-stack)
- [AWS CLI](#aws-cli)
- [AMI Pipeline](#ami-pipeline)
- [Tạo S3 Bucket](#tạo-s3-bucket)
- [Tạo IAM Role](#tạo-iam-role)
- [Tạo Security Group](#tạo-security-group)
- [Tạo một Builder Component](#tạo-một-builder-component)
- [Tạo một Builder Recipe](#tạo-một-builder-recipe)
- [Tạo một Builder Pipeline](#tạo-một-builder-pipeline)
- [Thực thi Builder Pipeline](#thực-thi-builder-pipeline)

#### CloudFormation Stack
Để tiến hành triển khai hạ tầng, chúng ta sẽ sử dụng dịch vụ **AWS CloudFormation** thông qua AWS Console hoặc AWS CLI.

| Thành Phần | Giá Trị (Bắt buộc) |
| ---------- | ------- |
| Stack Name | pattern3-pipeline |
| Template URL | [pattern3-pipeline.yml](https://www.wellarchitectedlabs.com/Security/300_Autonomous_Patching_With_EC2_Image_Builder_and_Systems_Manager/Code/templates/section3/pattern3-pipeline.yml) |

Hoặc bạn có thể tải về template bên dưới:

{{%attachments title="Template" pattern=".*(yml)"/%}}

#### AWS CLI
Sau đây là các bước khởi tạo thông qua AWS CLI:
1. Khởi tạo CloudFormation Stack
   ```
   aws cloudformation create-stack --stack-name pattern3-pipeline --template-body file://pattern3-pipeline.yml --parameters  ParameterKey=MasterAMI,ParameterValue=ami-0f96495a064477ffb ParameterKey=BaselineVpcStack,ParameterValue=pattern3-base --capabilities CAPABILITY_IAM --region ap-southeast-2
   ```

![cloudformation-cli-create-stack](/images/4-cloudformation-cli-create-stack.png?featherlight=false&width=90pc)

2. Xác nhận CloudFormation Stack đã khởi tạo hoàn tất với *StackStatus* là `CREATE_COMPLETE`.
   ```
   aws cloudformation describe-stacks --stack-name pattern3-pipeline --region ap-southeast-2
   ```

3. Ghi chú lại các giá trị tại *Output*.

![cloudformation-describe-stack](/images/4-cloudformation-describe-stack.png?featherlight=false&width=90pc)

{{% notice tip %}}
Bài thực hành dựa trên *Golden AMI ID* của **Amazon Linux 2 AMI (HVM)** tại AWS Region `ap-southeast-2`, nếu bạn muốn sử dụng AWS Region khác, giá trị *Golden AMI ID* cần được thay thế một cách tương ứng.
{{% /notice %}}

#### AMI Pipeline
Bây giờ chúng ta sẽ tập trung vào xây dựng phần chính yếu của một **AMI Pipeline**. Chúng ta sẽ lần lượt thực hiện các bước sau đây:
1. Tạo một S3 bucket cho tác vụ logging.
2. Tạo một IAM Role dành cho **EC2 Image Builder**.
3. Tạo một **Image Builder Component**.
4. Tạo một **Image Builder Recipe**.
5. Tạo một **Image Builder Pipeline**.

#### Tạo S3 Bucket
Bởi tính chất của dịch vụ S3, khi đặt tên cho S3 Bucket, chúng ta cần phải tuân theo một số quy tắc được đặc tả tại [URL](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucketnamingrules.html) sau.

1. Tạo ra một chuỗi UUID đặc thù với kí tự in thường.
   ```
   uuidgen | awk -F- '{print tolower($1$2$3)}'
   ```

![s3-uuid](/images/4-s3-uuid.png?featherlight=false&width=90pc)

2. Tạo ra một S3 Bucket với tên gọi là tổ hợp giữa chuỗi `pattern3-logging` và chuỗi UUID trên.
   ```
   aws s3 mb s3://pattern3-logging-<UUID_STRING> --region ap-southeast-2
   ```

![s3-create](/images/4-s3-create.png?featherlight=false&width=90pc)

{{% notice tip %}}
Chúng ta có thể xem tại [URL](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/create-bucket.html) sau về chi tiết việc khởi tạo S3 bucket.
{{% /notice %}}

#### Tạo IAM Role
IAM Role này sẽ được sử dụng cho **EC2 Image Builder** với vai trò là một *instance profile* cho một EC2 instance được khởi tạo tạm thời. EC2 instance này sẽ thực hiện tác vụ **OS Patching**.

Sau đây là quá trình tạo ra một **IAM Role**:
1. Đăng nhập vào AWS Console và truy cập đến dịch vụ IAM theo đường dẫn - https://console.aws.amazon.com/iam/.
2. Ở thanh điều hướng bên tay trái, chọn `Roles` và nhấn nút `Create role`.

![iam-roles](/images/4-iam-roles.png?featherlight=false&width=90pc)

3. Ở màn hình tạo mới, chúng ta chọn `AWS service` và chọn **usecase** là `EC2`.

![iam-create-role](/images/4-iam-create-role.png?featherlight=false&width=90pc)

4. Ở khu vực `Attach permissions policies`, chúng ta sẽ lần lượt chọn các chính sách dưới đây:
   1. **EC2InstanceProfileForImageBuilder**
   2. **AmazonSSMManagedInstanceCore**
5. Chọn lần lượt `Next: Tags` và `Next: Review` để tiến hành xem xét.
6. Điền tên `pattern3-recipe-instance-role` cùng với miêu tả cụ thể.

![iam-create-role-review](/images/4-iam-create-role-review.png?featherlight=false&width=90pc)

7. Tiến hành tạo bằng cách nhấn `Create role`.
8. Sau khi tạo thành công, trong danh sách các IAM roles, chúng ta chọn `pattern3-recipe-instance-role`.
9.  Tiến hành thêm một chính sách mới, nhấn chọn `Add inline policy`.

![iam-role-add-inline-policy](/images/4-iam-role-add-inline-policy.png?featherlight=false&width=90pc)

10. Tại phần **JSON**, chúng ta sẽ sử dụng đặc tả sau. Thay thế `<S3_BUCKET_NAME>` với tên S3 Bucket ở bước trước.
   ```json
   {
      "Version": "2012-10-17",
      "Statement": [
         {
               "Effect": "Allow",
               "Action": "s3:*",
               "Resource": [
                  "arn:aws:s3:::<S3_BUCKET_NAME>/*"               
               ]
         }
      ]
   }
   ```

![iam-policy-editor](/images/4-iam-policy-editor.png?featherlight=false&width=90pc)

11. Nhấn chọn `Review Policy`.
12. Điền tên và tiến hành khởi tạo, nhấn chọn `Create Policy`.

![iam-role-inline-policy-summary](/images/4-iam-role-inline-policy-summary.png?featherlight=false&width=90pc)

#### Tạo Security Group
Chúng ta sẽ cần một EC2 Security Group cho EC2 instance tạm thời. Security Group sẽ không cần bất kỳ một *Inbound Rule* nào, tuy nhiên chúng ta cần đảm bảo *Outbound Rule* là mặc định mà có thể truy câp Internet.

1. Truy cập vào dịch vụ Amazon EC2.
2. Ở thanh điều hướng bên tay trái, chọn **Security Group**.

![ec2-security-groups](/images/4-ec2-security-groups.png?featherlight=false&width=90pc)

3. Nhấn chọn `Create security group`.
4. Ở phần thông tin cơ bản, chúng ta điền lần lượt tên `pattern3-pipeline-instance-security-group` và miêu tả.
5. Ở phần VPC, chúng ta sẽ chọn VPC tương ứng.
6. Đối với *Inbound Rule* và *Outbound Rule*, chúng ta sẽ giữ mặc định và không thêm bất kỳ giá trị nào.
7. Nhấn `Create` để khởi tạo.

![ec2-create-security-group](/images/4-ec2-create-security-group.png?featherlight=false&width=90pc)

#### Tạo một Builder Component
Chúng ta sẽ tiến hành tạo một **Component** của dịch vụ **EC2 Image Builder**.
1. Truy cập vào dịch vụ [EC2 Image Builder](https://ap-southeast-2.console.aws.amazon.com/imagebuilder/home?region=ap-southeast-2).
2. Tại thanh điều hướng bên tay trái, chọn **Components**.
3. Nhấn nút `Create Component`.
4. Tiến hành chọn các giá trị như sau:
   1. Version: `1.0.0`.
   2. Platform: `Linux`.
   3. Compatible OS versions: `Amazon Linux 2`.
   4. Component Name: `pattern3-pipeline-ConfigureOSComponent`.
   5. Description: `Component to update the OS with latest package versions`.

![ec2-image-builder-create-component-details](/images/4-ec2-image-builder-create-component-details.png?featherlight=false&width=90pc)

5. Nhấn nút `Define document content`.
6. Sao chép và điền đặc tả sau:
   ```yml
   name: ConfigureOS
    schemaVersion: 1.0
    phases:
    - name: build
        steps:
        - name: UpdateOS
            action: UpdateOS
   ```

![ec2-image-builder-create-component-definition-document](/images/4-ec2-image-builder-create-component-definition-document.png?featherlight=false&width=90pc)

7. Nhấn nút `Create Component` để tiến hành khởi tạo.

![ec2-image-builder-components](/images/4-ec2-image-builder-components.png?featherlight=false&width=90pc)

{{% notice note %}}
Ở phần thực hành này, chúng ta chỉ đơn giản đặc tả với hành động `UpdateOS` nhằm cập nhật các phần mềm hiện tại. Chúng ta có thể tham khảo thêm các hành động khác tại [URL](https://docs.aws.amazon.com/imagebuilder/latest/userguide/image-builder-application-documents.html#document-example) sau.
{{% /notice %}}

#### Tạo một Builder Recipe
Trước khi đặc tả **Builder Pipeline**, chúng ta cần phải có một **Builder Recipe**.

1. Truy cập vào dịch vụ [EC2 Image Builder](https://ap-southeast-2.console.aws.amazon.com/imagebuilder/home?region=ap-southeast-2).
2. Tại thanh điều hướng bên tay trái, chọn **Image recipes**.
3. Nhấn nút `Create Recipe`.
4. Ở phần *Recipe details*, điền các giá trị như sau:
   1. Name: `pattern3-pipeline-ImageRecipe`.
   2. Version: `1.0.0`.
   3. Description: `Pattern3 Configure OS Recipepackage versions`.
5. Ở phần *Source Image*, chọn `Enter custom AMI ID` và diền giá trị `ami-0f96495a064477ffb`.

![ec2-image-builder-create-image-recipe-source-image](/images/4-ec2-image-builder-create-image-recipe-source-image.png?featherlight=false&width=90pc)

6. Ở phần *Build Components*, chọn **Component** ở bước trước, ngoài ra chúng ta có thể tìm kiếm với bộ lọc `Owned by me`.

![ec2-image-builder-create-image-recipe-components](/images/4-ec2-image-builder-create-image-recipe-components.png?featherlight=false&width=90pc)

7. Nhấn nút `Create Recipe` để tiến hành khởi tạo.

![ec2-image-builder-image-recipes](/images/4-ec2-image-builder-image-recipes.png?featherlight=false&width=90pc)

#### Tạo một Builder Pipeline
1. Truy cập vào dịch vụ [EC2 Image Builder](https://ap-southeast-2.console.aws.amazon.com/imagebuilder/home?region=ap-southeast-2).
2. Tại thanh điều hướng bên tay trái, chọn **Image recipes**.
3. Nhấn chọn `pattern3-pipeline-ImageRecipe` và nhấn nút `Create pipeline from this recipe`.

![ec2-image-builder-image-recipes-actions](/iamges/../images/4-ec2-image-builder-image-recipes-actions.png?featherlight=false&width=90pc)

4. Ở phần *Specify pipeline details*, điền các thông tin sau:
   1. Name: `pattern3-pipeline-ImagePipeline`.
   2. Description: `Pattern 3 pipeline to update OS`.
   3. Role: Chọn IAM Role mà chúng ta đã khởi tạo ở bước trước.

![ec2-image-builder-create-pipeline-details](/iamges/../images/4-ec2-image-builder-create-pipeline-details.png?featherlight=false&width=90pc)

5. Ở phần *Choose recipe*, nhấn nút `Next`.
6. Ở phần *Infrastructure Settings*, chọn `Create a new infrastructure configuration`.

![ec2-image-builder-create-pipeline-define-infra-config](/images/4-ec2-image-builder-create-pipeline-define-infra-config.png?featherlight=false&width=90pc)

7. Điền các thông tin sau:
   1. **IAM Role**: Dựa vào **EC2 Instance Profile** đã tạo trước đó.
   2. **VPC/Subnet**: Dựa vào *Output* của `pattern3-base`, chọn VPC và Private Subnet tương ứng.
   3. **Security Group**: Dựa vào **Security Group ID** đã tạo trước đó.
   4. **Instance Type**: `m5.large` hoặc nhỏ hơn có thể.
   5. **S3 Bucket**: chọn S3 bucket đã tạo trước đó.

![ec2-image-builder-create-pipeline-define-infra-config-iam-type](/images/4-ec2-image-builder-create-pipeline-define-infra-config-iam-type.png?featherlight=false&width=90pc)

![ec2-image-builder-create-pipeline-define-infra-config-vpc-s3](/images/4-ec2-image-builder-create-pipeline-define-infra-config-vpc-s3.png?featherlight=false&width=90pc)

8. Ở phần *Define distribution settings*, Nhấn nút `Next`.
9.  Nhấn nút `Review`.
10. Nhấn nút `Create Pipeline` để khởi tạo.

{{% notice note %}}
Với `m5.large` có thể sẽ mất khoảng 20 đến 30 phút để khởi tạo, chúng ta có thể dùng loại nhỏ hơn để tiết kiệm chi phí nhưng thời gian khởi tạo sẽ lâu hơn.
{{% /notice %}}

![ec2-image-builder-image-pipelines](/images/4-ec2-image-builder-image-pipelines.png?featherlight=false&width=90pc)

#### Thực thi Builder Pipeline
Sau khi quá trình đặc tả đã hoàn tất, chúng ta tiến hành thử nghiệm **Builder Pipeline**.

![ec2-image-builder-image-pipelines-actions](/images/4-ec2-image-builder-image-pipelines-actions.png?featherlight=false&width=90pc)

1. Nhấn nút `Actions` và chọn `Create pipeline from this recipe`.
2. Một khi thực thi hoàn tất, chúng ta sẽ kiểm tra liệu **AMI** mới có được khởi tạo hay không.

![ec2-image-builder-pipeline-result](/images/4-ec2-image-builder-pipeline-result.png?featherlight=false&width=90pc)

{{% notice note %}}
**EC2 Image Builder** sử dụng **Automation Document** của dịch vụ System Manager để hoàn thành quá trình khởi tạo, chúng ta có thể truy cập vào dịch System Manager để theo dõi quá trình này cũng như kiểm tra trạng thái - `ImageBuilderBuildImageDocument`.
{{% /notice %}}

{{% notice tip %}}
Tham khảo thêm về quá trình kiểm tra **Automation Document** tại [Automation Working URL](https://docs.aws.amazon.com/systems-manager/latest/userguide/automation-working.html) sau.
{{% /notice %}}

Bây giờ, chúng ta sẽ sang phần kế tiếp với việc triển khai quá trình **Build Automation** với dịch vụ System Manager.
