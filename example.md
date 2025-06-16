1. if gRPC status != OK:
       # Infrastructure-level error (C1–C4, S2)
       if trailer 包含 x-error-code:
           parse trailer 中的 x-error-* 欄位
       else:
           fallback：用 gRPC status 作 retry 或通用處理
2. else (gRPC status == OK):
       # 代表呼叫成功，但仍可能是 S1 應用邏輯錯誤
       if response body.success == false:
           parse error_code, error_message, error_details
       else:
           treat as success