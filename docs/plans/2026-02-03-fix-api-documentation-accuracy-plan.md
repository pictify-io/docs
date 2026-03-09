---
title: "Fix API Documentation Accuracy"
type: fix
date: 2026-02-03
---

# Fix API Documentation Accuracy

## Enhancement Summary

**Deepened on:** 2026-02-03
**Sections enhanced:** 7 phases
**Research agents used:** OpenAPI best practices researcher, Mintlify patterns researcher, Architecture strategist, Simplicity reviewer, SDK consistency analyzer, Context7 documentation

### Key Improvements

1. **Inverted Implementation Order**: Update OpenAPI spec FIRST, then MDX docs (not vice versa) - single source of truth prevents drift
2. **Consolidated Phases**: Combined 7 phases into 3 focused work units for efficiency
3. **Skip Internal Endpoints**: Health checks, thumbnail regeneration, expression helpers are internal - don't document
4. **RFC 9457 Error Format**: Use Problem Details standard for error responses

### Research Insights

**OpenAPI Best Practices:**
- Use `$ref` for reusable schemas (already present in v1.yaml)
- Add `example` values to every field for better documentation
- Use `oneOf`/`anyOf` for polymorphic types (like dataSource.credentials)
- Mark required fields explicitly with `required` array

**Mintlify Patterns:**
- Use `<ParamField>` for parameter documentation (not raw tables)
- Use `<ResponseField>` for response documentation
- Use `<CodeGroup>` for multi-language examples
- Use `<Note>` and `<Warning>` for callouts
- Use `openapi: get /path` frontmatter to auto-generate from spec

**SDK Naming Conventions:**
| Language | Convention | Example |
|----------|------------|---------|
| Node.js | camelCase | `templateUid`, `variableSets` |
| Python | snake_case | `template_uid`, `variable_sets` |
| Go | PascalCase | `TemplateUID`, `VariableSets` |
| Ruby | snake_case | `template_uid`, `variable_sets` |

**Architecture Decision:**
The OpenAPI spec (`openapi/v1.yaml`) already has CORRECT schemas for Bindings API. The MDX docs drifted from the spec. Fix strategy: Sync MDX to OpenAPI, not the other way around.

### Implementation Order (Revised)

1. **Work Unit 1: OpenAPI Spec** - Fix all schemas first (single source of truth)
2. **Work Unit 2: Core API Docs** - Update images, gifs, pdfs, bindings, templates, webhooks MDX files
3. **Work Unit 3: SDK Docs** - Sync Node.js, Python, Go, Ruby docs with naming conventions

### Endpoints to Skip (Internal)

- `GET /image/health` - Health check
- `GET /gif/health` - Health check
- `POST /templates/:uid/regenerate-thumbnail` - Admin tool
- `POST /templates/regenerate-thumbnails` - Admin tool
- `POST /templates/expression/*` - Expression helpers (document in concepts instead)

---

## Overview

The Pictify documentation at `/docs-pictify` contains significant inaccuracies compared to the actual API implementation in `/html-to-gif`. This plan details all discrepancies and the specific fixes required to make the documentation accurate and factual.

## Problem Statement

After thorough analysis comparing the actual API routes in `html-to-gif/routes/` against the documentation in `docs-pictify/`, the following critical issues were identified:

1. **Image API** documents 12+ parameters that don't exist in the actual implementation
2. **Bindings API** uses completely wrong field names (`dataUrl` vs `dataSource`, `dataMapping` vs `mapping`, `schedule` vs `refreshPolicy`)
3. **Response formats** documented don't match actual responses
4. **20+ endpoints** exist but aren't documented
5. **Authentication methods** are incorrectly documented for PDF endpoints
6. **Missing query parameters** for Templates filtering/sorting

## Technical Approach

### Phase 1: Image API Documentation Fixes

**Files to modify:**
- `/Users/suyashthakur/docs-pictify/api-reference/generation/images.mdx`
- `/Users/suyashthakur/docs-pictify/openapi/v1.yaml`

**Changes required:**

1. **Remove non-existent parameters from Image API:**
   - Remove: `format` (actual: `fileExtension`)
   - Remove: `quality` (not supported for images)
   - Remove: `deviceScaleFactor` (not in actual API)
   - Remove: `transparent` (not in actual API)
   - Remove: `fullPage` (not in actual API)
   - Remove: `waitForSelector` (not in actual API)
   - Remove: `waitForTimeout` (not in actual API)
   - Remove: `userAgent` (not in actual API)
   - Remove: `cookies` (not in actual API)
   - Remove: `headers` (not in actual API)

