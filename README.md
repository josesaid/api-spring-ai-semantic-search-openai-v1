# API Spring AI — Semantic Search (OpenAI v1)

**Project page:** https://josesaid.github.io/api-spring-ai-semantic-search-openai-v1/

A small Spring Boot demo that shows how to use [Spring AI](https://spring.io/projects/spring-ai) with OpenAI to:

- Generate **text embeddings** from strings.
- Compute **cosine similarity** between embeddings to compare how semantically close two pieces of text are (semantic search).
- Expose a simple **chat REST endpoint** backed by an OpenAI chat model.

It is part of a training series on Java + Spring AI (`03 - Semantic Search`).

## What this project demonstrates

### 1. Embeddings & cosine similarity (`SemanticSearchRunner`)

[`SemanticSearchRunner`](src/main/java/com/mx/development/jose/said/runner/SemanticSearchRunner.java) is a `CommandLineRunner` bean that runs automatically on application startup. It uses Spring AI's `EmbeddingModel` to turn text into vectors and then measures similarity between them with a hand-written cosine similarity function.

The active example (`embedding4`) runs a tiny semantic search:

1. Takes a query string (e.g. `"Pinsa"`, a misspelling of "Pizza").
2. Embeds a list of candidate documents (`Vehículo`, `Coche`, `Carro`, `Bicicleta`, `Avión`, `Pizza`).
3. Embeds the query and compares it against every document using cosine similarity.
4. Prints the documents ranked by similarity score, highest first.

This shows the core idea behind semantic search: instead of exact keyword matching, texts are compared by the *meaning* captured in their embedding vectors, so even a misspelled or related query can surface the right result.

The class also keeps several other example runners (currently disabled via commented-out `@Bean`) that were used to explore the same concept step by step:

- `embedding`: generates a single embedding and prints its dimension and first values.
- `embeddingCompare`: compares a few handpicked sentence pairs (e.g. technical vs. unrelated topics).
- `embedding2` / `embedding3`: build a full similarity matrix across a list of words/phrases.

You can enable any of these by uncommenting the corresponding `@Bean` annotation.

### 2. Chat endpoint (`ChatController` / `ChatService`)

A minimal REST API wraps Spring AI's `ChatClient` to send a free-form question to the OpenAI chat model and return its answer:

- **Endpoint:** `GET /api/v1/chat-service/chat?question=<your question>`
- **Controller:** [`ChatController`](src/main/java/com/mx/development/jose/said/controller/ChatController.java)
- **Service:** [`ChatServiceImpl`](src/main/java/com/mx/development/jose/said/service/ChatServiceImpl.java) builds a `ChatClient` from the auto-configured `ChatClient.Builder` and forwards the question to it.

## Tech stack

- **Java 24**
- **Spring Boot 4.1.0**
- **Spring AI 2.0.0** (`spring-ai-starter-model-openai`)
- **Maven** (with the Maven Wrapper, `mvnw`)
- OpenAI as the model provider (chat + embeddings)

## Project structure

```
src/main/java/.../com/mx/development/jose/said/
├── ApiSpringAiDemoV1Application.java   # Spring Boot entry point
├── controller/
│   └── ChatController.java             # REST endpoint for chat
├── dto/
│   └── Resultado.java                  # (text, score) pair used in similarity results
├── runner/
│   └── SemanticSearchRunner.java       # Embedding & cosine similarity examples
└── service/
    ├── ChatService.java                # Chat service contract
    └── ChatServiceImpl.java            # Chat service implementation using ChatClient

src/main/resources/
├── application.yml                     # Base config (port, active profile, logging)
└── application-openai.yml              # OpenAI API key configuration (profile: openai)
```

## Configuration

The application runs with the `openai` Spring profile active by default (see `application.yml`). The OpenAI API key is read from an environment variable:

```yaml
# application-openai.yml
spring:
  ai:
    openai:
      api-key: ${API_KEY}
```

You need a valid OpenAI API key — see https://platform.openai.com/api-keys.

The server runs on port **8088** by default (`server.port` in `application.yml`).

## Running the project

1. Export your OpenAI API key:

   ```bash
   export API_KEY=sk-...
   ```

2. Run the application with the Maven Wrapper:

   ```bash
   ./mvnw spring-boot:run
   ```

   On startup, `SemanticSearchRunner` will print the embedding/similarity results to the console.

3. Call the chat endpoint:

   ```bash
   curl "http://localhost:8088/api/v1/chat-service/chat?question=What%20is%20Spring%20AI%3F"
   ```

## Notes

- Logging for `org.springframework.ai` and `org.springframework.web.client` is set to `DEBUG` in `application.yml`, which is useful for inspecting the requests/responses exchanged with OpenAI while experimenting.
- Several runner methods in `SemanticSearchRunner` are intentionally commented out (`//@Bean`); they are alternative examples kept for learning purposes and can be toggled on individually.
