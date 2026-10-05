# statewalker-shell-react

## What it is

React renderers for the statewalker application shell. Each package here is
the view half of a logic package: the logic package (for example
`@statewalker/shell.core`) holds state, commands and slots with no React, and
the package here (`@statewalker/shell.view.react`) renders them. Together they
give an app a React mount, a dockview-based shell, a settings dialog, a file
explorer, viewers for images, Markdown, PDF and video, and the AI settings
tabs. All packages are published to npm under `@statewalker/*`.

## The shape: fifteen view packages, no apps

```
packages/
  ui.view.react             React mount, <AppRoot>, core:views slot, workspace hooks, theme CSS
  ui.view.shadcn            shadcn/ui primitives and cn()
  render.view.react         <SpecRenderer>; re-exports from @json-render/react
  shell.view.react          dockview host, MainShell, header, the "json" dock panel, tab icons
  workspace.view.react      workspace label header item, switch-workspace button
  settings.view.react       settings dialog and settings button
  explorer.view.react       file-explorer panels (list, breadcrumbs, filter, drag and drop)
  inline.view.react         <InlineContent> and the built-in inline components
  mime.view.image           image/* viewer
  mime.view.markdown        text/markdown viewer and a reusable <Markdown> component
  mime.view.pdf             application/pdf viewer (browser built-in PDF viewer)
  mime.view.video           video/* viewer
  ai-config.view.react      "Remote Models" settings tab
  ai-local-models.view.react "Local Models" settings tab
  webapp.view.react         <SiteFrame>: iframe dock panel for a web app hosted from the workspace
```

