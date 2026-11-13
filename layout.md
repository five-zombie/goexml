# Layout Elements

Structure is built from three explicit containers: the flow boxes
`vbox` and `hbox`, and the region-based `border`. There is no implicit
layout anywhere — a group of children always sits inside one of these.
The [style.md](style.md) dimension utilities (`w-full`, `h-N`,
padding, margin) apply on top of any of these.

All three elements **require an `id`** and accept the common
attributes (`id`, `visible`, `style`).

## `vbox` — vertical box

Lays children out top to bottom in document order.

| Attribute | Value | Default |
|---|---|---|
| `spacing` | Non-negative integer, px gap between children | `0` |
| `align` | `start` / `center` / `end` — cross-axis (horizontal) alignment of each child | `start` |
| `justify` | `start` / `center` / `end` / `space-between` / `space-around` / `space-evenly` — main-axis (vertical) distribution | `start` |
| `column` | `grid` — turns the box into a column-flow grid, see below | off |
| `min` / `max` | Integers ≥ 1 — grid column count bounds, only with `column="grid"` | — |

```xml
<goexml version="0.1.0">
  <window title="Vertical" size="320x240">
    <vbox id="root" style="p-2" spacing="4" align="start">
      <label id="a" text="First" />
      <label id="b" text="Second" />
      <label id="c" text="Third" />
    </vbox>
  </window>
</goexml>
```

Rules:

- A `vbox` or `hbox` must have at least one child; an empty box is
  rejected.
- `align` and `justify` validate strictly; anything outside the value
  lists above is an error.
- `spacing` validates as a non-negative integer; there is no unit
  suffix, values are px. Inner padding is not an attribute: the style
  `p-N` utilities add it on any element (an older `padding` attribute
  on boxes was removed — use `style="p-2"` for 8 px).

## `hbox` — horizontal box

Same attributes as `vbox` except the axes swap: the main axis is
horizontal (`justify` distributes left to right) and the cross axis is
vertical (`align` positions each child vertically). `hbox` has no
`column` attribute.

```xml
<goexml version="0.1.0">
  <window title="Actions" size="320x120">
    <hbox id="actions" spacing="8" justify="end" align="center">
      <button id="cancel" text="Cancel" onClick="OnCancel" />
      <button id="save" text="Save" onClick="OnSave" />
    </hbox>
  </window>
</goexml>
```

## Grid columns on a `vbox`

`column="grid"` reinterprets the vbox's children as cells flowing into
columns; after the last column the flow wraps to the next row. The
column count adapts to the available width between `min` and `max`
(defaults: `min` 1, unbounded `max`).

```xml
<goexml version="0.1.0">
  <window title="Grid" size="480x240">
    <vbox id="tiles" column="grid" min="2" max="4" spacing="4">
      <label id="c1" text="One" />
      <label id="c2" text="Two" />
      <label id="c3" text="Three" />
      <label id="c4" text="Four" />
      <label id="c5" text="Five" />
    </vbox>
  </window>
</goexml>
```

Rules:

- `column` must be exactly `grid` if present; other values are
  rejected.
- `min` and `max` are only legal together with `column="grid"`, must
  be integers ≥ 1, and `min` must not exceed `max`.
- In a grid, the cell height is the row height: an `h-N` utility on a
  cell only raises the row minimum. For the same reason the `h-*`
  utilities are rejected on [tables](widgets.md).

## `vbox-expand` — an append-only stack

A vertical stack whose items are **never declared in the document**:
the XML reserves the place, and the runtime appends items into it —
typically a feed, a log, or a result list.

```xml
<goexml version="0.1.0">
  <window title="Feed" size="400x600">
    <vbox-expand id="feed" style="w-full" scrollable />
  </window>
</goexml>
```

| Attribute | Value | Default |
|---|---|---|
| `spacing` | Non-negative integer, px gap between items | `0` |
| `align` | `start` / `center` / `end` — cross-axis alignment of each item | `start` |
| `justify` | `start` / `center` / `end` / `space-between` / `space-around` / `space-evenly` — main-axis distribution | `start` |
| `column` | `grid` — flows items into a column-flow grid | off |
| `min` / `max` | Integers ≥ 1 — grid column count bounds, only with `column="grid"` | — |
| `scrollable` | bare, or `="true"` | absent |

Rules:

- Takes no children. Items come from
  [fragments](fragments.md) appended at runtime; a document that
  declares children is rejected.
- The layout attributes mirror a static `vbox`: `spacing` sets the
  gap between items, `align` and `justify` position them, and
  `column="grid"` with `min`/`max` flows them into columns. A style
  utility on an item root (`h-N`, `w-full`) sizes that item inside
  the stack, exactly like a child of a `vbox`.
- Each appended item is a fragment build of its own, with a **private
  id space**: the same fragment can stack many times — `id="title"`
  inside ten items means ten titles, reachable per item through the
  runtime's list handle, never colliding in the document's global id
  space.
- The runtime supplies per-item data: fragment slots get their
  content and image resources per append, and the runtime can fill or
  swap widget values on a single item later.
- The stack sizes to its items by default. The
  [dimension utilities](style.md) size it like any element — with
  `scrollable`, `h-*` bounds the visible height so the stack scrolls
  inside it; without `scrollable`, the stack simply grows.
- The [visual layer](visual-style.md) applies like on every element:
  `bg-*`, `b-*` and `rounded` draw behind the stack.
- The runtime can empty the stack; handles of removed items are
  retired — writing into them afterwards is an explicit error, never
  a silent write into a detached widget.

## `border` — five-region layout

`border` divides the area into named regions: `top`, `bottom`, `left`,
`right`, and `center`. Each region takes **at most one child** (wrap a
group in a box), appears at most once, and the regions that are absent
simply do not exist — `center` alone is a valid, common case.

```xml
<goexml version="0.1.0">
  <window title="Shell" size="640x480">
    <border id="shell">
      <top>
        <label id="header" text="Header bar" />
      </top>
      <left>
        <vbox id="nav" spacing="4">
          <button id="navHome" text="Home" onClick="OnHome" />
          <button id="navSettings" text="Settings" onClick="OnSettings" />
        </vbox>
      </left>
      <center>
        <label id="content" text="Main area" />
      </center>
      <bottom>
        <label id="status" text="Ready" />
      </bottom>
    </border>
  </window>
</goexml>
```

Rules:

- The region elements take no attributes at all — they are pure
  placeholders.
- Only `top`, `bottom`, `left`, `right`, `center` are legal children;
  anything else is rejected as "is not a border region".
- `border` requires at least one region.
- `container` is legal inside a region; see
  [containers-and-slots.md](containers-and-slots.md).
