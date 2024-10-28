---
title: Building a ChatGPT-like System
date: 2024-10-28 15:42:35
categories:
- ChatGPT
tags:
- ChatGPT
---

- [Introduction](#introduction)
- [System Overview](#system-overview)
- [Session and Historical Message Management](#session-and-historical-message-management)
  - [Communication Protocols and APIs](#communication-protocols-and-apis)
  - [Standardized Prompt Format](#standardized-prompt-format)
  - [Managing Historical Messages](#managing-historical-messages)
- [System Expansion](#system-expansion)
  - [Adding Cache Mechanism](#adding-cache-mechanism)
  - [Scope of Caching](#scope-of-caching)
  - [Elastic Scalability](#elastic-scalability)
- [Production Readiness](#production-readiness)
- [Conclusion](#conclusion)
- [References](#references)

---

<a name="introduction"></a>
## Introduction

**ChatGPT** is a conversational system based on **Large Language Models (LLMs)**. This article introduces the process of building a system similar to ChatGPT, covering everything from the **model**, **inference engine**, to the overall **architecture**.

---

<a name="system-overview"></a>
## System Overview

Our primary focus will be on the core of the conversation system.

**System Diagram:**
```
                +--------------+
                |    Web       |
                |  Interface   |
                +--------------+
                       |
                       |
                       v
                +--------------+
                |   Server     |
                +--------------+
                       |
                       |
                       v
         +--------------------------+
         |   Inference Runtime      |
         |   (LLM Inference Engine) |
         +--------------------------+
                       |
                       |
                       v
                +--------------+
                |   Response   |
                +--------------+
```

![Core Dialogue Flow](images/Building-a-ChatGPT-like-System/image-1.png)

In this basic system framework:
- The **web interface** interacts with users, 
- The **server** receives user requests and forwards them to the **inference runtime** (inference engine), 
- The **inference runtime** loads the **LLM** to generate a response, returning it to the user.

---

<a name="session-and-historical-message-management"></a>
## Session and Historical Message Management

However, the above system has a critical flaw: it lacks **session** and **historical message management** for users. Common **inference runtimes** are typically **stateless** and do not directly support multi-turn conversations with historical context storage, meaning the system "forgets" previous exchanges within a single session.

To address this, we need some essential modifications.

**Enhanced System Diagram:**
```
                +--------------+
                |    Web       |
                |  Interface   |
                +--------------+
                       |
                       |
                       v
                +--------------+
                |   Server     |
                +--------------+
                       |
                       |
                       v
         +--------------------------+
         |   Inference Runtime      |
         |   (LLM Inference Engine) |
         +--------------------------+
                       |
                       |
                       v
                +--------------+
                |  Database    |
                +--------------+
```

![Enhanced System with Session ahdn History Management](images/Building-a-ChatGPT-like-System/image-2.png)

In this setup, we introduce a **database component** to store user sessions and historical messages.

### Communication Protocols and APIs
For communication, the **web interface** and **server** can use **HTTP** protocols. Basic REST APIs might include:

- **`POST /chat`**: Initiate a new session.
- **`POST /chat/:chatID/completion`**: Continue a conversation within an existing session.
- **`GET /chats`**: Retrieve a list of sessions.
- **`DELETE /chat/:chatID`**: Delete a specific session.

The **server** persists data in the database, structuring conversation messages with essential fields like `userID`, `chatID`, `userMessage`, and `assistantMessage`.

### Standardized Prompt Format

When sending data to the **inference runtime**, the server uses a standardized **prompt format**:

```json
[
    {
        "role": "system",
        "content": "You are a helpful assistant."
    },
    {
        "role": "user",
        "content": "Hello!"
    },
    {
        "role": "assistant",
        "content": "Hello there, how may I assist you today?"
    },
    {
        "role": "user",
        "content": "How are you?"
    }
]
```

The `role` field represents different participants: `system` defines the conversation context, `user` represents user input, and `assistant` represents the model’s output.

### Managing Historical Messages

There are several ways to handle historical messages:

- **Direct Prompt Filling**: Add historical messages in the prompt as `user` and `assistant` entries. This works well for short conversations.
- **Dynamic Context Adjustment**: Discard earlier messages to fit within the LLM’s token limit.
- **Message Summarization**: Use the inference engine to summarize conversation history, compressing information before filling the prompt.

With session and historical message management, we establish a basic system framework.

---

<a name="system-expansion"></a>
## System Expansion

As the user base grows, the system will require further scalability.

### Adding Cache Mechanism

One approach to scalability is implementing a **cache** to avoid repeated inference processes. The cache **key-value** pair would be the user's question and the AI's response. A cache hit would depend on whether the semantic content of the question is similar. For example, if two questions have different wording but the same meaning, the system should return the same response.

**Enhanced Caching System Diagram:**
```
                +--------------+
                |    Web       |
                |  Interface   |
                +--------------+
                       |
                       |
                       v
                +--------------+
                |   Server     |
                +--------------+
                       |
                       |
                       v
         +--------------------------+
         |   Inference Runtime      |
         |   (LLM Inference Engine) |
         +--------------------------+
                       |
                       |
                       v
         +--------------------------+
         |       Embedding          |
         |    Runtime & Model       |
         +--------------------------+
                       |
                       |
                       v
         +--------------------------+
         | Vector Storage & Search  |
         +--------------------------+
```

![Cache with Vector Embeddings for Similar Query Detection](images/Building-a-ChatGPT-like-System/image-3.png)


Apart from the **cache module**, we add **embedding runtime** and a **text embedding model** to convert text into **vectors**. If two vectors are close, their text content is semantically similar. **Vector storage and search** modules are used to store and retrieve vectors.

### Scope of Caching

Caching can be scoped to:
- **Single-user cache**: Caching value is low as users rarely repeat the same question.
- **Global cache**: Allows reuse across users, but risks exposing sensitive data.

After data analysis and validation, decide if cache implementation is beneficial.

### Elastic Scalability

**Elastic scaling** is another key to handling high concurrency. The **server** is stateless, which allows for easy horizontal scaling.

**Scalability System Diagram:**
```
                +--------------+
                |    Web       |
                |  Interface   |
                +--------------+
                       |
                       |
                       v
                +--------------+
                |   Gateway    |
                |  (Load Bal.) |
                +--------------+
                       |
                       |
                       v
         +--------------------------+
         |   Server Instances       |
         +--------------------------+
                       |
                       |
                       v
         +--------------------------+
         |   Inference Runtimes     |
         +--------------------------+
```

![Load Balanced System Architecture with Elastic Scaling](images/Building-a-ChatGPT-like-System/image-4.png)

Here, we introduce a **gateway** for load balancing. The **inference runtime** is also stateless and supports elastic scaling, but it requires more hardware resources and may respond slower than the server. To maintain stability under peak load, add a **message queue (MQ)** and change request handling to asynchronous, improving system resilience.

---

<a name="production-readiness"></a>
## Production Readiness

The system architecture covers the basic logic, but to achieve **production-readiness**, further steps are necessary:

- **Technology Selection**: Choose a database (e.g., PostgreSQL or MongoDB) and inference engine (e.g., `llama.cpp`, `HuggingFace Transformers`, `vLLM`) and map logical components to physical components.
- **Observability**: Integrate **logging**, **tracing**, **metrics**, and set up monitoring and alerting.
- **CI/CD and Deployment Environment**: Configure automation pipelines for continuous integration and deployment, and select the appropriate environment (e.g., **cloud platforms** or **Kubernetes**).

---

<a name="conclusion"></a>
## Conclusion

This article provides an overview of constructing a ChatGPT-like system, covering the core conversation flow and expanding to include session management and historical message storage. We discussed strategies for system expansion and laid out steps toward production readiness.

---

<a name="references"></a>
## References

- [llama.cpp GitHub](https://github.com/ggerganov/llama.cpp)
- [OpenAI API Reference](https://platform.openai.com/docs/api-reference/chat/create)
```

This enhanced Markdown document covers each requested update, including detailed technical sections, diagrams, keywords, and sections for expansion and production readiness.