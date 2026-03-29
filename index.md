---
layout: default
title: Home
---

# mdocUI

Generative UI for LLMs using Markdoc `{% raw %}{% %}{% endraw %}` tag syntax inline with markdown prose.

LLMs write natural markdown **and** drop interactive UI components in the same stream — charts, buttons, forms, tables, cards, and more. No custom DSL, no JSON blocks, no JSX confusion.

```
pnpm add @mdocui/core @mdocui/react
```

---

## What it looks like

An LLM writes this naturally:

```
The Q4 results show strong growth across all segments.

{% raw %}{% chart type="bar" labels=["Jan","Feb","Mar"] values=[120,150,180] /%}{% endraw %}

Revenue grew **12%** quarter-over-quarter.

{% raw %}{% callout type="info" title="Action Required" %}{% endraw %}
Review the pipeline before end of quarter.
{% raw %}{% /callout %}{% endraw %}

{% raw %}{% button action="continue" label="Show by region" /%}{% endraw %}
{% raw %}{% button action="continue" label="Export as PDF" /%}{% endraw %}
```

Prose flows instantly during streaming. Components render when their tags are complete.

---

## Syntax: Markdown + Markdoc Tags

mdocUI combines two syntaxes in a single stream:

- **[Markdown](https://commonmark.org/)** -- the universal prose format created by John Gruber (2004), standardized by [CommonMark](https://github.com/commonmark/commonmark-spec). Handles headings, bold, italic, lists, links, code blocks -- everything an LLM already knows how to write.
- **[Markdoc](https://github.com/markdoc/markdoc) `{% raw %}{% %}{% endraw %}` tags** -- the tag syntax from Stripe's Markdoc framework (2022, MIT). Markdoc extends Markdown with `{% raw %}{% %}{% endraw %}` custom tags for structured content.

mdocUI borrows **only the `{% raw %}{% %}{% endraw %}` tag syntax** from Markdoc. We do not use Markdoc's parser, runtime, compiler, schema system, or config layer. We built our own streaming parser from scratch, purpose-built for token-by-token LLM output.

### Tag forms

Self-closing (no body):

```
{% raw %}{% tagname attr="value" /%}{% endraw %}
```

With body content:

```
{% raw %}{% tagname attr="value" %}{% endraw %}
Body content here -- can include markdown or nested tags.
{% raw %}{% /tagname %}{% endraw %}
```

### Why `{% raw %}{% %}{% endraw %}` works for streaming

The character sequence `{% raw %}{%{% endraw %}` never appears in normal prose, standard markdown, or fenced code blocks. This makes it a reliable delimiter that a character-by-character streaming parser can detect without ambiguity -- no lookahead, no backtracking, no fragile heuristics.

The LLM writes both markdown **and** component tags in the same response. The parser separates them into prose nodes and component nodes as tokens arrive.

---

## Why mdocUI?

| Approach | Prose? | Components? | Streaming? |
|----------|--------|-------------|------------|
| Plain markdown | Yes | No | Yes |
| OpenUI Lang | No | Yes | Yes |
| JSON blocks in markdown | Yes | Yes | Fragile |
| JSX in markdown | Yes | Yes | Fragile |
| **mdocUI** | **Yes** | **Yes** | **Yes** |

Markdoc's `{% raw %}{% %}{% endraw %}` delimiters are unambiguous — they never appear in normal prose or code, making streaming parsing reliable.

---

## Packages

| Package | Description | Status |
|---------|-------------|--------|
| [`@mdocui/core`](https://github.com/mdocui/mdocui/tree/main/packages/core) | Streaming parser, tokenizer, component registry, prompt generator | Stable |
| [`@mdocui/react`](https://github.com/mdocui/mdocui/tree/main/packages/react) | React renderer, 24 default components, `useRenderer` hook | Stable |
| [`@mdocui/cli`](https://github.com/mdocui/mdocui/tree/main/packages/cli) | Scaffold, generate system prompts, preview | Stable |

---

## 24 Built-in Components

**Layout:** `stack` `grid` `card` `divider` `accordion` `tabs` `tab`

**Interactive:** `button` `button-group` `input` `textarea` `select` `checkbox` `toggle` `form`

**Data:** `chart` `table` `stat` `progress`

**Content:** `callout` `badge` `image` `code-block` `link`

All components render theme-neutral semantic HTML with `data-mdocui-*` attributes. They use `currentColor` and `inherit` — no hardcoded colors. They adapt to any light or dark theme automatically. Pass `classNames` to the Renderer for Tailwind or custom CSS, or swap in your own components entirely.

---

## How it works

```
LLM token stream
      |
  Tokenizer — character-by-character, IN_PROSE / IN_TAG / IN_STRING
      |
  StreamingParser — buffers incomplete tags, emits ASTNode[]
      |
  ComponentRegistry — validates props via Zod schemas
      |
  Renderer — maps AST to React components
      |
  Live UI
```

The core is framework-agnostic pure TypeScript. React adapter ships today. Vue, Svelte, and Angular adapters can follow the same pattern.

---

## Quick Start

### 1. Generate a system prompt

```typescript
import { generatePrompt } from '@mdocui/core'
import { createDefaultRegistry, defaultGroups } from '@mdocui/react'

const registry = createDefaultRegistry()
const systemPrompt = generatePrompt(registry, {
  preamble: 'You are a helpful assistant.',
  groups: defaultGroups,
})
```

### 2. Render streamed output

```tsx
import { Renderer, defaultComponents, useRenderer, createDefaultRegistry } from '@mdocui/react'

const registry = createDefaultRegistry()

function Chat() {
  const { nodes, isStreaming, push, done } = useRenderer({ registry })

  // Call push(chunk) as tokens arrive from your LLM
  // Call done() when the stream ends

  return (
    <Renderer
      nodes={nodes}
      components={defaultComponents}
      isStreaming={isStreaming}
      onAction={(event) => {
        if (event.action === 'continue') {
          sendMessage(event.label)
        }
      }}
    />
  )
}
```

### 3. Handle actions

```typescript
onAction={(event) => {
  switch (event.action) {
    case 'continue':
      // Send event.label as a new user message
      break
    case 'submit:formName':
      // event.formState has all field values
      break
    case 'open_url':
      // event.params.url has the URL
      break
  }
}}
```

---

## Links

- [Live Demo](https://mdocui.vercel.app)
- [GitHub Repository](https://github.com/mdocui/mdocui)
- [@mdocui/core README](https://github.com/mdocui/mdocui/tree/main/packages/core)
- [@mdocui/react README](https://github.com/mdocui/mdocui/tree/main/packages/react)

---

MIT License · Built by [pnutmath](https://github.com/pnutmath)
