# Overview

|Port | Protocol | Transport | Description |
|-----|----------|-----------|-------------|
| 8501 | HTTP | TCP | Restulf API |
| 8500 | gRPC | TCP | gRPC API |

## Images
Images need to be 223x223 pixel in size to work

## APIs

Obtain the IP of the service
```
export IP_ADDRESS="localhost"
```

### Is OK (Running)
```bash
curl http://${IP_ADDRESS}:8501/v1/models/ssd-mobilenet-v1-1/versions/1
```

## Bash
curl -H "Content-Type: application/json" -X POST -d '{"data": {"keys": [[1.0], [2.0]], "features": [[10, 10, 10, 8, 6, 1, 8, 9, 1], [6, 2, 1, 1, 1, 1, 7, 1, 1]]}}' http://127.0.0.1:8500