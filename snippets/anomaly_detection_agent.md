# Attendance Anomaly Detection Agent

**System:** Life Reports  
**Problem:** Pastoral teams couldn't manually track which members had gone quiet across hundreds of centers.

## What It Does

Runs autonomously across all centers and flags:
1. **Member absences** — consecutive missed events (≥3 threshold → medium, ≥5 → high severity)
2. **Center drops** — attendance fell ≥30% comparing first-half vs second-half of recent events

## Key Code (Python)

```python
class AttendanceAnomalyDetector:
    def __init__(self):
        self.consecutive_absence_threshold = 3
        self.attendance_drop_threshold = 0.30

    async def detect_member_absences(self, center_id: int) -> List[Dict]:
        # Builds attendance map: member_id -> [attended: bool per event]
        # Then scans for consecutive False streaks
        for member_id, data in attendance_map.items():
            consecutive_absences = 0
            for attendance in data["attendance"]:
                if not attendance["attended"]:
                    consecutive_absences += 1
                else:
                    consecutive_absences = 0

                if consecutive_absences >= self.consecutive_absence_threshold:
                    anomalies.append({
                        "severity": "high" if consecutive_absences >= 5 else "medium",
                        "member_name": data["member"]["name"],
                        "consecutive_count": consecutive_absences,
                    })
                    break

    async def detect_center_attendance_drops(self, center_id: int) -> List[Dict]:
        # Splits events into first/second half, compares averages
        mid_point = len(events) // 2
        first_avg = avg(events[:mid_point])
        second_avg = avg(events[mid_point:])
        drop_pct = (first_avg - second_avg) / first_avg

        if drop_pct >= self.attendance_drop_threshold:
            return [{"severity": "high" if drop_pct >= 0.5 else "medium",
                     "drop_percentage": round(drop_pct * 100, 1)}]

    async def run_detection(self, center_ids=None) -> Dict:
        # Fetches all center IDs if none provided, runs both detectors
        for center_id in center_ids:
            results["member_anomalies"].extend(await self.detect_member_absences(center_id))
            results["center_anomalies"].extend(await self.detect_center_attendance_drops(center_id))
```

## Output Shape

```json
{
  "member_anomalies": [
    {"severity": "high", "member_name": "Emeka Obi", "consecutive_count": 5}
  ],
  "center_anomalies": [
    {"severity": "medium", "drop_percentage": 34.2, "previous_avg": 45.0, "current_avg": 29.7}
  ],
  "summary": {
    "total_member_anomalies": 12,
    "high_severity_members": 3,
    "centers_analyzed": 47
  }
}
```
