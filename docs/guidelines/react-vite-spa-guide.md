# Vite + React 開發規範

## 目錄

* [Feature-First 架構](#feature-first-架構)
  * [目錄結構](#目錄結構)
  * [Routes Configuration](#routes-configuration)
  * [Component 抽離原則](#component-抽離原則)
  * [架構演進策略（避免 Over-design）](#架構演進策略避免-over-design)
  * [模組匯出規範 (Module Exporting)](#模組匯出規範-module-exporting)
* [開發工具配置 (Development Tooling)](#開發工具配置-development-tooling)
  * [路徑別名 (Path Alias) 規範](#路徑別名-path-alias-規範)
  * [完整的 jsconfig.json 模板](#完整的-jsconfigjson-模板)
  * [jsconfig 配置逐行詳解](#jsconfig-配置逐行詳解)

## Feature-First 架構

以模組為核心（Module-based Structure），並允許頁面擁有私有 Component（Page-specific Components）。

在需求不明確或專案快速迭代時，這套架構能兼顧「開發速度」與「擴展韌性」。

### 目錄結構

```plaintext
src/
├── components/            # Global 通用、基礎 Component (Button, Input)
├── pages/
│ ├── Product/             # Module
│ │ ├── components/        # Module 私有 Component (僅在 Module 內部 reuse)
│ │ │ ├── PriceTag.jsx
│ │ │ └── ProductCard.jsx
│ │ ├── List.jsx           # 頁面主體
│ │ ├── Detail.jsx
│ │ └── index.js           # Module's Entry Point
│ ├── Auth/
│ │ ├── ...
│ └── Home.jsx             # 獨立小型頁面 (未達模組化規模時)
```

### Routes Configuration

利用 React Router 的巢狀特性 (Nested Routes) 對應目錄結構。

App.jsx

```javascript
<Routes>
  {/* Product Module */}
  <Route path="/product">
    <Route index element={<ProductList />} />
    <Route path=":id" element={<ProductDetail />} />
  </Route>

  {/* Auth Module */}
  <Route path="/auth">
    <Route path="login" element={<Login />} />
    <Route path="register" element={<Register />} />
  </Route>
</Routes>
```

### Component 抽離原則

當你在編寫頁面發現某段 JSX 可以獨立時，遵循以下路徑進行抽離：

1. 若該 Component 僅在 `Product` 相關頁面出現，放在 `pages/Product/components/`。
2. 若該 Component 在不同 Module（如 `Product` 與 `Member`）皆有需求，提升至 `src/components/`。

補充：

- 按需加載 (Performance)：所有 Page 級別的 Component 一律使用 `React.lazy()` 導入，以確保 Vite 打包出最優的 Chunk。

### 架構演進策略 (避免 Over-design)

1. (開始) Flat：少於 5 個頁面時，全部平鋪在 `pages/` 下。
2. (成長) Module：單一功能超過 3 個頁面時，升級為資料夾管理。
3. (優化) Refactor：當頁面邏輯超過 300 行時，為了減少 Prop Drilling (層層傳遞參數)，強制抽離私有 Component。

### 模組匯出規範 (Module Exporting)

本專案採用 Barrel Export 模式，透過資料夾下的 `index.js` 統一對外入口，確保模組的高內聚與引用簡潔。

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

## 開發工具配置 (Development Tooling)

### 路徑別名 (Path Alias) 規範

為了避免深度嵌套導致的 `../../../../` 路徑地獄，本專案統一使用 `@` 作為 `src/` 目錄的別名。

Vite 配置範例 (vite.config.js)

```javascript
import { defineConfig } from "vite";
import path from "path";

export default defineConfig({
  resolve: {
    alias: {
      // 將 @ 映射到 src 資料夾
      "@": path.resolve(__dirname, "./src"),
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
      "@/*": ["src/*"]
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
      "@/*": ["src/*"]
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

    /* 6. paths: { "@/*": ["src/*"] } */
    "paths": {
      "@/*": ["src/*"]
    },
    // 解釋：最重要的部分！這讓編輯器知道 `@/` 代表 `src/`。
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
