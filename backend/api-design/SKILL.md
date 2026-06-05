# API Design & Documentation

Design consistent, predictable REST APIs with OpenAPI/Swagger.

## Rules

### 1. REST Conventions
```
GET    /stores/:storeId/products          — List (paginated)
GET    /stores/:storeId/products/:id      — Get one
POST   /stores/:storeId/products          — Create
PUT    /stores/:storeId/products/:id      — Full update
PATCH  /stores/:storeId/products/:id      — Partial update
DELETE /stores/:storeId/products/:id      — Soft delete
POST   /stores/:storeId/products/:id/restore — Undo soft delete
```

### 2. URL Design
- Use plural nouns: `/products` not `/product`.
- Nested resources for ownership: `/stores/:storeId/products`.
- Actions as sub-resources: `/transactions/:id/void` (POST).
- Kebab-case: `line-items` (only if necessary, prefer camelCase in JSON).
- No verbs in URLs: use HTTP methods instead.

### 3. Response Format
```json
// List
{
  "items": [...],
  "total": 150,
  "page": 1,
  "limit": 50,
  "totalPages": 3
}

// Detail
{ "id": "uuid", "name": "...", "createdAt": "..." }

// Error
{ "statusCode": 400, "message": "Validation failed", "errors": [...] }
```

### 4. Query Parameters
- Pagination: `?page=1&limit=50` (max 100).
- Filtering: `?status=active&categoryId=xxx`.
- Sorting: `?sort=createdAt&order=desc`.
- Search: `?search=keyword`.
- Delta sync: `?since=2024-01-01T00:00:00Z`.

### 5. Status Codes
- 200 OK — GET, PUT, PATCH success
- 201 Created — POST success
- 204 No Content — DELETE success
- 400 Bad Request — validation error
- 401 Unauthorized — missing/invalid token
- 403 Forbidden — valid token but insufficient permissions
- 404 Not Found — resource doesn't exist
- 409 Conflict — duplicate (e.g., unique constraint)
- 422 Unprocessable — business rule violation
- 429 Too Many Requests — rate limit

### 6. Versioning
- URL versioning: `/api/v1/products` (preferred for public APIs).
- Header versioning: `Accept: application/vnd.api+json;version=1` (for internal APIs).
- Don't version unless necessary. Start without version, add later.

### 7. OpenAPI / Swagger
```ts
@ApiTags('Products')
@ApiBearerAuth()
export class ProductController {
  @ApiOperation({ summary: 'List products' })
  @ApiQuery({ name: 'page', required: false })
  @ApiResponse({ status: 200, type: [ProductDto] })
  @Get()
  findAll() { ... }
}
```

## Anti-Patterns
- ❌ Verbs in URLs: `/getProducts`, `/createUser`
- ❌ Returning different shapes for list vs detail
- ❌ 200 OK for errors with `{ success: false }`
- ❌ Nested pagination (pages within pages)
- ❌ Missing or inconsistent error response format
