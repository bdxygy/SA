# Solo Studio AI — AI Agent Documentation

## Goal
This document describes the **AI Agent responsibilities, data-flow, and API interaction patterns** for the Solo Studio (SoloPhoto) app.

It is based on `docs/gemini.md` as a working reference implementation, and is intended to guide an AI agent (and humans) when:

- Updating `index.html` (Alpine / vanilla JS)
- Adding new photo workflows (tabs)
- Integrating Gemini / Imagen endpoints safely
- Keeping UI behavior consistent (upload → options → generate → results → download)

## Scope & Non-Goals
- **In scope**: client-side architecture, prompt-building strategy, request/response patterns, UI state machine.
- **Out of scope**: server-side proxying, authentication, billing, model fine-tuning.

---

## High-level Architecture

### UI Structure
The reference (`docs/gemini.md`) implements a **multi-tab single-page UI**:

- **Gabungkan Gambar** (merge 2–5 images)
- **Photoshoot Produk**
  - Produk Saja
  - Produk + Model
- **Pre+Wedding**
- **Foto Model**
  - Buat Model (Imagen)
  - Ubah Pose (Gemini image)
- **Edit Foto**
  - Inpaint (Gemini image + mask)
  - Enhance (Gemini image)
- **Bikin Banner Iklan** (Gemini text → JSON ideas → Gemini image)

Navigation is handled by:

- **Desktop sidebar** buttons
- **Mobile bottom nav** buttons + scroll indicators

### Execution Model
All interactions are **event-driven**:

- UI events (`click`, `input`, `change`, drag/drop)
- State variables held in JS (per feature)
- Async generation uses `fetch()` + `Promise.allSettled()` to run multiple variations concurrently

---

## Core Reusable Utilities (Patterns)

### 1) Image Upload Normalization
Reference function: `setupImageUpload(input, uploadArea, onFile)`

**Contract**: convert a selected file into:

- `base64` (no header)
- `mimeType`
- `dataUrl` (full `data:<mime>;base64,...`)

**Why this matters**:

- Gemini `inlineData.data` requires base64 *without* the `data:` prefix.
- UI preview uses `dataUrl`.

**Agent rule**:

- Always store both `base64` and `mimeType`.
- Always validate `file` existence before reading.
- Prefer a single canonical representation `{ base64, mimeType, dataUrl }`.

### 2) Option Buttons as Single-select Groups
Reference function: `setupOptionButtons(container)`

**Contract**:

- Buttons in a container behave like radio options.
- Selected button has class `selected`.
- Reading selection: `container.querySelector('.selected').dataset.value`

**Agent rule**:

- Any new option group must follow this pattern for consistency.

### 3) Aspect Ratio Mapping
Reference function: `getAspectRatioClass(ratio)`

- UI grid cards use Tailwind `aspect-*` classes.
- Prompts also inject `aspect ratio of X`.

**Agent rule**:

- Keep a single source of truth for supported ratios.
- Ensure the UI options and prompt enforcement are aligned.

### 4) Modal for Preview / Edit
Reference pattern:

- `universalModal` + `modalTitle` + `modalBody`
- Used for preview and for edit-instructions.

**Agent rule**:

- For any “view larger” feature, reuse the modal.
- For edit flows, pass card identifiers so the original result card can be updated.

---

## Gemini / Imagen API Integration

### General Principles
- **Never hardcode API keys** in source.
- Key is supplied by the user (input field) and may be stored in `localStorage` (optional).
- Always handle non-200 responses and attempt to parse error body safely.

### A) Gemini image / multimodal generation
Reference endpoint shape:

`POST https://generativelanguage.googleapis.com/v1beta/models/{MODEL}:generateContent?key={API_KEY}`

**Common models used**:

- `gemini-2.5-flash-image-preview` (image output)
- `gemini-2.5-flash-preview-05-20` (text-only / analysis / validation)

**Payload pattern**:

- `contents: [{ parts: [...] }]`
- `parts` can contain:
  - `{ text: "..." }`
  - `{ inlineData: { mimeType, data } }`
- For image output: `generationConfig: { responseModalities: ['IMAGE'] }`

**Response parsing pattern**:

- Find base64 image in:
  - `result.candidates[0].content.parts.find(p => p.inlineData)?.inlineData?.data`
- Convert to data URL:
  - `data:image/png;base64,${base64Data}`

**Agent rule**:

- Treat response as untrusted: guard all optional chains.
- If missing image data, throw a specific error like “No inlineData found”.

### B) Imagen generation (text-to-image)
Reference endpoint shape:

`POST https://generativelanguage.googleapis.com/v1beta/models/imagen-3.0-generate-002:predict?key={API_KEY}`

**Payload pattern**:

- `instances: [{ prompt: "..." }]`
- `parameters: { sampleCount: 1, aspectRatio: "1:1" }`

**Response parsing**:

- `result.predictions[0].bytesBase64Encoded`

**Agent rule**:

- This endpoint differs from Gemini. Do not reuse Gemini response parsing.

---

## Workflow Blueprints (Reference Features)

### 1) Gabungkan Gambar (2–5 images → 4 variations)
Key patterns:

- Dynamic upload slots:
  - min 2, max 5
  - auto-add a new empty slot when last is filled
- Magic prompt:
  - Gemini text model analyzes images → returns Indonesian instruction
