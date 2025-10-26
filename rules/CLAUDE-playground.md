# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

WordPress Playground is an experimental in-browser WordPress environment that runs entirely in WebAssembly (WASM) without requiring a PHP server. It enables users to run full WordPress instances directly in their browser using PHP compiled to WebAssembly.

**Live Demo**: https://playground.wordpress.net/
**Documentation**: https://wordpress.github.io/wordpress-playground/

## Repository Structure

This is an Nx monorepo with two main package families:

### PHP-WASM Packages (`packages/php-wasm/*`)
PHP runtime compiled to WebAssembly with JavaScript bindings:
- `@php-wasm/universal` - Core platform-agnostic PHP API and interfaces
- `@php-wasm/web` - Browser-optimized PHP runtime with Service Worker support
- `@php-wasm/node` - Node.js-optimized PHP runtime with full extension support
- `@php-wasm/cli` - Command-line interface for running PHP
- `@php-wasm/util` - Helper utilities (Semaphore, error handling, path utilities)
- `@php-wasm/progress` - Progress tracking for long operations
- `@php-wasm/logger` - Logging infrastructure
- `@php-wasm/fs-journal` - Filesystem operation journaling for syncing
- `@php-wasm/stream-compression` - Stream compression utilities
- `@php-wasm/compile` - Build system for compiling PHP to WebAssembly

### Playground Packages (`packages/playground/*`)
WordPress Playground application layer:
- `@wp-playground/client` - JavaScript/TypeScript client API for controlling Playground instances
- `@wp-playground/blueprints` - Declarative JSON configuration system for WordPress setup
- `@wp-playground/remote` - Bridge between client iframe and worker threads
- `@wp-playground/storage` - Filesystem and data persistence (IndexedDB, OPFS)
- `@wp-playground/sync` - Real-time synchronization between Playground instances
- `@wp-playground/components` - Reusable React UI components
- `@wp-playground/website` - Main playground.wordpress.net SPA
- `@wp-playground/wordpress` - WordPress integration utilities
- `@wp-playground/wordpress-builds` - Pre-minified WordPress distributions
- `@wp-playground/cli` - Command-line interface for local development
- `@wp-playground/common` - Shared constants and utilities

## Development Commands

### Setup
```bash
# Clone with faster options (only latest trunk revision)
git clone -b trunk --single-branch --depth 1 --recurse-submodules https://github.com/WordPress/wordpress-playground.git
cd wordpress-playground
npm install
```

### Development
```bash
# Start development server (website at http://127.0.0.1:5400/)
npm run dev

# Build all packages
npm run build
# Or specific package
npx nx build <package-name>

# Development server for docs
npm run dev:docs

# Build documentation site
npm run build:docs
npx nx build docs-site
```

### Testing
```bash
# Run all tests
npm test

# Run tests for specific package
npx nx test <package-name>

# Run single test file
npx nx test <package-name> -- <test-file-path>

# Type checking
npm run typecheck
npx nx run-many --all --target=typecheck
```

### Linting and Formatting
```bash
# Lint all packages
npm run lint

# Format code
npm run format

# Format only uncommitted changes
npm run format:uncommitted
```

### PHP WASM Commands
```bash
# Build and run PHP.wasm CLI
npx nx start php-wasm-cli

# Recompile PHP 7.0-8.4 for web (asyncify)
npx nx recompile-php:all php-wasm-web

# Recompile PHP 7.0-8.4 for node
npx nx recompile-php:all php-wasm-node

# Recompile with DWARF debug info
npx nx recompile-php:all php-wasm-node --WITH_DEBUG=yes

# Recompile with source maps
npx nx recompile-php:all php-wasm-node --WITH_SOURCEMAPS=yes

# Run PHP.wasm in CLI
npx @php-wasm/cli -v
PHP=7.4 npx @php-wasm/cli -v
```

### WordPress Builds
```bash
# Build latest WordPress releases
npx nx bundle-wordpress:all playground-wordpress-builds
npm run rebuild:wordpress-builds
```

### Playground CLI (Local Development)
```bash
# Start server in plugin/theme directory
cd my-plugin-or-theme-directory
npx @wp-playground/cli server --auto-mount

# Specify PHP and WordPress versions
npx @wp-playground/cli server --wp=6.8 --php=8.4

# Use blueprint
npx @wp-playground/cli server --blueprint=./my-blueprint.json

# Run blueprint without server
npx @wp-playground/cli run-blueprint --blueprint=./blueprint.json

# Build snapshot
npx @wp-playground/cli build-snapshot --blueprint=./blueprint.json --outfile=site.zip
```

