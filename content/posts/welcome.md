+++
date = 2024-02-24T04:14:54-08:00
draft = false
title = 'Hệ thống lưu trữ tệp linh hoạt - Amazon Elastic File System (AWS)'
layout= 'welcome'
weight = 10
author = 'Ngọc Bảo - Bảo Bảo Lucif'
favicon = "hugo-aws/public/img/ava.png"
logo = "hugo-aws/public/img/ava.png"
navallcaps = true
sharing_image = "hugo-aws/public/img/ava.png"
+++

![Amazon Elastic File System](hugo-aws/public/img/Picture1.jpg)

1. [Amazon Elastic File System là gì?](#efs-la-gi)
2. [Đặc điểm của EFS so với các hệ thống lưu trữ khác](#dac-diem-efs)
3. [Chi phí và giá](#chi-phi-va-gia)
4. [Kết luận](#ket-luan)
5. [Setup triển khai EFS (cơ bản)](#setup-trien-khai-efs)

## Amazon Elastic File System là gì?{#efs-la-gi}
Đầu tiên, để bắt đầu chúng ta sẽ tìm hiểu qua về **EC2 (Amazon Elastic Compute Cloud)**. Đây là một cơ sở hạ tầng điện toán đám mây được cung cấp bởi **Amazon Web Services (AWS)** giúp cung cấp các tài nguyên liên quan tới ảo hóa máy tính theo yêu cầu.

Để đơn giản hóa, bạn có thể hiểu EC2 cung cấp cho bạn bộ công cụ giúp xây dựng một "máy tính ảo" với đầy đủ các thông số như bộ nhớ (ram), vi xử lý, hệ điều hành, bộ nhớ, thiết lập mạng v..v..
Các máy chủ ảo có thể kết hợp với nhau để dễ dàng triển khai một cách nhanh chóng và có tính sẵn sàng. Người dùng cũng sẽ dự tính được chi phí trước cả khi thực sự dùng dịch vụ. Rất tiện lợi.

![EFS](hugo-aws/public/img/Picture2.png)

## Đặc điểm của EFS so với các hệ thống lưu trữ khác{#dac-diem-efs}
**Giữa muôn vàn lựa chọn, tại sao lại là EFS?**

Để trả lời câu hỏi này chúng ta sẽ so sánh EFS cùng hai “anh em” chung nhà đó là EBS và S3. 
Thông thường, có thể sử dụng lưu trữ thông qua Amazon S3 hoặc [**Amazon Elastic Block Store**](https://aws.amazon.com/vi/ebs) (EBS) tuy nhiên:
+ [**Amazon S3**](https://aws.amazon.com/vi/s3/) hướng tới việc lưu trữ đối tượng (ảnh, vid, tài liệu v...v...), dung lượng 5Tb cho một đối tượng.
+ [**Amazon EBS**](https://aws.amazon.com/vi/ebs) lưu trữ khối nhưng chỉ sử dụng được cho một EC2 instance, ngoài ra bị giới hạn kích thước file = kích thước volume tạo.


Vậy, với nhu cầu tạo ra nhiều [**EC2 instance**](https://aws.amazon.com/vi/ec2), tối ưu tốc độ tải và truy cập cũng như quản lý, bảo mật những file có dung lượng lớn giữa nhiều [**EC2**](https://aws.amazon.com/vi/ec2) instant chúng ta nên sử dụng EFS.

[**EFS**](https://aws.amazon.com/vi/efs): Hệ thống quản lý tệp toàn diện có khả năng mở rộng cao. Nó có thể được thiết lập sử dụng như một nguồn dữ liệu chung cho bất kỳ ứng dụng, tác vụ chạy trên nhiều EC2 instantces

|  | **EFS** | **EBS** | **S3** |
|-------|-------|-------|-------|
| Loại lưu trữ | System | Block | Object |
| Dung lượng file | 47.9Tb / file | Theo volume | 5Tb / file |
| Dung lượng tối đa | Không giới hạn | 16Tb hoặc 64Tb | Không giới hạn |
| Độ trễ | Thấp, có thể tối ưu | Thấp nhất trong cả 3 | Tùy theo yêu cầu |
| Băng thông | 10+ GBs | 2GBs | 25+ GBs |  

*Bảng so sánh một số tính năng giữa 3 dịch vụ lưu trữ nhà Amazon*

[**Amazon S3**](https://aws.amazon.com/vi/s3/) là dịch vụ lưu trữ đối tượng, như là ảnh, video, file, và các trang web cơ bản.
[**Amazon EBS**](https://aws.amazon.com/vi/ebs) là dịch vụ lưu trữ khối cho các [**EC2 instance**](https://aws.amazon.com/vi/ec2) , đơn giản như là ổ cứng của máy tính.
[**Amazon EFS**](https://aws.amazon.com/vi/efs) là hệ thống lưu trữ file cho nhiều các [**EC2 instance**](https://aws.amazon.com/vi/ec2).

**Khả năng Bảo mật và Sao lưu chủ động**

EFS cung cấp khả năng kiểm soát quyền truy cập vào hệ thống bằng các quy tắc nhóm VPC và chính sách IAM. Nhóm bảo mật VPC giúp kiểm soát lưu lượng mạng vào ra hệ thống. Và bằng cách thêm chính sách IAM và thiết lập, bạn có thể kiểm soát máy khách nào có thể đính kèm tệp theo quyền.
Ngoài ra, EFS còn cung cấp cơ chế bảo mật thông qua điểm truy cập và dịch vụ quản lý khóa (KMS) bạn có thể tham khảo thêm tại đây.

Sao lưu cũng là một ưu điểm của EFS, với một bảng điều khiển tập trung cùng khả năng lập lịch sao lưu tự động, nhờ đó bạn có thể quản lý toàn phần việc sao lưu cùng dịch vụ Amazone EFS.
Ngoài ra, EFS cũng có tính năng quản lý vòng đời, một tính năng tương tự như S3. Nôm na, EFS sẽ cho phép dịch chuyển tự động những khối dữ liệu không thường xuyên truy cập sang một lớp lữu trữ riêng giúp tích kiệm chi phí.

**Hiệu suất**

Không giống như EBS, bạn không thể tùy chọn cấu hình từng phần (như loại ổ đĩa, chỉ định IOPS, phiên bản EC2, v..v..) tuy nhiên hiệu suất cơ bản của EFS đủ nhanh cho hầu hết các loại hình công việc. Nguyên nhân là vì ES là một hệ thống lưu trữ tệp phân tán nên sẽ tập trung xử lý thông lượng mỗi giây cao hơn nhiều so với EBS.

**Vậy còn Khả năng tương thích của EFS?**

EFS tương tự như EBS, có thể sao chép và truy cập nhiều máy chủ trong cùng một tài khoản AZ. Tuy nhiên, EFS không bắt phải ước lượng trước kích thước cụ thể nên có thể tăng - giảm quy mô nhanh chóng và tự động để đáp ứng nhu cầu khối lượng công việc. Đây cũng có thể coi là một ưu điểm của EFS hơn so với EBS

Tuy nhiên phải lưu ý. Ổ đĩa EBS có thể đính kèm vào cả máy Windows và các EC2 không dùng Win. Nhưng ổ đĩa EFS thì chỉ có thể sử dụng cho máy chủ dựa trên Linux.


## Chi phí và giá{#chi-phi-va-gia}

Chúng ta có thể tính trước mức giá phải trả cho EFS thông qua [**Công cụ tính giá Amazone EFS**](https://calculator.aws/#/addService/EFS). Giá được tính trên dung lượng lưu trữ, thông lượng và mức sử dụng trên một tháng (chưa bao gồm thuế). Dùng nhiều trả nhiều. Với cùng một lượng sử dụng, mức giá của EFS sẽ nhỉnh hơn so với việc sử dụng EBS. 

![Bảng giá EFS](hugo-aws/public/img/Picture3.png)

*Khá chi tiết, với mức lưu trữ 30GB tôi mất khoản 2.91$ mỗi tháng*

## Kết luận{#ket-luan}

Khi nào nên sử dụng EFS:

+ Nếu bạn cần truy cập dữ liệu từ các máy khác nhau hoặc từ các availability zone khác nhau, EFS là lựa chọn tốt nhất cho bạn.
+ EFS phù hợp nhất cho máy chủ tệp doanh nghiệp, hệ thống sao lưu, cụm Big Data lớn, hệ thống Massively Parallel Processing(MPP), Content Distribution Networks (CDN) và các trường hợp sử dụng lớn khác.
+ EFS cũng dành cho các hệ thống yêu cầu nhiều thông lượng.


## Setup triển khai EFS{#setup-trien-khai-efs}
Truy cập tài khoản -> Services ->

![Step1](hugo-aws/public/img/Picture4.png)

File system -> Create new

![Step2](hugo-aws/public/img/Picture5.png)

Attach vào VPC hiện tại và attach vào một instance Linux

![Step3](hugo-aws/public/img/Picture6.png)





