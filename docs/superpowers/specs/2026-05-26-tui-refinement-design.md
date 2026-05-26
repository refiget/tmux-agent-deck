# TUI Refinement Design

**Date:** 2026-05-26  
**Scope:** Visual polish of existing sidebar UI — no layout changes  
**Approach:** Animation foundation layer first, then visual components

---

## Goal

Refine the existing sidebar TUI to be more polished and expressive. The three-panel layout (agents panel / pet divider / bottom panel) stays unchanged. All improvements are within individual rendering components.

---

## Phase 1: Animation Foundation Layer

### 1.1 New Frame Arrays — `src/lib.rs`

Add alongside the existing `SPINNER_PULSE`:

```rust
pub const RUNNING_GLYPHS: [&str; 10] = ["⠋","⠙","⠹","⠸","⠼","⠴","⠦","⠧","⠇","⠏"];
pub const WAITING_GLYPHS: [&str; 4]  = ["◐","◓","◑","◒"];
pub const WAITING_PULSE:  [u8; 4]    = [178, 214, 172, 208]; // amber family
pub const BG_PULSE:       [u8; 2]    = [25, 33];             // deep blue / bright blue
pub const ATTN_PULSE:     [u8; 2]    = [208, 214];           // orange / amber
```

Frame cadence: driven by the existing 200ms `spinner_frame` tick.  
Waiting uses `frame / 2` (400ms per step) to convey "stuck" vs "active".

### 1.2 Replace `running_icon_for` — `src/ui/panes/row/status.rs`

Remove `running_icon_for()`. Replace with:

```rust
pub(super) fn animated_icon(
    status: &PaneStatus,
    attention: bool,
    frame: usize,
) -> (&'static str, Color) {
    if attention {
        return ("◉", Color::Indexed(ATTN_PULSE[frame % 2]));
    }
    match status {
        PaneStatus::Running    => (RUNNING_GLYPHS[frame % 10],        Color::Indexed(SPINNER_PULSE[frame % 8])),
        PaneStatus::Background => ("⊙",                               Color::Indexed(BG_PULSE[frame % 2])),
        PaneStatus::Waiting    => (WAITING_GLYPHS[(frame / 2) % 4],   Color::Indexed(WAITING_PULSE[(frame / 2) % 4])),
        PaneStatus::Idle       => ("○",                               Color::Indexed(236)),
        PaneStatus::Error      => ("⊗",                               Color::Indexed(203)),
        PaneStatus::Unknown    => ("·",                               Color::Indexed(235)),
    }
}
```

**Attention** overrides the status icon entirely — `◉` with fast amber pulse signals any pane that needs user action, regardless of its `PaneStatus`.

**Frame cadence detail:** All callers pass the raw `spinner_frame` counter. `animated_icon` handles per-status speed internally: Running at full 200ms pace (`frame % 10`), Waiting at half pace (`(frame / 2) % 4` → 400ms per step) to convey "stuck". Callers never divide the frame themselves.

**Filter bar** keeps static icons (via `icons.status_icon()`) — only the agent list rows use `animated_icon`. A pulsing filter bar would be distracting.

### 1.3 Update Default Icons — `src/ui/icons.rs`

```rust
// Before                After
error:      "✕"    →    "⊗"
background: "◎"    →    "⊙"
```

These match the glyphs used in `animated_icon`. The tmux override mechanism (`@sidebar_icon_*`) continues to work unchanged.

---

## Phase 2: Visual Components

### 2.1 Filter Bar — `src/ui/panes/filter_bar.rs`

**`render_filter_bar()`:**

- Separator between items: `"  "` → `" │ "` (col.238, dim)
- Selected item: wrap icon+count with `"["` / `"]"` (accent color)

```
Before:  ≡2  ●1  ◎0  ◐0  ○1  ✕0
After:  [≡2] │ ⠋1 │ ⊙0 │ ◐0 │ ○1 │ ⊗0
```

No structural changes to the function's return type or callers.

### 2.2 Secondary Header — `src/ui/panes/filter_bar.rs` + `src/ui/panes.rs`

