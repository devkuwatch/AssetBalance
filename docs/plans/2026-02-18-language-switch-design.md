# Language Switch Feature Design

**Date**: 2026-02-18
**Feature**: JA/EN language toggle in settings

## Requirements

- Language toggle button (JA/EN segment) placed in the header alongside the existing ⚙ button
- UI labels fully translated (section headings, buttons, placeholders, table headers, error messages, chart center text)
- Default asset type names (国内株, 海外先進国株, …) translated when in EN mode; user-customized names remain as-is
- Language preference persisted in `localStorage` (same `assetBalanceData` key)

## Architecture

### Translation Dictionaries

Two objects at the top of the script:

```js
const TRANSLATIONS = {
  ja: { appTitle: 'アセットバランス管理', ... },
  en: { appTitle: 'Asset Balance Manager', ... }
};

const ASSET_TYPE_MAP = {
  '国内株': 'Domestic Equity',
  '海外先進国株': 'Intl Developed Equity',
  '新興国株': 'Emerging Markets Equity',
  '国内債券': 'Domestic Bonds',
  '海外先進国債券': 'Intl Developed Bonds',
  '新興国債券': 'Emerging Markets Bonds',
  '海外REIT': 'Global REIT',
  '国内REIT': 'Domestic REIT',
  'コモディティ': 'Commodities'
};
```

### Helper Functions

- `t('key')` — returns `TRANSLATIONS[lang][key]`; falls back to key if missing
- `tAsset(name)` — returns `ASSET_TYPE_MAP[name]` when `lang === 'en'` and the name exists in the map; otherwise returns `name` unchanged

### Language State

- `let lang = 'ja'` module-level variable
- Loaded from `localStorage` on init; saved alongside funds/targetPcts

### UI

Header before: `[アセットバランス管理] [⚙]`
Header after:  `[アセットバランス管理] [JA|EN] [⚙]`

The JA/EN toggle uses the same `.rebalance-toggle` CSS pattern (segment button group).

### `applyLang()` Function

Called whenever language changes:

1. Update `<html lang="">` attribute
2. Update `document.title`
3. Query `[data-i18n]` elements → set `textContent` from `t(key)`
4. Query `[data-i18n-placeholder]` elements → set `placeholder` from `t(key)`
5. Call `renderAll()` to re-render dynamic content with new language

### Static HTML Changes

Add `data-i18n` / `data-i18n-placeholder` attributes to all static translatable elements:
- `<h1>`, section `<h2>` elements
- Button labels (追加, 削除, 計算, エクスポート, インポート, クリア)
- Table `<th>` elements (static ones only; dynamic headers rebuilt in JS)
- Input placeholders
- Empty state messages

### Dynamic Content

All JS template strings that produce HTML use `t()` and `tAsset()`:
- `renderFundTable()` — delete button, empty message
- `renderChart()` — chart center text "合計"/"Total", empty legend message
- `renderAllocTable()` — table headers per mode, notes, totals row
- `openSettings()` — dialog title, add/default/cancel/save buttons, alert/confirm messages
- Withdrawal section — error messages, placeholder

## Data Integrity

- **Data stored as Japanese** internally (asset type keys remain Japanese)
- `tAsset()` converts display-only; lookups/mutations always use the stored Japanese name
- Custom asset type names (not in `ASSET_TYPE_MAP`) are never translated

## Persistence

```js
// save()
{ funds, targetPcts, assetTypes, assetColors, lang }

// load()
if (data.lang) lang = data.lang;
```

## Key Translation Strings (partial list)

| Key | JA | EN |
|---|---|---|
| `appTitle` | アセットバランス管理 | Asset Balance Manager |
| `sectionFund` | ファンド入力 | Fund Entry |
| `sectionChart` | 現在のアセット配分 | Current Asset Allocation |
| `sectionAlloc` | 目標アロケーション & 差分 | Target Allocation & Diff |
| `sectionWithdraw` | 取り崩しシミュレーション | Withdrawal Simulation |
| `sectionData` | データ管理 | Data Management |
| `btnAdd` | 追加 | Add |
| `btnDelete` | 削除 | Delete |
| `btnCalc` | 計算 | Calculate |
| `btnExport` | JSONエクスポート | Export JSON |
| `btnImport` | JSONインポート | Import JSON |
| `btnClear` | 全データクリア | Clear All Data |
| `placeholderFundName` | ファンド名 | Fund Name |
| `placeholderAssetType` | アセット種別 | Asset Type |
| `placeholderAmount` | 金額 | Amount |
| `modeNormal` | 通常 | Normal |
| `modeBuyonly` | 買い増しリバランス | Buy-only Rebalance |
| `modeTsumitate` | 積立 | Accumulation |
| `chartTotal` | 合計 | Total |
| `emptyFunds` | ファンドが登録されていません… | No funds registered… |
