# Voice Capture Pipeline

**System:** Life Reports — Church attendance platform (10K users)  
**Problem:** Field counsellors at crusade events needed to capture convert details hands-free, in noisy environments, offline-capable, in multiple Nigerian languages.

## Architecture: Hybrid Inference

```
Audio chunk (10s)
    │
    ▼
[Real-time: cheap regex parse per chunk]  ← instant UI feedback
    │
    ▼
Progressive field fill (name, phone, address...)
    │
    ▼
Session end → [LLM extraction on full transcript]  ← accuracy pass
    │
    ▼
Structured form pre-filled for counsellor review
```

**Why hybrid?** Regex per chunk = zero latency for UX. LLM at session end = accuracy without blocking the conversation.

## Key Code: Chunk Recording + Progressive Fill (TypeScript/React)

```typescript
// ConversationalCapture.tsx — simplified
recorder.onstop = async () => {
  const blob = new Blob(audioChunksRef.current, { type: actualType });

  // 1. Transcribe chunk
  const resp = await fetch(`${API_BASE}/public/voice/transcribe`, {
    method: 'POST', body: formData,
  });
  const { text: chunkTranscript } = await resp.json();

  // 2. Quick regex parse — fills fields progressively as counsellor speaks
  const { parsed } = await fetch(`${API_BASE}/public/voice/parse-decision`, {
    method: 'POST',
    body: JSON.stringify({ text: chunkTranscript }),
  }).then(r => r.json());

  // Only fill fields not yet captured
  const newFields = Object.fromEntries(
    Object.entries(parsed).filter(([k]) => !currentFields[k])
  );
  setFields(prev => ({ ...prev, ...newFields }));

  // 3. Fire-and-forget server backup (offline resilience)
  fetch(backupUrl, { method: 'POST', body: backupForm }).catch(() => {});
};

// Session end — full LLM extraction pass
const resp = await fetch(`${API_BASE}/public/voice/extract-conversation`, {
  method: 'POST',
  body: JSON.stringify({ transcript, language, event_id: eventId }),
});
const { parsed } = await resp.json();
// parsed contains all fields with higher accuracy than per-chunk regex
```

## Key Code: Multilingual NLP Parser (Python)

```python
# decision_parser.py — domain-specific NLP, no external ML dependency
DECISION_KEYWORDS = {
    "salvation": [
        "saved", "accepted christ", "born again",       # English
        "ceto", "ya karbi yesu",                         # Hausa
        "igbala", "gba jesu",                            # Yoruba
        "nzoputa", "nara yesu",                          # Igbo
    ],
    # ... healing, deliverance, rededication, holy_ghost
}

AREA_CORRECTIONS = {
    "cobra": "Kubwa",   # speech-to-text mishears
    "nanya": "Nyanya",
    "gaki":  "Garki",
}

def _calculate_confidence(self, result: Dict) -> float:
    score = 0.2
    if result["full_name"]:     score += 0.25
    if result["mobile_number"]: score += 0.20
    if result["address"]:       score += 0.10
    if result["invited_by"]:    score += 0.10
    return min(1.0, score)
```

## Offline Resilience

- Audio chunks stored in IndexedDB via `sessionStore` — survives page refresh
- Transcription skipped if offline, audio kept for later retry
- Server backup is fire-and-forget — never blocks the counsellor
- Draft saved to IndexedDB if LLM extraction fails or times out

## Languages Supported
English · Hausa · Yoruba · Igbo · Efik · Idoma
