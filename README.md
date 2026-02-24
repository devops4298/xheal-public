# 🚀 Xheal: AI-Powered Self-Healing for Test Automation

![License](https://img.shields.io/npm/l/xheal)
![Version](https://img.shields.io/npm/v/xheal)
![AI Integrations](https://img.shields.io/badge/GenAI-OpenAI%20%7C%20Gemini%20%7C%20VertexAI-blue)
![Frameworks](https://img.shields.io/badge/Support-Playwright%20%7C%20Selenium%20%7C%20TestCafe-brightgreen)

**Xheal** is an enterprise-grade, universally compatible self-healing library that supercharges your existing test automation pipelines. Say goodbye to flaky tests and constant maintenance. Xheal intelligently repairs broken UI selectors and element interactions on the fly using cutting-edge Generative AI.

---

## 🌟 Why Xheal? The Unique Advantage

Unlike traditional self-healing tools that are tied to a single framework or rely on rudimentary DOM diffing, Xheal brings true **Agentic AI capabilities** directly into your CI/CD pipelines. 

### 1. 🤖 Unrivaled GenAI Integrations
We support the industry's most advanced LLMs out of the box. No vendor lock-in; choose the brain that powers your automation:
- **OpenAI (GPT-4/GPT-4o)**
- **Google Gemini (GenAI)**
- **Google Vertex AI (Enterprise-grade security)**

### 2. 🔌 Universal Framework Support
Xheal isn't just for one ecosystem. It seamlessly wraps around the tools your team already uses:
- **Playwright**
- **Selenium**
- **TestCafe**

### 3. 📊 Enterprise Observability & Telemetry
Every healing action is tracked, measured, and stored for complete transparency and compliance.
- **BigQuery Storage Integration:** Archive thousands of test runs for historical analysis.
- **OpenTelemetry Support:** Integrate self-healing metrics directly into DataDog, New Relic, or Grafana.
- **Rich Reporting:** Automatic generation of HTML, JSON, and Markdown reports for deep insights into what healed, why, and how much time was saved.

---

## 🚀 Quick Start

### Installation

```bash
npm install xheal
```

### Configuration

Set up your `.env` file with your preferred AI provider:

```env
# Choose your provider (openai, gemini, vertexai)
LLM_PROVIDER=openai

# If using OpenAI:
OPENAI_API_KEY=your-api-key

# If using Google Gemini:
GEMINI_API_KEY=your-api-key
```

### Basic Usage Example (Playwright)

```typescript
import { test, expect } from '@playwright/test';
import { HealingClient } from 'xheal';

const healer = new HealingClient();

test('Example Test with Xheal', async ({ page }) => {
    await page.goto('https://your-app.com');

    // Traditional way (breaks if ID changes):
    // await page.click('#submit-btn-v2'); 

    // With Xheal (Agentic Healing):
    await healer.click(page, 'Submit Application Button', '#submit-btn-v2');
});
```

---

## 📈 Observability & Reporting

Xheal isn't a black box. Our observability matrix ensures you always know what the AI is doing.

- **Traceability:** Every interaction is logged with precise tokens used and success rates.
- **Rich HTML Reports:** After execution, a beautifully rendered HTML report shows before/after DOM states.
- Read more about our Observability in the [Docs](./docs/Observability.md).

---

## 🤝 Community & Feedback

Xheal is currently expanding. We would love for the community, engineers, and QA leaders to try out the package and provide feedback. 

If you encounter issues or have feature requests, please [open an issue](https://github.com/your-org/xheal-public/issues) on our public repository.

---

*Transform your test flakiness into a competitive advantage with Xheal.*