**`render_secondary_header()`:**

- Repo button: `"label ▾"` → `"‹ label ›"` (`‹`/`›` in accent color, label in text_muted)

**`PaneLayout` in `src/ui/panes.rs`:**

Add two separator rows between the header rows and the list:

```
row 0: filter bar
row 1: dim ─────── line  (new)
row 2: secondary header
row 3: dim ─────── line  (new)
row 4+: agent list
```

`PaneLayout::compute()` changes `list_area.y` from `area.y + 2` to `area.y + 4`, and `list_area.height` shrinks by 2. The separator lines are rendered in `draw_agents()` as `Paragraph` with `"─".repeat(width)` in `theme.border_inactive` color.

### 2.3 Repo Group Label — `src/ui/panes/row_collector.rs`

Current: plain colored text `"tmux"`

New format: `"── TMUX ────────────────── +"`

- Left: `"── "` (dim)
- Name: uppercase, accent color, `display_width`-aware
- Fill: `"─"` repeating to pad width minus name and button
- Right: `" +"` (accent color) — existing click target column stays the same

### 2.4 Running Row — `src/ui/panes/row/status.rs` + `row/`

**Status row (line 1):**

- Leftmost character: `▌` (accent color when tmux-focused or sidebar-selected; same background color as row when neither — effectively invisible)
- Icon: `animated_icon()` replaces static icon
- Right side: elapsed time + `▐` (col.238, scrollbar hint)

**Progress row (line 2, existing `task_progress_row`):**

- Unfilled segment: empty → `▒` (col.238), making remaining capacity visible
- Append `" n/total"` count (dim color) after the bar
- Append most recent tool call name, right-aligned, truncated to fit (dim color)

**Prompt rows:**

- Color: `text_active` → `text_muted`
- Fold behavior: allow at most 1 line (truncate with `…`); remove multi-line wrapping

### 2.5 Waiting / Attention Row — `src/ui/panes/row/status.rs`

- Icon: `animated_icon()` → `◐◓◑◒` slow rotation in amber
- `wait_reason` row: prepend `"◈ "`, apply background `Color::Indexed(52)` (dark amber) to the row
- When `attention == true`: append `" perm ⚠"` right-aligned on the status line (amber color)

### 2.6 Response Preview Row — `src/ui/panes/row/` (prompt rendering)

The `▷` response prefix currently uses bold cyan. Change to:

- Prefix: `"← "` (col.238 dark green, not bold)
- Row background: `Color::Indexed(22)` (dark green)
- Text color: `Color::Indexed(238)` (near-black on dark green)

This gives response lines a distinct green tint, immediately distinguishing them from prompt lines (which have no background).

---

## Affected Files

| File | Change |
|------|--------|
| `src/lib.rs` | Add 5 frame array constants |
| `src/ui/icons.rs` | Update `Default`: `✕→⊗`, `◎→⊙` |
| `src/ui/panes/row/status.rs` | `running_icon_for` → `animated_icon`; `▌` left indicator; `▐` right; waiting attention label |
| `src/ui/panes/filter_bar.rs` | `[x]` selected brackets; `│` separators; `‹ ›` repo button |
| `src/ui/panes/row_collector.rs` | Repo group label → `──NAME──` separator style |
| `src/ui/panes/row.rs` / `row/` | Progress bar `▒` fill + count; prompt single-line; response `←` + bg |
| `src/ui/panes.rs` | `PaneLayout` +2 header rows; render separator lines in `draw_agents` |

## Snapshot Tests

All `insta::assert_snapshot!` tests covering the above rendering paths will produce diffs. After implementation, run:

```
cargo insta review
```

Accept snapshots that match the new visual output. Do not accept any snapshot that shows unintended content outside the changed component.

---

## Out of Scope

- Layout structure (3-panel split unchanged)
- Bottom panel content (git/activity rendering)
- Pet animation
- Popup rendering (spawn modal, remove confirm, repo dropdown)
- Worktree, hook, or CLI logic
