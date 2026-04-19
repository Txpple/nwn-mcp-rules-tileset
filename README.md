# nwn-mcp-rules-tileset

Documentation and Claude Code rules for working on Neverwinter Nights: Enhanced Edition modules and custom content with AI assistance. This repo is geared specifically toward custom content creation for tilesets — the rules below cover NWN Enhanced Edition tileset development, letting you use Claude to edit your `.set` files, ITP files, and perform many tileset-related tedious operations with AI assistance.

## Using Claude Code for working on a module

You can use an MCP (Model Context Protocol) server for Neverwinter Nights: Enhanced Edition module files for AI assisted module editing. You stay in creative control while an AI helps build, modify, and extend your module via natural language. You can get the tool here: https://github.com/Txpple/nwn-mcp

## Using Claude Code for working on custom content

If you use Claude Code, it can significantly help your custom content work, as well as modernize your workflows past using legacy tools like Tileset Duplicator, Set Editor, GFFEdit to manage your palettes, and the like. This assumes you have working knowledge of Claude Code and/or are willing to teach yourself. Of course you need a subscription, too. To make the most of it, having CLI tools are essential, and you should get them at https://github.com/niv/neverwinter.nim

One way you can help your workflow is set your working folder to `development` or `override` in Claude Code and use rules to inform it of the files you are working with. Claude Code rules are markdown files that provide Claude with domain-specific knowledge when working in a project. They act as persistent reference material — whenever Claude encounters relevant files, the rules automatically load into context so it understands the conventions, file formats, and pitfalls specific to your work.

## How to Use These Rules

1. In your NWN project directory that CC is pointed to (usually `development` or maybe `override`), create a `.claude/rules/` folder.
2. Save each rule as a `.md` file in that folder (e.g., `.claude/rules/nwn-set-file-format.md`).
3. When you start a Claude Code session in that project, the rules load automatically based on the files you're working with.
4. Rules can optionally include a `paths:` frontmatter to scope when they load. For example, a rule with `paths: ["**/*.set"]` only activates when you're working with `.set` files.

## Available Rules

| Rule | Applies to | Description |
| --- | --- | --- |
| SET File Format | `.set`, `.mdl`, `.wok` files | Structure of the `.set` tileset definition, terrain/crosser types, tile entries, door sections, groups, and MDL/WOK internal references |
| ITP Palette Format | `.itp` files | Structure of the `.itp` toolset palette, branch IDs, subfolder ID values, terrain paint layout, and JSON entry formats |
| Tileset Operations | `.set`, `.itp`, `.mdl`, `.wok` files | Procedures for reordering tiles, duplicating terrain families, adding tiles, troubleshooting, and verification |
