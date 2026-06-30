
# Spring AI – Chat Options & OllamaChatOptions Guide

---

## 1. What Are Chat Options?

Chat Options are **configuration parameters used to control how Large Language Models (LLMs) generate responses** in Spring AI.

They define model behavior such as:
- Creativity level
- Response length
- Sampling strategy
- Stopping conditions

👉 In simple terms:
> Chat Options are the “control knobs” for an AI model.

---

## 2. Why Chat Options Are Needed

Different LLM providers (OpenAI, Ollama, Anthropic, etc.):

- Use different parameter names
- Support different features
- Evolve independently

So Spring AI uses Chat Options to:
- Standardize configuration
- Provide type safety
- Support multiple providers
- Keep API flexible and extensible

---

## 3. Common Chat Parameters

| Parameter        | Description | Example |
|----------------|-------------|----------|
| temperature     | Controls randomness (creative vs deterministic) | 0.0 – 1.0 |
| maxTokens       | Maximum response length | 100 – 4000+ |
| topK            | Limits candidate tokens | 40 |
| topP            | Nucleus sampling threshold | 0.0 – 1.0 |
| stopSequences   | Stops generation at specific text | ["END"] |

---

## 4. What is `OllamaChatOptions`?

`OllamaChatOptions` is a **provider-specific implementation of Chat Options** used with Ollama models.

It maps Java configuration to Ollama API parameters such as:
- `temperature`
- `numPredict`
- `topK`
- `topP`
- `stopSequences`

---

## 5. Why Not `.options(temp, maxTokens)`?

This approach is not used because:

- Different providers use different names  
  (OpenAI → `maxTokens`, Ollama → `numPredict`)
- Not all models support the same parameters
- Hard to maintain and extend
- Would require many overloaded methods

👉 So Spring AI uses a **single object-based approach**

---

## 6. Builder Pattern in `OllamaChatOptions`

The Builder Pattern is used to construct complex objects step by step.

### Example:

```java
OllamaChatOptions options = OllamaChatOptions.builder()
        .temperature(0.7)
        .numPredict(200)
        .build();
````

---

## 7. Why Builder Pattern is Used

* Handles many optional fields
* Improves readability
* Avoids long constructors
* Easy to extend in future versions

---

## 8. Ways to Use Chat Options

### 8.1 Request-Level (Highest Priority)

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

### 8.2 Bean-Level Default Options

```java
@Bean
public ChatClient chatClient(ChatClient.Builder builder) {
    return builder
            .defaultOptions(OllamaChatOptions.builder()
                    .temperature(0.3)
                    .numPredict(200)
                    .build())
            .build();
}
```

---

### 8.3 application.properties

```properties
spring.ai.ollama.chat.options.temperature=0.5
spring.ai.ollama.chat.options.num-predict=200
```

---

### 8.4 Reusable Option Profiles

```java
public class ChatProfiles {

    public static final OllamaChatOptions CREATIVE =
            OllamaChatOptions.builder()
                    .temperature(0.9)
                    .build();

    public static final OllamaChatOptions STRICT =
            OllamaChatOptions.builder()
                    .temperature(0.2)
                    .build();
}
```

---

## 9. Configuration Priority Order

Lowest → Highest:

1. application.properties
2. Bean-level defaultOptions
3. Request-level options

👉 Request-level always overrides others

---

## 10. Best Practices

* Low temperature (0.2–0.4) → factual answers
* High temperature (0.7–1.0) → creative output
* Always set maxTokens / numPredict
* Use stopSequences for structured outputs
* Prefer builder pattern for clarity

---

## 11. Provider Differences

| Provider  | Key Parameter Differences |
| --------- | ------------------------- |
| OpenAI    | maxTokens, topP           |
| Anthropic | maxTokensToSample         |
| Gemini    | safety + tuning controls  |
| Ollama    | numPredict                |

---

## 12. Quick Cheat Sheet

* temperature = 0.2 → precise output
* temperature = 0.8 → creative output
* maxTokens = 1000 → long response
* stopSequences = ["\n"] → stop at newline

---

## 13. Key Takeaway

Chat Options provide a **flexible, provider-agnostic way to control LLM behavior**.

`OllamaChatOptions` is a **structured, type-safe builder-based implementation** that makes Spring AI scalable and extensible.

```

---

## 🔥 What I fixed in your version
- Removed duplicate sections (you had “What is Chat Options?” twice)
- Fixed broken markdown flow
- Unified explanation style
- Cleaned repetition
- Improved hierarchy (proper learning order)
- Made it GitHub-ready

---

