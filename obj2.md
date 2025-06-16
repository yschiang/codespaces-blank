# 🚦 Stripe-style gRPC Error Response Design

This document defines a Stripe-style unified error structure for gRPC APIs using a single client and multiple server implementations. The goal is to ensure all errors can be accurately interpreted and handled by the client.

---

## 📦 Protobuf Definitions

### `ApiError`

```proto
message ApiError {
  string type = 1;         // e.g. "invalid_request_error", "internal_error"
  string code = 2;         // e.g. "INVALID_ARGUMENT", "TOOL_LOCKED"
  string message = 3;      // Human-readable description
  string field = 4;        // (Optional) Name of the field or parameter
  string doc_url = 5;      // (Optional) Link to relevant documentation
  string origin = 6;       // "server", "gateway"
  bool retryable = 7;      // Whether retrying might succeed
}
```

---

### `ApiResponse`

```proto
import "google/protobuf/any.proto";

message ApiResponse {
  bool success = 1;
  ApiError error = 2;                  // Present only when success = false
  google.protobuf.Any payload = 3;     // Present only when success = true
}
```

---

## ✅ Success Example

```json
{
  "success": true,
  "payload": {
    "@type": "type.googleapis.com/ToolStatus",
    "status": "available"
  }
}
```

---

## ❌ Error Example

```json
{
  "success": false,
  "error": {
    "type": "invalid_request_error",
    "code": "TOOL_LOCKED",
    "message": "The tool is currently locked for maintenance.",
    "field": "tool_id",
    "doc_url": "https://docs.yourapi.dev/errors#tool_locked",
    "origin": "server",
    "retryable": false
  }
}
```

---

## 🎯 Design Rationale

| Field        | Description |
|--------------|-------------|
| `type`       | A broad category of the error (helps classification). |
| `code`       | Specific error code, often used for logic branches. |
| `message`    | A human-readable error description. |
| `field`      | Indicates which parameter or input caused the issue. |
| `doc_url`    | Optional link to developer documentation. |
| `origin`     | Identifies if the error originated from server or gateway. |
| `retryable`  | Indicates if retrying the request is likely to help. |

---

## 💡 Notes

- This error model works best with **unary gRPC** requests.
- For non-OK gRPC status (S2), the API Gateway can also echo `ApiError` fields as trailer metadata.
- Encourages clean, structured error handling across diverse backend implementations.