# DevFit — Smart System Compatibility & Developer Environment Optimizer

DevFit answers three questions for students and developers:

1. **Can my computer run this software or project?**
2. **What is causing compatibility problems or slowing down my computer?**
3. **What is the safest configuration for my system, and how can I apply it?**

This repository contains the **web build** (React + TypeScript + Vite + Tailwind). The full DevFit architecture targets a
Tauri desktop shell with an OS-level detection service; this build implements the same compatibility engine, project
analyzer, recommendation engine and reporting pipeline inside the browser sandbox — and is explicit about what the
sandbox can and cannot see.

## Core principles

- **Local-first, zero network calls.** The compatibility database ships with the app. No scan, project file or report ever
  leaves the device.
- **Evidence over claims.** Every data point is tagged `measured | reported | estimated | unknown`. Every compatibility
  check carries evidence and a confidence level. Verdicts (`ready / likely-ok / constrained / not-ready / insufficient-data`)
  explain the specific factors behind them — no fake universal scores.
- **Read-only diagnostics.** Project files are parsed as untrusted text, never executed (`postinstall` scripts are even
  flagged as a risk). The guided setup assistant shows commands and requires explicit consent; the user runs them.
- **Honest about limitations.** When the browser hides a value (RAM capped at 8 GB, no disk free-space, no CPU model),
  checks are marked *unverifiable* with exact manual verification instructions instead of guessing.

## Architecture

```
src/
  types.ts                 Shared domain types (DataPoint provenance, checks, verdicts, reports)
  lib/
    systemDetect.ts        Hardware/OS detection via browser APIs, with provenance tagging
    compatDb.ts            Software requirements DB (19 tools) with sources + verification dates
    compatEngine.ts        Rule-based requirement comparison → evidence-backed verdicts
    projectAnalyzer.ts     Safe parsers: package.json, lockfiles, requirements.txt, pyproject.toml,
                           pom.xml, build.gradle(.kts), Dockerfile, docker-compose, .csproj, .env templates
    history.ts             Local storage persistence, settings, JSON/Markdown export
  screens/                 Overview · System Info · Compatibility Explorer · Project Analyzer ·
                           Resource Monitor · Recommendations · Setup Assistant · History · Settings
  components/ui.tsx        Shared UI primitives (status badges, charts, code blocks, states)
tests/engine.test.ts       24 automated assertions for the engine and parsers
```

## Running

```bash
npm install
npm run dev      # development server
npm run build    # production build → dist/
npx tsx tests/engine.test.ts   # run the automated tests
```

## What the browser build can measure

| Capability | Source | Limitation |
|---|---|---|
| OS family/version, CPU arch | UA Client Hints | Firefox/Safari fall back to UA parsing (`reported`) |
| Logical CPU cores | `hardwareConcurrency` | No CPU model/clock speed |
| Installed RAM | `deviceMemory` | Capped at 8 GB; unavailable in Firefox/Safari |
| GPU model | WebGL unmasked renderer | No VRAM size |
| Disk | `storage.estimate()` | Quota only — a lower-bound signal, not free space |
| Tab memory / FPS / stalls | `performance.memory`, rAF, longtask observer | This tab only, not system-wide |

Everything the sandbox hides is listed in-app with exact manual check commands (Task Manager, `lscpu`, `df -h`, …).
The planned Tauri build replaces these probes with OS APIs (exact CPU, total/free RAM and disk, process list,
installed runtime versions) behind the same `DataPoint` interface — the engine and UI are unchanged.

## Security notes

- Strict read-only file handling; 2 MB per-manifest cap; `node_modules`, `.git`, build dirs skipped.
- No `eval`, no YAML/TOML execution — only line-based safe extraction.
- No credentials, no elevated privileges, no background processes.
- History and settings live in `localStorage` and can be wiped from Settings → “Delete all local data”.