2. **Document actual Image API parameters:**
   ```javascript
   POST /image accepts:
   {
     url: string,              // Optional: URL to capture
     html: string,             // Optional: HTML to render
     width: number,            // Optional: image width
     height: number,           // Optional: image height
     template: string,         // Optional: template UID
     variables: object,        // Optional: template variables
     selector: string,         // Optional: CSS selector
     fileExtension: string     // Optional: 'png', 'jpg', 'jpeg', 'webp'
   }
   ```

3. **Fix Image API response format:**
   ```json
   // Current documentation (INCORRECT):
   {
     "url": "...",
     "id": "...",
     "width": 1200,
     "height": 630,
     "format": "png",
     "size": 45678,
     "createdAt": "..."
   }

   // Actual response (CORRECT):
   {
     "url": "https://cdn.pictify.io/...",
     "userStorageUrl": "...",      // Only if user storage configured
     "id": "img_abc123",
     "createdAt": "2026-01-29T10:30:00Z"
   }
   ```

4. **Document missing Image endpoints:**
   - `POST /image/canvas` - Generate from FabricJS canvas data
   - `POST /image/agent-screenshot` - AI-powered screenshot
   - `POST /image/agent-screenshot-stream` - SSE streaming version
   - `GET /image/page-content` - Get rendered HTML from URL
   - `GET /image/health` - Health check endpoint

### Phase 2: GIF API Documentation Fixes

**Files to modify:**
- `/Users/suyashthakur/docs-pictify/api-reference/generation/gifs.mdx`
- `/Users/suyashthakur/docs-pictify/openapi/v1.yaml`

**Changes required:**

1. **Fix GIF response structure (wrapped in `gif` object):**
   ```json
   // Current documentation (INCORRECT):
   {
     "url": "...",
     "id": "...",
     ...
   }

   // Actual response (CORRECT):
   {
     "gif": {
       "url": "https://cdn.pictify.io/...",
       "userStorageUrl": "...",
       "id": "gif_abc123",
       "width": 400,
       "height": 400,
       "animationLength": 1.0,
       "createdAt": "..."
     },
     "_meta": {
       "processingTime": 1234
     }
   }
   ```

2. **Fix `/gif/capture` parameters:**
   - `quality` is a string preset (`'low'`, `'medium'`, `'high'`), not an integer
   - Document actual quality preset effects:
     - `low`: 10 fps
     - `medium`: 15 fps
     - `high`: 24 fps

3. **Document missing GIF endpoints:**
   - `GET /gif/health` - Health check endpoint

### Phase 3: PDF API Documentation Fixes

**Files to modify:**
- `/Users/suyashthakur/docs-pictify/api-reference/generation/pdfs.mdx`
- `/Users/suyashthakur/docs-pictify/openapi/v1.yaml`

**Changes required:**

1. **Clarify authentication:**
   - `POST /pdf/render` - Uses cookie-based authentication (dashboard only)
   - `POST /pdf/multi-page` - Uses cookie-based authentication
   - `POST /pdf/from-fabric` - No authentication required
   - Note: API token auth may not work for PDF endpoints

2. **Fix response format (add `success` field):**
   ```json
   {
     "success": true,
     "url": "https://cdn.pictify.io/...",
     "userStorageUrl": "...",
     "pageCount": 1,
     "preset": "A4",
     "pageSize": { "width": 595, "height": 842 }
   }
   ```

3. **Document missing PDF endpoints:**
   - `GET /pdf/presets` - Get available PDF size presets
   - `POST /pdf/from-fabric` - Generate PDF from FabricJS data (no auth)
   - `POST /pdf/multi-from-fabric` - Multi-page from FabricJS array

### Phase 4: Bindings API Documentation Rewrite (CRITICAL)

**Files to modify:**
- `/Users/suyashthakur/docs-pictify/api-reference/bindings.mdx`
- `/Users/suyashthakur/docs-pictify/openapi/v1.yaml`

**This section requires complete rewrite. Current docs are fundamentally incorrect.**

**Wrong field names in current docs:**
| Current (WRONG) | Actual (CORRECT) |
|-----------------|------------------|
| `dataUrl` | `dataSource.url` |
| `dataMapping` | `mapping` |
| `schedule` (cron) | `refreshPolicy` |
| N/A | `dataSource.credentials` |
| N/A | `outputConfig` |

