# SAM Examples & Templates

## Project Structure

```
project/
├── cmd/
│   ├── get-user/
│   │   └── main.go
│   ├── create-user/
│   │   └── main.go
│   └── process-queue/
│       └── main.go
├── internal/
│   ├── domain/
│   ├── handler/
│   └── repository/
├── events/
│   └── get-user.json
├── statemachine/
│   └── workflow.asl.json
├── template.yaml
├── samconfig.toml
├── Makefile
└── README.md
```

## Makefile for SAM

```makefile
.PHONY: build deploy test local clean

build:
	sam build

deploy: build
	sam deploy

deploy-prod: build
	sam deploy --config-env prod

local:
	sam local start-api --port 3000

invoke:
	sam local invoke $(FUNCTION) --event events/$(EVENT).json

logs:
	sam logs --name $(FUNCTION) --stack-name $(STACK) --tail

sync:
	sam sync --stack-name $(STACK) --watch

clean:
	rm -rf .aws-sam

test:
	go test ./...
```

## Sample Event Files

### API Gateway HTTP API Event
```json
{
  "version": "2.0",
  "routeKey": "GET /users/{id}",
  "rawPath": "/users/123",
  "pathParameters": {
    "id": "123"
  },
  "headers": {
    "content-type": "application/json"
  },
  "requestContext": {
    "http": {
      "method": "GET",
      "path": "/users/123"
    }
  }
}
```

### SQS Event
```json
{
  "Records": [
    {
      "messageId": "msg-123",
      "body": "{\"userId\": \"123\", \"action\": \"process\"}",
      "attributes": {
        "ApproximateReceiveCount": "1"
      }
    }
  ]
}
```
