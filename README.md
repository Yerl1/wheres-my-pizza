# wheres-my-pizza
![wheres-my-pizza](https://img.shields.io/badge/status-competed-green)

## Overview 
A distributed restaurant order management system implemented in Go, using microservices architecture with RabbitMQ for messaging and PostgreSQL for data persistence. This system simulates a real-world restaurant workflow, from receiving orders to processing in the kitchen and tracking their status.

## Features

- Order Service – Receives and validates customer orders via HTTP API, stores them in PostgreSQL, and publishes messages to RabbitMQ.
- Kitchen Worker – Consumes order messages, processes them, updates order status, and publishes status updates to a fanout exchange.
- Tracking Service – Provides a read-only HTTP API to query the current status and history of orders, and monitor kitchen worker statuses.
- Notification Subscriber – Listens to status updates and outputs notifications, simulating a real-time notification system.

## Architecture
The system uses a microservices design with asynchronous communication:
```rust
HTTP Client --> Order Service --> RabbitMQ --> Kitchen Worker --> RabbitMQ --> Notification Subscriber
                             |
                             v
                        PostgreSQL DB
                             |
                             v
                     Tracking Service
```
- RabbitMQ handles message routing using work queue, publish/subscribe, and routing patterns.
- PostgreSQL stores orders, order items, order status logs, and worker information.
- All services implement structured JSON logging for observability.

## Tech Stack
- Language: Go
- Database: PostgreSQL (pgx/v5)
- Message Broker: RabbitMQ (amqp091-go)
- Concurrency & Microservices: Goroutines, channels, and messaging patterns

## Setup
1. Start RabbitMQ and PostgreSQL.
2. Create the required database and tables (see SQL migration scripts).
3. Configure ```config.yaml``` with database and RabbitMQ credentials:
```yaml
database:
  host: localhost
  port: 5432
  user: restaurant_user
  password: restaurant_pass
  database: restaurant_db

rabbitmq:
  host: localhost
  port: 5672
  user: guest
  password: guest

```
4. Build the application:
```sh
go build -o restaurant-system .
```
5. Run individual services:
```sh
# Order Service
./restaurant-system --mode=order-service --port=3000

# Kitchen Worker
./restaurant-system --mode=kitchen-worker --worker-name="chef_anna"

# Tracking Service
./restaurant-system --mode=tracking-service --port=3002

# Notification Subscriber
./restaurant-system --mode=notification-subscriber
```
## Logging
- Structured JSON logs include ```timestamp```, ```level```, ```service```, ```action```, ```message```, ```hostname```, and ```request_id```.
- Error logs also contain a structured ```error``` object with ```msg``` and ```stack```.
