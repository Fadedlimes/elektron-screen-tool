This file is the harness-neutral context document. `CLAUDE.md` is a symlink to it; other agentic
harnesses get their own symlink to the same file.

**First contact (fresh clone):** the harness-specific files (`CLAUDE.md`, `GEMINI.md`, `AGENTS.md`,
…) are gitignored, so on a new machine the user will ask the agent to read this file. Then: create
your harness's instruction file as a link to `CONTEXT.md` (relative symlink; on Windows a one-line
`@CONTEXT.md` import or a copy if symlinks are unavailable), and ask the user where the
shared-memory repo is before creating `.shared_memory` (see Shared memory).

## What this is

`screendump` renders an Elektron instrument's screen in the terminal, over MIDI, using the
instrument's own stock screenshot command. **Nothing on the instrument is modified.** One file, one
dependency (`python-rtmidi`), macOS/Linux/Windows. `README.md` is the user-facing doc; the wire
format and pixel packing are in the comment block at the top of the script.

## Hard rules

- ⛔ **`-x` / `--external` and `--bench` SEND MIDI to real hardware. Never run them unasked** — a
  per-occasion go from the user each time, not a standing one. Every other mode is offline and free.
- ⛔ **An instrument is `implemented` only after a real screen has been captured from it and read.**
  Never on the strength of a datasheet, a forum post, or a plausible-looking id. Without its two id
  bytes an instrument is not merely untested, it is unreachable — there is no request to send.
- ⛔ **The user owns git**: commits, branches, tags, pushes, merges. Never create any unasked.

## How we work

- The user leads. Discussion is the default — no unprompted plans, no option-pickers, no building
  until explicitly told to.
- Investigation is work, not a delay before work.
- Address every topic in a multi-topic message, in the user's order; name anything deferred.

## Verifying

- `--self-test` must pass offline; it derives its expected sizes from the instrument's geometry, so
  it stays correct for a newly added machine.
- **A rewrite that claims to change nothing must prove it** — byte-identical output against the
  thing it replaces, on the same captured frame, *before* the old one is deleted.
- Two failure modes when adding an instrument, both of which look convincing at a glance:
  **sheared across the middle** ⇒ the pixel offset is wrong, compare the frame size on the wire
  against what the geometry implies; **upside down** ⇒ try `--flip`, and confirm against **text** on
  the screen, never an icon — a vertical flip leaves graphics looking perfectly plausible.

## Shared memory

Durable notes live in a separate shared-memory git repo, not in the harness's own memory store.
Which repo that is, is decided per machine by the user and is deliberately **not** named here.

**Mount (local, untracked):** `.shared_memory` in the repo root → the shared-memory repo, as a
relative link. Gitignored as `/.shared_memory`. Never stage it or a gitlink to it.

**If `.shared_memory` is missing or dangling:** tell the user and stop. Do not create the link or
guess a location.

**At session start:** read `.shared_memory/MEMORY.md`, then the index for this repo's topic if one
exists, then `shared/INDEX.md`, then `user/<name>/`. This tool's history currently lives under the
`elektron_hack` topic, where the format was reverse-engineered.

**Writing:** "persist" = write/update files under `.shared_memory/` only, no git. "synchronise" =
pull (merge, never rebase), add, commit, push, **inside the shared-memory repo**. Memory is on
request only.
