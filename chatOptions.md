
# Spring AI Chat Options & OllamaChatOptions Builder Pattern

## 1. What is Spring AI Chat Options?

In Spring AI, **Chat Options** are configuration settings that control how a Large Language Model (LLM) generates responses.

These options define **model behavior**, such as:
- How creative or random the response should be
- Maximum length of response
- Sampling strategy
- Model-specific parameters

In simple terms:

> Chat Options are the “control knobs” for an AI model.

---

## 2. What is `OllamaChatOptions`?

`OllamaChatOptions` is a **provider-specific configuration class** used when working with the **Ollama LLM integration in Spring AI**.

It is used to define parameters that are sent to the Ollama backend when generating chat responses.

It acts as a **data container for model settings**, such as:
- temperature
- numPredict (max tokens equivalent in Ollama)
- topK / topP
- stop sequences
- model overrides

---

## 3. Why do we need Chat Options?

Different LLM providers (OpenAI, Ollama, Anthropic, etc.):
- Use different parameter names
- Support different features
- Evolve independently

So Spring AI uses Chat Options to:
- Standardize configuration handling in Java
- Provide type safety
- Support multiple providers easily
- Avoid hardcoding request parameters

---

## 4. Why not use simple method like `.options(temp, maxTokens)`?

This approach is not used because:

- Providers use different parameter names  
  - OpenAI → `maxTokens`  
  - Ollama → `numPredict`
- Models support different capabilities
- Hard to maintain and extend
- Would require many overloaded methods

So instead, Spring AI uses a **single object-based approach**.

---

## 5. What is Builder Pattern in `OllamaChatOptions`?

The Builder Pattern is a **design pattern used to construct complex objects step by step**.

In `OllamaChatOptions`, it helps manage many optional parameters safely.

### Example:

```java
OllamaChatOptions options = OllamaChatOptions.builder()
        .temperature(0.7)
        .numPredict(200)
        .build();
````

---

## 6. Why Builder Pattern is used?

### ✔ Handles many optional fields

* temperature
* topK
* topP
* numPredict
* stop sequences

### ✔ Improves readability

Code is easier to understand than constructors with many parameters.

### ✔ Flexible for future updates

New fields can be added without breaking existing code.

---

## 7. Usage in Spring AI ChatClient

### Request-level options (highest priority)

```java
String response = chatClient.prompt()
        .options(OllamaChatOptions.builder()
                .temperature(0.5)
                .numPredict(150)
                .build())
        .user("Hello")
        .call()
        .content();
```

---

## 8. Bean-level configuration (default options)

```java
@Bean
public ChatClient chatClient(ChatClient.Builder builder) {
    return builder
            .defaultOptions(OllamaChatOptions.builder()
                    .temperature(0.3)
                    .build())
            .build();
}
```

---

## 9. Configuration Priority Order

Lowest → Highest priority:

1. application.properties / system.properties
2. Bean-level default options
3. Request-level options

👉 Request-level options always override others

---

## 10. Key Takeaway

* Chat Options define **how the LLM behaves**
* `OllamaChatOptions` is a **provider-specific implementation**
* Builder pattern provides **flexibility, readability, and scalability**
* Spring AI uses object-based configuration to support multiple LLM providers

---

## 11. Simple Analogy

* Chat Options = control panel of AI
* Builder = filling a configuration form step by step
* Model = machine that follows those settings

