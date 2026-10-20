# Questions & Initial Discovery

## 1. Scope & Core Requireme    nts
* **What is the most fundamental feature set required for a minimal Redis implementation?**
  * Core commands: `PING`, `SET`, `GET`, `DEL`, and `EXISTS`.
  * Protocol: Redis Serialization Protocol (RESP) or a simplified text-based protocol.
* **Should the implementation support persistence (RDB/AOF) or remain strictly in-memory?**
  * Strictly in-memory to keep the baseline design minimal and focus on networking/data handling.
* **Is concurrency required for the basic version?**
  * A single-threaded event loop (or basic thread-per-connection / asyncio model) is sufficient to mimic Redis's single-threaded core model.

## 2. Technical Stack & AI Workflow
* **Language selection:** Python (using `asyncio`) or Node.js/Go for simplicity and clear socket handling. Python `asyncio` is chosen for readable protocol parsing.
* **AI Assistance Strategy:**
  * Prompting for protocol parser generation.
  * Prompting for key-value storage class with lookup/deletion methods.
  * Generating test cases and client connection scripts.