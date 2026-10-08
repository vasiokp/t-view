# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file proof of concept (`index.html`) for reading employee badges on an Android device (apparently a ProDVX display with an NFC module, judging by the included PDFs). The page shows a clock and displays whatever badge UID the reader produces, with diagnostics meant for remote testers to send back.

Hardware reference docs live in the repo root: `9010100_Pogo_NFC_Module_Datasheet.pdf` (the pogo-pin NFC module) and `ProDVX NFC Solutions Guide.pdf`. They are git-ignored, so they only exist in local checkouts.

## Build / run

There is no build, package manager, lint or test setup. It is plain HTML/CSS/JS with no dependencies. To try it, open `index.html` in a browser or serve the folder statically (e.g. `npx serve .`). Note that the Clipboard API only works in a secure context (HTTPS or localhost), so over plain HTTP the "Copy report" button falls back to a textarea.

Keep it as a single self-contained file with no external scripts, since it gets loaded on the device as-is.

## How badge input works

The reader is assumed to act as a **HID keyboard wedge**: it types the UID as fast keystrokes, followed by Enter. The script has two input paths, and both end in `handleKeyboardValue()`:

1. **Global `keydown` listener**: it buffers single-character keys and flushes on Enter/Tab or after `IDLE_FLUSH_MS` of inactivity. Inputs shorter than `MIN_LENGTH` are logged as warnings and dropped.
2. **The `#test-input` field**: it reads the field's `value` on `input` events instead of key events. This exists because Android IMEs may send `key === "Unidentified"` / `keyCode 229` with no usable character, which breaks path 1. The global listener skips events whose target is this field.

`interpretUid()` shows decimal/hex conversions and the byte-reversed form, because readers differ in UID format and byte order. Use `BigInt`, since UIDs can be longer than 53 bits.

Results go to `render()` (the "last badge" card, styled `ok`/`error`) and `log()` (the event log, capped at 100 entries). Global `error`/`unhandledrejection` handlers send uncaught failures to the UI, so errors show on the device without devtools.
