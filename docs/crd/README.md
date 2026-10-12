# CRDs: Component Reference Documents

A CRD (Component *Reference* Document) travels with a component for its whole life and
carries what Figma, the code and the agent rules cannot tell you. `crd-how-to.md` § What a CRD
is explains why.

## Layout

Every CRD lives in this one directory. State is the `lifecycle` key in its front matter —
`requested`, `backlog`, `active`, `stable`, `built`, `deprecated` or `rejected` — never the
folder. The values and the track between them are defined in `crd-how-to.md` § Lifecycle.

## Finding one

Glob this folder. Filenames follow config § Paths → `crd_filename` (`Basic_Card_CRD.md`), but
match against the `name:` front matter too. The `id` is the stable handle; the filename is a
convenience.

## The documents

| File | What it is |
|---|---|
| `crd-template.md` | What a new CRD starts from |
| `crd-how-to.md` | The guide for people: conventions, sections, versioning, lifecycle |
| `examples/` | Worked CRDs to copy the shape from |

## Format

CRDs carry `format: 2` in front matter and follow the template. A CRD in an older or
different shape converts **on touch**, when a substantive change reaches it, not in bulk.
