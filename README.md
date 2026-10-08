# naz-ux-skills

Agent skills for UX/UI designers, by **naz** (Md. Nazmul Hassan).

## Skills

### `ux-design-audit`
A senior-level UX audit of Figma designs: dashboards, websites, landing pages and mobile apps.

It reviews six areas:
- **UX copy & content**: labels, CTAs, error and empty states, consistency (with rewritten copy)
- **UX complexity**: cognitive load, hierarchy, steps, density
- **Usability heuristics**: Nielsen's 10, with platform conventions for mobile, web and dashboards
- **Accessibility**: WCAG 2.2 AA checks that can be verified from a static design
- **Color**: computed contrast ratios, color-only meaning, color-vision safety, palette consistency
- **Severity**: every issue rated Critical / High / Medium / Low, plus an overall score

The agent then creates a **"UX Audit Report"** page inside your Figma file with:
- A summary (score, issue counts, top priorities, what works well)
- Annotated copies of each screen with numbered issue markers (your original frames are never modified)
- An issues table: `#` · Screen / Element · Category · Issue · Why it matters · Severity · Guideline · Recommended solution

## Install

```bash
npx skills add nazmulhassan0007/naz-ux-skills
```

Or copy it manually for Claude Code:

```bash
git clone https://github.com/nazmulhassan0007/naz-ux-skills.git
cp -r naz-ux-skills/skills/ux-design-audit ~/.claude/skills/
```

## Requirements
- The **Figma MCP server** connected to your agent. For Claude Code:
  ```bash
  claude mcp add --transport http figma https://mcp.figma.com/mcp
  ```
  then run `/mcp` and log in to Figma.
- **Edit access** to the Figma file (the report is written into it).

## Usage
Copy a frame link in Figma (**Copy link to selection**, so the URL includes `node-id`), then:

```
/ux-design-audit https://www.figma.com/design/XXXX/File?node-id=12-345
```

or just ask in plain language:

```
Review this dashboard design and tell me where to improve: <figma link>
```

## License
MIT © naz
