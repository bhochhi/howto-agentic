---
name: amazon-lex-v2
description: "Expert guidance for Amazon Lex V2 chatbot development. Use when: designing conversational bots, configuring intents/slots/fulfillment, writing Lambda fulfillment handlers, setting up multi-turn dialogs, deploying Lex V2 bots via CDK/CloudFormation/Terraform, integrating with Connect/Messaging channels, troubleshooting utterance resolution or slot elicitation issues, migrating from Lex V1 to V2."
argument-hint: 'Describe what you want to build or troubleshoot with Amazon Lex V2'
---

# Amazon Lex V2 Expert

Provides step-by-step guidance for building, deploying, and troubleshooting Amazon Lex V2 conversational bots.

## When to Use

- Designing a new Lex V2 bot (intents, slots, utterances, dialogs)
- Writing or debugging Lambda fulfillment/code-hook functions
- Configuring multi-turn conversations and slot elicitation
- Setting up conditional branching and dialog code hooks
- Deploying Lex bots with IaC (CDK, CloudFormation, Terraform)
- Integrating Lex V2 with Amazon Connect, Twilio, Slack, Facebook, or web apps
- Migrating from Lex V1 to V2
- Troubleshooting recognition, fulfillment, or channel integration issues

## Key Concepts

### Lex V2 Resource Hierarchy

```
Bot
├── Bot Locale (en_US, es_ES, ...)
│   ├── Intents
│   │   ├── Sample Utterances
│   │   ├── Slots (with Slot Types)
│   │   ├── Slot Priorities
│   │   ├── Confirmation Prompt / Decline Response
│   │   ├── Dialog Code Hook (validation)
│   │   ├── Fulfillment Code Hook (business logic)
│   │   └── Conditional Branches
│   ├── Slot Types (custom / built-in)
│   └── Custom Vocabulary
├── Bot Aliases (PROD, STAGING, ...)
│   └── Lambda association (per locale)
└── Bot Versions (immutable snapshots)
```

### V2 vs V1 Key Differences

| Feature | V1 | V2 |
|---------|----|----|
| Multi-language | Separate bots | Single bot, multiple locales |
| Versioning | Per-resource | Whole-bot versions |
| Streaming | Not supported | Supported |
| Conversation flow | Linear | Conditional branches, wait-and-continue |
| API | `lex-models`, `lex-runtime` | `lexv2-models`, `lexv2-runtime` |

## Procedure: Design a New Bot

1. **Define the conversation goal** — What task does the bot complete? What information does it need?
2. **Model intents** — One intent per user goal. Use `FallbackIntent` for unmatched input.
3. **Define slots** — Each piece of required data is a slot. Choose built-in types (`AMAZON.Date`, `AMAZON.Number`, `AMAZON.City`, etc.) or create custom slot types.
4. **Write sample utterances** — 15–20 per intent minimum. Include variations with slot references: `I want to book a {RoomType} room on {CheckInDate}`.
5. **Configure slot elicitation** — Set prompts, retry limits, default values, wait-and-continue settings.
6. **Add confirmation** — For high-stakes actions, add a confirmation prompt before fulfillment.
7. **Implement fulfillment** — Use a Lambda function or return a closing response.
8. **Set up conditional branches** — Use conditions on slot values or session attributes to branch dialog flow.
9. **Build & test** — Build the locale, test in the Lex V2 console or via API.
10. **Create alias & deploy** — Create a version, point an alias to it, integrate with channels.

## Procedure: Lambda Fulfillment Handler

### Event Structure (V2 Format)

