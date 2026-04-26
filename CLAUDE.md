# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**ContactSystem** (聯絡單管理系統) is a single-file web application for managing and printing class contact forms in Traditional Chinese. It's designed for teachers to create, edit, and distribute weekly class newsletters that can be printed in landscape A4 format with two pages side-by-side for efficient paper use.

## Architecture

### Core Structure
- **Single HTML file** (`ContactSystem.html`) containing HTML, CSS, and JavaScript
- No build system, bundling, or transpilation required
- Runs entirely in the browser with cloud backend integration
- `index.html` is a redirect to the main application
- **Framework**: Vanilla JavaScript + Tailwind CSS + Supabase

### Key Components

**Frontend UI**
- Authentication screen with email/password login
- Main app with sticky top control bar (Print, Edit Toggle, Logout buttons)
- Printable form layout with two A4 half-pages side-by-side (`print-container`)
- 5 print containers total (10 half-pages: p1-p10)
- Edit mode that makes left-page content editable with contentEditable
- Status notification bar at bottom

**Data Model**
- All editable content uses `.editable-field` class with `data-key` attributes
- Keys follow pattern: `p{n}-{fieldname}` (e.g., `p1-title`, `p1-lesson`)
- Odd pages (p1, p3, p5, p7, p9) are left column; even pages (p2, p4, p6, p8, p10) are right column
- Each page contains: title, date, lessons, homework, communication notes, and 20 vocabulary items

**Backend Integration**
- Supabase (PostgreSQL + Auth)
- Table: `contact_drafts(user_id, draft_data)` — stores user content as JSON
- Auth: Email/password with password recovery flow

### Data Flow

**On Application Load:**
1. Supabase auth state change listener activates
2. If authenticated: load draft from Supabase, fall back to localStorage
3. Apply loaded data to all `.editable-field` elements
4. Sync odd pages (left) to even pages (right) via `syncOddToEven()`

**On Content Edit:**
1. User toggles edit mode → makes left-page fields `contentEditable="true"`
2. User edits field → `blur` or `input` event triggers
3. `saveDraft()` collects all field data and saves to localStorage
4. Debounced (1500ms) `syncToSupabase()` uploads to cloud
5. `syncOddToEven()` mirrors changes to right pages in real-time

**On Print:**
1. Exit edit mode if active
2. CSS media query hides all `.no-print` elements
3. Browser print dialog opens for A4 landscape
4. Two pages render per sheet with dashed cut line between

## Key Implementation Details

### Edit Mode Restrictions
- Only odd-page fields (p1-,p3-,p5-,p7-,p9-) are editable
- Even-page fields are read-only mirrors (for left-to-right mirroring)
- Toggle controlled by `isEditMode` flag and `.edit-mode` body class
- Button styling changes when active (blue → emerald)

### Synchronization
- Local storage key: `'s2_contact_draft'`
- Cloud sync uses UPSERT on `contact_drafts` table keyed by `user_id`
- Debounce prevents excessive cloud calls (1500ms delay)
- Status messages provide user feedback on sync success/failure
- Password recovery flow: auth listener detects `PASSWORD_RECOVERY` event and prompts for new password

### Print Styling
- A4 landscape: 297mm × 210mm
- Two-column grid layout (148.5mm each)
- Dashed border between pages for cutting guide
- Print media query removes UI elements and backgrounds
- All content boxes, headers, and vocabulary grids preserve borders and spacing

### Chinese Localization
- Noto Sans TC font imported from Google Fonts
- All UI text and labels in Traditional Chinese
- Form content examples include Chinese text
- No i18n library; text is hard-coded

## Common Development Tasks

**To modify form structure:**
- Edit the HTML within `.a4-half-page` divs (lines 230–845)
- Add new fields with `.editable-field` class and unique `data-key`
- For pages 1–4, add to both left (`page-left` / `page-left-2`) and right (`page-right` / `page-right-2`)
- For pages 5–10, add to `page-left-3`, `page-left-4`, `page-left-5`, etc.
- Follow naming convention: p1-, p2-, p3-, ... p10-

**To add a new field:**
1. Wrap content in `<div class="editable-field" data-key="p{n}-{fieldname}"></div>`
2. For odd pages only (they will auto-sync to even pages)
3. JavaScript will automatically enable editing on toggle if `contentEditable` is set to false initially

**To change default content:**
- Edit the inner HTML of any `.editable-field` element
- Text and HTML changes persist on load via Supabase/localStorage

**To modify styling:**
- CSS is embedded in the `<style>` block (lines 9–177)
- Tailwind CSS classes are used in HTML for responsive design
- Media query `@media print` controls print-specific styling (line 61–67)
- Color scheme uses CSS variables: `--primary-blue`, `--light-blue-bg`, `--border-gray`, `--text-main`

**To test authentication:**
- Supabase project credentials are in the JavaScript (lines 851–854)
- Password recovery uses Supabase's built-in reset flow
- Local development: use any email registered in the Supabase project

## Debugging Tips

- **Check localStorage**: Open DevTools Console, run `localStorage.getItem('s2_contact_draft')`
- **Monitor Supabase calls**: All API calls log errors to console via `console.error()`
- **Edit mode not working**: Verify `.editable-field` elements exist and only odd-page fields are contentEditable
- **Print issues**: Check that A4 landscape margins are set in browser print settings; dashed lines should appear between pages
- **Sync failures**: Check network tab and Supabase error messages; local draft is preserved if cloud fails

## Performance Considerations

- Single HTML file keeps load time minimal; no separate requests for CSS/JS modules
- `syncOddToEven()` runs on every edit but only processes ~40 fields (p1–p10 vocabulary + content)
- Debounced Supabase sync prevents throttling
- localStorage provides instant offline access
- No lazy-loading needed; all pages render on load

## Supabase Setup

The application assumes these exist in the connected Supabase project:
- Auth users table (built-in)
- `contact_drafts` table with columns:
  - `user_id` (UUID, primary key)
  - `draft_data` (JSONB)
- Row-level security disabled (public read/write for authenticated users)

Do not alter the table structure without updating the JavaScript queries.
