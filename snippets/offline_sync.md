# Offline-First Architecture

**System:** Life Reports  
**Problem:** Church reporters in low-connectivity areas needed to submit attendance reports without internet. Data had to sync reliably when connection returned.

## Two Layers

| Layer | Technology | Queue Storage |
|-------|-----------|---------------|
| Mobile (Flutter) | `connectivity_plus` + SQLite | `pending_sync` table |
| Web (Next.js) | `navigator.onLine` + `online` event | `localStorage` |

## Mobile Sync Service (Dart)

```dart
class SyncService {
  static bool _isSyncing = false;  // prevents concurrent syncs

  static Future<void> syncData() async {
    if (_isSyncing) return;
    if (!await isOnline()) return;

    _isSyncing = true;
    try {
      await _syncPendingOperations();  // push local queue to server
      await _syncFromServer();         // pull centers, meetings, last 30 days reports
    } finally {
      _isSyncing = false;
    }
  }

  static Future<void> schedulePeriodicSync() async {
    // Triggers sync automatically when connectivity is restored
    Connectivity().onConnectivityChanged.listen((result) {
      if (result != ConnectivityResult.none) syncData();
    });
  }
}
```

## Web Offline Queue (TypeScript)

```typescript
class OfflineQueue {
  async add(request: QueuedRequest): Promise<void> {
    const queue = this.getQueue();
    queue.push({ ...request, id: Date.now(), retries: 0 });
    localStorage.setItem(QUEUE_KEY, JSON.stringify(queue));
  }

  async processQueue(): Promise<void> {
    for (const item of this.getQueue()) {
      try {
        await this.executeWithTimeout(item, 10_000);  // 10s timeout
        this.removeFromQueue(item.id);
      } catch (err: any) {
        if (err.status >= 500) {
          // Server error — keep for retry (max 5 attempts)
          if (item.retries >= 5) this.removeFromQueue(item.id);
          else this.incrementRetry(item.id);
        } else {
          // Client error (4xx) — discard, won't succeed on retry
          this.removeFromQueue(item.id);
        }
      }
    }
  }
}

// Auto-sync when connection returns
window.addEventListener('online', () => offlineQueue.processQueue());
```

## SQLite Schema (Mobile)

```sql
-- Mirrors server schema for full offline capability
CREATE TABLE pending_sync (
  id INTEGER PRIMARY KEY,
  operation TEXT,   -- 'CREATE_REPORT', etc.
  data TEXT,        -- JSON payload
  created_at TEXT
);
-- Also caches: users, centers, reports, members, meetings
```
