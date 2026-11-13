# Document Structure

The envelope rules that every goexml file follows: the root element,
the window, the explicit-layout rule, and how the parser reports
errors. Element-specific details live in their own documents:
[layout.md](layout.md), [widgets.md](widgets.md), [style.md](style.md).

## The `<goexml>` root

A document is a single `<goexml>` element with a `version` attribute.
The only supported version is `0.1.0`:

```xml
<goexml version="0.1.0">
  <window title="Hello">
    <label id="greeting" text="Hello" />
  </window>
</goexml>
```

- The root takes no other attributes and no inner text.
- A document holds **exactly one** `<window>`.
- A UTF-8 BOM is accepted and stripped, so files saved by Windows
  editors do not shift error line numbers.

## The `<window>` element

`<window>` describes the top-level surface:

| Attribute | Value | Required |
|---|---|---|
| `title` | Window title text | yes |

The window's size is **set from Go**: call `Resize` on the
`fyne.Window` after `Build`; without it the toolkit default applies.
A `size` attribute in the XML fails decode.

The window holds **exactly one child**. This is the explicit-layout
rule: two siblings would stack in an implicit, unconfigured box, so
the author wraps them in a `<vbox>` and configures the box instead.
The same rule applies inside [fragments](fragments.md).

```xml
<goexml version="0.1.0">
  <window title="Preferences">
    <vbox id="root" style="p-2" spacing="8">
      <label id="info" text="General settings" />
      <entry id="name" placeholder="Display name" />
    </vbox>
  </window>
</goexml>
```

## Common attributes

Every widget-level element accepts three common attributes, listed in
[reference.md](reference.md):

| Attribute | Value | Notes |
|---|---|---|
| `id` | `[A-Za-z0-9_]+` | **Optional**, unique across the whole file when present. Give it to elements the runtime addresses; skip it on possibly-static elements. Still **required** on `container` (slots are filled by id), `vbox-expand` (the runtime handle comes from it), an `image` without `src` (a slot image must be addressable), and any element whose text carries a `{{placeholder}}` template (templates render through the runtime's setters, which address by id). |
| `visible` | `true` / `false` | Defaults to `true`. `visible="false"` hides the element; a hidden box collapses in its parent's layout. |
| `style` | Space-separated utilities | See [style.md](style.md) and [visual-style.md](visual-style.md). |

Some widgets accept `disabled` (`button`, `entry`, `select`); the
schema lists it per tag instead of promising it everywhere, so a
`disabled` on a `label` is rejected.

## Inner text

Inner text is not allowed on elements that do not define a meaning for
it. Only two elements take it — `<text>` (as an alternative to its `v`
attribute) and `<cell>` (as an alternative to its `text` attribute).
Everything else must express content through attributes; inner text on
any other element is rejected rather than ignored.

## Error format

Validation happens before any widget is built. Errors are explicit and
line-precise:

```text
goexml: line 12: <vbox id="main"> has no attribute "algn"
goexml: line 7:  <label id="ok"> needs a text attribute
goexml: line 3:  <window> needs exactly one child; wrap multiple children in a <vbox>
```

Every error carries the line, the tag, and the id when one is present.
Unknown attributes are named one by one, sorted, so the message is
deterministic. There is no "best effort" mode: an invalid document
produces no window.
