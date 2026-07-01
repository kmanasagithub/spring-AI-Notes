# ChatResponse vs ChatClientResponse in Spring AI

Spring AI provides two related response types: `ChatResponse` and `ChatClientResponse`. While they may seem similar, they serve different purposes.

- **`ChatResponse`** represents the **raw response from the AI model**.
- **`ChatClientResponse`** represents the **entire execution result of the `ChatClient`**, including the `ChatResponse` and additional client-side context.

---

# ChatResponse

`ChatResponse` is the response returned directly by the AI model.

It contains:

- The model's generated content
- Multiple generations (if applicable)
- Token usage
- Finish reason
- Response metadata from the model

Example:

```java
ChatResponse response = chatClient.prompt()
    .user("Tell me a joke")
    .call()
    .chatResponse();
```

You can inspect it like this:

```java
String text = response.getResult().getOutput().getText();

Usage usage = response.getMetadata().getUsage();

System.out.println(usage.getPromptTokens());
System.out.println(usage.getCompletionTokens());
System.out.println(usage.getTotalTokens());
```

### Conceptually

```text
AI Provider
     │
     ▼
ChatResponse
```

It represents **what the LLM returned**.

---

# ChatClientResponse

`ChatClientResponse` is a higher-level wrapper introduced by the `ChatClient`.

Instead of only containing the model response, it contains:

- the `ChatResponse`
- execution context
- advisor context
- information produced by Advisors
- other client-side metadata

Conceptually:

```text
ChatClientResponse
    ├── ChatResponse
    ├── Advisor Context
    ├── Execution Context
    └── Client Metadata
```

Example:

```java
ChatClientResponse response = chatClient.prompt()
    .user("Explain Spring AI")
    .call()
    .chatClientResponse();
```

Then you can access the underlying `ChatResponse` and other context:

```java
ChatResponse chatResponse = response.chatResponse();

// Advisor or execution context (API may vary by Spring AI version)
Map<String, Object> context = response.context();
```

> **Note:** The exact methods available on `ChatClientResponse` may vary depending on the Spring AI version.

---

# Why Does Spring AI Have Both?

Suppose your application uses a `QuestionAnswerAdvisor`.

The execution flow looks like this:

```text
User
  │
  ▼
ChatClient
  │
  ▼
Advisor
  │
  ▼
Vector Store
  │
  ▼
LLM
  │
  ▼
ChatResponse
  │
  ▼
ChatClientResponse
```

The advisor may retrieve documents such as:

```text
Document A
Document B
Document C
```

Those retrieved documents are **not part of the model response**.

`ChatResponse` only contains information returned by the LLM:

```text
Generated text
Token usage
Finish reason
Metadata
```

`ChatClientResponse` contains both the model response **and** additional execution information:

```text
Generated text
Token usage
Retrieved documents
Advisor state
Memory state
Execution context
```

---

# When Should You Use Each?

## Use `ChatResponse` when you only need the model output.

Typical use cases:

- Generated text
- Token usage
- Finish reason
- Response metadata

Example:

```java
ChatResponse response = chatClient.prompt()
    .user("Hello")
    .call()
    .chatResponse();
```

---

## Use `ChatClientResponse` when you're using advanced ChatClient features.

Examples include:

- Advisors
- Retrieval-Augmented Generation (RAG)
- Chat Memory
- Observability
- Execution context
- Custom advisor data

Example:

```java
ChatClientResponse response = chatClient.prompt()
    .user(question)
    .advisors(new QuestionAnswerAdvisor(vectorStore))
    .call()
    .chatClientResponse();
```

---

# Analogy

Imagine ordering food from a restaurant.

## ChatResponse

The meal itself:

```text
🍔 Burger
🍟 Fries
🥤 Drink
```

## ChatClientResponse

The complete order:

```text
Receipt
Customer details
Discount applied
Delivery information
Payment status

+
🍔 Burger
🍟 Fries
🥤 Drink
```

The meal (`ChatResponse`) is part of the order (`ChatClientResponse`), but the order contains additional information beyond the meal.

---

# Summary

| Feature | ChatResponse | ChatClientResponse |
|----------|--------------|--------------------|
| Model-generated text | ✅ | ✅ (via wrapped `ChatResponse`) |
| Token usage | ✅ | ✅ |
| Finish reason | ✅ | ✅ |
| Response metadata | ✅ | ✅ |
| Advisor context | ❌ | ✅ |
| RAG retrieval information | ❌ | ✅ |
| Chat memory context | ❌ | ✅ |
| Execution/client metadata | ❌ | ✅ |
| Represents raw LLM output | ✅ | ❌ (wraps it) |
| Represents the entire `ChatClient` execution | ❌ | ✅ |

---

# Key Takeaway

- **`ChatResponse`** is the **raw response returned by the language model**.
- **`ChatClientResponse`** is the **complete result of a `ChatClient` execution**, containing the `ChatResponse` along with advisor state, execution context, memory information, and other client-side metadata.

If you're simply generating text, `ChatResponse` is usually sufficient. If you're building applications that use RAG, memory, advisors, or other advanced Spring AI features, `ChatClientResponse` provides the additional context needed to inspect and work with the entire execution pipeline.
