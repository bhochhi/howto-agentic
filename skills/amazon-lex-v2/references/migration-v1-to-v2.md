# Migration Guide: Amazon Lex V1 to V2

## Overview

Amazon Lex V2 is a complete rewrite with a different API surface, resource model, and Lambda event format. There is no automatic migration — you must recreate bots in V2.

## Migration Steps

### 1. Export V1 Bot

```bash
aws lex-models get-export \
  --name HotelBot \
  --version '$LATEST' \
  --resource-type BOT \
  --export-type LEX
```

### 2. Map Resources

| V1 Resource | V2 Equivalent | Notes |
|-------------|---------------|-------|
| Bot | Bot + Bot Locale | V2 separates language from bot |
| Intent | Intent (under locale) | Same concept; different API |
| Slot Type | Slot Type (under locale) | Same concept |
| Alias | Bot Alias → Bot Version | V2 aliases point to immutable versions |
| `$LATEST` | `DRAFT` | Mutable working copy |
| Built-in intents | Same names | Signature format: `AMAZON.FallbackIntent` |

### 3. Recreate in V2

Use the console **migration tool** (limited) or recreate via API/IaC:

1. Create the bot with `CreateBot`
2. Create locale(s) with `CreateBotLocale`
3. Create custom slot types with `CreateSlotType`
4. Create intents with `CreateIntent` — include utterances, slots, prompts
5. Build the locale with `BuildBotLocale`
6. Create a version with `CreateBotVersion`
7. Create an alias with `CreateBotAlias`, attach Lambda

### 4. Update Lambda Handlers

**Critical change**: The V2 Lambda event format is completely different from V1.

#### V1 Event (old)
```json
{
  "currentIntent": {
    "name": "BookRoom",
    "slots": { "RoomType": "queen", "CheckInDate": "2025-03-15" }
  },
  "invocationSource": "FulfillmentCodeHook",
  "outputDialogMode": "Text",
  "sessionAttributes": {}
}
```

#### V2 Event (new)
```json
{
  "invocationSource": "FulfillmentCodeHook",
  "sessionState": {
    "intent": {
      "name": "BookRoom",
      "slots": {
        "RoomType": {
          "value": { "interpretedValue": "queen", "resolvedValues": ["queen"] }
        }
      },
      "state": "InProgress"
    },
    "sessionAttributes": {}
  },
  "interpretations": [...]
}
```

#### Key Differences in Handler Code

| Aspect | V1 | V2 |
|--------|----|----|
| Get intent name | `event['currentIntent']['name']` | `event['sessionState']['intent']['name']` |
| Get slot value | `event['currentIntent']['slots']['SlotName']` | `event['sessionState']['intent']['slots']['SlotName']['value']['interpretedValue']` |
| Response format | `dialogAction` at root | `sessionState.dialogAction` nested |
| Close action | `{"dialogAction": {"type": "Close", ...}}` | `{"sessionState": {"dialogAction": {"type": "Close"}, "intent": {..., "state": "Fulfilled"}}}` |
| Must include | `dialogAction` | Full `sessionState` with intent + dialogAction |

### 5. Update Client Code

| V1 SDK | V2 SDK |
|--------|--------|
| `LexRuntime.postText()` | `LexRuntimeV2.recognizeText()` |
| `LexRuntime.postContent()` | `LexRuntimeV2.recognizeUtterance()` |
| Bot name + alias | Bot ID + alias ID + locale ID |

### 6. Update IAM Policies

- Replace `lex:Post*` with `lexv2-runtime:Recognize*`
- Replace `lex-models:*` with `lexv2-models:*`
- Update resource ARNs: `arn:aws:lex:region:account:bot/bot-id` → `arn:aws:lex:region:account:bot/bot-id`

### 7. Test Thoroughly

- Verify all intents trigger correctly
- Test multi-turn conversations end-to-end
- Confirm slot elicitation and validation
- Test fallback behavior
- Validate session attribute propagation

## Common Pitfalls

- **Forgetting to build**: V2 requires an explicit `BuildBotLocale` call after any change.
- **Slot value access**: V2 slots are nested objects, not simple strings. Always access via `['value']['interpretedValue']`.
- **Missing sessionState in response**: V2 Lambda responses MUST include the full `sessionState` object.
- **Alias vs version confusion**: V2 aliases point to immutable versions, not `DRAFT`. You must create a version first.
