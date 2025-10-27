# Internationalization (i18n) for WordPress Playground Website

This directory contains the internationalization infrastructure for the WordPress Playground website, using the `@wordpress/i18n` package.

## Overview

The i18n system provides:
- Translation functions (`__`, `_x`, `_n`, `_nx`, `sprintf`)
- React hook (`useI18n`) for accessing translation functions
- Support for multiple locales (English, Spanish, Portuguese, Japanese)
- Automatic locale detection from browser settings, URL params, or localStorage
- Right-to-left (RTL) language support

## Architecture

### Core Files

- **[config.ts](./config.ts)** - Configuration constants and locale management
  - Text domain: `playground-website`
  - Supported locales
  - Locale metadata
  - Browser locale detection
  - LocalStorage persistence

- **[use-i18n.ts](./use-i18n.ts)** - React hook for translation functions
  - Provides `__()`, `_x()`, `_n()`, `_nx()`, `sprintf()`, `isRTL()`
  - All functions automatically scoped to the `playground-website` text domain

- **[provider.tsx](./provider.tsx)** - React provider component
  - Initializes i18n system
  - Loads locale data dynamically
  - Manages locale state

- **[index.ts](./index.ts)** - Main export file

- **[locales/](./locales/)** - Translation files in Jed format
  - `en.json` - English (default)
  - `es.json` - Spanish
  - `pt-br.json` - Portuguese (Brazil)
  - `ja.json` - Japanese

## Usage

### Basic Translation

```tsx
import { useI18n } from '@/lib/i18n';

function MyComponent() {
  const { __ } = useI18n();

  return <h1>{__('Save Playground')}</h1>;
}
```

### Translation with Context

Use `_x()` to disambiguate identical strings with different meanings:

```tsx
function MyComponent() {
  const { _x } = useI18n();

  // "Save" as in "download"
  const saveButton = _x('Save', 'button label');

  // "Save" as in "rescue"
  const saveAction = _x('Save', 'action verb');

  return <button>{saveButton}</button>;
}
```

### Plural Forms

```tsx
function FileCounter({ count }: { count: number }) {
  const { _n, sprintf } = useI18n();

  const text = sprintf(
    _n('Saving %d file...', 'Saving %d files...', count),
    count
  );

  return <div>{text}</div>;
}
```

### String Formatting

```tsx
function ProgressIndicator({ current, total }: { current: number; total: number }) {
  const { __, sprintf } = useI18n();

  return <p>{sprintf(__('Saving %d / %d files...'), current, total)}</p>;
}
```

### RTL Support

```tsx
function DirectionAwareComponent() {
  const { isRTL } = useI18n();

  return (
    <div style={{ direction: isRTL() ? 'rtl' : 'ltr' }}>
      Content
    </div>
  );
}
```

## Setup

The i18n system is initialized in [main.tsx](../../main.tsx):

```tsx
import { I18nProvider } from './lib/i18n';

root.render(
  <Provider store={store}>
    <I18nProvider>
      <YourApp />
    </I18nProvider>
  </Provider>
);
```

## Locale Management

### Locale Detection Priority

1. **URL parameter `locale`**: `?locale=es`
2. **URL parameter `language`**: `?language=pt_BR` (WordPress format)
3. **LocalStorage**: `playground-locale` key
4. **Browser preference**: `navigator.languages`
5. **Default**: English (`en`)

**Note**: The `language` parameter supports WordPress locale format (e.g., `pt_BR`, `es_ES`) and will be automatically normalized to our format (`pt-br`, `es`).

### Changing Locale

For debugging/testing, use the browser console:

```javascript
// Global function exposed by I18nProvider
window.__setPlaygroundLocale('es'); // Switch to Spanish
window.__setPlaygroundLocale('pt-br'); // Switch to Portuguese
window.__setPlaygroundLocale('ja'); // Switch to Japanese
window.__setPlaygroundLocale('en'); // Switch back to English
```

The selected locale is automatically saved to localStorage.

## Adding a New Language

1. **Add locale to supported list** in [config.ts](./config.ts):

```typescript
export const SUPPORTED_LOCALES = [
  'en',
  'es',
  'pt-br',
  'ja',
  'fr', // Add new locale
] as const;

export const LOCALE_METADATA: Record<SupportedLocale, { name: string; nativeName: string }> = {
  // ... existing locales
  fr: { name: 'French', nativeName: 'Français' },
};
```

2. **Create translation file** at `locales/fr.json`:

```json
{
  "domain": "playground-website",
  "locale_data": {
    "playground-website": {
      "": {
        "domain": "playground-website",
        "lang": "fr",
        "plural-forms": "nplurals=2; plural=(n > 1);"
      },
      "Save Playground": ["Enregistrer Playground"],
      "Cancel": ["Annuler"],
      "Saving %d / %d files...": ["Enregistrement de %d / %d fichiers..."]
    }
  }
}
```

