---
title: Xác thực người dùng với Spring Security và JWT
date: 2025-07-07
categories: [Spring Boot, Security]
tags: [springboot, security, jwt, authentication]
---

Trong bài viết này, chúng ta sẽ xây dựng một hệ thống xác thực người dùng bằng **Spring Security** kết hợp với **JSON Web Token (JWT)** – một kỹ thuật phổ biến để bảo vệ REST API.

---

## 💡 Ý tưởng

1. Người dùng đăng nhập bằng email/password → nhận JWT token.
2. Gửi JWT ở mỗi request → xác thực & cho phép truy cập tài nguyên bảo vệ.

---

## 🔐 Cấu hình Security

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http.csrf().disable()
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/auth/**").permitAll()
                .anyRequest().authenticated())
            .sessionManagement(sm -> sm.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .addFilterBefore(new JwtFilter(), UsernamePasswordAuthenticationFilter.class)
            .build();
    }
}
```
🛠️ JWT Utility
public class JwtUtil {
    private final String SECRET = "secret_key";

    public String generateToken(String username) {
        return Jwts.builder()
            .setSubject(username)
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + 86400000)) // 1 ngày
            .signWith(SignatureAlgorithm.HS256, SECRET)
            .compact();
    }

    public String extractUsername(String token) {
        return Jwts.parser().setSigningKey(SECRET)
            .parseClaimsJws(token).getBody().getSubject();
    }
}
✅ Đăng nhập
@RestController
@RequestMapping("/auth")
public class AuthController {
    @PostMapping("/login")
    public ResponseEntity<?> login(@RequestBody LoginRequest request) {
        // Xác thực user (ví dụ hardcoded)
        if(request.getUsername().equals("admin") && request.getPassword().equals("123456")) {
            String token = jwtUtil.generateToken(request.getUsername());
            return ResponseEntity.ok(new JwtResponse(token));
        }
        return ResponseEntity.status(401).body("Unauthorized");
    }
}

---

## 📄 `2025-07-07-springboot-microservices-communication.md`
**Chủ đề:** Giao tiếp giữa các microservice bằng RestTemplate và OpenFeign

```markdown
---
title: Giao tiếp giữa các Microservice trong Spring Boot
date: 2025-07-07
categories: [Spring Boot, Microservices]
tags: [springboot, microservices, resttemplate, feign]
---

Khi xây dựng hệ thống microservices, việc **giao tiếp giữa các service** là bắt buộc. Trong bài này, chúng ta sẽ thực hiện bằng cả `RestTemplate` và `OpenFeign`.

---

## 🧩 Kiến trúc

- `UserService`: Quản lý người dùng
- `OrderService`: Gọi sang `UserService` để lấy thông tin người dùng

---

## 🔁 RestTemplate

```java
@Service
public class UserClient {
    private final RestTemplate restTemplate = new RestTemplate();

    public UserDTO getUserById(Long id) {
        return restTemplate.getForObject("http://localhost:8081/users/" + id, UserDTO.class);
    }
}
```
⚡ OpenFeign
@FeignClient(name = "user-service", url = "http://localhost:8081")
public interface UserClient {
    @GetMapping("/users/{id}")
    UserDTO getUserById(@PathVariable Long id);
}
Kích hoạt Feign:
@SpringBootApplication
@EnableFeignClients
public class OrderServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}

💬 Khi nào dùng gì?

Công cụ	          Ưu điểm	                Nhược điểm
RestTemplate	  Chủ động, dễ hiểu	    Phải viết tay nhiều
OpenFeign	Gọn,    dễ mở rộng	          Cần thêm config khi scale lên

