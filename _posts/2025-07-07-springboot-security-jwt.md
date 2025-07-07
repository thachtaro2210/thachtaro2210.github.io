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
```java
@Component
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
        return Jwts.parser()
            .setSigningKey(SECRET)
            .parseClaimsJws(token)
            .getBody()
            .getSubject();
    }
}
```
✅ Đăng nhập
```java
@RestController
@RequestMapping("/auth")
public class AuthController {

    @Autowired
    private JwtUtil jwtUtil;

    @PostMapping("/login")
    public ResponseEntity<?> login(@RequestBody LoginRequest request) {
        // Xác thực user (ví dụ hardcoded)
        if ("admin".equals(request.getUsername()) && "123456".equals(request.getPassword())) {
            String token = jwtUtil.generateToken(request.getUsername());
            return ResponseEntity.ok(new JwtResponse(token));
        }
        return ResponseEntity.status(401).body("Unauthorized");
    }
}
```

