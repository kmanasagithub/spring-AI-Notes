

# 🧠 Spring AI – Response Handling (Complete Deep Notes)

These notes explain **Spring AI response types in depth**, including internal flow, structure, usage, and best practices.

---

# 📌 1. What is a Response in Spring AI?

In Spring AI, every request to a model goes through:

```java
chatClient.prompt()
    .user("input")
    .call();
````

The response is the **AI-generated output + metadata + optional structured mapping**.

---

## 🔁 Internal Flow

```text
User Input
   ↓
Prompt Builder
   ↓
Chat Model (OpenAI / Ollama / etc.)
   ↓
ChatResponse
   ↓
AssistantMessage
   ↓
Final Output (String / POJO / List / Stream)
```

---

# 💬 2. Simple Text Response (.content)

## ✔ Code

```java
String response = chatClient.prompt()
        .user("What is AI?")
        .call()
        .content();
```

## 📌 What happens internally

* Model returns full response
* Spring AI extracts only `textContent`

## 📌 Output

```text
AI is a field of computer science...
```

## 📌 Limitations

* No metadata
* No structure
* No validation

## ✅ Use when

* Chatbots
* FAQs
* Basic Q&A apps

---

# 🤖 3. AssistantMessage Response

## ✔ Code

```java
AssistantMessage message = chatClient.prompt()
        .user("Hello")
        .call()
        .chatResponse()
        .getResult()
        .getOutput();
```

---

## 📌 Structure

```text
AssistantMessage
 ├── messageType      → ASSISTANT
 ├── textContent      → actual AI response
 ├── toolCalls        → function calls (if any)
 ├── metadata         → extra model info
```

---

## 📌 Example

```text
textContent:
"AI stands for Artificial Intelligence..."
```

---

## 📌 Extract text

```java
String text = message.getTextContent();
```

---

## 📌 When toolCalls appear

If AI decides to call a function:

```json
{
  "toolCalls": [
    {
      "name": "getWeather",
      "args": { "city": "Bhimavaram" }
    }
  ]
}
```

---

## ✅ Use when

* AI agents
* Function calling
* Debugging responses

---

# 📦 4. ChatResponse (FULL RESPONSE)

## ✔ Code

```java
ChatResponse response = chatClient.prompt()
        .user("Explain AI")
        .call()
        .chatResponse();
```

---

## 📌 Full internal structure

```text
ChatResponse
 ├── result
 │     ├── output → AssistantMessage
 │     ├── metadata
 ├── usage
 │     ├── input tokens
 │     ├── output tokens
 ├── model
 ├── finishReason
 ├── id
 ├── timestamp
```

---

## 📌 Why it is important

This gives **full control over AI interaction**.

---

## 📌 Use cases

* Logging AI usage cost
* Debugging bad outputs
* Performance monitoring
* Analytics dashboards

---

# 🧩 5. Entity Response (POJO Mapping)

## ✔ Code

```java
Recipe recipe = chatClient.prompt()
        .user("Give a vegetarian recipe")
        .call()
        .entity(Recipe.class);
```

---

## 📌 How it works internally

```text
AI Output → JSON → Jackson Mapper → Java Object
```

---

## 📌 Example JSON from AI

```json
{
  "name": "Paneer Curry",
  "ingredients": ["paneer", "tomato", "spices"],
  "description": "Spicy curry",
  "duration": "30 min"
}
```

---

## 📌 Java class mapping

```java
public class Recipe {
    private String name;
    private List<String> ingredients;
    private String description;
    private String duration;
}
```

---

## ⚠ Important rules

* Field names must match JSON keys
* Use lowercase fields
* Avoid complex nested mismatches

---

## ✅ Use when

* REST APIs
* Structured AI output
* Backend services

---

# 📚 6. List Response (Multiple Objects)

## ✔ Code

```java
List<Recipe> recipes = chatClient.prompt()
        .user("Give 3 recipes")
        .call()
        .entity(new ParameterizedTypeReference<List<Recipe>>() {});
```

---

## 📌 Why ParameterizedTypeReference?

Because Java erases generic types at runtime:

```text
List<Recipe> → becomes List (type erased)
```

So Spring AI needs explicit type info.

---

## 📌 Example output

```json
[
  {
    "name": "Pasta",
    "ingredients": ["noodles", "sauce"]
  },
  {
    "name": "Rice Bowl",
    "ingredients": ["rice", "vegetables"]
  }
]
```

---

## ❌ Common mistake

```java
.entity(List.class)  // WRONG
```

---

## ✅ Use when

* Recommendations
* Search results
* Multiple suggestions

---

# ⚡ 7. Streaming Response

## ✔ Code

```java
chatClient.prompt()
        .user("Tell a story")
        .stream()
        .content();
```

---

## 📌 How it works

```text
Token 1 → "Once"
Token 2 → " upon"
Token 3 → " a time"
...
```

---

## 📌 Advantages

* Real-time UI experience
* Faster perceived response
* Lower waiting time

---

## ✅ Use when

* ChatGPT-like UI
* Live typing animation
* Long responses

---

# 🧾 8. JSON Mode (Strict Output Control)

## ✔ Instruction

```java
.system("Return only valid JSON")
```

---

## 📌 Why needed

LLMs naturally return text → not structured data.

So we force:

```text
ONLY JSON
NO explanation
NO markdown
```

---

## 📌 Example output

```json
{
  "name": "Burger",
  "ingredients": ["bun", "patty"]
}
```

---

## ⚠ Benefits

* Safe mapping to POJOs
* No parsing errors
* Stable APIs

---

# 🔍 9. Tool Calling Response (AI Agents)

## ✔ Code

```java
message.getToolCalls();
```

---

## 📌 What is tool calling?

AI decides:

```text
"I need external data → call a function"
```

---

## 📌 Example tool call

```json
{
  "name": "getWeather",
  "arguments": {
    "city": "Bhimavaram"
  }
}
```

---

## 📌 Use cases

* Weather APIs
* Database queries
* External services
* AI automation agents

---

# 📊 10. Complete Response Types Summary

| Type              | Method             | Output        |
| ----------------- | ------------------ | ------------- |
| Simple Text       | `.content()`       | String        |
| Assistant Message | `.getOutput()`     | Object        |
| Full Response     | `.chatResponse()`  | ChatResponse  |
| Entity            | `.entity(Class)`   | POJO          |
| List Entity       | `.entity(TypeRef)` | List<POJO>    |
| Streaming         | `.stream()`        | Live text     |
| Tool Calls        | `getToolCalls()`   | Function data |

---

# 🚀 11. Real World Usage Patterns

## 💬 Chatbot

```java
.content()
```

## 🍲 Recipe API

```java
.entity(Recipe.class)
```

## 📦 Multiple results

```java
List<T>
```

## 📊 Monitoring system

```java
ChatResponse
```

## ⚡ UI streaming

```java
.stream()
```

## 🤖 AI Agent system

```java
toolCalls
```

---

# 🧠 12. Final Concept

Spring AI does NOT change the model output.

It only **wraps the same AI response in different formats**:

* Simple → text only
* Structured → POJO
* Multiple → List
* Debug → metadata
* Live → stream
* Automation → tool calls

---

# 🔥 Key Insight

👉 Spring AI = **One response, many representations**

---

