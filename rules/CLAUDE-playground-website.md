# CLAUDE.md - WordPress Playground Website Package

This file provides guidance to Claude Code when working specifically with the **@wp-playground/website** package.

For general project information, see the [root CLAUDE.md](/Users/fellyph/Sites/wordpress-playground/CLAUDE.md).

## Package Overview

The WordPress Playground Website (`@wp-playground/website`) is the main SPA hosted at https://playground.wordpress.net/. It provides the browser-based UI for running WordPress instances entirely in WebAssembly.

**Package Name**: `@wp-playground/website`
**Location**: `packages/playground/website/`
**Build Tool**: Vite
**Framework**: React 18 + Redux Toolkit
**Styling**: CSS Modules
**Components**: WordPress Components (`@wordpress/components`)

## Coding Guidelines

This package follows the [WordPress Gutenberg Coding Guidelines](https://developer.wordpress.org/block-editor/contributors/code/coding-guidelines/) and [WordPress Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/).

### JavaScript/TypeScript Standards

#### Module Organization & Imports

Organize imports into **three categories** with clear separation:

```typescript
/**
 * External dependencies
 */
import { useState, useEffect } from 'react';
import saveAs from 'file-saver';

/**
 * WordPress dependencies
 */
import { Button, TextControl } from '@wordpress/components';
import { __ } from '@wordpress/i18n';

/**
 * Internal dependencies
 */
import { usePlaygroundClient } from '../../lib/use-playground-client';
import { useI18n } from '../../lib/i18n';
import css from './style.module.css';
```

**Categories**:
1. **External dependencies** - Third-party packages from npm
2. **WordPress dependencies** - `@wordpress/*` packages
3. **Internal dependencies** - Relative imports from this package

#### Language Features

- Target **ECMAScript Stage 4** ("Finished") proposals only
- Use modern syntax enabled by `@wordpress/babel-preset-default`
- Support **JSX** syntax for React components
- Use **TypeScript** for type safety

#### Code Formatting

```typescript
// ✅ Good: Clear, typed component
interface SaveButtonProps {
	onSave: () => void;
	disabled?: boolean;
}

export function SaveButton({ onSave, disabled = false }: SaveButtonProps) {
	const { __ } = useI18n();

	return (
		<Button variant="primary" disabled={disabled} onClick={onSave}>
			{__('Save')}
		</Button>
	);
}

// ❌ Bad: Missing types, no i18n
export function SaveButton({ onSave, disabled }) {
	return <Button onClick={onSave}>Save</Button>;
}
```

#### Naming Conventions

**Files**:
- Components: `kebab-case` (e.g., `save-site-modal/index.tsx`)
- Utilities: `kebab-case` (e.g., `use-playground-client.ts`)
- CSS Modules: `*.module.css`
- Types: `types.d.ts` or inline in component files

**Variables/Functions**:
- `camelCase` for variables and functions
- `PascalCase` for React components and TypeScript types
- `UPPER_SNAKE_CASE` for constants

```typescript
// ✅ Good
const activeSite = useAppSelector(selectActiveSite);
const DEFAULT_LOCALE = 'en';
interface SiteMetadata { ... }
function MyComponent() { ... }

// ❌ Bad
const ActiveSite = useAppSelector(selectActiveSite);
const defaultLocale = 'en';
interface siteMetadata { ... }
function my_component() { ... }
```

### CSS/Styling Standards

#### Class Naming (CSS Modules)

CSS Modules automatically scope class names, but follow these conventions:

```css
/* style.module.css */

/* Root component class: componentName */
.saveModal {
	display: flex;
	flex-direction: column;
}

/* Child elements: componentName__descriptor */
.saveModal__header {
	font-weight: bold;
}

.saveModal__content {
	padding: 16px;
}

/* State modifiers: is-*, has-* (separate classes) */
.saveModal.isLoading {
	opacity: 0.6;
	pointer-events: none;
}

.saveModal__submitButton.isDisabled {
	cursor: not-allowed;
}
```

```typescript
// Usage in component
import css from './style.module.css';
import clsx from 'clsx';

function SaveModal({ isLoading }: { isLoading: boolean }) {
	return (
		<div className={clsx(css.saveModal, isLoading && css.isLoading)}>
			<h2 className={css.saveModal__header}>Save Playground</h2>
			<div className={css.saveModal__content}>...</div>
		</div>
	);
}
```

#### State Modifier Classes

Use **semantic prefixes** for state:
- `is-*` - Current state (e.g., `is-active`, `is-open`, `is-loading`)
- `has-*` - Content/feature presence (e.g., `has-error`, `has-warning`)

Apply as **separate classes** alongside component classes:

```typescript
// ✅ Good: Separate classes
<div className={clsx(css.modal, isOpen && css.isOpen)}>

// ❌ Bad: Combined class names
<div className={isOpen ? css.modalOpen : css.modalClosed}>
```

#### Component Isolation

**Never reference another component's class names externally**:

```typescript
// ❌ Bad: Referencing another component's styles
import modalCss from '../modal/style.module.css';
import buttonCss from '../button/style.module.css';

function MyComponent() {
	return <div className={modalCss.modal}>
		<button className={buttonCss.primary}>Click</button>
	</div>;
}

// ✅ Good: Render component instances
import { Modal } from '../modal';
import { Button } from '../button';

function MyComponent() {
	return (
		<Modal>
			<Button variant="primary">Click</Button>
		</Modal>
	);
}

// ✅ Good: Duplicate styles locally if needed
import css from './style.module.css';

function MyComponent() {
	return <div className={css.container}>
		<button className={css.primaryButton}>Click</button>
	</div>;
}
```

**Benefits**:
- Improves maintainability
- Reduces coupling
- Prevents cascading style changes
- Easier to refactor

### React Component Standards

#### Component Structure

```typescript
/**
 * External dependencies
 */
import { useState } from 'react';

/**
 * WordPress dependencies
 */
import { Button } from '@wordpress/components';
import { __ } from '@wordpress/i18n';

/**
 * Internal dependencies
 */
import { useI18n } from '../../lib/i18n';
import css from './style.module.css';

/**
 * Props interface with JSDoc
 */
interface SaveModalProps {
	/** Site slug to save */
	siteSlug: string;
	/** Callback when save completes */
	onSaved?: () => void;
	/** Whether the modal is disabled */
	disabled?: boolean;
}

/**
 * Modal for saving a playground site.
 *
 * @param {Object} props - Component props
 * @param {string} props.siteSlug - Site slug to save
 * @param {Function} [props.onSaved] - Callback when save completes
 * @param {boolean} [props.disabled=false] - Whether the modal is disabled
 * @returns {JSX.Element} Save modal component
 *
 * @example
 * ```tsx
 * <SaveModal
 *   siteSlug="my-site"
 *   onSaved={() => console.log('Saved!')}
 * />
 * ```
 */
export function SaveModal({
	siteSlug,
	onSaved,
	disabled = false,
}: SaveModalProps) {
	const { __ } = useI18n();
	const [isLoading, setIsLoading] = useState(false);

	const handleSave = async () => {
		setIsLoading(true);
		try {
			// Save logic
			onSaved?.();
		} finally {
			setIsLoading(false);
		}
	};

	return (
		<div className={css.saveModal}>
			<Button
				variant="primary"
				disabled={disabled || isLoading}
				onClick={handleSave}
			>
				{__('Save')}
			</Button>
		</div>
	);
}
```

#### Component Best Practices

**Export default function**:
```typescript
// ✅ Good: Default export
export default function SaveModal(props: SaveModalProps) { ... }

// ✅ Also good: Named export
export function SaveModal(props: SaveModalProps) { ... }
```

**Use TypeScript interfaces for props**:
```typescript
// ✅ Good
interface MyComponentProps {
	title: string;
	count?: number;
}

// ❌ Bad: Using 'any'
function MyComponent(props: any) { ... }
```

**Destructure props**:
```typescript
// ✅ Good
function MyComponent({ title, count = 0 }: MyComponentProps) {
	return <h1>{title}: {count}</h1>;
}

// ❌ Less readable
function MyComponent(props: MyComponentProps) {
	return <h1>{props.title}: {props.count}</h1>;
}
```

**Use React hooks properly**:
```typescript
// ✅ Good: Hooks at top level
function MyComponent() {
	const { __ } = useI18n();
	const [count, setCount] = useState(0);

	return <button onClick={() => setCount(count + 1)}>
		{__('Count')}: {count}
	</button>;
}

// ❌ Bad: Conditional hooks
function MyComponent({ showCount }: { showCount: boolean }) {
	if (showCount) {
		const [count, setCount] = useState(0); // ❌ Violates rules of hooks
	}
}
```

### TypeScript Standards

#### Type Definitions

```typescript
// ✅ Good: Clear type definitions
interface SiteMetadata {
	name: string;
	slug: string;
	storage: 'none' | 'opfs' | 'local-fs';
	createdAt?: Date;
}

type SiteStorageType = 'none' | 'opfs' | 'local-fs';

// Generic types
interface ApiResponse<T> {
	data: T;
	error?: string;
}

// Nullable types
function getSite(slug: string): SiteMetadata | null {
	// Implementation
}

// Void return
function logMessage(message: string): void {
	console.log(message);
}
```

#### Type Imports

Use `type` keyword for type-only imports:

```typescript
// ✅ Good: Type-only import
import type { PlaygroundClient } from '@wp-playground/client';
import type { SiteMetadata } from './types';

// ✅ Good: Mixed import
import { useState, type FC } from 'react';
```

### JSDoc Documentation

Document all exported functions and components:

```typescript
/**
 * Saves a playground site to storage.
 *
 * @param {string} siteSlug - The unique identifier for the site
 * @param {SiteStorageType} storageType - Where to save (opfs or local-fs)
 * @param {Object} options - Additional save options
 * @param {string} [options.siteName] - Custom name for the site
 * @param {FileSystemDirectoryHandle} [options.localFsHandle] - Directory handle for local-fs
 * @returns {Promise<void>} Promise that resolves when save completes
 * @throws {Error} If storage is not available or save fails
 *
 * @example
 * ```typescript
 * await saveSite('my-site', 'opfs', {
 *   siteName: 'My Test Site'
 * });
 * ```
 */
export async function saveSite(
	siteSlug: string,
	storageType: SiteStorageType,
	options: SaveOptions = {}
): Promise<void> {
	// Implementation
}
```

### API Design Patterns

#### Avoid Experimental/Unstable APIs

**Don't use** `__experimental` or `__unstable` prefixed APIs from WordPress packages:

```typescript
// ❌ Bad: Using unstable API
import { __experimentalFeature } from '@wordpress/components';

// ✅ Good: Use stable APIs only
import { Button, TextControl } from '@wordpress/components';
```

**Why**: These APIs can change or be removed without notice, creating maintenance burden.

#### Private vs Public APIs

Mark internal implementation details clearly:

```typescript
/**
 * Internal helper function. Do not use outside this module.
 * @private
 */
function _internalHelper() {
	// Implementation
}

/**
 * Public API for saving sites.
 * @public
 */
export function saveSite() {
	_internalHelper();
}
```

### Code Quality Checklist

Before submitting code, verify:

- [ ] **Imports organized** into three categories with comments
- [ ] **All user-facing text** wrapped in i18n functions (`__()`)
- [ ] **TypeScript types** defined for all props and function parameters
- [ ] **CSS classes** follow naming conventions (component__element, is-state)
- [ ] **Components isolated** - no cross-component style references
- [ ] **JSDoc comments** for all exported functions/components
- [ ] **No experimental APIs** from WordPress packages
- [ ] **Code formatted** with Prettier (runs via `npm run format`)
- [ ] **Type checks pass** (`npx nx typecheck playground-website`)
- [ ] **Linting passes** (`npx nx lint playground-website`)

### Prettier & ESLint

Code formatting is enforced by Prettier and ESLint:

```bash
# Format code
npm run format

# Format only uncommitted changes
npm run format:uncommitted

# Lint
npx nx lint playground-website
```

Configuration files:
- `.prettierrc` - Prettier configuration (root)
- `.eslintrc.json` - ESLint configuration (root)

### References

- [WordPress Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/)
- [Gutenberg Coding Guidelines](https://developer.wordpress.org/block-editor/contributors/code/coding-guidelines/)
- [WordPress JavaScript Documentation Standards](https://developer.wordpress.org/coding-standards/inline-documentation-standards/javascript/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)

## Directory Structure

```
packages/playground/website/
├── src/
│   ├── main.tsx                    # Application entry point
│   ├── components/                 # React components
│   │   ├── layout/                 # Main layout wrapper
│   │   ├── browser-chrome/         # Address bar and toolbar
│   │   ├── site-manager/           # Site management UI
│   │   ├── modal/                  # Reusable modal components
│   │   ├── missing-site-modal/
│   │   ├── rename-site-modal/
│   │   ├── save-site-modal/
│   │   ├── start-error-modal/
│   │   ├── toolbar-buttons/        # Menu items (8 components)
│   │   ├── import-form/
│   │   └── offline-notice/
│   ├── lib/                        # Utilities and libraries
│   │   ├── i18n/                   # Internationalization (see below)
│   │   ├── state/                  # Redux state management
│   │   │   ├── redux/              # Redux store and slices
│   │   │   ├── opfs/               # OPFS storage
│   │   │   └── url/                # URL routing
│   │   ├── hooks/                  # Custom React hooks
│   │   ├── use-playground-client.ts
│   │   ├── config.ts
│   │   └── tracking.ts
│   ├── styles.css                  # Global styles
│   └── forms.module.css            # Form styles
├── vite.config.ts                  # Vite configuration
├── tsconfig.json                   # TypeScript config
├── project.json                    # Nx project config
└── package.json                    # Package metadata
```

## State Management (Redux)

### Redux Store Structure

Location: `src/lib/state/redux/store.ts`

The app uses Redux Toolkit with three main slices:

```typescript
{
  ui: UIState,        // UI state (modals, active site, offline status)
  sites: SitesState,  // Site metadata and configuration
  clients: ClientsState // Playground client connections
}
```

### Redux Slices

**1. UI Slice** (`slice-ui.ts`)
- `activeSite` - Currently selected site
- `activeModal` - Which modal is open (if any)
- `offline` - Network connectivity status
- `siteManagerIsOpen` - Site manager visibility
- `siteManagerSection` - Active section in site manager

**2. Sites Slice** (`slice-sites.ts`)
- Entity adapter for site management
- Site metadata (name, slug, storage type, etc.)
- CRUD operations for sites

**3. Clients Slice** (`slice-clients.ts`)
- Playground client instances
- Connection status
- OPFS sync status

### Using Redux in Components

```typescript
import { useAppSelector, useAppDispatch } from '../../lib/state/redux/store';
import { setActiveModal } from '../../lib/state/redux/slice-ui';

function MyComponent() {
  const dispatch = useAppDispatch();
  const activeSite = useAppSelector((state) => state.ui.activeSite);

  const openModal = () => {
    dispatch(setActiveModal('SAVE_SITE'));
  };

  return <button onClick={openModal}>Save</button>;
}
```

## Internationalization (i18n)

### Overview

The website package uses `@wordpress/i18n` for internationalization with full React integration.

**Text Domain**: `playground-website`
**Supported Locales**: `en`, `es`, `pt-br`, `ja`
**Documentation**: [src/lib/i18n/README.md](src/lib/i18n/README.md)

### Quick Start

```typescript
import { useI18n } from '../../lib/i18n';

function MyComponent() {
  const { __ } = useI18n();

  return <h1>{__('Save Playground')}</h1>;
}
```

### Translation Functions

```typescript
const { __, _x, _n, sprintf, isRTL } = useI18n();

// Basic translation
__('Save')

// With context (for disambiguation)
_x('Save', 'button label')

// Plural forms
_n('1 file', '%d files', count)

// String formatting
sprintf(__('Saving %d / %d files...'), current, total)

// RTL detection
isRTL() // Returns true for RTL languages
```

### Adding New Translatable Strings

1. **Import the hook**:
   ```typescript
   import { useI18n } from '../../lib/i18n';
   ```

2. **Use in component**:
   ```typescript
   function MyComponent() {
     const { __ } = useI18n();
     return <button>{__('Click me')}</button>;
   }
   ```

3. **Add translation to locale files**:
   Edit `src/lib/i18n/locales/{locale}.json`:
   ```json
   {
     "Click me": ["Clique aqui"]  // pt-br
   }
   ```

### Translation Files

Location: `src/lib/i18n/locales/`

- `en.json` - English (default, no translations needed)
- `es.json` - Spanish
- `pt-br.json` - Brazilian Portuguese (✅ fully translated)
- `ja.json` - Japanese

### Testing Translations

In browser console:
```javascript
// Switch to Brazilian Portuguese
window.__setPlaygroundLocale('pt-br');

// Back to English
window.__setPlaygroundLocale('en');
```

### i18n Best Practices

✅ **DO**:
- Use `__()` for all user-facing text
- Use `_x()` when strings are ambiguous
- Use `sprintf()` for dynamic content
- Keep translation strings as complete sentences

❌ **DON'T**:
- Concatenate translated strings
- Use template literals for translations
- Split sentences into multiple calls
- Translate technical terms or brand names

## Components

### Modal Components

All modals use the shared `<Modal>` component wrapper and `<ModalButtons>` for consistent button layout.

**Modal Slugs** (defined in `components/layout/index.tsx`):
- `MISSING_SITE` - Site not found prompt
- `RENAME_SITE` - Rename playground
- `SAVE_SITE` - Save playground to storage
- `IMPORT_FORM` - Import from .zip
- `LOG` - View logs
- `ERROR_REPORT` - Report error
- `GITHUB_EXPORT` - Export to GitHub
- `GITHUB_IMPORT` - Import from GitHub
- `PREVIEW_PR_WP` - Preview WordPress PR
- `PREVIEW_PR_GUTENBERG` - Preview Gutenberg PR

**Opening a modal**:
```typescript
import { setActiveModal } from '../../lib/state/redux/slice-ui';
import { modalSlugs } from '../layout';

dispatch(setActiveModal(modalSlugs.SAVE_SITE));
```

**Closing a modal**:
```typescript
dispatch(setActiveModal(null));
```

### Component Patterns

#### Using WordPress Components

```typescript
import { Button, TextControl, Notice } from '@wordpress/components';

function MyComponent() {
  return (
    <>
      <TextControl
        label="Name"
        value={name}
        onChange={setName}
      />
      <Button variant="primary">Save</Button>
      <Notice status="warning">Warning message</Notice>
    </>
  );
}
```

#### CSS Modules

```typescript
import css from './style.module.css';

function MyComponent() {
  return <div className={css.myClass}>Content</div>;
}
```

#### Playground Client Hook

```typescript
import { usePlaygroundClient } from '../../lib/use-playground-client';

function MyComponent() {
  const playground = usePlaygroundClient();

  const handleClick = async () => {
    if (!playground) return;
    const response = await playground.run({ code: '<?php echo "Hi"; ?>' });
    console.log(response.text);
  };

  return <button onClick={handleClick}>Run PHP</button>;
}
```

## Development

### Starting Dev Server

```bash
# From repo root
npm run dev

# Or using Nx directly
npx nx dev playground-website

# Server runs at http://127.0.0.1:5400/
```

### Building

```bash
# Production build
npx nx build playground-website

# Output: dist/packages/playground/website/
```

### Type Checking

```bash
npx nx typecheck playground-website
```

### Testing

```bash
npx nx test playground-website
```

### Linting

```bash
npx nx lint playground-website
```

## Styling Guidelines

### Global Styles

Global styles are in `src/styles.css` and loaded in `main.tsx`.

### CSS Modules

Component-specific styles use CSS Modules (`.module.css`):

```css
/* my-component.module.css */
.container {
  display: flex;
  gap: 16px;
}

.button {
  padding: 8px 16px;
}
```

```typescript
// MyComponent.tsx
import css from './my-component.module.css';

function MyComponent() {
  return (
    <div className={css.container}>
      <button className={css.button}>Click</button>
    </div>
  );
}
```

### WordPress Component Styles

WordPress components come with their own styles. Don't override unless necessary.

## Storage

### OPFS (Origin Private File System)

Location: `src/lib/state/opfs/`

OPFS is used for persisting WordPress files in the browser:

- `opfs-site-storage.ts` - Main OPFS storage interface
- `opfs-directory-handle-storage.ts` - Directory handle abstraction
- `opfs-site-storage-worker-for-safari.ts` - Safari compatibility

### Local File System Access

The website supports saving to local directories using the File System Access API:

```typescript
const handle = await window.showDirectoryPicker();
// Save Playground files to local directory
```

Used in: `save-site-modal/index.tsx`

## URL Routing

Location: `src/lib/state/url/`

The website uses URL parameters for configuration:

- `?locale=pt-br` - Set locale
- Blueprint parameters (parsed in `resolve-blueprint-from-url.ts`)

Router hooks in `router-hooks.ts` provide access to URL state.

## Important Files

### Entry Point
- `src/main.tsx` - Application bootstrap
  - Initializes Redux Provider
  - Initializes I18nProvider
  - Renders root Layout

### Configuration
- `src/lib/config.ts` - App configuration constants
- `vite.config.ts` - Vite build configuration
- `tsconfig.json` - TypeScript compiler options

### State Management
- `src/lib/state/redux/store.ts` - Redux store
- `src/lib/state/redux/slice-ui.ts` - UI state slice
- `src/lib/state/redux/slice-sites.ts` - Sites state slice
- `src/lib/state/redux/slice-clients.ts` - Clients state slice

## Common Tasks

### Adding a New Modal

1. **Create modal component** in `src/components/my-modal/`:
   ```typescript
   import { Modal } from '../modal';
   import { useI18n } from '../../lib/i18n';

   export function MyModal() {
     const { __ } = useI18n();
     return (
       <Modal title={__('My Title')}>
         {/* Content */}
       </Modal>
     );
   }
   ```

2. **Add modal slug** to `components/layout/index.tsx`:
   ```typescript
   export const modalSlugs = {
     // ... existing
     MY_MODAL: 'MY_MODAL',
   };
   ```

3. **Register in Layout** component to render when active

4. **Open from component**:
   ```typescript
   dispatch(setActiveModal(modalSlugs.MY_MODAL));
   ```

### Adding a New Toolbar Button

1. **Create component** in `src/components/toolbar-buttons/`:
   ```typescript
   import { MenuItem } from '@wordpress/components';
   import { useI18n } from '../../lib/i18n';

   export function MyMenuItem({ onClose }: { onClose: () => void }) {
     const { __ } = useI18n();
     return (
       <MenuItem
         aria-label={__('My action description')}
         onClick={() => {
           // Action
           onClose();
         }}
       >
         {__('My Action')}
       </MenuItem>
     );
   }
   ```

2. **Add to toolbar** in the toolbar component

3. **Add translations** to locale files

### Adding a New Translation

See [Internationalization](#internationalization-i18n) section above.

## Debugging

### Redux DevTools

The Redux store is configured for Redux DevTools. Install the browser extension to inspect state.

### Locale Debugging

```javascript
// Browser console
window.__setPlaygroundLocale('pt-br');
```

### Playground Client Debugging

The Playground client emits events:

```typescript
playground.addEventListener('request.start', (event) => {
  console.log('Request started:', event);
});
```

## TypeScript

### Path Aliases

Configured in `tsconfig.json` via the root `tsconfig.base.json`:

```typescript
import { useI18n } from '@/lib/i18n'; // Won't work, use relative paths
import { useI18n } from '../../lib/i18n'; // Correct
```

### Type Imports

```typescript
import type { PlaygroundClient } from '@wp-playground/client';
import type { SiteMetadata } from '../../lib/state/redux/slice-sites';
```

## Dependencies

### Key Dependencies

**WordPress Packages**:
- `@wordpress/components` - UI components
- `@wordpress/i18n` - Internationalization
- `@wordpress/element` - React wrapper
- `@wordpress/data` - State management

**Playground Packages**:
- `@wp-playground/client` - Playground client API
- `@wp-playground/blueprints` - Blueprint system
- `@php-wasm/logger` - Logging

**UI/State**:
- `react` - UI library
- `react-redux` - Redux bindings
- `@reduxjs/toolkit` - Redux state management

**Utilities**:
- `file-saver` - File downloads
- `@zip.js/zip.js` - Zip file handling

## Build Configuration

### Vite Config

Location: `vite.config.ts`

Key features:
- React plugin with Fast Refresh
- TypeScript path resolution
- CSS Modules support
- Development server on port 5400

## Testing Strategy

- **Unit tests**: `.spec.ts` files alongside source
- **Framework**: Vitest
- **Location**: Co-located with source files in `src/`

## Contributing to This Package

When making changes to the website package:

1. **Run type checking**: `npx nx typecheck playground-website`
2. **Test locally**: `npm run dev`
3. **Add translations**: Update locale files for new strings
4. **Update this file**: Keep CLAUDE.md current with changes
5. **Follow patterns**: Use existing components as examples

## Resources

- [WordPress Components Storybook](https://wordpress.github.io/gutenberg/?path=/docs/components-introduction--docs)
- [Redux Toolkit Docs](https://redux-toolkit.js.org/)
- [Vite Documentation](https://vitejs.dev/)
- [WordPress i18n Package](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-i18n/)
- [Main Project Documentation](https://wordpress.github.io/wordpress-playground/)
