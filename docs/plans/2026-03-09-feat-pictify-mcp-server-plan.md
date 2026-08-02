---
title: "feat: Create @pictify/mcp-server"
type: feat
status: completed
date: 2026-03-09
deepened: 2026-03-09
---

# feat: Create @pictify/mcp-server

## Enhancement Summary

**Deepened on:** 2026-03-09
**Sections enhanced:** 8
**Research agents used:** TypeScript reviewer, Architecture strategist, Security sentinel, Performance oracle, Code simplicity reviewer, Agent-native reviewer, Pattern recognition specialist, Best-practices researcher, Framework-docs researcher, Agent-native architecture skill

### Key Improvements
1. Added MCP tool annotations (`readOnlyHint`, `destructiveHint`) per MCP spec 2025-06-18
2. Enhanced tool descriptions with activation criteria and parameter documentation (97% of MCP tools have description "smells" per research)
3. Added `outputSchema` support for structured content alongside text content
4. Specified image content return pattern (base64 MCP image blocks for rendered images)
5. Added tool count optimization guidance (keep under 20-25 tools for optimal AI accuracy)

### New Considerations Discovered
- MCP spec supports `title` field on tools (human-readable display name separate from tool name)
- Tool annotations provide `readOnlyHint`, `destructiveHint`, `openWorldHint` for trust/safety
- Research shows flat input schemas with `.describe()` on every field dramatically improve AI tool selection
- `isError: true` should be used for tool execution errors; protocol errors for invalid tool names
- SEP-1686 MCP Tasks pattern exists for async operations (future consideration for batch ops)

## Overview

Build an MCP (Model Context Protocol) server that wraps the Pictify REST API, enabling AI agents (Claude Desktop, Claude Code, Cursor, etc.) to generate images, GIFs, and PDFs programmatically. The project will be created at `/Users/suyashthakur/pictify-mcp` and published as `@pictify/mcp-server` on npm.

## Problem Statement / Motivation

Pictify has 43 REST API endpoints but no MCP server implementation. The existing documentation at `docs-pictify/agent-integration/mcp-server.mdx` describes the desired interface but the `@pictify/mcp-server` npm package does not exist. AI agents need an MCP server to interact with Pictify's rendering capabilities natively.

## Proposed Solution

Create a TypeScript MCP server using `@modelcontextprotocol/sdk` v1.x with stdio transport. The server exposes ~20 tools covering image/GIF/PDF generation, template management, batch operations, and experiments. It uses a thin HTTP client (not the existing `@pictify/sdk` which has a `/v1/` path prefix bug) with retry logic and structured error handling.

## Technical Approach

### Architecture

```
pictify-mcp/
  src/
    index.ts              # Entry point, server setup, stdio transport
    api-client.ts         # Thin HTTP client for Pictify API
    tools/
      images.ts           # Image generation + screenshot tools
      gifs.ts             # GIF creation + capture tools
      pdfs.ts             # PDF generation tools
      templates.ts        # Template CRUD + render tools
      batch.ts            # Batch render tools
      experiments.ts      # Experiment management tools
  package.json
  tsconfig.json
  README.md
  LICENSE
```

### Research Insights: Architecture

**Best Practices:**
- Keep each tool file under 300 LOC for maintainability
- Export a `registerTools(server: McpServer, client: PictifyClient)` function from each tool module
- Single responsibility: `api-client.ts` handles HTTP only, tool files handle MCP schema + response formatting only

**Tool Count Optimization:**
- Research shows AI model accuracy degrades significantly above 25-30 tools
- Cursor caps at 80 tools, Claude at 120, but accuracy drops well before those limits
- Our 27 tools is at the upper edge — consider merging `list_images`/`list_gifs` into a single `pictify_list_renders` tool if accuracy issues arise
- Use MCP tool `title` field for human-readable names and keep `name` concise

### Key Design Decisions

