# Glossary

The words this workflow uses, in plain language. A CRD, ticket or doc note may use any term
defined here without explaining it. Any other house term is defined where it first appears, or
replaced with plain words (`crd_how_to` § Keeping it tight).

Two parts. **Your terms** belongs to your team: setup adds the words that come up while
connecting your system, and you add more as they appear. **Kit terms** are the workflow's own;
change one only if you change how the workflow uses it.

# Part 1 · Your terms

Words your team uses that a newcomer, or an agent, wouldn't know: product names, internal
nicknames for components, token families, team names, anything said in a meeting that would
puzzle someone from outside. Say what it means here, and what it's called in Figma or the code
if that differs.

| Term | Means | Also called |
|---|---|---|
| *Example (Acme):* Shelf | The horizontal scrolling row of cards on a home screen | `Carousel` in the code |

# Part 2 · Kit terms

## CRDs

| Term | Means |
|---|---|
| **CRD** | Component Reference Document. One per component: what it must do, the decisions behind it, and what Figma and the code can't say. Consulted for the component's whole life, not just before build |
| **Statement** | One numbered requirement in section 1, e.g. `B3`. True or false, never half. Only statements are the contract |
| **Statement prefixes** | F functional, S states, B behaviour, C content, A accessibility, I integration |
| **Contract** | All the statements in section 1 together: what the build must meet and the tests check |
| **Anatomy** | The named parts of a component (container, label, leading icon). Statements use these nouns and no others |
| **`[open]`** | Marks a statement as a provisional hunch, not yet decided. Design settles every one before dev |
| **`[review]`** | Marks a statement a test can't check, so a person verifies it in review |
| **Open question** | A bigger unknown with a named owner and a "by when", kept in a table rather than marked `[open]` |
| **Ruled out** | Options considered and rejected, one line each with the reason, so nobody proposes them again |
| **Design decision** | A choice someone could plausibly have made the other way, logged with its date and why |
| **Precedent** | Another CRD whose decisions this one should stay consistent with |
| **Dependency** | Another CRD an agent must read to build this one: a child part, or a parent that sets rules for it |
| **Lifecycle** | Where the CRD's component is: requested, backlog, active, stable, built, deprecated or rejected (`crd_how_to` § Lifecycle) |
| **Section status** | Whether each CRD section is not started, in progress or complete |
| **Amendment** | A change to a component that has already shipped, written as a temporary block at the top of its CRD while design works on it |
| **Fold** | Merging a finished amendment into the main requirements and deleting the temporary block |
| **Ready for dev** | A condition, never a stored flag: requirements and design specs complete, the design review held, no `[open]` left in section 1, every open question closed or deferred |
| **Design review** | A designer checking the CRD against the Figma component while designing it |
| **Dev review** | A developer checking every statement is buildable and testable, as the first step of building it |
| **Atom versus consumer** | A basic component owns how it looks; what it means and where it goes belong to the components that use it |
| **Stable component** | A component built and shipped before anyone designed it properly. It works, and is waiting for design to harden it |
| **Global accessibility rules** | The rules every component meets (`accessibility.md`). A CRD never repeats them; it holds only what's true of that one component |

## Design and Figma

| Term | Means |
|---|---|
| **Component library** | The Figma file holding the system's components, published for other files to use |
| **Component set** | One component with all its variants, in Figma |
| **Variant** | One combination of a component's options, e.g. size large, state hover |
| **Variant axis** | One option that varies, e.g. size or state |
| **Figma-only prop** | An option that exists in Figma to help designers but has no equivalent in the code |
| **Code-only prop** | An option in the code that Figma can't show |
| **Token** | A named design value (a colour, a spacing size) stored as a Figma variable and exported to code |
| **Token tiers** | Layers of tokens, each pointing at the one below. Components use only the tier your foundations name |
| **Mode** | One setting of a variable collection, e.g. light or dark. Switching a frame's mode swaps every value at once |
| **Mode axis** | One independent kind of mode, e.g. theme or breakpoint |
| **Binding** | A link from a layer's colour, spacing or size to a token, so it follows the token |
| **Text style** | A saved type setting. Every text layer uses one, never hand-picked fonts |
| **Publish** | Releasing library changes so other files, and developers, can see them |
| **Context card** | A card at the top of a component's Figma page: links, status and a short changelog |
| **Doc frames** | The documentation frames beside the component on its page: anatomy, usage and modes |
| **Doc note** | A short labelled caption in a doc frame: which CRD rule it shows, a title, one line of description |
| **Demo** | An example of the component in use on its Figma page. Designers copy from these |
| **Specimen tile** | The backdrop behind a demo, to show it on a realistic surface. Not part of the component |
| **Dev Mode annotation** | A note pinned to a layer for developers, shown in Figma's Dev Mode |
| **Dev resource link** | A link on a component that Dev Mode shows to developers. Here, it points at the CRD |
| **Auto-layout** | Figma's way of arranging a frame's contents in rows, columns or a grid, so it resizes properly |
| **Spacer frame** | An empty frame used to push things apart. Never used: it leaks into every copy |
| **Clip content** | A frame setting that hides whatever spills outside it. Off unless it's deliberate and annotated |
| **Detached copy** | Something that looks like a component but is no longer linked to it, so it won't update |
| **Page status symbols** | Emoji after a page name in the library, showing its state at a glance (config § Figma) |

## Tickets

| Term | Means |
|---|---|
| **Tracker** | Wherever the team keeps tickets: GitHub Issues, Azure Boards, Jira, Trello |
| **Container** | The ticket grouping all the work for one component or one amendment |
| **Work item** | One unit of work inside a container: one pull request, one person |
| **Defect** | A bug in a component that has shipped |
| **Stage** | Where a ticket is: new, ready for dev, in progress, in review, verify, done, blocked, removed |
| **Verify** | Merged, and waiting for someone to confirm it works. Not yet done |
| **Spin-up** | Raising a component's container and its work items in one go |
| **Ticket ref** | How a commit or pull request cites a ticket, e.g. `#123` |
| **Adapter** | The kit file that knows one tracker's mechanics, so the workflow works the same in any tracker |

## Setup and the repo

| Term | Means |
|---|---|
| **Config** | `.claude/design-system.config.md`: your system's details, which every skill reads |
| **Foundations** | `design-foundations.md`: your token architecture, plus general Figma know-how |
| **System rules** | Your system's own quirks, each written as a rule and the question it raises when writing a CRD |
| **House rule** | A convention the kit ships with that your team may change |
| **Skill** | A set of instructions Claude follows for one job, e.g. writing a CRD |
| **Integration branch** | The branch pull requests merge into, e.g. `main` or `develop` |
| **Skill owners** | The people who change the skills and reference docs, in their own pull requests |
