# Observability in Xheal: A Layman's Guide

This document explains how **Observability** (specifically using **OpenTelemetry**) is implemented in Xheal. Think of this as the "Flight Recorder" or "Black Box" for your automated tests. When something goes wrong or when the AI fixes a test, we want to know exactly *what* happened, *why* it happened, and *how* it was fixed.

This guide walks through the code file-by-file, following the journey of a self-healing attempt.

---

## 1. The Setup: Installing the Recorder (`src/utils/telemetry.ts`)

Just like a flight recorder needs to be installed and turned on before a flight, we need to set up our telemetry system. This file handles the initialization.

### `initTelemetry()`
This function is the "ON" switch. It checks if you have enabled telemetry (via `OTEL_ENABLED=true`) and, if so, starts recording.

*   **NodeTracerProvider**: Think of this as the engine that powers the recorder. It manages all the data collection.
*   **SimpleSpanProcessor**: This processes each "event" (called a **Span**) as it happens.
*   **JsonFileSpanExporter**: This is our custom writer. Instead of sending data to a complex server (like Jaeger), it simply writes every event into a local file called `xheal-traces.json`. This makes it easy for you to see what happened without setting up extra infrastructure.

**Key Concept:** A **Span** is a single unit of work, like "Click Button" or "Ask AI". Spans can have children, creating a tree structure that shows the full story.

```typescript
// src/utils/telemetry.ts

export function initTelemetry() {
  // 1. Check if the "ON" switch is flipped
  if (process.env.OTEL_ENABLED !== 'true') return;

  // 2. Set up the engine (Provider) with our custom writer (Exporter)
  const provider = new NodeTracerProvider({
    resource: resourceFromAttributes({
      [SemanticResourceAttributes.SERVICE_NAME]: 'xheal', // Name of our service
    }),
    spanProcessors: [new SimpleSpanProcessor(new JsonFileSpanExporter())]
  });

  // 3. Start the engine!
  provider.register();
}
```

---

## 2. The Journey Begins: The Main Event (`src/client/healing-client.ts`)

This is where the "flight" actually starts. When a test fails and Xheal steps in to fix it, we start a new **Root Span**. This span wraps the entire healing process from start to finish.

### `heal()`
This function is called when your test fails.

1.  **`tracer.startActiveSpan('xheal.heal', ...)`**: This starts the main chapter of our story. We name it `xheal.heal`.
2.  **`span.setAttribute(...)`**: We add important details (Attributes) to this chapter title, such as:
    *   `xheal.original_selector`: What failed? (e.g., `#login-btn`)
    *   `xheal.gherkin_step`: What was the test trying to do? (e.g., "Click login button")
    *   `test.framework`: Are we using Playwright or TestCafe?
3.  **`span.end()`**: Once the healing is done (success or failure), we close the chapter.

```typescript
// src/client/healing-client.ts

public async heal(page, errorMessage, gherkinStep) {
  // 1. Start the main chapter "xheal.heal"
  return tracer.startActiveSpan('xheal.heal', async (span) => {
    try {
      // 2. Add details to the chapter
      span.setAttribute('xheal.original_selector', errorMessage);
      span.setAttribute('xheal.gherkin_step', gherkinStep);

      // ... Perform the healing magic ...
      
      // 3. If successful, mark it!
      span.setAttribute('xheal.success', true);
      span.setAttribute('xheal.healed_selector', result.locator);

    } catch (error) {
      // 4. If it failed, record the error
      span.recordException(error);
      span.setStatus({ code: SpanStatusCode.ERROR });
    } finally {
      // 5. Close the chapter
      span.end();
    }
  });
}
```

---

## 3. Gathering Facts: Reading the Map (`src/core/element-mapper.ts`)

Before the AI can fix anything, it needs to see the webpage. It needs a map of all the elements (buttons, inputs, etc.). This process is called **Context Extraction**.

### `extractComprehensiveDOMData()`
This function reads the webpage and creates a list of all elements.

