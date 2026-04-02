# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development
npm run dev          # Start dev server with Turbopack
npm run build        # Build for production
npm run start        # Start production server

# Code Quality
npm run lint         # Run ESLint
npm run test         # Run Vitest test suite

# Database
npm run setup        # Install deps + generate Prisma client + run migrations
npm run db:reset     # Reset database (destructive)
```

Run a single test file: `npx vitest run src/path/to/__tests__/file.test.ts`

## Architecture

UIGen is an AI-powered React component generator with live preview. Users describe components in natural language, Claude generates them via tool calling, and users see a live preview.

### Core Flow

1. User sends a chat message → `POST /api/chat/route.ts`
2. Claude AI uses two tools (`str_replace_editor`, `file_manager`) to create/modify files in a **virtual in-memory file system**
3. The file system state streams back to the client in real time
4. The preview iframe compiles JSX via Babel standalone + ESM CDN import maps and renders the component

### Key Abstractions

**Virtual File System** (`src/lib/file-system.ts`): All generated files live in memory as a `FileSystem` class instance. Files are serialized to JSON for SQLite persistence. The `FileSystemContext` (`src/lib/contexts/file-system-context.tsx`) manages this state client-side.

**AI Tools** (`src/lib/tools/`): Two tools expose file operations to Claude:
- `str_replace_editor` — view, create, str_replace, insert commands
- `file_manager` — rename, delete commands

**Preview** (`src/components/preview/PreviewFrame.tsx`): Renders in an iframe using Babel standalone for JSX→JS transformation in-browser. React 19 and common libs are loaded via ESM CDN (esm.sh) import maps.

**LLM Provider** (`src/lib/provider.ts`): Returns a Claude (claude-haiku-4-5) provider if `ANTHROPIC_API_KEY` is set, otherwise falls back to a mock provider that returns static components — useful for development without an API key.

**Authentication**: JWT tokens in HTTP-only cookies via `jose`. `src/middleware.ts` protects API routes. Server actions in `src/actions/` handle sign-up/sign-in and project CRUD. Anonymous users can generate components but cannot save projects.

### Data Model

Prisma + SQLite with two models: `User` (bcrypt password) and `Project` (stores virtual file system as serialized JSON, chat messages as JSON string, optional user FK with cascade delete).

### Path Alias

`@/*` maps to `./src/*` throughout the codebase.
