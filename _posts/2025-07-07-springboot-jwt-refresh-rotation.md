---
title: "Spring Boot Security: JWT with Refresh Token Rotation"
date: 2025-07-07
categories: [Spring Boot, Security, JWT]
tags: [Spring Security, JWT, Refresh Token, Java, Backend]
---

Implementing JWT authentication is common, but securing it properly with **refresh token rotation** is a next-level skill. In this guide, we will:

- Issue **access** and **refresh tokens**
- Store refresh tokens securely
- Rotate refresh tokens on each use to prevent replay attacks
- Blacklist old tokens using a token store

This is an **advanced and production-grade** pattern, often used in systems like Auth0 or Keycloak.

---

## 🧠 Why Rotate Refresh Tokens?

Without rotation, if a refresh token is stolen, it can be used indefinitely. With rotation, the previous token is **invalidated immediately after use**, drastically reducing the attack window.

---

## 📦 1. Dependencies

Use the following Spring Boot dependencies:

```java
implementation 'org.springframework.boot:spring-boot-starter-security'
implementation 'io.jsonwebtoken:jjwt-api:0.11.5'
runtimeOnly 'io.jsonwebtoken:jjwt-impl:0.11.5'
runtimeOnly 'io.jsonwebtoken:jjwt-jackson:0.11.5'
```
🔐 2. JWT Utility
```java
@Component
public class JwtUtils {

    private final Key key = Keys.secretKeyFor(SignatureAlgorithm.HS512);

    public String generateToken(String username, long expirationMillis) {
        return Jwts.builder()
                .setSubject(username)
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + expirationMillis))
                .signWith(key)
                .compact();
    }

    public String getUsername(String token) {
        return Jwts.parserBuilder().setSigningKey(key).build()
                .parseClaimsJws(token).getBody().getSubject();
    }
}
```
🔁 3. Refresh Token Rotation
```java
@Entity
public class RefreshToken {
    @Id
    private String token;
    private String username;
    private Instant expiryDate;
}
```
On login:
Generate both access and refresh tokens

Save refresh token to DB

On token refresh:
Validate refresh token

Delete it from DB

Issue a new access + refresh token pair

Save the new refresh token

This ensures old refresh tokens are invalid after a single use.

🔄 4. Refresh Endpoint Example
```java
@PostMapping("/refresh")
public ResponseEntity<?> refreshToken(@RequestBody String oldToken) {
    Optional<RefreshToken> saved = refreshTokenRepository.findById(oldToken);
    if (saved.isEmpty() || saved.get().getExpiryDate().isBefore(Instant.now())) {
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
    }

    refreshTokenRepository.delete(saved.get());

    String username = saved.get().getUsername();
    String newAccessToken = jwtUtils.generateToken(username, 15 * 60 * 1000); // 15 mins
    String newRefreshToken = UUID.randomUUID().toString();

    RefreshToken newToken = new RefreshToken(newRefreshToken, username, Instant.now().plus(7, ChronoUnit.DAYS));
    refreshTokenRepository.save(newToken);

    return ResponseEntity.ok(Map.of("accessToken", newAccessToken, "refreshToken", newRefreshToken));
}
```
🧷 5. Security Config
```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.csrf().disable()
        .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
        .and()
        .authorizeHttpRequests()
            .requestMatchers("/login", "/refresh").permitAll()
            .anyRequest().authenticated();
}
```
🔚 Conclusion
JWT alone is not enough for a secure auth system. Refresh token rotation ensures your tokens are safe, even in case of interception. You now have a battle-tested authentication system.

✅ Next step: Add IP/device tracking and logout support by invalidating refresh tokens manually!
