---
name: apify-api-integration
description: Run Apify Actors/tasks via API, wait for completion, retrieve data. Covers sync/async flows, endpoints, client libraries.
---

# Apify API Integration

## Use When
User needs to run Apify Actors/tasks via API, wait for completion, retrieve data.

## Key Concepts
- **Actor**: Serverless program (scraping, automation)
- **Task**: Pre-configured Actor input
- **Sync** (≤5min): Single call waits for result
- **Async** (>5min): Start → Wait → Retrieve

## Requirements
- Actor/Task name (`username~actorName`) or ID
- API token: https://console.apify.com/account?tab=integrations
- Input JSON (optional)

## Endpoints

**Run Actor:**
`POST https://api.apify.com/v2/acts/ACTOR_ID/runs?token=TOKEN`

**Run Task:**
`POST https://api.apify.com/v2/actor-tasks/TASK_ID/runs?token=TOKEN`

**Parameters:** `&memory=8192&build=beta&waitForFinish=60`

## Synchronous (≤5min)

**Dataset output:**
`POST https://api.apify.com/v2/acts/ACTOR_ID/run-sync-get-dataset-items?token=TOKEN`

**KV Store output:**
`POST https://api.apify.com/v2/acts/ACTOR_ID/run-sync?token=TOKEN`

## Asynchronous (>5min)

**1. Start:** POST to `/runs` endpoint → Get `id`, `defaultDatasetId`, `defaultKeyValueStoreId`

**2. Wait (choose one):**
- **waitForFinish param**: Add `&waitForFinish=60` (max 60s)
- **Webhook**: Set in console, receives POST on completion (respond 200)
- **Polling**: GET `/runs/RUN_ID` every 5-10s, check `status`: SUCCEEDED/FAILED

**3. Retrieve:**

Dataset: `GET https://api.apify.com/v2/datasets/DATASET_ID/items?format=json&limit=250000&offset=0`

KV Store: `GET https://api.apify.com/v2/key-value-stores/STORE_ID/records/OUTPUT`

KV Keys: `GET https://api.apify.com/v2/key-value-stores/STORE_ID/keys?exclusiveStartKey=lastKey`

## Client Libraries (Recommended)

**JS:**
```js
import { ApifyClient } from 'apify-client';
const client = new ApifyClient({ token: 'TOKEN' });
const run = await client.actor('ACTOR_ID').call(input);
const { items } = await client.dataset(run.defaultDatasetId).listItems();
```

**Python:**
```python
from apify_client import ApifyClient
client = ApifyClient(token='TOKEN')
run = client.actor("ACTOR_ID").call(run_input=input)
for item in client.dataset(run["defaultDatasetId"]).iterate_items():
    print(item)
```

## Response Fields
- `id`: Run ID
- `status`: READY, RUNNING, SUCCEEDED, FAILED
- `defaultDatasetId`: Results location
- `defaultKeyValueStoreId`: Files location

## Limits
- Sync timeout: 5 minutes
- Dataset items per request: 250,000 (use offset for more)
- KV Store keys per request: 1,000 (use exclusiveStartKey)
- waitForFinish max: 60 seconds

## Best Practices
- ≤5min runs: sync endpoints
- >5min runs: async + webhooks/polling
- Never expose tokens client-side
- Check Actor input schema before calling
- Handle status: FAILED with retries