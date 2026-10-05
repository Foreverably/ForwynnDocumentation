# Contributing

This repository holds the source for the Forwynn documentation site, a
[Mintlify](https://mintlify.com) project covering Ticketr, Castor, Kycha, and the
Astraea Launcher.

This file covers **process and conventions**. If you are an AI agent, also read
[`AGENTS.md`](./AGENTS.md) as it holds the behavioural rules for working in this repo.

## Local development

Install the Mintlify CLI once:

```bash
npm i -g mint
```

Then run the site locally from the repo root, where `docs.json` lives:

```bash
mint dev
```

The preview is served at `http://localhost:3000`. Run `mint update` if the CLI behaves
unexpectedly.

## Before you commit

A change is not done until it passes:

```bash
mint validate
```

It catches MDX syntax errors, unresolved components, invalid `docs.json` schema, and
non-existent pages referenced from navigation. A failure here blocks deploys.

Worth checking by hand, because `mint validate` does not:

- **`docs.json` parses as JSON.** Mixed tab/space indentation is easy to break.
- **No orphaned pages.** Every `.mdx` on disk should appear in navigation. A new page
  that isn't referenced will not be built.
- **No broken internal links.** Internal links use site paths like `/ticketr/index`, not
  file paths. Confirm the target `.mdx` exists.
- **No staleness.** If a command signature or feature changes, grep the whole `ticketr/`
  tree for the old form as command syntax tends to be duplicated across overview pages and
  FAQ entries.

Commit subjects are sentence case, no prefixes:

```
Add new command documentation for Ticketr and update existing entries
```

Pushing to `main` deploys to production automatically through the Mintlify GitHub app.

Additionally, please refer to the contribution policies in the [Discord](https://discord.forwynn.net)

## Repository layout

| Path                            | Contents                                                    |
| :------------------------------ | :---------------------------------------------------------- |
| `index.mdx`                     | Site landing page, the `Home` tab                           |
| `docs.json`                     | All site config and the entire navigation tree              |
| `ticketr/`                      | Ticketr docs. `commands/` holds the slash-command reference |
| `castor/`, `kycha/`, `astraea/` | One `index.mdx` per project                                 |
| `images/`, `logo/`              | Site imagery and light/dark logos                           |
| `favicon.svg`                   | Favicon                                                     |

`README.md`, `LICENSE`, `CHANGELOG.md`, and `CONTRIBUTING.md` are ignored by Mintlify and
never built into the site.

## MDX page conventions

### Frontmatter

Every page starts with `title`, `icon`, and `description`. The `description` is what
appears under the title and in search results, so write it as a full sentence.

```yaml
---
title: "Ticket Logging"
icon: "clipboard"
description: "How Ticketr records ticket opens, closes, and updates."
---
```

Icons are Font Awesome, Lucide, or Tabler names so no `lucide-react` prefix. If a page
needs a non-standard layout, `mode` supports `wide`, `center`, `custom`, and `frame`.

### Components

Use Mintlify's own components. **Do not add bare `import` statements** as Mintlify only
permits local imports, and `import { Ticket } from "lucide-react"` fails the build. Icons
come from frontmatter and component props instead.

| Use for                       | Component                                         |
| :---------------------------- | :------------------------------------------------ |
| Grouping related links        | `<CardGroup cols={2}>` + `<Card title icon href>` |
| Documenting a field or option | `<ParamField body type required default>`         |
| Collapsible detail            | `<AccordionGroup>` + `<Accordion title>`          |
| Aside, caution, or tip        | `<Info>`, `<Note>`, `<Warning>`, `<Tip>`          |
| Ordered walkthrough           | `<Steps>` + `<Step title>`                        |
| Extra sections inside a field | `<Expandable title>`                              |

Indentation inside components uses **tabs**, matching the rest of the repo.

### Commands

The command reference lives at `ticketr/commands/`, one page per parent command:

- One page per top-level command. `commands/ticket.mdx`, not `commands/ticket-close.mdx`
- Document subcommands as `## /ticket close` sections, nested under the parent
- Open a subcommand page with a summary table of its subcommands
- State explicitly when a command takes **no options**
- Record option constraints from the Discord API payload: required, defaults, `min`/`max`,
  autocomplete
- Put the usage block above the field list:

````markdown
```bash
/ticket panel panel: <name>
```

<ParamField body="panel" type="string" required>
	The panel to post or update. This option **autocompletes**.
</ParamField>
````

### Images

Reference images by relative path from the page (`ticketpanelexample.png`), or from
`images/` for shared site assets. Commit the file alongside the page that uses it.

## Navigation and file rules

**Every `.mdx` file must be referenced from `docs.json`.** Unreferenced pages are not
built. This is the most common mistake in this repo.

Page paths in navigation omit the extension and include the folder prefix:

```json
"ticketr/commands/ticket"
```

Internal links use the same path with a leading slash and the `index` segment:

```markdown
See [Logging](/ticketr/logging) and [Getting Started](/ticketr/index).
```

Keep these conventions:

- **One tab per product**, in `navigation.tabs`, each with an `icon`.
- **Group pages within a tab** by purpose. For example, `Getting Started`, `Commands`,
  `Configuration`, `Plans`, `Help`.
- **Lowercase filenames.** `ticketr/Transcripts.mdx` predates this convention and is the
  one exception; reference it with exact case so case-sensitive builds resolve it. Rename
  it if you touch it.
- **One `index.mdx` per product folder** serves as that product's landing page.

### 1. Create the page

```bash
mkdir astraea
```

Write `astraea/index.mdx` with `title`, `icon`, and `description` frontmatter.

If the product isn't documented yet, say so plainly in an `<Info>` or `<Warning>` callout
near the top and link to whatever does exist like the project's website, its GitHub repo, or
the support server. Don't leave the page feeling abandoned.

If the product involves licensing, private servers, or third-party game data, carry the
project's own disclaimer near the top. Never invent or soften legal wording.

### 2. Add the tab

Append to `navigation.tabs` in `docs.json`:

```json
{
	"tab": "Astraea",
	"icon": "rocket",
	"pages": ["astraea/index"]
}
```

Tabs are ordered deliberately as is: `Home` first, then products. Place a new product after
the related ones rather than at the end by default.

### 3. Add the landing page card

Add a `<Card>` to `index.mdx` under the heading matching the product's type. Discord bots
go under **Discord projects**; desktop tools go under **Desktop tools**.

```markdown
<Card title="Astraea Launcher" icon="rocket" href="/astraea/index">
  A launcher for playing Honkai: Star Rail on private servers with your own beta
  build. Windows only.
</Card>
```

Update the frontmatter `description` on `index.mdx` if it enumerates products.

### 4. Validate

```bash
mint validate
```

Then confirm the new page has no orphans, the card link resolves, and the tab renders in
the right position.

### 5. Update this file

If the new product adds a directory or convention, reflect it in the layout table above.

## Reporting problems

If a page contradicts the product's actual behaviour, open an issue with the page path
and what's wrong. If the fix is unambiguous, a pull request is welcome. See
[Before you commit](#before-you-commit).
