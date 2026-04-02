---
description: Run all tests with coverage and race detection
---

// turbo-all

1. Run unit tests with coverage
2. Run `go test -v -cover -coverprofile=coverage.out ./...`
3. Run race condition detection
4. Run `go test -race ./...`
5. Print coverage summary
6. Run `go tool cover -func=coverage.out | tail -1`
