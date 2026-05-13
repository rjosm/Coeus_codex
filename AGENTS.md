# Coeus — AGENTS.md

## What this app is

Coeus is a digital health equity grocery tool. It helps people make informed product decisions in-store by comparing grocery products against their own priorities.

The app has two major product capabilities:

1. A scoring and comparison engine that ranks products using a user's selected priorities, preferences, and budget sensitivity.
2. A barcode scanning flow that lets users identify products in-store quickly.

Today, manual lookup is the safest working input path and must remain stable. Over time, barcode scanning should be treated as a first-class product capability and prioritized accordingly.

Allergen-aware product handling is also an important upcoming capability. The UI concept exists now, but full allergen filtering still needs to be implemented and preserved once added.

## Core product priorities

When making changes, preserve and prioritize these areas in this order:

1. Do not break the scoring engine.
2. Do not break product lookup from the CSV data source.
3. Do not break the comparison results flow.
4. Keep manual search working as a reliable fallback at all times.
5. Improve barcode scanning toward being a primary user input path.
6. Build allergen filtering in a way that integrates with lookup, scoring, and comparison without weakening the current experience.

## Current architecture

- Frontend: single-file HTML/CSS/JavaScript, no framework
- Hosting: Vercel
- Repo: GitHub-backed static site
- Product data source: CSV file stored in the GitHub repo and fetched at runtime from raw GitHub
- Backend: none for core app logic
- Data management helpers: Google Sheets + Apps Script for operational logging and support workflows

There is no build step. There is no npm install requirement for the current web app.

## Repository and data source

- GitHub repository: `https://github.com/rjosm/Coeusv2/`
- CSV data source: `https://raw.githubusercontent.com/rjosm/Coeusv2/refs/heads/main/Product-UPCs%20-%20Automated%20UPC%20Look%20up.csv`

## Local development

Run locally by either:

- opening `index.html` directly in a browser, or
- using a simple static server such as `npx serve .` or VS Code Live Server

The current app should continue to work as a static client-side application.

## Current code reality

The present app is effectively centered in `index.html`.

Important logic currently lives inline:

- CSV fetch and parsing
- product normalization
- product lookup by GTIN
- manual search and selection
- focus and preference capture
- scoring and ranking
- barcode scanning flow
- results rendering

Supporting PWA files:

- `manifest.json`
- `sw.js`

## What must be preserved at all costs

### 1. Scoring engine

Preserve the current comparison model and preference-driven ranking behavior.

This includes:

- the current product scoring flow
- normalized cross-product comparison
- variable-based priorities such as sugar, cost, fiber, sodium, protein, ingredients, and calories
- user-defined direction and weight for each selected priority

Do not rewrite this casually. If it is extracted or reused elsewhere, preserve behavior exactly unless a change is explicitly intended.

### 2. CSV data model and lookup

Preserve the CSV-driven product system.

This includes:

- fetching product data from the GitHub-hosted CSV at runtime
- row-to-product mapping
- GTIN-based matching
- tolerance for barcode formatting differences such as leading zeroes and spacing

If scanning logic changes, product matching should still resolve through the same lookup model unless there is a deliberate migration plan.

### 3. Manual search

Manual product search by name or barcode number is the current launch-safe path.

Do not break:

- free-text product search
- brand/name suggestion matching
- direct numeric barcode entry
- slot-based product selection for comparison

Any scanner work must fail gracefully back to this flow.

### 4. Comparison results flow

Preserve the current user journey:

1. choose focus areas
2. refine preferences
3. select products
4. compare products
5. inspect detailed product info

Avoid redesigns or refactors that disrupt this flow unless explicitly requested.

## Barcode scanning guidance

### Product direction

Scanning is not just a nice-to-have. It is expected to become more important than manual lookup over time. Design changes with that future in mind.

Short term:

- keep manual search as the reliable fallback
- improve scanning without destabilizing the rest of the app

Medium term:

- treat scanning as a primary in-store input path
- make product resolution from scan feel fast and dependable

### Current state

The current scanner implementation uses Quagga and is only partially reliable.

Known history:

- ZXing was tried and camera access worked, but read reliability was poor
- Quagga was tried and camera access worked, but behavior became unstable and the video feed could go black
- browser camera constraints are a real limiting factor in store conditions

### Required next attempt

Before introducing any new JS barcode dependency, test the native `BarcodeDetector` API.

Required formats:

- `ean_13`
- `upc_a`

Expected behavior:

1. user taps scan
2. app tries `BarcodeDetector`
3. if a code is detected, normalize it and look it up in the CSV-backed product set
4. if matched, fill the selected product slot
5. if not matched, log the missing GTIN and show the existing missing-product path
6. if `BarcodeDetector` is unavailable or unsupported, route the user back to manual search cleanly

Do not add a new scanning library unless `BarcodeDetector` is tested and clearly insufficient for the target Android Chrome use case.

### Testing expectations

Barcode work must be tested on a physical Android device running Chrome.

Desktop testing is not enough. Real grocery packaging and real store lighting matter.

## Allergen guidance

Allergen support is a priority feature and should be treated as product-significant work, not cosmetic UI.

Current reality:

- the app has an allergen profile UI concept
- allergen filtering is not yet fully enforced in the scoring and lookup workflow

Future requirement:

- users should be able to express allergens and avoid unsafe products
- allergen logic should integrate with product filtering, lookup behavior, and comparison outcomes
- this must be implemented carefully so it does not corrupt the existing scoring engine

When allergen filtering is added, document the exact source of truth and matching rules clearly.

## Google Sheets / Apps Script role

Google Sheets currently supports operational workflows such as:

- focus combination tracking
- missing GTIN logging
- price entry logging

Preserve these integrations unless there is an explicit decision to retire or replace them.

## Migration guidance: Expo / React Native

The long-term goal is to migrate Coeus from a PWA to a React Native Expo app.

This should be done incrementally.

### Migration rule

Do not rewrite the whole app in one pass.

### Preserve during migration

- scoring engine behavior
- CSV-based lookup model
- product data structure
- comparison flow
- future allergen behavior

### Recommended first step

The first Expo phase should focus on:

- scaffolding the Expo app structure
- porting routing/navigation
- porting the scoring/comparison experience
- reusing existing logic rather than rethinking it

### Native camera direction

For the native app, barcode scanning should use:

- VisionCamera
- MLKit

## Change rules for future agents

- Prefer minimal, behavior-preserving changes over broad rewrites.
- Do not introduce a backend just to solve client-side logic that already works.
- Do not replace the CSV data layer without explicit approval.
- Do not break manual search while improving scanning.
- Do not change scoring behavior silently.
- Treat scanning reliability and allergen support as strategic product work.
- If extracting logic for reuse, keep the current outputs stable and test against existing behavior.

## Practical definition of success

A successful near-term change:

- keeps the current app lightweight
- preserves the scoring engine
- preserves manual search
- improves scanning safely
- creates a clean path for future allergen enforcement
- does not block the Expo migration path

## Notes for future implementation work

- Be honest about current-state limitations in the codebase.
- Distinguish clearly between what exists now and what is planned.
- When uncertain, protect user trust first: incorrect product identification is worse than falling back to manual search.
