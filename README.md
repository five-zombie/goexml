# goexml

The **goexml** XML standard — declarative UI markup, version **0.1.0** —
described as reviewable files.

This repository holds the standard's documentation. It is named
**goexml**: the XML schema and its rules, independent of any
implementation.

The standard is **platform-neutral**: any runtime — desktop, browser,
mobile — can implement it. These documents cover the **XML side only**:
elements, attributes, style utilities, and the validation rules the
parser enforces. Handler implementations, the build API, and runtime
value setters belong to each implementation and are documented by it;
where a XML feature connects to the runtime, these pages say so in one
sentence.

## How the docs are split

Each file owns exactly one topic:

| Document | Responsibility |
|---|---|
| [document.md](document.md) | The document envelope: `<goexml>`, `<window>`, global rules, error format |
| [layout.md](layout.md) | Structure elements: `vbox`, `hbox`, `border` and their layout attributes |
| [widgets.md](widgets.md) | Leaf widgets: `label`, `text`, `button`, `entry`, `select`, `table`, `rect` |
| [style.md](style.md) | The `style` attribute: dimension and spacing utilities |
| [visual-style.md](visual-style.md) | The `style` attribute: background, border, rounded corners |
| [containers-and-slots.md](containers-and-slots.md) | Injection points: `container` and `slot` |
| [fragments.md](fragments.md) | Injectable fragments: `goexml-item` |
| [dynamic-values.md](dynamic-values.md) | `{{token}}` placeholders in text attributes |
| [reference.md](reference.md) | One-table-per-element attribute and children reference |

## The standard in one paragraph

A document is one `<goexml version="0.1.0">` envelope holding exactly
one `<window>`. The window holds exactly one child — layout comes from
explicit boxes (`vbox`, `hbox`, `border`), never from implicit
stacking. Widgets carry unique `id`s; handlers are referenced by name
in XML attributes and implemented by the host application; values that
change at runtime are set by the runtime, never computed in XML. The
parser rejects unknown elements, unknown attributes, invalid values,
and any construct that would render nothing, with a line-precise
error.

```xml
<goexml version="0.1.0">
  <window title="Hello" size="400x300">
    <vbox id="root" style="p-2" spacing="4">
      <label id="greeting" text="Hello, goexml" />
      <button id="ok" text="OK" onClick="OnOk" />
    </vbox>
  </window>
</goexml>
```

## Conventions used in these documents

- Fenced `xml` examples are always **complete envelopes**: a full
  `<goexml>` document or a full `<goexml-item>` fragment, exactly as a
  file on disk would look.
- Inline code spans are fragments of markup, not complete files.
- "Rejected" means the parser returns an error before anything is
  built; nothing half-valid is ever rendered.
