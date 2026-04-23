# Varinomics Review Scope

This document governs what reviewers - human or LLM - must *not* flag when reviewing Varinomics code. It applies in addition to the coding-style documents; the coding-style docs govern what you *write*, this document governs what you *critique*.

## Principle

Evaluate code against the project's actual scope, not against hypothetical scopes that are not in play.

Review bandwidth is finite and noise displaces signal. A report that mixes real issues with warnings about problems the project will never have trains the reader to ignore the real issues. A reviewer who flags out-of-scope concerns anyway is not being thorough; they are being *miscalibrated*. Calibration means reporting what matters to *this* project, based on what the project actually targets.

When in doubt about whether a concern is in scope, check the repository's `CLAUDE.md`, `AGENTS.md`, README, and the code itself for evidence of the target. If the evidence is mixed, ask the user before writing the finding.

---

## Rule 1 - Platform portability is a statement, not an assumption

Do not flag a portability concern for a platform the project does not target.

- **Windows-only projects:** do not raise "this would break on Linux/macOS" unless the project declares intent to port, or the file is an explicit cross-platform abstraction layer. Qt projects built only for Windows, using `#ifdef Q_OS_WIN` throughout, depending on `QWinExtras`, Win32 APIs, or DirectX directly, are Windows projects. Treat them as such.
- **x86-only projects:** do not raise "this would break on ARM" unless ARM builds are actually declared (CI matrix with ARM targets, NEON intrinsics alongside SSE, ARM toolchain files checked in). A single `_mm_*` intrinsic in a Windows x86_64 target is not a portability bug.
- **Desktop-only projects:** do not raise "this would fail on mobile" or flag touch-first UX concerns.

Portability becomes in-scope when the evidence supports it: cross-platform CI, abstraction layers used consistently, explicit portability goals stated in the docs. Absent that evidence, portability concerns are out of scope for this review.

## Rule 2 - Internationalization is a statement, not an assumption

Do not flag "this should go through `qsTr()` / `tr()` / equivalent" for bare strings unless evidence shows the application is meant to be multi-language.

Evidence that internationalization is in scope:

- `.ts` translation files present and maintained in the repo.
- `qsTr()` / `tr()` used consistently across the existing UI, not just in isolated spots.
- A translation workflow described in docs, build scripts, or CI.
- Multiple locales declared in the app's deployment configuration.

Absent that evidence, bare string literals in C++ or QML user-facing code are not an issue. A single stray `tr()` left over from scaffolding is not evidence - the question is whether the application actually ships in multiple languages.

## Rule 3 - Accessibility is a statement, not an assumption

Do not flag the absence of `Accessible.role`, `Accessible.name`, ARIA equivalents, keyboard-navigation affordances, or screen-reader hooks unless the project states accessibility as a requirement, or uses these features consistently throughout the existing UI.

Covers:

- Missing `Accessible.*` properties on QML controls.
- Absent tab-order configuration.
- Lack of screen-reader labels on new elements.
- No high-contrast or font-size override support.

A project that does not currently support accessibility is not "wrong" - accessibility is a product decision with real implementation cost, and the decision to defer it is a decision. If the project's docs (README, `CLAUDE.md`, or equivalent) state that accessibility is not supported, treat that as authoritative.

If you believe accessibility *ought* to be a requirement for a given product, raise it as a product question to the user in a separate channel. Do not file it as a code-review finding.

---

## When the target is unclear

If the scope for platforms, languages, or accessibility isn't obvious from the repo:

1. Check `CLAUDE.md`, `AGENTS.md`, and README for explicit statements.
2. Look at the code: which patterns are used *consistently*, not just once?
3. If still unclear, ask the user what's in scope before writing the report. Do not fill the report with hypotheticals.

## Summary

Every review concern belongs in one of two buckets:

- **In scope** - the thing the project actually has to do, per its stated goals and the evidence of the code. Report it.
- **Out of scope** - everything else. Leave it out. Out-of-scope items in a review report are not "nice-to-haves"; they are noise that crowds out the real issues.

The three rules above are specific instances of this principle. They are not an exhaustive list - the principle applies to any concern that assumes a scope the project has not declared. Other examples include cloud-deployment concerns for a desktop app, database-scaling concerns for a single-user tool, real-time-latency concerns for an offline batch pipeline.
