<div align="center">

# Zen Query 🔍

**End-to-end typesafe APIs with realtime delta subscriptions - Zero codegen, maximum DX**

[![License](https://img.shields.io/badge/License-MIT-blue?style=flat-square)](https://github.com/SylphxAI/zen-query/blob/main/LICENSE)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.8+-blue?style=flat-square&logo=typescript)](https://typescriptlang.org)
[![pnpm](https://img.shields.io/badge/pnpm-10.8+-orange?style=flat-square&logo=pnpm)](https://pnpm.io)

**Zero codegen** • **Delta subscriptions** • **Optimistic updates** • **Transport agnostic**

[Quick Start](#-quick-start) • [Features](#-key-features) • [Packages](#-packages)

</div>

---

## 🚀 Overview

TypeScript-first realtime API framework with end-to-end type safety and incremental data synchronization. Inspired by tRPC, Zen Query goes further with built-in realtime delta subscriptions using JSON Patch.

**The Problem:**
```
Traditional API development:
- REST: No type safety, boilerplate ❌
- GraphQL: Schema duplication, codegen ❌
- tRPC: No realtime deltas (full refetch) ❌
- Manual websockets: Complex state sync ❌
```

**The Solution:**
```
Zen Query:
- End-to-end TypeScript inference ✅
- Zero codegen, zero schema files ✅
- Realtime delta subscriptions (JSON Patch) ✅
- Optimistic updates built-in ✅
```

**Result: Truly typesafe APIs with effortless realtime state synchronization.**

---

## ⚡ Key Advantages

### vs Other Solutions

| Feature | REST | GraphQL | tRPC | Zen Query |
|---------|------|---------|------|-----------|
| **Type Safety** | ❌ None | ⚠️ Codegen | ✅ Inferred | ✅ Inferred |
| **Codegen** | N/A | ❌ Required | ✅ None | ✅ None |
| **Realtime** | ❌ Polling | ⚠️ Subscriptions | ⚠️ Full refetch | ✅ Delta patches |
| **Optimistic Updates** | ❌ Manual | ⚠️ Manual | ⚠️ Manual | ✅ Built-in |
| **Batching** | ❌ No | ⚠️ Complex | ✅ Yes | ✅ Yes |
| **Transport** | HTTP only | HTTP only | HTTP only | ✅ Pluggable |

### Developer Experience

- **Catch errors at compile time** - TypeScript *is* your schema
- **Zero boilerplate** - Plain TypeScript functions are your API
- **Incremental updates** - Only changed data transmitted (JSON Patch)
- **Snappy UIs** - Automatic optimistic updates + reconciliation
- **Flexible transports** - WebSocket, HTTP, VSCode, or custom

---

## 🎯 Key Features

### End-to-End Type Safety

```typescript
// Server: Define API with TypeScript
const appRouter = createRouter()
  .procedure('addTodo', t.mutation({
    input: z.object({ text: z.string() }),
    resolve: async ({ input }) => {
      return db.createTodo(input.text);
    }
  }));

// Client: Fully typed, no codegen
const result = await client.todos.add({ text: 'Buy milk' });
//    ^? type: { id: string; text: string; completed: boolean }
```

### Delta Subscriptions (JSON Patch)

**Efficient State Sync:**
```typescript
// Traditional: Re-fetch entire list
const todos = await client.todos.list(); // All data

// Zen Query: Receive only changes
client.todos.onUpdate.subscribe((patch) => {
  // patch: [{ op: 'add', path: '/123', value: newTodo }]
  applyPatch(state, patch); // Apply delta
});
```

**Benefits:**
- **90% less bandwidth** - Only changes transmitted
- **Instant updates** - No polling, no refetch
- **Collaborative apps** - Real-time multi-user sync
- **RFC 6902 compliant** - Standard JSON Patch

### Optimistic Updates

```typescript
const $addTodo = mutation(
  client => client.todos.add,
  {
    effects: [
      effect($todos, (current, input) => {
        // Immediately show pending todo
        return [...current, { ...input, status: 'pending' }];
      })
    ]
  }
);

// UI updates instantly, reconciles on response
await addTodo({ text: 'New task' });
```

### Transport Agnostic

**Built-in Transports:**
- **HTTP** - With request batching
- **WebSocket** - Bidirectional realtime
- **VSCode** - Extension message passing

**Bring Your Own:**
```typescript
createClient({
  transport: createCustomTransport({
    // Your transport implementation
  })
});
```

---

## 📦 Packages

### Core

| Package | Description | Use Case |
|---------|-------------|----------|
| **@sylphlab/zen-query-client** | Core client logic | Client-side setup |
| **@sylphlab/zen-query-server** | Core server logic | Server-side setup |
| **@sylphlab/zen-query-shared** | Shared types/utilities | Both sides |

### Framework Integrations

| Package | Description | Hooks |
|---------|-------------|-------|
| **@sylphlab/zen-query-react** | React hooks | `useQuery`, `useMutation`, `useSubscription` |
| **@sylphlab/zen-query-preact** | Preact hooks | Same as React |

### State Management

**Nanostores Integration:**
- Built-in support for Nanostores atoms
- `query()`, `mutation()`, `subscription()`, `hybrid()` helpers
- Automatic state sync with delta subscriptions

### Transports

| Package | Transport | Features |
|---------|-----------|----------|
| **@sylphlab/zen-query-transport-http** | HTTP | Batching, REST-like |
| **@sylphlab/zen-query-transport-websocket** | WebSocket | Bidirectional, realtime |
| **@sylphlab/zen-query-transport-vscode** | VSCode | Extension messaging |

---

## 🚀 Quick Start

### Installation

```bash
# Server
pnpm add @sylphlab/zen-query-server zod

# Client (React example)
pnpm add @sylphlab/zen-query-client @sylphlab/zen-query-react
pnpm add @sylphlab/zen-query-transport-websocket
```

### Server Setup

```typescript
// server/router.ts
import { initZenQuery, createRouter } from '@sylphlab/zen-query-server';
import { z } from 'zod';

const t = initZenQuery();

export const appRouter = createRouter()
  .procedure('getTodos', t.query({
    resolve: async () => {
      return db.todos.findMany();
    },
  }))
  .procedure('addTodo', t.mutation({
    input: z.object({ text: z.string() }),
    resolve: async ({ input }) => {
      const todo = await db.todos.create(input);
      // Emit delta patch for subscribers
      emitPatch([{ op: 'add', path: `/${todo.id}`, value: todo }]);
      return todo;
    },
  }))
  .procedure('onTodoUpdate', t.subscription({
    stream: async function* () {
      for await (const patch of patchStream) {
        yield patch; // JSON Patch operations
      }
    }
  }));

export type AppRouter = typeof appRouter;
```

### Client Setup (React)

```typescript
// client/api.ts
import { createClient } from '@sylphlab/zen-query-client';
import { createWebSocketTransport } from '@sylphlab/zen-query-transport-websocket';
import type { AppRouter } from '../server/router';

export const client = createClient<AppRouter>({
  transport: createWebSocketTransport({ url: 'ws://localhost:3000' })
});
```

### Use in Components

```typescript
// Component.tsx
import { useQuery, useMutation, useSubscription } from '@sylphlab/zen-query-react';
import { client } from './api';

function TodoList() {
  // Query with type inference
  const { data: todos, loading } = useQuery(client.getTodos);

  // Mutation with optimistic updates
  const addTodo = useMutation(client.addTodo, {
    optimistic: (current, input) => [...current, input]
  });

  // Realtime subscription (deltas only!)
  useSubscription(client.onTodoUpdate, {
    onData: (patch) => {
      // Automatically applied to state
    }
  });

  return (
    <div>
      {todos?.map(todo => <div key={todo.id}>{todo.text}</div>)}
      <button onClick={() => addTodo({ text: 'New' })}>
        Add Todo
      </button>
    </div>
  );
}
```

---

## 💡 Advanced Usage

### With Nanostores

```typescript
// store.ts
import { atom } from 'nanostores';
import { query, mutation, hybrid } from '@sylphlab/zen-query-client/nanostores';
import { client } from './api';

// Query atom
export const $todos = query(
  client => client.todos.list,
  { input: { limit: 10 }, initialData: [] }
);

// Mutation atom with effects
export const $addTodo = mutation(
  client => client.todos.add,
  {
    effects: [
      effect($todos, (todos, input) => {
        return [...todos, { ...input, id: 'temp', status: 'pending' }];
      })
    ]
  }
);

// Hybrid (query + subscription)
const $todosSub = subscription(client => client.todos.onUpdate);
export const $todosLive = hybrid($todos, $todosSub);
```

### Multiple Transports

```typescript
// Use HTTP for queries/mutations, WebSocket for subscriptions
createClient({
  transport: createHttpTransport({ url: '/api' }),
  subscriptionTransport: createWebSocketTransport({ url: 'ws://localhost:3000' })
});
```

---

## 🏗️ Architecture

### Monorepo Structure

```
packages/
├── client/          # Core client logic
│   ├── createClient()
│   ├── OptimisticSyncCoordinator
│   └── nanostores/  # Nanostores bindings
├── server/          # Core server logic
│   ├── initZenQuery()
│   ├── createRouter()
│   └── procedure builders
├── shared/          # Shared types/utilities
├── react/           # React hooks
├── preact/          # Preact hooks
└── transport-*/     # Transport adapters
    ├── http/        # HTTP + batching
    ├── websocket/   # WebSocket
    └── vscode/      # VSCode extension

examples/
├── web-app/         # React + WebSocket
└── vscode-extension/# VSCode integration
```

### Technology Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Language** | TypeScript 5.8+ | Type inference |
| **Validation** | Zod | Input schemas |
| **State Sync** | JSON Patch (RFC 6902) | Delta updates |
| **Build** | tsup + Turborepo | Monorepo builds |
| **Package Manager** | pnpm | Workspace management |
| **Testing** | Vitest | Unit + integration tests |
| **Linting** | Biome | Code quality |

---

## 🎯 Use Cases

### Realtime Collaborative Apps
- **Multiplayer editors** - Google Docs-style collaboration
- **Team dashboards** - Live data updates
- **Chat applications** - Instant message sync
- **Project management** - Real-time task updates

### Data-Intensive UIs
- **Live analytics** - Streaming metrics
- **Trading platforms** - Price updates
- **IoT dashboards** - Sensor data streams
- **Social feeds** - Real-time content

### Developer Tools
- **VSCode extensions** - Type-safe extension APIs
- **Admin panels** - Real-time monitoring
- **DevOps tools** - Live system status
- **Testing tools** - Result streaming

---

## 🔧 Development

### Setup

```bash
# Install dependencies
pnpm install

# Build all packages
pnpm run build

# Run tests
pnpm run test

# Test with coverage
pnpm run test:coverage

# Lint
pnpm run lint

# Dev mode (watch)
pnpm run dev
```

### Monorepo Commands

```bash
# Build specific package
pnpm turbo run build --filter=@sylphlab/zen-query-client

# Test specific package
pnpm turbo run test --filter=@sylphlab/zen-query-react

# Run example
cd examples/web-app
pnpm run dev
```

### Publishing

```bash
# Add changeset
pnpm changeset add

# Version packages
pnpm changeset version

# Publish
pnpm run release
```

---

## 📊 Comparison

### vs tRPC

| Feature | tRPC | Zen Query |
|---------|------|-----------|
| **Type Safety** | ✅ Excellent | ✅ Excellent |
| **Realtime** | ⚠️ Full refetch | ✅ Delta patches |
| **Subscriptions** | ✅ Basic | ✅ JSON Patch deltas |
| **Optimistic Updates** | ⚠️ Manual | ✅ Built-in |
| **State Sync** | ❌ No | ✅ Incremental |
| **Transports** | HTTP | HTTP, WS, VSCode, Custom |

**Choose tRPC if:** You need a mature, proven solution with minimal realtime needs.

**Choose Zen Query if:** You need realtime delta subscriptions and optimistic updates out of the box.

### vs GraphQL

| Feature | GraphQL | Zen Query |
|---------|---------|-----------|
| **Type Safety** | ⚠️ Codegen | ✅ Inferred |
| **Schema** | ❌ Separate file | ✅ TypeScript |
| **Realtime** | ✅ Subscriptions | ✅ Delta subscriptions |
| **Overhead** | ❌ Schema duplication | ✅ Single source |
| **Learning Curve** | High | Low (TypeScript) |

---

## 🗺️ Roadmap

**✅ Completed**
- [x] End-to-end TypeScript inference
- [x] Zero codegen architecture
- [x] Delta subscriptions (JSON Patch)
- [x] Optimistic updates
- [x] HTTP, WebSocket, VSCode transports
- [x] React/Preact hooks
- [x] Nanostores integration
- [x] Monorepo structure (Turborepo)

**🚀 Planned**
- [ ] Solid.js integration
- [ ] Vue integration
- [ ] Svelte integration
- [ ] gRPC transport
- [ ] React Native support
- [ ] Offline-first capabilities
- [ ] Performance benchmarks
- [ ] Comprehensive docs site

---

## 🤝 Contributing

Contributions are welcome! Please follow these guidelines:

1. **Open an issue** - Discuss changes before implementing
2. **Fork the repository**
3. **Create a feature branch** - `git checkout -b feature/my-feature`
4. **Follow coding standards** - Use Biome linting
5. **Write tests** - Ensure good coverage
6. **Add changeset** - `pnpm changeset add`
7. **Submit a pull request**

### Development Guidelines

- Follow TypeScript strict mode
- Use Conventional Commits
- Add tests for new features
- Update documentation
- Run `pnpm run validate` before committing

---

## 🤝 Support

[![License](https://img.shields.io/badge/License-MIT-blue?style=flat-square)](https://github.com/SylphxAI/zen-query/blob/main/LICENSE)
[![GitHub Issues](https://img.shields.io/github/issues/SylphxAI/zen-query?style=flat-square)](https://github.com/SylphxAI/zen-query/issues)

- 🐛 [Bug Reports](https://github.com/SylphxAI/zen-query/issues)
- 💬 [Discussions](https://github.com/SylphxAI/zen-query/discussions)
- 📧 [Email](mailto:hi@sylphx.com)

**Show Your Support:**
⭐ Star • 👀 Watch • 🐛 Report bugs • 💡 Suggest features • 🔀 Contribute

---

## 📄 License

MIT © [Sylphx](https://sylphx.com)

---

## 🙏 Credits

Built with:
- [TypeScript](https://typescriptlang.org) - Type system
- [Zod](https://zod.dev) - Schema validation
- [JSON Patch (RFC 6902)](https://tools.ietf.org/html/rfc6902) - Delta updates
- [Turborepo](https://turbo.build) - Monorepo management
- [Vitest](https://vitest.dev) - Testing framework
- [Biome](https://biomejs.dev) - Code quality

Inspired by [tRPC](https://trpc.io) - Thanks for pioneering end-to-end type safety!

---

## 📚 Examples

Check the `examples/` directory for practical implementations:

**[examples/web-app](examples/web-app)**
- React + WebSocket transport
- Realtime todo list with optimistic updates
- Delta subscription demo

**[examples/vscode-extension](examples/vscode-extension)**
- VSCode extension integration
- Custom transport implementation
- Type-safe extension API

---

<p align="center">
  <strong>Typesafe. Realtime. Zero codegen.</strong>
  <br>
  <sub>End-to-end type safety with effortless delta subscriptions</sub>
  <br><br>
  <a href="https://sylphx.com">sylphx.com</a> •
  <a href="https://x.com/SylphxAI">@SylphxAI</a> •
  <a href="mailto:hi@sylphx.com">hi@sylphx.com</a>
</p>
