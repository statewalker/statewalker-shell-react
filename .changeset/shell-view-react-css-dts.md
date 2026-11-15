---
"@statewalker/shell.view.react": patch
---

Ship `dist/internal/dock-view-host.d.css.ts`, so the CSS import kept in `dist/public/dock-view-host.d.ts` resolves for TypeScript consumers (attw's InternalResolutionError).
