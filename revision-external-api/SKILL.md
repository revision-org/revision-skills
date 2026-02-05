---
name: revision-external-api
description: Interacts with the Revision External API for managing architecture components, diagrams, attributes, tags, and templates. Use when the user mentions Revision API, architecture components, C4 diagrams, or wants to sync architecture data with Revision.
---

# Revision External API

REST API for managing architecture documentation in Revision workspaces.

## Authentication

All requests require a Bearer token (API key from workspace settings):

```
Authorization: Bearer <api-key>
```

## Base URL

- Production: `https://app.revision.so`
- Local dev: `http://localhost:3002`

## Resources

| Resource | Endpoints | Description |
|----------|-----------|-------------|
| Components | CRUD + batch upsert | Architecture components (services, databases, etc.) |
| Diagrams | CRUD + batch upsert | Architecture diagrams with component instances, relations, textareas |
| Attributes | CRUD + batch upsert | Custom attribute definitions on components |
| Tags | CRUD + batch upsert | Tags for categorizing diagrams |
| Types | Read-only | Component type definitions |
| Template | POST | Bulk sync of components + diagrams in a single transaction |
| Search | GET | Search components, diagrams, and dependencies |

## Quick Start

### List components

```bash
curl -H "Authorization: Bearer $API_KEY" \
  https://app.revision.so/api/external/components
```

### Create a component

```bash
curl -X POST -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '[{"name": "User Service", "state": "ACTIVE"}]' \
  https://app.revision.so/api/external/components
```

### Update a component

```bash
curl -X POST -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "User Service", "state": "ACTIVE", "desc": "Handles user auth"}' \
  https://app.revision.so/api/external/components/component-id
```

## Standard CRUD Pattern

Every resource (components, diagrams, attributes, tags) follows the same pattern:

| Method | Path | Action |
|--------|------|--------|
| GET | `/api/external/{resource}` | List all |
| POST | `/api/external/{resource}` | Create (array of items, no `id` field) |
| GET | `/api/external/{resource}/{id}` | Get by ID |
| POST | `/api/external/{resource}/{id}` | Update by ID |
| DELETE | `/api/external/{resource}/{id}` | Delete by ID |
| POST | `/api/external/{resource}/upsert-batch` | Batch upsert (items with `id` are updated, without are created) |

**Create requests** accept an array of items. Items must NOT include an `id` field.

**Batch upsert** is the most powerful pattern: send items with `id` to update, without `id` to create, all in one request.

## Component Schema

```json
{
  "id": "string",
  "name": "string",
  "state": "DRAFT | ACTIVE | ARCHIVED",
  "ref": "string | null",
  "desc": "string | null",
  "inlineDesc": false,
  "typeId": "string | null",
  "apiContext": "string",
  "attributes": [{ "id": "string", "value": "boolean | number | string | null" }],
  "linksTo": ["diagram-id-1", "diagram-id-2"]
}
```

- `ref`: External reference identifier
- `apiContext`: Opaque string for API-managed metadata
- `linksTo`: Array of diagram IDs this component links to
- `attributes`: Component attribute values (reference attribute definitions by `id`)

## Diagram Schema

```json
{
  "id": "string",
  "name": "string",
  "state": "DRAFT | ACTIVE | ARCHIVED",
  "url": "string",
  "desc": "string | null",
  "level": "C0 | C1 | C2 | C3 | C4 | D1 | P0 | null",
  "tags": ["tag-id-1"],
  "apiContext": "string",
  "componentInstances": [],
  "relations": [],
  "textareas": []
}
```

### Diagram Levels

| Level | Meaning |
|-------|---------|
| C0 | Landscape |
| C1 | System Context |
| C2 | Container |
| C3 | Component |
| C4 | Code |
| D1 | Deployment |
| P0 | Process |

### Component Instances

Placed on diagrams. Two types:

**Non-container** (default):
```json
{
  "ref": "unique-ref",
  "componentId": "component-id | null",
  "position": { "x": 100, "y": 200 },
  "width": 200,
  "height": 100,
  "parent": "container-ref",
  "isContainer": false,
  "placeholder": { "text": "Name", "typeId": "type-id" }
}
```

**Container** (groups other instances):
```json
{
  "ref": "unique-ref",
  "componentId": "component-id | null",
  "position": { "x": 50, "y": 50 },
  "width": 500,
  "height": 400,
  "isContainer": true
}
```

- `ref` is required and must be unique within the diagram
- `componentId` links the instance to a component definition (null for placeholders)
- Non-containers can reference a `parent` container via the container's `ref`
- Containers cannot have a parent

### Relations

Directed edges between component instances:
```json
{
  "fromRef": "instance-ref-1",
  "toRef": "instance-ref-2",
  "label": "string | null",
  "desc": "string | null",
  "linksTo": ["diagram-id"]
}
```

### Textareas

Free text on diagrams:
```json
{
  "position": { "x": 100, "y": 100 },
  "width": 300,
  "text": "string | null"
}
```

## Attribute Schema

Custom fields on components:
```json
{
  "id": "string",
  "name": "string",
  "desc": "string | null",
  "type": "STRING | NUMBER | BOOLEAN | LINK | USERLIST | LIST",
  "list": ["option1", "option2"],
  "forTypes": ["type-id-1"],
  "required": false,
  "apiContext": "string"
}
```

- `list` is only valid when `type` is `LIST` (and required for LIST)
- `forTypes`: Restrict attribute to specific component types

## Tag Schema

```json
{
  "id": "string",
  "name": "string",
  "desc": "string | null",
  "color": "gray | red | orange | yellow | green | blue | purple",
  "apiContext": "string"
}
```

## Type Schema (Read-Only)

```json
{
  "id": "string",
  "name": "string"
}
```

Types are managed in the Revision UI. The API can only list them via `GET /api/external/types`.

## Template Sync

Bulk sync components and diagrams in a single transaction:

```bash
curl -X POST -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"components": [...], "diagrams": [...]}' \
  https://app.revision.so/api/external/template
```

Accepts both JSON and YAML. Components and diagrams follow their standard schemas.

## Search

### Search Diagrams

```
GET /api/external/search/diagrams?componentId=...&tagId=...&name=...&level=...&state=...
```

At least one filter required.

### Search Components

```
GET /api/external/search/components?name=...&typeId=...&tagId=...&attributeId=...&attributeValue=...&state=...
```

At least one filter required. `attributeValue` requires `attributeId`.

### Search Dependencies

```
GET /api/external/search/dependencies?componentId=...
```

Returns upstream and downstream direct dependencies for a component.

## Error Responses

All errors return:
```json
{ "error": "description" }
```

| Status | Meaning |
|--------|---------|
| 400 | Validation error |
| 401 | Missing or invalid API key |
| 404 | Resource not found |
| 405 | Method not allowed |

## Full OpenAPI Spec

For the complete OpenAPI 3.0.3 specification with all request/response schemas, see [OPENAPI.md](OPENAPI.md).
