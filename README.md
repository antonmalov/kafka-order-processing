# Kafka Order Processing

[![GitHub](https://img.shields.io/badge/Java-21-blue)](https://www.java.com/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-4.0.4-green)](https://spring.io/projects/spring-boot)
[![Apache Kafka](https://img.shields.io/badge/Apache%20Kafka-4.1.2-red)](https://kafka.apache.org/)
[![Allure Report](https://img.shields.io/badge/Allure-Report-blue)](https://github.com/antonmalov/kafka-order-e2e-tests#allure-report)

Проект демонстрирует асинхронную обработку заказов с использованием Apache Kafka.

## Архитектура

┌─────────────┐     ┌─────────┐     ┌─────────────┐     ┌─────────────┐
│   Client    │ ──► │Producer │ ──► │   Kafka     │ ──► │  Consumer   │
│  (REST)     │     │(8081)   │     │(orders-v2)  │     │  (8080)     │
└─────────────┘     └─────────┘     └─────────────┘     └─────────────┘
                                           │
                                           ▼
                                     ┌─────────────┐
                                     │     DLT     │
                                     │(orders-v2.  │
                                     │    DLT)    │
                                     └─────────────┘

## Репозитории

| Репозиторий | Описание | Ссылка |
|-------------|----------|--------|
| **common-dto** | Общие DTO | [kafka-order-common-dto](https://github.com/antonmalov/kafka-order-common-dto) |
| **producer** | Продюсер (REST → Kafka) | [kafka-order-producer](https://github.com/antonmalov/kafka-order-producer) |
| **consumer** | Консьюмер с ретраями и DLT | [kafka-order-consumer](https://github.com/antonmalov/kafka-order-consumer) |
| **e2e-tests** | E2E тесты с Allure-отчётами | [kafka-order-e2e-tests](https://github.com/antonmalov/kafka-order-e2e-tests) |

## Быстрый старт

### 1. Запустить инфраструктуру

docker-compose up -d

### 2. Клонировать и собрать сервисы

# Клонируем и собираем common-dto
git clone https://github.com/antonmalov/kafka-order-common-dto.git
cd kafka-order-common-dto
mvn clean install

# Клонируем и собираем продюсера
cd ..
git clone https://github.com/antonmalov/kafka-order-producer.git
cd kafka-order-producer
mvn clean package

# Клонируем и собираем консьюмера
cd ..
git clone https://github.com/antonmalov/kafka-order-consumer.git
cd kafka-order-consumer
mvn clean package

# Клонируем E2E-тесты
cd ..
git clone https://github.com/antonmalov/kafka-order-e2e-tests.git
cd kafka-order-e2e-tests

### 3. Запустить сервисы

# Продюсер (порт 8081)
java -jar kafka-order-producer/target/order-service-0.0.1-SNAPSHOT.jar

# Консьюмер (порт 8080)
java -jar kafka-order-consumer/target/notification-service-0.0.1-SNAPSHOT.jar

### 4. Запустить тесты и сгенерировать Allure-отчёт

cd ../kafka-order-e2e-tests
.\run-tests.bat

После выполнения тестов сгенерируйте отчёт:

allure generate target/allure-results -o target/site/allure-maven --clean
allure open target/site/allure-maven

Отчёт покажет детальную статистику, историю тестов и логи выполнения.  
Подробнее о настройке Allure читайте в [README репозитория e2e-tests](https://github.com/antonmalov/kafka-order-e2e-tests#allure-report).

## Ручное тестирование

# Успешная обработка
curl -X POST http://localhost:8081/orders \
  -H "Content-Type: application/json" \
  -d '{"orderId":"1","product":"book","quantity":2}'

# Ошибка → DLT
curl -X POST http://localhost:8081/orders \
  -H "Content-Type: application/json" \
  -d '{"orderId":"2","product":"fail","quantity":1}'