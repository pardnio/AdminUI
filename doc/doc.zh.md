# AdminUI - 技術文件

最後更新：2026-10-25

> 返回 [README](./README.zh.md)

## 前置需求

- 瀏覽器端相依（以 CDN 載入）：
  - [QuickUI](https://github.com/pardnchiu/QuickUI) `0.6.6`（必要，負責 `:if`、`@click`、`<temp :path>` 等模板語法）
  - [Font Awesome](https://fontawesome.com) `6.5.2`（必要，所有圖示）
  - [NanoMD](https://github.com/pardnchiu/NanoMD) `1.11.1`、[NanoJSON](https://github.com/pardnchiu/NanoJSON) `0.3.4`、[FlexPlyr](https://github.com/pardnchiu/FlexPlyr) `2.2.7`（選用，Markdown／JSON／影音頁面）
- 從原始碼建置：Node.js 與 npm（devDependencies：`sass` 1.62.0、`terser` 5.31.3、`chokidar-cli` 3.0.0、`npm-run-all` ^4.1.5）
- 靜態範例：任一 HTTP 靜態伺服器（如 `python3 -m http.server`）

## 安裝

### 透過 CDN

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@pardnchiu/adminui@0.4.3/dist/AdminUI.css">
<script src="https://cdn.jsdelivr.net/npm/@pardnchiu/adminui@0.4.3/dist/AdminUI.js"></script>
```

### 透過 npm

```bash
npm install @pardnchiu/adminui
```

發佈的套件包含 `dist/`、`LICENSE` 與 `README.md`。

### 從原始碼

```bash
git clone https://github.com/pardnchiu/AdminUI.git
cd AdminUI
npm install
# 編譯 JS：dist/AdminUI.debug.js 與 dist/AdminUI.js
npm run build:debug
npm run build:min
# 編譯 SCSS（監看模式）：dist/AdminUI.css
npm run sass:dist
```

## 使用方式

### 基礎：套用版面

```html
<!DOCTYPE html>
<html lang="zh-hant">
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
        <!-- 左側導覽列 -->
        <section class="system-block-body-left" :data-min="is_body_left_min">
            <header><strong>{{ title }}</strong></header>
            <button class="mobile" @click="body_left_show">
                <i class="fa-solid fa-chevron-left"></i>
                <span>收合列表</span>
            </button>
            <div>
                <p @click="tab_show">
                    <i class="fa-solid fa-newspaper"></i>
                    <span>資料管理</span>
                    <i class="fa-solid fa-chevron-right"></i>
                </p>
                <section data-row="1">
                    <a href="/datas"><i class="fa-solid fa-table-list"></i><span>資料列表</span></a>
                </section>
            </div>
        </section>
        <!-- 右側內容 -->
        <section class="system-block-body-right">
            <nav>
                <section><button @click="body_left_show"><i class="fa-solid fa-bars-staggered"></i></button></section>
                <strong>{{ title }}</strong>
                <section></section>
            </nav>
            <header><strong>入口頁</strong></header>
        </section>
    </section>
    <script>
        document.addEventListener("DOMContentLoaded", _ => {
            // page 為 AdminUI.js 宣告的全域變數
            page = new QUI({
                id: "body",
                data: {
                    title: "後台",
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

`body_left_type` 會把側邊欄收合狀態寫入 cookie `is_body_left_min`，重新整理後以 `getCookie` 還原。

### 進階：輸入驗證表單

`textInput` 會把錯誤狀態寫在 input 的祖父元素（`label`）上的 `data-error`／`data-message`；`textarea` 後方的 `<pre>` 會同步內容以撐開高度。

```html
<form id="input-form" class="system-component-form">
    <label title="帳號" data-hint="僅接受半形英文 / 數字 / 底線">
        <section>
            <input name="account" data-type="account" data-force="0" data-min="8" data-max="24" @input="textInput">
        </section>
    </label>
    <label title="數字" data-hint="-10 ~ 30">
        <section>
            <input name="number" data-type="number" data-force="0" data-min="-10" data-max="30" @input="textInput">
        </section>
    </label>
    <label title="單行輸入">
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
            // 有任一欄位錯誤時阻止送出
            if (e.target.querySelector(`*[data-error="1"]`) != null) {
                alert("請檢查欄位是否正確.");
                return;
            };
        });
    });
</script>
```

### 進階：列表篩選條件編碼

列表頁可把篩選條件以 URL-safe Base64 放入 `?filter=`，後端以 URL-safe Base64 解碼後 `JSON.parse` 還原。

```js
const filter = [
    { key: "single", compare: "LIKE", value: "%demo%" }
];
// encodeToSafeBase64 內部會先 JSON.stringify
location.href = "/datas?filter=" + encodeToSafeBase64(filter);
```

### 靜態範例

範例頁位於專案根目錄，以相對路徑載入 `./dist`、`./static` 與 `./component`；QuickUI 的 `<temp :path>` 需透過 HTTP 取得元件，因此需在專案根目錄啟動任一靜態伺服器：

```bash
cd AdminUI
python3 -m http.server 8000
# 開啟 http://localhost:8000/
```

靜態範例的登入只會寫入 cookie `is_auth=1`，不驗證密碼；頁面位於專案根目錄（`demo-data-list.html`、`demo-data-editor.html`、`demo-article-editor.html`、`demo-folder.html`、`demo-media.html`、`demo-md-viewer.html`、`demo-text.html`、`demo-json.html`），共用元件位於 `component/`（`demo-login.html`、`demo-left-tab.html`、`demo-right-nav.html`）。線上版本：[demo-admin.pardn.io](https://demo-admin.pardn.io)。

## API 參考

### 全域函式（`dist/AdminUI.js`）

| 函式 | 簽章 | 說明 |
|------|------|------|
| `addCookie` | `addCookie(targetKey = "", body, expireSec)` | 寫入 cookie（`path=/`）；物件會先 `JSON.stringify`；`expireSec` 預設 3600；回傳 `document.cookie` |
| `getCookie` | `getCookie(targetKey = "")` | 讀取 cookie，可解析為 JSON 時回傳物件，否則回傳字串；不存在回傳 `undefined` |
| `addEvent` | `addEvent(config = {})` | `config.dom` 為目標元素，其餘鍵值（如 `onclick`）指派為事件處理器；回傳該元素 |
| `createElement` | `createElement(tag = "", val0, val1)` | 以 `tag#id.class` 語法建立元素（`temp` 建立 `DocumentFragment`）；參數可為屬性物件（含 `dataset`）、字串／數字或子元素陣列 |
| `encodeToSafeBase64` | `encodeToSafeBase64(obj)` | `JSON.stringify` 後以 UTF-8 編為 URL-safe Base64（`-`、`_`，去除 `=`） |
| `decodeFromSafeBase64` | `decodeFromSafeBase64(base64)` | 反向解碼並 `JSON.parse` |
| `textInput` | `textInput(e)` | `input` 事件處理器，依 `data-*` 屬性過濾與驗證輸入 |
| `hasgtagKeyup` | `hasgtagKeyup(e)` | 按下 Enter 時把以 `#`／`＃`／逗號／空白分隔的標籤正規化為 `#a #b #`（每個 2–24 字、去重） |

### `system_events`

傳入 QuickUI `event` 使用。

| 事件 | 說明 |
|------|------|
| `password_show` | 切換相鄰密碼欄位的 `type`（`password`／`text`）與眼睛圖示 |
| `body_left_show` | 切換 `section.system-block-body-left` 的 `data-show`（行動版展開／收合） |
| `body_left_type` | 切換側邊欄 `data-min`（窄版／完整），並寫入 cookie `is_body_left_min` |
| `tab_show` | 展開點擊的群組並收合其他已展開群組 |

### `textInput` 屬性

| 屬性 | 值 | 說明 |
|------|----|------|
| `data-type` | `text`、`account`、`number`、`email`、`password` | 驗證類型；未設定時取 `type`，再預設 `text` |
| `data-wrap` | `1`（預設）、`0` | `text` 類型是否允許換行；`0` 時移除換行並短暫標示錯誤 |
| `data-force` | `1`（預設）、`0` | `account`／`number`／`password` 是否直接移除不合法字元 |
| `data-max`／`data-min` | 數字 | `number` 類型為數值範圍；其他類型為字元長度 |

### 版面 class

| Class | 用途 |
|-------|------|
| `system-block-body-left` | 左側導覽列（`data-min`、`data-show`） |
| `system-block-body-right` | 右側內容區（含 `nav`、`header`） |
| `system-component-form` | 表單欄位排版 |
| `system-component-folder-row` | 資料夾列 |
| `system-page-data-list`／`system-page-data-editor` | 資料列表／資料編輯 |
| `system-page-article`／`system-page-article-editor`／`system-page-article-viewer` | 文章頁面 |
| `system-page-folder`／`system-page-media`／`system-page-text` | 檔案管理、媒體、文字頁面 |
| `login` | 登入區塊（`body > section.login`） |

### npm scripts

| 指令 | 說明 |
|------|------|
| `npm run build:debug` | 合併未壓縮 JS 至 `dist/AdminUI.debug.js` |
| `npm run build:min` | 以 `terser.config.json` 壓縮至 `dist/AdminUI.js` |
| `npm run sass:dist` | 監看並編譯 SCSS 至 `dist/` |

***

©️ 2024 [邱敬幃 Pardn Chiu](https://www.linkedin.com/in/pardnchiu)