## Architecture Overview

### Client-Server Pattern via Comlink + postMessage

```
Parent Frame (startPlaygroundWeb)
    ↓ (message passing)
    ├─ Remote.html iframe
    │   ├─ Service Worker (request interception)
    │   ├─ PHP Worker Thread
    │   │   └─ PHP + WordPress runtime
    │   └─ Comlink RPC bridge
    ↓ (postMessage protocol)
Client code
```

### Communication Flow
1. **Client Initialization**: `startPlaygroundWeb()` creates iframe pointing to `remote.html`
2. **Remote Boot**: `bootPlaygroundRemote()` in iframe registers Service Worker and spawns PHP Worker
3. **Client Connection**: `consumeAPI()` creates RPC proxy to worker via Comlink
4. **Method Calls**: All client calls serialized via postMessage, executed in worker, results returned
5. **Service Worker**: Intercepts fetch requests from WordPress, routes to worker

### Key Architectural Patterns

- **RPC via Comlink**: Serializes method calls across thread/process boundaries
- **Semaphore-based Concurrency**: Single-threaded PHP execution (concurrency = 1)
- **Event System**: PHP instances dispatch `request.start`, `request.end`, `request.error`
- **Disposable Pattern**: Resources implement `Disposable`/`AsyncDisposable` for cleanup
- **Service Worker as Request Interceptor**: Stateless routing, caching for offline support
- **Runtime Rotation**: PHP runtime recreates after errors or request limits (default: 400 requests)
- **Progress Tracking**: `ProgressTracker` monitors multi-step operations with step weighting

## Blueprints System

Blueprints are JSON configuration files that declaratively set up WordPress instances. They support 25+ step types including:

**Installation**: `installPlugin`, `installTheme`, `runWpInstallationWizard`
**Configuration**: `setSiteOptions`, `defineSiteUrl`, `setPhpConstants`
**Operations**: `login`, `activatePlugin`, `activateTheme`
**Data**: `runSQL`, `runPHP`, `importWxr`, `importThemeStarterContent`
**Filesystem**: `writeFile`, `writeFiles`, `mkdir`, `rm`, `cp`, `mv`
**Advanced**: `enableMultisite`, `setSiteLanguage`, `resetData`, `wpCli`

### Blueprint Execution Pipeline
```
Blueprint JSON → compileBlueprintV1() → runBlueprintV1Steps() → Execution
```

### Resource Descriptors
- `wordpress.org/plugins` - Direct WP.org plugin downloads
- `wordpress.org/themes` - Direct WP.org theme downloads
- `literal` - Inline file content
- `vfs` - Virtual filesystem paths
- `url` - HTTP URLs (with CORS proxy support)
- `github` - GitHub repository references with sparse checkout

## PHP and WordPress Version Support

### PHP Versions
Supported: 7.2, 7.3, 7.4, 8.0, 8.1, 8.2, 8.3, 8.4
Build variants: Asyncify (slower, more compatible) and JSPI (faster, newer spec)

### WordPress Versions
Supported: 6.3, 6.4, 6.5, 6.6, 6.7, 6.8, beta, nightly
Pre-minified builds available for smaller downloads

## Testing Strategy

- Test files use `.spec.ts` extension
- Tests located alongside source files in `src/` directories
- Framework: Jest with ts-jest
- Run tests with `npx nx test <package-name>`
- Blueprint steps have comprehensive unit tests in `packages/playground/blueprints/src/lib/steps/`

## Important Constraints

### Node.js and npm Requirements
- Node.js 20.18.3 or higher required
- npm 10.1.0 or higher required
- Specified in `package.json` engines field

### Package Manager
- Uses npm 10.9.2 (specified in `packageManager` field)
- Workspaces defined for monorepo structure

### TypeScript Configuration
- Target: ES2021
- Module: esnext
- JSX: react
- Module resolution: bundler
- Path aliases defined in `tsconfig.base.json` for all packages

## PHP-WASM Core Classes

