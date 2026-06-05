# NestJS API Builder

You are an expert NestJS backend developer. Build production-grade APIs following these conventions.

## Core Rules

### 1. Module Structure
```
src/
├── feature-name/
│   ├── feature-name.module.ts
│   ├── feature-name.controller.ts
│   ├── feature-name.service.ts
│   ├── dto/
│   │   ├── create-feature.dto.ts
│   │   └── update-feature.dto.ts
│   └── entities/
│       └── feature.entity.ts
```
One module per domain concept. Keep modules focused and small.

### 2. Controllers
```ts
@Controller('stores/:storeId/products')
@UseGuards(JwtAuthGuard)
export class ProductController {
  @Post()
  create(@Param('storeId') storeId: string, @CurrentUser() user: AuthUser, @Body() dto: CreateDto) {
    return this.service.create(storeId, user.userId, dto);
  }

  @Get()
  findAll(@Param('storeId') storeId: string, @Query('page') page?: string, @Query('limit') limit?: string) {
    return this.service.findAll(storeId, { page: +page || 1, limit: Math.min(+limit || 50, 100) });
  }
}
```
- Always paginate list endpoints. Default page=1, limit=50, max limit=100.
- Use `@UseGuards(JwtAuthGuard)` at class level when all routes need auth.
- Use `@CurrentUser()` decorator to extract authenticated user.

### 3. DTOs & Validation
```ts
import { IsString, IsNumber, IsOptional, Min, IsEnum } from 'class-validator';
import { ApiProperty } from '@nestjs/swagger';

export class CreateProductDto {
  @ApiProperty() @IsString() name!: string;
  @ApiProperty({ required: false }) @IsOptional() @IsString() description?: string;
  @ApiProperty() @IsNumber() @Min(0) sellPrice!: number;
  @ApiProperty({ required: false }) @IsOptional() @IsNumber() @Min(0) stockQuantity?: number;
}
```
- ALWAYS use `class-validator` decorators. Never trust unvalidated input.
- Use `@ApiProperty()` for Swagger documentation.
- `@IsOptional()` before other validators.

### 4. Service Pattern
```ts
@Injectable()
export class ProductService {
  async create(storeId: string, dto: CreateProductDto, userId?: string): Promise<Product> {
    // 1. Validate business rules
    // 2. Create entity
    // 3. Save to DB
    // 4. Return saved entity (with relations if needed)
  }
}
```
- Services handle business logic, repositories handle data access.
- Use `@InjectRepository()` with TypeORM for DB operations.
- Return entities directly from create/update — don't wrap in `{ data: }` unless needed.

### 5. Error Handling
- Use NestJS built-in exceptions: `NotFoundException`, `BadRequestException`, `UnauthorizedException`, `ForbiddenException`, `ConflictException`.
- DON'T catch and return `{ success: false }` — let exceptions bubble up.
- Use exception filters ONLY for logging/transformation, never for business logic.
- ValidationPipe global: `app.useGlobalPipes(new ValidationPipe({ whitelist: true, forbidNonWhitelisted: true }))`.

### 6. Database (TypeORM)
- Use `@CreateDateColumn()` and `@UpdateDateColumn()` for timestamps.
- Use UUID primary keys: `@PrimaryGeneratedColumn('uuid')`.
- Always add indexes on foreign keys and frequently queried columns.
- Use QueryBuilder for complex queries, `find/findOne` for simple ones.
- NEVER use raw SQL with template strings (SQL injection). Use parameterized queries.

### 7. API Response Format
```json
// List: paginated response
{
  "items": [...],
  "total": 150,
  "page": 1,
  "limit": 50,
  "totalPages": 3
}

// Single entity: direct entity (no wrapper)
{ "id": "...", "name": "...", ... }

// Error: let NestJS exception filter handle
{ "statusCode": 404, "message": "Product not found" }
```

### 8. Async & Transactions
- Use `@nestjs/typeorm` `DataSource.transaction()` for atomic operations.
- Use `Promise.all()` for independent async operations.
- Use BullMQ for background jobs (email, notifications, heavy processing).

## Anti-Patterns
- ❌ Business logic in controllers
- ❌ Catching exceptions in controllers (let them propagate)
- ❌ Using `any` type for query results
- ❌ Missing validation decorators on DTOs
- ❌ Returning `{ success: true, data }` wrapper (inconsistent)
