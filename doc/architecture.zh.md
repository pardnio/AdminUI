# AdminUI - 架構

最後更新：2026-10-25

> 返回 [README](./README.zh.md)

## 概覽

```mermaid
graph TB
    subgraph BUILD[建置]
        SASS[src/sass] -->|sass| CSS[dist/AdminUI.css]
        JS[src/js] -->|terser| DIST[dist/AdminUI.js]
    end
    subgraph BROWSER[瀏覽器]
        PAGE[HTML 頁面] --> QUI[QuickUI]
        QUI --> EV[system_events / textInput]
        PAGE --> EXT[NanoMD / NanoJSON / FlexPlyr]
    end
    CSS --> PAGE
    DIST --> PAGE
    subgraph STATIC[靜態範例]
        COMP[component/*.html] -->|temp :path| PAGE
        DEMO[static/js/demo.js<br>demo_datas / demo_events] --> PAGE
    end
```
```

## 模組：樣式（src/sass）

`AdminUI.scss` 為唯一入口，在 `body` 範圍內匯入預設版面與登入樣式，編譯為 `dist/AdminUI.css`。

```mermaid
graph TB
    subgraph SRCSASS[src/sass]
        ENTRY[AdminUI.scss] --> FN[function/_hsbTohsl.scss<br>hsb / hsba]
        ENTRY --> INIT[_init.scss<br>色彩與間距變數]
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

## 模組：腳本（src/js）

`src/js/*.js` 與 `src/js/**/*.js` 由 terser 合併為單一檔案，所有函式皆為全域。

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

## 模組：靜態範例（index.html、demo-*.html、component/、static/）

不需後端即可瀏覽的展示頁，以 QuickUI `<temp :path>` 載入共用元件，`static/js/demo.js` 提供假資料與 cookie 登入。

```mermaid
graph TB
    subgraph STATIC[靜態範例]
        IDX[index.html] --> LOGINC[component/demo-login.html]
        IDX --> LEFTC[component/demo-left-tab.html]
        IDX --> NAVC[component/demo-right-nav.html]
        LEFTC --> PG[demo-*.html]
        DEMOJS[static/js/demo.js<br>demo_datas / demo_events]
    end
    DEMOJS -->|is_auth cookie| IDX
    DEMOJS --> PG
    PG -->|NanoMD / NanoJSON / FlexPlyr| VIEW[Markdown / JSON / 影音檢視]
```

## 資料流

### 靜態範例登入

```mermaid
sequenceDiagram
    participant U as 使用者
    participant B as 瀏覽器
    participant Q as QuickUI
    U->>B: 開啟 index.html
    B->>B: getCookie("is_auth") 決定 is_guest
    Q->>B: 渲染 component/demo-login.html
    U->>Q: 點擊登入（auth_login）
    Q->>B: addCookie("is_auth", 1)
    B->>B: 500 ms 後 location.reload()
    Q->>B: 渲染側邊欄、導覽列與內容
    U->>Q: 點擊登出（logout）
    Q->>B: addCookie("is_auth", 0) 並重新整理
```

## 狀態機

### 側邊欄

```mermaid
stateDiagram-v2
    state "完整" as Full
    state "窄版" as Min
    state "行動版" as Mobile {
        state "隱藏" as Hidden
        state "顯示" as Shown
        Hidden --> Shown: body_left_show
        Shown --> Hidden: body_left_show
    }
    [*] --> Full: is_body_left_min = 0
    [*] --> Min: is_body_left_min = 1
    Full --> Min: body_left_type（寫入 cookie）
    Min --> Full: body_left_type（寫入 cookie）
```

***

©️ 2024 [邱敬幃 Pardn Chiu](https://www.linkedin.com/in/pardnchiu)
