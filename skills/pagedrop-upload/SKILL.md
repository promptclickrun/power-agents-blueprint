---
name: pagedrop-upload
description: >
  Upload self-contained HTML files to PageDrop for shareable, time-limited links.
  No API key required. Supports private visibility and configurable TTL
  (1 hour, 1 day, 3 days, or single-view).
---

# Skill: PageDrop Upload

> Upload self-contained HTML files to PageDrop for shareable, time-limited links. No API key required. Supports private visibility and configurable TTL (1 hour, 1 day, 3 days, or single-view).

---

## When to Use This Skill

Use this skill when you need to **share an HTML file via a temporary URL**. Common scenarios:

- Sharing architecture dashboards with stakeholders
- Distributing interactive reports without requiring file access
- Generating preview links for HTML documentation
- Creating time-limited shareable links for review cycles

---

## API Reference

### Endpoint

```
POST https://pagedrop.io/api/upload
Content-Type: application/json
```

**No API key required.** The API is open for programmatic use.

> ⚠️ **Browser requests are blocked** — the API rejects requests with `Origin` or `Sec-Fetch-*` headers. Use PowerShell `Invoke-RestMethod`, `curl`, or equivalent.

### Request Body

| Field        | Type   | Required | Description                                          |
| ------------ | ------ | -------- | ---------------------------------------------------- |
| `html`       | string | **Yes**  | Full HTML content (max 16 MB)                        |
| `ttl`        | enum   | **Yes**  | Time-to-live: `1h`, `1d`, `3d`, `once`               |
| `fileName`   | string | No       | Display filename for the page                        |
| `customPath` | string | No       | Custom URL slug (3–63 chars, lowercase alphanumeric + hyphens) |
| `password`   | string | No       | Password-protect the page (max 128 chars)            |
| `visibility` | string | No       | Set to `private` to exclude from public explore page |

### Success Response (201)

```json
{
  "success": true,
  "data": {
    "url": "https://{slug}.pagedrop.io",
    "slug": "{slug}",
    "isExplore": false
  }
}
```

### Error Response

```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable description"
  }
}
```

#### Common Error Codes

| Code | Meaning | Recovery |
|---|---|---|
| `REQUEST_TOO_LARGE` | HTML exceeds 16 MB | Minify the HTML and retry |
| `PATH_UNAVAILABLE` | Custom path already taken | Omit `customPath` to auto-generate |
| `CONTENT_BLOCKED` | Content flagged by moderation | Open file locally instead |
| `RATE_LIMIT_EXCEEDED` | Too many requests | Wait and retry after delay |
| `INVALID_TTL` | TTL value not in allowed enum | Use one of: `1h`, `1d`, `3d`, `once` |

---

## PowerShell Upload Script

```powershell
# Read the HTML file
$htmlContent = Get-Content "{full-path-to-html-file}" -Raw

# Build the request payload
$payload = @{
    html       = $htmlContent
    ttl        = "3d"
    fileName   = "{solution-name}-architecture.html"
    visibility = "private"
} | ConvertTo-Json -Depth 3

# Upload to PageDrop
$response = Invoke-RestMethod -Uri "https://pagedrop.io/api/upload" `
    -Method POST `
    -ContentType "application/json" `
    -Body $payload

# Output the response
$response | ConvertTo-Json -Depth 5
```

### Extracting the URL

```powershell
if ($response.success) {
    $url = $response.data.url
    Write-Output "Live at: $url"
} else {
    Write-Output "Upload failed: $($response.error.code) - $($response.error.message)"
}
```

---

## Integration Pattern: Ask → Upload or Open Locally

When integrating into an agent workflow, follow this pattern after generating an HTML file:

### 1. Ask the User

Use the `ask_user` tool:
- **Question**: "Would you like a shareable link for this file? The link will be active for 3 days via PageDrop. Otherwise, I'll open it locally in your browser."
- **Choices**: `["Yes — generate a shareable link (active for 3 days)", "No — just open it locally"]`

### 2a. If YES — Upload to PageDrop

```powershell
$htmlContent = Get-Content "{path}" -Raw
$payload = @{ html = $htmlContent; ttl = "3d"; fileName = "{name}"; visibility = "private" } | ConvertTo-Json -Depth 3
$response = Invoke-RestMethod -Uri "https://pagedrop.io/api/upload" -Method POST -ContentType "application/json" -Body $payload
```

Present the result:
> 🔗 Your file is live for 3 days: **{url}**

Also open locally with `Start-Process` so the user has both options.

If upload fails, fall back to local open with an error note:
> Upload failed — opened locally instead. Error: {error message}

### 2b. If NO — Open Locally

```powershell
Start-Process "{full-path-to-html-file}"
```

Present the file path:
> 📂 File saved to: `{full-path-to-html-file}`

---

## Constraints

- **Max file size**: 16 MB (HTML content in the `html` field)
- **TTL options**: `1h` (1 hour), `1d` (1 day), `3d` (3 days), `once` (single view then deleted)
- **Custom paths**: 3–63 characters, lowercase letters/numbers/hyphens only, no leading/trailing hyphens
- **Passwords**: Max 128 characters
- **Rate limiting**: Applied per IP; retry with backoff if `RATE_LIMIT_EXCEEDED`
- **Content policy**: No malicious scripts, phishing, or illegal content — violations return `CONTENT_BLOCKED`
