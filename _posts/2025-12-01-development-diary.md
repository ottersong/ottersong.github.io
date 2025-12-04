---
layout: single
title: "Development Weekly Diary"
date: "2025-12-01 18:00:00 +0900"
last_modified_at: "2025-12-01 18:00:00 +0900"
categories: ["weekly_diary"]
---

<div style="text-align: center;">
  <h2><strong>Weekly Development Log</strong></h2>
</div>

<br/>
📆
**2025** 
**12 · 04 · THU**


---
## 🔌 Client Proxy란?

NestJS 마이크로서비스 간에 **메시지 기반 통신(TCP, Redis, NATS, Kafka, RabbitMQ 등)** 을 사용할 때 요청을 보내는 도구이다.

HTTP 서버끼리 REST로 통신하는 방식이 아닌,  
NestJS 자체의 **Microservices 메시징 시스템**을 사용할 때 등장한다.

<br/>

### 📡 서비스 간 흐름 구조

```txt
API Gateway (REST)
        ↓
Payment Microservice
        ↓
User Microservice
        ↓
Notification Microservice
```

API Gateway는 각 마이크로서비스에 요청을 전달해야 하고,  
이때 HTTP 대신 **메시지 기반 통신(ClientProxy)** 을 사용하는 구조다.

<br/>

---

## ⚙️ ClientProxy 동작 방식

ClientProxy는 두 가지 방식으로 메시지를 전송한다.

### 1) `send()` — RPC 요청/응답 (Request → Response)

요청 후 **응답을 기다리는 방식**
```ts
this.client.send('get_user', userId)
```

---

### 2) `emit()` — 이벤트 발행 (Fire-and-forget)

응답 없이 **이벤트만 전송**
```ts
this.client.emit('user_created', userData)
```

<br/>

---

## 🏗 ClientProxy 생성 방법

### 1) ClientsModule.register()로 설정

```ts
constructor(
    @Inject('USER_SERVICE') private client: ClientProxy,
) {}
```

### 2) 예시: Redis Transport

```ts
ClientsModule.register([
    {
        name: 'USER_SERVICE',
        transport: Transport.REDIS,
        options: {
            host: 'localhost',
            port: 6379,
        },
    },
])
```

<br/>

---

## 🧪 실 사용 예시

### 📍 Gateway → User Service: 유저 정보 가져오기
```ts
const user = await this.userClient
    .send({ cmd: 'get-user' }, { id: 10 })
    .toPromise();
```

### 📍 이벤트 발행
```ts
this.userClient.emit(
    { cmd: 'user-updated' },
    { id: 10, name: '현승' }
);
```

<br/>

---

## 🚀 ClientProxy를 사용하는 이유

```md
NestJS는 마이크로서비스를 단순한 HTTP 서버가 아닌
“메시지 기반 서비스”로 구성하기 위한 기능을 지원한다.
```

### 주요 장점

- 서비스 간 결합도 감소(Decoupling)
- 비동기 메시징 기반
- 트래픽 비동기 처리
- Redis, Kafka, NATS 등 브로커 사용 가능
- 마이크로서비스 확장성 증가
- 장애 허용성 향상

<br/>

---

## 💡 언제 사용하나?

일반적으로 **같은 도메인이 아닌 다른 도메인** 간 통신할 때 사용  
(ex. `curation ↔ spot`, `gateway ↔ payment`)

---