| Package | npm |
| --- | --- |
| [`@statewalker/ui.view.react`](packages/ui.view.react) | [npm](https://www.npmjs.com/package/@statewalker/ui.view.react) |
| [`@statewalker/ui.view.shadcn`](packages/ui.view.shadcn) | [npm](https://www.npmjs.com/package/@statewalker/ui.view.shadcn) |
| [`@statewalker/render.view.react`](packages/render.view.react) | [npm](https://www.npmjs.com/package/@statewalker/render.view.react) |
| [`@statewalker/shell.view.react`](packages/shell.view.react) | [npm](https://www.npmjs.com/package/@statewalker/shell.view.react) |
| [`@statewalker/workspace.view.react`](packages/workspace.view.react) | [npm](https://www.npmjs.com/package/@statewalker/workspace.view.react) |
| [`@statewalker/settings.view.react`](packages/settings.view.react) | [npm](https://www.npmjs.com/package/@statewalker/settings.view.react) |
| [`@statewalker/explorer.view.react`](packages/explorer.view.react) | [npm](https://www.npmjs.com/package/@statewalker/explorer.view.react) |
| [`@statewalker/inline.view.react`](packages/inline.view.react) | [npm](https://www.npmjs.com/package/@statewalker/inline.view.react) |
| [`@statewalker/mime.view.image`](packages/mime.view.image) | [npm](https://www.npmjs.com/package/@statewalker/mime.view.image) |
| [`@statewalker/mime.view.markdown`](packages/mime.view.markdown) | [npm](https://www.npmjs.com/package/@statewalker/mime.view.markdown) |
| [`@statewalker/mime.view.pdf`](packages/mime.view.pdf) | [npm](https://www.npmjs.com/package/@statewalker/mime.view.pdf) |
| [`@statewalker/mime.view.video`](packages/mime.view.video) | [npm](https://www.npmjs.com/package/@statewalker/mime.view.video) |
| [`@statewalker/ai-config.view.react`](packages/ai-config.view.react) | [npm](https://www.npmjs.com/package/@statewalker/ai-config.view.react) |
| [`@statewalker/ai-local-models.view.react`](packages/ai-local-models.view.react) | [npm](https://www.npmjs.com/package/@statewalker/ai-local-models.view.react) |
| [`@statewalker/webapp.view.react`](packages/webapp.view.react) | [npm](https://www.npmjs.com/package/@statewalker/webapp.view.react) |

Most packages share one source layout:

```
src/
  index.ts      re-exports public/index.ts and the default init
  fragment.ts   re-exports the default init
  public/       exported components, slots, catalogs, init
  internal/     implementation and tests; not reachable through exports
  styles.css    Tailwind v4 @source globs (ui.view.react also defines the theme)
```

Each package exports `.` and `./fragment` from `dist/` (JS and `.d.ts`) and,
when it has a stylesheet, `./styles` from `src/styles.css`.
`render.view.react` exports only `.`. `webapp.view.react` has no `./styles` and
keeps its sources directly in `src/`. Packages ship both `dist/` and `src/`.

The packages use these `@statewalker` packages from npm: `shell.core`,
`workspace.core`, `workspace.browser`, `render.core`, `mime.core`,
`explorer.core`, `inline.core`, `settings.core`, `webapp.browser`,
`ai-config.core`, `ai-local-models.core`, `shared-baseclass`,
`shared-commands`, `shared-registry`, `shared-slots`, `webrun-files` and (in
tests) `webrun-files-mem`.

## How to run it

1. Install Node.js 24.
2. Enable corepack so the pinned pnpm 10 is used: `corepack enable`.
3. Install: `pnpm install`.
4. Build every package: `pnpm build`.
5. Run the tests: `pnpm test`.

There is no app in this repository to start. The packages are activated by a
host app; see "How a host app wires the packages" below.

## Why it is the way it is

### Logic and view live in separate packages

A `*.view.*` package is the only kind of package allowed to import a UI
library (React, shadcn, the json-render React bindings). The logic packages
stay React-free, so they can be tested without a DOM and a different renderer
could replace these packages. Values that cross from logic to view are data:
a logic package names a component by a string `viewKey`, and the view
package registers the component under that key in the `core:views` slot.

### How a host app wires the packages

Every package with a `./fragment` export has a default export
`init(ctx) => cleanup`. It reads the `Workspace` from `ctx`, registers
components into slots, and returns a function that removes them.

```
host boot
  create Workspace, put it into ctx
  init logic fragments      (shell.core, settings.core, explorer.core, ...)
  init view fragments       (ui.view.react, shell.view.react, settings.view.react, ...)
                            ui.view.react: createRoot(#app).render(<AppRoot/>)
  import each package's ./styles once
```

`ui.view.react` renders whatever is registered under `shell:root` in
`core:views`; `shell.view.react` registers `MainShell` there. Neither imports
the other's components.

### Dependency specifiers

Packages inside this repository depend on each other with `workspace:^`.
Everything else comes from the pnpm catalog in `pnpm-workspace.yaml`
(`catalog:`); `react` and `react-dom` are peers (`catalog:peers`, `>=18`).

## What will surprise you

- **Components render without colors or spacing.** The host did not import
  `@statewalker/ui.view.react/styles` (theme variables) or a package's
  `./styles` (Tailwind `@source` globs), so Tailwind did not emit its classes.
- **`useAppWorkspace must be used inside <AppWorkspaceProvider>.`** A
  component that uses the workspace hooks was rendered outside `<AppRoot>`.
- **`No adapter registered for ...`** at fragment init. A view fragment ran
  before the logic fragment that installs the adapter it needs (for example
  `SpecStore`, `LayoutStore`, `DockHost`).
- **A dock tab shows "Spec ... is missing." or "Catalog ... is not
  registered."** The panel was restored from the saved layout, but no
  fragment created its spec or registered its catalog.
- **The page stays empty and nothing is logged.** There is no element with
  `id="app"`; `ui.view.react`'s init then skips the mount silently.
- **Nothing renders after the folder is opened.** No component is registered
  under `shell:root`: `shell.view.react`'s fragment was not activated.
- **Cleanup functions return promises.** Most `init` functions return
  `() => Promise<void>`; await them when tearing down.

## Reference

### Commands

| Command | What it does |
| --- | --- |
| `pnpm install` | Install dependencies |
| `pnpm build` | Build every package with tsdown |
| `pnpm test` | Run every package's vitest suite |
| `pnpm typecheck` | Type-check every package |
| `pnpm lint` / `pnpm lint:check` | Biome check, with or without fixes |
| `pnpm format` / `pnpm format:check` | Biome format, with or without writes |
| `pnpm changeset` | Add a changeset to pick a version bump and changelog text |

### Releases

Packages are published to npm from CI with changesets. After CI passes on
`main`, a job adds a patch changeset for each package whose packed contents
differ from npm and opens a "chore: version packages" pull request; merging it
publishes. Add your own changeset with `pnpm changeset` to choose the bump or
the changelog text. Dependency updates come from Renovate.

### License

MIT. See [LICENSE](LICENSE).
