# RabbitMQ Learning Project - E-Commerce Order Processing

A hands-on learning project to master RabbitMQ with Spring Boot microservices. This project simulates a real-world e-commerce order processing system where multiple services communicate asynchronously.

## 📋 Overview

This project demonstrates RabbitMQ concepts through a practical use case:
- **Order Service** - Creates orders and publishes events
- **Inventory Service** - Manages stock and inventory
- **Payment Service** - Processes payments with retry logic
- **Notification Service** - Sends customer notifications

## 🚀 Quick Start

### 1. Start RabbitMQ
```bash
docker-compose up -d
```

### 2. Access RabbitMQ Management UI
- URL: http://localhost:15672
- Username: `guest`
- Password: `guest`

### 3. Create Spring Boot Projects
Visit [start.spring.io](https://start.spring.io) and create projects as per the learning plan.

## 📚 Learning Plan

See **[RABBITMQ_LEARNING_PLAN.md](RABBITMQ_LEARNING_PLAN.md)** for the complete step-by-step guide.

### Learning Phases
- **Phase 1:** Basic Producer-Consumer Pattern
- **Phase 2:** Multiple Consumers & Routing
- **Phase 3:** Error Handling & Dead Letter Queues
- **Phase 4:** Production-Ready Features

## 🏗️ Project Structure (After Creating Services)

```
Rabitmq-test/
├── docker-compose.yml
├── README.md
├── RABBITMQ_LEARNING_PLAN.md
├── common-models/          # Shared DTOs/Events
├── order-service/          # Port 8081
├── inventory-service/      # Port 8082
├── notification-service/   # Port 8083
└── payment-service/        # Port 8084
```

## 🎯 Key RabbitMQ Concepts Covered

- ✅ Queues, Exchanges, and Bindings
- ✅ Direct, Fanout, and Topic Exchanges
- ✅ Message Routing and Patterns
- ✅ Manual Acknowledgements
- ✅ Dead Letter Queues (DLQ)
- ✅ Retry Mechanisms
- ✅ Message Durability
- ✅ Publisher Confirms
- ✅ Performance Optimization

## 📖 Next Steps

1. ✅ Read RABBITMQ_LEARNING_PLAN.md
2. ✅ Start RabbitMQ with docker-compose
3. Create Spring Boot projects from start.spring.io
4. Follow the learning plan phase by phase
5. Experiment and break things to learn!

## 🐛 Troubleshooting

If RabbitMQ doesn't start:
```bash
# Check if it's running
docker ps

# View logs
docker logs rabbitmq

# Restart
docker-compose restart
```

## 📚 Resources

- [RabbitMQ Official Documentation](https://www.rabbitmq.com/documentation.html)
- [Spring AMQP Reference](https://docs.spring.io/spring-amqp/reference/)
- [RabbitMQ Management UI Guide](https://www.rabbitmq.com/management.html)

---

**Happy Learning! 🎉**