# Element Reference

A compact, one-place reference of every element: its own attributes,
the [common attributes](document.md) it accepts, and its legal
children. The behavioral details live in the topic documents — this
page is the checklist.

Common attributes (`id`, `visible`, `style`) are marked **common**.
`id` is **optional** everywhere, unique across the file when present.
It stays required on the elements the runtime must reference by name:
`container`, `vbox-expand`, an `image` without `src`, and any element
whose text carries a `{{placeholder}}` template.

## Envelopes

| Element | Own attributes | Children |
|---|---|---|
| `goexml` | `version` (required, `0.1.0`) | exactly one `window` |
| `goexml-item` | `version` (required, `0.1.0`) | exactly one widget |

## Structure

| Element | Own attributes | Children |
|---|---|---|
| `window` | `title` (req), `size` (req, `WxH`) | exactly one widget |
| `vbox` | `spacing`, `align`, `justify`, `column`, `min`, `max` + **common** | ≥ 1 widget; see [layout.md](layout.md) for grid rules |
| `vbox-expand` | `spacing`, `align`, `justify`, `column`, `min`, `max`, `scrollable` (bare, or `="true"`) + **common** | no children; items appended at runtime from [fragments](fragments.md), each with a private id space |
| `hbox` | `spacing`, `align`, `justify` + **common** | ≥ 1 widget |
| `border` | — + **common** | ≥ 1 region (`top`/`bottom`/`left`/`right`/`center`), each at most once |
| `top` / `bottom` / `left` / `right` / `center` | — (no attributes) | ≤ 1 widget |
| `container` | — + **common** (`id` required) | exactly 1 widget whose subtree holds exactly 1 `slot`; no nested `container` |
| `slot` | `style` (dimension utilities only) | none; only legal inside `container` |

## Widgets

| Element | Own attributes | Notes |
|---|---|---|
| `label` | `text` (req) + **common** | no children; placeholders allowed |
| `text` | `v`, `wrap` + **common** | content from `v` **or** inner text, not both; no children |
| `button` | `text` (req), `onClick` (req), `disabled` + **common** | no children |
| `entry` | `text`, `placeholder`, `disabled` + **common** | no children |
| `select` | `onChange` (req), `disabled` + **common** | ≥ 1 `option`; nothing else |
| `option` | `value` (req), `text` (req), `selected` (empty) | inside `select` only; unique `text`; ≤ 1 `selected` |
| `table` | `onSelect` + **common** | ≥ 1 `row`; rectangular; `h-*` rejected |
| `row` | — | only `cell` children; inside `table` only |
| `cell` | `text` | content from `text` **or** inner text, not both; no children |
| `rect` | `color` (req, `#rrggbb`) + **common** | no children |
| `image` | `src` (req, resource key), `fit` (`contain`/`cover`/`fill`) + **common** | no children |

## Value domains

| Attribute | Accepted values |
|---|---|
| `version` | `0.1.0` |
| `id` | `[A-Za-z0-9_]+`, unique per file |
| `visible` | `true` / `false` |
| `disabled` | `true` / `false` (only where listed) |
| `align` | `start` / `center` / `end` |
| `justify` | `start` / `center` / `end` / `space-between` / `space-around` / `space-evenly` |
| `wrap` | `word` / `break` / `off` |
| `column` | `grid` (vbox only) |
| `min`, `max` | integers ≥ 1, only with `column="grid"` |
| `spacing` | non-negative integer (px); inner padding moved to the style `p-N` utilities |
| `size` | `WxH`, e.g. `800x600` |
| `color` | `#rrggbb` |
| `src` | optional resource key: `[A-Za-z0-9._/-]+`; absent = runtime-filled slot |
| `fit` | `contain` (default) / `cover` / `fill` |
| `selected` | bare, or `="true"` |
| `style` | see [style.md](style.md) and [visual-style.md](visual-style.md) |

Flag attributes — the ones whose presence alone means true
(`scrollable`, `selected`) — read best bare: `<vbox-expand
scrollable />`. The decoder rewrites a bare flag to `="true"` before
parsing, so both spellings mean the same thing and any other value is
rejected.

## Global rules that apply everywhere

- Unknown elements, unknown attributes, and unknown style utilities
  are rejected — named one by one in the error, with line and id.
- Inner text is rejected everywhere except `text` and `cell`, and
  there only as the exclusive alternative to the attribute form.
- `{{token}}` placeholders validate at decode wherever they are
  allowed; see [dynamic-values.md](dynamic-values.md).
- Nothing half-valid is ever built: a document or fragment either
  passes the whole check or produces a single explicit error.
