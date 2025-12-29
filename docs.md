# s16.api docs

## Getting Games

### List All Games

Retrieve the full game catalog with pagination.

**Request:**

```
GET /api
```

**Parameters:**

| Parameter | Type | Default | Range |
|-----------|------|---------|-------|
| page | integer | 1 | 1+ |
| pageSize | integer | 50 | 1-100 |

**Example:**

```bash
curl "http://localhost:8080/api?page=1&pageSize=25"
```

**Response:**

```json
{
  "gms": [
    {
      "Title": "Game Name",
      "Description": "Game description",
      "Instructions": "How to play",
      "Type": "html5",
      "Url": "https://example.com/game",
      "Asset": ["https://example.com/cover.jpg"],
      "Category": ["action"],
      "Tag": ["multiplayer"]
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 25,
    "total": 1000,
    "totalPages": 40,
    "hasNext": true,
    "hasPrev": false
  }
}
```

### Search Games

Search by title, category, or tag. Multiple words require all matches.

**Request:**

```
GET /api/query/:query
```

**Parameters:**

| Parameter | Type | Required | Notes |
|-----------|------|----------|-------|
| query | string | Yes | Max 200 characters, URL encode |
| page | integer | No | Default: 1 |
| pageSize | integer | No | Default: 50, Max: 100 |

**Examples:**

```bash
# Single word
curl "http://localhost:8080/api/query/puzzle"

# Multiple words (both must match)
curl "http://localhost:8080/api/query/racing%20cars"

# With pagination
curl "http://localhost:8080/api/query/action?page=2&pageSize=20"
```

**Response:**

```json
{
  "query": "puzzle",
  "gms": [...],
  "pagination": {...}
}
```

## Game Object Structure

Each game contains these fields:

| Field | Type | Description |
|-------|------|-------------|
| Title | string | Game name |
| Description | string | Game description |
| Instructions | string | How to play |
| Type | string | Game type (usually "html5") |
| Url | string | Direct link to play |
| Asset | array | Cover image URLs |
| Category | array | Genre categories |
| Tag | array | Additional tags |

## Additional Endpoints

### Health Check

Check if the API is running.

**Request:**

```
GET /health
```

**Response:**

```json
{
  "status": "healthy",
  "timestamp": "2025-11-08T10:30:00.000Z",
  "uptime": 3600,
  "cache": {
    "active": true,
    "age": 120000
  }
}
```

### Metrics

View API usage statistics.

**Request:**

```
GET /metrics
```

**Response:**

```json
{
  "requests": 1000,
  "cache_hits": 850,
  "cache_misses": 150,
  "errors": 5,
  "cache_hit_rate": "85.00%"
}
```

### Schema

Get the JSON schema for API responses.

**Request:**

```
GET /api/schema
```

Returns the JSON Schema (draft-07) definition.

## Rate Limits

The API limits requests to prevent overload:

* **Soft limit:** 50 requests per minute (adds 500ms delay per request after)
* **Hard limit:** 100 requests per minute (returns 429 error)

When rate limited, you'll receive:

```json
{
  "error": "Too many requests, please slow down"
}
```

## Caching

Responses are cached for better performance.

**Cache Headers:**

The API includes `ETag` headers for cache validation. Send the `If-None-Match` header with the ETag value to check if your cached data is still current.

**Example:**

```bash
# First request
curl -i "http://localhost:8080/api"
# Note the ETag in response headers

# Subsequent request
curl -H "If-None-Match: abc123def456" "http://localhost:8080/api"
# Returns 304 Not Modified if cache is valid
```

**Cache duration:** 5 minutes (300 seconds)

## Error Responses

All errors return JSON with an error message:

```json
{
  "error": "Description of what went wrong"
}
```

**Common Status Codes:**

| Code | Meaning |
|------|---------|
| 200 | Success |
| 304 | Not Modified (cached data is current) |
| 400 | Invalid parameters |
| 404 | Endpoint not found |
| 429 | Rate limit exceeded |
| 500 | Server error |