1.  **`tracer.startActiveSpan('xheal.extract_context', ...)`**: This starts a *sub-chapter* called `xheal.extract_context`. It is a child of `xheal.heal`.
2.  **`span.setAttribute(...)`**: We record details about the page:
    *   `dom.element_count`: How many elements did we find? (e.g., 500)
    *   `dom.interactive_count`: How many are clickable? (e.g., 20)
    *   `dom.extraction_time`: How long did it take to read the page? (e.g., 150ms)

This helps us spot performance issues. If reading the page takes 10 seconds, we know something is wrong here!

```typescript
// src/core/element-mapper.ts

export async function extractComprehensiveDOMData(page) {
  // 1. Start sub-chapter "xheal.extract_context"
  return tracer.startActiveSpan('xheal.extract_context', async (span) => {
    const startTime = Date.now();
    
    // ... Read the DOM (Webpage) ...

    // 2. Record stats
    span.setAttribute('dom.element_count', elements.length);
    span.setAttribute('dom.extraction_time', Date.now() - startTime);
    
    // 3. Close sub-chapter
    span.end();
    return data;
  });
}
```

---

## 4. Consulting the Expert: Asking the AI (`src/communication/openai-provider.ts`)

Now we have the map (DOM data), we send it to the AI (OpenAI, Gemini, etc.) and ask for a solution.

### `callModel()`
This function sends the prompt to the AI.

1.  **`tracer.startActiveSpan('xheal.llm_request', ...)`**: This starts a sub-chapter called `xheal.llm_request`.
2.  **`span.setAttribute(...)`**: We record details about the conversation:
    *   `llm.provider`: Who did we ask? (e.g., `openai`)
    *   `llm.model`: Which brain did we use? (e.g., `gpt-4o`)
    *   `llm.usage.total_tokens`: How much did this question cost? (Tokens = Money)

This is crucial for **Cost Analysis**. You can see exactly how much each healing attempt costs.

```typescript
// src/communication/openai-provider.ts

async callModel(prompt) {
  // 1. Start sub-chapter "xheal.llm_request"
  return tracer.startActiveSpan('xheal.llm_request', async (span) => {
    span.setAttribute('llm.provider', 'openai');
    
    // ... Call OpenAI API ...

    // 2. Record the cost
    span.setAttribute('llm.usage.total_tokens', usage.total_tokens);
    
    // 3. Close sub-chapter
    span.end();
    return response;
  });
}
```

---

## 5. Testing the Solution: Verifying the Fix (`src/core/element-mapper.ts`)

The AI gives us a suggestion (e.g., "Use the button with ID `#submit-btn`"). But we can't just trust it; we have to verify it works on the live page.

### `testHealedSelector()`
This function tries to find the element using the AI's suggestion.

1.  **`tracer.startActiveSpan('xheal.verify_selector', ...)`**: This starts a sub-chapter called `xheal.verify_selector`.
2.  **`span.setAttribute(...)`**: We record the outcome:
    *   `xheal.verify_result`: Did it work? (`success` or `failed`)
    *   If it failed, we record *why* (e.g., "Element not found").

```typescript
// src/core/element-mapper.ts

async testHealedSelector(page, suggestion) {
  // 1. Start sub-chapter "xheal.verify_selector"
  return tracer.startActiveSpan('xheal.verify_selector', async (span) => {
    
    // ... Try to find the element ...
    
    if (found) {
       span.setAttribute('xheal.verify_result', 'success');
       return { success: true };
    } else {
       span.setAttribute('xheal.verify_result', 'failed');
       return { success: false };
    }
    
    // 2. Close sub-chapter
    span.end();
  });
}
```

---

## Summary: The Full Picture

When you put it all together, you get a **Trace** that looks like a waterfall. It tells the complete story of a healing attempt:

1.  **`xheal.heal`** (Root) - "Started healing attempt for 'Click Login'"
    *   **`xheal.extract_context`** (Child) - "Read page structure (took 200ms)"
    *   **`xheal.llm_request`** (Child) - "Asked GPT-4o for help (cost 500 tokens)"
    *   **`xheal.verify_selector`** (Child) - "Tested suggestion '#login-btn' -> Success!"

By looking at `xheal-traces.json`, you can see exactly where time was spent, how much it cost, and why decisions were made.
