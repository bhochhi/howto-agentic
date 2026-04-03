# Amazon Lex V2 Advanced Patterns

## Multi-Intent Conversations

### Intent Chaining via Session Attributes

After fulfilling one intent, guide users to a follow-up intent:

```python
def build_fulfillment_response(event, intent):
    return {
        "sessionState": {
            "dialogAction": {"type": "ElicitIntent"},
            "sessionAttributes": {
                **event['sessionState'].get('sessionAttributes', {}),
                "lastCompletedIntent": intent['name'],
            }
        },
        "messages": [
            {"contentType": "PlainText", "content": "Room booked! Would you like to add breakfast or a spa reservation?"}
        ]
    }
```

### Sub-Intents Pattern

Use session attributes to track parent intent context:

```python
session_attrs = event['sessionState'].get('sessionAttributes', {})
parent_intent = session_attrs.get('parentIntent')

if parent_intent == 'BookRoom' and intent_name == 'AddBreakfast':
    room_booking_id = session_attrs.get('bookingId')
    # Add breakfast to existing booking
```

## Conditional Branches

V2 supports conditional branching in the visual builder and via API. Conditions evaluate:

- Slot values: `{SlotName} == "value"`
- Session attributes: `{SessionAttribute.key} == "value"`
- Intent confidence: Built-in thresholds

### Setting Conditions via CDK

```typescript
intentClosingSetting: {
  closingResponse: {
    messageGroupsList: [{
      message: { plainTextMessage: { value: 'Done!' } }
    }],
  },
  conditional: {
    isActive: true,
    conditionalBranches: [{
      name: 'HighValue',
      condition: { expressionString: '{Amount} > 1000' },
      response: {
        messageGroupsList: [{
          message: { plainTextMessage: { value: 'For orders over $1000, a manager will review.' } }
        }],
      },
      nextStep: { dialogAction: { type: 'Close' }, intent: { name: 'ManagerReview' } },
    }],
    defaultBranch: {
      nextStep: { dialogAction: { type: 'Close' } },
    },
  },
},
```

## Wait and Continue

Pause a conversation and resume later (useful for async operations):

```python
# In DialogCodeHook — tell Lex to wait
return {
    "sessionState": {
        "dialogAction": {
            "type": "ElicitSlot",
            "slotToElicit": "Confirmation"
        },
        "intent": intent,
        "sessionAttributes": {
            "asyncTaskId": "task-123",
            "waitingForCallback": "true"
        }
    },
    "messages": [
        {"contentType": "PlainText", "content": "Processing your request. I'll let you know when it's ready. You can say 'check status' anytime."}
    ]
}
```

## Response Cards

Rich responses with buttons:

```python
{
    "messages": [{
        "contentType": "ImageResponseCard",
        "imageResponseCard": {
            "title": "Select Room Type",
            "subtitle": "Choose your preferred room",
            "imageUrl": "https://example.com/rooms.jpg",
            "buttons": [
                {"text": "King Room", "value": "king"},
                {"text": "Queen Room", "value": "queen"},
                {"text": "Deluxe Suite", "value": "deluxe"}
            ]
        }
    }]
}
```

## SSML for Voice

When integrated with voice channels (Connect, streaming):

```python
{
    "messages": [{
        "contentType": "SSML",
        "content": "<speak>Your booking is confirmed for <say-as interpret-as='date'>20250315</say-as>. <break time='500ms'/> Is there anything else?</speak>"
    }]
}
```

## Custom Vocabulary

Boost recognition of domain terms:

```json
{
  "customVocabularyItems": [
    { "phrase": "CPAP", "weight": 3 },
    { "phrase": "BiPAP", "weight": 3 },
    { "phrase": "Amoxicillin", "weight": 2, "displayAs": "amoxicillin" }
  ]
}
```

Weight: 0 (no boost) to 3 (maximum boost).

## Streaming Conversations (WebSocket)

Use `StartConversation` for real-time bidirectional streaming:

