# RabbitMQ Learning Plan - E-Commerce Order Processing System

## 📌 Use Case Overview

**Scenario:** E-Commerce Order Processing System

When a customer places an order, multiple services need to be notified and process the order:
- **Order Service** → Creates order and publishes event
- **Inventory Service** → Reserves/updates stock
- **Payment Service** → Processes payment
- **Notification Service** → Sends confirmation emails/SMS

This mimics real-world microservices architecture where services communicate asynchronously.

---

## 🏗️ Project Structure

```
Rabitmq-test/
├── docker-compose.yml                 # RabbitMQ + Management UI
├── RABBITMQ_LEARNING_PLAN.md         # This file
├── README.md
├── common-models/                     # Shared DTOs/Events (Phase 1)
│   ├── pom.xml
│   └── src/main/java/com/ecommerce/common/
│       ├── dto/
│       │   ├── OrderDTO.java
│       │   ├── OrderStatus.java
│       │   └── OrderEvent.java
│       └── config/
├── order-service/                     # Port 8081 (Phase 1)
│   ├── pom.xml
│   └── src/main/java/com/ecommerce/order/
│       ├── OrderServiceApplication.java
│       ├── controller/
│       │   └── OrderController.java
│       ├── service/
│       │   └── OrderService.java
│       ├── publisher/
│       │   └── OrderPublisher.java
│       └── config/
│           └── RabbitMQConfig.java
├── inventory-service/                 # Port 8082 (Phase 1)
│   ├── pom.xml
│   └── src/main/java/com/ecommerce/inventory/
│       ├── InventoryServiceApplication.java
│       ├── consumer/
│       │   └── OrderConsumer.java
│       ├── service/
│       │   └── InventoryService.java
│       └── config/
│           └── RabbitMQConfig.java
├── notification-service/              # Port 8083 (Phase 2)
│   ├── pom.xml
│   └── src/main/java/com/ecommerce/notification/
│       ├── NotificationServiceApplication.java
│       ├── consumer/
│       │   └── OrderEventConsumer.java
│       ├── service/
│       │   └── NotificationService.java
│       └── config/
│           └── RabbitMQConfig.java
└── payment-service/                   # Port 8084 (Phase 3)
    ├── pom.xml
    └── src/main/java/com/ecommerce/payment/
        ├── PaymentServiceApplication.java
        ├── consumer/
        │   └── OrderConsumer.java
        ├── service/
        │   └── PaymentService.java
        └── config/
            └── RabbitMQConfig.java
```

---

## 🎯 Phase 1: Basic Producer-Consumer Pattern (Day 1)

### Goal
Understand basic message publishing and consuming with direct exchange.

### Services to Create
1. **common-models** (Shared library)
2. **order-service** (Producer)
3. **inventory-service** (Consumer)

### Dependencies for Each Service

**common-models/pom.xml:**
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
    </dependency>
</dependencies>
```

**order-service, inventory-service/pom.xml:**
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### RabbitMQ Concepts to Learn
- ✅ **Queue**: Where messages are stored
- ✅ **Exchange**: Routes messages to queues
- ✅ **Binding**: Links exchange to queue
- ✅ **Direct Exchange**: Routes based on exact routing key match
- ✅ **Producer**: Publishes messages
- ✅ **Consumer**: Receives and processes messages

### Configuration Files

**order-service/application.yml:**
```yaml
server:
  port: 8081

spring:
  application:
    name: order-service
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest

rabbitmq:
  exchange:
    name: order.exchange
  queue:
    name: order.queue
  routing:
    key: order.created
```

**inventory-service/application.yml:**
```yaml
server:
  port: 8082

spring:
  application:
    name: inventory-service
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest

rabbitmq:
  exchange:
    name: order.exchange
  queue:
    name: order.queue
  routing:
    key: order.created
