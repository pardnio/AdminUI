# AdminUI - Architecture

Last updated: 2026-10-25

> Back to [README](../README.md)

## Overview

```mermaid
graph TB
    subgraph BUILD[Build]
        SASS[src/sass] -->|sass| CSS[dist/AdminUI.css]
        JS[src/js] -->|terser| DIST[dist/AdminUI.js]
    end
    subgraph BROWSER[Browser]
        PAGE[HTML Pages] --> QUI[QuickUI]
        QUI --> EV[system_events / textInput]
        PAGE --> EXT[NanoMD / NanoJSON / FlexPlyr]
    end
    CSS --> PAGE
    DIST --> PAGE
    subgraph STATIC[Static Demo]
        COMP[component/*.html] -->|temp :path| PAGE
        DEMO[static/js/demo.js<br>demo_datas / demo_events] --> PAGE
    end
```
```

## Module: Styles (src/sass)

`AdminUI.scss` is the single entry point; it imports the default layout and login styles inside the `body` scope and compiles to `dist/AdminUI.css`.

```mermaid
graph TB
    subgraph SRCSASS[src/sass]
        ENTRY[AdminUI.scss] --> FN[function/_hsbTohsl.scss<br>hsb / hsba]
        ENTRY --> INIT[_init.scss<br>color and spacing variables]
        ENTRY --> DEF[default/_index.scss]
        ENTRY --> LOGIN[_login.scss]
        DEF --> LEFT[system-block-body-left]
        DEF --> RIGHT[system-block-body-right]
        RIGHT --> FORM[system-component-form]
        RIGHT --> ROW[system-component-folder-row]
        RIGHT --> PAGES[system-page-*<br>data-list / data-editor / article<br>folder / media / text]
    end
    FN --> INIT
    ENTRY -->|sass --style compressed| OUT[dist/AdminUI.css]
```

## Module: Scripts (src/js)

Terser concatenates `src/js/*.js` and `src/js/**/*.js` into one file, and every function is global.

```mermaid
graph TB
    subgraph SRCJS[src/js]
        INDEX[index.js<br>page / system_events]
        subgraph FUNC[function]
            COOKIE[addCookie / getCookie]
            EVENT[addEvent]
            CE[createElement]
            B64[encodeToSafeBase64 / decodeFromSafeBase64]
            TI[textInput / hasgtagKeyup]
        end
    end
    INDEX -->|body_left_type| COOKIE
    SRCJS -->|build:debug| D1[dist/AdminUI.debug.js]
    SRCJS -->|build:min| D2[dist/AdminUI.js]
    CFG[terser.config.json] --> D2
```

## Module: Static Demo (index.html, demo-*.html, component/, static/)

Backend-free showcase pages that load shared components through QuickUI `<temp :path>`; `static/js/demo.js` supplies mock data and cookie login.

```mermaid
graph TB
    subgraph STATIC[Static Demo]
        IDX[index.html] --> LOGINC[component/demo-login.html]
        IDX --> LEFTC[component/demo-left-tab.html]
        IDX --> NAVC[component/demo-right-nav.html]
        LEFTC --> PG[demo-*.html]
        DEMOJS[static/js/demo.js<br>demo_datas / demo_events]
    end
    DEMOJS -->|is_auth cookie| IDX
    DEMOJS --> PG
    PG -->|NanoMD / NanoJSON / FlexPlyr| VIEW[Markdown / JSON / Media Viewers]
```

## Data Flow

### Static Demo Login

```mermaid
sequenceDiagram
    participant U as User
    participant B as Browser
    participant Q as QuickUI
    U->>B: Open index.html
    B->>B: getCookie("is_auth") decides is_guest
    Q->>B: Render component/demo-login.html
    U->>Q: Click login (auth_login)
    Q->>B: addCookie("is_auth", 1)
    B->>B: location.reload() after 500 ms
    Q->>B: Render sidebar, nav, and content
    U->>Q: Click logout (logout)
    Q->>B: addCookie("is_auth", 0) and reload
```

## State Machine

### Sidebar

```mermaid
stateDiagram-v2
    state "Full" as Full
    state "Narrow" as Min
    state "Mobile" as Mobile {
        state "Hidden" as Hidden
        state "Shown" as Shown
        Hidden --> Shown: body_left_show
        Shown --> Hidden: body_left_show
    }
    [*] --> Full: is_body_left_min = 0
    [*] --> Min: is_body_left_min = 1
    Full --> Min: body_left_type (writes cookie)
    Min --> Full: body_left_type (writes cookie)
```

***

©️ 2024 [邱敬幃 Pardn Chiu](https://www.linkedin.com/in/pardnchiu)