```typescript
import { LexRuntimeV2Client, StartConversationCommand } from '@aws-sdk/client-lex-runtime-v2';

const client = new LexRuntimeV2Client({ region: 'us-east-1' });

// The StartConversation API uses HTTP/2 streaming
const response = await client.send(new StartConversationCommand({
  botId: 'BOT_ID',
  botAliasId: 'ALIAS_ID',
  localeId: 'en_US',
  sessionId: 'session-123',
  conversationMode: 'AUDIO',  // or 'TEXT'
  requestEventStream: asyncIterableOfEvents,
}));

for await (const event of response.responseEventStream) {
  if (event.TranscriptEvent) {
    console.log('Transcript:', event.TranscriptEvent.transcript);
  }
  if (event.TextResponseEvent) {
    console.log('Bot says:', event.TextResponseEvent.messages);
  }
  if (event.IntentResultEvent) {
    console.log('Intent:', event.IntentResultEvent.sessionState.intent.name);
  }
}
```

## Terraform Deployment

```hcl
resource "aws_lexv2models_bot" "hotel_bot" {
  name                        = "HotelBot"
  idle_session_ttl_in_seconds = 300
  role_arn                    = aws_iam_role.lex_role.arn

  data_privacy {
    child_directed = false
  }
}

resource "aws_lexv2models_bot_locale" "en_us" {
  bot_id                           = aws_lexv2models_bot.hotel_bot.id
  bot_version                      = "DRAFT"
  locale_id                        = "en_US"
  n_lu_intent_confidence_threshold = 0.40
}

resource "aws_lexv2models_intent" "book_room" {
  bot_id      = aws_lexv2models_bot.hotel_bot.id
  bot_version = "DRAFT"
  locale_id   = "en_US"
  name        = "BookRoom"

  sample_utterance {
    utterance = "I want to book a {RoomType} room"
  }
  sample_utterance {
    utterance = "Reserve a room for {CheckInDate}"
  }

  fulfillment_code_hook {
    enabled = true
  }
}
```

## Error Handling Patterns

### Graceful Slot Validation

```python
def validate_date(check_in_date):
    """Validate the date is in the future."""
    from datetime import datetime, date
    try:
        parsed = datetime.strptime(check_in_date, '%Y-%m-%d').date()
        if parsed <= date.today():
            return False, "The check-in date must be in the future. What date would you like?"
        return True, None
    except ValueError:
        return False, "I didn't understand that date. Please provide a date like March 15, 2025."


def handler(event, context):
    intent = event['sessionState']['intent']
    slots = intent.get('slots', {})

    if event['invocationSource'] == 'DialogCodeHook':
        check_in = slots.get('CheckInDate', {})
        if check_in and check_in.get('value', {}).get('interpretedValue'):
            valid, message = validate_date(check_in['value']['interpretedValue'])
            if not valid:
                # Clear the invalid slot and re-elicit
                slots['CheckInDate'] = None
                return {
                    "sessionState": {
                        "dialogAction": {"type": "ElicitSlot", "slotToElicit": "CheckInDate"},
                        "intent": {**intent, "slots": slots},
                        "sessionAttributes": event['sessionState'].get('sessionAttributes', {})
                    },
                    "messages": [{"contentType": "PlainText", "content": message}]
                }
        return {
            "sessionState": {
                "dialogAction": {"type": "Delegate"},
                "intent": intent,
                "sessionAttributes": event['sessionState'].get('sessionAttributes', {})
            }
        }
```

### Fallback with Context

```python
def handle_fallback(event):
    session_attrs = event['sessionState'].get('sessionAttributes', {})
    attempt_count = int(session_attrs.get('fallbackCount', '0')) + 1
    session_attrs['fallbackCount'] = str(attempt_count)

    if attempt_count >= 3:
        return {
            "sessionState": {
                "dialogAction": {"type": "Close"},
                "intent": {**event['sessionState']['intent'], "state": "Failed"},
                "sessionAttributes": session_attrs
            },
            "messages": [{"contentType": "PlainText", "content": "Let me connect you with a human agent."}]
        }

    return {
        "sessionState": {
            "dialogAction": {"type": "ElicitIntent"},
            "sessionAttributes": session_attrs
        },
        "messages": [{"contentType": "PlainText", "content": f"I didn't catch that. I can help you book a room, cancel a reservation, or check availability. What would you like to do?"}]
    }
```
