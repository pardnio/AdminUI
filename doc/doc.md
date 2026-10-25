# AdminUI - Documentation

Last updated: 2026-10-25

> Back to [README](../README.md)

## Prerequisites

- Browser dependencies (loaded from CDN):
  - [QuickUI](https://github.com/pardnchiu/QuickUI) `0.6.6` (required; provides the `:if`, `@click`, and `<temp :path>` template syntax)
  - [Font Awesome](https://fontawesome.com) `6.5.2` (required; all icons)
  - [NanoMD](https://github.com/pardnchiu/NanoMD) `1.11.1`, [NanoJSON](https://github.com/pardnchiu/NanoJSON) `0.3.4`, [FlexPlyr](https://github.com/pardnchiu/FlexPlyr) `2.2.7` (optional; Markdown, JSON, and media pages)
- Building from source: Node.js and npm (devDependencies: `sass` 1.62.0, `terser` 5.31.3, `chokidar-cli` 3.0.0, `npm-run-all` ^4.1.5)
- Static demo: any HTTP static server (such as `python3 -m http.server`)

## Installation

### Via CDN

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@pardnchiu/adminui@0.4.3/dist/AdminUI.css">
<script src="https://cdn.jsdelivr.net/npm/@pardnchiu/adminui@0.4.3/dist/AdminUI.js"></script>
```

### Via npm

```bash
npm install @pardnchiu/adminui
```

The published package contains `dist/`, `LICENSE`, and `README.md`.

### From Source

```bash
git clone https://github.com/pardnchiu/AdminUI.git
cd AdminUI
npm install
# Build JS: dist/AdminUI.debug.js and dist/AdminUI.js
npm run build:debug
npm run build:min
# Compile SCSS (watch mode): dist/AdminUI.css
npm run sass:dist
```

## Usage

### Basic: Apply the Layout

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@fortawesome/fontawesome-free@6.5.2/css/all.min.css">
    <script src="https://cdn.jsdelivr.net/npm/@pardnchiu/quickui@0.6.6/dist/QuickUI.js" defer></script>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@pardnchiu/adminui@0.4.3/dist/AdminUI.css">
    <script src="https://cdn.jsdelivr.net/npm/@pardnchiu/adminui@0.4.3/dist/AdminUI.js"></script>
</head>
<body id="body">
    <section>
        <!-- Left sidebar -->
        <section class="system-block-body-left" :data-min="is_body_left_min">
            <header><strong>{{ title }}</strong></header>
            <button class="mobile" @click="body_left_show">
                <i class="fa-solid fa-chevron-left"></i>
                <span>Collapse</span>
            </button>
            <div>
                <p @click="tab_show">
                    <i class="fa-solid fa-newspaper"></i>
                    <span>Data</span>
                    <i class="fa-solid fa-chevron-right"></i>
                </p>
                <section data-row="1">
                    <a href="/datas"><i class="fa-solid fa-table-list"></i><span>Data List</span></a>
                </section>
            </div>
        </section>
        <!-- Right content -->
        <section class="system-block-body-right">
            <nav>
                <section><button @click="body_left_show"><i class="fa-solid fa-bars-staggered"></i></button></section>
                <strong>{{ title }}</strong>
                <section></section>
            </nav>
            <header><strong>Home</strong></header>
        </section>
    </section>
    <script>
        document.addEventListener("DOMContentLoaded", _ => {
            // page is a global declared by AdminUI.js
            page = new QUI({
                id: "body",
                data: {
                    title: "Admin",
                    is_body_left_min: getCookie("is_body_left_min") || 0
                },
                event: {
                    ...system_events
                }
            });
        });
    </script>
</body>
</html>
```

`body_left_type` writes the sidebar state to the `is_body_left_min` cookie, and `getCookie` restores it after reload.

### Advanced: Validated Form

`textInput` writes the error state to `data-error` / `data-message` on the input's grandparent (`label`); the `<pre>` after a `textarea` mirrors its content to grow the height.

```html
<form id="input-form" class="system-component-form">
    <label title="Account" data-hint="Letters, digits, and underscores only">
        <section>
            <input name="account" data-type="account" data-force="0" data-min="8" data-max="24" @input="textInput">
        </section>
    </label>
    <label title="Number" data-hint="-10 ~ 30">
        <section>
            <input name="number" data-type="number" data-force="0" data-min="-10" data-max="30" @input="textInput">
        </section>
    </label>
    <label title="Single line">
        <section>
            <textarea name="single" data-type="text" data-wrap="0" @input="textInput"></textarea>
            <pre></pre>
        </section>
    </label>
</form>
<script>
    document.addEventListener("DOMContentLoaded", _ => {
        page = new QUI({
            id: "body",
            event: {
                ...system_events,
                textInput
            }
        });

        document.getElementById("input-form").addEventListener("submit", e => {
            e.preventDefault();
            // Block submission when any field has an error
            if (e.target.querySelector(`*[data-error="1"]`) != null) {
                alert("Please check the fields.");
                return;
            };
        });
    });
</script>
```

### Advanced: Encode List Filters

List pages can put filters into `?filter=` as URL-safe Base64; the backend restores them by decoding URL-safe Base64 and running `JSON.parse`.

```js
const filter = [
    { key: "single", compare: "LIKE", value: "%demo%" }
];
// encodeToSafeBase64 runs JSON.stringify internally
location.href = "/datas?filter=" + encodeToSafeBase64(filter);
```

### Static Demo

Demo pages live in the project root and load `./dist`, `./static`, and `./component` by relative path; QuickUI's `<temp :path>` fetches components over HTTP, so start any static server from the project root:

```bash
cd AdminUI
python3 -m http.server 8000
# Open http://localhost:8000/
```

The static login only sets the `is_auth=1` cookie without checking a password; pages live in the project root (`demo-data-list.html`, `demo-data-editor.html`, `demo-article-editor.html`, `demo-folder.html`, `demo-media.html`, `demo-md-viewer.html`, `demo-text.html`, `demo-json.html`), and shared components live in `component/` (`demo-login.html`, `demo-left-tab.html`, `demo-right-nav.html`). Live version: [demo-admin.pardn.io](https://demo-admin.pardn.io).

## API Reference

### Global Functions (`dist/AdminUI.js`)

| Function | Signature | Description |
|----------|-----------|-------------|
| `addCookie` | `addCookie(targetKey = "", body, expireSec)` | Writes a cookie (`path=/`); objects are `JSON.stringify`-ed; `expireSec` defaults to 3600; returns `document.cookie` |
| `getCookie` | `getCookie(targetKey = "")` | Reads a cookie, returning parsed JSON when possible, otherwise the string; returns `undefined` when missing |
| `addEvent` | `addEvent(config = {})` | Assigns every key except `dom` (such as `onclick`) as a handler on `config.dom`; returns the element |
| `createElement` | `createElement(tag = "", val0, val1)` | Creates an element from `tag#id.class` syntax (`temp` creates a `DocumentFragment`); arguments may be an attribute object (including `dataset`), a string or number, or an array of children |
| `encodeToSafeBase64` | `encodeToSafeBase64(obj)` | Encodes `JSON.stringify(obj)` as UTF-8 URL-safe Base64 (`-`, `_`, no `=`) |
| `decodeFromSafeBase64` | `decodeFromSafeBase64(base64)` | Decodes in reverse and runs `JSON.parse` |
| `textInput` | `textInput(e)` | `input` event handler that filters and validates by `data-*` attributes |
| `hasgtagKeyup` | `hasgtagKeyup(e)` | On Enter, normalizes tags separated by `#` / `＃` / commas / spaces into `#a #b #` (2-24 chars each, deduplicated) |

### `system_events`

Spread into the QuickUI `event` option.

| Event | Description |
|-------|-------------|
| `password_show` | Toggles the adjacent password field `type` (`password` / `text`) and the eye icon |
| `body_left_show` | Toggles `data-show` on `section.system-block-body-left` (mobile open / close) |
| `body_left_type` | Toggles the sidebar `data-min` (narrow / full) and writes the `is_body_left_min` cookie |
| `tab_show` | Expands the clicked group and collapses other open groups |

### `textInput` Attributes

| Attribute | Values | Description |
|-----------|--------|-------------|
| `data-type` | `text`, `account`, `number`, `email`, `password` | Validation type; falls back to `type`, then `text` |
| `data-wrap` | `1` (default), `0` | Whether `text` allows line breaks; `0` strips them and briefly flags an error |
| `data-force` | `1` (default), `0` | Whether `account` / `number` / `password` strip invalid characters immediately |
| `data-max` / `data-min` | number | Value range for `number`; character length for other types |

### Layout Classes

| Class | Purpose |
|-------|---------|
| `system-block-body-left` | Left sidebar (`data-min`, `data-show`) |
| `system-block-body-right` | Right content area (with `nav` and `header`) |
| `system-component-form` | Form field layout |
| `system-component-folder-row` | Folder row |
| `system-page-data-list` / `system-page-data-editor` | Data list / data editor |
| `system-page-article` / `system-page-article-editor` / `system-page-article-viewer` | Article pages |
| `system-page-folder` / `system-page-media` / `system-page-text` | File manager, media, and text pages |
| `login` | Login block (`body > section.login`) |

### npm Scripts

| Command | Description |
|---------|-------------|
| `npm run build:debug` | Concatenates unminified JS into `dist/AdminUI.debug.js` |
| `npm run build:min` | Minifies into `dist/AdminUI.js` with `terser.config.json` |
| `npm run sass:dist` | Watches and compiles SCSS into `dist/` |

***

©️ 2024 [邱敬幃 Pardn Chiu](https://www.linkedin.com/in/pardnchiu)
