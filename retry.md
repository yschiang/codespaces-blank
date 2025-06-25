# gRPC Gateway Retry & Error Handling Specification

## 🎯 Client Behavior

- The client performs **automatic retries** by default.
- Each request must carry a unique `TraceId` in gRPC metadata for observability.
- No cross-retry transaction ID is currently supported (future enhancement).
- The client determines **retry eligibility** based on:
  - gRPC status codes (e.g., `UNAVAILABLE`, `DEADLINE_EXCEEDED`)
  - `x-retryable` flag in gRPC trailers (if available)

---

## 🧭 API Gateway Behavior

- The gateway operates in **proxy mode**:
  - Forwards requests and responses transparently
  - Does **not parse or interpret** the response body
  - Adds standard gRPC trailers when catching infrastructure-level errors

- The gateway handles and wraps errors during:
  - Request decoding or plugin processing (e.g., AuthN failure)
  - Upstream connection and request forwarding
  - Upstream response relay

**When errors occur**, the gateway:
- Sets a gRPC status code (e.g., `INTERNAL`, `UNAVAILABLE`, `NOT_FOUND`)
- Adds the following metadata in trailers:
  - `x-error-code`: a structured internal error code
  - `x-error-message`: human-readable message
  - `x-retryable`: `"true"` or `"false"`
  - `x-trace-id`: from request or generated

---

## 🔁 Retry Behavior

- **The client, not the gateway**, decides whether to retry.
- Gateway **does not deduplicate** retried requests.
- Services behind the gateway must implement **idempotency** safeguards to handle retries correctly.

---

## 🔎 Observability

- Every request must include a `TraceId` in gRPC metadata.
- The same `TraceId` is included in all responses and errors.
- Currently, **there is no transaction ID that spans across retries**. Future enhancement may introduce `TxnId`.

---

## 📄 Sample Metadata

### Request Metadata:
```plaintext
metadata:
  trace-id: "abcd-1234-efgh"