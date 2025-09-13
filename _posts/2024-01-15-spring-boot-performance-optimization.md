---
layout: post
title: "Spring Boot 애플리케이션 성능 최적화 경험기"
date: 2024-01-15 10:00:00 +0900
categories: [Backend, Spring Boot]
tags: [spring-boot, performance, optimization, java]
description: >
  Spring Boot 애플리케이션의 성능을 크게 개선한 경험을 공유합니다.
image:
  path: /assets/img/blog/spring-boot-performance.jpg
  alt: Spring Boot Performance Optimization
---

최근 담당하고 있는 Spring Boot 애플리케이션에서 성능 이슈가 발생했습니다. 
트래픽이 증가하면서 응답 시간이 늘어나고, 때로는 타임아웃이 발생하는 상황이었습니다.

이 글에서는 문제를 분석하고 해결한 과정을 정리해보겠습니다.

## 문제 상황

### 초기 증상
- API 응답 시간: 평균 3-5초 (목표: 1초 이하)
- 피크 시간대 타임아웃 발생 빈도 증가
- 데이터베이스 커넥션 풀 고갈

### 성능 분석

먼저 **Spring Boot Actuator**와 **Micrometer**를 활용해 애플리케이션의 메트릭을 수집했습니다.

```java
@Component
public class PerformanceMetrics {
    
    private final MeterRegistry meterRegistry;
    private final Timer.Sample sample;
    
    public PerformanceMetrics(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
    }
    
    @EventListener
    public void handleRequest(RequestEvent event) {
        Timer.Sample.start(meterRegistry)
            .stop(Timer.builder("http.requests")
                .tag("uri", event.getUri())
                .tag("method", event.getMethod())
                .register(meterRegistry));
    }
}
```

## 해결 방안

### 1. 데이터베이스 쿼리 최적화

가장 큰 병목은 N+1 쿼리 문제였습니다.

**Before:**
```java
@Entity
public class Order {
    @OneToMany(fetch = FetchType.LAZY)
    private List<OrderItem> orderItems;
}

// N+1 쿼리 발생
public List<OrderDto> getAllOrders() {
    List<Order> orders = orderRepository.findAll();
    return orders.stream()
        .map(order -> OrderDto.builder()
            .id(order.getId())
            .items(order.getOrderItems()) // 각 order마다 추가 쿼리 발생
            .build())
        .collect(Collectors.toList());
}
```

**After:**
```java
// Fetch Join 사용
@Query("SELECT DISTINCT o FROM Order o LEFT JOIN FETCH o.orderItems")
List<Order> findAllWithItems();

// 또는 @EntityGraph 사용
@EntityGraph(attributePaths = {"orderItems"})
List<Order> findAll();
```

### 2. 캐싱 전략 도입

자주 조회되지만 변경이 적은 데이터에 대해 Redis 캐싱을 적용했습니다.

```java
@Service
@RequiredArgsConstructor
public class ProductService {
    
    private final ProductRepository productRepository;
    
    @Cacheable(value = "products", key = "#id")
    public ProductDto getProduct(Long id) {
        Product product = productRepository.findById(id)
            .orElseThrow(() -> new ProductNotFoundException(id));
        return ProductDto.from(product);
    }
    
    @CacheEvict(value = "products", key = "#productDto.id")
    public ProductDto updateProduct(ProductDto productDto) {
        // 업데이트 로직
    }
}
```

### 3. 커넥션 풀 튜닝

HikariCP 설정을 최적화했습니다.

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20
      minimum-idle: 10
      connection-timeout: 20000
      idle-timeout: 300000
      max-lifetime: 1200000
      leak-detection-threshold: 60000
```

### 4. 비동기 처리 도입

시간이 오래 걸리는 작업은 비동기로 처리하도록 변경했습니다.

```java
@Service
public class NotificationService {
    
    @Async("taskExecutor")
    public CompletableFuture<Void> sendEmailNotification(String email, String message) {
        // 이메일 발송 로직 (시간 소요)
        emailSender.send(email, message);
        return CompletableFuture.completedFuture(null);
    }
}

@Configuration
@EnableAsync
public class AsyncConfig {
    
    @Bean(name = "taskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(4);
        executor.setMaxPoolSize(8);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-task-");
        executor.initialize();
        return executor;
    }
}
```

## 결과

### 성능 개선 효과
- **API 응답 시간**: 3-5초 → 평균 800ms (84% 개선)
- **타임아웃 발생률**: 5% → 0.1% (95% 감소)
- **데이터베이스 쿼리 수**: 평균 50% 감소
- **서버 CPU 사용률**: 80% → 45% (44% 감소)

### 모니터링 개선

```java
@RestController
public class HealthController {
    
    @Autowired
    private DataSource dataSource;
    
    @GetMapping("/health/db")
    public ResponseEntity<Map<String, Object>> checkDatabaseHealth() {
        Map<String, Object> health = new HashMap<>();
        
        try (Connection connection = dataSource.getConnection()) {
            health.put("status", "UP");
            health.put("responseTime", measureResponseTime());
        } catch (SQLException e) {
            health.put("status", "DOWN");
            health.put("error", e.getMessage());
        }
        
        return ResponseEntity.ok(health);
    }
}
```

## 교훈

1. **측정 없이는 개선도 없다**: 정확한 메트릭 수집이 최적화의 첫 걸음
2. **데이터베이스가 병목의 주범**: 대부분의 성능 이슈는 DB 쿼리에서 발생
3. **캐싱은 양날의 검**: 적절한 캐시 전략과 무효화 정책이 중요
4. **비동기 처리의 힘**: 사용자 경험 개선에 큰 효과

앞으로는 개발 초기 단계부터 성능을 고려한 설계를 하고, 
지속적인 모니터링을 통해 성능 저하를 사전에 감지하도록 하겠습니다.

---

혹시 Spring Boot 성능 최적화 관련해서 궁금한 점이 있으시다면 댓글로 남겨주세요! 🚀