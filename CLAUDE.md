# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UIGen is an AI-powered React component generator with live preview. Users describe React components in natural language, and the AI generates them with real-time preview and code editing capabilities.

## Common Commands

```bash
npm run setup          # Install deps + Prisma generate + database migrations
npm run dev            # Development server with Turbopack (http://localhost:3000)
npm run build          # Production build
npm run lint           # ESLint (extends next/core-web-vitals)
npm run test           # Run vitest tests
npx prisma studio      # Open Prisma database browser
npm run db:reset       # Reset database (destructive)
```

## Architecture

### State Management
- **ChatContext** (`src/lib/contexts/chat-context.tsx`): Manages chat state and AI interactions
- **FileSystemContext** (`src/lib/contexts/file-system-context.tsx`): Manages virtual file system and tool execution
- **useChat hook** from @ai-sdk/react handles LLM streaming

### Core Systems

**VirtualFileSystem** (`src/lib/file-system.ts`): In-memory file system that stores generated code. No disk writes - all files exist only in memory during session. Serializable for database persistence.

**JSX Transformer** (`src/lib/transform/jsx-transformer.ts`): Uses Babel standalone to transform JSX for browser preview. Generates import maps pointing to esm.sh CDN.

**AI Tools** (`src/lib/tools/`):
- `str-replace.ts`: String replacement tool for code editing
- `file-manager.ts`: File creation/management tool

**Provider** (`src/lib/provider.ts`): Wraps Anthropic SDK. Falls back to mock provider returning static responses when `ANTHROPIC_API_KEY` is not set.

### API & Server Actions
- `/api/chat` route: Streams AI responses, executes tool calls, persists project state
- Server actions in `src/actions/`: Auth functions (signUp, signIn, signOut), project CRUD

### Authentication
- JWT sessions (7-day expiry, HttpOnly cookies) in `src/lib/auth.ts`
- Middleware (`src/middleware.ts`) protects API routes
- Anonymous mode works without auth but has no persistence

### Database
Prisma with SQLite. Schema defined in `prisma/schema.prisma`. Two models:
- **User**: id, email, password (bcrypt), projects relation
- **Project**: id, name, userId (optional), messages (JSON), data (JSON file system)

Prisma client generated to `src/generated/prisma`. Database file at `prisma/dev.db`.

## Testing

Tests use Vitest with Testing Library and jsdom. Test files are in `__tests__` directories alongside source files.

```bash
npm run test              # Run all tests
npx vitest run <pattern>  # Run specific test file
```

## Key Patterns

- Path alias: `@/*` maps to `./src/*`
- UI components use shadcn/ui (Radix primitives + Tailwind)
- Preview renders in iframe with import maps resolving to esm.sh CDN
- Claude Haiku 4.5 model used for generation with prompt caching

## Environment Variables

- `ANTHROPIC_API_KEY`: Optional - without it, mock provider returns static example components
- `JWT_SECRET`: Defaults to "development-secret-key" in dev