1. **Base URL**: `https://api.pictify.io` (NO `/v1/` prefix — the API has no URL versioning)
2. **HTTP Client**: Custom thin client using native `fetch` (Node 18+), NOT the existing `@pictify/sdk` (has `/v1/` path bug and only covers 6 of 43 endpoints)
3. **Error Handling**: Return structured error content with `isError: true` (not thrown exceptions) so AI agents can self-correct
4. **Transport**: Stdio only (standard for local MCP servers)
5. **Tool Naming**: `pictify_{verb}_{resource}` pattern (e.g., `pictify_create_image`, `pictify_list_templates`)
6. **Logging**: `console.error()` only (stdout reserved for stdio transport JSON-RPC)
7. **Tool Annotations**: Use MCP spec annotations (`readOnlyHint`, `destructiveHint`) on every tool
8. **Content Return**: Return image URLs as text + base64 image content blocks for rendered images

### Research Insights: Tool Description Quality

Per research (arxiv.org/html/2602.14878v2), 97.1% of MCP tool descriptions contain at least one "smell" that degrades AI tool selection. Augmented descriptions yield 5.85pp improvement in task success rates.

**Required description pattern for every tool:**
```typescript
server.tool(
  "pictify_create_image",
  // Description must include: purpose, when to use, what it returns, limitations
  "Generate a static image (PNG, JPEG, or WebP) from raw HTML/CSS content. " +
  "Use this when the user wants to create an image from scratch with custom HTML. " +
  "For rendering a saved template, use pictify_render_template instead. " +
  "Returns the image URL and metadata. Maximum dimensions: 4000x4000 pixels.",
  {
    html: z.string().describe("Complete HTML content to render as an image. Include inline CSS for styling."),
    width: z.number().min(1).max(4000).default(1200).describe("Image width in pixels (1-4000)"),
    height: z.number().min(1).max(4000).default(630).describe("Image height in pixels (1-4000)"),
    format: z.enum(["png", "jpeg", "webp"]).default("png").describe("Output image format"),
  },
  async (params) => { /* ... */ }
);
```

**Key principles:**
- Every Zod field MUST have `.describe()` — this is exposed to the AI and dramatically improves accuracy
- Use `.default()` for optional params so AI doesn't need to guess sensible values
- Use `.min()`/`.max()` constraints to prevent invalid requests before they hit the API
- Prefer flat schemas over nested objects — AI models handle flat params better
- Include "Use X instead" disambiguation in descriptions when tools overlap

### Implementation Phases

#### Phase 1: Project Scaffold + Core Tools (Priority)

**Tasks:**

- [ ] Initialize project at `/Users/suyashthakur/pictify-mcp`
  - `package.json` with `"type": "module"`, `"bin": { "pictify-mcp": "./build/index.js" }`
  - `tsconfig.json` targeting ES2022, Node16 module resolution
  - Dependencies: `@modelcontextprotocol/sdk@^1.27`, `zod@^3.25`
  - Dev dependencies: `@types/node@^22`, `typescript@^5.7`
  - Build script: `"build": "tsc && chmod 755 build/index.js"`
  - Prepublish: `"prepublishOnly": "npm run build"`
  - Inspector script: `"inspector": "npx @modelcontextprotocol/inspector node build/index.js"`
- [ ] Create `src/api-client.ts` — thin HTTP client
  - Bearer token auth from `PICTIFY_API_KEY` env var
  - Base URL from `PICTIFY_BASE_URL` env var (default: `https://api.pictify.io`)
  - Retry logic: 3 retries with exponential backoff for 5xx errors
  - Timeout: 60s (GIF capture can take 30s+)
  - User-Agent: `@pictify/mcp-server/{version}`
  - Returns typed responses; surfaces RFC 9457 error details
  - AbortController-based timeout handling
- [ ] Create `src/index.ts` — server entry point
  - Shebang: `#!/usr/bin/env node`
  - Validate `PICTIFY_API_KEY` exists at startup (fail fast with clear error to stderr)
  - Validate key format matches `pk_live_*` or `pk_test_*` pattern
  - Log to stderr if test key detected: "Using test API key — renders will be sandboxed"
  - Create `McpServer` with name `"pictify"` and version from package.json
  - Connect stdio transport
  - Graceful shutdown on SIGINT/SIGTERM
