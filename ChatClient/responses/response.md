# Returning an Entity (`entity()`)

In many applications, you don't want to work with the raw AI response. Instead, you want the response to be converted directly into a Java object. Spring AI provides the `entity()` method for this purpose.

Rather than returning a `ChatResponse` or `ChatClientResponse`, the `entity()` method maps the AI model's output into a Java type.

---

## Example

Suppose you have the following Java record:

```java
record ActorFilms(String actor, List<String> movies) {}
```

You can ask the model to generate data and have Spring AI automatically deserialize the response into the record:

```java
ActorFilms actorFilms = chatClient.prompt()
    .user("Generate the filmography for a random actor.")
    .call()
    .entity(ActorFilms.class);
```

Instead of receiving JSON as a `String`, you directly receive a populated `ActorFilms` object.

Conceptually:

```text
AI Provider
     │
     ▼
ChatResponse (JSON/Text)
     │
     ▼
Spring AI
     │
     ▼
ActorFilms
```

---

# How `entity()` Works

Internally, Spring AI performs the following steps:

1. Sends your prompt to the LLM.
2. Instructs the model to return output matching your Java type.
3. Receives the response as a `ChatResponse`.
4. Deserializes the structured output into your Java class.
5. Returns the Java object.

So even though you receive an `ActorFilms` instance, a `ChatResponse` still exists internally—it is simply hidden from you.

---

# Generic Types

`entity()` also supports generic types using `ParameterizedTypeReference`.

Example:

```java
List<ActorFilms> actorFilms = chatClient.prompt()
    .user("Generate the filmography of Tom Hanks and Bill Murray.")
    .call()
    .entity(new ParameterizedTypeReference<List<ActorFilms>>() {});
```

This allows Spring AI to correctly deserialize collections and other generic types.

---

# Provider-Native Structured Output

By default, Spring AI appends the JSON schema as text instructions in the prompt.

If your AI provider supports native structured output (for example, OpenAI's Structured Outputs), you can enable it:

```java
ActorFilms actorFilms = chatClient.prompt()
    .user("Generate the filmography for a random actor.")
    .call()
    .entity(
        ActorFilms.class,
        spec -> spec.useProviderStructuredOutput()
    );
```

In this mode, the JSON schema is sent directly to the provider instead of being included in the prompt, resulting in more reliable structured responses.

---

# When Should You Use `entity()`?

Use `entity()` when:

- You need a strongly typed Java object.
- You don't need token usage or response metadata.
- Your application consumes structured data.
- You want to avoid manually parsing JSON.

Examples include:

- Customer records
- Product information
- Weather reports
- Movie details
- Configuration objects
- Lists of entities

---

# Comparison

| Return Type | Returns | Best Used When |
|-------------|----------|----------------|
| `content()` | `String` | You only need plain text. |
| `entity()` | Java object (`T`) | You want structured data mapped to a Java class. |
| `chatResponse()` | `ChatResponse` | You need the model response, token usage, or metadata. |
| `chatClientResponse()` | `ChatClientResponse` | You need the complete execution context, including advisors, RAG, memory, and the underlying `ChatResponse`. |

---

# Relationship Between the Return Types

```text
                ChatClient
                     │
                     ▼
                 AI Provider
                     │
                     ▼
               ChatResponse
                /         \
               /           \
        content()       entity()
           │                │
           ▼                ▼
        String         Java Object (T)

                     │
                     ▼
            ChatClientResponse
      (wraps ChatResponse + execution context)
```

---

# Key Takeaway

- **`content()`** returns plain text.
- **`entity()`** returns a Java object by mapping the model's structured output.
- **`ChatResponse`** exposes the raw AI response along with metadata such as token usage.
- **`ChatClientResponse`** wraps the `ChatResponse` and includes additional execution details such as advisor context, RAG information, and chat memory.

Choose the return type based on the level of abstraction your application needs:
- **Plain text** → `content()`
- **Structured Java object** → `entity()`
- **Model response with metadata** → `chatResponse()`
- **Complete ChatClient execution context** → `chatClientResponse()`
