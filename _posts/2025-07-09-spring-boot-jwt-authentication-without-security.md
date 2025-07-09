---
title: "Build JWT Authentication from Scratch in Spring Boot (No Spring Security)"
date: 2025-07-09
categories: [Spring Boot, JWT, Security]
tags: [spring boot, jwt, authentication, filter, security, token]
---

Most tutorials use **Spring Security** for handling JWT authentication. While it's powerful, it's also complex and full of "magic".

What if you want to **learn the internals**? In this post, we’ll build a **JWT-based authentication system from scratch**, with:

- 🔐 Login endpoint that issues JWTs
- 🧰 Manual token validation using filters
- 🧑‍💼 Role-based access to endpoints
- ❌ No Spring Security at all

---

## ⚙️ Technologies

- Java 17
- Spring Boot 3.2+
- `io.jsonwebtoken` for JWT parsing

---

## 📦 1. Add Dependencies

```xml
<!-- pom.xml -->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
```
## 🧠 2. JWT Utility Class
```java
public class JwtUtil {
    private static final String SECRET_KEY = "my-very-secure-secret-key";

    public static String generateToken(String username, String role) {
        return Jwts.builder()
            .setSubject(username)
            .claim("role", role)
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60)) // 1 hour
            .signWith(Keys.hmacShaKeyFor(SECRET_KEY.getBytes()))
            .compact();
    }

    public static Claims validateToken(String token) throws JwtException {
        return Jwts.parserBuilder()
            .setSigningKey(SECRET_KEY.getBytes())
            .build()
            .parseClaimsJws(token)
            .getBody();
    }
}
```
## 🧪 3. AuthController: Login to get token
```java
@RestController
@RequestMapping("/auth")
public class AuthController {

    @PostMapping("/login")
    public ResponseEntity<?> login(@RequestBody AuthRequest request) {
        // 🔐 Hardcoded demo user
        if ("user".equals(request.getUsername()) && "pass".equals(request.getPassword())) {
            String token = JwtUtil.generateToken(request.getUsername(), "USER");
            return ResponseEntity.ok(Map.of("token", token));
        }
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body("Invalid credentials");
    }

    record AuthRequest(String username, String password) {}
}
```
## 🛡️ 4. JWT Filter: Intercept & validate
```java
@Component
public class JwtAuthFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {

        String header = request.getHeader("Authorization");
        if (header != null && header.startsWith("Bearer ")) {
            String token = header.substring(7);
            try {
                Claims claims = JwtUtil.validateToken(token);
                String username = claims.getSubject();
                String role = claims.get("role", String.class);

                request.setAttribute("username", username);
                request.setAttribute("role", role);
            } catch (JwtException e) {
                response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
                response.getWriter().write("Invalid token");
                return;
            }
        }

        filterChain.doFilter(request, response);
    }
}
```

## 🧷 5. Filter Configuration
```java
@Configuration
public class FilterConfig {

    @Bean
    public FilterRegistrationBean<JwtAuthFilter> jwtFilter() {
        FilterRegistrationBean<JwtAuthFilter> registrationBean = new FilterRegistrationBean<>();
        registrationBean.setFilter(new JwtAuthFilter());
        registrationBean.addUrlPatterns("/api/*"); // Secure only /api/*
        return registrationBean;
    }
}
```
## 🔐 6. Secured Controller with Role Check
```java
@RestController
@RequestMapping("/api")
public class SecureController {

    @GetMapping("/hello")
    public ResponseEntity<String> hello(HttpServletRequest request) {
        String username = (String) request.getAttribute("username");
        String role = (String) request.getAttribute("role");

        if (username == null) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body("Missing token");
        }

        if (!"USER".equals(role)) {
            return ResponseEntity.status(HttpStatus.FORBIDDEN).body("Insufficient permissions");
        }

        return ResponseEntity.ok("Hello, " + username + " 👋");
    }
}
```
## 🧪 7. Test with Postman or Curl

# 🔑 Step 1: Login to get token
```
curl -X POST http://localhost:8080/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"user","password":"pass"}'
```
# 🔐 Step 2: Call secure endpoint
```
curl http://localhost:8080/api/hello \
  -H "Authorization: Bearer <your_token_here>"
```
## 📌 Summary
Built full JWT auth flow without Spring Security

Learned about filters, JWT structure, and role checking

Easy to extend to refresh tokens, logout, etc.




