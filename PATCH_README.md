# NoteVault fork of flutter_quill

This is a minimal fork of [`singerdmx/flutter-quill`](https://github.com/singerdmx/flutter-quill)
used by **Extreme Note / NoteVault** via a `dependency_override` in its `pubspec.yaml`.

Branch **`notevault-autocorrect`** is cut from the upstream tag **`v11.5.1`** (the version the app
pins) and adds exactly one capability: letting the app **toggle the platform keyboard's autocorrect
and spell-check suggestions** (and the composing underline they draw) in the editor.

## Why the fork is needed

Upstream `v11.5.1` **hardcodes** the editor's `TextInputConfiguration` in
`lib/src/editor/raw_editor/raw_editor_state_text_input_client_mixin.dart`:

- `enableSuggestions: !widget.config.readOnly` (so `true` for an editable note)
- `autocorrect` is **omitted**, so it falls back to the Flutter default `true`

There is **no** `autocorrect` / `enableSuggestions` field on `QuillEditorConfig` to override them,
so the app cannot turn them off without forking.

## The patch (2 files)

1. **`lib/src/editor/config/editor_config.dart`** — added two fields to `QuillEditorConfig`,
   both defaulting to `true` (= upstream behavior), wired through the constructor and `copyWith`:
   - `final bool autocorrect;`
   - `final bool enableSuggestions;`

2. **`lib/src/editor/raw_editor/raw_editor_state_text_input_client_mixin.dart`** — the hardcoded
   `TextInputConfiguration` now reads from config:
   - `autocorrect: widget.config.autocorrect,`
   - `enableSuggestions: widget.config.enableSuggestions && !widget.config.readOnly,`

Defaults reproduce upstream behavior exactly, so the change is **back-compatible** — existing
callers that don't set the new fields are unaffected.

## How to refresh against a future flutter_quill release

1. `git fetch upstream` (upstream = `singerdmx/flutter-quill`).
2. Branch from the new tag (e.g. `v11.6.0`): `git checkout -b notevault-autocorrect-v11.6.0 v11.6.0`.
3. Re-apply the two edits above (check whether upstream has since added native
   `autocorrect`/`enableSuggestions` support — if so, **drop this fork** and use upstream).
4. Push, then update the `ref:` in NoteVault's `pubspec.yaml` `dependency_override`.
