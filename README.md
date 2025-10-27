# cursor-rules

**Author: @fellyph**

A comprehensive collection of instructions, rules, and guidelines for Cursor IDE, Claude Code, GitHub Copilot, and other AI coding assistants. Organized by category to support different development workflows and tools.

## Directory Structure

```
cursor-rules/
├── rules/                        # Cursor IDE rules and coding standards
│   ├── wordpress-global-standards.mdc
│   └── wordpress-js-plugins.mdc
├── claude/                       # Claude Code guidance and CLAUDE.md documentation
│   ├── CLAUDE-playground.md              # WordPress Playground project guide
│   ├── CLAUDE-playground-website.md      # Playground website package guide
│   └── skills/                           # Specialized skills for Claude
│       └── SKILL-BLUEPRINT-MAKER.md      # WordPress Playground Blueprint creation
├── github-instructions/          # GitHub Copilot instructions (Coming soon)
├── agents/                       # AI agent configurations (Coming soon)
├── playwright/                   # Playwright testing guidelines (Coming soon)
├── CLAUDE.md                     # Metadata and guidance for Claude Code
└── README.md                     # This file
```

## Categories

### `/rules/` - Cursor IDE Rules

Cursor-specific rules for different technology stacks:
- **wordpress-global-standards.mdc** - General WordPress coding standards
- **wordpress-js-plugins.mdc** - JavaScript-based WordPress plugin standards

### `/claude/` - Claude Code Guidance

Comprehensive guides (CLAUDE.md format) for Claude Code when working on specific projects:
- **CLAUDE-playground.md** - Complete WordPress Playground project guide
- **CLAUDE-playground-website.md** - Website package and component standards

### `/claude/skills/` - Claude Skills

Specialized skill guides for specific tasks:
- **SKILL-BLUEPRINT-MAKER.md** - Creating and managing WordPress Playground Blueprints

### `/github-instructions/` - GitHub Copilot Instructions

Instructions for GitHub Copilot (coming soon)

### `/agents/` - AI Agent Configurations

Configurations and prompts for specialized AI agents (coming soon)

### `/playwright/` - Playwright Testing Guidelines

Guidelines and patterns for Playwright testing (coming soon)

## Usage

1. **For Cursor IDE**: Place `.mdc` files from `/rules/` in your project's Cursor rules directory
2. **For Claude Code**: Reference the appropriate `CLAUDE.md` file when working on projects
3. **For GitHub Copilot**: Use files from `/github-instructions/` in your repository
4. **For Custom Agents**: Adapt configurations from `/agents/` for your specific needs
5. **For Testing**: Reference patterns and guidelines from `/playwright/`

## Current Focus

The initial focus is on **WordPress ecosystem** development:
- WordPress Playground (full-stack JavaScript/PHP WebAssembly project)
- WordPress block development and standards
- WordPress plugin and theme development

## Contributing

When adding new rules or guidelines:
1. Choose the appropriate category folder
2. Follow existing naming conventions
3. Include a descriptive header explaining the scope
4. Keep documentation focused and actionable

## License

Please check individual files for license information.
