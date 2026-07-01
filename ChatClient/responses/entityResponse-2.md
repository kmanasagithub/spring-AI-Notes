
## `EntityParamSpec`

All three `entity()` overloads accept an optional `Consumer<EntityParamSpec>`. This allows you to customize how Spring AI generates and validates structured output before mapping it to your Java object.

For example:

```java
.entity(ActorFilms.class, spec -> {
    // Configure entity mapping
})
```

### What is `spec`?

The `spec` parameter is simply a lambda variable representing an instance of `EntityParamSpec`. It is **not** a special Java keyword—you can name it anything.

For example, these are all equivalent:

```java
.entity(
    ActorFilms.class,
    spec -> spec.useProviderStructuredOutput()
);
```

```java
.entity(
    ActorFilms.class,
    config -> config.useProviderStructuredOutput()
);
```

```java
.entity(
    ActorFilms.class,
    entitySpec -> entitySpec.useProviderStructuredOutput()
);
```

The `entity()` method internally has a signature similar to:

```java
<T> T entity(
    Class<T> type,
    Consumer<EntityParamSpec> consumer
)
```

Spring AI creates an `EntityParamSpec` object and passes it to your lambda. Inside the lambda, you configure the options you want.

For example:

```java
spec -> spec
    .useProviderStructuredOutput()
    .validateSchema()
```

is conceptually equivalent to:

```java
EntityParamSpec spec = new EntityParamSpec();

spec.useProviderStructuredOutput();
spec.validateSchema();
```

The difference is that Spring AI creates and manages the `EntityParamSpec` instance for you.

`EntityParamSpec` currently supports two independent features:

- **Provider-Native Structured Output**
- **Schema Validation with Retry**

These features can be used individually or together.

---

## 1. Provider-Native Structured Output

By default, when you use `entity()`, Spring AI generates a JSON schema for your Java class and appends it to the prompt as text instructions.

For example, the prompt sent to the model may effectively include instructions like:

```text
Return the response as JSON matching this schema...
```

This approach works with virtually all language models because it relies on prompt engineering.

### Why is this needed?

Large Language Models are not guaranteed to follow instructions perfectly. Even when asked to return JSON, they may sometimes return plain text.

For example, instead of:

```json
{
  "actor": "Tom Hanks",
  "movies": [
    "Forrest Gump",
    "Cast Away"
  ]
}
```

the model might respond with:

```text
Tom Hanks starred in Forrest Gump, Cast Away, Saving Private Ryan, and many other films.
```

Since `entity()` expects structured JSON, Spring AI cannot deserialize plain text into your `ActorFilms` object.

### Using `useProviderStructuredOutput()`

Some AI providers (such as OpenAI models that support Structured Outputs) allow Spring AI to send the JSON schema directly as part of the API request instead of embedding it in the prompt.

You can enable this behavior as follows:

```java
ActorFilms actorFilms = chatClient.prompt()
    .user("Generate the filmography for a random actor.")
    .call()
    .entity(
        ActorFilms.class,
        spec -> spec.useProviderStructuredOutput()
    );
```

When enabled:

- The JSON schema is sent directly to the AI provider.
- The provider instructs the model to strictly follow the schema.
- The model is much more likely to return valid JSON instead of plain text.
- This results in more reliable mapping to your Java object.

### Why isn't it enabled by default?

Native structured output is not enabled by default because support varies across providers and models.

Some limitations include:

- Some Ollama models running in reasoning/thinking mode may return plain text instead of JSON.
- OpenAI currently does not support top-level JSON array schemas.

Because compatibility is inconsistent, Spring AI defaults to the prompt-based approach.

### Enabling it globally

Instead of enabling it for a single call, you can enable native structured output globally using an advisor parameter:

```java
ActorFilms actorFilms = chatClient.prompt()
    .advisors(AdvisorParams.ENABLE_NATIVE_STRUCTURED_OUTPUT)
    .user("Generate the filmography for a random actor.")
    .call()
    .entity(ActorFilms.class);
```

---

## 2. Schema Validation with Retry

Even if the model returns JSON, the JSON might not exactly match your Java class.

For example:

Expected:

```json
{
  "actor": "Tom Hanks",
  "movies": ["Forrest Gump", "Cast Away"]
}
```

Model returns:

```json
{
  "name": "Tom Hanks",
  "films": "Forrest Gump"
}
```

Although the response is valid JSON, it does **not** match the expected schema.

Calling `validateSchema()` instructs Spring AI to validate the generated JSON before attempting to deserialize it.

```java
ActorFilms actorFilms = chatClient.prompt()
    .user("Generate the filmography for a random actor.")
    .call()
    .entity(
        ActorFilms.class,
        spec -> spec.validateSchema()
    );
```

### What happens if validation fails?

Spring AI automatically:

1. Detects the schema validation error.
2. Appends the validation error message to the prompt.
3. Sends the request to the model again.
4. Repeats the process until the response matches the schema or the retry limit is reached.

Internally, this feature uses the `StructuredOutputValidationAdvisor`.

By default, Spring AI retries up to **3 times** (`maxRepeatAttempts = 3`).

---

## Using Both Features Together

You can enable both native structured output and schema validation:

```java
ActorFilms actorFilms = chatClient.prompt()
    .user("Generate the filmography for a random actor.")
    .call()
    .entity(
        ActorFilms.class,
        spec -> spec
            .useProviderStructuredOutput()
            .validateSchema()
    );
```

In this configuration:

- The provider enforces the JSON schema during generation.
- Spring AI validates the generated JSON.
- If validation fails, Spring AI automatically retries the request.

This provides the highest level of reliability for structured output.

---
