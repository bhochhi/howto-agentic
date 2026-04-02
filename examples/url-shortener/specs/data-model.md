# URL Shortener — Data Model Specification

## Table: urls

| Attribute | Type | Key | Description |
|---|---|---|---|
| `short_code` | String | Partition Key | Unique short code (e.g., "abc1234" or "spring-sale") |
| `original_url` | String | — | The destination URL |
| `created_at` | String (ISO 8601) | — | Creation timestamp |
| `created_by` | String | — | Creator identifier (IP or future user ID) |
| `click_count` | Number | — | Atomic counter for total clicks |

### Access Patterns

| Pattern | Operation | Key Condition |
|---|---|---|
| Get URL by code | GetItem | PK = short_code |
| Create URL | PutItem | PK = short_code (ConditionExpression: attribute_not_exists) |
| Increment clicks | UpdateItem | PK = short_code (SET click_count = click_count + 1) |

---

## Table: clicks

| Attribute | Type | Key | Description |
|---|---|---|---|
| `short_code` | String | Partition Key | Which short URL was clicked |
| `timestamp` | String (ISO 8601) | Sort Key | When the click happened |
| `referrer` | String | — | HTTP Referer header |
| `user_agent` | String | — | Client user agent |
| `ip_country` | String | — | Geo from IP (if available) |
| `ttl` | Number | — | DynamoDB TTL epoch (90 days from creation) |

### Access Patterns

| Pattern | Operation | Key Condition |
|---|---|---|
| Record click | PutItem | PK = short_code, SK = timestamp |
| Get clicks for URL (date range) | Query | PK = short_code, SK BETWEEN start AND end |
| Auto-delete old clicks | TTL | DynamoDB automatic deletion after 90 days |
