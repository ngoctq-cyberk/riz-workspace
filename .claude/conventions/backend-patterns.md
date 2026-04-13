# Backend Patterns

## Prisma Schema Convention

```typescript
// All entities must have:
id         BigInt   @id @default(autoincrement())
createdAt  DateTime @default(now())
updatedAt  DateTime @updatedAt
profileId  BigInt?  // Optional, related to profile
```

## Entity Structure

Entities live in `libs/*/entities/*.entity.ts`:

```typescript
import { Expose, Type } from "class-transformer";
import { ApiProperty } from "@nestjs/swagger";

export class TodoEntity {
  @ApiProperty()
  @Expose()
  id: bigint;

  @ApiProperty()
  @Expose()
  createdAt: Date;

  @ApiProperty()
  @Expose()
  updatedAt: Date;

  @ApiProperty()
  @Expose()
  title: string;

  @ApiProperty()
  @Expose()
  profileId: bigint;

  // Relations section (at bottom)
  @ApiProperty({ type: () => ProfileEntity })
  @Type(() => ProfileEntity)
  @Expose()
  profile: ProfileEntity;
}
```

## DTO Conventions

**Create DTO** (`create-*.dto.ts`): No `profileId` field (taken from user), use class-validator + ApiProperty decorators.

**Update DTO** (`edit-*.dto.ts`): Use `PickType` + `PartialType`:

```typescript
import { PartialType, PickType } from "@nestjs/swagger";
import { CreateTodoDto } from "./create-todo.dto";

class _UpdateTodoDto extends PickType(CreateTodoDto, ["title", "description"]) {}
export class UpdateTodoDto extends PartialType(_UpdateTodoDto) {}
```

**Query DTO** (`query-*.dto.ts`): Support where, sort, select, include, skip, take. Number fields must have `@Type(() => Number)`.

## Controller Pattern

Custom decorators to use:

- `@CurUser() user: User` — get current user from JWT
- `@RawQuery() dto: QueryDto` — parse query params
- `@AppCacheKey((req) => ...)` — custom cache key
- `ParseBigIntPipe` — parse BigInt params (`@Param('id', ParseBigIntPipe) id: bigint`)
- `@UseGuards(JwtGuard)` + `@ApiBearerAuth()` — auth required endpoints
- `@ApiOkResponse({ type: () => Entity })` — Swagger response type

## Service Layer

Services must include `user` parameter in create/update/delete methods. Use `th.toInstanceSafe()` and `th.toInstancesSafe()` from `@app/helper/transform.helper` to transform Prisma results to Entity classes:

```typescript
async createTodo(dto: CreateTodoDto, user: User) {
  const todo = await this.prisma.todo.create({
    data: {
      ...dto,
      profileId: user.profileId, // Attach to user's profile
    },
  })
  return th.toInstanceSafe(TodoEntity, todo)
}

async updateTodo(id: bigint, dto: UpdateTodoDto, user: User) {
  const todo = await this.prisma.todo.update({
    where: { id, profileId: user.profileId }, // Security: only update own data
    data: dto,
  })
  return th.toInstanceSafe(TodoEntity, todo)
}
```
