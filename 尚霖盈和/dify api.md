# 1

```bash
curl -X POST 'http://192.168.0.108:8500/v1/chat-messages' \
--header 'Authorization: Bearer app-XpvudAk9YqdbDpTu9TxC8vYi' \
--header 'Content-Type: application/json' \
--data-raw '{
    "inputs": {},
    "query": "顾客想去旅游",
    "response_mode": "streaming",
    "conversation_id": "",
    "user": "abc-123",
    "files": [
      {
        "type": "image",
        "transfer_method": "remote_url",
        "url": "https://cloud.dify.ai/logo/logo-site.png"
      }
    ]
}'
```
# 2

```bash
curl -X POST 'http://192.168.0.108:8500/chat/M63u8R4EgIO0feai' \
--header 'Authorization: Bearer app-XpvudAk9YqdbDpTu9TxC8vYi' \
--header 'Content-Type: application/json' \
--data-raw '{
    "inputs": {},
    "query": "顾客想去旅游",
    "response_mode": "streaming",
    "conversation_id": "",
    "user": "abc-123",
    "files": [
      {
        "type": "image",
        "transfer_method": "remote_url",
        "url": "https://cloud.dify.ai/logo/logo-site.png"
      }
    ]
}'
```
# 3
```bash
curl -X POST 'http://192.168.0.108:8500/v1/workflows/run' \
--header 'Authorization: Bearer app-ipHfKvds86JSs9clet1iZiAQ' \
--header 'Content-Type: application/json' \
--data-raw '{
    "inputs": {
	    "query":"顾客想去旅游"
    },
    "response_mode": "streaming",
    "user": "abc-123"
}'
```

