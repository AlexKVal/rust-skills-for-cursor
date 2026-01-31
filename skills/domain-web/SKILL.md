---
name: domain-web
description: "Use when building web services. Keywords: web server, HTTP, REST API, axum, actix, handler, middleware, state management"
user-invocable: false
---

# Web Domain

> **Layer 3: Domain Constraints**

## Domain Constraints → Rust Implications

| Domain Rule | Why | Rust |
|-------------|-----|------|
| Stateless HTTP | No request-local globals | State in extractors |
| Concurrency | Many connections | Async, Send + Sync |
| Latency SLA | Fast response | Don't block |

## Critical Rules

- **Handlers must not block** → async/await, spawn_blocking for CPU work
- **Shared state must be thread-safe** → `Arc<T>`, `Arc<RwLock<T>>`
- **Rc in state is wrong** → Use `Arc`

## Axum Pattern

```rust
async fn handler(
    State(db): State<Arc<DbPool>>,
    Json(payload): Json<CreateUser>,
) -> Result<Json<User>, AppError> {
    let user = db.create_user(&payload).await?;
    Ok(Json(user))
}

impl IntoResponse for AppError {
    fn into_response(self) -> Response {
        let (status, msg) = match self {
            Self::NotFound => (StatusCode::NOT_FOUND, "Not found"),
            Self::Internal(_) => (StatusCode::INTERNAL_SERVER_ERROR, "Error"),
        };
        (status, Json(json!({"error": msg}))).into_response()
    }
}
```

## Framework Comparison

| Framework | Style | Best For |
|-----------|-------|----------|
| axum | Functional, tower | Modern APIs |
| actix-web | Actor-based | High performance |

## Key Crates

| Purpose | Crate |
|---------|-------|
| HTTP server | axum, actix-web |
| HTTP client | reqwest |
| Auth/JWT | jsonwebtoken |
| Database | sqlx, diesel |
| Middleware | tower |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Blocking in handler | spawn_blocking |
| Rc in state | Arc |
| No validation | Type-safe extractors |
