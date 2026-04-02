---
description: Build and deploy the URL Shortener to AWS
---

1. Run tests first
// turbo
2. Run `go test ./...`
3. Build the Lambda binary for Linux
// turbo
4. Run `GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build -tags lambda.norpc -o bootstrap cmd/api/main.go`
5. Package the deployment artifact
// turbo
6. Run `zip deployment.zip bootstrap`
7. Review Terraform plan
8. Run `cd terraform && terraform plan`
9. Confirm the plan looks correct before applying
10. Run `cd terraform && terraform apply -auto-approve`
11. Show deployed API URL
// turbo
12. Run `cd terraform && terraform output api_url`
