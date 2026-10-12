# Component classification

Every CRD classifies its component with three front matter keys, defined here. The skills use
them to fill CRD front matter and to place component pages in Figma.

```
tier: core            # core | feature
composition: basic    # basic | composite
category: actions     # one value from the table below
```

## Tier

| Value | Meaning |
|---|---|
| `core` | General-purpose and product-agnostic. Usable anywhere. |
| `feature` | Built for one product area or context, such as AI or a CMS. |

When unsure, choose `core` only if the component makes sense outside the context it was
designed for.

## Composition

| Value | Meaning |
|---|---|
| `basic` | A leaf. It renders no other design system components. |
| `composite` | Built from other design system components. |

An icon does not make a component composite.

## Category

Choose one category, by the component's primary job. A composite is classified by what the
whole component does, not by its parts.

**Fill this in for your system.** The table is a starter set. Rename, merge or drop rows; keep
the `Value` column lowercase kebab-case
(it becomes the first segment of Figma set names, per config § Figma → Library conventions →
`set_name`), and keep the rows in the order you want category headers to appear in the Figma
library.

| Value | Category | What belongs | Examples |
|---|---|---|---|
| `foundations` | Foundations | Non-visual providers, theming, and the pages that describe how the system works rather than what it renders | Scrollbar, locale provider, design tokens, typography scale, colour docs |
| `actions` | Actions | Things the user clicks to make something happen | Button, Link, Chip, Toggle button |
| `forms` | Forms | Anything that collects a value from the user, plus the labelling and validation scaffolding around it | Text field, Search bar, Checkbox, File upload, Label, Fieldset |
| `navigation` | Navigation | Wayfinding: moves the user between locations or shows where they are | Breadcrumb, Tabs, Stepper, Pagination |
| `feedback` | Feedback | Tells the user what the system is doing or what just happened: status, progress, and the absence of content | Toast, Status indicator, Badge, Banner, Progress bar, Spinner, Skeleton, Empty state |
| `overlays` | Overlays | Content rendered above the page in its own layer, dismissible, usually trapping or managing focus | Modal, Tooltip, Popover, Drawer, Context menu |
| `containers` | Containers | Bounded boxes with a visible frame that hold arbitrary content, from a card up to the page regions that frame a site | Card, Accordion, Header, Footer, Sidebar |
| `data-display` | Data display | Presents content the consumer supplies: lists, tables, identity and text structure | List, List item, Avatar, Avatar group, Table, Blockquote |
| `media` | Media | Images, video, audio and their controls | Image, Video player, Document preview, Logo |
| `layout` | Layout | Positions and separates other content rather than carrying content of its own | Divider, Stack, Grid |
| `utilities` | Utilities | Shared helpers: hooks and functions | |

Nothing fits → ask. Never add a category on your own; a new category is a decision for config
§ Roles → `skill_owners`.
