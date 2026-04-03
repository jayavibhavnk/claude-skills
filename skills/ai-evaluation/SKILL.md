---
name: ai-evaluation
description: Evaluate AI systems - build test sets, measure quality, track regression, and A/B test models.
metadata:
  priority: 9
  docs:
    - "https://docs.braintrust.dev"
  pathPatterns:
    - "**/eval/**"
    - "**/test/**"
  bashPatterns:
    - '\beval\b'
    - '\bbenchmark\b'
  promptSignals:
    phrases:
      - "evaluate"
      - "benchmark"
      - "quality"
    anyOf:
      - "evaluation"
      - "testing"
      - "metrics"
---

## AI Evaluation

### Building Test Sets

```typescript
interface TestCase {
  id: string;
  input: string;
  expectedOutput: string;
  metadata?: {
    category: string;
    difficulty: 'easy' | 'medium' | 'hard';
    tags: string[];
  };
}

// Curated test set
const testCases: TestCase[] = [
  {
    id: 'math-1',
    input: 'What is 2+2?',
    expectedOutput: '4',
    metadata: { category: 'math', difficulty: 'easy' },
  },
  {
    id: 'code-1',
    input: 'Write a function to reverse a string in Python',
    expectedOutput: 'def reverse_string(s): return s[::-1]',
    metadata: { category: 'coding', difficulty: 'medium' },
  },
];

// LLM-as-judge test case
const judgmentPrompt = `
You are an expert evaluator. Judge if the response answers the query accurately.

Query: {input}
Response: {response}
Expected: {expected}

Respond with: PASS | PARTIAL | FAIL
`;
```

### Metrics

```typescript
// Exact match (for factual QA)
function exactMatch(response: string, expected: string): boolean {
  return response.trim().toLowerCase() === expected.trim().toLowerCase();
}

// Substring match
function containsMatch(response: string, expected: string): boolean {
  return response.toLowerCase().includes(expected.toLowerCase());
}

// Levenshtein distance (fuzzy match)
function editDistance(a: string, b: string): number {
  // Implementation
}

// ROUGE score (for summaries)
function rougeScore(candidate: string, reference: string): number {
  // ROUGE-1 (unigrams)
  const candidateWords = candidate.split(' ');
  const referenceWords = reference.split(' ');
  const overlap = candidateWords.filter(w => referenceWords.includes(w));
  return overlap.length / candidateWords.length;
}
```

### Automated Evaluation Pipeline

```typescript
interface EvaluationResult {
  testCase: TestCase;
  response: string;
  metrics: {
    exactMatch: boolean;
    containsMatch: boolean;
    latency: number;
    tokenCount: number;
  };
  judgment?: 'PASS' | 'PARTIAL' | 'FAIL';
}

async function evaluateModel(
  model: string,
  testCases: TestCase[]
): Promise<EvaluationResult[]> {
  const results: EvaluationResult[] = [];

  for (const testCase of testCases) {
    const start = Date.now();

    const { text, usage } = await generateText({
      model,
      prompt: testCase.input,
    });

    const latency = Date.now() - start;

    const result: EvaluationResult = {
      testCase,
      response: text,
      metrics: {
        exactMatch: exactMatch(text, testCase.expectedOutput),
        containsMatch: containsMatch(text, testCase.expectedOutput),
        latency,
        tokenCount: usage.totalTokens,
      },
    };

    // LLM judge evaluation
    const judgment = await generateText({
      model: 'openai/gpt-5.4',
      prompt: judgmentPrompt
        .replace('{input}', testCase.input)
        .replace('{response}', text)
        .replace('{expected}', testCase.expectedOutput),
    });

    result.judgment = parseJudgment(judgment.text);

    results.push(result);
  }

  return results;
}
```

### A/B Testing Models

```typescript
interface ABTest {
  id: string;
  variantA: string;  // Model A name
  variantB: string;  // Model B name
  testCases: TestCase[];
  resultsA: EvaluationResult[];
  resultsB: EvaluationResult[];
}

async function runABTest(
  variantA: string,
  variantB: string,
  testCases: TestCase[]
): Promise<{ winner: string; confidence: number }> {
  const [resultsA, resultsB] = await Promise.all([
    evaluateModel(variantA, testCases),
    evaluateModel(variantB, testCases),
  ]);

  const scoreA = averageScore(resultsA);
  const scoreB = averageScore(resultsB);

  const n = testCases.length;
  const z = (scoreA - scoreB) / Math.sqrt((scoreA * (1 - scoreA) + scoreB * (1 - scoreB)) / n);

  const confidence = 1 - normalCDF(Math.abs(z));

  return {
    winner: scoreA > scoreB ? variantA : variantB,
    confidence,
  };
}
```

### Tracking Over Time

```typescript
interface MetricSnapshot {
  date: Date;
  model: string;
  testSet: string;
  metrics: {
    passRate: number;
    avgLatency: number;
    avgTokens: number;
  };
}

// Track metrics over time
async function trackMetrics(
  model: string,
  testSet: string
): Promise<MetricSnapshot> {
  const results = await evaluateModel(model, getTestSet(testSet));

  return {
    date: new Date(),
    model,
    testSet,
    metrics: {
      passRate: results.filter(r => r.judgment === 'PASS').length / results.length,
      avgLatency: average(results.map(r => r.metrics.latency)),
      avgTokens: average(results.map(r => r.metrics.tokenCount)),
    },
  };
}
```

### Regression Detection

```typescript
async function detectRegression(
  model: string,
  currentResults: EvaluationResult[],
  baselineResults: EvaluationResult[]
): Promise<{ hasRegression: boolean; changes: string[] }> {
  const changes: string[] = [];

  for (const current of currentResults) {
    const baseline = baselineResults.find(b => b.testCase.id === current.testCase.id);

    if (baseline && current.judgment !== baseline.judgment) {
      if (
        (baseline.judgment === 'PASS' && current.judgment !== 'PASS') ||
        (baseline.judgment === 'PARTIAL' && current.judgment === 'FAIL')
      ) {
        changes.push(
          `Regression on ${current.testCase.id}: ${baseline.judgment} → ${current.judgment}`
        );
      }
    }
  }

  return {
    hasRegression: changes.length > 0,
    changes,
  };
}
```

### Evals Best Practices

1. **Diverse test set** - Cover edge cases and common scenarios
2. **Curated + synthetic** - Mix human-labeled with LLM-generated
3. **Track over time** - Monitor quality as models evolve
4. **Regression tests** - Catch degradation before shipping
5. **Human evaluation** - Spot-check automated judgments
6. **Real inputs** - Use production traffic patterns
7. **Multiple metrics** - No single number tells full story