**Correct Bindings API structure:**
```javascript
POST /bindings accepts:
{
  templateId: string,              // Required: template UID
  name: string,                    // Optional: binding name
  dataSource: {                    // Required
    type: 'http' | 'webhook' | 'static',
    url: string,                   // URL to fetch data from
    method: 'GET' | 'POST',        // HTTP method
    headers: object,               // Custom headers
    body: any,                     // Request body (for POST)
    credentials: {                 // Authentication config
      type: 'api_key' | 'bearer_token' | 'basic_auth' | 'custom_header',
      value: string,               // Token/key value
      username: string,            // For basic_auth
      headerName: string           // For custom_header
    }
  },
  mapping: object,                 // Required: { templateVar: 'data.path' }
  defaults: object,                // Optional: fallback values
  refreshPolicy: {                 // Optional
    type: 'ttl' | 'etag' | 'webhook' | 'manual',
    ttlSeconds: number,            // 60-604800
    onError: 'serve_stale' | 'serve_error' | 'serve_default'
  },
  outputConfig: {                  // Optional
    format: string,
    quality: number,
    width: number,
    height: number
  }
}
```

**Document missing Bindings endpoints:**
- `POST /bindings/:uid/test` - Test binding with sample data
- `POST /bindings/:uid/refresh` - Manually trigger refresh
- `POST /bindings/:uid/pause` - Pause binding
- `POST /bindings/:uid/resume` - Resume binding

**Fix Bindings response structure:**
```json
{
  "binding": {
    "uid": "bind_abc123",
    "name": "GitHub Stats",
    "templateId": "tmpl_xyz",
    "dataSource": { ... },
    "mapping": { ... },
    "defaults": { ... },
    "refreshPolicy": { ... },
    "outputConfig": { ... },
    "status": "active",
    "renderUrl": "https://cdn.pictify.io/bindings/...",
    "webhookUrl": "https://api.pictify.io/webhooks/bind_abc123",
    "lastFetchAt": "...",
    "lastRenderAt": "...",
    "lastError": null,
    "errorCount": 0,
    "createdAt": "...",
    "updatedAt": "..."
  }
}
```

### Phase 5: Templates API Documentation Fixes

**Files to modify:**
- `/Users/suyashthakur/docs-pictify/api-reference/templates.mdx`
- `/Users/suyashthakur/docs-pictify/openapi/v1.yaml`

**Changes required:**

1. **Add missing query parameters for `GET /templates`:**
   ```
   ?page=1              // Page number (default: 1)
   &limit=12            // Results per page (default: 12, max: 100)
   &sort=newest         // 'newest', 'oldest', 'name'
   &outputFormat=all    // 'all', 'image', 'pdf'
   &hasDynamicLink=true // 'true', 'false'
   ```

2. **Document batch rendering CSV mode:**
   ```javascript
   // Mode 1: Direct variableSets
   POST /templates/:uid/batch-render
   {
     variableSets: [...],
     format: 'png',
     quality: 0.9,
     concurrency: 5
   }

   // Mode 2: CSV URL
   POST /templates/:uid/batch-render
   {
     csvUrl: 'https://example.com/data.csv',
     mappings: { templateVar: 'csvColumn' },
     format: 'png',
     quality: 0.9,
     concurrency: 5
   }
   ```

3. **Document Template object additional fields:**
   ```javascript
   {
     uid: string,
     name: string,
     html: string,
     fabricJSData: object,       // FabricJS canvas data
     width: number,
     height: number,
     outputFormat: 'image' | 'pdf',  // Output type
     hasDynamicLink: boolean,        // Has shareable link
     pdfPreset: string,              // PDF paper size
     pages: array,                   // Multi-page templates
     variableDefinitions: array,
     createdAt: string,
     updatedAt: string
   }
   ```

4. **Document missing Template endpoints:**
   - `GET /templates/search?q=query` - Search templates
   - `POST /templates/:uid/regenerate-thumbnail` - Regenerate single thumbnail
   - `POST /templates/regenerate-thumbnails` - Regenerate all thumbnails
   - `POST /templates/upload-csv` - Upload CSV for batch rendering
   - Expression endpoints (new section):
     - `POST /templates/expression/validate` - Validate expression syntax
     - `POST /templates/expression/test` - Test expression with variables
     - `POST /templates/expression/interpolate` - Interpolate text with variables
     - `GET /templates/expression/functions` - List available functions

### Phase 6: Webhooks API Documentation Fixes

**Files to modify:**
- `/Users/suyashthakur/docs-pictify/api-reference/webhooks.mdx`
- `/Users/suyashthakur/docs-pictify/openapi/v1.yaml`

**Changes required:**

1. **Add missing webhook events to OpenAPI schema:**
   - Currently documented: `render.completed`, `render.failed`, `binding.updated`, `binding.failed`
   - Verify if `batch.completed` and `batch.failed` are actually implemented (mentioned in docs but not in schema)

