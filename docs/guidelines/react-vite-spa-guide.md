# Vite + React 開發規範

## 目錄

- [Project Structure](#project-structure)
  - [Feature-First](#feature-first)
  - [Routes Configuration](#routes-configuration)
  - [Component 抽離原則](#component-抽離原則)
  - [架構演進策略 (避免 Over-design)](#架構演進策略-避免-over-design)
  - [模組匯出規範 (Module Exporting)](#模組匯出規範-module-exporting)
- [Code Style](#code-style)
  - [命名規範](#命名規範)
  - [寫法](#寫法)
- [開發工具配置 (Development Tooling)](#開發工具配置-development-tooling)
  - [路徑別名 (Path Alias) 規範](#路徑別名-path-alias-規範)
  - [完整的 jsconfig.json 模板](#完整的-jsconfigjson-模板)

## Project Structure

### Feature-First

為兼顧「開發速度」與「擴展韌性」，以模組為核心（Module-based Structure），並允許頁面擁有私有 Component（Page-specific Components）。

```plaintext
./
├── app/
│   ├── assets/              # 圖片、字型等靜態資源
│   ├── components/          # Global 通用、基礎 Component (Button, Input, Navbar)
│   ├── features/            # 核心功能模組 (取代 modules)
│   │   └── product/
│   │       ├── api/         # 該功能專屬 API
│   │       ├── components/  # 該功能專屬 UI
│   │       └── hooks/       # 該功能專屬邏輯
│   ├── hooks/               # Global Hooks
│   ├── i18n/
│   ├── styles/              # Global Styles (如 CSS Reset, Theme Variables)
│   ├── utils/               # Global Utils (如表單驗證規則)
│   ├── routes.js            # Routes
│   ├── root.jsx             # Data Mode 叫 App.jsx，負責全域 Provider/Layout
│   └── entry.client.jsx     # Data Mode 叫 main.jsx，但內容都只負責渲染 App
├── docs/                    # 專案相關文件、開發文件
├── index.html               # HTML 模板
├── public/                  # 靜態資源 (favicon, robots.txt)
├── tests/                   # 測試相關 (單元測試、整合測試)
├── .eslintrc.js             # ESLint 配置
├── .prettierrc              # Prettier 配置
├── jsconfig.json            # VS Code 路徑別名配置
├── package.json
└── vite.config.js           # Vite 配置
```

### Routes Configuration

無論使用 Data mode 或 Framework mode，路由定義統一放在 `app/routes.js`。

### Component 抽離原則

當你在編寫頁面發現某段 JSX 可以獨立時，遵循以下路徑進行抽離：

1. 若該 Component 僅在 `Product` 相關頁面出現，放在 `app/features/product/components/`。
2. 若該 Component 在不同 Module（如 `Product` 與 `Member`）皆有需求，提升至 `app/components/`。

補充：

- 按需加載 (Performance)：所有 Page 級別的 Component 一律使用 `React.lazy()` 導入，以確保 Vite 打包出最優的 Chunk。

### 架構演進策略 (避免 Over-design)

1. (開始) Flat：少於 5 個頁面時，全部平鋪在 `pages/` 下。
2. (成長) Module：單一功能超過 3 個頁面時，升級為資料夾管理。
3. (優化) Refactor：當頁面邏輯超過 300 行時，為了減少 Prop Drilling (層層傳遞參數)，強制抽離私有 Component。

### 模組匯出規範 (Module Exporting)

專案採用 Barrel Export 模式，透過資料夾下的 `index.js` 統一對外入口，確保模組的高內聚與引用簡潔。

#### 原則：`index.js` 僅作為轉運站 (Re-exporting)

- 不撰寫邏輯：避免在 `index.js` 撰寫業務邏輯、API 呼叫或複雜樣式。
- 清晰介面：僅負責將內部的 Page 或 Component 導出，讓開發者一眼看清模組對外暴露的功能。

#### 範例：`pages/Product/index.js`

```javascript
// 推薦做法：清晰的轉運站
export { default as ProductList } from "./List";
export { default as ProductDetail } from "./Detail";
export { default as ProductEdit } from "./Edit";
```

#### 引用對比

```javascript
// 不良示範 (路徑冗長且暴露內部結構)
import List from "@/pages/Product/List";
import Detail from "@/pages/Product/Detail";

// 推薦做法 (簡潔且具備封裝性)
import { ProductList, ProductDetail } from "@/pages/Product";
```

---

## Code Style

### 命名規範

| 對象       | 規範             | 範例                            |
| ---------- | ---------------- | ------------------------------- |
| Folder     | kebab-case       | app/user-profile/               |
| Component  | PascalCase       | UserProfile.jsx                 |
| Hook, Util | camelCase        | useFetchData.js, stringUtils.js |
| 常數, 設定 | UPPER_SNAKE_CASE | `export const API_URL = ""`     |

### 寫法

#### setState

```javascript
// 錯誤示範：直接賦值
// 這樣可能導致 state 更新不正確，因為 React 的 setState 是異步的，且可能會合併多次更新。
onChange={(e) => setBill(bill + 10)}

// 正確示範：一律使用 Functional Update 形式，確保 state 更新的正確性與一致性。
onChange={(e) => setBill(prevBill => prevBill + 10)}

// 補充：在某些情況下（如表單輸入），你可能需要從事件對象中取值。為了避免事件對象被釋放，先將值存起來再使用。
// 在 React 的舊版本或某些異步情況下，e.target 在回呼函式執行時可能已經被釋放（Nullified）。
// 雖然在現代 React 版本中這比較少見，但慣例上我們會先將值取出：
onChange={(e) => {
  const value = Number(e.target.value); // 先存起來
  setBill(prevBill => value < 0 ? prevBill : value);
}}
```

---

## 開發工具配置 (Development Tooling)

### 路徑別名 (Path Alias) 規範

為了避免深度嵌套導致的 `../../../../` 路徑地獄，專案統一使用 `@` 作為 `app/` 目錄的別名。

Vite 配置範例 (vite.config.js)

```javascript
import { defineConfig } from "vite";
import path from "path";

export default defineConfig({
  resolve: {
    alias: {
      // 將 @ 映射到 app 資料夾
      "@": path.resolve(__dirname, "./app"),
    },
  },
});
```

IDE Support 配置範例 (`jsconfig.json` 或 `tsconfig.json`)

- 用於確保 VS Code 的自動補全與點擊跳轉功能。

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["app/*"]
    }
  }
}
```

使用範例

```javascript
// 不良示範
// 1. 路徑脆弱：當檔案在 pages/ 深處移動位置時，需要重新計算相對路徑的 ../ 數量
// 2. 難以閱讀
import Button from "../../../components/Button";

// 推薦做法 (結構清晰、不易出錯)
import Button from "@/components/Button";
import { useAuth } from "@/pages/Auth/hooks/useAuth";
```

### 完整的 `jsconfig.json` 模板

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "node",
    "jsx": "react-jsx",
    "baseUrl": ".",
    "paths": {
      "@/*": ["app/*"]
    },
    "checkJs": true
  },
  "exclude": ["node_modules", "dist"]
}
```

逐行詳解

```json
{
  "compilerOptions": {
    /* 1. target: ESNext */
    "target": "ESNext",
    // 解釋：告訴編輯器你的 JS 環境支援最新的語法（如 Optional Chaining, Nullish coalescing）。
    // 不設定：VS Code 可能會在你使用最新語法時畫紅線，認為你的環境不支援。

    /* 2. module: ESNext */
    "module": "ESNext",
    // 解釋：指定模組系統為標準的 ESM (import/export)。
    // 不設定：編輯器可能無法正確追蹤檔案間的引用關係。

    /* 3. moduleResolution: node */
    "moduleResolution": "node",
    // 解釋：告訴編輯器「找檔案的路徑邏輯要跟 Node.js 一樣」（會去尋找 node_modules 或 index.js）。
    // 不設定：你 import 資料夾時，VS Code 可能找不到裡面的檔案，導致無法跳轉定義。

    /* 4. jsx: react-jsx */
    "jsx": "react-jsx",
    // 解釋：支援 React 17+ 的 JSX 轉換（不需要手動 import React）。
    // 不設定：VS Code 可能會提示「找不到 React 變數」，或者無法在 JSX 語法中提供補全。

    /* 5. baseUrl: . */
    "baseUrl": ".",
    // 解釋：設定所有相對路徑的根目錄為「當前專案目錄」。
    // 不設定：下方的 paths（別名）會失效，因為它不知道起始點在哪。

    /* 6. paths: { "@/*": ["app/*"] } */
    "paths": {
      "@/*": ["app/*"]
    },
    // 解釋：最重要的部分！這讓編輯器知道 `@/` 代表 `app/`。
    // 不設定：當你寫 `import '@/components/Button'`，VS Code 會報錯說「找不到模組」，
    // 且你按著 Cmd/Ctrl 點擊路徑時，無法跳轉到該檔案。

    /* 7. checkJs: true */
    "checkJs": true
    // 解釋：讓 VS Code 對 JS 進行靜態檢查（類似輕量級 TypeScript）。
    // 不設定：如果你寫錯變數名稱或傳錯參數類型，編輯器不會提醒你，要等到執行時才出錯。
  },

  /* 8. exclude */
  "exclude": ["node_modules", "dist"]
  // 解釋：告訴編輯器「不要去掃描這兩個資料夾」。
  // 不設定：VS Code 會花費大量 CPU 資源去索引上萬個第三方套件，導致電腦變燙、風扇狂轉、補全變超慢。
}
```
