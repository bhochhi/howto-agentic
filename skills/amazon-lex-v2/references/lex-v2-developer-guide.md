# Amazon Lex V2 Developer Guide Reference

## API Overview

### Modeling APIs (`lexv2-models`)

| Operation | Purpose |
|-----------|---------|
| `CreateBot` | Create a new bot |
| `CreateBotLocale` | Add a language/locale |
| `CreateIntent` | Add an intent to a locale |
| `CreateSlot` | Add a slot to an intent |
| `CreateSlotType` | Define a custom slot type |
| `BuildBotLocale` | Train the NLU model for a locale |
| `CreateBotVersion` | Snapshot the DRAFT into an immutable version |
| `CreateBotAlias` | Create a named pointer to a version |
| `UpdateBotAlias` | Change version or Lambda association |
| `ListBotAliases` / `ListBotVersions` | Enumerate aliases/versions |
| `DeleteBot` | Delete bot and all child resources |

### Runtime APIs (`lexv2-runtime`)

| Operation | Purpose |
|-----------|---------|
| `RecognizeText` | Send text, get bot response |
| `RecognizeUtterance` | Send audio or text, get audio or text response |
| `StartConversation` | Bidirectional streaming (WebSocket) |
| `PutSession` | Set session state programmatically |
| `GetSession` | Retrieve current session state |
| `DeleteSession` | End a session |

## Built-in Slot Types

| Slot Type | Examples |
|-----------|---------|
| `AMAZON.AlphaNumeric` | ABC123 |
| `AMAZON.City` | Seattle, New York |
| `AMAZON.Country` | United States, Japan |
| `AMAZON.Date` | tomorrow, 2025-03-15, next Friday |
| `AMAZON.Duration` | 5 minutes, 2 hours |
| `AMAZON.EmailAddress` | user@example.com |
| `AMAZON.FirstName` | John, Maria |
| `AMAZON.LastName` | Smith, García |
| `AMAZON.Number` | 42, one hundred |
| `AMAZON.Percentage` | 50%, twenty percent |
| `AMAZON.PhoneNumber` | 206-555-1234 |
| `AMAZON.State` | Washington, California |
| `AMAZON.StreetAddress` | 123 Main St |
| `AMAZON.Time` | 3 PM, noon, 14:30 |

## Custom Slot Types

### Resolution Strategies

| Strategy | Behavior |
|----------|----------|
| `ORIGINAL_VALUE` | Accept any value; use enumerated values for training only |
| `TOP_RESOLUTION` | Resolve to the closest matching enumerated value |

### Composite Slot Types
Combine multiple slots into a single structured slot:
```json
{
  "name": "FullAddress",
  "slotTypeValues": [],
  "compositeSlotTypeSetting": {
    "subSlots": [
      { "name": "Street", "slotTypeId": "AMAZON.StreetAddress" },
      { "name": "City", "slotTypeId": "AMAZON.City" },
      { "name": "State", "slotTypeId": "AMAZON.State" }
    ]
  }
}
```

## Conversation Logs

Enable on a bot alias to capture text/audio logs:

```json
{
  "textLogSettings": [{
    "enabled": true,
    "destination": {
      "cloudWatch": {
        "cloudWatchLogGroupArn": "arn:aws:logs:us-east-1:123456789012:log-group:/aws/lex/HotelBot"
      }
    }
  }],
  "audioLogSettings": [{
    "enabled": true,
    "destination": {
      "s3Bucket": {
        "s3BucketArn": "arn:aws:s3:::my-lex-audio-logs",
        "logPrefix": "hotel-bot/"
      }
    }
  }]
}
```

## IAM Permissions

### Bot Service Role
The bot needs a service role with:
- `lex:*` (managed by the service)
- `polly:SynthesizeSpeech` (if voice enabled)
- `lambda:InvokeFunction` (for code hooks — granted via resource policy)
- `cloudwatch:PutMetricData` (metrics)

### Lambda Resource Policy
Allow Lex to invoke the function:
```json
{
  "Effect": "Allow",
  "Principal": { "Service": "lexv2.amazonaws.com" },
  "Action": "lambda:InvokeFunction",
  "Resource": "arn:aws:lambda:us-east-1:123456789012:function:LexFulfillment",
  "Condition": {
    "ArnLike": {
      "AWS:SourceArn": "arn:aws:lex:us-east-1:123456789012:bot-alias/*"
    }
  }
}
```

## Limits (Default)

| Resource | Limit |
|----------|-------|
| Bots per account | 100 |
| Intents per locale | 200 |
| Slots per intent | 100 |
| Slot types per locale | 250 |
| Sample utterances per intent | 1,500 |
| Synonyms per slot type value | 10,000 |
| Custom vocabulary phrases per locale | 500 |
| Session timeout | 5 min – 24 hours |
| Input text length | 1,024 characters |