```json
{
  "sessionId": "session-id",
  "inputTranscript": "user utterance",
  "interpretations": [...],
  "invocationSource": "DialogCodeHook | FulfillmentCodeHook",
  "sessionState": {
    "intent": {
      "name": "BookRoom",
      "slots": {
        "RoomType": { "value": { "interpretedValue": "queen", "resolvedValues": ["queen"] } },
        "CheckInDate": { "value": { "interpretedValue": "2025-03-15", "resolvedValues": ["2025-03-15"] } }
      },
      "state": "InProgress",
      "confirmationState": "None"
    },
    "sessionAttributes": {},
    "dialogAction": { "type": "ElicitSlot | Delegate | Close" }
  },
  "bot": { "id": "bot-id", "name": "HotelBot", "aliasId": "alias-id", "localeId": "en_US", "version": "DRAFT" }
}
```

### Response Structure

```python
def handler(event, context):
    intent = event['sessionState']['intent']
    invocation_source = event['invocationSource']

    if invocation_source == 'DialogCodeHook':
        # Validation logic
        return build_validation_response(event, intent)

    if invocation_source == 'FulfillmentCodeHook':
        # Business logic
        return build_fulfillment_response(event, intent)


def build_validation_response(event, intent):
    """Return Delegate to let Lex handle next step, or ElicitSlot to re-prompt."""
    return {
        "sessionState": {
            "dialogAction": {"type": "Delegate"},
            "intent": intent,
            "sessionAttributes": event['sessionState'].get('sessionAttributes', {})
        }
    }


def build_fulfillment_response(event, intent):
    """Close the intent with a fulfillment message."""
    return {
        "sessionState": {
            "dialogAction": {"type": "Close"},
            "intent": {**intent, "state": "Fulfilled"},
            "sessionAttributes": event['sessionState'].get('sessionAttributes', {})
        },
        "messages": [
            {"contentType": "PlainText", "content": "Your booking is confirmed!"}
        ]
    }
```

### Common Dialog Actions

| Action | When to Use |
|--------|------------|
| `Delegate` | Let Lex handle the next step (elicit next slot, confirm, fulfill) |
| `ElicitSlot` | Re-prompt a specific slot (validation failed). Set `slotToElicit`. |
| `ElicitIntent` | Ask user for a new intent (reset) |
| `ConfirmIntent` | Ask user to confirm before fulfillment |
| `Close` | End the conversation. Set intent `state` to `Fulfilled` or `Failed`. |

## Procedure: CDK Deployment (TypeScript)

```typescript
import * as lex from 'aws-cdk-lib/aws-lex';
import * as lambda from 'aws-cdk-lib/aws-lambda';

// Lambda for fulfillment
const fulfillmentFn = new lambda.Function(this, 'LexFulfillment', {
  runtime: lambda.Runtime.PYTHON_3_12,
  handler: 'index.handler',
  code: lambda.Code.fromAsset('lambda/lex-fulfillment'),
});

// Lex V2 Bot
const bot = new lex.CfnBot(this, 'HotelBot', {
  name: 'HotelBot',
  roleArn: lexRole.roleArn,
  dataPrivacy: { ChildDirected: false },
  idleSessionTtlInSeconds: 300,
  botLocales: [{
    localeId: 'en_US',
    nluConfidenceThreshold: 0.40,
    intents: [{
      name: 'BookRoom',
      sampleUtterances: [
        { utterance: 'I want to book a {RoomType} room' },
        { utterance: 'Reserve a room for {CheckInDate}' },
      ],
      slots: [{
        name: 'RoomType',
        slotTypeName: 'RoomTypeSlotType',
        valueElicitationSetting: {
          slotConstraint: 'Required',
          promptSpecification: {
            messageGroupsList: [{ message: { plainTextMessage: { value: 'What type of room? (king, queen, deluxe)' } } }],
            maxRetries: 2,
          },
        },
      }],
      fulfillmentCodeHook: { enabled: true },
    },
    {
      name: 'FallbackIntent',
      parentIntentSignature: 'AMAZON.FallbackIntent',
    }],
    slotTypes: [{
      name: 'RoomTypeSlotType',
      valueSelectionSetting: { resolutionStrategy: 'ORIGINAL_VALUE' },
      slotTypeValues: [
        { sampleValue: { value: 'king' } },
        { sampleValue: { value: 'queen' } },
        { sampleValue: { value: 'deluxe' } },
      ],
    }],
  }],
});

// Bot Version + Alias
const botVersion = new lex.CfnBotVersion(this, 'BotV1', {
  botId: bot.attrId,
  botVersionLocaleSpecification: [{
    localeId: 'en_US',
    botVersionLocaleDetails: { sourceBotVersion: 'DRAFT' },
  }],
});

const alias = new lex.CfnBotAlias(this, 'ProdAlias', {
  botId: bot.attrId,
  botAliasName: 'prod',
  botVersion: botVersion.attrBotVersion,
  botAliasLocaleSettings: [{
    localeId: 'en_US',
    botAliasLocaleSetting: {
      enabled: true,
      codeHookSpecification: {
        lambdaCodeHook: {
          codeHookInterfaceVersion: '1.0',
          lambdaArn: fulfillmentFn.functionArn,
        },
      },
    },
  }],
});
```

