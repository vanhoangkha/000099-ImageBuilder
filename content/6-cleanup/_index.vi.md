---
title : "Dọn dẹp tài nguyên"
date: 2024-01-01
weight : 6
chapter : false
pre : " <b> 6. </b> "
---
**Thu hồi tài nguyên**

Sau khi hoàn tất bài thực hành, chúng ta tiến hành dọn dẹp những tài nguyên đã được tạo bởi CloudFormation.

Lần lượt thực hiện các bước sau:
1. Xoá **Automation Stack** - `pattern3-automate`.
2. Xoá **Pipeline Stack** 
   1. Xoá nội dung bên trong S3 Bucket `Pattern3LoggingBucket`, nếu không, bước kế tiếp sẽ không thể thành công.
   2. Từ CloudFormation Console, xoá Stack: `pattern3-pipeline`.
3. Xoá **Application Stack** - `pattern3-app`.
4. Xoá **Infrastructure Stack** - `pattern3-base`.