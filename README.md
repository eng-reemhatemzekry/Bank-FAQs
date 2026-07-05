# Bank-FAQs
# 🏦 Egypt Bank AI Assistant

A multilingual, privacy-aware banking FAQ chatbot built with **LangGraph**, **Gemini**, **FAISS**, and **HuggingFace embeddings**. It answers customer questions about accounts, cards, loans, and transfers using Retrieval-Augmented Generation (RAG) grounded strictly in a curated FAQ knowledge base.

## Key Features

This project is built around three priorities that matter most for a real-world banking assistant:

### Multilingual support
Users can ask questions in Arabic, English, or any other language. The pipeline:
1. Detects the input language (`langdetect`)
2. Translates the question to English for retrieval (the knowledge base and embedding model are English-based)
3. Retrieves and reasons over the FAQ context in English
4. Translates the final answer back into the user's original language

### Client data security
A dedicated guard node runs **before** any retrieval, classification, or LLM call:
- Detects sensitive data in the raw message using regex — national IDs, card/account numbers, phone numbers, passwords, PINs, and OTPs
- If sensitive data is found, the assistant **refuses to process the request** and redirects the user to official bank channels — the sensitive text is never sent to the LLM or stored
- Anything printed or logged elsewhere (e.g. the CLI chat loop) uses a **masked** version of the input, so raw sensitive data never appears in logs or notebook output

> This is a demonstration-grade guard, not a compliance solution. Pair it with your organization's actual data-handling, PCI-DSS, and KYC policies before production use.

### Intent classification & output quality
- **Structured intent classification**: instead of brittle keyword matching, the LLM returns strict JSON (`{"intent": "..."}`) to classify each message as `greeting`, `banking`, or `other` — robust across phrasing and languages
- **Groundedness check**: after generating an answer, a second LLM call verifies the answer is actually supported by the retrieved FAQ context. If it isn't, the answer is replaced with an honest fallback instead of risking a hallucinated response
