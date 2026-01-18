# ivoz-api Development - Claude Code Context

This directory contains the forked ivoz-api source code - the Swagger/OpenAPI library used by IvozProvider REST APIs.

---

## Repository Information

| Property | Value |
|----------|-------|
| **Fork** | https://github.com/mii502/ivoz-api |
| **Upstream** | https://github.com/irontec/ivoz-api |
| **Version** | 5.x |
| **License** | GPL-3.0 |
| **Type** | Composer package (irontec/ivoz-api) |

---

## What is ivoz-api?

ivoz-api is a PHP library that provides:

1. **Swagger/OpenAPI Generation** - Auto-generates API documentation from code
2. **Documentation Normalizers** - Transforms API spec for Swagger UI consumption
3. **API Platform Integration** - Works with Symfony and API Platform
4. **Reference Handling** - Manages `$ref` relationships in OpenAPI spec

IvozProvider's REST APIs (client, brand, platform) use this library to generate `/api/*/docs.json` endpoints.

---

## Directory Structure

```
ivoz-api/
├── CLAUDE.md                           # This file
├── composer.json                       # Package definition
├── Swagger/
│   └── Serializer/
│       └── DocumentationNormalizer/
│           ├── ReferenceFixerDecorator.php    # Fixes entity references
│           └── UnusedDefinitionRemover.php    # Removes unused definitions
└── ... (other API utilities)
```

---

## Key Components

### Documentation Normalizers

| Component | Purpose |
|-----------|---------|
| `ReferenceFixerDecorator` | Fixes `$ref` relationships between entities |
| `UnusedDefinitionRemover` | Removes unused type definitions from output |

These normalizers transform the raw OpenAPI spec into a clean, consumable format for Swagger UI.

---

## Patches Applied (2026-01-18)

### 1. ReferenceFixerDecorator - isset() checks

**Problem:** API endpoints with array properties (e.g., `items.$ref` but no direct `$ref`) caused "Undefined array key '$ref'" errors.

**Fix:** Added `isset()` checks before accessing `$property['$ref']`:
```php
$hasRef = isset($property['$ref']);
$hasItemsRef = isset($property['items']['$ref']);

if ($hasRef && $this->isEntity($property['$ref'], $definitions) ...) {
```

### 2. UnusedDefinitionRemover - null coalescing

**Problem:** Parameters with `schema` but no `$ref` caused errors.

**Fix:** Added null coalescing operator:
```php
$ref = $parameter['schema']['$ref'] ?? null;
```

**Impact:** Fixes 500 errors when loading docs.json on brand/platform portals with custom API endpoints.

---

## When to Modify ivoz-api

**DO modify** when you need to:
- Fix Swagger/OpenAPI generation bugs
- Add custom documentation features
- Handle edge cases in API spec generation

**DON'T modify** when:
- Adding new API endpoints (modify ivozprovider instead)
- Changing API behavior (modify ivozprovider REST controllers)
- Customizing documentation output format (use API Platform annotations)

---

## Development Workflow

### 1. Make Changes Locally

```bash
cd servers/vm-ivozprovider-lab/src/ivoz-api
# Edit files in Swagger/Serializer/DocumentationNormalizer/
```

### 2. Commit and Push

```bash
git add .
git commit -m "fix: description of fix"
git push origin 5.x
```

### 3. Deploy to Server

```bash
rsync -av --delete \
  src/ivoz-api/Swagger/ \
  user@185.16.41.36:/opt/irontec/ivozprovider/library/vendor/irontec/ivoz-api/Swagger/
```

### 4. Clear Cache

```bash
ssh user@185.16.41.36 "cd /opt/irontec/ivozprovider/web/rest/brand && sudo -u www-data php bin/console cache:clear"
ssh user@185.16.41.36 "cd /opt/irontec/ivozprovider/web/rest/client && sudo -u www-data php bin/console cache:clear"
```

### 5. Verify

```bash
curl -s -k "https://brand.ivoz.voip.ing/api/brand/docs.json" | head -1
# Should return: {"swagger":"2.0",...
```

---

## Syncing with Upstream

```bash
git fetch upstream
git checkout 5.x
git merge upstream/5.x --no-edit
# Resolve conflicts (re-apply patches if needed)
git push origin 5.x
```

---

## Relationship with IvozProvider

```
┌──────────────────────────────────────────────────────────────┐
│                     IvozProvider REST APIs                    │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │   Client    │  │    Brand    │  │  Platform   │          │
│  │    API      │  │    API      │  │    API      │          │
│  │  /api/client│  │  /api/brand │  │ /api/platform          │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘          │
│         │                │                │                   │
│         └────────────────┼────────────────┘                   │
│                          │                                    │
│                          ▼                                    │
│               ┌──────────────────┐                            │
│               │   irontec/       │  ◄── THIS REPO             │
│               │    ivoz-api      │                            │
│               └──────────────────┘                            │
│                          │                                    │
│                          ▼                                    │
│               ┌──────────────────┐                            │
│               │   API Platform   │                            │
│               │    + Swagger     │                            │
│               └──────────────────┘                            │
└──────────────────────────────────────────────────────────────┘
```

---

## References

### Local
- **IvozProvider source:** `../ivozprovider/` (sibling directory)
- **IvozProvider CLAUDE.md:** `../CLAUDE.md` (parent context)
- **ivoz-ui source:** `../ivoz-ui/` (sibling directory)

### External
- **ivoz-api GitHub:** https://github.com/irontec/ivoz-api
- **API Platform:** https://api-platform.com/
- **OpenAPI Spec:** https://swagger.io/specification/
- **IvozProvider docs:** https://irontec.github.io/ivozprovider/en/
