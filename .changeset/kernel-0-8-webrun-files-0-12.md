---
"@statewalker/ai-config.view.react": minor
"@statewalker/ai-local-models.view.react": minor
"@statewalker/explorer.view.react": minor
"@statewalker/inline.view.react": minor
"@statewalker/mime.view.image": minor
"@statewalker/mime.view.markdown": minor
"@statewalker/mime.view.pdf": minor
"@statewalker/mime.view.video": minor
"@statewalker/settings.view.react": minor
"@statewalker/shell.view.react": minor
"@statewalker/ui.view.react": minor
"@statewalker/webapp.view.react": minor
"@statewalker/workspace.view.react": minor
---

Move to the kernel 0.8 packages (workspace.core, shell.core, explorer.core, mime.core, render.core, settings.core, webapp.browser, workspace.browser), ai-config.core and ai-local-models.core 0.7, and `@statewalker/webrun-files` 0.12. These are breaking lines of their dependencies: `..` path segments are now resolved before any filter or mount sees the path, and a path climbing above the root throws.
