# Handoff: Filter-Combobox + Benutzertabelle

## Overview
A combined **search-and-filter combobox** sitting above a **user data table**. The combobox merges free-text search and faceted filters (role, last-online, status) into a *single* token-input field: you type to search, and active facets appear as removable chips/tokens inside the same field. Search text and facets are applied together (AND logic) to filter the table live.

This bundle isolates **only the combobox and the table** from a larger “Schule → Benutzer” admin screen.

## About the Design Files
The file in this bundle (`filter-table.html`) is a **design reference created in HTML/React-via-Babel** — a working prototype that shows the intended look and behavior. It is **not production code to copy directly**. The task is to **recreate this design in the target codebase’s existing environment** (React, Vue, Svelte, Angular, native, etc.), using its established component library, styling system, and data layer. If no front-end environment exists yet, choose the most appropriate framework for the project and implement it there.

The prototype uses inline styles and a single in-file React component purely for portability. In a real codebase you would split this into components and use the project’s styling approach (CSS modules, Tailwind, styled-components, design-system primitives, etc.).

## Fidelity
**High-fidelity (hifi).** Colors, spacing, typography, and interactions are final. Recreate the UI faithfully using the codebase’s existing primitives. Exact values are listed under **Design Tokens**.

## Screens / Views

### View: User list with combined filter
- **Purpose:** Let an admin find users by typing a name AND/OR narrowing by role, last-online recency, and account status — all from one control.
- **Layout (top → bottom):**
  1. **Toolbar** — horizontal flex row, padding `18px 28px 12px`. Contains the combobox (`flex: 1 1 440px`, `min-width 260`, `max-width 660`). In the full app this row also held an “Aktion” menu and a primary “Benutzer hinzufügen” button on the right — omitted here.
  2. **Table** — scrollable region (`flex: 1; overflow: auto`), fixed table layout.
  3. **Footer** — count line, padding `8px 28px`, top border `1px solid #f1f5f9`.

## Components

### 1. Filter-Combobox (the core component)
A single bordered field that holds, left → right: a search icon, zero or more **facet tokens**, a growing **text input**, and a **clear-all (×)** button. Clicking anywhere in the field focuses the input. Focusing/typing opens a **dropdown** below it.

**Field container**
- Display: `flex`, `align-items: center`, `flex-wrap: wrap`, `gap: 6px`
- Padding: `4px 8px 4px 32px` (left padding leaves room for the absolutely-positioned search icon at `left:10 top:10`)
- Border: `1.5px solid #e2e8f0`; **focused/open:** `1.5px solid #0E7AB4`
- Border-radius: `8px`; background `#fafafa`; `min-height: 36px`; cursor `text`
- Transition: `border-color 0.15s`

**Search icon** — 14×14 magnifier, stroke `#9aa5b4`, absolutely positioned, `pointer-events: none`.

**Facet token (chip)** — class `.filter-chip`:
- `inline-flex`, `gap: 5px`, padding `3px 10px`, `border-radius: 100px`
- Background `#e8f6f9`, text `#1a8fa8`, border `1px solid #b3e4ef`, font-size `12px`, font-weight `500`
- Hover background `#d0eef5`
- Label format: muted group prefix (`opacity: 0.55, weight 400`) + value, e.g. **“Rolle: Lehrperson”**, **“Status: Gesperrt”**, **“Zuletzt online: Heute”**
- Trailing **×** (10px) removes that one facet. Clicking the chip body also removes it (`stopPropagation` so it doesn’t re-open via the container click).

**Text input**
- `flex: 1 1 130px`, `min-width: 110px`, borderless, transparent, font-size `13px`, color `#374151`
- Placeholder: `"Suchen oder filtern – Name, Rolle, Status …"` when no tokens; `"Weiter filtern oder suchen …"` when tokens exist.

**Clear-all (×) button** — appears when there’s any token OR typed text. Resets all facets AND clears the search text. Color `#9aa5b4`.

