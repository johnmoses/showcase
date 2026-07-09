# Conversational Agent — Multi-Turn Dialog + Entity Extraction + DB Writes

**System:** ChurchReports (predecessor to Life Reports)  
**Problem:** Church reporters needed to submit attendance, offerings, and testimonies via chat — not forms. The system had to handle multi-turn conversations, extract structured data from free text, and write directly to the database.

## Architecture: Intent → Route → Extract → Execute

```
User message
    │
    ▼
[BERT Intent Classifier] → tag + confidence score
    │
    ▼
[SocketIO Router] → routes to correct AI module based on starter_id
    │                  - starter_id 715 → Support (T5 QA)
    │                  - starter_id 615 → Enquiry (T5 SQL)
    │                  - default       → Collator (BERT + state machine)
    ▼
[Dialog State Machine] → tracks entity_vars across turns
    │
    ▼
[Entity Extraction] → regex-based token parsing from natural language
    │
    ▼
[Database Write] → creates Counting/Giving/Testimony records
```

**This is an agentic pattern — intent classification → tool selection → parameter extraction → autonomous execution — built before LangChain or function-calling APIs existed.**

## Key Code: WebSocket AI Router (Python/Flask-SocketIO)

```python
@socketio.on('send message')
def send_message(data):
    content = data['content']
    sender_id = data['sender_id']
    starter_id = data['starter_id']
    is_bot = data['is_bot']

    if is_private and is_bot:
        # Route to correct AI module based on chat type
        if starter_id == 715:
            response = support_response(content)      # T5 QA model
        elif starter_id == 615:
            response = sql_response(content)          # T5 text-to-SQL model
        else:
            response, ent_vars = base_response(       # BERT classifier + state machine
                content, sender_id, entity_vars)

        ai_response({"content": response, "chat_id": data['chat_id'],
                     "sender_username": "Assistant"})
```

## Key Code: Multi-Turn Dialog State Machine (Python)

```python
class BaseModelMixin:
    def response(self, message_content, sender_id, ent_vars, model_tag):
        entity_vars = {}
        dialog_mode = ent_vars.get('dialog_mode', False)

        if dialog_mode:
            # CONTINUATION — user is providing missing info from previous turn
            tag = ent_vars['chat_tag']

            if tag == 'fellowship_attendance_list':
                fellowship_id = extract_number(cleaned_content)
                if fellowship_id:
                    entity_vars['dialog_mode'] = False
                    countings = Counting.query.filter(
                        Counting.fellowship_id == fellowship_id).limit(5)
                    for ctg in countings:
                        response += f"\n 'Date' {ctg.meeting_date} 'Men' {ctg.men}..."
                else:
                    entity_vars['dialog_mode'] = True  # keep asking
                    response += "\n Could not find fellowship ID"

        else:
            # NEW INTENT — classified by BERT model
            tag = model_tag
            entity_vars['chat_tag'] = tag

            if tag == 'fellowship_attendance_list':
                fellowship_id = extract_number(cleaned_content)
                if fellowship_id is None:
                    # Missing parameter → enter dialog mode (ask user)
                    entity_vars['dialog_mode'] = True
                    response += "\n Please add fellowship ID"
                else:
                    # All params present → execute immediately
                    countings = Counting.query.filter(...).limit(5)
                    response += "\n Satisfied with the answers?"
```

## Key Code: Natural Language Report Submission (Entity Extraction → DB Write)

Users type: `"Report: fellowshipid is 485, meetingid is 1, meetingdate is 2024/06/16, men are 2, women are 3, youthboys are 4"`

```python
if tag == 'report_submission':
    # Extract all entities from free-text message
    entity_tokens, sentence_tokens = get_entity_tokens(message_content)

    fellowship_id = entity_tokens.get('fellowshipid', None)
    location_id = entity_tokens.get('locationid', None)
    meeting_id = entity_tokens.get('meetingid', None)
    meeting_date = entity_tokens.get('meetingdate', None)
    men = entity_tokens.get('men', None)
    women = entity_tokens.get('women', None)
    # ... youth_boys, youth_girls, children, offerings, testimony

    if not meeting_date or not meeting_id:
        response += "\n Need meeting date and ID!"
    else:
        if fellowship_id:
            # Write attendance to database
            CountingController.create_fellowship_counting(
                fellowship_id, meeting_id, sender_id, meeting_date,
                men, women, youth_boys, youth_girls, children_boys, children_girls,
                new_comers, new_converts)

            # Write testimony if present in same message
            if testimony:
                TestimonyController.create_fellowship_testimony(
                    fellowship_id, meeting_id, sender_id, meeting_date,
                    testimony[:10], testimony)

        if location_id and slip_number:
            # Write offering to database
            GivingController.create_giving(
                location_id, meeting_id, sender_id, meeting_date,
                slip_number, naira, euro, dollar, pound, yuan, ...)
```

## What Makes This Agentic

| Modern Agent Concept | ChurchReports Implementation (2024) |
|---------------------|--------------------------------------|
| Intent/tool selection | BERT classifier → 44 intents |
| Parameter extraction | Regex entity parser from free text |
| Multi-turn memory | `entity_vars` dict persisted across turns |
| Missing param handling | `dialog_mode` flag → asks user for info |
| Autonomous execution | Direct DB writes (Counting, Giving, Testimony) |
| Multi-agent routing | SocketIO routes to 3 different AI modules |
| Confidence gating | Score < 0.8 → refuse to act |

## Conversation Example

```
User:  "List fellowship attendance at 485"
Bot:   "Fellowship attendance list:
        Date 2024/06/16 Men 12 Women 15 Boys 8 Girls 10..."

User:  "List fellowship attendance"          ← missing parameter
Bot:   "Please add fellowship ID"            ← enters dialog mode

User:  "485"                                 ← provides missing param
Bot:   "Date 2024/06/16 Men 12 Women 15..."  ← executes with context

User:  "Report: fellowshipid is 485, meetingid is 1, meetingdate is 2024/06/23, men are 14, women are 18"
Bot:   "God bless you for submitting your report"  ← wrote to DB
```
