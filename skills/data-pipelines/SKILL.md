---
name: data-pipelines
description: Build ETL/data pipelines - extract, transform, load with proper error handling and monitoring.
metadata:
  priority: 8
  docs:
    - "https://airflow.apache.org/docs/"
  pathPatterns:
    - "**/pipeline/**"
    - "**/etl/**"
    - "**/data/**"
  bashPatterns:
    - '\betl\b'
    - '\bpipeline\b'
    - '\bairflow\b'
  promptSignals:
    phrases:
      - "etl"
      - "pipeline"
      - "data engineering"
    anyOf:
      - "extract"
      - "transform"
      - "load"
      - "batch"
---

## Data Pipelines

### Pipeline Architecture

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Source  │───▶│ Extract  │───▶│Transform │───▶│  Load   │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
                                          │            │
                                          ▼            ▼
                                    ┌──────────┐  ┌──────────┐
                                    │   Data   │  │  Audit   │
                                    │  Store   │  │   Log    │
                                    └──────────┘  └──────────┘
```

### Basic Pipeline (Node.js)

```typescript
interface PipelineStep<TInput, TOutput> {
  name: string;
  transform: (input: TInput) => Promise<TOutput>;
  onError?: (error: Error, input: TInput) => Promise<TOutput>;
}

class DataPipeline<TInput, TOutput> {
  constructor(private steps: PipelineStep<any, any>[]) {}

  async run(input: TInput): Promise<TOutput> {
    let current = input;

    for (const step of this.steps) {
      console.log(`Running step: ${step.name}`);
      try {
        current = await step.transform(current);
      } catch (error) {
        if (step.onError) {
          current = await step.onError(error as Error, current);
        } else {
          throw error;
        }
      }
    }

    return current;
  }
}
```

### Source Extraction

```typescript
// Extract from API
async function extractFromAPI(url: string): Promise<Data[]> {
  const response = await fetch(url);
  return response.json();
}

// Extract from database
async function extractFromDatabase(query: string): Promise<Data[]> {
  const { rows } = await db.query(query);
  return rows;
}

// Extract from S3
async function extractFromS3(bucket: string, key: string): Promise<string> {
  const s3 = new S3Client();
  const response = await s3.send(new GetObjectCommand({ Bucket: bucket, Key: key }));
  return response.Body?.transformToString() || '';
}
```

### Transformations

```typescript
// Clean and normalize data
function cleanUserData(users: RawUser[]): CleanUser[] {
  return users
    .filter(u => u.email && u.email.includes('@'))
    .map(u => ({
      id: u.id,
      email: u.email.toLowerCase().trim(),
      name: u.name?.trim() || null,
      createdAt: new Date(u.created_at),
      metadata: u.extra || {},
    }));
}

// Aggregate data
function aggregateOrders(orders: Order[]): DailySummary[] {
  const byDay = groupBy(orders, o => format(o.createdAt, 'yyyy-MM-dd'));

  return Object.entries(byDay).map(([day, dayOrders]) => ({
    date: day,
    totalOrders: dayOrders.length,
    totalRevenue: sum(dayOrders.map(o => o.amount)),
    avgOrderValue: avg(dayOrders.map(o => o.amount)),
  }));
}
```

### Loading Data

```typescript
// Batch insert with upsert
async function loadToDatabase(
  table: string,
  records: Record<string, any>[]
): Promise<number> {
  if (records.length === 0) return 0;

  const columns = Object.keys(records[0]);
  const values = records.map(r => columns.map(c => r[c]));
  const placeholders = values.map((_, i) =>
    `(${columns.map((_, j) => `$${i * columns.length + j + 1}`).join(',')})`
  ).join(',');

  const query = `
    INSERT INTO ${table} (${columns.join(',')})
    VALUES ${placeholders}
    ON CONFLICT (id) DO UPDATE SET
    ${columns.map(c => `${c} = EXCLUDED.${c}`).join(',')}
  `;

  await db.query(query, values.flat());
  return records.length;
}
```

### Error Handling & Retries

```typescript
async function withRetry<T>(
  fn: () => Promise<T>,
  options: { maxRetries: number; delay: number } = { maxRetries: 3, delay: 1000 }
): Promise<T> {
  let lastError: Error;

  for (let i = 0; i < options.maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;
      console.log(`Attempt ${i + 1} failed: ${error}. Retrying...`);
      await sleep(options.delay * Math.pow(2, i)); // Exponential backoff
    }
  }

  throw new Error(`Failed after ${options.maxRetries} attempts: ${lastError}`);
}
```

### Monitoring & Logging

```typescript
interface PipelineRun {
  id: string;
  startTime: Date;
  endTime?: Date;
  status: 'running' | 'success' | 'failed';
  recordsProcessed: number;
  errors: PipelineError[];
}

async function logPipelineRun(run: PipelineRun) {
  await metrics.put({
    metric: 'pipeline.run',
    value: run.recordsProcessed,
    tags: {
      pipeline: run.id,
      status: run.status,
    },
  });
}
```

### Airflow DAG Example

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta

default_args = {
    'owner': 'data-team',
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
}

with DAG(
    'daily_etl',
    default_args=default_args,
    schedule_interval='0 2 * * *',  # 2 AM daily
    start_date=datetime(2024, 1, 1),
) as dag:

    extract = PythonOperator(
        task_id='extract',
        python_callable=extract_data,
    )

    transform = PythonOperator(
        task_id='transform',
        python_callable=transform_data,
    )

    load = PythonOperator(
        task_id='load',
        python_callable=load_data,
    )

    extract >> transform >> load
```

### Best Practices

1. **Idempotency** - Running twice shouldn't cause issues
2. **Atomicity** - Each step succeeds or fails completely
3. **Monitoring** - Log everything, alert on failures
4. **Backfill** - Support historical data reprocessing
5. **Schema evolution** - Handle changing data schemas
