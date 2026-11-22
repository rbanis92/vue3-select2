# Changelog

All notable changes to `vue3-select2` are documented here.
The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and the project uses
[Semantic Versioning](https://semver.org/) (pre-1.0: minor versions may contain behaviour changes).

## [0.3.0] - 2026-11-22

### Added
- `multiple` prop, bound to the `<select multiple>` attribute (no need to pass it through `settings`).
- Live AJAX search: `searchUrl`, `searchParams` (read on every request, so a parent field can narrow
  results without a rebuild), `searchDelay` and `searchResults` (custom response mapper).
  Responses may be `{results, pagination: {more}}` or a plain array, and are mapped through
  `id_field` / `text_field`.
- `preselected` prop (array of items or a single `{id, text}` item) that supplies labels for selected
  values that are not in `options`, such as a saved value with live search.
- `dropdownParent` prop (selector string, HTMLElement or jQuery object) for use inside modals.
  `settings.dropdownParent` still takes precedence; `dParent` is accepted as a deprecated alias.
- `isolate` prop: ignore `selected` flags on option items so only `v-model` decides the selection.
- `options` can also be an object map (`{a: {...}, b: {...}}`); its values are used as items.
- Option groups: `children` arrays are normalized with `id_field` / `text_field` too.
- New events: `unselect`, `clear`, `open`, `close`.
- Methods for template refs: `open()`, `close()`, `focus()`, `getInstance()`.
- Re-initialises when `multiple`, `isolate`, `searchUrl`, `searchDelay` or `dropdownParent` change.
- `vue` declared as a peer dependency; `CHANGELOG.md` is included in the npm package.

### Changed
- Default `width` is now `'100%'` (was select2's `'resolve'`, which often renders 0px inside hidden
  tabs or modals). Override with `:settings="{ width: 'resolve' }"`.
- With `searchUrl`, `minimumResultsForSearch` defaults to `0` so the search box is always shown.
  It still defaults to `10` otherwise, and `settings` can override both.
- Replacing `options` with an array that has the same content no longer rebuilds the widget.
- Setting `v-model` to the value it already has is a no-op, so select2 isn't re-triggered.
- jQuery and select2 are loaded once and shared across all instances.

### Fixed
- `null` / `undefined` / `[]` values now clear the selection consistently.
- An empty-string value no longer creates a blank selected option when no label is given, so the
  placeholder stays visible.
- Teardown closes an open dropdown first (so `close` listeners run) and removes select2's leftover
  `scroll.select2.<id>` handlers from ancestor elements.
- Empty `id` / `name` props no longer render empty `id=""` / `name=""` attributes.

## [0.2.1] - 2026-11-19

### Fixed
- jQuery and select2 are loaded inside `mounted()` (async) instead of a top-level dynamic import race;
  load failures emit `error`.
- Removed the 300 ms `setTimeout` from mount; unmounting before init no longer throws.
- Re-initialising on options change destroys the previous instance first (events no longer fire twice).
- `settings` default is a real object; `settings`, `placeholder`, `id_field`, `text_field` and
  `text_sep` are watched.
- Option items are copied instead of mutated (no more deep-watcher loops on reactive data).
- `modelValue` accepts `Number`; `update:modelValue`, `change`, `select` and `error` are declared in `emits`.
- `minimumResultsForSearch` can be overridden from `settings`.
- Dependencies bumped to jquery `^3.7.1` and select2 `^4.0.13`.

## [0.2.0] and earlier

Initial releases: `v-model`, `options`, `settings`, `id_field`, `text_field`, `text_sep`, `select` event.

[0.3.0]: https://github.com/rbanis92/vue3-select2/compare/v0.2.1...v0.3.0
[0.2.1]: https://github.com/rbanis92/vue3-select2/releases/tag/v0.2.1