2. **Add missing platform options:**
   ```yaml
   platform:
     type: string
     enum: [zapier, make, n8n, pipedream, custom]
   ```

3. **Document missing endpoints:**
   - `GET /webhook-subscriptions/stats` - Get webhook statistics
   - `/api/webhook-subscriptions/*` - API token authenticated routes

4. **Document stats endpoint response:**
   ```json
   {
     "subscriptions": {
       "total": 15,
       "byEvent": {
         "render.completed": 8,
         "render.failed": 4,
         "binding.updated": 3
       },
       "byStatus": {
         "active": 12,
         "paused": 2,
         "failed": 1
       },
       "byPlatform": {
         "custom": 10,
         "zapier": 3,
         "make": 2
       }
     },
     "queue": { ... }
   }
   ```

### Phase 7: OpenAPI Specification Updates

**File to modify:**
- `/Users/suyashthakur/docs-pictify/openapi/v1.yaml`

**Summary of all OpenAPI changes:**

1. Fix `ImageRequest` schema - remove non-existent fields, rename `format` to `fileExtension`
2. Fix `ImageResponse` schema - remove `width`, `height`, `format`, `size`
3. Fix `GifResponse` schema - wrap in `gif` object, add `_meta`
4. Fix `PdfResponse` schema - add `success` field
5. Completely rewrite `Binding` schema with correct structure
6. Add missing paths for all undocumented endpoints
7. Add `n8n` and `pipedream` to webhook platform enum
8. Update `WebhookSubscription` schema
9. Add query parameters for templates listing

## Acceptance Criteria

### Functional Requirements

- [ ] All documented API parameters match actual implementation in `html-to-gif/routes/`
- [ ] All response formats match actual API responses
- [ ] All field names are correct (especially bindings API)
- [ ] All missing endpoints are documented
- [ ] OpenAPI spec validates without errors
- [ ] Code examples work when executed
- [ ] SDK documentation matches API changes

### Non-Functional Requirements

- [ ] Documentation follows Mintlify conventions
- [ ] Code examples provided in cURL, Node.js, Python at minimum
- [ ] Parameter tables are accurate and complete
- [ ] Response examples match actual API output

### Quality Gates

- [ ] Every documented parameter verified against route file
- [ ] Every response field verified against actual handler response
- [ ] OpenAPI spec passes `mintlify openapi-check`
- [ ] All links and references working

## Dependencies & Prerequisites

- Access to `/Users/suyashthakur/html-to-gif/routes/` for verification
- Mintlify documentation framework knowledge
- OpenAPI 3.1.0 specification knowledge

## Risk Analysis & Mitigation

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| SDK docs become out of sync | Medium | High | Update SDKs docs in same PR |
| Breaking changes for existing users | Low | Medium | Document as corrections, not changes |
| Missed discrepancy | Low | Medium | Cross-reference each route file |

## Implementation Order

1. **Bindings API** (most critical - completely wrong)
2. **Image API** (many incorrect parameters)
3. **GIF API** (response format wrong)
4. **PDF API** (auth method unclear)
5. **Templates API** (missing features)
6. **Webhooks API** (minor gaps)
7. **OpenAPI Spec** (consolidate all changes)
8. **SDK Documentation** (ensure consistency)

## Files to Modify

### Primary Documentation Files
- `api-reference/generation/images.mdx`
- `api-reference/generation/gifs.mdx`
- `api-reference/generation/pdfs.mdx`
- `api-reference/templates.mdx`
- `api-reference/bindings.mdx`
- `api-reference/webhooks.mdx`
- `openapi/v1.yaml`

### Secondary Files (SDK docs)
- `sdks/nodejs.mdx`
- `sdks/python.mdx`
- `sdks/go.mdx`
- `sdks/ruby.mdx`

### Concept Pages (verify accuracy)
- `concepts/templates.mdx`
- `concepts/expressions.mdx`
- `concepts/webhooks.mdx`

## References

### Internal References
- Image routes: `/Users/suyashthakur/html-to-gif/routes/image.js`
- GIF routes: `/Users/suyashthakur/html-to-gif/routes/gif.js`
- PDF routes: `/Users/suyashthakur/html-to-gif/routes/pdf.js`
- Template routes: `/Users/suyashthakur/html-to-gif/routes/template.js`
- Bindings routes: `/Users/suyashthakur/html-to-gif/routes/bindings.js`
- Webhook routes: `/Users/suyashthakur/html-to-gif/routes/webhook-subscriptions.js`

### Research Findings
- Comprehensive route analysis completed
- All discrepancies catalogued
- SpecFlow analysis completed with user flow mapping
