# Dynamic Values

Static text lives in XML attributes. Values that change at runtime are
set from Go by id — **there is no logic in XML**: no expressions, no
`if`, no loops. What XML does have is the `{{token}}` placeholder: a
hole in a text attribute that Go fills with values.

## Where placeholders may appear

`{{token}}` may appear in the text-bearing attributes of:

- `label` — `text`
- `button` — `text`
- `entry` — `text`
- `text` — `v` or inner text
- `option` — `text` (the displayed label; the `value` stays static)

Anywhere else a placeholder is just literal text (and inside `cell`
content it stays literal too).

## Syntax

A token is `{{` + letters, digits, underscore + `}}` — the same
alphabet as [ids](document.md):

```xml
<goexml version="0.1.0">
  <window title="Hello" size="320x200">
    <label id="greeting" text="Hello, {{name}}" />
  </window>
</goexml>
```

```xml
<goexml version="0.1.0">
  <window title="Status" size="320x200">
    <text id="status">Loaded {{count}} of {{total}} items</text>
  </window>
</goexml>
```

```xml
<goexml version="0.1.0">
  <window title="City" size="320x200">
    <select id="city" onChange="OnCity">
      <option value="jkt" text="Jakarta" />
      <option value="city" text="City: {{city}}" />
    </select>
  </window>
</goexml>
```

Rules:

- Multiple tokens may appear in one attribute; one value map fills them
  all.
- Malformed tokens are **rejected at decode**, not rendered literally:
  an unclosed group, an empty token, or a token with spaces is an
  error naming the position. There is no escaping — a literal `{{` in
  a label is not expressible, and the docs say so instead of inventing
  an escape syntax.

## Rendering contract (what the Go side does)

The parser stores the **raw template** — the exact text from the XML.
Until the application renders it, the raw template is what shows
(`Hello, {{name}}`). This is deliberate and documented: nothing is
rendered implicitly, and there is no hidden "must render" enforcement
because the XML side has no notion of "shown".

The Go API mirrors this contract:

- `SetTextFormat(id, vals)` re-renders the template from its raw XML
  text on every call. Missing tokens are errors that name the token;
  extra values are ignored so one map can serve many widgets.
- A plain `SetText(id, v)` **removes** the template registration for
  that id — the last call wins, so a widget is either templated or
  static, never ambiguously both. A `SetTextFormat` after that is
  rejected instead of silently resurrecting the template.
- `SetValueFormat(id, vals)` renders a select's option labels from
  their raw templates and updates the option list; the current
  selection survives by value.
- Values are always `map[string]string`; numbers are formatted by the
  application before they go in.

Token tables and value maps are **Go data**. XML never references
where values come from — that is the line the schema holds: XML
describes the shape of the hole, Go owns what goes through it.
