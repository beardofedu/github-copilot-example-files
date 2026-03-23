---
# FILE: .github/instructions/frontend.instructions.md
#
# PURPOSE: Path-specific custom instructions for GitHub Copilot
# SCOPE: Only applies to files matching the "applyTo" glob pattern below
#
# DOCUMENTATION:
#   https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions#creating-path-specific-custom-instructions
#
# HOW IT WORKS:
#   When Copilot is working on a file that matches the "applyTo" glob pattern,
#   it automatically loads these instructions in addition to any repo-wide
#   instructions in .github/copilot-instructions.md. You can have multiple
#   *.instructions.md files, each targeting different parts of the codebase.
#
# NAMING CONVENTION:
#   Files must be named <NAME>.instructions.md and placed in the
#   .github/instructions/ directory. The NAME can be anything descriptive
#   (e.g., frontend.instructions.md, api.instructions.md, tests.instructions.md).
#
# THE "applyTo" FRONTMATTER PROPERTY:
#   This YAML frontmatter key controls which files these instructions apply to.
#   It accepts standard glob patterns:
#     "**/*.ts"        → all TypeScript files anywhere in the repo
#     "src/api/**"     → everything under the src/api directory
#     "**/*.test.*"    → all test files
#     "**"             → all files (equivalent to copilot-instructions.md)
#   Multiple patterns can be listed as a YAML array.
#
# WHEN INSTRUCTIONS CONFLICT:
#   If both copilot-instructions.md and a path-specific file apply, Copilot
#   combines them. More specific instructions generally take precedence.
#
# DEVELOPMENT NOTE:
#   This particular file is an EXAMPLE for a hypothetical frontend codebase.
#   In a real project you would customize the applyTo pattern and the
#   instructions below to match your actual tech stack and conventions.

applyTo: "src/frontend/**"
---

# Frontend Development Instructions

<!--
  These instructions apply only to files under src/frontend/.
  They supplement (not replace) the repo-wide instructions in
  .github/copilot-instructions.md.
-->

## Framework and Libraries

<!--
  DEVELOPMENT NOTE: Specify your exact frontend framework version and the
  key libraries you use. This prevents Copilot from mixing patterns from
  different framework versions or suggesting libraries you don't use.
-->

- Use **React 18** with functional components and hooks (no class components)
- Use **TypeScript** with strict mode enabled
- Use **Tailwind CSS** for styling (no CSS-in-JS, no plain CSS files)
- Use **React Query** for server state management
- Use **Zustand** for client state management
- Import React explicitly: `import React from 'react'` is NOT required (JSX transform is configured)

## Component Patterns

<!--
  DEVELOPMENT NOTE: Document your component conventions so Copilot generates
  code that matches your existing patterns. Without this, Copilot may generate
  perfectly valid React but in a style inconsistent with the rest of your code.
-->

- One component per file
- Component file names use PascalCase: `UserProfile.tsx`
- Component functions use named exports (not default exports):
  ```tsx
  // ✅ Preferred
  export function UserProfile({ userId }: UserProfileProps) { ... }

  // ❌ Avoid
  export default function UserProfile({ userId }: UserProfileProps) { ... }
  ```
- Define prop types with a `Props` suffix interface above the component:
  ```tsx
  interface UserProfileProps {
    userId: string;
    onClose?: () => void;
  }
  ```
- Keep components under 150 lines; extract logic into custom hooks when needed

## Hooks

<!--
  DEVELOPMENT NOTE: If you have naming or structural conventions for hooks,
  document them here. Custom hooks are often where complex business logic
  lives, so clear conventions are especially important.
-->

- Custom hooks go in `src/frontend/hooks/`
- Hook file names use camelCase with `use` prefix: `useUserProfile.ts`
- Hooks that fetch data should use React Query's `useQuery` or `useMutation`
- Keep hooks focused on a single concern

## Styling

- All styling via Tailwind utility classes directly in JSX
- Use `clsx` or `cn` helper for conditional class names
- No inline `style` props unless absolutely necessary for dynamic values
- Follow the mobile-first responsive design pattern

## Testing

<!--
  DEVELOPMENT NOTE: Specify your frontend testing approach. In particular,
  clarify whether you prefer Testing Library's user-event approach vs.
  direct enzyme-style interaction, and whether you use snapshot tests.
-->

- Use **Vitest** with **React Testing Library**
- Test files live alongside their components: `UserProfile.test.tsx`
- Write tests from the user's perspective (test behavior, not implementation)
- Avoid snapshot tests; prefer explicit assertions
- Mock API calls at the React Query layer using `msw`

## Accessibility

- All interactive elements must have accessible labels (`aria-label`, `aria-labelledby`, or visible text)
- Use semantic HTML elements (`<button>`, `<nav>`, `<main>`, etc.)
- Ensure keyboard navigation works for all interactive UI
- Minimum contrast ratio of 4.5:1 for text

## Performance

- Memoize expensive computations with `useMemo`
- Use `useCallback` for event handlers passed to child components
- Lazy-load route-level components with `React.lazy` and `Suspense`
- Avoid unnecessary re-renders by keeping state as local as possible
