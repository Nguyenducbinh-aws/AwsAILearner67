---
title: "Worklog Tuần 3"
date: 2026-09-21
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---


### Mục tiêu tuần 3:

* Hiểu cách mở rộng ứng dụng tự động với EC2 Auto Scaling.
* Giám sát tài nguyên bằng Amazon CloudWatch (metric, alarm, log, dashboard).
* Nắm kiến thức nền về container với Docker.
* Triển khai container trên Amazon ECS, so sánh launch type EC2 và AWS Fargate.
* Đọc tài liệu chính thức về việc dùng CloudWatch alarm để phát hiện deployment ECS thất bại và rollback.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu EC2 Auto Scaling <br>&emsp; + Launch template, Auto Scaling group <br>&emsp; + Desired/min/max capacity, scaling policy <br>&emsp; + Kết hợp với Load Balancer <br> - **Thực hành:** tạo launch template và Auto Scaling group sau Load Balancer theo workshop, quan sát hệ thống tự thêm/bớt instance | 21/09/2026 | 21/09/2026 | <https://000006.awsfcaj.com> |
| 3 | - Tìm hiểu Amazon CloudWatch <br>&emsp; + Metrics, Alarms và trạng thái OK/ALARM/INSUFFICIENT_DATA <br>&emsp; + CloudWatch Logs <br>&emsp; + Dashboard <br> - **Thực hành:** xem metric của EC2, tạo alarm theo ngưỡng CPU, tạo dashboard đơn giản | 22/09/2026 | 22/09/2026 | <https://000008.awsfcaj.com> |
| 4 | - Tìm hiểu Docker <br>&emsp; + Image, container, Dockerfile <br>&emsp; + Build, run, đẩy image lên registry <br> - Đọc phần giới thiệu container trong lộ trình (Module 3, Module 5) <br> - **Thực hành:** viết Dockerfile, build image và chạy container cho ứng dụng mẫu theo workshop | 23/09/2026 | 23/09/2026 | <https://000015.awsfcaj.com> <br> <https://cloudjourney.awsfcaj.com/3-optimize/> <br> <https://cloudjourney.awsfcaj.com/5-container/> |
| 5 | - Tìm hiểu Amazon ECS <br>&emsp; + Cluster, task definition, task, service <br>&emsp; + Launch type EC2 <br>&emsp; + Kết hợp ECS service với Load Balancer <br> - **Thực hành:** tạo ECS cluster, task definition và service để chạy image đã build | 24/09/2026 | 24/09/2026 | <https://000016.awsfcaj.com> |
| 6 | - Tìm hiểu ECS với AWS Fargate <br>&emsp; + Serverless compute cho container, so sánh với launch type EC2 <br> - Đọc tài liệu AWS: phát hiện deployment ECS thất bại bằng CloudWatch alarm <br>&emsp; + Gắn alarm có sẵn vào deploymentConfiguration (alarms={alarmNames=[...],enable=true,rollback=true}) <br>&emsp; + ECS gọi DescribeAlarms trong lúc deploy; alarm vào ALARM trước khi đạt steady state + bake time thì deployment = FAILED <br>&emsp; + Bật rollback: ECS tạo deployment mới khôi phục deployment hoàn tất gần nhất <br>&emsp; + Chỉ áp dụng cho rolling update (deployment controller type ECS); theo dõi rolloutState: IN_PROGRESS → COMPLETED / FAILED <br>&emsp; + Dùng chung với circuit breaker: cơ chế nào kích hoạt trước sẽ làm fail <br>&emsp; + Lưu ý: throttle DescribeAlarms, alarm đã ở ALARM từ trước bị bỏ qua, bake time mặc định dưới 5 phút <br>&emsp; + Metric gợi ý: ALB HTTPCode_ELB_5XX_Count/4XX_Count, CPUUtilization, MemoryUtilization, SQS ApproximateNumberOfMessagesNotVisible <br> - **Thực hành:** triển khai service trên Fargate theo workshop; ôn tập tuần 3 và **dọn tài nguyên** (ECS service/cluster, Load Balancer, Auto Scaling group, alarm) | 25/09/2026 | 25/09/2026 | <https://000067.awsfcaj.com> <br> <https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-alarm-failure.html> |


### Kết quả đạt được tuần 3:

* Hiểu cơ chế Auto Scaling group, launch template, scaling policy và cách kết hợp với Load Balancer.
* Biết tạo CloudWatch alarm, đọc metric và dựng dashboard cơ bản.
* Viết được Dockerfile, build và chạy container cho ứng dụng mẫu.
* Hiểu các khái niệm ECS (cluster, task definition, task, service) và khác biệt giữa launch type EC2 và Fargate.
* Hiểu cách ECS dùng CloudWatch alarm để đánh dấu deployment FAILED và tự rollback, các giới hạn cần lưu ý và quan hệ với deployment circuit breaker.
