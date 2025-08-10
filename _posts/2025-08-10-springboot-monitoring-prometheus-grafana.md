---
title: "Giám sát ứng dụng Spring Boot với Prometheus và Grafana"
date: 2025-08-10
tags: [spring boot, prometheus, grafana, monitoring, devops]
categories: [Spring Boot, Monitoring]
---

🗓️ Aug 10, 2025 &nbsp;&nbsp;&nbsp;&nbsp;📂 Spring Boot, Monitoring, DevOps

---

Trong bài viết này, chúng ta sẽ cùng xây dựng hệ thống **monitoring** (giám sát) cho ứng dụng Spring Boot bằng cách sử dụng:

- 📈 **Prometheus** để thu thập metrics.
- 📊 **Grafana** để hiển thị dữ liệu trực quan.

---

## 🎯 Mục tiêu

- Tích hợp Prometheus vào Spring Boot thông qua thư viện `micrometer`.
- Cấu hình Prometheus để pull metrics từ ứng dụng.
- Thiết lập dashboard trong Grafana để theo dõi thông số như HTTP request, memory, CPU, GC...

---

## 🧱 1. Thêm dependency vào Spring Boot

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

---

## ⚙️ 2. Cấu hình `application.properties`

```properties
management.endpoints.web.exposure.include=health,info,prometheus
management.endpoint.prometheus.enabled=true
management.metrics.export.prometheus.enabled=true
```

✅ Endpoint `/actuator/prometheus` sẽ hiển thị metrics theo format chuẩn Prometheus.

---

## 📡 3. Cấu hình Prometheus

Tạo file `prometheus.yml` như sau:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'spring-boot-app'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['host.docker.internal:8080']
```

💡 Nếu app Spring Boot chạy trên Docker, dùng `host.docker.internal` để Prometheus truy cập máy host thật.

---

## 📊 4. Thiết lập Grafana

1. Truy cập Grafana qua trình duyệt (thường là `http://localhost:3000`)
2. Add **Prometheus** làm Data Source.
3. Import dashboard mẫu:
   - Vào menu “Dashboard” > “Import”
   - Nhập ID: `4701` (Spring Boot metrics)
   - Chọn Prometheus làm nguồn dữ liệu

---

## 🎯 Kết quả

Bạn có thể theo dõi các số liệu như:

- Tổng số HTTP requests
- Tỷ lệ lỗi (error rate)
- JVM memory (heap/non-heap)
- CPU load
- Thời gian phản hồi (latency)
- Thống kê Garbage Collector

---

## 🔚 Kết luận

Giám sát là một phần thiết yếu trong quá trình **DevOps hiện đại**. Việc sử dụng Prometheus + Grafana giúp bạn:

✅ Phát hiện lỗi sớm  
✅ Tối ưu hiệu năng hệ thống  
✅ Giảm thiểu downtime  

---

💡 **Tip nhỏ:**  
Bạn có thể export metrics custom trong code với `@Timed`, `@Counted`, hoặc sử dụng `MeterRegistry`.

---

✏️ *Source code demo sẽ được cập nhật tại mục [My Project](#) sớm!*
