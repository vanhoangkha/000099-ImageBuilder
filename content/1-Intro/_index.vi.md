---
title : "Giới thiệu"
date :  "`r Sys.Date()`" 
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

{{% notice note %}}
Bài thực hành sẽ được thiết lập ở **ap-southeast-2 (Sydney)**.
{{% /notice %}}

#### Tác Giả
- **Tim Robinson**, Well-Architected Geo Solutions Architect
- **Stephen Salim**, Well-Architected Geo Solutions Architect

#### Giới Thiệu
Trong quá trình xây dựng chiến lược bảo mật, **OS Patching** là hoạt động không thể thiếu nhằm bảo đảm các EC2 instances chạy các ứng dụng quan trọng luôn sử dụng hệ điều hành với các bản vá bảo mật mới nhất, điều này sẽ giảm thiểu tối đa tiềm năng về các lỗ hổng bảo mật và bề mặt tấn công.

Đa số các chuẩn bảo mật trên thế giới đều tối thiểu yêu cầu bằng chứng của quá trình cập nhật và vá lỗi hệ thống bởi đây là một hoạt động thường nhật và bắt buộc. Mặt khác với doanh nghiệp có số lượng tài nguyên lớn, các nhà quản lý cần phải cẩn trọng để tránh các rủi ro có thể xảy bởi **Operational Overhead** và đảm bảo thời gian gián đoạn luôn luôn là tối thiểu.

Thế nên **Automated Patching Solution** là hết sức cần thiết, ngoài việc giúp giảm thiểu **Operational Overhead** còn tạo điều kiện sẵn sàng cho các hoạt động **Audits** trong tương lai.

Có khá nhiều cách tiếp cận để mà chúng ta có thể tự động hoá tác vụ **OS Patching** thông qua sự kết hợp của các dịch vụ AWS.

Một trong số đó tiêu biểu là sử dụng phương pháp **Blue/Green Deployment** nhằm xây mới *Amazon Machine Image (AMI)* mà chứa các bản vá lỗi mới nhất, khi đó *AMI* này sẽ hoàn toàn sẵn sàng được sử dụng cho các EC2 instances đang chạy ứng dụng. Để dễ hình dung hơn, các quá trình dưới đây sẽ được thực hiện:
- Tự động hoá khởi tạo *AMI* với **EC2 Image Builder**.
- Quản lý và phân tán các tác vụ với **System Manager Automated Document**.
- Triển khai *AMI* mới nhanh chóng với **CloudFormation** và bảo đảm thời gian gián đoạn là tối thiểu với chính sách [AutoScalingReplacingUpdate](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-attribute-updatepolicy.html).

#### Mục Tiêu
1. Cấu hình thành công **EC2 Image Builder**
2. Cấu hình thành công **System Manager Automated Documents**

#### Điều Kiện Cần
1. Một tài khoản AWS được dùng cho mục đích TESTING.

> **Lưu ý**: Khi hoàn thành bài thực hành này, bạn sẽ bị tính phí đối với những tài nguyên không nằm trong hạng mục AWS Free Tier.

#### Quá Trình Triển Khai
1. Base Infrastructure
2. Application Infrastructure
3. AMI Builder Pipeline
4. SSM Build Automation

Chúng ta sẽ tiến hành triển khai phần 1 và 2 với **CloudFormation Templates** để cho môi trường được chuẩn hoá hiệu quả nhất có thể. Sau đó, chúng ta sẽ dễ dàng tập trung hơn với phần 3 và 4 cũng là hai phần quan trọng nhất, ngoài ra chúng ta có thể lựa chọn giữa *pre-built templates* hoặc *manual steps*.

Sau khi hoàn tất bài thực hành này, dựa trên [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/), chúng ta sẽ đạt được những kĩ năng quan trọng giúp tối ưu hoá quá trình bảo mật hạ tầng và ứng dụng hiện tại.