- [ ] Create `src/tools/images.ts` — 3 tools:
  - `pictify_create_image` — POST /image with `html`, `width`, `height`, `format`
    - Annotations: `{ readOnlyHint: false, destructiveHint: false, openWorldHint: false }`
  - `pictify_screenshot` — POST /image with `url`, `width`, `height`, `fullPage`
    - Annotations: `{ readOnlyHint: true, destructiveHint: false, openWorldHint: true }`
  - `pictify_list_images` — GET /image with `limit`, `offset`
    - Annotations: `{ readOnlyHint: true, destructiveHint: false }`
- [ ] Create `src/tools/gifs.ts` — 3 tools:
  - `pictify_create_gif` — POST /gif with `html`, `width`, `height`
  - `pictify_capture_gif` — POST /gif/capture with `url`, `width`, `height`, `quality`, `duration`
    - Tool description must warn: "This operation can take up to 30 seconds depending on duration"
  - `pictify_list_gifs` — GET /gif with `limit`, `offset`
- [ ] Create `src/tools/pdfs.ts` — 3 tools:
  - `pictify_render_pdf` — POST /pdf/render with `templateId`, `variables`, `options`
  - `pictify_render_multi_page_pdf` — POST /pdf/multi-page with `templateId`, `variableSets`, `options`
  - `pictify_list_pdf_presets` — GET /pdf/presets

**Acceptance Criteria:**
- [ ] `npm run build` compiles without errors
- [ ] Server starts with valid `PICTIFY_API_KEY` and fails with clear error without it
- [ ] All 9 tools appear in MCP Inspector with correct schemas
- [ ] Image generation returns URL in structured text content
- [ ] Errors return `isError: true` content blocks with RFC 9457 details
- [ ] All tool descriptions include purpose, when-to-use, return value, and limitations

#### Phase 1 Research Insights

**API Client Implementation Pattern:**
```typescript
// src/api-client.ts — recommended implementation
import { readFileSync } from "fs";
import { fileURLToPath } from "url";
import { dirname, join } from "path";

const __dirname = dirname(fileURLToPath(import.meta.url));
const pkg = JSON.parse(readFileSync(join(__dirname, "..", "package.json"), "utf-8"));

export class PictifyApiError extends Error {
  constructor(
    public status: number,
    public type: string,
    public title: string,
    public detail: string,
    public errors?: Array<{ field: string; message: string; code: string }>,
    public retryAfter?: number,
  ) {
    super(detail);
    this.name = "PictifyApiError";
  }
}

export class PictifyClient {
  private baseUrl: string;
  private apiKey: string;
  private maxRetries: number;
  private timeout: number;

  constructor(apiKey: string, baseUrl = "https://api.pictify.io") {
    this.apiKey = apiKey;
    this.baseUrl = baseUrl.replace(/\/+$/, ""); // Strip trailing slashes
    this.maxRetries = 3;
    this.timeout = 60_000;
  }

  async request<T>(method: string, path: string, body?: unknown): Promise<T> {
    let lastError: Error | undefined;

    for (let attempt = 0; attempt <= this.maxRetries; attempt++) {
      if (attempt > 0) {
        const delay = Math.pow(2, attempt - 1) * 1000; // 1s, 2s, 4s
        await new Promise((r) => setTimeout(r, delay));
      }

      const controller = new AbortController();
      const timeoutId = setTimeout(() => controller.abort(), this.timeout);

      try {
        const res = await fetch(`${this.baseUrl}${path}`, {
          method,
          headers: {
            "Authorization": `Bearer ${this.apiKey}`,
            "Content-Type": "application/json",
            "User-Agent": `@pictify/mcp-server/${pkg.version}`,
          },
          body: body ? JSON.stringify(body) : undefined,
          signal: controller.signal,
        });

        clearTimeout(timeoutId);

        if (!res.ok) {
          const errorBody = await res.json().catch(() => ({}));

          // Don't retry client errors
          if (res.status < 500) {
            throw new PictifyApiError(
              res.status,
              errorBody.type ?? "unknown",
              errorBody.title ?? res.statusText,
              errorBody.detail ?? "Request failed",
              errorBody.errors,
              res.headers.get("Retry-After")
                ? parseInt(res.headers.get("Retry-After")!, 10)
                : undefined,
            );
          }

          // Retry server errors
          lastError = new PictifyApiError(
            res.status,
            errorBody.type ?? "server_error",
            errorBody.title ?? "Server Error",
            errorBody.detail ?? `Server returned ${res.status}`,
          );
          continue;
        }

        return await res.json() as T;
      } catch (err) {
        clearTimeout(timeoutId);
        if (err instanceof PictifyApiError) throw err;
        lastError = err as Error;
        if ((err as Error).name === "AbortError") {
          throw new PictifyApiError(408, "timeout", "Request Timeout", "The request timed out after 60 seconds");
        }
        if (attempt === this.maxRetries) throw err;
      }
    }

    throw lastError;
  }

  get<T>(path: string, params?: Record<string, unknown>): Promise<T> {
    const qs = params ? "?" + new URLSearchParams(
      Object.entries(params)
        .filter(([, v]) => v !== undefined)
        .map(([k, v]) => [k, String(v)])
    ).toString() : "";
    return this.request("GET", `${path}${qs}`);
  }

  post<T>(path: string, body?: unknown): Promise<T> {
    return this.request("POST", path, body);
  }

  put<T>(path: string, body?: unknown): Promise<T> {
    return this.request("PUT", path, body);
  }

  del<T>(path: string): Promise<T> {
    return this.request("DELETE", path);
  }
}
```

