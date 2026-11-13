# Containers and Slots

`container` and `slot` are the injection points between XML and the
runtime: a `container` is a hole in the document whose body is
supplied at runtime. This page covers the XML contract; the build and
content APIs belong to each implementation.

## `container`

A container holds **exactly one child**: the element that surrounds
the injection point. Its subtree must contain **exactly one** `slot`.

```xml
<goexml version="0.1.0">
  <window title="Page" size="480x360">
    <container id="page">
      <vbox id="shell" style="p-2" spacing="8">
        <label id="head" text="goexml page" />
        <slot style="h-full" />
        <label id="foot" text="footer stays in XML" />
      </vbox>
    </container>
  </window>
</goexml>
```

Rules:

- `id` is required — it is the name the runtime addresses the
  container by.
- Exactly one child, exactly one `<slot>` in the subtree (a count of
  zero or two or more is rejected).
- **No nesting**: a `container` may not appear inside another
  `container`. One container is one injection unit; nesting would blur
  who owns which slot. The rule is per file — siblings are fine, and
  runtime composition is unaffected.
- The container accepts the common attributes (`id`, `visible`,
  `style`) and nothing else; it has no layout attributes of its own.

## `slot`

The hole itself. Its position inside the surrounding child decides
where the injected object lands; the injected object is placed exactly
there and stretched to fill the slot's area.

- `style` is the only attribute a slot accepts — the dimension and
  spacing utilities matter (`w-full`, `h-N`, `p-*`, `m-*`): they
  decide the area the slot claims, which the injected content
  then fills. The [visual utilities](visual-style.md) (`bg-`, `b-`,
  `rounded`) are rejected on a slot — a slot is a hole, there is
  nothing to paint the layer on.
- A slot takes no children.
- A `slot` outside a `container` has no owner to fill it — exactly the
  accepted-then-dead pattern the parser rejects — so it is an error.

## What the XML side guarantees

- The document is complete and valid **before** any injection: the
  chrome (headers, footers, navigation — everything that does not
  change) is fully described in XML.
- The injection replaces the slot's content only; everything around it
  keeps the declared layout.
- Because a container can hold any single child, the slot can sit
  anywhere in a structure: a grid cell, a row of an hbox.

For page-shaped content that is swapped in and out at runtime, see
[fragments.md](fragments.md): a fragment is built against the
document's containers and replaces their slot content as a unit.
