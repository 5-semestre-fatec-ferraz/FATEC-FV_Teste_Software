---
adr: 1
title: Definição da Arquitetura de Software em Java, Stack Tecnológico Core e Estratégia de Testes Exaustivos de Login
status: Approved
date: 2026-10-10
authors:
  - Josias Wattrelos
  - Antigravity AI
impacted_components:
  - directory: src/main/java/com/betaengine/
  - directory: src/test/java/com/betaengine/
  - file: pom.xml (ou build.gradle.kts)
  - file: application.yml
  - directory: docs/architecture/
rules:
  architectural_pattern: "Clean Architecture (Hexagonal / Ports & Adapters) + DDD Modular"
  web_engine: "Spring Boot 3.4+ com Tomcat Embutido (suporte a Undertow/Netty)"
  reverse_proxy: "NGINX (Reverse Proxy / TLS Termination) ou API Gateway"
  runtime_language: "Java 21 LTS (com Virtual Threads / Project Loom ativo)"
  http_framework: "Spring Web MVC / Spring Security 6.x"
  database_rdbms: "MySQL 8.0 / PostgreSQL 16+ via Spring Data JPA / Hibernate (HikariCP)"
  cache_primary: "Redis 7.x via Spring Data Redis (com fallback para Caffeine Cache em memória)"
  messaging_primary: "RabbitMQ 3.x via Spring AMQP (com fallback para Spring Event Publisher / DB Queue)"
  compliance_standards:
    - Jakarta EE 10 (JPA, Bean Validation, Servlet 6.0)
    - Google Java Style Guide / Checkstyle
    - RFC 6749 / RFC 7519 (OAuth2 / JWT)
    - OWASP Top 10 Security Guidelines
---

# ADR 0001: Definição da Arquitetura de Software em Java para Ambiente de Testes Exaustivo de Login

## 1. Status
**Proposta** (2026-10-10)

---

## 2. Contexto e Declaração do Problema

O módulo de autenticação e segurança requer um ambiente de testes e execução que seja:
1. **Robusto e Exaustivo para Testes de Login:** Suporte nativo a testes de unidade puros (sem boot de framework), testes com dublês (Mocks/Stubs), testes de contrato e testes de integração com subida rápida de containers temporários (via Testcontainers).
2. **Heterogeneidade de Infraestrutura:** Capacidade de rodar tanto em instâncias de nuvem escaláveis (Docker / Kubernetes na AWS, GCP) consumindo Redis e RabbitMQ, quanto em ambientes de desenvolvimento/CI locais e servidores monolíticos de baixo custo (usando banco H2/MySQL local e cache em memória via Caffeine).
3. **Independência de Framework no Domínio:** Evitar acoplamento do *Core Business Logic* com bibliotecas de persistência (JPA/Hibernate) ou de transporte HTTP (Spring Web), garantindo que as regras de autenticação, políticas de senha e bloqueio de tentativas sejam 100% testáveis isoladamente.

---

## 3. Fatores de Decisão (Drivers Arquiteturais)

* **Concorrência e Baixo Footprint:** Utilização de **Java 21 Virtual Threads (Project Loom)** no Tomcat embutido, viabilizando milhares de requisições de login simultâneas com modelo síncrono simples e baixo consumo de threads do SO.
* **Desacoplamento e Testabilidade (Clean Architecture / Hexagonal):** 
  - Regras de domínio e casos de uso testáveis em milissegundos com JUnit 5 + Mockito, sem necessidade de carregar o `ApplicationContext` do Spring.
* **Segurança em Profundidade (Defense-in-Depth):** Implementação de cadeia de filtros via Spring Security (`SecurityFilterChain`), blindando contra ataques de força bruta, CSRF, Session Fixation e Credential Stuffing.
* **Resiliência e Fallback:** Operar com Redis/RabbitMQ em produção, mas degradar suavemente para soluções locais em ambientes de teste/desenvolvimento.

---

## 4. Decisão

Decidimos estabelecer a seguinte arquitetura e matriz de tecnologias em Java:

