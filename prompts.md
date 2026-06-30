
# Spring AI: Prompt, Message Types, and ChatClient Summary

## 1. What is a Prompt?

A **Prompt** is a container that holds one or more messages. These messages are sent to the LLM (Large Language Model) in the order they are added.

A prompt does **not** represent a single message—it represents the entire conversation.

Example:

```java
Prompt prompt = new Prompt(List.of(
    new SystemMessage("You are a Java expert."),
    new UserMessage("Explain Dependency Injection.")
));
```

Conversation sent to the model:

```
System: You are a Java expert.
User: Explain Dependency Injection.
```

---

# 2. Message Types in Spring AI

Spring AI represents every conversation using different message types.

| Message Type | Purpose | Who Creates It? |
|--------------|---------|-----------------|
| **SystemMessage** | Defines the AI's behavior, personality, or instructions. | Developer |
| **UserMessage** | Represents the user's question or request. | User |
| **AssistantMessage** | Represents the AI's previous response. Used for conversation history. | AI |

---

## 3. SystemMessage

A **SystemMessage** tells the model **how it should behave**.

Examples:

- You are a Java expert.
- You are a professional chef.
- Answer in simple English.
- Respond only in JSON.

Example:

```java
SystemMessage systemMessage =
    new SystemMessage("You are an expert Java developer.");
```

Conversation:

```
System: You are an expert Java developer.
User: Explain Spring Boot.
```

---

## 4. UserMessage

A **UserMessage** represents the actual question or request from the user.

Example:

```java
UserMessage userMessage =
    new UserMessage("What is Spring AI?");
```

Conversation:

```
User: What is Spring AI?
```

---

## 5. AssistantMessage

An **AssistantMessage** represents a previous response from the AI.

It is mainly used when maintaining conversation history.

Example:

```java
AssistantMessage assistantMessage =
    new AssistantMessage("Spring AI is a framework for integrating AI into Spring applications.");
```

Conversation:

```
User: What is Spring AI?
Assistant: Spring AI is a framework for integrating AI into Spring applications.
User: Explain Prompt.
```

The previous assistant response provides context for the next answer.

---

# 6. Prompt(String) Defaults to UserMessage

This is one of the most important concepts.

When you write:

```java
Prompt prompt = new Prompt("You are an expert chef.");
```

Spring AI automatically converts it to:

```java
Prompt prompt = new Prompt(
    new UserMessage("You are an expert chef.")
);
```

### Why?

Because a plain `String` has no information about its role.

Spring AI cannot determine whether it should be:

- SystemMessage
- UserMessage
- AssistantMessage

Therefore, it assumes the most common role:

**UserMessage**

---

# 7. How to Create a System Prompt

If you want the text to act as an instruction instead of a user question, explicitly create a `SystemMessage`.

```java
Prompt prompt = new Prompt(
    new SystemMessage("You are an expert chef.")
);
```

Conversation:

```
System: You are an expert chef.
```

---

# 8. What Does chatClient.prompt(prompt) Do?

It starts the conversation using an existing Prompt.

Example:

```java
Prompt prompt = new Prompt(
    new SystemMessage("You are a Java expert.")
);

chatClient.prompt(prompt)
    .user("Explain Spring Boot.")
    .call();
```

Final conversation:

```
System: You are a Java expert.
User: Explain Spring Boot.
```

The existing prompt is preserved, and additional messages are appended.

---

# 9. What Does .system() Do?

`.system()` is a convenience method provided by `ChatClient`.

Example:

```java
chatClient.prompt()
    .system("You are a Java expert.")
    .user("Explain Spring Boot.")
    .call();
```

Internally, Spring AI creates:

```java
new SystemMessage("You are a Java expert.");
```

So,

```java
.system("...")
```

is simply shorthand for creating a `SystemMessage`.

---

# 10. What Does .user() Do?

`.user()` adds a `UserMessage` to the prompt.

Example:

```java
chatClient.prompt()
    .user("Explain Spring Boot.")
    .call();
```

Internally:

```java
new UserMessage("Explain Spring Boot.");
```

Conversation:

```
User: Explain Spring Boot.
```

---

# 11. What Does .assistant() Do?

`.assistant()` adds an `AssistantMessage`.

Example:

```java
chatClient.prompt()
    .assistant("Spring Boot is a Java framework.")
    .user("Can you explain it in detail?")
    .call();
```

Conversation:

```
Assistant: Spring Boot is a Java framework.
User: Can you explain it in detail?
```

This is useful for continuing conversations or supplying previous AI responses as context.

---

# 12. .system(), .user(), and .assistant() are Convenience Methods

These methods simply create the corresponding message objects.

| Builder Method | Internally Creates |
|----------------|--------------------|
| `.system()` | `SystemMessage` |
| `.user()` | `UserMessage` |
| `.assistant()` | `AssistantMessage` |

Example:

```java
chatClient.prompt()
    .system("You are a Java expert.")
    .user("Explain Streams.");
```

Internally equivalent to:

```java
Prompt prompt = new Prompt(List.of(
    new SystemMessage("You are a Java expert."),
    new UserMessage("Explain Streams.")
));

chatClient.prompt(prompt);
```

---

# 13. Manual Prompt vs Fluent API

### Manual Prompt

```java
Prompt prompt = new Prompt(List.of(
    new SystemMessage("You are a Java expert."),
    new UserMessage("Explain Spring AI.")
));

chatClient.prompt(prompt)
    .call();
```

---

### Fluent API

```java
chatClient.prompt()
    .system("You are a Java expert.")
    .user("Explain Spring AI.")
    .call();
```

Both produce the same conversation.

---

# 14. Conversation Example

```java
chatClient.prompt()
    .system("You are a Java expert.")
    .user("What is Spring Boot?")
    .assistant("Spring Boot simplifies Spring application development.")
    .user("Explain Auto Configuration.")
    .call();
```

Conversation sent to the LLM:

```
System:
You are a Java expert.

User:
What is Spring Boot?

Assistant:
Spring Boot simplifies Spring application development.

User:
Explain Auto Configuration.
```

The LLM sees the **entire conversation**, not just the latest message.

---

# 15. Key Takeaways

- A **Prompt** is a container for one or more messages.
- A conversation consists of **System**, **User**, and **Assistant** messages.
- `Prompt(String)` creates a **UserMessage** by default because a plain `String` has no role information.
- Use `SystemMessage` when creating a prompt manually and you want to define the AI's behavior.
- `.system()`, `.user()`, and `.assistant()` are convenience methods in the `ChatClient` fluent API.
- Internally:
  - `.system()` → `SystemMessage`
  - `.user()` → `UserMessage`
  - `.assistant()` → `AssistantMessage`
- `chatClient.prompt(prompt)` starts with an existing prompt and appends any additional messages added through the fluent API.
- The LLM always receives the **complete conversation**, including system instructions, user requests, and any assistant responses provided as context.

---

# Quick Revision

```
Prompt
│
├── SystemMessage
│     ↓
│   AI Instructions
│
├── UserMessage
│     ↓
│   User Question
│
└── AssistantMessage
      ↓
   Previous AI Response
```

```
Prompt(String)
        │
        ▼
UserMessage(String)
```

```
.system()     → SystemMessage
.user()       → UserMessage
.assistant()  → AssistantMessage
```

**Golden Rule:**

> **Prompt is the conversation. Messages define the roles within that conversation.** A plain `String` becomes a `UserMessage` by default because Spring AI cannot infer any other role.

