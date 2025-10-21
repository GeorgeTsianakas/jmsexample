# JMS Messaging (Spring Boot + ActiveMQ Artemis)

A minimal, production-style example of JMS messaging with Spring Boot and an embedded ActiveMQ Artemis broker. It demonstrates:
- Fire-and-forget messaging (producer -> queue -> listener)
- Request-reply messaging using JMSReplyTo
- JSON payload conversion with Spring’s MappingJackson2MessageConverter
- Scheduling and async execution in Spring

![](https://img.shields.io/badge/Editor-IntelliJ-informational?style=flat&logo=intellij-idea)
![](https://img.shields.io/badge/Java-8+-informational?style=flat&logo=java)
![](https://img.shields.io/badge/Spring_Boot-2.2.6.RELEASE-informational?style=flat&logo=spring)
![](https://img.shields.io/badge/Broker-ActiveMQ_Artemis-informational)

---

## Table of Contents
- Overview
- Architecture
- Features
- Prerequisites
- Getting Started
  - Build
  - Run (development)
  - Run (packaged JAR)
- Configuration
- How It Works
- Project Structure
- Troubleshooting
- References

## Overview
This project boots an embedded ActiveMQ Artemis broker and a Spring Boot application that sends and receives JMS messages every 2 seconds. It is ideal for learning JMS concepts (queues, message converters, request-reply) without requiring an external broker.

## Architecture
- Embedded Broker: ActiveMQ Artemis started in-process with in-VM transport (vm://0), security disabled, no persistence.
- Producer: Scheduled sender publishes HelloWorldMessage payloads as JSON.
- Consumer: JMS listener consumes messages. A dedicated listener handles request-reply and sends a response to JMSReplyTo.
- Message Format: HelloWorldMessage { id: UUID, message: String } serialized as JSON with type info header `_type`.

Queues used:
- my-hello-world — fire-and-forget messages
- reply-back-to-me — request-reply messages

## Features
- Spring Boot auto-configuration for JMS (spring-boot-starter-artemis)
- Embedded Artemis server (artemis-server, artemis-jms-server)
- JSON TextMessage conversion via MappingJackson2MessageConverter
- Scheduled message production (every 2s)
- Request-reply pattern using JmsTemplate.sendAndReceive

## Prerequisites
- Java 8 or newer (pom.xml sets `<java.version>1.8</java.version>`)
- Maven Wrapper (included) or Maven 3.6+

Optional tools:
- IntelliJ IDEA for development & running directly

## Getting Started

### Build
Windows PowerShell / CMD:
- .\mvnw.cmd clean package

Unix-like shells:
- ./mvnw clean package

### Run (development)
Run with the Spring Boot Maven Plugin:
- Windows: .\mvnw.cmd spring-boot:run
- Unix: ./mvnw spring-boot:run

### Run (packaged JAR)
After building:
- java -jar target/jmsexample-0.0.1-SNAPSHOT.jar

The application starts an embedded Artemis broker (in-VM) and begins sending and receiving messages automatically.

## Configuration
No external configuration is required for the demo. Key defaults:
- Embedded Artemis journal: target/data/journal
- Persistence: disabled
- Security: disabled
- Transport: in-VM (no network port required)

Message conversion:
- Spring JMS MappingJackson2MessageConverter targets TEXT messages and sets `_type` header for type mapping.

Queue names (JmsConfig):
- MY_QUEUE = "my-hello-world"
- MY_SEND_RCV_QUEUE = "reply-back-to-me"

## How It Works
- Producer (HelloSender):
  - sendMessage(): Every 2 seconds, sends HelloWorldMessage(id=UUID, message="Hello World!") to `my-hello-world`.
  - sendandReceiveMessage(): Every 2 seconds, sends HelloWorldMessage(id=UUID, message="Hello") to `reply-back-to-me` using JmsTemplate.sendAndReceive and prints the reply body.

- Consumer (HelloMessageListener):
  - listen(): Subscribes to `my-hello-world` (no-op handler shown for simplicity, extend to process messages).
  - listenForHello(): Subscribes to `reply-back-to-me` and replies to JMSReplyTo with HelloWorldMessage(message="World!!").

- Embedded Broker (JmsexampleApplication):
  - Starts Artemis with in-VM acceptor `vm://0`, no persistence/security, then launches Spring Boot.

Example console output snippets:
- Sending Hello
- {"id":"<uuid>","message":"World!!","_type":"com.example.jmsexample.model.HelloWorldMessage"}

## Project Structure
- src/main/java/com/example/jmsexample
  - JmsexampleApplication.java — boots embedded Artemis and Spring
  - config/JmsConfig.java — JMS converter and queue names
  - config/TaskConfig.java — enables scheduling/async, provides TaskExecutor
  - model/HelloWorldMessage.java — simple DTO (UUID id, String message)
  - sender/HelloSender.java — scheduled producer and request-reply client
  - listener/HelloMessageListener.java — consumers, including request-reply responder

## Troubleshooting
- No external broker needed: Uses in-VM transport; no TCP port conflicts.
- ClassNotFound or Lombok errors: Ensure Lombok plugin is enabled in your IDE and annotation processing is on.
- Java version: Confirm `java -version` reports 1.8+.
- Clean build issues: Try `.\mvnw.cmd -U clean package` to force dependency updates.

## References
- Spring for JMS: https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms
- Spring Boot + Artemis: https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#messaging.jms
- ActiveMQ Artemis: https://activemq.apache.org/components/artemis/

---

Note: This repository is an educational example. Adapt error handling, logging, and security for production use.
