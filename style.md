# Style: Dimensions and Spacing

The `style` attribute is a space-separated list of utilities, in the
spirit of Tailwind. It works on every widget-level element and on
`slot`. This document covers **dimension and spacing** utilities; the
visual layer (background, border, rounded) has its own page,
[visual-style.md](visual-style.md).

The parser is strict: an unknown utility is an error, not a silent
no-op, so a typo can never render as a surprise.

```xml
<goexml version="0.1.0">
  <window title="Style" size="320x200">
    <vbox id="root" spacing="4">
      <entry id="name" placeholder="Name" style="w-full" />
      <vbox id="card" style="p-2 pt-4 m-1">
        <label id="l" text="Padded card" />
      </vbox>
    </vbox>
  </window>
</goexml>
```

## Width and height

| Utility | Meaning |
|---|---|
| `w-full` | Fill the parent's width |
| `h-full` | Fill the parent's height |
| `h-N` | Exact content height of N*4 px (`h-2` = 8 px) whatever the widget contains |

Rules:

- Both fills resolve against the parent's **content area** — the
  parent's padding is never overlapped. `style="w-full h-full"` fills
  both axes.
- `h-0` is a real zero height: an invisible element that still takes
  part in layout through its margin — a spacer.
- A tag carries **at most one** `w-*` and one `h-*` class. Duplicates
  and conflicts (`h-2 h-full`) are rejected with a "more than one"
  error.
- In a [grid](layout.md), the cell height is the row height: `h-N`
  only raises the row minimum. `h-*` is rejected on
  [tables](widgets.md) entirely — a table sizes to its content.

## Padding — inside the element

`p-N` pads all sides; axis and side forms override. Values run 0–4
steps, each step 4 px.

| Utility | Sides |
|---|---|
| `p-N` | All |
| `px-N` / `py-N` | Left+right / top+bottom |
| `pt-N` / `pr-N` / `pb-N` / `pl-N` | Top / right / bottom / left |

```xml
<goexml version="0.1.0">
  <window title="Pad" size="320x200">
    <vbox id="panel" style="p-2 pt-4">
      <!-- 16 px top, 8 px on the other three sides -->
      <label id="l" text="Content" />
    </vbox>
  </window>
</goexml>
```

## Margin — outside the element

Same shape as padding, `m` prefix. Margin is the space between the
element's border and its siblings.

| Utility | Sides |
|---|---|
| `m-N` | All |
| `mx-N` / `my-N` | Left+right / top+bottom |
| `mt-N` / `mr-N` / `mb-N` / `ml-N` | Top / right / bottom / left |

## Specificity

Side utilities beat axis utilities beat all-side utilities,
**regardless of order**: `p-2 pt-4` and `pt-4 p-2` both give a 16 px
top and 8 px elsewhere. Within the same specificity, the later token
wins. N runs 0–4 for both families; out-of-range or non-numeric values
are rejected with the token named:

```text
goexml: line 9: <vbox id="panel"> invalid style: bad padding "p-9", want p-0 to p-4
```

A `style` attribute can mix everything freely — dimensions, spacing,
and the [visual utilities](visual-style.md) — because each utility
family owns disjoint prefixes.