3. **Test the new locale**:

```javascript
window.__setPlaygroundLocale('fr');
```

## Translation Workflow

### For Developers

When adding new user-facing strings:

1. Import the `useI18n` hook
2. Wrap all user-facing strings with `__()`
3. Use `_x()` if the string needs context
4. Use `_n()` for plurals
5. Use `sprintf()` for dynamic values

### For Translators

Translation files use the [Jed format](https://messageformat.github.io/Jed/):

```json
{
  "domain": "playground-website",
  "locale_data": {
    "playground-website": {
      "": {
        "domain": "playground-website",
        "lang": "LOCALE_CODE",
        "plural-forms": "PLURAL_EXPRESSION"
      },
      "English String": ["Translated String"],
      "String with context\u0004context": ["Translated with context"]
    }
  }
}
```

## Components Using i18n

The following components have been internationalized:

### Modal Components
- [missing-site-modal](../../components/missing-site-modal/index.tsx)
- [rename-site-modal](../../components/rename-site-modal/index.tsx)
- [save-site-modal](../../components/save-site-modal/index.tsx)
- [start-error-modal](../../components/start-error-modal/index.tsx)
- [modal-buttons](../../components/modal/modal-buttons.tsx)

### Toolbar Components
- [download-as-zip](../../components/toolbar-buttons/download-as-zip.tsx)
- [restore-from-zip](../../components/toolbar-buttons/restore-from-zip.tsx)
- [view-logs](../../components/toolbar-buttons/view-logs.tsx)
- [report-error](../../components/toolbar-buttons/report-error.tsx)
- [github-export-menu-item](../../components/toolbar-buttons/github-export-menu-item.tsx)
- [github-import-menu-item](../../components/toolbar-buttons/github-import-menu-item.tsx)
- [wordpress-pr-menu-item](../../components/toolbar-buttons/wordpress-pr-menu-item.tsx)
- [gutenberg-pr-menu-item](../../components/toolbar-buttons/gutenberg-pr-menu-item.tsx)

### Other UI Components
- [import-form](../../components/import-form/index.tsx)
- [offline-notice](../../components/offline-notice/index.tsx)

## Best Practices

### DO

✅ Always use translation functions for user-facing text
✅ Provide context with `_x()` when strings are ambiguous
✅ Use placeholders with `sprintf()` for dynamic content
✅ Keep translation strings complete sentences when possible
✅ Extract long text into separate variables for readability

### DON'T

❌ Don't concatenate translated strings
❌ Don't put HTML inside translation strings (use separate strings)
❌ Don't use template literals for translations (use `sprintf()`)
❌ Don't translate technical terms or brand names
❌ Don't split sentences into multiple translation calls

### Good Examples

```tsx
// ✅ Good: Complete sentence with sprintf
const { __, sprintf } = useI18n();
const message = sprintf(__('Saving %d / %d files...'), current, total);

// ✅ Good: Context for disambiguation
const saveButton = _x('Save', 'button label');
const saveAction = _x('Save', 'rescue action');

// ✅ Good: Proper plural handling
const fileText = sprintf(
  _n('1 file', '%d files', count),
  count
);
```

### Bad Examples

```tsx
// ❌ Bad: String concatenation
const message = __('Saving') + ' ' + current + ' / ' + total + ' ' + __('files');

// ❌ Bad: Template literal
const message = __(`Saving ${current} / ${total} files...`);

// ❌ Bad: Split sentence
<p>{__('Click')} <a href="#">{__('here')}</a> {__('to continue')}</p>
```

## Testing

### Manual Testing

1. Start the development server:
   ```bash
   npm run dev
   ```

2. Open browser console and switch locale:
   ```javascript
   window.__setPlaygroundLocale('es');
   ```

3. Verify UI updates with translated strings

### Automated Testing

Currently no automated tests for i18n. Future considerations:
- Test locale detection logic
- Test translation function calls
- Test RTL rendering
- Validate translation files format

## Resources

- [WordPress i18n Package Documentation](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-i18n/)
- [Jed Format Specification](https://messageformat.github.io/Jed/)
- [WordPress Polyglots Handbook](https://make.wordpress.org/polyglots/handbook/)
- [CLDR Plural Rules](http://www.unicode.org/cldr/charts/43/supplemental/language_plural_rules.html)

## Future Enhancements

Potential improvements for the i18n system:

- [ ] Add language switcher UI component
- [ ] Integrate with Redux store for global locale state
- [ ] Add script to extract translatable strings
- [ ] Set up Crowdin or similar for community translations
- [ ] Add date/time formatting utilities
- [ ] Add number formatting utilities
- [ ] Support for translation interpolation with React components
- [ ] Add translation coverage reporting
