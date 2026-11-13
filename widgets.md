# Widgets

The leaf elements that render content: `label`, `text`, `button`,
`entry`, `select`, `table`, `rect`, and `image`. Structure elements
live in [layout.md](layout.md); injection points in
[containers-and-slots.md](containers-and-slots.md).

All widgets **require an `id`** and accept the common attributes
(`id`, `visible`, `style`). Widgets that can be disabled list it
explicitly: `button`, `entry`, `select`.

## `label`

Static text, always from the `text` attribute.

```xml
<goexml version="0.1.0">
  <window title="Label">
    <label id="title" text="Settings" />
  </window>
</goexml>
```

- `text` is required.
- May carry [`{{token}}` placeholders](dynamic-values.md).
- Takes no children.

## `text`

A wrapping paragraph of running text. Its content comes from either
the `v` attribute or the element's inner text — exactly one of the
two, never both:

```xml
<goexml version="0.1.0">
  <window title="Text">
    <text id="intro" wrap="word">Long running text that wraps at the
window width and flows over multiple lines.</text>
  </window>
</goexml>
```

| Attribute | Value | Default |
|---|---|---|
| `v` | The text content (alternative to inner text) | — |
| `wrap` | `word` / `break` / `off` | `word` |

- May carry [`{{token}}` placeholders](dynamic-values.md) in `v` or
  inner text.
- Takes no children.

## `button`

```xml
<goexml version="0.1.0">
  <window title="Button">
    <button id="save" text="Save" onClick="OnSave" />
  </window>
</goexml>
```

| Attribute | Value | Required |
|---|---|---|
| `text` | Button label | yes |
| `onClick` | Handler name — a name the host application registers | yes |
| `disabled` | `true` / `false` | no |

- The `onClick` value is a **reference**: the parser checks the name
  exists in the handler map at build time; the implementation lives in
  the host application.
- The label may carry [`{{token}}` placeholders](dynamic-values.md).
- Takes no children.

## `entry`

A single-line text input.

```xml
<goexml version="0.1.0">
  <window title="Entry">
    <entry id="name" text="" placeholder="Display name" style="w-full" />
  </window>
</goexml>
```

| Attribute | Value | Notes |
|---|---|---|
| `text` | Initial content | optional; may carry placeholders |
| `placeholder` | Hint shown while empty | optional |
| `disabled` | `true` / `false` | optional |

- Takes no children.
- At runtime the entry reflects what the user typed; the runtime reads
  and writes it by id.

## `select` and `option`

A dropdown. Options are declared in XML; selection is addressed by
value.

```xml
<goexml version="0.1.0">
  <window title="City">
    <select id="city" onChange="OnCity">
      <option value="jkt" text="Jakarta" />
      <option value="bdg" text="Bandung" selected />
      <option value="sby" text="Surabaya" />
    </select>
  </window>
</goexml>
```

`<select>` attributes: `onChange` (required, handler name), `disabled`,
plus the common attributes.

Each `<option>` is required to carry:

| Attribute | Value |
|---|---|
| `value` | The submitted value — non-empty, never rendered |
| `text` | The displayed label — may carry placeholders |
| `selected` | Bare, or `="true"` — both mean the initial choice; any other value is rejected |

Rules:

- At least one `<option>`; nothing else may appear inside a `select`.
- Option `text` values must be unique within the select — the text is
  what the widget displays, so duplicates would be ambiguous.
- At most one `selected` option. The marker is the bare
  `selected` attribute; `selected="true"` spells the same thing —
  the decoder rewrites bare flags to `="true"` — and any other
  value is rejected.
- With no `selected`, nothing is chosen initially.

## `table`, `row`, `cell`

A content-sized table. Rows are declared in XML as `row` elements of
`cell` elements; a cell's content is its `text` attribute or its inner
text, exactly one of the two.

```xml
<goexml version="0.1.0">
  <window title="Prices">
    <table id="prices" onSelect="OnPick">
      <row>
        <cell text="Coffee" />
        <cell>2.50</cell>
      </row>
      <row>
        <cell text="Tea" />
        <cell>1.80</cell>
      </row>
    </table>
  </window>
</goexml>
```

`<table>` attributes: `onSelect` (handler name, optional), plus the
common attributes.

Rules:

- At least one `row`; rows hold only `cell` elements; cells take no
  children.
- Rows are rectangular: every row must have the same cell count as the
  first row. Ragged tables would break during rendering, so they are
  rejected at decode.
- The `h-*` style utilities are **rejected** on a table: a table always
  sizes to its content, so an exact or filling height would overflow
  its slot and overlap siblings. Width utilities are fine.

## `rect`

A solid color block, a spacer with a visible body:

```xml
<goexml version="0.1.0">
  <window title="Rect">
    <rect id="divider" color="#2e2e2e" style="w-full h-0" />
  </window>
</goexml>
```

- `color` is required and must look like `#rrggbb` (hex digits).
- Takes no children.
- Combine with the [dimension utilities](style.md) to draw separators,
  panels, and spacers; combine with the
  [visual style](visual-style.md) `rounded` to round its corners.

## `image`

A picture from the runtime's **registered resource set**. XML names a
resource key; the key may look like a path (`assets/logo.png`) but it
is still a key into the set — it never carries a URL or a `{{token}}`,
and nothing is ever read from disk or network. Where the bytes live
(embedded bundle, asset pack, network loader) is a runtime decision,
which keeps documents portable across platforms.

```xml
<goexml version="0.1.0">
  <window title="Photo">
    <image id="photo" src="assets/logo.png" fit="contain" style="w-full" />
  </window>
</goexml>
```

| Attribute | Value | Default |
|---|---|---|
| `src` | Resource key: letters, digits, dot, dash, slash, underscore; **optional** — omit for a runtime-filled slot | — |
| `fit` | `contain` / `cover` / `fill` | `contain` |

Rules:

- With `src`, it must name a resource the runtime registered; an
  unknown key, or bytes that are not a decodable image, fail the
  build — an image can never silently render as an empty box.
- Without `src`, the image is a **runtime-filled slot**, like a
  container slot: the XML reserves the place, the runtime supplies
  the pixels later. A slot renders empty and invisible until filled;
  the runtime fills it by id and can audit images still unfilled
  before showing, so no slot is forgotten.
- Without size style the image is as large as its pixels. The
  [dimension utilities](style.md) size it like any leaf — `h-*` is
  **allowed** on an image (unlike a [table](widgets.md): pictures
  scale, tables do not). `h-0` still means an invisible spacer.
- `fit` decides how the picture fills its box: `contain` keeps the
  aspect ratio inside it, `cover` fills it and crops the overflow,
  `fill` stretches without keeping the aspect.
- The [visual layer](visual-style.md) draws **behind** the pixels and
  follows the same rules as everywhere else — but rounded corners do
  not clip the picture itself.
- The runtime can swap the resource by id at runtime (see the
  implementation's image setter); an unknown key is rejected there
  too.