## Procedure: Troubleshooting

### Bot doesn't recognize utterances
1. Check that the bot locale is **built** after changes.
2. Verify sample utterances cover natural phrasing variations.
3. Lower `nluConfidenceThreshold` (default 0.40) if too many go to FallbackIntent.
4. Enable **conversation logs** on the alias to inspect raw NLU scores.
5. Add **custom vocabulary** for domain-specific terms.

### Lambda not invoked
1. Confirm `fulfillmentCodeHook` or `dialogCodeHook` is enabled on the intent.
2. Verify the alias has Lambda associated for the correct locale.
3. Check Lambda resource policy allows `lex.amazonaws.com` to invoke it.
4. Review CloudWatch Logs for the Lambda function.

### Slot not elicited
1. Ensure slot `slotConstraint` is `Required`, not `Optional`.
2. Check slot priority order — Lex elicits in priority order.
3. If using `DialogCodeHook`, ensure the response returns `Delegate` or `ElicitSlot` (not `Close`).

### Session attributes lost
1. Always pass `sessionAttributes` back in every Lambda response.
2. Session attributes persist within a session but are lost across sessions.

## Procedure: Testing via AWS CLI

```bash
# Start a conversation
aws lexv2-runtime recognize-text \
  --bot-id "BOT_ID" \
  --bot-alias-id "ALIAS_ID" \
  --locale-id "en_US" \
  --session-id "test-session-1" \
  --text "I want to book a room"

# Continue the conversation (same session-id)
aws lexv2-runtime recognize-text \
  --bot-id "BOT_ID" \
  --bot-alias-id "ALIAS_ID" \
  --locale-id "en_US" \
  --session-id "test-session-1" \
  --text "queen"
```

## Procedure: Channel Integration

### Amazon Connect
1. Create a Lex V2 bot and alias.
2. In the Connect console, add the bot under **Contact flows → Amazon Lex**.
3. In the contact flow, add a **Get customer input** block → select the Lex bot and alias.
4. Branch on intent name from the block output.

### Web / Mobile (AWS SDK)
Use `LexRuntimeV2Client` with `RecognizeText` or `RecognizeUtterance` (for voice):

```typescript
import { LexRuntimeV2Client, RecognizeTextCommand } from '@aws-sdk/client-lex-runtime-v2';

const client = new LexRuntimeV2Client({ region: 'us-east-1' });
const response = await client.send(new RecognizeTextCommand({
  botId: 'BOT_ID',
  botAliasId: 'ALIAS_ID',
  localeId: 'en_US',
  sessionId: 'user-session-123',
  text: 'I want to book a room',
}));
```

## References

- [Amazon Lex V2 Developer Guide](./references/lex-v2-developer-guide.md)
- [Migration Guide: V1 to V2](./references/migration-v1-to-v2.md)
- [Advanced Patterns](./references/advanced-patterns.md)
