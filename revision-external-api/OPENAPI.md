# Revision External API - OpenAPI Reference

Full endpoint reference. See [SKILL.md](SKILL.md) for overview and quick start.

## Contents

- [Components](#components) — CRUD + batch upsert
- [Diagrams](#diagrams) — CRUD + batch upsert
- [Attributes](#attributes) — CRUD + batch upsert
- [Tags](#tags) — CRUD + batch upsert
- [Types](#types) — Read-only
- [Template](#template) — Bulk sync
- [Search](#search) — Diagrams, components, dependencies
- [Schemas](#schemas) — Component, Diagram, ComponentInstance, Relation, Textarea, Attribute, Tag, Type, ErrorResponse

## Components

### GET /api/external/components

List all components in the workspace.

**Response 200**: `Component[]`

### POST /api/external/components

Create a single component. Optionally include `id` for a predictable ID; omit for auto-generated.

**Request body**: `Component`

**Required fields**: `name`

**Response 201**: `Component` (created)

### GET /api/external/components/{id}

Get a single component by ID (slug or ref).

**Parameters**:
- `id` (path, required): The component ID (slug or ref)

**Response 200**: `Component`

### PATCH /api/external/components/{id}

Update a component by ID.

**Parameters**:
- `id` (path, required): The component ID (slug or ref)

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

List all diagrams in the workspace.

**Response 200**: `Diagram[]`

### POST /api/external/diagrams

Create a single diagram. Optionally include `id` for a predictable ID; omit for auto-generated.

**Request body**: `Diagram`

**Required fields**: `name`

**Response 201**: `Diagram`

### GET /api/external/diagrams/{id}

Get a single diagram by ID (slug or ref).

**Parameters**:
- `id` (path, required): The diagram ID (slug or ref)

**Response 200**: `Diagram`

### PATCH /api/external/diagrams/{id}

Update a diagram by ID.

**Parameters**:
- `id` (path, required): The diagram ID (slug or ref)

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
- `id` (path, required): The attribute ID (slug or ref)

**Response 200**: `Attribute`

### PATCH /api/external/attributes/{id}

Update an attribute by ID.

**Parameters**:
- `id` (path, required): The attribute ID (slug or ref)

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
- `id` (path, required): The tag ID (slug or ref)

**Response 200**: `Tag`

### PATCH /api/external/tags/{id}

Update a tag by ID.

**Parameters**:
- `id` (path, required): The tag ID (slug or ref)

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

List all component types. Read-only.

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

## Search

### GET /api/external/search/diagrams

Search diagrams by filters. At least one query parameter required.

**Query parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `componentId` | string | Filter by component ID |
| `tagId` | string | Filter by tag ID |
| `name` | string | Filter by name (partial match) |
| `level` | enum | `C0`, `C1`, `C2`, `C3`, `C4`, `D1`, `P0` |
| `state` | enum | `DRAFT`, `ACTIVE`, `ARCHIVED` |

**Response 200**: `DiagramSearchResult[]` — `{ id, name, url, state, level, tags }`

### GET /api/external/search/components

Search components by filters. At least one query parameter required.

**Query parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `name` | string | Filter by name (partial match) |
| `typeId` | string | Filter by type ID |
| `tagId` | string | Filter by tag ID |
| `attributeId` | string | Filter by attribute ID |
| `attributeValue` | string | Filter by attribute value (requires `attributeId`) |
| `state` | enum | `DRAFT`, `ACTIVE`, `ARCHIVED` |

**Response 200**: `ComponentSearchResult[]` — `{ id, name, state, desc, type: { id, name } | null, tags, attributes: [{ id, value: string | null }] }`

### GET /api/external/search/dependencies

Find direct upstream and downstream dependencies for a component.

**Query parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `componentId` | string | The component ID to find dependencies for |

**Response 200**: `DependencySearchResult[]` — `{ component: { id, name }, direction: "upstream" | "downstream", diagrams: [{ id, name }] }`

---

## Schemas

### Component

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | no | Provide for predictable ID, omit for auto-generated |
| `name` | string | create | Component name |
| `state` | enum | no | `DRAFT` (default), `ACTIVE`, `ARCHIVED` |
| `ref` | string\|null | no | External reference |
| `desc` | string\|null | no | Description (HTML) |
| `inlineDesc` | boolean | no | Show description inline (default: false) |
| `typeId` | string\|null | no | Component type ID |
| `apiContext` | string | no | Opaque API metadata |
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
| `apiContext` | string | no | Opaque API metadata |
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
| `apiContext` | string | no | Opaque API metadata |

### Tag

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | no | Provide for predictable ID, omit for auto-generated |
| `name` | string | create | Tag name (min 1 char) |
| `color` | enum | create | `gray`, `red`, `orange`, `yellow`, `green`, `blue`, `purple` |
| `desc` | string\|null | no | Description |
| `apiContext` | string | no | Opaque API metadata |

### Type

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | read-only | Type ID |
| `name` | string | read-only | Type name |

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