**Tool Result Pattern — Return Image Content Blocks:**
```typescript
// For image generation tools, return both URL and inline image
async ({ html, width, height, format }) => {
  try {
    const result = await client.post<{ url: string }>("/image", { html, width, height, fileExtension: format });
    return {
      content: [
        { type: "text", text: `Image generated successfully.\n\nURL: ${result.url}` },
        // Optionally fetch and return inline for vision-capable models:
        // { type: "image", data: base64Data, mimeType: `image/${format}` },
      ],
    };
  } catch (error) {
    if (error instanceof PictifyApiError) {
      return {
        content: [{
          type: "text",
          text: `Error (${error.status}): ${error.title}\n${error.detail}` +
            (error.errors ? `\n\nValidation errors:\n${error.errors.map(e => `- ${e.field}: ${e.message}`).join("\n")}` : "") +
            (error.retryAfter ? `\n\nRetry after ${error.retryAfter} seconds.` : ""),
        }],
        isError: true,
      };
    }
    return {
      content: [{ type: "text", text: `Unexpected error: ${(error as Error).message}` }],
      isError: true,
    };
  }
}
```

**Security Insight — Input Sanitization:**
- The `html` parameter is passed directly to the Pictify API which renders it server-side. The MCP server itself doesn't execute HTML, so XSS is not a concern at the MCP layer.
- However, validate `url` parameters to prevent SSRF — only allow `http://` and `https://` schemes.
- Never log the API key value; only log its prefix (`pk_live_...` or `pk_test_...`).

#### Phase 2: Template + Batch Tools

**Tasks:**

- [ ] Create `src/tools/templates.ts` — 7 tools:
  - `pictify_list_templates` — GET /templates with `page`, `limit`, `sort`, `outputFormat`
    - Annotations: `{ readOnlyHint: true }`
  - `pictify_get_template` — GET /templates/{uid}
    - Annotations: `{ readOnlyHint: true }`
  - `pictify_get_template_variables` — GET /templates/{uid}/variables
    - Annotations: `{ readOnlyHint: true }`
    - Description: "Get the variable definitions for a template. Call this before pictify_render_template to know what variables are available and their types."
  - `pictify_render_template` — POST /templates/{uid}/render with `variables`, `format`, `quality`
    - Description: "Render a saved template with variable substitutions. Use pictify_get_template_variables first to discover available variables. For rendering raw HTML without a template, use pictify_create_image instead."
  - `pictify_create_template` — POST /templates with `name`, `html`, `width`, `height`, `type`, `tags`
    - Annotations: `{ readOnlyHint: false, destructiveHint: false }`
  - `pictify_update_template` — PUT /templates/{uid}
    - Annotations: `{ readOnlyHint: false, destructiveHint: false }`
  - `pictify_delete_template` — DELETE /templates/{uid}
    - Annotations: `{ readOnlyHint: false, destructiveHint: true }`
