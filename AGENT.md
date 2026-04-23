# AGENT Guide

This repo builds a native macOS app for managing `SKILL.md` libraries across AI coding agents. Use this file as the quick-start contract and read the detailed guide in [docs/macos-agent-guide.md](/Users/tongtong/code/skills-manager/docs/macos-agent-guide.md:1) before large changes.

## Priority

1. Follow direct user instructions.
2. Follow this file.
3. Follow the detailed guide in `docs/macos-agent-guide.md`.
4. Keep OpenSpec artifacts aligned with behavior changes in `openspec/`.

## Product Rules

- Disk is the source of truth for skill content.
- GRDB stores metadata, snapshots, and sync state, not full skill bodies.
- Support Claude Code, Codex CLI, Gemini CLI, and Cursor path conventions.
- Prefer symlink-based sync; document any fallback to copied files.

## Implementation Rules

- Prefer native SwiftUI and macOS scene APIs before using AppKit bridges.
- Keep views presentation-only and move coordination into view models or services.
- Preserve one-way flow: Views -> ViewModels -> Services -> persistence and file system.
- Use `@Observable` for shared app-owned state and `@State` for view-local state.
- Make sync source-of-truth and conflict rules explicit before changing behavior.


## File System And Watchers

- Use `FSEventStream` for directory-tree monitoring.
- Use `DispatchSource.makeFileSystemObjectSource` only for narrow supplemental observation.
- Design watcher changes with debounce, coalescing, cold-start scan, and self-trigger suppression.
- Treat delete, overwrite, migration, and rename flows as explicit product decisions with recovery paths.
- Keep path normalization and symlink resolution rules consistent across services.

## Permissions And Errors

- Assume a non-sandboxed macOS app unless the user requests otherwise.
- Use security-scoped bookmarks to restore user-selected locations across launches; do not frame them as sandbox permission renewal.
- Prefer typed throws at service boundaries, and wrap system or third-party errors at the edge.
- Every user-visible failure should point to a recovery action.

## Testing

- Add or update unit tests for parsing, sync decisions, migrations, and bookmark recovery.
- Use `XCUITest` for stable macOS UI flows and regressions.
- Keep accessibility labels and identifiers automation-friendly.
- Use Playwright only for browser-based surfaces, not as the primary macOS app test tool.
- Use Accessibility Inspector and Instruments when UI automation or performance needs deeper validation.

## Workflow

- Update OpenSpec artifacts before implementing changes that affect behavior, architecture, persistence, or UX.
- Keep tasks small, testable, and split UI work from service or persistence work.
- If adding docs, put durable guidance in `docs/` and keep this file concise.
- Before claiming completion, verify the exact build, test, or review step you changed.

For expanded guidance, see [docs/macos-agent-guide.md](/Users/tongtong/code/skills-manager/docs/macos-agent-guide.md:1).