```

### Implementation Steps

1. **Create common-models project from start.spring.io:**
   - Project: Maven
   - Language: Java
   - Spring Boot: 3.2.x
   - Packaging: Jar
   - Java: 17
   - Dependencies: Lombok

2. **Create order-service from start.spring.io:**
   - Project: Maven
   - Language: Java
   - Spring Boot: 3.2.x
   - Dependencies: Spring Web, Spring for RabbitMQ, Lombok

3. **Create inventory-service from start.spring.io:**
   - Same as order-service

4. **Setup Docker Compose for RabbitMQ**

5. **Implement:**
   - OrderDTO, OrderEvent in common-models
   - REST endpoint to create order in order-service
   - RabbitMQ publisher in order-service
   - RabbitMQ consumer in inventory-service
   - Basic inventory logic

6. **Test:**
   - Start RabbitMQ: `docker-compose up -d`
   - Start inventory-service
   - Start order-service
   - POST order via REST API
   - Check logs in inventory-service
   - Check RabbitMQ Management UI: http://localhost:15672

### Expected Learning Outcomes
- Understand message flow: Publisher → Exchange → Queue → Consumer
- See messages in RabbitMQ Management UI
- Understand JSON serialization/deserialization
- Basic Spring AMQP configuration

---

## 🎯 Phase 2: Multiple Consumers & Routing (Day 2)

### Goal
Learn message routing patterns and broadcasting to multiple services.

### Services to Add
1. **notification-service** (Consumer)

### RabbitMQ Concepts to Learn
- ✅ **Fanout Exchange**: Broadcasts to all bound queues
- ✅ **Topic Exchange**: Pattern-based routing (wildcards)
- ✅ **Routing Keys**: Used for intelligent routing
- ✅ **Multiple Queues**: One exchange to many queues
- ✅ **Binding Keys**: Pattern matching for topic exchange

### Routing Patterns to Implement

**Topic Exchange Routing:**
```
order.created       → All services
order.cancelled     → Inventory, Notification
order.completed     → Notification only
order.updated       → Inventory, Notification
```

### Configuration Updates

**order-service/application.yml:**
```yaml
server:
  port: 8081

spring:
  application:
    name: order-service
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest

rabbitmq:
  exchange:
    name: order.topic.exchange
    type: topic
```

**inventory-service/application.yml:**
```yaml
server:
  port: 8082

spring:
  application:
    name: inventory-service
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest

rabbitmq:
  exchange:
    name: order.topic.exchange
  queue:
    name: inventory.queue
  routing:
    key: order.created,order.cancelled,order.updated
```

**notification-service/application.yml:**
```yaml
server:
  port: 8083

spring:
  application:
    name: notification-service
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest

rabbitmq:
  exchange:
    name: order.topic.exchange
  queue:
    name: notification.queue
  routing:
    key: order.*  # Listens to all order events
```

### Implementation Steps

1. **Create notification-service from start.spring.io**
2. **Update order-service:**
   - Add methods to publish different event types
   - Create endpoints: POST /orders, PUT /orders/{id}/cancel, PUT /orders/{id}/complete
3. **Update inventory-service:**
   - Configure topic exchange
   - Handle multiple event types
4. **Implement notification-service:**
   - Configure to listen to all order.* events
   - Log/send notifications for each event type

### Testing Scenarios

1. Create order → Both inventory and notification receive
2. Cancel order → Both inventory and notification receive
3. Complete order → Only notification receives
4. Check RabbitMQ UI to see exchange bindings

### Expected Learning Outcomes
- Understand topic exchange wildcards (* and #)
- See how one message can reach multiple consumers
- Understand selective routing based on patterns
- Learn queue binding strategies

---

## 🎯 Phase 3: Error Handling & Resilience (Day 3)

### Goal
Handle failures gracefully with retries and dead letter queues.

### Services to Add
1. **payment-service** (Consumer with failure scenarios)

### RabbitMQ Concepts to Learn
- ✅ **Manual Acknowledgement**: Control when messages are acknowledged
- ✅ **NACK (Negative Acknowledgement)**: Reject messages
- ✅ **Dead Letter Exchange (DLX)**: Handle failed messages
- ✅ **Dead Letter Queue (DLQ)**: Store failed messages
- ✅ **Message TTL**: Time-to-live for messages
- ✅ **Retry Mechanism**: Exponential backoff
- ✅ **Message Requeue**: Return message to queue

### Configuration Updates

**payment-service/application.yml:**
```yaml
server:
  port: 8084

spring:
  application:
    name: payment-service
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
    listener:
      simple:
        acknowledge-mode: manual  # Manual ACK
        prefetch: 1               # Process one message at a time
        retry:
          enabled: true
          initial-interval: 2000  # 2 seconds
          max-attempts: 3
          max-interval: 10000     # 10 seconds
          multiplier: 2           # Exponential backoff