**Dropdown panel**
- Position: absolute, `top: calc(100% + 6px)`, `left:0 right:0` (matches field width), `z-index: 200`
- Background `#fff`, border `1px solid #e2e8f0`, radius `10px`, shadow `0 8px 24px rgba(0,0,0,0.10)`, padding `7px`, `max-height: 360px`, `overflow-y: auto`
- Entrance animation `filterPanelIn` (160ms ease-out, fade + translateY −6px → 0)
- **Free-text hint row** (only when text is typed): light row (`background #f0fbfd`, radius 7, padding `8px 10px`) reading *Volltextsuche nach „<text>“* with the term bolded in `#0E7AB4`.
- **Grouped facet list**, group order: **Rolle → Zuletzt online → Status**. Each group has an uppercase header (`font-size 11`, weight 600, color `#9aa5b4`, `letter-spacing 0.06em`, padding `7px 10px 4px`).
- **Facet option row**: `flex`, `gap: 10px`, padding `7px 10px`, radius `7px`. Highlighted row (keyboard or hover) background `#f1f5f9`. Leading 16×16 check-box square: border `1.5px solid #c2c7d0`; when active fill `#0E7AB4` + white check; label font-size `13`, color `#374151`.
- Only facets whose **label or group** matches the typed text are shown; if none match, show a muted note that the input is being used as full text.

### 2. Data table
- `width: 100%`, `border-collapse: collapse`, `table-layout: fixed`.
- **Column widths (colgroup):** `40px` (checkbox), `16%` Nachname, `13%` Vorname, `22%` Benutzername, `20%` E-Mail-Adresse, `10%` Rolle, `14%` Zuletzt Online, `44px` (kebab).
- **Header row:** `border-bottom: 2px solid #e8ecf0`, `position: sticky; top: 0; z-index: 5`, background `#fff`. Header cells: font-size `12`, weight `600`, color `#6b7280`, each with a sort glyph (`IconSort`, opacity 0.45). Headers are clickable affordances (sorting not implemented in the prototype).
- **Select-all checkbox** in the first header cell (checked when every filtered row is selected).
- **Body rows:** `border-bottom: 1px solid #f1f5f9`, cursor pointer, `transition: background 0.1s`.
  - Default `#fff`; **hover** `#f7fdfe`; **selected** `#edf8fb`.
  - Cells `padding: 10px 12px`, `vertical-align: middle`. Nachname `13.5px #111827` (weight 500 when selected); Vorname `13.5px #374151`; Benutzername `13px #374151` (ellipsis truncation); E-Mail `13px #6b7280` (ellipsis); Rolle `13px #374151`.
  - **Zuletzt-Online cell:** if `locked && days` → a red pill badge (`background #fef2f2`, text `#dc2626`, border `1px solid #fecaca`, radius 100, padding `2px 9px`, font `11.5px`/500) with a warning icon, reading *“vor N Tagen”*. Else if `lastOnline` → plain text `#6b7280`. Else empty.
  - **Kebab cell:** 16px three-dot button, color `#9aa5b4`, hover background `#eef2f6`. (In the full app this opens a per-row action menu; here it’s a stub. `stopPropagation` on click.)
- **Empty state:** centered “Keine Benutzer gefunden.” (`color #9aa5b4`, `14px`, padding `48px 28px`).

### 3. Checkbox (`Checkbox`)
- 36×36 hit area (`.cb-wrap`) wrapping a 16×16 box (`.cb-box`).
- Unchecked: border `1.5px solid #c2c7d0`, white fill. Checked/indeterminate: fill + border `#0E7AB4`, white check / dash.
- Hover ripple: a circular `#0E7AB4` overlay at `opacity 0.13`, scale 0.3 → 1.
- Supports `checked`, `indeterminate`, `onChange(nextBool)`. Stops click propagation.

## Interactions & Behavior

