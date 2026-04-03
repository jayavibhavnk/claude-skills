---
name: firecrawl-scraping
description: Firecrawl web scraping and content extraction - crawl sites, extract structured data, and handle JavaScript-rendered pages.
metadata:
  priority: 8
  docs:
    - "https://docs.firecrawl.dev"
  pathPatterns:
    - "**/scraping/**"
    - "**/crawl/**"
  bashPatterns:
    - '\bfirecrawl\b'
    - '\bscraping\b'
  promptSignals:
    phrases:
      - "firecrawl"
      - "web scraping"
      - "crawl website"
    anyOf:
      - "scrape"
      - "crawl"
      - "extract.*web"
---

## Firecrawl Scraping

### What is Firecrawl?

Firecrawl is a web scraping tool that handles JavaScript-rendered pages and extracts structured content.

```typescript
import Firecrawl from '@firecrawl/sdk';

const firecrawl = new Firecrawl('your-api-key');

// Scrape a single page
const response = await firecrawl.scrapeUrl('https://example.com', {
  formats: ['markdown', 'html'],
});

// Crawl a website
const crawl = await firecrawl.crawlUrl('https://example.com', {
  maxDepth: 2,
  limit: 100,
});
```

### Page Scraping

```typescript
// Extract content from a page
const page = await firecrawl.scrapeUrl('https://example.com/blog/post', {
  formats: ['markdown', 'html', 'links'],
});

// Access extracted content
console.log(page.data.markdown); // Clean markdown content
console.log(page.data.html);     // Original HTML
console.log(page.data.links);    // All links on page
```

### Website Crawling

```typescript
// Start a crawl job
const job = await firecrawl.crawlUrl('https://example.com', {
  maxDepth: 2,
  limit: 50,
  scrapeOptions: {
    formats: ['markdown'],
  },
});

// Poll for completion
while (job.status !== 'completed') {
  await new Promise(r => setTimeout(r, 5000));
  const status = await firecrawl.getCrawlStatus(job.id);
  console.log(`Progress: ${status.completed}/${status.total}`);
}

// Get all discovered pages
for (const page of status.data) {
  console.log(page.url, page.markdown?.substring(0, 100));
}
```

### Structured Data Extraction

```typescript
// Extract with schema
const response = await firecrawl.scrapeUrl('https://example.com/products', {
  formats: ['markdown'],
  extract: {
    schema: {
      products: [{
        name: 'string',
        price: 'number',
        description: 'string',
      }],
    },
  },
});

console.log(response.data.products);
```

### JavaScript Rendering

```typescript
// Wait for JavaScript to render
const page = await firecrawl.scrapeUrl('https://example.com/spa', {
  formats: ['markdown'],
  pageOptions: {
    waitForSelector: '.content-loaded', // Wait for element
    timeout: 30000,                     // 30 second timeout
  },
});
```

### Error Handling

```typescript
try {
  const page = await firecrawl.scrapeUrl('https://example.com');
} catch (error) {
  if (error.status === 404) {
    console.log('Page not found');
  } else if (error.status === 403) {
    console.log('Access forbidden - try adding wait time');
  } else if (error.status === 429) {
    console.log('Rate limited - implement backoff');
  }
}
```

### Best Practices

1. **Respect robots.txt** - Check before crawling
2. **Add delays** - Rate limit your requests
3. **Handle errors** - Implement retry logic
4. **Use selectors** - Wait for dynamic content
5. **Monitor usage** - Track API credits
6. **Cache results** - Store scraped content
7. **User agent** - Set appropriate headers
