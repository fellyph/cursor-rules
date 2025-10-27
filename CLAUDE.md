# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

The **cursor-rules** repository is a curated collection of detailed Cursor/Claude AI agent instructions and guidelines for various technical projects and practices. These files are designed to provide Claude Code with comprehensive context about specific projects, architectural patterns, development workflows, and coding standards.

This is a **prompt library** that guides Claude Code when working on external projects like WordPress Playground and WordPress-based development.

## Repository Structure

### Main Directories

```
cursor-rules/
├── README.md                     # High-level overview of the repository
├── CLAUDE.md                     # This file (metadata for Claude Code)
├── rules/                        # Cursor rules and coding standards
│   ├── wordpress-global-standards.mdc
│   └── wordpress-js-plugins.mdc
├── claude/                       # Claude Code guidance and documentation
│   ├── CLAUDE-playground.md              # WordPress Playground project guide
│   ├── CLAUDE-playground-website.md      # Playground website package guide
│   └── skills/
│       └── blueprint-maker/              # WordPress Playground Blueprint creation
│           ├── SKILL.md                  # Main Blueprint creation guide
│           ├── wp-cli-commands.md        # WP-CLI command reference
│           ├── woocommerce-cli-commands.md # WooCommerce CLI reference
│           └── wp-cli-troubleshooting.md # WP-CLI troubleshooting guide
├── github-instructions/          # GitHub Copilot instructions
├── agents/                       # AI agent configurations
├── playwright/                   # Playwright testing guidelines
```

### File Categories

#### **Cursor Rules** (`/rules/`)

- **`wordpress-global-standards.mdc`** (~3.3 KB)
  - General WordPress coding standards and best practices
  - Format: `.mdc` (proprietary/specialized format)
  - For: Cursor IDE rule configuration for WordPress projects

- **`wordpress-js-plugins.mdc`** (~10.3 KB)
  - Standards specific to JavaScript-based WordPress plugin development
  - Format: `.mdc` (proprietary/specialized format)
  - For: Cursor IDE rule configuration for WordPress JS/PHP hybrid projects

#### **Claude Code Guidance** (`/claude/`)

- **`CLAUDE-playground.md`** (~16.8 KB)
  - Complete guide for the WordPress Playground project (Nx monorepo)
  - Covers: Architecture, 25+ packages (php-wasm, playground families), development commands
  - Includes: Testing strategies, PHP-WASM core classes, blueprints system, build system details
  - For: Contributors to the WordPress Playground codebase itself
  - Key sections: Project Overview, Repository Structure, Development Commands, Architecture Overview
  - Location: `claude/CLAUDE-playground.md`

- **`CLAUDE-playground-website.md`** (~36.4 KB)
  - Detailed guide for the `@wp-playground/website` React component package
  - Covers: Component architecture, Redux state management, React patterns, CSS Modules, i18n
  - Includes: Comprehensive testing with Playwright, debugging strategies, common development tasks
  - For: Frontend developers working on the Playground website UI specifically
  - Key sections: Component Structure, TypeScript Standards, Testing Strategy, Common Tasks
  - Location: `claude/CLAUDE-playground-website.md`

#### **Claude Skills** (`/claude/skills/`)

- **`blueprint-maker/SKILL.md`** (~31.5 KB)
  - Specialized guide for creating WordPress Playground Blueprints (JSON configuration files)
  - Covers: Blueprint JSON schema, 25+ step types, resource references, best practices
  - Includes: Common patterns, troubleshooting, validation, loading methods
  - For: Anyone creating or maintaining Blueprint JSON configurations
  - Key sections: Blueprint Structure, Step Reference, Advanced Features, Schema Validation
  - Location: `claude/skills/blueprint-maker/SKILL.md`

- **`blueprint-maker/wp-cli-commands.md`** (~6.6 KB)
  - Reference guide for WP-CLI commands used in WordPress Playground Blueprints
  - For: Understanding available WP-CLI operations in Blueprint steps
  - Location: `claude/skills/blueprint-maker/wp-cli-commands.md`

- **`blueprint-maker/woocommerce-cli-commands.md`** (~10.7 KB)
  - Reference guide for WooCommerce CLI commands
  - For: Creating Blueprint steps that configure WooCommerce
  - Location: `claude/skills/blueprint-maker/woocommerce-cli-commands.md`

- **`blueprint-maker/wp-cli-troubleshooting.md`** (~7.9 KB)
  - Troubleshooting guide for common WP-CLI issues in Blueprints
  - For: Debugging Blueprint execution problems
  - Location: `claude/skills/blueprint-maker/wp-cli-troubleshooting.md`

## Development Workflow

