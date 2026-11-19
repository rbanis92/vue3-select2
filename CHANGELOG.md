# Changelog

## 0.2.1 — unreleased

- Await plugin loading instead of racing a timer; prevent initialization after unmount.
- Guard early prop updates and clean up owned event handlers.
- Avoid mutating option objects; ignore null entries and tolerate non-array options.
- Synchronize clear/change events and declare documented events.
- Preserve caller search settings and react to settings/mapping changes.
- Accept numeric model values and return an object from the settings default.
- Load jQuery locally; a global jQuery assignment is no longer required.
- Require jQuery >=3.7.1 and Select2 >=4.0.13 within their existing major versions.

Validation: source review only for this release candidate. Runtime regression tests,
build verification, and a complete dependency audit remain required before publishing.
