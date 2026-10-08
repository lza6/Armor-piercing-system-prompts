---
name: react-next-best-practices
description: React/Next.js 项目最佳实践：组件拆分、数据获取、性能、bundle、RSC、hydration 和路由。
---

# React / Next.js Best Practices

Use this skill when creating, reviewing, or refactoring React or Next.js applications.

## Focus Areas

- Component boundaries: keep components cohesive and avoid unnecessary abstraction.
- Data fetching: avoid waterfalls; keep server/client responsibilities explicit.
- Rendering model: distinguish Server Components, Client Components, SSR, SSG, and dynamic routes.
- State: keep local state local; avoid global state unless shared behavior requires it.
- Performance: watch bundle size, memoization misuse, expensive renders, image loading, and caching.
- Hydration: avoid browser-only values during server render unless guarded.
- Accessibility: use semantic HTML before custom ARIA.
- Testing: prefer targeted tests for changed behavior.

## Review Output

Return concrete findings with file paths, severity, and suggested fixes. Avoid generic advice unless tied to the current code.

## PilotDeck Migration Note

- Source inspiration: Vercel Agent Skills for React/Next.js workflows.
- This is a PilotDeck-native draft, not a verbatim copy and is not Vercel-platform-specific.