rabbitmq:
  exchange:
    name: order.topic.exchange
  queue:
    name: payment.queue
    dlq: payment.dlq            # Dead Letter Queue
  routing:
    key: order.created
  dlx:
    name: order.dlx.exchange    # Dead Letter Exchange
  dlx-routing-key: payment.failed
```

### Dead Letter Queue Setup

**RabbitMQConfig.java (payment-service):**
```java
@Configuration
public class RabbitMQConfig {
    
    @Bean
    public Queue paymentQueue() {
        return QueueBuilder.durable("payment.queue")
            .withArgument("x-dead-letter-exchange", "order.dlx.exchange")
            .withArgument("x-dead-letter-routing-key", "payment.failed")
            .withArgument("x-message-ttl", 60000) // 60 seconds TTL
            .build();
    }
    
    @Bean
    public Queue paymentDLQ() {
        return new Queue("payment.dlq", true);
    }
    
    @Bean
    public DirectExchange deadLetterExchange() {
        return new DirectExchange("order.dlx.exchange");
    }
    
    @Bean
    public Binding dlqBinding() {
        return BindingBuilder
            .bind(paymentDLQ())
            .to(deadLetterExchange())
            .with("payment.failed");
    }
}
```

### Implementation Steps

1. **Create payment-service from start.spring.io**
2. **Implement payment processing with failure scenarios:**
   - Simulate payment gateway timeout
   - Simulate insufficient funds
   - Simulate network errors
3. **Implement manual acknowledgement:**
   - ACK on success
   - NACK on failure
   - Retry logic
4. **Configure DLQ:**
   - Setup dead letter exchange
   - Create DLQ monitoring endpoint
5. **Test failure scenarios:**
   - Temporary failure → Retry → Success
   - Permanent failure → Move to DLQ

### Testing Scenarios

1. **Success Case:**
   - Valid payment → Acknowledged → Order completed

2. **Retry Case:**
   - Payment gateway timeout → NACK → Retry 3 times → Success

3. **DLQ Case:**
   - Invalid card → NACK → Retry 3 times → Move to DLQ

4. **Manual Recovery:**
   - Check DLQ
   - Fix issue
   - Reprocess message from DLQ

### Expected Learning Outcomes
- Understand message acknowledgement lifecycle
- Know when to use manual vs auto acknowledgement
- Implement retry strategies
- Handle permanently failed messages
- Monitor and recover from failures

---

## 🎯 Phase 4: Production-Ready Features (Day 4)

### Goal
Make the system production-ready with durability, monitoring, and optimization.

### RabbitMQ Concepts to Learn
- ✅ **Durable Queues**: Survive broker restart
- ✅ **Persistent Messages**: Messages survive broker restart
- ✅ **Publisher Confirms**: Ensure message delivery
- ✅ **Consumer Prefetch**: Control concurrent message processing
- ✅ **Connection Pooling**: Optimize connections
- ✅ **Health Checks**: Monitor service health
- ✅ **Message Priority**: Prioritize urgent messages

### Configuration Updates

**All Services - Enhanced application.yml:**
```yaml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
    publisher-confirm-type: correlated    # Enable publisher confirms
    publisher-returns: true
    template:
      mandatory: true
      retry:
        enabled: true
        initial-interval: 1000
        max-attempts: 3
        max-interval: 10000
        multiplier: 2.0
    listener:
      simple:
        acknowledge-mode: manual
        prefetch: 5                       # Process 5 messages concurrently
        retry:
          enabled: true
          initial-interval: 2000
          max-attempts: 3
          multiplier: 2.0
        default-requeue-rejected: false   # Don't requeue on rejection
    cache:
      channel:
        size: 25                          # Channel pool size
        checkout-timeout: 0

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: always
```

### Durable Queue Configuration

```java
@Bean
public Queue orderQueue() {
    return QueueBuilder.durable("order.queue")  // Durable queue
        .withArgument("x-max-priority", 10)      // Enable priority
        .build();
}

@Bean
public TopicExchange orderExchange() {
    return ExchangeBuilder
        .topicExchange("order.topic.exchange")
        .durable(true)                           // Durable exchange
        .build();
}
```

### Publisher Confirms Implementation

```java
@Service
public class OrderPublisher {
    
    private final RabbitTemplate rabbitTemplate;
    