### `PHP` Class
Main class implementing Emscripten PHP runtime wrapper. Key responsibilities:
- PHP execution via `run()` method
- Filesystem operations (`writeFile`, `readFile`, `mkdir`, etc.)
- HTTP request handling via `PHPRequestHandler`
- Event dispatching for lifecycle hooks
- Resource cleanup via Disposable pattern

### `PHPRequestHandler`
HTTP request handler for WordPress-style requests:
- Processes HTTP requests through PHP
- Returns `PHPResponse` objects with headers, body, status codes
- Manages PHP superglobals ($_GET, $_POST, $_SERVER, etc.)

### `PHPProcessManager`
Manages multiple PHP process instances:
- Concurrency control with semaphore (default: 1, serializes requests)
- Request limit per runtime (default: 400)
- Automatic runtime rotation on errors
- Process pooling for multi-instance support

## Build System (Nx)

This project uses Nx for build orchestration:
- Caching enabled for `build`, `lint`, `test`, `e2e` targets
- Target dependencies: builds depend on `^build` (dependencies build first)
- Workspace layout: both apps and libs in `packages/`
- Named inputs for production builds exclude test files

### Nx Commands
```bash
# Run command for all packages
npx nx run-many --all --target=<command>

# Run for specific package
npx nx <command> <package-name>

# Clear Nx cache
npm run reset
npx nx reset
```

## Documentation

### Generating Documentation
```bash
npm run build:docs
```

Documentation site built with Docusaurus and TypeDoc API generation.

### Documentation File Structure
- English documentation: `packages/docs/site/docs/`
- Translations: `packages/docs/site/i18n/<LANGUAGE_CODE>/docusaurus-plugin-content-docs/current/`
- Configuration: `packages/docs/site/docusaurus.config.js`

### Updating English Documentation

When updating English documentation files:

1. **Edit documentation files** in `packages/docs/site/docs/`
2. **Test locally** with the development server:
   ```bash
   npm run dev:docs
   # or from packages/docs/site directory:
   npm run dev
   ```
3. **Build documentation** to verify no errors:
   ```bash
   npm run build:docs
   ```
4. **Update translations** - After updating English docs, corresponding translations (Spanish and Portuguese) should be updated. See "Updating Existing Translations" below.
5. **Submit PR** with clear description of documentation changes

### Working with Translations

#### File Structure for Translations
Translation files must mirror the English documentation structure. For example:
- English: `packages/docs/site/docs/main/intro.md`
- Spanish: `packages/docs/site/i18n/es/docusaurus-plugin-content-docs/current/main/intro.md`
- Portuguese: `packages/docs/site/i18n/pt-br/docusaurus-plugin-content-docs/current/main/intro.md`

#### Setting Up a New Language

1. **Add language to configuration** in `packages/docs/site/docusaurus.config.js`
2. **Generate UI translation files** (run from `packages/docs/site` directory):
   ```bash
   npm run write-translations -- --locale <LANGUAGE_CODE>
   ```
   Examples:
   ```bash
   npm run write-translations -- --locale es      # Spanish
   npm run write-translations -- --locale pt-br   # Portuguese (Brazil)
   npm run write-translations -- --locale ja      # Japanese
   ```
