[![npm version](https://img.shields.io/npm/v/@itrocks/asset-loader?logo=npm)](https://www.npmjs.org/package/@itrocks/asset-loader)
[![npm downloads](https://img.shields.io/npm/dm/@itrocks/asset-loader)](https://www.npmjs.org/package/@itrocks/asset-loader)
[![GitHub](https://img.shields.io/github/last-commit/itrocks-ts/asset-loader?color=2dba4e&label=commit&logo=github)](https://github.com/itrocks-ts/asset-loader)
[![issues](https://img.shields.io/github/issues/itrocks-ts/asset-loader)](https://github.com/itrocks-ts/asset-loader/issues)
[![discord](https://img.shields.io/discord/1314141024020467782?color=7289da&label=discord&logo=discord&logoColor=white)](https://25.re/ditr)

# asset-loader

Dynamically loads CSS and JavaScript files from your modules and appends them to the &lt;head&gt;.

*This documentation was written by an artificial intelligence and may contain errors or approximations.
It has not yet been fully reviewed by a human. If anything seems unclear or incomplete,
please feel free to contact the author of this package.*

## Installation

```bash
npm i @itrocks/asset-loader
```

This package is meant to be used in a browser environment. It expects
`document.head` to be available.

## Usage

`@itrocks/asset-loader` exposes two helpers:

- `loadCss` – injects a `<link rel="stylesheet">` pointing to your CSS
  file.
- `loadScript` – injects a `<script>` tag pointing to your JavaScript
  file.

Both helpers are idempotent: calling them multiple times with the same
`fileName` will **not** create duplicate `<link>` / `<script>` elements.
If the asset is already present, the optional `onLoad` callback is
called immediately with the existing element.

### Minimal example

```ts
import { loadCss, loadScript } from '@itrocks/asset-loader'

// Load a CSS file once
loadCss('/assets/styles/theme.css')

// Load a script and run some code when it is available
loadScript('/assets/vendor/chart.js', (script) => {
	// At this point the script has been loaded by the browser
	console.log('Script loaded from', script.src)
})
```

### Complete example with conditional loading

In a typical it.rocks front‑end you may want to load optional assets
only on some pages, while still avoiding duplicates when navigating.

```ts
import { loadCss, loadScript } from '@itrocks/asset-loader'

function enableCharts ()
{
	// Ensure chart styles are present (only added once, even if called
	// several times during navigation)
	loadCss('/assets/vendor/chart.css')

	// Load the script and initialize the library when ready
	loadScript('/assets/vendor/chart.js', () => {
		// The global `Chart` object is now available
		const canvas = document.querySelector<HTMLCanvasElement>('#sales')
		if (!canvas) return

		// Example chart rendering, depending on how your library works
		// (implementation details are up to your app)
		// new Chart(canvas, { ... })
	})
}

// You can call this whenever a page that needs charts becomes active
// (for example after routing or view activation)
```

## API

### `loadCss(fileName: string, onLoad?: (link: HTMLLinkElement) => void): void`

Injects a `<link rel="stylesheet">` element into `document.head` if it
does not already exist for the given `fileName`.

#### Parameters

- `fileName` – URL or path to the CSS file, as it should appear in the
  `href` attribute. It can be absolute (`https://...`) or relative to
  the current page.
- `onLoad` *(optional)* – callback invoked when the stylesheet has
  finished loading. It receives the created or existing
  `HTMLLinkElement`.

#### Behaviour

- If a `<link>` with `href === fileName` is already in `document.head`,
  no new element is created and `onLoad` (if provided) is called
  immediately with that element.
- Otherwise a new `<link>` is created, configured, appended to
  `document.head`, and `onLoad` is called when the browser fires the
  `load` event.

### `loadScript(fileName: string, onLoad?: (script: HTMLScriptElement) => void): void`

Injects a `<script>` element into `document.head` if it does not
already exist for the given `fileName`.

#### Parameters

- `fileName` – URL or path to the JavaScript file, as it should appear
  in the `src` attribute. It can be absolute or relative to the current
  page.
- `onLoad` *(optional)* – callback invoked when the script has finished
  loading. It receives the created or existing `HTMLScriptElement`.

#### Behaviour

- If a `<script>` with `src === fileName` is already in `document.head`,
  no new element is created and `onLoad` (if provided) is called
  immediately with that element.
- Otherwise a new `<script>` is created with `script.src = fileName`,
  appended to `document.head`, and `onLoad` is called when the browser
  fires the `load` event.

## Typical use cases

- Lazily loading third‑party libraries (charts, widgets, editors, etc.)
  only on the pages that need them.
- Injecting theme‑specific or feature‑specific stylesheets without
  bloating your main CSS bundle.
- Ensuring the same CSS or JS asset is not added multiple times when
  navigating in a single‑page application.
- Loading experimental or optional assets behind feature flags while
  keeping the loading logic simple and declarative.
