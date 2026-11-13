# Fragments (`goexml-item`)

A fragment is an injectable piece of UI in its own file: a page body,
a card, a panel — anything built outside the document and attached at
runtime. Fragments keep the XML-only contract while letting the
application compose screens from parts.

```xml
<goexml-item version="0.1.0">
  <vbox id="pageBody" padding="8" spacing="4">
    <label id="title" text="Orders" />
    <rect id="sep" color="#2e2e2e" style="w-full h-0" />
    <label id="summary" text="12 open orders" />
  </vbox>
</goexml-item>
```

## Envelope rules

- The root is `<goexml-item>` with `version="0.1.0"` — the same
  version as a document.
- The fragment holds **exactly one child widget**; wrap a group in a
  `<vbox>`, like in a document.
- There is no `<window>`: a fragment is body content, not a surface.
- The same schema rules apply as everywhere else — the child tree is
  validated with the identical checker, so a fragment cannot contain
  anything a document could not.

## Ids join the document's id space

A fragment's ids become part of the built screen's global id space:

- Ids must be unique **within the fragment**.
- A fragment id must **not collide with the document's ids** — that is
  a build error.
- Fragment ids may replace **each other** across rebuilds.

## Containers and slots inside fragments

A fragment may itself contain a `container` with a `slot`, with the
same rules as in a document: the content for such an inner slot is
supplied when the fragment is built. This lets a fragment be both body
and shell at once.

## Rebuild semantics: the last built page wins

Fragments are typically rebuilt in place — switching from page A to
page B means building fragment B where fragment A stood:

- Rebuilding replaces the fragment's ids and slot content.
- Ids the new fragment **no longer declares** become unreachable:
  widgets, selects, and templates of the old page never outlive it in
  lookups or value setters.
- A shared id that rebuilt into a **plain** widget loses the previous
  page's state: a select that was templated stays plain, a label with
  a template that rebuilt as static text no longer carries the old
  template. The last call wins, per id.
- The document's own ids are never touched by a fragment rebuild.

## Appending fragments into a `vbox-expand`

Fragments are also the unit of growth for the
[`vbox-expand`](layout.md) stack: the runtime builds a fragment per
appended item. Unlike a page rebuild, each appended item keeps its
**own private id space** — the same fragment can be appended many
times without its ids colliding, and per-item slot content and image
resources are supplied at each append. Emptying the stack retires
that stack's items; the document's global id space is never touched.

## Handlers

`onClick` / `onChange` / `onSelect` inside a fragment are handler
**references**, identical to a document's. The names are resolved
against the same handler map the document uses; a fragment cannot
introduce new handler implementations, only reference existing ones.
