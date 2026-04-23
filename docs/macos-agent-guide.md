# macOS App Guide

## Scope

This repo targets a native macOS app for managing SKILL.md content across AI coding agents. Treat the disk as the source of truth, and keep SwiftUI, file sync, and system integration decisions explicit.

## Architecture

- Prefer native SwiftUI for app structure, settings, menus, and window flows.
- Use AppKit only when SwiftUI cannot provide the required macOS behavior.
- Keep views presentation-only. Put coordination in view models or services.
- Keep file sync, snapshots, watchers, bookmarks, and persistence in dedicated services with clear ownership.
- Preserve one-way dependencies: Views -> ViewModels -> Services -> Persistence and file system.

## State And Data Flow

- Use `@Observable` for app-owned state and keep view-local state in `@State`.
- Inject shared services instead of recreating them inside views.
- Keep derived UI state close to the UI; keep domain decisions in services.
- Make source-of-truth decisions explicit for every sync feature: library, target directory, or database index.

## File System And Sync

- Favor symlinks as the default sync strategy. Document any fallback to file copies.
- Normalize and compare paths consistently, including symlink resolution strategy.
- Define conflict handling before implementation: existing file, broken symlink, renamed directory, deleted source, concurrent edits.
- Treat destructive operations as opt-in. Document confirmation, backup, and rollback behavior.
- Keep GRDB limited to metadata, indexing, snapshots, and sync state. Do not store full skill bodies there.

## Watchers And Background Work

- Use `FSEventStream` for directory-tree monitoring.
- Use `DispatchSource.makeFileSystemObjectSource` only for narrow supplemental file or single-directory observation.
- Design for debounce, event coalescing, cold-start scans, and self-trigger suppression after writes.
- Explain battery and performance impact whenever watcher scope expands.

## Permissions And System Integration

- Assume a non-sandboxed macOS app unless requirements explicitly change.
- Use security-scoped bookmarks to remember user-selected locations and restore them across launches; do not describe them as sandbox permission renewal.
- Use MenuBar or window-scene APIs in SwiftUI before introducing AppKit wrappers.
- Keep secrets out of plist and user defaults; use Keychain for credentials.

## Errors And User Feedback

- Prefer typed throws on domain and service boundaries.
- Wrap file system, database, and third-party errors at boundary layers before surfacing them.
- Every user-visible failure must have a clear recovery path: retry, reselect directory, reveal conflict, or open logs.

## Testing Strategy

- Unit test parsing, sync decisions, migrations, bookmark restoration, and conflict rules.
- Use `XCTest` or Swift Testing for logic tests.
- Use `XCUITest` for stable macOS UI regressions and end-to-end flows.
- Add accessibility labels and identifiers so UI automation and AI-driven inspection stay reliable.
- Use Playwright only for web content or browser-based tooling, not as the main macOS app test harness.
- Use Accessibility Inspector to validate automation surfaces and accessibility regressions.
- Use Instruments or `xctrace` for launch, CPU, memory, and responsiveness investigations.

## Documentation Expectations

- Changes affecting sync, watchers, permissions, or migrations must update OpenSpec artifacts first.
- Keep `AGENT.md` short and action-oriented; put durable rationale and expanded guidance in docs.
- When adding a new service or persistence field, document ownership, migration plan, and test coverage.
