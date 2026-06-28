# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A to-do list app contained entirely in a single file: `index.html` (HTML + CSS in `<style>` + vanilla JavaScript in `<script>`). No framework, no build step, no dependencies, no tests. `version.txt` is a plain-text version marker.

## Running

- Open `index.html` directly in a browser (`open index.html`) — it works from a `file://` URL since everything is self-contained.
- For browser-automation/verification tools, `file://` URLs are blocked. Serve over HTTP instead: `python3 -m http.server 8753` then load `http://localhost:8753/index.html`. (Note: `localStorage` is per-origin, so data from a `file://` session and a `localhost` session are separate.)

There are no build, lint, or test commands.

## Architecture

- **State**: a single global `tasks` array of `{ text: string, done: boolean }`, persisted to `localStorage` under the key `todo-tasks` via `loadTasks()` / `saveTasks()`.
- **Rendering**: `render()` is the single source of UI truth — it clears and rebuilds the entire task list on every call. The pattern after any mutation is always: mutate `tasks` → `saveTasks(tasks)` → `render()`.
- **Filter** (`All` / `Active` / `Completed`): an in-memory `filter` variable, **not** persisted — it resets to `all` on reload.
- The **Clear completed** button's visibility is derived in `render()` from `tasks.some(t => t.done)`.

## Critical gotcha: index-based mutations in `render()`

`render()` iterates the *full* `tasks` array with `tasks.forEach((task, i) => …)` and uses the index `i` directly for both toggle (`tasks[i].done = …`) and delete (`tasks.splice(i, 1)`).

Do **not** pre-filter the array before rendering — that would desync `i` from the real positions and mutate the wrong task. To hide tasks (e.g. the filter feature), skip non-matching ones *inside* the loop (`if (...) return;`) so `i` stays aligned with the underlying `tasks` array.
