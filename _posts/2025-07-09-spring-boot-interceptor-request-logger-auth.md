---
title: "Spring Boot Interceptor: Create a Custom Request Logger and Auth Layer"
date: 2025-07-09
categories: [Spring Boot, Middleware, Web]
tags: [spring boot, interceptor, logging, auth, middleware]
---

Interceptors are a powerful but often overlooked feature of Spring Boot. They allow you to intercept every HTTP request **before** it reaches your controller — perfect for logging, authentication, or custom header checks.

In this guide, we’ll build:

- A custom request logger
- An interceptor that blocks requests without a valid API key
- Integration with Spring Boot’s `WebMvcConfigurer`

---

## 🧱 Step 1: Create a Basic Interceptor

```java
public class RequestLoggingInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) throws Exception {
        System.out.println("🔍 Incoming Request: " + request.getMethod() + " " + request.getRequestURI());
        return true; // allow request to continue
    }
}
```
## 🔐 Step 2: Add Simple Auth Check
```java
public class ApiKeyAuthInterceptor implements HandlerInterceptor {
    private static final String VALID_API_KEY = "123456";

    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) throws Exception {
        String apiKey = request.getHeader("X-API-KEY");
        if (!VALID_API_KEY.equals(apiKey)) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            response.getWriter().write("❌ Invalid API Key");
            return false;
        }
        return true;
    }
}
```
## 🔗 Step 3: Register Interceptors
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new RequestLoggingInterceptor());
        registry.addInterceptor(new ApiKeyAuthInterceptor())
                .addPathPatterns("/api/**"); // only apply to API paths
    }
}
```
## 🧪 Test the Interceptor
curl -H "X-API-KEY: 123456" http://localhost:8080/api/hello

## ✅ Summary
In this post, you learned how to:

Create a custom interceptor in Spring Boot

Log every incoming request

Block requests based on custom headers

Apply interceptors to specific paths only


