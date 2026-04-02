# URL Shortener — Requirements Specification

## Overview
A serverless URL shortener for internal use by the marketing team.
Custom branded domain, click analytics, and API-first design.

## Functional Requirements

### FR-1: Create Short URL
- **Endpoint**: POST /shorten
- **Input**: Original URL (required), custom slug (optional)
- **Behavior**:
  - If custom slug provided and available → use it
  - If custom slug taken → return 409 Conflict
  - If no slug → generate random 7-character alphanumeric code
  - Validate URL format (must be valid HTTP/HTTPS URL)
- **Output**: Short URL, original URL, created timestamp, short code

### FR-2: Redirect
- **Endpoint**: GET /{code}
- **Behavior**:
  - Look up code in database
  - If found → 301 redirect to original URL
  - If not found → 404 Not Found
  - Record click event asynchronously (do not block redirect)

### FR-3: Click Analytics
- **Endpoint**: GET /{code}/stats
- **Output**: Total clicks, clicks per day (last 30 days), top referrers

## Non-Functional Requirements

| Requirement | Target |
|---|---|
| Redirect latency (P99) | < 50ms |
| Create latency (P99) | < 200ms |
| Availability | 99.9% |
| Max URL length | 2048 characters |
| Short code format | `[a-zA-Z0-9]{7}` |
| Analytics retention | 90 days |
| Rate limiting | 100 creates/min, 10K redirects/min |

## Out of Scope (V1)
- User authentication (internal tool, VPN-only)
- URL expiration / TTL
- QR code generation
- Bulk import