- [ ] Create `src/tools/batch.ts` — 3 tools:
  - `pictify_batch_render` — POST /templates/{uid}/batch-render (returns batchId immediately, does NOT block)
    - Description: "Start a batch render job for a template with multiple variable sets (up to 100). Returns a batchId immediately. Use pictify_get_batch_results to poll for completion."
  - `pictify_get_batch_results` — GET /templates/batch/{batchId}/results
    - Description: "Check the status and results of a batch render job. Returns status (pending/processing/completed/failed), progress, and result URLs when complete."
  - `pictify_cancel_batch` — POST /templates/batch/{batchId}/cancel
    - Annotations: `{ destructiveHint: true }`

**Acceptance Criteria:**
- [ ] Template CRUD works end-to-end
- [ ] Batch render returns batchId, get_batch_results returns status + results
- [ ] Pagination parameters work correctly on list tools
- [ ] Delete tools have `destructiveHint: true` annotation

#### Phase 2 Research Insights

**Batch Operation Pattern:**
- Batch render returns HTTP 202 with a `batchId`. The MCP tool should return immediately with the batchId.
- The AI agent then calls `pictify_get_batch_results` to poll. The tool description should guide the agent on when to poll.
- Do NOT auto-poll inside the tool — this blocks the MCP tool call and may timeout.

**Template Parameter Naming:**
- POST /image uses `template` field
- POST /pdf/render uses `templateUid`
- POST /templates/{uid}/render uses `uid` in URL path
- **MCP tools use `templateId`** as the consistent user-facing name
- The tool handler maps `templateId` to the correct API field internally

#### Phase 3: Experiment Tools

**Tasks:**

- [ ] Create `src/tools/experiments.ts` — 8 tools:
  - `pictify_list_experiments` — GET /experiments/api with `type`, `status` filters
  - `pictify_create_experiment` — POST /experiments/api (simplified schema: name, type, slug, variants with id/name/weight/config)
    - Client-side validation: weights sum to 10000, slug 3-60 chars, no reserved words
    - Description: "Create a new A/B test experiment. Variant weights must sum to exactly 10000 (basis points). Reserved slugs: events, api, admin, health, status, pixel, track, sdk."
  - `pictify_get_experiment` — GET /experiments/api/{uid}
  - `pictify_update_experiment` — PUT /experiments/api/{uid}
  - `pictify_delete_experiment` — DELETE /experiments/api/{uid}
    - Annotations: `{ destructiveHint: true }`
  - `pictify_start_experiment` — POST /experiments/api/{uid}/start
    - Description: "Start a draft or paused experiment. Only valid for experiments in 'draft' or 'paused' status."
  - `pictify_pause_experiment` — POST /experiments/api/{uid}/pause
    - Description: "Pause a running experiment. Only valid for experiments in 'running' status."
  - `pictify_complete_experiment` — POST /experiments/api/{uid}/complete with `winnerVariantId`
    - Description: "Complete an experiment by declaring a winning variant. Only valid for 'running' or 'paused' experiments. Requires winnerVariantId."

**Acceptance Criteria:**
- [ ] Full experiment lifecycle works: create -> start -> pause -> complete
- [ ] Invalid state transitions return clear error messages with guidance
- [ ] Weight validation catches sum != 10000 before API call
- [ ] Reserved slug validation catches invalid slugs before API call

#### Phase 3 Research Insights

**State Machine Error Messages:**
When a state transition fails (409 Conflict from API), include guidance:
```typescript
// Example error message for invalid transition
"Cannot start experiment: current status is 'completed'. " +
"Completed experiments cannot be restarted. " +
"Valid transitions: draft→running, paused→running."
```

