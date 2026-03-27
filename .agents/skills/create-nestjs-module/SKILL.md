---
name: create-nestjs-module
description: Tạo một NestJS feature module mới theo chuẩn dự án (controller, service, dto, module)
---

# Tạo NestJS Feature Module

Skill này hướng dẫn cách tạo một feature module mới trong backend NestJS, tuân thủ cấu trúc dự án TikTok Clone.

## Cấu trúc module chuẩn

Mỗi module trong `apps/api/src/modules/` phải có cấu trúc sau:

```
modules/<tên-module>/
├── <tên-module>.module.ts        # Module definition
├── <tên-module>.controller.ts    # Route handlers
├── <tên-module>.service.ts       # Business logic
├── dto/
│   ├── create-<tên>.dto.ts       # DTO cho create
│   ├── update-<tên>.dto.ts       # DTO cho update
│   └── query-<tên>.dto.ts        # DTO cho query/filter (nếu cần)
└── <tên-module>.controller.spec.ts  # Unit tests (optional)
```

## Quy tắc đặt tên
- Tên folder module: `kebab-case` (ví dụ: `videos`, `users`, `follow`)
- Tên file: `kebab-case` (ví dụ: `videos.controller.ts`, `videos.service.ts`)
- Tên class: `PascalCase` (ví dụ: `VideosController`, `VideosService`)
- Dùng **named exports** — KHÔNG dùng `export default`

## Template: Module File

```typescript
// modules/example/example.module.ts
import { Module } from '@nestjs/common';
import { ExampleController } from './example.controller';
import { ExampleService } from './example.service';
import { PrismaModule } from '../../prisma/prisma.module';

@Module({
  imports: [PrismaModule],
  controllers: [ExampleController],
  providers: [ExampleService],
  exports: [ExampleService],
})
export class ExampleModule {}
```

## Template: Controller File

```typescript
// modules/example/example.controller.ts
import {
  Controller,
  Get,
  Post,
  Patch,
  Delete,
  Body,
  Param,
  Query,
  UseGuards,
  HttpCode,
  HttpStatus,
} from '@nestjs/common';
import { ExampleService } from './example.service';
import { CreateExampleDto } from './dto/create-example.dto';
import { UpdateExampleDto } from './dto/update-example.dto';
import { JwtAuthGuard } from '../../common/guards/jwt-auth.guard';
import { CurrentUser } from '../../common/decorators/current-user.decorator';

@Controller('api/v1/examples')
export class ExampleController {
  constructor(private readonly exampleService: ExampleService) {}

  @Post()
  @UseGuards(JwtAuthGuard)
  @HttpCode(HttpStatus.CREATED)
  async create(
    @CurrentUser('id') userId: string,
    @Body() dto: CreateExampleDto,
  ) {
    const data = await this.exampleService.create(userId, dto);
    return { success: true, data };
  }

  @Get()
  async findAll(@Query('page') page = 1, @Query('limit') limit = 20) {
    const { data, total } = await this.exampleService.findAll(page, limit);
    return {
      success: true,
      data,
      meta: { page, limit, total, hasMore: page * limit < total },
    };
  }

  @Get(':id')
  async findOne(@Param('id') id: string) {
    const data = await this.exampleService.findOne(id);
    return { success: true, data };
  }

  @Patch(':id')
  @UseGuards(JwtAuthGuard)
  async update(
    @Param('id') id: string,
    @CurrentUser('id') userId: string,
    @Body() dto: UpdateExampleDto,
  ) {
    const data = await this.exampleService.update(id, userId, dto);
    return { success: true, data };
  }

  @Delete(':id')
  @UseGuards(JwtAuthGuard)
  @HttpCode(HttpStatus.NO_CONTENT)
  async remove(
    @Param('id') id: string,
    @CurrentUser('id') userId: string,
  ) {
    await this.exampleService.remove(id, userId);
  }
}
```

## Template: Service File

```typescript
// modules/example/example.service.ts
import { Injectable, NotFoundException, ForbiddenException } from '@nestjs/common';
import { PrismaService } from '../../prisma/prisma.service';
import { CreateExampleDto } from './dto/create-example.dto';
import { UpdateExampleDto } from './dto/update-example.dto';

@Injectable()
export class ExampleService {
  constructor(private readonly prisma: PrismaService) {}

  async create(userId: string, dto: CreateExampleDto) {
    return this.prisma.example.create({
      data: { ...dto, userId },
    });
  }

  async findAll(page: number, limit: number) {
    const skip = (page - 1) * limit;
    const [data, total] = await Promise.all([
      this.prisma.example.findMany({
        skip,
        take: limit,
        orderBy: { createdAt: 'desc' },
      }),
      this.prisma.example.count(),
    ]);
    return { data, total };
  }

  async findOne(id: string) {
    const item = await this.prisma.example.findUnique({ where: { id } });
    if (!item) throw new NotFoundException('Example not found');
    return item;
  }

  async update(id: string, userId: string, dto: UpdateExampleDto) {
    const item = await this.findOne(id);
    if (item.userId !== userId) throw new ForbiddenException();
    return this.prisma.example.update({
      where: { id },
      data: dto,
    });
  }

  async remove(id: string, userId: string) {
    const item = await this.findOne(id);
    if (item.userId !== userId) throw new ForbiddenException();
    return this.prisma.example.delete({ where: { id } });
  }
}
```

## Template: DTO File

```typescript
// modules/example/dto/create-example.dto.ts
import { IsString, IsOptional, IsBoolean, MaxLength } from 'class-validator';

export class CreateExampleDto {
  @IsString()
  @MaxLength(500)
  caption: string;

  @IsOptional()
  @IsBoolean()
  isPublic?: boolean;
}
```

## Sau khi tạo module

1. Import module mới vào `app.module.ts`:
```typescript
import { ExampleModule } from './modules/example/example.module';

@Module({
  imports: [
    // ...other modules
    ExampleModule,
  ],
})
export class AppModule {}
```

2. Nếu module cần model mới trong DB, sửa `prisma/schema.prisma` rồi:
```bash
# ⚠️ KHÔNG tạo migration — Dùng db push
yarn db:push
yarn db:generate
```

## Response format chuẩn

Mọi response đều phải tuân thủ format:
```json
{
  "success": true,
  "data": {},
  "meta": { "page": 1, "limit": 20, "total": 100, "hasMore": true }
}
```

Error response:
```json
{
  "success": false,
  "error": {
    "code": "NOT_FOUND",
    "message": "Resource not found"
  }
}
```
