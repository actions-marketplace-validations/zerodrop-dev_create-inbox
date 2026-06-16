# zerodrop/create-inbox

> GitHub Action — generate a disposable email inbox for CI testing

**[Documentation](https://docs.zerodrop.dev)** · [GitHub](https://github.com/zerodrop-dev) · [npm](https://npmjs.com/package/zerodrop-client)

No Docker. No SMTP config. No signup required.

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-ZeroDrop%20Create%20Inbox-green?logo=github)](https://github.com/marketplace/actions/zerodrop-create-inbox)
[![Test Action](https://github.com/zerodrop-dev/create-inbox/actions/workflows/test.yml/badge.svg)](https://github.com/zerodrop-dev/create-inbox/actions/workflows/test.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## What it does

Generates a disposable `@zerodrop-sandbox.online` inbox as a CI step and exposes it as an output variable. Your tests use the inbox to catch real emails — password resets, magic links, verification codes — without any infrastructure.

Each inbox is isolated per-run — parallel matrix builds work out of the box with zero cross-test contamination.

```yaml
- name: Generate test inbox
  id: inbox
  uses: zerodrop-dev/create-inbox@v1

- name: Run tests
  run: npx playwright test
  env:
    TEST_INBOX: ${{ steps.inbox.outputs.inbox }}
```

---

## Usage

### Basic (free sandbox)

```yaml
steps:
  - uses: zerodrop-dev/create-inbox@v1
    id: inbox

  - run: npx playwright test
    env:
      TEST_INBOX: ${{ steps.inbox.outputs.inbox }}
```

### With Workspace API key

```yaml
steps:
  - uses: zerodrop-dev/create-inbox@v1
    id: inbox
    with:
      api_key: ${{ secrets.ZERODROP_API_KEY }}

  - run: npx playwright test
    env:
      TEST_INBOX: ${{ steps.inbox.outputs.inbox }}
```

---

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `api_key` | No | `''` | ZeroDrop Workspace API key. Omit for free sandbox mode. |

## Outputs

| Output | Description | Example |
|--------|-------------|---------|
| `inbox` | Full email address | `swift-x7k2m@zerodrop-sandbox.online` |
| `inbox_name` | Inbox name without domain | `swift-x7k2m` |

---

## In your test

```typescript
import { ZeroDrop } from 'zerodrop-client';

const mail = new ZeroDrop();

// Use the inbox injected by the Action, or generate one locally
const inbox = process.env.TEST_INBOX ?? mail.generateInbox();

// waitForLatest uses SSE by default — sub-second delivery in CI
const email = await mail.waitForLatest(inbox, { timeout: 15000 });
const link = email.body.match(/https?:\/\/\S+verify\S+/)?.[0];
await page.goto(link);
```

---

## Security

The inbox generation runs entirely locally — no network request is made during the Action step. The generated address is a random string; no data leaves the runner until your tests poll the ZeroDrop API.

**Supply chain hardening — pin to a commit SHA:**

```yaml
# Floating tag (convenient)
uses: zerodrop-dev/create-inbox@v1

# Pinned to specific commit (recommended for production)
uses: zerodrop-dev/create-inbox@8706a59  # v1.0.0 initial release
```

The edge worker that receives and stores emails is fully open source:
→ [zerodrop-dev/zerodrop-worker](https://github.com/zerodrop-dev/zerodrop-worker)

---

## Free tier limits

- Shared domain (`zerodrop-sandbox.online`)
- AI spam filtering
- 30-minute email TTL
- No signup required

## Workspaces

For teams who need custom domains, longer retention, and API keys:
→ [zerodrop.dev](https://zerodrop.dev)

---

## License

MIT