    public void publishWithConfirm(OrderEvent event) {
        rabbitTemplate.setConfirmCallback((correlationData, ack, cause) -> {
            if (ack) {
                log.info("Message delivered successfully");
            } else {
                log.error("Message delivery failed: {}", cause);
                // Handle delivery failure
            }
        });
        
        rabbitTemplate.setReturnsCallback(returned -> {
            log.error("Message returned: {}", returned.getMessage());
            // Handle returned message
        });
        
        // Send with priority
        rabbitTemplate.convertAndSend(
            exchange, 
            routingKey, 
            event,
            message -> {
                message.getMessageProperties().setPriority(5);
                message.getMessageProperties().setDeliveryMode(MessageDeliveryMode.PERSISTENT);
                return message;
            }
        );
    }
}
```

### Monitoring & Health Checks

```java
@Component
public class RabbitMQHealthIndicator implements HealthIndicator {
    
    private final RabbitTemplate rabbitTemplate;
    
    @Override
    public Health health() {
        try {
            rabbitTemplate.getConnectionFactory()
                .createConnection()
                .close();
            return Health.up()
                .withDetail("rabbitmq", "Connection successful")
                .build();
        } catch (Exception e) {
            return Health.down()
                .withDetail("rabbitmq", "Connection failed")
                .withException(e)
                .build();
        }
    }
}
```

### Performance Optimization

**Consumer with Batching:**
```java
@RabbitListener(queues = "high-volume.queue")
public void processInBatch(List<OrderEvent> events) {
    // Process multiple messages at once
    log.info("Processing batch of {} orders", events.size());
    // Bulk database operations
}
```

### Implementation Steps

1. **Update all services:**
   - Enable publisher confirms
   - Configure durable queues and exchanges
   - Add message priority support
   - Implement health checks

2. **Add monitoring:**
   - Spring Boot Actuator endpoints
   - Custom metrics for message processing
   - Error rate monitoring

3. **Performance testing:**
   - Load test with 1000+ messages
   - Monitor consumer performance
   - Tune prefetch count

4. **Add logging and tracing:**
   - Request ID propagation
   - Distributed tracing setup
   - Structured logging

### Testing Scenarios

1. **Durability Test:**
   - Send messages
   - Stop RabbitMQ
   - Restart RabbitMQ
   - Verify messages still exist

2. **Performance Test:**
   - Send 10,000 messages
   - Monitor throughput
   - Check consumer lag

3. **Failure Recovery:**
   - Kill a consumer
   - Verify messages are redelivered
   - Check message requeue behavior

4. **Priority Test:**
   - Send low priority messages
   - Send high priority messages
   - Verify high priority processed first

### Expected Learning Outcomes
- Configure production-grade RabbitMQ
- Implement reliable message delivery
- Optimize for performance
- Monitor and troubleshoot issues
- Understand durability guarantees

---

## 🐳 Docker Compose Configuration

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  rabbitmq:
    image: rabbitmq:3.13-management
    container_name: rabbitmq
    ports:
      - "5672:5672"    # AMQP port
      - "15672:15672"  # Management UI
    environment:
      RABBITMQ_DEFAULT_USER: guest
      RABBITMQ_DEFAULT_PASS: guest
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    networks:
      - ecommerce-network
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "ping"]
      interval: 30s
      timeout: 10s
      retries: 5

volumes:
  rabbitmq_data:

networks:
  ecommerce-network:
    driver: bridge
```

---

## 📝 Spring Initializr Settings for Each Service

### common-models
- **Project:** Maven
- **Language:** Java
- **Spring Boot:** 3.2.x
- **Packaging:** Jar
- **Java:** 17
- **Group:** com.ecommerce
- **Artifact:** common-models
- **Dependencies:** 
  - Lombok

### order-service
- **Project:** Maven
- **Language:** Java
- **Spring Boot:** 3.2.x
- **Packaging:** Jar
- **Java:** 17
- **Group:** com.ecommerce
- **Artifact:** order-service
- **Dependencies:**
  - Spring Web
  - Spring for RabbitMQ
  - Lombok
  - Spring Boot Actuator (Phase 4)

### inventory-service
- **Project:** Maven
- **Language:** Java
- **Spring Boot:** 3.2.x
- **Packaging:** Jar
- **Java:** 17
- **Group:** com.ecommerce
- **Artifact:** inventory-service
- **Dependencies:**
  - Spring Web
  - Spring for RabbitMQ
  - Lombok
  - Spring Boot Actuator (Phase 4)