3. **Create tracking issue** on GitHub repository (model after issue #2202)

#### Creating New Translations

1. **Check existing issues** for language-specific tracking issues
2. **Copy the English .md file** to the appropriate language directory:
   ```bash
   # Example for Spanish
   mkdir -p packages/docs/site/i18n/es/docusaurus-plugin-content-docs/current/main/
   cp packages/docs/site/docs/main/intro.md \
      packages/docs/site/i18n/es/docusaurus-plugin-content-docs/current/main/intro.md
   ```
3. **Keep English content as HTML comments** for reviewer context:
   ```markdown
   <!-- ## Original English Title -->
   ## Título Traducido

   <!-- This is the original English paragraph. -->
   Este es el párrafo traducido.
   ```
4. **Test the translation locally**:
   ```bash
   cd packages/docs/site
   npm run dev -- --locale es
   # or
   npm run dev -- --locale pt-br
   ```
5. **Verify the build**:
   ```bash
   npm run build:docs
   ```
6. **Submit PR** with `[i18n]` prefix in title (e.g., "[i18n] Add Spanish translation for intro.md")

#### Updating Existing Translations

When English documentation changes, translations need updating:

1. **Identify changed files** by reviewing the English documentation diff
2. **Update translation files** in `packages/docs/site/i18n/<LANGUAGE_CODE>/`
3. **Update HTML comments** with new English content for reviewer context
4. **Test locally** with language-specific dev server:
   ```bash
   cd packages/docs/site
   npm run dev -- --locale es      # Spanish
   npm run dev -- --locale pt-br   # Portuguese
   ```
5. **Submit PR** with `[i18n]` prefix and specify language (e.g., "[i18n][es] Update Spanish translations")

#### Reviewing Translations

1. **Verify original English context** is preserved in HTML comments
2. **Check translation accuracy** against English source
3. **Test locally** using the language-specific preview
4. **Request feedback** via Make WordPress Polyglots platform with locale tag (e.g., #es, #pt-br, #ja)
5. **Verify build passes**:
   ```bash
   npm run build:docs
   ```

#### Translation Workflow for Spanish and Portuguese

For Spanish (es) translations:
```bash
# Test Spanish translations
cd packages/docs/site
npm run dev -- --locale es

# Location: packages/docs/site/i18n/es/docusaurus-plugin-content-docs/current/
```

For Portuguese (pt-br) translations:
```bash
# Test Portuguese translations
cd packages/docs/site
npm run dev -- --locale pt-br

# Location: packages/docs/site/i18n/pt-br/docusaurus-plugin-content-docs/current/
```

#### GitHub Web Interface for Translations

Contributors without local development setup can edit via GitHub:

1. **Navigate to the translation file** in the repository
2. **Click the pencil icon** to edit existing files
3. **Use "Add file" > "Create new file"** for new translations
   - Type full path: `packages/docs/site/i18n/es/docusaurus-plugin-content-docs/current/main/intro.md`
   - GitHub creates folders automatically when using `/` in filename
4. **Submit PR** directly from GitHub interface

#### Language Switcher Visibility

A language appears in the public language switcher only when the entire Documentation hub is translated, including:
- Quick Start Guide
- Web instance overview
- About section
- All Guides
- Contributing section
- Resources

### Documentation Resources
- Translation guide: https://wordpress.github.io/wordpress-playground/contributing/translations
- Code contributions: https://wordpress.github.io/wordpress-playground/docs/contributing/code
- Documentation guide: https://wordpress.github.io/wordpress-playground/docs/contributing/documentation
- Make WordPress Polyglots: https://make.wordpress.org/polyglots/
- Slack channel: #playground at Make WordPress

## Offline Support Testing

Build for offline mode:
```bash
PLAYGROUND_URL=http://localhost:9999 npx nx run playground-website:build:wasm-wordpress-net
```

Serve locally:
```bash
php -S localhost:9999 -t dist/packages/playground/wasm-wordpress-net
# or with Docker
docker run --rm -p 9999:80 -v $(pwd)/dist/packages/playground/wasm-wordpress-net:/usr/share/nginx/html:ro nginx:alpine
```

Test in browser: Open DevTools → Network tab → Select "Offline" → Refresh

## Package Publishing

Managed via Lerna:
```bash
npm run release
```

This publishes patch versions of all non-private packages to npm.

## Key File Locations

- Nx configuration: `nx.json`
- TypeScript paths: `tsconfig.base.json`
- Root package config: `package.json`
- Jest config: `jest.config.ts` and `jest.preset.js`
- Lerna config: `lerna.json`
- ESLint: `.eslintrc.json` and `.eslintignore`
- Prettier: `.prettierrc` and `.prettierignore`
- Git submodule: `isomorphic-git/` (forked dependency)

## Git Workflow

Main branch: `trunk`
Current branch shown in git status output
Commits follow conventional format (see recent commits for style)

## Common Gotchas

1. **Memory Management**: Always use Disposable pattern for PHP instances
2. **Concurrency**: PHP runtime is single-threaded by default (semaphore = 1)
3. **Runtime Rotation**: PHP instances recreate after 400 requests or errors
4. **Blueprint Resources**: Resources must specify correct descriptor type
5. **Service Worker**: Required for request interception in browser
6. **Path Aliases**: Use TypeScript path aliases from `tsconfig.base.json`
7. **Nx Cache**: Clear with `npm run reset` if builds seem stale
8. **Symlinks**: Git submodule for `isomorphic-git` must be initialized