**Experiment Schema Simplification:**
Complex nested fields like `ConditionTree`, `BanditConfig`, and `VariantSchedule` should accept raw JSON objects in the MCP schema rather than deeply typed Zod schemas. AI models struggle with deeply nested required schemas. Example:
```typescript
variants: z.array(z.object({
  id: z.string().describe("Unique variant identifier"),
  name: z.string().describe("Human-readable variant name"),
  weight: z.number().min(0).max(10000).describe("Traffic weight in basis points (all weights must sum to 10000)"),
  config: z.record(z.unknown()).optional().describe("Variant-specific configuration as JSON object"),
})).min(2).describe("At least 2 variants required for an experiment"),
```

#### Phase 4: Polish + Publish

**Tasks:**

- [ ] Write README.md with:
  - Installation instructions for Claude Desktop, Claude Code, Cursor
  - Full tool reference table
  - Configuration options (env vars)
  - Example conversations
- [ ] Add `PICTIFY_DEBUG=true` env var support for verbose stderr logging
- [ ] Test with MCP Inspector: `npx @modelcontextprotocol/inspector node build/index.js`
- [ ] Verify package size < 2MB
- [ ] Publish to npm: `npm publish --access public`
- [ ] Update `docs-pictify/agent-integration/mcp-server.mdx` to reference the real package
- [ ] Fix base URL in `docs-pictify/agent-integration/mcp-server.mdx` line 197: change `https://api.pictify.io/v1` to `https://api.pictify.io`

**Acceptance Criteria:**
- [ ] `npx -y @pictify/mcp-server` works out of the box
- [ ] All tools pass MCP Inspector validation
- [ ] README covers all installation methods
- [ ] Package.json `files` field limits published package to `["build"]`

#### Phase 4 Research Insights

**README Structure (from ankane-style pattern):**
```markdown
# @pictify/mcp-server
Generate images, GIFs, and PDFs with AI agents using Pictify.

## Installation
[Claude Desktop / Claude Code / Cursor configs]

## Available Tools
[Table of all tools with descriptions]

## Configuration
| Variable | Description | Default |
[env var table]

## Examples
[2-3 conversation examples]

## Development
[Build/test/inspector instructions]
```

**Publishing Checklist:**
- Ensure `#!/usr/bin/env node` shebang in `src/index.ts`
- `chmod 755 build/index.js` in build script
- `"files": ["build"]` limits npm package to compiled output only
- Test with `npx @modelcontextprotocol/inspector` before every publish
- Use `npm pack --dry-run` to verify package contents before publishing

## Tool Reference (27 tools total)

| Tool | API Endpoint | Description | Annotations |
|------|-------------|-------------|-------------|
| `pictify_create_image` | POST /image | Generate image from HTML content | write |
| `pictify_screenshot` | POST /image | Capture screenshot of a URL | read, openWorld |
| `pictify_list_images` | GET /image | List generated images | read |
| `pictify_create_gif` | POST /gif | Create animated GIF from HTML with CSS animations | write |
| `pictify_capture_gif` | POST /gif/capture | Capture GIF from a live URL (up to 30s) | write, openWorld |
| `pictify_list_gifs` | GET /gif | List generated GIFs | read |
| `pictify_render_pdf` | POST /pdf/render | Generate single-page PDF from template | write |
| `pictify_render_multi_page_pdf` | POST /pdf/multi-page | Generate multi-page PDF from template | write |
| `pictify_list_pdf_presets` | GET /pdf/presets | List available PDF size presets | read |
| `pictify_list_templates` | GET /templates | List templates with pagination | read |
| `pictify_get_template` | GET /templates/{uid} | Get template details | read |
| `pictify_get_template_variables` | GET /templates/{uid}/variables | Get template variable definitions | read |
| `pictify_render_template` | POST /templates/{uid}/render | Render template with variables | write |
| `pictify_create_template` | POST /templates | Create a new template | write |
| `pictify_update_template` | PUT /templates/{uid} | Update an existing template | write |
| `pictify_delete_template` | DELETE /templates/{uid} | Delete a template | destructive |
| `pictify_batch_render` | POST /templates/{uid}/batch-render | Start batch render (returns batchId) | write |
| `pictify_get_batch_results` | GET /templates/batch/{batchId}/results | Get batch job status and results | read |
| `pictify_cancel_batch` | POST /templates/batch/{batchId}/cancel | Cancel a batch job | destructive |
| `pictify_list_experiments` | GET /experiments/api | List experiments | read |
| `pictify_create_experiment` | POST /experiments/api | Create an A/B test or smart link | write |
| `pictify_get_experiment` | GET /experiments/api/{uid} | Get experiment details | read |
| `pictify_update_experiment` | PUT /experiments/api/{uid} | Update experiment | write |
| `pictify_delete_experiment` | DELETE /experiments/api/{uid} | Delete experiment | destructive |
| `pictify_start_experiment` | POST /experiments/api/{uid}/start | Start experiment | write |
| `pictify_pause_experiment` | POST /experiments/api/{uid}/pause | Pause experiment | write |
| `pictify_complete_experiment` | POST /experiments/api/{uid}/complete | Complete experiment with winner | write |

