---
title: "Building Centralized Authentication System with Spring Cloud Gateway & OAuth2"
date: 2025-07-07
categories: [Spring Boot, Security, OAuth2, Architecture]
tags: [OAuth2, Spring Security, Gateway, Microservice, Distributed Auth]
---

When your system scales into multiple microservices, managing secure access across services becomes essential. A **centralized authentication mechanism (SSO)** ensures a consistent security policy and a seamless login experience.

In this post, we’ll build an **OAuth2-based centralized authentication system** using:

- **Spring Cloud Gateway** as the entry point
- A separate **Authorization Server** (Keycloak or Spring Auth Server)
- **JWT tokens** for stateless access control
- Secure access to downstream services

---

## 🧭 System Architecture Overview

[Client] --> [Spring Cloud Gateway] --> [Service A]
                           |                   |
                           v                   v
                 [Authorization Server] <--- [UserDB]
---

## 🧱 1. Spring Cloud Gateway as Resource Server

```java
@Configuration
@EnableWebFluxSecurity
public class SecurityConfig {

    @Bean
    public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
        http
            .authorizeExchange(ex -> ex
                .pathMatchers("/login", "/oauth/**").permitAll()
                .anyExchange().authenticated())
            .oauth2ResourceServer(ServerHttpSecurity.OAuth2ResourceServerSpec::jwt);
        return http.build();
    }
}
```
This config:

Protects all routes except login

Validates JWT for incoming requests

🔑 2. Authorization Server (Keycloak Example)
Use Keycloak or Spring Authorization Server.

Create in Keycloak:

Realm: myrealm

Client: gateway-client

Redirect URI: http://localhost:8080/login/oauth2/code/gateway-client

Grant type: Authorization Code
Configure application.yml:
```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          keycloak:
            client-id: gateway-client
            client-secret: <your-secret>
            scope: openid
            redirect-uri: "{baseUrl}/login/oauth2/code/{registrationId}"
            authorization-grant-type: authorization_code
        provider:
          keycloak:
            issuer-uri: http://localhost:8080/realms/myrealm
```
🚪 3. Securing Microservices with JWT
```java
@Configuration
@EnableWebSecurity
public class ResourceServerConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/internal/**").denyAll()
                .anyRequest().authenticated()
            .and()
            .oauth2ResourceServer()
                .jwt();
    }
}
```
📦 4. Role-based Access Control (RBAC)
```java
@GetMapping("/admin")
@PreAuthorize("hasAuthority('ROLE_ADMIN')")
public ResponseEntity<String> onlyAdminAccess() {
    return ResponseEntity.ok("Welcome admin!");
}
```
To access claims directly:
```java
@GetMapping("/profile")
public ResponseEntity<?> getProfile(@AuthenticationPrincipal Jwt jwt) {
    String username = jwt.getClaim("preferred_username");
    return ResponseEntity.ok(Map.of("user", username));
}
```
🛡️ 5. Gateway Filters: Rate Limiting & Resilience
Protect backend services with RequestRateLimiter and circuit breakers:
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: product
          uri: lb://product-service
          predicates:
            - Path=/product/**
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20
```
✅ Summary
You've now built a scalable, secure, and centralized authentication system using:

Spring Gateway as API entry point

JWT tokens for secure identity propagation

Keycloak as OAuth2 Authorization Server

Role-based access control and filtering




