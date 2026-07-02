# Outreach Intelligence — Engagement Scoring & Churn Prediction

**System:** Life Reports  
**Problem:** Pastoral teams had no way to prioritize who to follow up with across thousands of members.

## Engagement Score (0–100)

Four weighted signals:

| Signal | Weight | Logic |
|--------|--------|-------|
| Attendance rate | 35% | % of events attended |
| Recency | 30% | Days since last seen (100 → 0 decay) |
| Interactions | 20% | Touchpoints in last 30 days |
| Membership stage | 15% | visitor(20) → leader(95) |

```python
def calculate_engagement_score(self, member_data: Dict) -> Dict:
    attendance  = self._score_attendance(member_data.get('attendance_history', []))
    recency     = self._score_recency(member_data.get('last_seen'))
    interaction = self._score_interactions(member_data.get('interactions', []))
    stage       = self._score_stage(member_data.get('membership_stage'))

    total = attendance * 0.35 + recency * 0.30 + interaction * 0.20 + stage * 0.15

    risk   = "low" if total >= 70 else "medium" if total >= 40 else "high"
    action = "Continue engagement" if risk == "low" else \
             "Increase contact"    if risk == "medium" else "Urgent intervention"

    return {"score": round(total, 1), "risk_level": risk, "recommended_action": action}
```

## Churn Prediction (Rule-Based)

```python
def predict_churn_risk(self, member_data: Dict) -> Dict:
    days_absent = (datetime.now() - member_data['last_seen']).days

    risk_score = 0
    if days_absent > 30:  risk_score += 40
    elif days_absent > 14: risk_score += 25
    elif days_absent > 7:  risk_score += 10

    if member_data.get('avg_attendance_rate', 0) < 0.3:
        risk_score += 30

    stage_risk = {'visitor': 20, 'convert': 15, 'member': 10, 'leader': 2}
    risk_score += stage_risk.get(member_data.get('membership_stage'), 10)

    # critical(≥70) → "Contact within 24h"
    # high(≥50)     → "Contact within 3 days"
    # medium(≥30)   → "Follow-up this week"
```

## Priority Queue

Combines churn probability + inverse engagement to rank who needs attention most:

```python
def prioritize_outreach_list(self, members: List[Dict]) -> List[Dict]:
    for member in members:
        churn      = self.predict_churn_risk(member)
        engagement = self.calculate_engagement_score(member)
        priority   = churn['probability'] * 60 + (100 - engagement['score']) * 0.4

    return sorted(scored, key=lambda x: x['priority_score'], reverse=True)
```

## Message Variant Generation

Generates 2–3 message options per context (re_engagement / evangelism / stage_advancement) with tone labels and a score, so the pastoral team picks the best fit rather than sending a generic message.
