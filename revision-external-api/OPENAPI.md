# Revision External API - OpenAPI Reference

Full endpoint reference. See [SKILL.md](SKILL.md) for overview and quick start.

## Contents

- [Components](#components) — CRUD + batch upsert
- [Diagrams](#diagrams) — CRUD + batch upsert
- [Attributes](#attributes) — CRUD + batch upsert
- [Tags](#tags) — CRUD + batch upsert
- [Types](#types) — Read-only
- [Template](#template) — Bulk sync
- [Filtering](#filtering) — Query parameters for components, diagrams, types
- [Dependencies](#dependencies) — Component dependency lookup
- [Schemas](#schemas) — Component, Diagram, ComponentInstance, Relation, Textarea, Attribute, Tag, Type, DependencySearchResult, ErrorResponse

## Components

### GET /api/external/components

List all components in the workspace. Supports optional query parameters for filtering.

**Query parameters** (all optional):
| Parameter | Type | Description |
|-----------|------|-------------|
| `name` | string | Filter by name (case-insensitive substring match) |
| `typeId` | string | Filter by type identifier |
| `tagId` | string | Filter by tag identifier |
| `attributeId` | string | Filter by attribute identifier |
| `attributeValue` | string | Filter by attribute value (requires `attributeId`) |
| `state` | enum | `DRAFT`, `ACTIVE`, `ARCHIVED` |

**Response 200**: `Component[]`

### POST /api/external/components

Create a single component. Optionally include `id` for a predictable ID; omit for auto-generated.

**Request body**: `Component`

**Required fields**: `name`

**Response 201**: `Component` (created)

### GET /api/external/components/{id}

Get a single component by ID.

**Parameters**:
- `id` (path, required): The component identifier

**Response 200**: `Component`

### PATCH /api/external/components/{id}

Update a component by ID.

**Parameters**:
- `id` (path, required): The component identifier

**Request body**: `Component` (partial, fields to update)

**Response 200**: `Component` (updated)

### DELETE /api/external/components/{id}

**Not implemented** — returns 501.

### PATCH /api/external/components/upsert-batch

Create or update multiple components. Items with `id` are updated; items without `id` are created.

**Request body**: `Component[]`

**Response 200**: `Component[]`

---

## Diagrams

### GET /api/external/diagrams

List all diagrams in the workspace. Supports optional query parameters for filtering.

**Query parameters** (all optional):
| Parameter | Type | Description |
|-----------|------|-------------|
| `componentId` | string | Filter by component identifier |
| `tagId` | string | Filter by tag identifier |
| `name` | string | Filter by name (case-insensitive substring match) |
| `level` | enum | `C0`, `C1`, `C2`, `C3`, `C4`, `D1`, `P0` |
| `state` | enum | `DRAFT`, `ACTIVE`, `ARCHIVED` |

**Response 200**: `Diagram[]`

### POST /api/external/diagrams

Create a single diagram. Optionally include `id` for a predictable ID; omit for auto-generated.

**Request body**: `Diagram`

**Required fields**: `name`

**Response 201**: `Diagram`

### GET /api/external/diagrams/{id}

Get a single diagram by ID.

**Parameters**:
- `id` (path, required): The diagram identifier

**Content negotiation**: Send `Accept: image/svg+xml` header to get an SVG rendering of the diagram instead of JSON.

**Response 200**: `Diagram` (JSON) or SVG string (`image/svg+xml`)

### PATCH /api/external/diagrams/{id}

Update a diagram by ID.

**Parameters**:
- `id` (path, required): The diagram identifier

**Request body**: `Diagram` (partial)

**Response 200**: `Diagram`

### DELETE /api/external/diagrams/{id}

**Not implemented** — returns 501.

### PATCH /api/external/diagrams/upsert-batch

Create or update multiple diagrams. Items with `id` are updated; items without `id` are created.

**Request body**: `Diagram[]`

**Response 200**: `Diagram[]`

---

## Attributes

### GET /api/external/attributes

List all attribute definitions.

**Response 200**: `Attribute[]`

### POST /api/external/attributes

Create a single attribute. Optionally include `id` for a predictable ID; omit for auto-generated.

**Request body**: `Attribute`

**Required fields**: `name`, `type`

**Validation**:
- `list` field only allowed when `type` is `LIST`
- `LIST` type must include a `list` array

**Response 201**: `Attribute`

### GET /api/external/attributes/{id}

Get a single attribute by ID.

**Parameters**:
- `id` (path, required): The attribute identifier

**Response 200**: `Attribute`

### PATCH /api/external/attributes/{id}

Update an attribute by ID.

**Parameters**:
- `id` (path, required): The attribute identifier

**Request body**: `Attribute` (partial)

**Response 200**: `Attribute`

### DELETE /api/external/attributes/{id}

**Not implemented** — returns 501.

### PATCH /api/external/attributes/upsert-batch

Create or update multiple attributes.

**Request body**: `Attribute[]`

**Response 200**: `Attribute[]`

---

## Tags

### GET /api/external/tags

List all tags.

**Response 200**: `Tag[]`

### POST /api/external/tags

Create a single tag. Optionally include `id` for a predictable ID; omit for auto-generated.

**Request body**: `Tag`

**Required fields**: `name`, `color`

**Response 201**: `Tag`

### GET /api/external/tags/{id}

Get a single tag by ID.

**Parameters**:
- `id` (path, required): The tag identifier

**Response 200**: `Tag`

### PATCH /api/external/tags/{id}

Update a tag by ID.

**Parameters**:
- `id` (path, required): The tag identifier

**Request body**: `Tag` (partial)

**Response 200**: `Tag`

### DELETE /api/external/tags/{id}

**Not implemented** — returns 501.

### PATCH /api/external/tags/upsert-batch

Create or update multiple tags.

**Request body**: `Tag[]`

**Response 200**: `Tag[]`

---

## Types

### GET /api/external/types

List all component types. Read-only. Supports optional query parameters for filtering.

**Query parameters** (all optional):
| Parameter | Type | Description |
|-----------|------|-------------|
| `name` | string | Filter by name (case-insensitive substring match) |

**Response 200**: `Type[]`

---

## Template

### POST /api/external/template

Sync components and diagrams in a single transaction. Accepts JSON or YAML.

**Request body**:
```json
{
  "components": [Component, ...],
  "diagrams": [Diagram, ...]
}
```

Both arrays default to `[]` if omitted.

**Response 200**: Sync result with upserted components and diagrams.

---

## Filtering

Filtering is done via optional query parameters on the standard list endpoints — there are no separate search paths. See the query parameter tables under [Components](#get-apiexternalcomponents), [Diagrams](#get-apiexternaldiagrams), and [Types](#get-apiexternaltypes) above.

All filters are optional. The list endpoints always return the full resource schema (`Component[]`, `Diagram[]`, `Type[]`).

---

## Dependencies

### GET /api/external/components/{id}/dependencies

Find direct upstream and downstream dependencies for a component.

**Parameters**:
- `id` (path, required): The component identifier

**Response 200**: `DependencySearchResult[]` — `{ component: { id, name }, direction: "upstream" | "downstream", diagrams: [{ id, name }] }`

---

## Schemas

### Component

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | no | Provide for predictable ID, omit for auto-generated |
| `name` | string | create | Component name |
| `state` | enum | no | `DRAFT` (default), `ACTIVE`, `ARCHIVED` |
| `desc` | string\|null | no | Description (HTML) |
| `inlineDesc` | boolean | no | Show description inline (default: false) |
| `typeId` | string\|null | no | Component type ID |
| `apiContext` | string | no | Optional label to group related imports (defaults to UTC timestamp) |
| `attributes` | array | no | `[{ id, value }]` |
| `linksTo` | string[] | no | Diagram IDs this component links to |

### Diagram

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | no | Provide for predictable ID, omit for auto-generated |
| `name` | string | create | Diagram name |
| `state` | enum | no | `DRAFT` (default), `ACTIVE`, `ARCHIVED` |
| `url` | string | read-only | Diagram URL |
| `desc` | string\|null | no | Description |
| `level` | enum\|null | no | `C0`, `C1`, `C2`, `C3`, `C4`, `D1`, `P0` |
| `tags` | string[] | no | Tag IDs |
| `apiContext` | string | no | Optional label to group related imports (defaults to UTC timestamp) |
| `componentInstances` | array | no | Component instances on diagram |
| `relations` | array | no | Relations between instances |
| `textareas` | array | no | Text areas on diagram |

### ComponentInstance (Non-Container)

> **Note**: `position`, `width`, and `height` are optional. When omitted, Revision auto-layouts and auto-sizes instances. Prefer omitting them unless the user explicitly requests specific positioning.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `ref` | string | yes | Unique reference within diagram |
| `componentId` | string\|null | no | Linked component ID |
| `position` | `{x, y}` | no | Position on canvas (omit for auto-layout) |
| `width` | number | no | Instance width (omit for auto-size) |
| `height` | number | no | Instance height (omit for auto-size) |
| `isContainer` | false | no | Must be false or omitted |
| `parent` | string | no | Parent container's ref |
| `placeholder` | object\|null | no | `{ text, typeId }` for unlinked instances |

### ComponentInstance (Container)

> **Note**: `position`, `width`, and `height` are optional. When omitted, Revision auto-layouts and auto-sizes containers. Prefer omitting them unless the user explicitly requests specific positioning.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `ref` | string | yes | Unique reference within diagram |
| `isContainer` | true | yes | Must be true |
| `componentId` | string\|null | no | Linked component ID |
| `placeholder` | object\|null | no | `{ text, typeId }` for unlinked instances |
| `position` | `{x, y}` | no | Position on canvas (omit for auto-layout) |
| `width` | number | no | Container width (omit for auto-size) |
| `height` | number | no | Container height (omit for auto-size) |

### Relation

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `fromRef` | string | yes | Source instance ref |
| `toRef` | string | yes | Target instance ref |
| `label` | string\|null | no | Edge label |
| `desc` | string\|null | no | Description |
| `linksTo` | string[] | no | Linked diagram IDs |

### Textarea

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `position` | `{x, y}` | yes | Position on canvas |
| `width` | number | no | Text area width |
| `text` | string\|null | no | Content |

### Attribute

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | no | Provide for predictable ID, omit for auto-generated |
| `name` | string | create | Attribute name (min 1 char) |
| `type` | enum | create | `STRING`, `NUMBER`, `BOOLEAN`, `LINK`, `USERLIST`, `LIST` |
| `desc` | string\|null | no | Description |
| `list` | string[] | LIST only | Options for LIST type |
| `forTypes` | string[] | no | Restrict to component type IDs |
| `required` | boolean | no | Whether attribute is required |
| `apiContext` | string | no | Optional label to group related imports (defaults to UTC timestamp) |

### Tag

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | no | Provide for predictable ID, omit for auto-generated |
| `name` | string | create | Tag name (min 1 char) |
| `color` | enum | create | `gray`, `red`, `orange`, `yellow`, `green`, `blue`, `purple` |
| `desc` | string\|null | no | Description |
| `apiContext` | string | no | Optional label to group related imports (defaults to UTC timestamp) |

### Type

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | read-only | Type ID |
| `name` | string | read-only | Type name |

### DependencySearchResult

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `component` | `{ id, name }` | yes | The dependent component |
| `direction` | enum | yes | `upstream` or `downstream` |
| `diagrams` | `[{ id, name }]` | yes | Diagrams where the dependency exists |

### ErrorResponse

```json
{ "error": "string" }
```

| Status | Description |
|--------|-------------|
| 400 | Bad request - validation error |
| 401 | Unauthorized - missing or invalid API key |
| 404 | Resource not found |
| 405 | Method not allowed |
| 501 | Not implemented (e.g. DELETE endpoints) |
