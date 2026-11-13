# Dynamic Values

Static text lives in XML attributes. Values that change at runtime are
set by id from the runtime side — **there is no logic in XML**: no
expressions, no `if`, no loops. What XML does have is the `{{token}}`
placeholder: a hole in a text attribute that the runtime fills with
values.

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
  <window title="Hello">
    <label id="greeting" text="Hello, {{name}}" />
  </window>
</goexml>
```

```xml
<goexml version="0.1.0">
  <window title="Status">
    <text id="status">Loaded {{count}} of {{total}} items</text>
  </window>
</goexml>
```

```xml
<goexml version="0.1.0">
  <window title="City">
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

## Rendering contract

The parser stores the **raw template** — the exact text from the XML.
Until the application renders it, the raw template is what shows
(`Hello, {{name}}`). This is deliberate and documented: nothing is
rendered implicitly, and there is no hidden "must render" enforcement
because the XML side has no notion of "shown".

Every implementation that supports dynamic values must honor this
contract:

- **Format render**: re-rendering a widget's template from its raw XML
  text replaces the displayed text; a value map that omits a token the
  template declares is an error naming the token, and extra values are
  ignored so one map can serve many widgets.
- **Plain set wins**: setting a widget's text without values removes
  the template registration for that id — the last call wins, so a
  widget is either templated or static, never ambiguously both.
  Re-rendering after that is rejected instead of silently resurrecting
  the template.
- **Selects render by label**: rendering a select's option labels from
  their raw templates updates the option list; the current selection
  survives by value.
- Values are always plain string maps; numbers are formatted by the
  application before they go in.

Token tables and value maps are **runtime data**. XML never references
where values come from — that is the line the schema holds: XML
describes the shape of the hole, the runtime owns what goes through it.
