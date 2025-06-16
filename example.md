# Client Error Handling Logic (gRPC + API Gateway)

This section defines how the client should determine whether to treat a response as an error and how to parse the corresponding error information.

---

## ✅ Decision Flow

```plaintext
1. if gRPC status != OK:
       # Infrastructure-level error (C1–C4, S2)
       if trailer contains x-error-code:
           parse trailer x-error-* fields
       else:
           fallback to gRPC status for retry or default error handling
2. else (gRPC status == OK):
       # Application-level logic error possible (S1)
       if response body.success == false:
           parse error_code, error_message, error_details
       else:
           treat as success
```

---

## 🔁 Category-specific Behavior

| Category | gRPC Status | Error Source        | Client Handling Strategy                  |
|----------|-------------|---------------------|--------------------------------------------|
| **C1/C2**| ≠ OK        | No trailer          | Use gRPC status fallback, typically retry  |
| **C3/C4**| ≠ OK        | Trailer metadata    | Parse `x-error-code`, `x-error-message`    |
| **S1**   | OK          | Response body       | Check `success = false`, parse body errors |
| **S2**   | ≠ OK        | Trailer metadata    | Parse `x-error-code`, or fallback on status|

---

## 🧾 Client Implementation Snippet (Pseudo)

```python
if grpc_status != grpc.StatusCode.OK:
    try:
        error_code = trailer.get("x-error-code")
        error_message = trailer.get("x-error-message")
        origin = trailer.get("x-error-origin")
    except:
        # Use status only
        handle_status(grpc_status)
else:
    if not response.success:
        handle_app_error(response.error_code, response.error_message)
    else:
        return response.payload
```

---

## 💡 Recommended: Client-side Error Adapter

Encapsulate error parsing and handling into a shared client utility:

```python
class ApiError:
    def __init__(self, code, message, retryable, origin):
        ...

    def should_retry(self) -> bool:
        ...

    def get_user_message(self) -> str:
        ...
```

This keeps the logic maintainable and aligned with both API Gateway and Server behaviors.