
# System Architecture

## Overview
Hệ thống microservices hỗ trợ đặt vé xem phim cho người dùng cuối, bao gồm quản lý phim, suất chiếu, thông tin khách hàng và các bước đặt vé. Mục tiêu là phân tách chức năng theo dịch vụ riêng biệt để dễ mở rộng và bảo trì.

## System Components

- **Movie Service**:  
  Quản lý thông tin về phim, suất chiếu và tình trạng ghế ngồi. Đây là dịch vụ trung tâm để cung cấp dữ liệu phim cho người dùng lựa chọn.

- **Customer Service**:  
  Xử lý thông tin người dùng như đăng ký, đăng nhập, xác minh thông tin và lưu thông tin đặt vé của khách hàng.

- **API Gateway**:  
  Là điểm truy cập duy nhất, định tuyến các request tới các microservice tương ứng. Hỗ trợ xác thực, phân quyền và thống kê request.

## Communication

- Các dịch vụ giao tiếp qua RESTful APIs.  
- API Gateway định tuyến các yêu cầu HTTP đến các service (ví dụ `/api/movies`, `/api/customers`).  
- Sử dụng nội bộ Docker Compose hoặc Kubernetes để định danh service qua hostname (`movie-service`, `customer-service`...).

## Data Flow

1. Người dùng gửi yêu cầu qua API Gateway.
2. Gateway gọi `Customer Service` để xác thực và lấy thông tin khách hàng.
3. Sau đó gọi `Movie Service` để hiển thị danh sách phim, suất chiếu và tình trạng ghế.
4. Kết quả được trả về cho người dùng.
5. Dữ liệu được lưu trữ riêng biệt trong database của từng service (ví dụ: PostgreSQL hoặc MySQL).

## Diagram

> (Đặt sơ đồ kiến trúc trong `docs/asset/architecture-diagram.png`)

## Scalability & Fault Tolerance

- Các service được container hóa, dễ dàng scale độc lập bằng Docker/K8s.
- Gateway có thể dùng load balancer để phân phối request.
- Service tách biệt nên nếu một service lỗi, phần còn lại vẫn có thể hoạt động.
- Dữ liệu được cô lập theo nguyên tắc Database per Service.

---

# 📊 Microservices System - Analysis and Design

## 1. 🎯 Problem Statement

Hệ thống giúp người dùng đặt vé xem phim online dễ dàng, tra cứu suất chiếu, chọn ghế và thanh toán. Người dùng có thể tạo tài khoản và lưu lại lịch sử đặt vé.

- **Người dùng**: khách đặt vé, quản trị viên hệ thống.
- **Mục tiêu**: đặt vé nhanh chóng, kiểm tra ghế trống, lưu thông tin lịch sử.
- **Dữ liệu xử lý**: thông tin khách hàng, phim, suất chiếu, ghế, trạng thái vé.

## 2. 🧩 Identified Microservices

| Service Name     | Responsibility                                  | Tech Stack     |
|------------------|--------------------------------------------------|----------------|
| movie-service    | Quản lý phim, suất chiếu, tình trạng ghế        | Spring Boot + SQL Server |
| customer-service | Quản lý người dùng và xác minh thông tin đặt vé | Spring Boot + SQL Server |
| api-gateway      | Định tuyến request tới service                  | Spring Cloud Gateway     |

## 3. 🔄 Service Communication

- API Gateway ⇄ Movie Service (REST)
- API Gateway ⇄ Customer Service (REST)
- Internal: Không có kết nối trực tiếp giữa 2 service (giao tiếp qua Gateway)

## 4. 🗂️ Data Design

- **movie-service**:  
  - `Movie`: id, title, genre, duration  
  - `Showtime`: id, movie_id, start_time, room  
  - `Seat`: id, showtime_id, seat_number, status  

- **customer-service**:  
  - `Customer`: id, name, email, phone  
  - `Booking`: id, customer_id, showtime_id, seat_id, status  

## 5. 🔐 Security Considerations

- Sử dụng JWT để xác thực người dùng
- Kiểm tra quyền truy cập trong từng API
- Mã hóa thông tin nhạy cảm (password)

## 6. 📦 Deployment Plan

- Mỗi service có Dockerfile riêng
- Sử dụng `docker-compose.yml` để khởi động toàn bộ hệ thống
- File `.env` quản lý biến môi trường như DB_URL, JWT_SECRET...

## 7. 🎨 Architecture Diagram

```
+--------------+
| API Gateway  |
+------+-------+
       |
  +----v----+     +-------------+
  | Customer|     |    Movie    |
  | Service |     |   Service   |
  +----+----+     +------+------+
       |                 |
  +----v----+     +------v------+
  |Customer |     |Movie Database|
  |Database |     +-------------+
  +---------+
```

## ✅ Summary

Kiến trúc này cho phép phát triển và triển khai từng thành phần độc lập. Việc chia nhỏ logic theo dịch vụ chuyên biệt giúp dễ mở rộng, bảo trì và triển khai CI/CD. Cơ chế bảo mật rõ ràng với Gateway và JWT cũng đảm bảo an toàn hệ thống.