### Excluded from v1 (lower AI agent relevance)

- Webhook CRUD (5 endpoints) — typically configured once, not by AI agents
- Binding CRUD (5 endpoints) — advanced feature, complex schema
- Experiment event tracking (POST /s/events) — requires separate `X-Write-Key` auth
- Click tracking redirect (GET /s/{slug}/click) — browser-only
- Tracking pixel (GET /s/{slug}/pixel.gif) — browser-only
- Experiment quota (GET /experiments/api/quota) — informational only
- Canvas image (POST /image/canvas) — requires FabricJS JSON, rarely used by AI agents

## Technical Considerations

### API Client Design (`src/api-client.ts`)

```typescript
// Key interface
interface PictifyClient {
  get<T>(path: string, params?: Record<string, unknown>): Promise<T>;
  post<T>(path: string, body?: unknown): Promise<T>;
  put<T>(path: string, body?: unknown): Promise<T>;
  del<T>(path: string): Promise<T>;
}
```

- Uses native `fetch` (Node 18+ required)
- Retries on 5xx with exponential backoff (delays: 1s, 2s, 4s)
- No retry on 4xx (client errors)
- Timeout: 60s default via AbortController
- All responses parsed as JSON
- Error responses extracted into `PictifyApiError` with RFC 9457 fields
- Query params filtered to exclude `undefined` values

### Error Handling Strategy

```typescript
// Tool handler pattern — errors returned as content, never thrown
async ({ html, width, height }) => {
  try {
    const result = await client.post('/image', { html, width, height });
    return {
      content: [{ type: "text", text: JSON.stringify(result, null, 2) }],
    };
  } catch (error) {
    if (error instanceof PictifyApiError) {
      return {
        content: [{
          type: "text",
          text: `Error (${error.status}): ${error.title}\n${error.detail}` +
            (error.errors ? `\n\nValidation errors:\n${error.errors.map(e => `- ${e.field}: ${e.message}`).join("\n")}` : "") +
            (error.retryAfter ? `\n\nRetry after ${error.retryAfter} seconds.` : ""),
        }],
        isError: true,
      };
    }
    return {
      content: [{ type: "text", text: `Unexpected error: ${(error as Error).message}` }],
      isError: true,
    };
  }
}
```

HTTP status code mapping:
| Status | MCP Behavior |
|--------|-------------|
| 2xx | Return structured content |
| 400 | Return error content with field-level validation details |
| 401 | Return error: "Invalid API key. Check PICTIFY_API_KEY environment variable." |
| 402 | Return error: "Quota exceeded. Upgrade your plan at pictify.io/dashboard." |
| 404 | Return error: "Resource not found. Verify the ID is correct." |
| 409 | Return error with valid state transitions guidance |
| 429 | Return error with Retry-After value: "Rate limited. Retry after N seconds." |
| 5xx | Retry 3 times with exponential backoff, then return error |