This repository has **no build process, linting, testing, or CI/CD automation**. It is purely documentation.

### Maintenance Process

1. **Updating existing guides**: Edit the relevant `.md` file in `/rules/`
2. **Adding new guides**: Create a new `.md` file in `/rules/` following the naming convention
3. **Referencing**: Include the schema reference block at the top of new guides for Cursor compatibility
4. **Version control**: Use git to track changes with descriptive commit messages
5. **Testing**: Validate that markdown renders correctly and links work

### File Naming Conventions

- **Project-specific guides**: `CLAUDE-<project-name>.md` or `CLAUDE-<project>-<component>.md`
- **Skill guides**: `SKILL-<feature-name>.md`
- **Standards guides**: `<technology>-<specialty>-standards.md` or `.mdc`

## Key Information About Documented Projects

### WordPress Playground

The WordPress Playground is a complex Nx monorepo that:
- Runs WordPress entirely in WebAssembly (WASM) in the browser
- Uses PHP compiled to WASM with JavaScript bindings
- Contains 25+ npm packages organized into two families: `php-wasm/*` and `playground/*`
- Supports multiple PHP versions (7.2-8.4) and WordPress versions
- Uses Comlink for RPC, Service Workers for request interception, and Jest for testing

**Common commands documented:**
- `npm run dev` - Start development server
- `npm run build` - Build all packages
- `npx nx test <package-name>` - Run tests for specific package
- `npm run lint` - Lint all packages
- `npm run format` - Format code with Prettier

### WordPress Blueprints

Blueprints are declarative JSON configuration files that set up WordPress Playground instances. The guide documents:
- Schema structure and validation
- 25+ step types (install, configure, operate, etc.)
- Resource descriptors (WordPress.org, literal, VFS, URL, GitHub)
- Best practices for creation and testing

## High-Level Architecture Context

### For WordPress Playground Project

```
Architecture: Client-Server via Comlink + postMessage
├── Parent Frame (startPlaygroundWeb)
├── Remote.html iframe
│   ├── Service Worker (request interception)
│   ├── PHP Worker Thread
│   │   └── PHP + WordPress runtime
│   └── Comlink RPC bridge
└── Client code (postMessage protocol)
```

Key patterns:
- RPC via Comlink serializes method calls across thread boundaries
- Semaphore-based concurrency (single-threaded PHP, default concurrency = 1)
- Event system for lifecycle hooks (request.start, request.end, request.error)
- Disposable pattern for resource cleanup
- Service Worker as stateless request interceptor

### For @wp-playground/website Package

Component structure:
- Feature-based folder organization
- Shared components in `/shared/` directory
- Redux for state management (reducers, actions, selectors)
- CSS Modules for component styling
- Comprehensive Playwright E2E tests (50+ tests)

Testing strategy:
- Unit tests colocated with source files
- E2E tests in `/e2e/` directory using Playwright
- TypeScript strict mode enabled
- Coverage expectations documented

## When to Update This File

Update this CLAUDE.md file when:
- Adding new guidance files to the `/rules/` directory
- Significantly restructuring the repository
- Changing naming conventions or organizational patterns
- Adding new categories of guidance files

**Do not** repeat content from individual guide files here. This file provides structural overview only.

## Quick Reference

| Location | File | Purpose | Audience | Size |
|----------|------|---------|----------|------|
| claude/ | CLAUDE-playground.md | Complete WordPress Playground project guide | Project contributors | ~16.8 KB |
| claude/ | CLAUDE-playground-website.md | Website package React/Redux guide | Frontend developers | ~36.4 KB |
| claude/skills/blueprint-maker/ | SKILL.md | Blueprint JSON creation guide | Blueprint developers | ~31.5 KB |
| claude/skills/blueprint-maker/ | wp-cli-commands.md | WP-CLI command reference | Blueprint developers | ~6.6 KB |
| claude/skills/blueprint-maker/ | woocommerce-cli-commands.md | WooCommerce CLI reference | WooCommerce developers | ~10.7 KB |
| claude/skills/blueprint-maker/ | wp-cli-troubleshooting.md | WP-CLI troubleshooting guide | Blueprint developers | ~7.9 KB |
| rules/ | wordpress-global-standards.mdc | General WordPress standards | WordPress developers | ~3.3 KB |
| rules/ | wordpress-js-plugins.mdc | WordPress JS plugin standards | JS/PHP hybrid developers | ~10.3 KB |

## Repository History

- **Created**: October 2025
- **Maintained by**: @fellyph
- **Git remote**: https://github.com/fellyph/cursor-rules.git
- **Current branch**: main
- **Recent focus**: WordPress Playground documentation and Blueprint guidance