### notification-service (Phase 2)
- Same as inventory-service
- **Artifact:** notification-service

### payment-service (Phase 3)
- Same as inventory-service
- **Artifact:** payment-service

---

## 🚀 Quick Start Commands

### Start RabbitMQ
```bash
docker-compose up -d
```

### Access RabbitMQ Management UI
```
URL: http://localhost:15672
Username: guest
Password: guest
```

### Start Services (in order)
```bash
# Terminal 1 - Inventory Service
cd inventory-service
./mvnw spring-boot:run

# Terminal 2 - Order Service
cd order-service
./mvnw spring-boot:run

# Terminal 3 - Notification Service (Phase 2+)
cd notification-service
./mvnw spring-boot:run

# Terminal 4 - Payment Service (Phase 3+)
cd payment-service
./mvnw spring-boot:run
```

### Test API Endpoints

**Create Order:**
```bash
curl -X POST http://localhost:8081/api/orders \
  -H "Content-Type: application/json" \
  -d '{
    "customerId": "CUST001",
    "productId": "PROD001",
    "quantity": 2,
    "totalAmount": 199.99
  }'
```

**Cancel Order:**
```bash
curl -X PUT http://localhost:8081/api/orders/ORDER123/cancel
```

**Complete Order:**
```bash
curl -X PUT http://localhost:8081/api/orders/ORDER123/complete
```

---

## 🎓 Learning Checkpoints

### After Phase 1
- [ ] Can create and send a message to RabbitMQ
- [ ] Understand queue, exchange, and binding
- [ ] Can consume messages in Spring Boot
- [ ] Can view messages in Management UI
- [ ] Understand direct exchange routing

### After Phase 2
- [ ] Understand fanout vs topic exchanges
- [ ] Can route messages to multiple consumers
- [ ] Understand routing key patterns
- [ ] Can use wildcards in topic exchange
- [ ] Understand broadcast patterns

### After Phase 3
- [ ] Can handle message failures
- [ ] Understand manual acknowledgement
- [ ] Can implement retry logic
- [ ] Understand dead letter queues
- [ ] Can recover from failures

### After Phase 4
- [ ] Understand message durability
- [ ] Can implement publisher confirms
- [ ] Know how to optimize performance
- [ ] Can monitor RabbitMQ health
- [ ] Production-ready configuration

---

## 🐛 Common Issues & Solutions

### Issue 1: Connection Refused
**Problem:** Cannot connect to RabbitMQ
**Solution:** 
- Check if RabbitMQ is running: `docker ps`
- Verify port 5672 is not in use
- Check firewall settings

### Issue 2: Messages Not Being Consumed
**Problem:** Messages stuck in queue
**Solution:**
- Check if consumer service is running
- Verify queue binding is correct
- Check routing key matches
- Look for exceptions in consumer logs

### Issue 3: Messages Lost After Restart
**Problem:** Messages disappear when RabbitMQ restarts
**Solution:**
- Use durable queues
- Set message delivery mode to PERSISTENT
- Enable publisher confirms

### Issue 4: Consumer Overwhelmed
**Problem:** Too many messages, consumer can't keep up
**Solution:**
- Reduce prefetch count
- Scale horizontally (multiple consumers)
- Implement batching
- Add more resources

---

## 📚 Additional Resources

### RabbitMQ Documentation
- Official Docs: https://www.rabbitmq.com/documentation.html
- AMQP Concepts: https://www.rabbitmq.com/tutorials/amqp-concepts.html

### Spring AMQP
- Reference: https://docs.spring.io/spring-amqp/reference/
- API Docs: https://docs.spring.io/spring-amqp/docs/current/api/

### Best Practices
- RabbitMQ Best Practices: https://www.cloudamqp.com/blog/part1-rabbitmq-best-practice.html
- Message Patterns: https://www.enterpriseintegrationpatterns.com/

---

## ✅ Next Steps

1. Create docker-compose.yml in root folder
2. Generate all Spring Boot projects from start.spring.io
3. Commit to repository
4. Follow this plan phase by phase
5. Test each phase before moving to next
6. Experiment with different scenarios
7. Read RabbitMQ documentation alongside implementation

---

**Happy Learning! 🎉**

Feel free to experiment, break things, and learn from failures. That's the best way to master RabbitMQ!