### Parameter Naming Resolution

The API has naming inconsistencies across endpoints:
- POST /image uses `template` field for template-based rendering
- POST /pdf/render uses `templateUid`
- POST /templates/{uid}/render uses `uid` in path
- POST /image uses `fileExtension` but template render uses `format`

**Decision**: MCP tools use `templateId` for template references and `format` for output format. The API client maps these to the correct field names per endpoint internally.

### Security Considerations

- **API Key Protection**: Never log the full API key. Only log the prefix for debugging: `pk_live_...xxxx`
- **URL Validation**: Validate `url` parameters in screenshot/capture tools — only allow `http://` and `https://` schemes to prevent SSRF
- **Input Validation**: Use Zod schemas with min/max constraints to reject invalid params before API calls
- **No HTML Execution**: The MCP server passes HTML to the Pictify API for server-side rendering — no XSS risk at the MCP layer
- **Environment Isolation**: API key is read from env var, never accepted as a tool parameter

### Performance Considerations

- **Connection Reuse**: Node 18+ `fetch` uses HTTP keep-alive by default — no special configuration needed
- **Timeout Strategy**: 60s default covers GIF capture (up to 30s) with margin. Consider per-tool timeouts if needed.
- **No Auto-Pagination**: List tools return one page at a time. AI agents can request more pages explicitly. This prevents accidentally fetching thousands of records.
- **Batch Non-Blocking**: Batch render tools return immediately with batchId. Polling is the AI agent's responsibility.
- **Package Size**: Target < 500KB published. No unnecessary dependencies beyond MCP SDK and Zod.

## Dependencies & Risks

| Risk | Mitigation |
|------|-----------|
| Base URL `/v1/` confusion | Default to `https://api.pictify.io` — matches OpenAPI spec |
| GIF capture timeouts (30s+) | Set 60s timeout; document in tool description |
| Batch operations are async | Return batchId immediately; provide separate results tool |
| Experiment schema complexity | Simplified input schemas; complex fields as raw JSON objects |
| SDK has path prefix bug | Don't use SDK; implement thin HTTP client |
| Too many tools degrades AI accuracy | 27 tools is at upper edge; monitor and merge if needed |
| Tool description "smells" | Follow augmented description pattern with purpose, activation, returns, limitations |

## Success Metrics

- All 27 tools pass MCP Inspector validation
- Package publishes to npm as `@pictify/mcp-server`
- `npx -y @pictify/mcp-server` starts successfully with valid API key
- End-to-end test: Claude Desktop generates an image via the MCP server
- All tool descriptions pass quality checklist (purpose, when-to-use, return value, limitations)

## Sources & References

### Internal References
- API specification: `/Users/suyashthakur/docs-pictify/openapi/v1.yaml`
- MCP server docs: `/Users/suyashthakur/docs-pictify/agent-integration/mcp-server.mdx`
- Node.js SDK reference: `/Users/suyashthakur/pictify-sdks/nodejs-sdk/src/client.ts`
- API overview: `/Users/suyashthakur/docs-pictify/api-reference/overview.mdx`
- Authentication: `/Users/suyashthakur/docs-pictify/authentication.mdx`

### External References
- MCP TypeScript SDK: https://github.com/modelcontextprotocol/typescript-sdk
- MCP Server build guide: https://modelcontextprotocol.io/docs/develop/build-server
- MCP Tools Specification (2025-06-18): https://modelcontextprotocol.io/specification/2025-06-18/server/tools
- MCP Tool Description Quality Research: https://arxiv.org/html/2602.14878v2
- REST to MCP Mapping Guide: https://www.scalekit.com/blog/map-api-into-mcp-tool-definitions
- MCP Best Practices: https://steipete.me/posts/2025/mcp-best-practices
- Publishing MCP Servers to npm: https://www.aihero.dev/publish-your-mcp-server-to-npm
- Tool Compatibility Layer (error rate reduction): https://mastra.ai/blog/mcp-tool-compatibility-layer
