<p align="center"><img src=".github/hero.svg" alt="cron" width="880"></p>

# Hanzo Cron

Scheduled jobs and delayed execution engine for reliable, distributed task scheduling.

## Features

- **Cron Expressions** - Full cron syntax support with seconds precision
- **One-Time Schedules** - Schedule jobs for specific future times
- **Timezone Support** - Schedule jobs in any timezone with DST handling
- **Distributed Locking** - Prevent duplicate execution across nodes
- **Retry Policies** - Configurable retry with exponential backoff
- **Job Dependencies** - Chain jobs with DAG-based dependencies
- **Observability** - Built-in metrics, tracing, and logging

## Quick Start

### Docker

```bash
docker run -d \
  -p 8080:8080 \
  -e DATABASE_URL=postgres://user:pass@host:5432/cron \
  -e REDIS_URL=redis://localhost:6379 \
  hanzoai/cron:latest
```

### Docker Compose

```yaml
version: '3.8'
services:
  cron:
    image: hanzoai/cron:latest
    ports:
      - "8080:8080"
    environment:
      DATABASE_URL: postgres://user:pass@postgres:5432/cron
      REDIS_URL: redis://redis:6379
    depends_on:
      - postgres
      - redis
```

## SDK Examples

### Python

```python
from hanzo import Cron

cron = Cron(api_key="your-api-key")

# Create a recurring job
job = cron.schedule(
    name="daily-report",
    cron="0 9 * * *",  # Every day at 9 AM
    timezone="America/New_York",
    handler="https://api.example.com/reports/generate",
    retry_policy={
        "max_attempts": 3,
        "backoff": "exponential"
    }
)

# Create a one-time job
job = cron.schedule_once(
    name="welcome-email",
    run_at="2024-01-15T10:00:00Z",
    handler="https://api.example.com/email/send",
    payload={"user_id": "123"}
)

# List scheduled jobs
jobs = cron.list(status="active")

# Cancel a job
cron.cancel(job_id="job_abc123")
```

### TypeScript

```typescript
import { Cron } from '@hanzo/sdk';

const cron = new Cron({ apiKey: 'your-api-key' });

// Create a recurring job
const job = await cron.schedule({
  name: 'daily-report',
  cron: '0 9 * * *',
  timezone: 'America/New_York',
  handler: 'https://api.example.com/reports/generate',
  retryPolicy: {
    maxAttempts: 3,
    backoff: 'exponential'
  }
});

// Create a one-time delayed job
const delayedJob = await cron.scheduleOnce({
  name: 'welcome-email',
  runAt: new Date(Date.now() + 3600000), // 1 hour from now
  handler: 'https://api.example.com/email/send',
  payload: { userId: '123' }
});

// Pause/resume jobs
await cron.pause(job.id);
await cron.resume(job.id);
```

### Go

```go
package main

import (
    "github.com/hanzoai/go-sdk/cron"
)

func main() {
    client := cron.New("your-api-key")

    // Create a recurring job
    job, err := client.Schedule(cron.ScheduleParams{
        Name:     "daily-report",
        Cron:     "0 9 * * *",
        Timezone: "America/New_York",
        Handler:  "https://api.example.com/reports/generate",
    })

    // Create a one-time job
    job, err = client.ScheduleOnce(cron.ScheduleOnceParams{
        Name:    "welcome-email",
        RunAt:   time.Now().Add(time.Hour),
        Handler: "https://api.example.com/email/send",
        Payload: map[string]interface{}{"userId": "123"},
    })
}
```

## API Reference

### Schedule a Job

```bash
curl -X POST https://cron.hanzo.ai/v1/jobs \
  -H "Authorization: Bearer $HANZO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "daily-report",
    "cron": "0 9 * * *",
    "timezone": "America/New_York",
    "handler": "https://api.example.com/reports/generate"
  }'
```

### List Jobs

```bash
curl https://cron.hanzo.ai/v1/jobs \
  -H "Authorization: Bearer $HANZO_API_KEY"
```

## Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `DATABASE_URL` | PostgreSQL connection string | required |
| `REDIS_URL` | Redis connection for distributed locking | required |
| `PORT` | HTTP server port | `8080` |
| `LOG_LEVEL` | Logging level (debug, info, warn, error) | `info` |
| `MAX_CONCURRENT_JOBS` | Maximum concurrent job executions | `100` |
| `JOB_TIMEOUT` | Default job execution timeout | `30s` |

## Architecture

```
┌─────────────────┐     ┌─────────────────┐
│   Scheduler     │────▶│   Job Queue     │
│   (Leader)      │     │   (Redis)       │
└─────────────────┘     └────────┬────────┘
                                 │
                    ┌────────────┼────────────┐
                    ▼            ▼            ▼
              ┌──────────┐ ┌──────────┐ ┌──────────┐
              │ Worker 1 │ │ Worker 2 │ │ Worker N │
              └──────────┘ └──────────┘ └──────────┘
                    │            │            │
                    └────────────┴────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │     PostgreSQL          │
                    │  (Job State & History)  │
                    └─────────────────────────┘
```

## License

MIT License - see [LICENSE](LICENSE) for details.

---

Part of the [Hanzo](https://hanzo.ai) execution layer.