- **Type to search:** every keystroke updates `searchQuery` (live) AND re-filters the dropdown facet suggestions. The table updates immediately.
- **Pick a facet:** clicking a dropdown row (or pressing Enter on the highlighted one) toggles that facet, **clears the typed text**, and keeps focus in the input. The facet appears as a token.
- **Combined filtering:** the table shows rows matching the free text **AND** every active facet (AND semantics across groups; within “Rolle” multiple selections are OR’d as a set membership test).
- **Remove a facet:** click its token (or the token’s ×). `Backspace` on an empty input removes the **last** token.
- **Clear everything:** the field’s trailing × resets all facets and clears the text.
- **Keyboard:** `ArrowDown`/`ArrowUp` move the highlighted dropdown row (opens it if closed); `Enter` toggles the highlighted facet; `Backspace` (empty input) removes last token; `Escape` closes the dropdown and blurs.
- **Outside click** closes the dropdown (mousedown listener on document, scoped by a wrapper ref).
- **Row selection:** clicking a row’s checkbox toggles it; the header checkbox selects/clears all *currently filtered* rows. Footer shows “N Benutzer · M ausgewählt” with an “Auswahl aufheben” link.

## State Management
Local component state (lift to a store/URL params as appropriate in the real app):
- `searchQuery: string` — free-text query (owned by the table, written by the combobox via `setSearchQuery`).
- `filters: { roles: string[], lastOnline: string|null, locked: boolean, simplLogin: boolean }` — facet state.
- Combobox-internal: `open: boolean`, `query: string` (the field’s text, mirrored into `searchQuery`), `activeIdx: number` (keyboard highlight).
- Table-internal: `selected: Set<id>`, `hoveredRow: id|null`.

**Facet model (`buildFacets`)** flattens `filters` into a list of `{ fid, group, label, active, toggle() }`:
- `role` → multi-select (toggles membership in `roles[]`)
- `lastOnline` → single-select (sets to value or `null`)
- `locked`, `simplLogin` → boolean flags

**Last-online matching (`matchLastOnline` + `effDays`):** parses `lastOnline` (`'Heute'` → 0; `'vor N Tagen'` → N; otherwise falls back to `days`; else `null` = never). Buckets: `Heute` (=0), `Letzte 7 Tage` (≤7), `Letzte 30 Tage` (≤30), `Nie` (null).

> Note: the `simplLogin` (“Vereinfachtes Login”) facet has no matching field in the demo data, so selecting it yields no rows until that attribute exists on real user records. Wire it to the real field during implementation.

## Design Tokens

**Colors**
- Primary / accent: `#0E7AB4` (focus border, active facet fill, links)
- Accent text (chips/links): `#1a8fa8`
- Chip background `#e8f6f9`, chip border `#b3e4ef`, chip hover `#d0eef5`
- Hint row background `#f0fbfd`
- Row hover `#f7fdfe`, row selected `#edf8fb`
- Danger badge: text `#dc2626`, bg `#fef2f2`, border `#fecaca`
- Text: `#111827` (strong), `#374151` (body), `#6b7280` (muted header/email), `#9aa5b4` (placeholder/disabled)
- Borders: `#e2e8f0` (controls), `#e8ecf0` (header rule), `#f1f5f9` (row/divider)
- Surfaces: page `#f5f6f7`, control bg `#fafafa`, card/table `#fff`

**Typography** — Inter (400/500/600). Body 13px; names 13.5px; headers 12px/600; group labels 11px/600 uppercase `letter-spacing 0.06em`; badges 11.5px/500.

**Radius** — controls/dropdown `8–10px`; chips/pills/badges `100px`; checkbox `3–4px`; rows `7px`.

**Shadow** — dropdown `0 8px 24px rgba(0,0,0,0.10)`.

**Spacing** — toolbar `18px 28px 12px`; cells `10px 12px`; dropdown rows `7px 10px`; chips `3px 10px`.

**Motion** — `filterPanelIn` 160ms ease-out (fade + −6px translateY); border-color transitions 150ms; row background 100ms.

## Assets
No external image assets. All icons are inline SVG (search, ×, sort, warning-triangle, three-dot kebab). Font: Inter via Google Fonts (swap for the codebase’s standard font if different).

## Files
- `filter-table.html` — the isolated, self-contained prototype (open directly in a browser). Contains: `USERS` demo data, inline icons, `Checkbox`, `effDays`/`matchLastOnline`/`buildFacets`, `FilterCombobox`, and `UserTable` (toolbar + table + footer).

The component to port is **`FilterCombobox`** (plus the `buildFacets` / `matchLastOnline` helpers) and the **table** in `UserTable`. Replace the inline `USERS` array with the real data source and the `filtered` computation with a server query if the dataset is large.
