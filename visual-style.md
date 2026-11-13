# Style: Visual Layer

Beyond [dimensions and spacing](style.md), the `style` attribute can
attach a visual layer behind an element's content: a solid
**background**, **borders**, and **rounded corners**. The layer is
drawn inside the element's bounds — it never moves or grows the
content, and [padding](style.md) stays the inner spacing on top of it.

Colors are always written as bracketed hex values, Tailwind arbitrary
value style: `bg-[#rrggbb]`.

## Background

```xml
<goexml version="0.1.0">
  <window title="Card" size="320x200">
    <vbox id="card" style="bg-[#1e293b] p-2">
      <label id="title" text="Card" />
    </vbox>
  </window>
</goexml>
```

- `bg-[#rrggbb]` paints a solid background behind the whole element,
  **including its padding area** — padding remains the distance
  between the background edge and the content.
- At most one `bg-` token per element; a duplicate is rejected.

## Border

Borders are declared per side. Each side draws only when it has
**both a width and a color** — either alone is a decode error, so a
half-configured border can never silently render as nothing.

| Token | Meaning |
|---|---|
| `b-[#rrggbb]` | Border color, all four sides |
| `bt-` / `br-` / `bl-` / `bb-[#rrggbb]` | Border color, one side |
| `b-N` | Border width, all four sides — direct px, 0–16 |
| `bt-N` / `br-N` / `bl-N` / `bb-N` | Border width, one side |

Widths are **direct pixels**, not 4 px steps: `b-1` is a 1 px border.

```xml
<goexml version="0.1.0">
  <window title="Panel" size="320x200">
    <vbox id="panel" style="bg-[#0f172a] b-2 b-[#475569] rounded-md p-2">
      <label id="l" text="Rounded panel with a border" />
    </vbox>
  </window>
</goexml>
```

```xml
<goexml version="0.1.0">
  <window title="Accent" size="320x200">
    <vbox id="accent" style="b-1 bt-3 b-[#334155] bt-[#f59e0b] p-2">
      <label id="l" text="Uniform border, thick amber top" />
    </vbox>
  </window>
</goexml>
```

Rules:

- Side tokens follow the same specificity as
  [spacing](style.md): side beats all-side, regardless of order.
- Every side carrying a width token must carry a color token and the
  reverse — `b-2` alone, `b-[#ff0000]` alone, and mixed pairs like
  `b-2 bt-[#00ff00]` (three sides left colorless) are all rejected.
- A colored side with zero width is rejected for the same reason.
- Border widths are drawn **inside** the element's bounds: they do not
  change the element's size or its neighbors' positions.

## Rounded corners

| Token | Meaning |
|---|---|
| `rounded` | 4 px radius, all corners |
| `rounded-md` | 8 px, all corners |
| `rounded-xl` | 16 px, all corners |
| `rounded-N` | N px, all corners (1–999) |
| `rounded-t-*` / `rounded-b-*` / `rounded-l-*` / `rounded-r-*` | Two corners of one side |
| `rounded-tl-*` / `rounded-tr-*` / `rounded-bl-*` / `rounded-br-*` | One corner |

The `*` is any size form: `rounded-t-md`, `rounded-bl-12`.

```xml
<goexml version="0.1.0">
  <window title="Rounded" size="320x200">
    <vbox id="card" style="bg-[#1e293b] rounded-xl p-2">
      <label id="title" text="Card" />
    </vbox>
  </window>
</goexml>
```

Rules:

- Named sizes stay on the schema's 4 px grid (4/8/16); free pixel
  values cover everything else.
- Side and corner forms require an explicit size — there is no bare
  `rounded-t`, because a token that changes nothing must not parse.
- Specificity: corner beats side beats all-corners, regardless of
  order.
- A radius larger than half the element is **clamped at render** to
  half the smaller dimension, so an oversized radius can never produce
  an undefined shape.
- `rounded` with nothing to round (no background, no border) is
  rejected — same no-dead-token discipline.

## Rounded + border interaction

A rounded border is drawn as a single stroked rounded rectangle, which
can only be uniform. Therefore: **rounded corners with a border
require a uniform border** — `b-…` tokens only, no per-side variation.
Violations are decode errors:

```text
goexml: line 4: <vbox id="card"> invalid style: rounded requires a uniform border width: use b-N without per-side variation
```

A rounded background with no border is fine, and a per-side border
without rounding is fine — only the combination is constrained.
