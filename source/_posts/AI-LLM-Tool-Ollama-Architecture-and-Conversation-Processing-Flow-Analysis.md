---
title: 'AI LLM Tool Ollama: Architecture and Conversation Processing Flow Analysis'
date: 2024-11-03 11:02:38
categories:
- AI
tags:
- AI
---


- [Overview of Ollama](#overview-of-ollama)
- [Ollama Architecture](#ollama-architecture)
- [Ollama Storage Structure](#ollama-storage-structure)
- [Ollama Conversation Processing Flow](#ollama-conversation-processing-flow)
- [Conclusion](#conclusion)
- [References:](#references)

<a name="overview-of-ollama"></a>
## Overview of Ollama

`Ollama` is a convenient tool for running `LLM` (Large Language Models) quickly. With `Ollama`, users can easily interact with large language models without complex environment configurations.

This article will analyze the overall architecture of `Ollama` and detail the specific processing flow when users engage in conversations with `Ollama`.

<a name="ollama-architecture"></a>
## Ollama Architecture

![Ollama Architecture](images/AI-LLM-Tool-Ollama-Architecture-and-Conversation-Processing-Flow-Analysis/image-1.png)

`Ollama` employs a classic CS (Client-Server) architecture, where:

- **Client**: Interacts with the user via the command line.
- **Server**: Can be started via the command line, desktop application (based on the Electron framework), or Docker. Regardless of the startup method, the same executable file is ultimately invoked.
- **Communication**: Client and Server communicate using HTTP.

The `Ollama Server` consists of two core components:

- **`ollama-http-server`**: Handles interactions with the client.
- **`llama.cpp`**: Serves as the LLM inference engine, responsible for loading and running large language models, processing inference requests, and returning results.
- **Interaction**: `ollama-http-server` and `llama.cpp` also communicate via HTTP.

**Note**: `llama.cpp` is an independent open-source project that is cross-platform and hardware-friendly, capable of running on devices without GPUs, including Raspberry Pi.

<a name="ollama-storage-structure"></a>
## Ollama Storage Structure

![Ollama Storage Architecture](images/AI-LLM-Tool-Ollama-Architecture-and-Conversation-Processing-Flow-Analysis/image-2.png)

`Ollama` uses the default local storage folder path `$HOME/.ollama`. The file structure is as follows:

```
$HOME/.ollama/
├── blobs/
├── manifests/
├── logs/
│   └── server.log
├── history
├── id_ed25519
└── id_ed25519.pub
```

Files can be categorized into three types:

- **Log Files**: Includes the `history` file, which records user conversation inputs, and the `logs/server.log` server log file.
- **Key Files**: Includes the `id_ed25519` private key and `id_ed25519.pub` public key.
- **Model Files**: Includes `blobs` raw data files and `manifests` metadata files.

The metadata files, such as `models/manifests/registry.ollama.ai/library/llama3.2/latest`, contain the following content:

![llama3.2-latest](images/AI-LLM-Tool-Ollama-Architecture-and-Conversation-Processing-Flow-Analysis/image-3.png)

```json
{
  "digest": "sha256:abcdef1234567890",
  "size": 123456789,
  "annotations": {
    "org.opencontainers.image.title": "llama3.2",
    "org.opencontainers.image.version": "latest"
  }
}
```

As shown above, `manifests` files are in JSON format and draw inspiration from the OCI spec (Open Container Initiative Specification) in cloud-native and container domains. The `digest` field in `manifests` corresponds to `blobs`.

<a name="ollama-conversation-processing-flow"></a>
## Ollama Conversation Processing Flow

![Ollama Conversation Processing Flow](images/AI-LLM-Tool-Ollama-Architecture-and-Conversation-Processing-Flow-Analysis/image-4.png)

The general flow of a user conversation with `Ollama` is as follows:

1. **User Initiates Conversation**: The user executes the command `ollama run llama3.2` via the CLI to start a conversation (`llama3.2` is an open-source large language model; you can also use other LLMs).

2. **Preparation Phase**:
   - The CLI client sends an HTTP request to `ollama-http-server` to fetch model information. The server attempts to read the local `manifests` metadata files. If not found, it responds with a 404 not found.
   - When the model does not exist, the CLI client requests `ollama-http-server` to pull the model from the remote repository to the local storage.
   - The CLI client then requests the model information again.

3. **Interactive Conversation Phase**:
   - The CLI sends an empty message `/api/generate` request to `ollama-http-server`, which internally handles some channel processing (using Go's channels).
   - If the model information contains `messages`, they are printed. Users can save the current model and session conversation history as a new model, and the conversation history is saved as `messages`.
   - The conversation officially begins: The CLI calls the `/api/chat` interface to request `ollama-http-server`, which relies on the `llama.cpp` engine to load the model and perform inference (`llama.cpp` also serves as an HTTP server). `ollama-http-server` first sends a `/health` request to `llama.cpp` to confirm its health status, then sends a `/completion` request to obtain the conversation response, and finally returns it to the CLI for display.

Through the above steps, `Ollama` completes the interaction between the user and the large language model.

<a name="conclusion"></a>
## Conclusion

`Ollama` integrates the `llama.cpp` inference engine and further encapsulates it, making complex LLM technology accessible. It provides developers and technical personnel with an efficient and flexible tool, effectively supporting large language model inference and interaction in various application scenarios.

<a name="references"></a>
## References:

- • [https://github.com/ollama/ollama](https://github.com/ollama/ollama)
- • [https://github.com/ggerganov/llama.cpp](https://github.com/ggerganov/llama.cpp)