- Generate:
  - create 4 placeholder cards
  - run `Promise.allSettled([1..4].map(ggGenerateSingleImage))`
  - each variation uses Gemini image model with all uploaded images in `parts`

**Agent notes**:

- Ensure `ggGenerateBtn.disabled` logic respects min images and prompt filled.

### 2) Photoshoot Produk — Product Only
Key patterns:

- Inputs:
  - one product image
  - lighting: `light|dark`
  - mood: `clean|crowd`
  - (conditional) location when mood = `crowd`: `indoor|outdoor`
  - ratio
- Prompt building:
  - Build `userInstructions` string
  - Create 4 effect prompts (shadow, bokeh, flying hero shot, studio)
- Generate:
  - 4 concurrent Gemini calls
  - Each result card supports:
    - download
    - view (modal)
    - edit (modal → regenerate edited image)

**Edit flow**:

- `showEditModal(imgSrc, imgTitle, cardId)`
- Convert current `imgSrc` back to base64 (split `data:` URL)
- Call `generateEditedImage(originalImageData, instruction)` (Gemini image model)
- Update the card’s `img`, button dataset, and download href

### 3) Photoshoot Produk — Product + Model
Key patterns:

- Two image uploads
- Prompt instructs: combine model + product photorealistically
- 4 variations concurrently

### 4) Pre+Wedding (2 persons + optional reference)
Key patterns:

- Validate that uploaded images contain a person:
  - Gemini text model with strict “yes/no” output
- Prompt emphasizes **identity preservation**
- Optional watermark text
- Optional reference image appended to `parts`

**Agent notes**:

- Validation is asynchronous; button enabling depends on `isValid` flags.

### 5) Model Generator
Two subflows:

- **Create model (Imagen)**
  - Optional “random prompt” generated via Gemini text model
  - Imagen predicts 4 results
- **Repose (Gemini image)**
  - Upload model image
  - choose preset pose or type custom
  - generate 4 variations

### 6) Edit Foto (Inpaint with mask)
Key patterns:

- Two canvases:
  - base image
  - drawing mask (pink strokes)
- Keep a mask history for undo/redo
- On generate:
  - create a full-size mask canvas scaled to original image size
  - send original image + mask image as `inlineData`
  - prompt instructs inpaint only inside mask

**Agent notes**:

- Mask must be PNG.
- Touch events use `{ passive: false }` to prevent scrolling.

### 7) Banner Generator
Key patterns:

- Upload image
- Auto-generate banner text (Gemini text)
- Auto-generate style suggestion (Gemini text)
- Generate:
  - `getBannerVariations()` requests a **JSON array** from Gemini
  - Uses `responseMimeType` and `responseSchema`
  - Then generates 4 banners using Gemini image model

**Agent rule**:

- JSON parsing must be wrapped in try/catch.
- Be strict about schema; fall back gracefully if model returns invalid JSON.

---

## Concurrency Pattern: “4 Variations”
Repeated across features:

- Render 4 placeholder cards
- Trigger 4 async requests
- Use `Promise.allSettled()` (not `Promise.all`) so one failure doesn’t cancel the others

**Agent rule**:

- Always restore button state in `finally`.
- Always show per-card errors in the UI (don’t fail silently).

---

## Error Handling & UX Rules

### HTTP errors
Minimum standard:

- If `!response.ok`, create a message containing:
  - status code
  - `error.message` from JSON body (if available)
  - `response.statusText` fallback

### Response errors
- If no image bytes are found, raise a structured error:
  - `"Invalid API response: no inlineData"`

### UI behavior
- Disable generate buttons when required inputs are missing.
- Show spinners in:
  - buttons
  - result cards

---

## Security & Privacy Notes (Client-side)

- API keys entered in the browser are sensitive.
- If storing in `localStorage`, provide a clear “clear key” / “forget key” option.
- Do not log API keys.
- Consider adding a server proxy later to avoid exposing keys in network logs.

---

## How the AI Agent Should Extend the App

When adding a new feature/tab:

1. **Define the input contract**
   - images required (0/1/2/…)
   - text fields
   - option groups

2. **Reuse utilities**
   - `setupImageUpload`
   - `setupOptionButtons`
   - `getAspectRatioClass`

3. **Define prompt templates**
   - include aspect ratio constraint
   - include “do not add watermark/text” if needed

4. **Implement generation**
   - render placeholder grid
   - do 4 variations with `Promise.allSettled`
   - parse response and attach download actions

5. **Add robust errors**
   - per-card failures should not break the whole page

---

## Implementation Checklist (Agent Self-check)

- **Inputs**
  - Upload produces `{ base64, mimeType, dataUrl }`
  - Generate button disables correctly

- **API calls**
  - Correct endpoint for chosen model (Gemini vs Imagen)
  - `generationConfig.responseModalities=['IMAGE']` for image output

- **Parsing**
  - Optional chaining used
  - Errors displayed when parsing fails

- **UX**
  - Spinners shown during requests
  - Buttons restore after completion
  - Download works on desktop and mobile

---

## Open Decisions (for this repo)

- Where the API key is stored (memory only vs `localStorage`). answer: memory only
- Whether to add a backend proxy to protect the API key. answer: no need
- Whether to standardize request building into a shared `callGemini()` function in `index.html` (recommended for maintainability). answer: yes, for cleaner code
