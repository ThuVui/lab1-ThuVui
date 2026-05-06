# Service Boundary của nhóm

## 1. Thông tin nhóm

- Tên nhóm: 3b
- Lớp: CNTT 17-10
- Thành viên: [Nguyễn Thị Thu Vui], [Đinh Mạnh Đà], [Hà Thị Phương Thanh], [Đỗ Công Ngọc Sơn]
- Service nhóm phụ trách: Kiểm tra môi trường và thu thập minh chứng buổi 1
- Sản phẩm tổng thể của lớp: Hệ thống thiết lập và kiểm tra môi trường lab cho học phần

## 2. Actor

Ai tương tác với hệ thống/service?

- Sinh viên/Developer: Sử dụng các service trong mini-stack để phát triển ứng dụng
- Tools: Docker Client, curl, Postman, Docker Daemon
- CI/CD Pipeline: Pull và deploy Docker images từ registry

## 3. System Boundary

Nhóm em xây phần nào?

Phần nhóm kiểm soát:

- Docker Compose configuration (docker-compose.smoke.yml)
- Container orchestration cho 3 services: Nginx, Redis, Docker Registry
- Scripts để tự động hóa việc pull images và chạy smoke tests
- Monitoring health checks cho từng service

Phần nhóm chỉ tích hợp:

- Docker Engine (cài đặt trên máy sinh viên)
- Docker images từ Docker Hub (nginx:alpine, redis:7-alpine, registry:2)
- Hệ điều hành của máy sinh viên

## 4. Service Boundary

Service của nhóm có trách nhiệm gì?

- Cung cấp web server (Nginx) trên cổng 8081 để test HTTP
- Cung cấp cache store (Redis) cho các ứng dụng khác
- Cung cấp Docker image registry trên cổng 5000 để lưu trữ images cục bộ
- Health check liên tục để đảm bảo các services chạy bình thường
- Tự động khởi động lại container khi lỗi

Service KHÔNG làm gì?

- Không xử lý dữ liệu application logic phức tạp
- Không lưu trữ persistent data (chỉ là test environment)
- Không quản lý authentication/authorization
- Không xử lý SSL/TLS encryption
- Không cung cấp monitoring, logging, tracing (ngoài health checks cơ bản)

## 5. Input / Output

### Input

- Docker Compose file (.yml) để cấu hình các services
- Docker images từ Docker Hub (nginx:alpine, redis:7-alpine, registry:2)
- HTTP requests từ clients (curl, Postman, browsers)
- Redis commands từ ứng dụng clients
- Docker push/pull commands từ Docker CLI
- System resources (CPU, memory, disk) từ host machine

### Output

- HTTP responses từ Nginx server (port 8081)
- Redis responses (PONG, stored values)
- Docker Registry responses (image manifests, blobs)
- Health check status (healthy/unhealthy)
- Container logs
- Evidence files (tool-versions.txt, hello-world.txt, smoke-test-result.txt)

## 6. API dự kiến

| Method | Endpoint/Command | Service | Mục đích |
|---|---|---|---|
| GET | http://localhost:8081/ | Nginx | Health check / Kiểm tra Nginx server |
| COMMAND | PING | Redis | Health check Redis server |
| GET | http://localhost:5000/v2/ | Registry | Registry API health check |
| POST | /v2/<name>/blobs/uploads/ | Registry | Upload Docker image layer |
| GET | /v2/<name>/manifests/<reference> | Registry | Get Docker image manifest |
| Command | docker-compose up | All | Start mini-stack |
| Command | docker-compose down | All | Stop mini-stack |
| Command | docker run --rm hello-world | Docker | Test Docker basic functionality |
| Command | docker pull <image> | Registry | Pull images từ local registry hoặc Docker Hub |
| Command | docker push <image> | Registry | Push images tới local registry |

## 7. Phụ thuộc service khác

Service này gọi đến service nào?

- Gọi đến Docker Hub (pull images: nginx:alpine, redis:7-alpine, registry:2)
- Gọi đến hệ điều hành host (cấp quyền file, port binding, network)
- Gọi đến Git repository (lưu trữ code, scripts, evidence)

Service nào gọi đến service này?

- Ứng dụng sinh viên xây dựng (kết nối tới Redis, Nginx)
- Docker CLI và Docker Daemon (quản lý containers)
- Scripts smoke_test.sh/ps1 (kiểm tra health của các services)
- CI/CD pipelines (push/pull images từ Registry)
- Các tools kiểm tra: wget, redis-cli (dùng trong health checks)

## 8. Sơ đồ minh họa

Có thể vẽ bằng Mermaid, draw.io, Ludichart hoặc ảnh chụp sơ đồ.
https://lucid.app/lucidchart/0e115824-b9b8-4682-86b7-5417bc8f2924/edit?viewport_loc=-1679%2C1109%2C5074%2C2780%2C0_0&invitationId=inv_d58f6279-6bf7-4910-8c35-8d112862dddb
```mermaid
flowchart LR
    User[Actor] --> Service[Service của nhóm]
    Service --> DB[(Database)]
    Service --> Other[Service khác]
