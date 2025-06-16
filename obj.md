# 📦 Unified gRPC Response Object: `ApiResponse`

This object is used across all unary gRPC methods to standardize how responses and errors are communicated between the Server and the Client.

---

## 🧾 Protobuf Definition

```proto
import "google/protobuf/any.proto";

message ApiResponse {
  // Indicates whether the request was processed successfully
  bool success = 1;

  // Populated only if success = false (application-level error)
  string error_code = 2;        // e.g., INVALID_ARGUMENT, DB_NOT_FOUND
  string error_message = 3;     // Human-readable description
  string error_origin = 4;      // e.g., "server", "gateway", "validator"
  bool error_retryable = 5;     // Whether the error is retryable

  // Populated only if success = true
  google.protobuf.Any payload = 6;
}
```

---

## ✅ Semantic Behavior

| Field               | Description                                                                 |
|--------------------|-----------------------------------------------------------------------------|
| `success`          | `true` if the operation succeeded. If `false`, inspect error fields.        |
| `error_code`       | Internal or public-facing code representing the application error type.     |
| `error_message`    | Human-readable explanation of the failure.                                  |
| `error_origin`     | Logical source of the error (`"server"`, `"gateway"`, etc.).                |
| `error_retryable`  | Indicates whether retrying the operation might succeed.                     |
| `payload`          | Result data of the API call, present only if `success = true`.              |

---

## 📘 Example Usages

### Success Response

```json
{
  "success": true,
  "payload": {
    "@type": "type.googleapis.com/SomeResponseType",
    "result": "value"
  }
}
```

### Error Response (Application-level)

```json
{
  "success": false,
  "error_code": "DB_NOT_FOUND",
  "error_message": "No record found for the given ID",
  "error_origin": "server",
  "error_retryable": false
}
```

---

## 🎯 Design Rationale

- Encourages separation of infrastructure errors (via gRPC trailer) and application errors (in body).
- Enables consistent client logic for all server implementations.
- Works well in API Gateway environments with pass-through behavior.
- Supports observability and error categorization.

---

> 💡 This object is designed for **unary gRPC methods only**. Streaming support requires different handling.