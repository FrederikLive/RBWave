# SpectraWave (Fred's RBWave) - Agents Operating Contract

## What is this project?
A Max for Live Audio Effect that displays a Rekordbox-style 3-band multicolor waveform for Ableton audio clips.

## Where is everything?
- Root directory contains project configuration.
- `IMPLEMENTATION_PLAN.md` contains the primary build specification.
- `docs/` will contain specific domain knowledge.

## Setup & Validation
- Node.js is required for the analysis service (`node.script`).
- The project targets Ableton Live 12.x.
- Do not build a monolithic Max patch. Maintain clear architectural boundaries between the LOM Adapter, Clip Controller, Analysis Service (Node), Warp Mapper, and Renderer (v8ui + MGraphics).

## Rules
- **Do not alter Ableton's native waveform.**
- **Do not run heavy audio analysis synchronously on the Max UI scheduler.** Use Node for Max.
- Follow the detailed specifications in `IMPLEMENTATION_PLAN.md`.
- Read `PROJECT_BOOTSTRAP.md` for core AI-native operating rules.
