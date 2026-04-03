---
name: chrome-devtools
description: Chrome DevTools debugging - performance profiling, network analysis, memory leaks, and JavaScript debugging.
metadata:
  priority: 8
  docs:
    - "https://developer.chrome.com/docs/devtools/"
  pathPatterns:
    - "*.js"
    - "*.ts"
  bashPatterns:
    - '\bdevtools\b'
    - '\bdebug\b'
  promptSignals:
    phrases:
      - "devtools"
      - "debugging"
      - "performance"
    anyOf:
      - "devtools"
      - "chrome"
      - "debug"
---

## Chrome DevTools

### Console API

```javascript
// Console methods
console.log('Info');           // Basic log
console.warn('Warning');      // Warning
console.error('Error');       // Error
console.debug('Debug');       // Debug (hidden by default)

// Structured logging
console.table([
  { name: 'Alice', age: 25 },
  { name: 'Bob', age: 30 }
]);

console.group('Request');
console.log('URL:', url);
console.log('Method:', method);
console.groupEnd();

// Timing
console.time('fetch');
await fetch(url);
console.timeEnd('fetch');

// Assertions
console.assert(count > 0, 'Count should be positive');
```

### Debugger Commands

```javascript
// Breakpoints
debugger; // Programmatic breakpoint

// Conditional breakpoint
// Right-click breakpoint -> Edit condition
// condition: userId === '123'

// Logpoint (no pause)
// Right-click -> Edit logpoint
// expression: `User: ${user.name}`

// Watch expressions
// Add to Watch panel
user?.profile?.settings

// Call stack
// Use named functions for better stack traces
const myFunction = function myNamedFunction() { };
```

### Network Analysis

```javascript
// Copy as fetch
// Right-click request -> Copy -> Copy as fetch

// Network timing
// Resource timing API
const [entry] = performance.getEntriesByName(url);
console.log('DNS:', entry.domainLookupEnd - entry.domainLookupStart);
console.log('TCP:', entry.connectEnd - entry.connectStart);
console.log('TTFB:', entry.responseStart - entry.requestStart);
console.log('Download:', entry.responseEnd - entry.responseStart);

// Monitor requests
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log(entry.name, entry.duration);
  }
});
observer.observe({ entryTypes: ['resource'] });
```

### Memory Debugging

```javascript
// Heap snapshots
// 1. Take snapshot (Memory tab)
// 2. Compare snapshots to find leaks

// Allocation timeline
// Track object allocations over time

// Memory leaks patterns
// Common causes:
- Detached DOM nodes still referenced
- Closures holding references
- Event listeners not removed
- Timers not cleared

// Example leak detection
function createLeak() {
  const data = new LargeObject();
  window.handler = () => console.log(data); // Leak!
}

function fixLeak() {
  window.handler = null; // Clean up
}
```

### Performance Profiling

```javascript
// User timing API
performance.mark('operation-start');
// ... operation ...
performance.mark('operation-end');
performance.measure('Operation', 'operation-start', 'operation-end');

// Long tasks
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (entry.duration > 50) {
      console.warn(`Long task: ${entry.duration}ms`);
    }
  }
});
observer.observe({ entryTypes: ['longtask'] });

// Frame rate
performance.getEntriesByType('paint');
// Look for 'first-contentful-paint'
```

### JavaScript Debugging

```javascript
// Async debugging
async function fetchData() {
  try {
    const response = await fetch(url);
    debugger; // Check response here
    return await response.json();
  } catch (error) {
    debugger; // Check error here
  }
}

// Promise debugging
// Enable "Async stack traces" in DevTools settings

// this binding
function showThis() {
  debugger; // Check this value in console
}
const obj = { showThis };
obj.showThis(); // this === obj
```

### Network Request Interception

```javascript
// Override with local files
// Network tab -> Right-click request -> Override content

// Fetch/XHR breakpoints
// Add pattern: **/api/users**

// Block requests
// Network tab -> Right-click -> Block request domain

// Cache control
// Disable cache checkbox in Network tab
```

### Rendering Debugging

```javascript
// Paint flashing
// Enable 'Show paint rectangles' in Rendering

// Scroll performance
// Enable 'Scrolling performance issues'

// Layer borders
// Enable 'Show layer borders'

// FPS meter
// Enable 'Show FPS meter'
```

### Useful Snippets

```javascript
// Clear all storage
Object.keys(localStorage).forEach(k => localStorage.removeItem(k));
sessionStorage.clear();

// Find event listeners
getEventListeners(document.getElementById('myElement'));

// Monitor events
monitorEvents(document.getElementById('myElement'));
monitorEvents(document.getElementById('myElement'), 'click');
unmonitorEvents(document.getElementById('myElement'));

// Copy to clipboard
copy(JSON.stringify(data, null, 2));
```

### Best Practices

1. **Use breakpoints** - Not just console.log
2. **Async stack traces** - Enable in settings
3. **Network timing** - Check TTFB and download
4. **Memory snapshots** - Compare to find leaks
5. **Performance profiler** - Find bottleneck functions
6. **Render tabs** - Debug layout and paint
7. **Console timestamps** - Enable for async debugging
