# Language Switch (JA/EN) Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add a JA/EN toggle button in the header that switches all UI labels and default asset type names between Japanese and English, persisted to localStorage.

**Architecture:** Single `index.html` file. A `TRANSLATIONS` object holds all UI strings; `ASSET_TYPE_MAP` holds default asset type name mappings. Helper functions `t(key)` and `tAsset(name)` return the correct string for the current language. `applyLang()` updates static DOM elements and re-runs `renderAll()`. Language persisted in the same `assetBalanceData` localStorage key.

**Tech Stack:** Vanilla JS, HTML, CSS — no build step, no framework. Open `index.html` directly in a browser to test.

---

## Context

The app is one file: `index.html`. All CSS is in `<style>`, all JS is in a `<script>` IIFE at the bottom. There is no test framework — verification is done visually in the browser and via browser console.

**Key patterns to know:**
- `renderAll()` re-renders everything; always call it after state changes
- `save()` writes to `localStorage.assetBalanceData`
- `load()` reads from it on startup
- Dynamic HTML is built with template literals in `renderFundTable()`, `renderChart()`, `renderAllocTable()`
- Settings dialog is built dynamically in `openSettings()`

---

## Task 1: Add Translation Dictionaries and Helper Functions

**File:** `index.html` — inside the `<script>` IIFE, right after the `const DEFAULT_COLORS` declaration (around line 305)

**Step 1: Add `TRANSLATIONS` constant**

Insert immediately after the `DEFAULT_COLORS` line:

```js
  const TRANSLATIONS = {
    ja: {
      appTitle: 'アセットバランス管理',
      sectionFund: 'ファンド入力',
      sectionChart: '現在のアセット配分',
      sectionAlloc: '目標アロケーション & 差分',
      sectionWithdraw: '取り崩しシミュレーション',
      sectionData: 'データ管理',
      btnAdd: '追加',
      btnDelete: '削除',
      btnCalc: '計算',
      btnExport: 'JSONエクスポート',
      btnImport: 'JSONインポート',
      btnClear: '全データクリア',
      phFundName: 'ファンド名',
      phAmount: '金額',
      phAssetType: 'アセット種別',
      phExtraInvest: '追加投資額',
      phTsumitate: '毎月の積立額',
      phWithdrawAmount: '取り崩し額',
      phWithdrawPct: '取り崩し率(%)',
      thFundName: 'ファンド名',
      thAssetType: 'アセット種別',
      thAmount: '金額',
      thTargetPct: '目標比率(%)',
      thCurrentAmt: '現在金額',
      thTargetAmt: '目標金額',
      thDiff: '差分',
      thDiffPct: '差分(%)',
      thBuyAmt: '買い増し額',
      thAfterAmt: '買増後金額',
      thAfterPct: '買増後比率',
      thMonthlyAlloc: '毎月積立額',
      thBalance: '現在残高',
      thSell: '売却額',
      thAfterBalance: '取崩後残高',
      thAfterRatio: '取崩後比率',
      thTargetRatio: '目標比率',
      totalLabel: '合計',
      chartTotal: '合計',
      noData: 'データがありません',
      emptyFunds: 'ファンドが登録されていません。上のフォームから追加してください。',
      allocWarn: '目標比率の合計が100%ではありません。',
      noteNoBuyExtra: '追加投資額を入力すると、その金額を最適に振り分けます',
      noteTsumitate: '毎月の積立額を入力すると、目標比率に応じた配分を表示します',
      modeNormal: '通常',
      modeBuyonly: '買い増しリバランス',
      modeTsumitate: '積立',
      modeAmount: '金額',
      modePct: 'パーセント',
      settingsDialogTitle: 'アセット種別設定',
      btnSettingsAdd: '＋ アセット種別を追加',
      btnSettingsDefault: 'デフォルトに戻す',
      btnSettingsCancel: 'キャンセル',
      btnSettingsSave: '保存',
      alertInvalidAmount: '金額を正しく入力してください。',
      alertMinAssetType: '少なくとも1つのアセット種別が必要です。',
      confirmClearAll: '全てのファンドデータと目標比率をクリアします。よろしいですか？',
      alertImportDone: 'インポートが完了しました。',
      alertImportFail: 'JSONファイルの読み込みに失敗しました。',
      errNoFunds: 'ファンドが登録されていません。',
      errNoAlloc: '目標アロケーションを設定してください。',
      errInvalidPct: '取り崩し率を正しく入力してください。',
      errInvalidWithdraw: '取り崩し額を正しく入力してください。',
      errExceedsTotal: '取り崩し額がポートフォリオ合計を超えています。',
    },
    en: {
      appTitle: 'Asset Balance Manager',
      sectionFund: 'Fund Entry',
      sectionChart: 'Current Asset Allocation',
      sectionAlloc: 'Target Allocation & Diff',
      sectionWithdraw: 'Withdrawal Simulation',
      sectionData: 'Data Management',
      btnAdd: 'Add',
      btnDelete: 'Delete',
      btnCalc: 'Calculate',
      btnExport: 'Export JSON',
      btnImport: 'Import JSON',
      btnClear: 'Clear All Data',
      phFundName: 'Fund Name',
      phAmount: 'Amount',
      phAssetType: 'Asset Type',
      phExtraInvest: 'Extra Investment',
      phTsumitate: 'Monthly Amount',
      phWithdrawAmount: 'Withdrawal Amount',
      phWithdrawPct: 'Withdrawal Rate (%)',
      thFundName: 'Fund Name',
      thAssetType: 'Asset Type',
      thAmount: 'Amount',
      thTargetPct: 'Target (%)',
      thCurrentAmt: 'Current',
      thTargetAmt: 'Target',
      thDiff: 'Diff',
      thDiffPct: 'Diff (%)',
      thBuyAmt: 'Buy Amount',
      thAfterAmt: 'After Buy',
      thAfterPct: 'After Buy Ratio',
      thMonthlyAlloc: 'Monthly Alloc',
      thBalance: 'Current Balance',
      thSell: 'Sell Amount',
      thAfterBalance: 'After Withdrawal',
      thAfterRatio: 'After Ratio',
      thTargetRatio: 'Target Ratio',
      totalLabel: 'Total',
      chartTotal: 'Total',
      noData: 'No data',
      emptyFunds: 'No funds registered. Add one using the form above.',
      allocWarn: 'Target percentages do not sum to 100%.',
      noteNoBuyExtra: 'Enter an extra investment amount to optimize allocation.',
      noteTsumitate: 'Enter a monthly accumulation amount to see allocation by target ratio.',
      modeNormal: 'Normal',
      modeBuyonly: 'Buy-only Rebalance',
      modeTsumitate: 'Accumulation',
      modeAmount: 'Amount',
      modePct: 'Percent',
      settingsDialogTitle: 'Asset Type Settings',
      btnSettingsAdd: '+ Add Asset Type',
      btnSettingsDefault: 'Reset to Default',
      btnSettingsCancel: 'Cancel',
      btnSettingsSave: 'Save',
      alertInvalidAmount: 'Please enter a valid amount.',
      alertMinAssetType: 'At least one asset type is required.',
      confirmClearAll: 'Clear all fund data and target percentages. Are you sure?',
      alertImportDone: 'Import complete.',
      alertImportFail: 'Failed to read JSON file.',
      errNoFunds: 'No funds registered.',
      errNoAlloc: 'Please set the target allocation.',
      errInvalidPct: 'Please enter a valid withdrawal rate.',
      errInvalidWithdraw: 'Please enter a valid withdrawal amount.',
      errExceedsTotal: 'Withdrawal amount exceeds portfolio total.',
    }
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

**Step 2: Add language state variable and helper functions**

Insert after `let lastAddedFundId = null;` (around line 316):

```js
  let lang = 'ja';

  function t(key) {
    return (TRANSLATIONS[lang] && TRANSLATIONS[lang][key]) || TRANSLATIONS.ja[key] || key;
  }

  function tAsset(name) {
    if (lang === 'en' && ASSET_TYPE_MAP[name]) return ASSET_TYPE_MAP[name];
    return name;
  }
```

**Step 3: Open index.html in browser and verify in console**

Open browser DevTools console and run:
```js
// These should be accessible from window (they're in IIFE so actually not — that's fine)
// Just visually confirm the file has no syntax errors by checking the page loads
```
Expected: Page loads normally, no console errors.

**Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add i18n translation dictionaries and t()/tAsset() helpers"
```

---

## Task 2: Add Language State to Persistence

**File:** `index.html` — `save()` and `load()` functions

**Step 1: Update `save()` to include `lang`**

Find this line in `save()`:
```js
    const data = { funds, targetPcts, assetTypes, assetColors };
```
Replace with:
```js
    const data = { funds, targetPcts, assetTypes, assetColors, lang };
```

**Step 2: Update `load()` to restore `lang`**

Find the line near the end of `load()`:
```js
    } catch(e) { /* ignore corrupt data */ }
```
Just before that line, inside the `if (!raw) return;` block, add after `assetColors` is set:
```js
      if (data.lang === 'en') lang = 'en';
```

The correct insertion point is after `while (assetColors.length < assetTypes.length)` block and before `if (Array.isArray(data.funds))`. Full updated load context:

```js
      while (assetColors.length < assetTypes.length) {
        assetColors.push(DEFAULT_COLORS[assetColors.length] || '#888888');
      }
      if (data.lang === 'en') lang = 'en';   // <-- ADD THIS LINE
      if (Array.isArray(data.funds)) funds = data.funds;
```

**Step 3: Verify in browser console**

```js
localStorage.setItem('assetBalanceData', JSON.stringify({...JSON.parse(localStorage.getItem('assetBalanceData')), lang: 'en'}));
location.reload();
```
Expected: Page reloads without errors. (Lang doesn't visually change yet — that's fine.)

**Step 4: Commit**

```bash
git add index.html
git commit -m "feat: persist language preference to localStorage"
```

---

## Task 3: Add JA/EN Toggle Button to Header

**File:** `index.html` — HTML `<header>` and CSS `<style>`

**Step 1: Add CSS for the language toggle**

Find `.settings-btn` CSS block (around line 183). Add after it, before the `/* Responsive */` comment:

```css
/* Language toggle */
.lang-toggle {
  display: inline-flex; border: 1px solid rgba(255,255,255,0.4); border-radius: 6px; overflow: hidden;
}
.lang-toggle button {
  padding: 0.25rem 0.6rem; border: none; background: transparent; color: rgba(255,255,255,0.7);
  cursor: pointer; font-size: 0.8rem; font-weight: 600; transition: background 0.2s, color 0.2s;
}
.lang-toggle button.active {
  background: rgba(255,255,255,0.2); color: #fff;
}
.lang-toggle button:hover:not(.active) { color: #fff; }
```

**Step 2: Update `<header>` HTML**

Find:
```html
  <div class="header-row">
    <h1>アセットバランス管理</h1>
    <button type="button" class="settings-btn" id="settingsBtn" title="アセット種別設定">⚙</button>
  </div>
```

Replace with:
```html
  <div class="header-row">
    <h1 data-i18n="appTitle">アセットバランス管理</h1>
    <div class="lang-toggle" id="langToggle">
      <button type="button" data-lang="ja" class="active">JA</button>
      <button type="button" data-lang="en">EN</button>
    </div>
    <button type="button" class="settings-btn" id="settingsBtn" title="アセット種別設定">⚙</button>
  </div>
```

**Step 3: Verify visually**

Open `index.html` in browser. You should see `[JA|EN]` segment buttons in the header, with JA highlighted. They don't do anything yet.

**Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add JA/EN toggle button to header"
```

---

## Task 4: Add `data-i18n` Attributes to Static HTML

**File:** `index.html` — HTML body, static elements only

**Step 1: Update `<title>`**

```html
<title data-i18n="appTitle">アセットバランス管理</title>
```

Note: `<title>` can't use textContent via data-i18n in the same way (it needs special handling). Handle it in `applyLang()` explicitly instead — skip the attribute on title, just update it in JS.

**Step 2: Update section headings**

```html
<!-- Fund section -->
<h2 data-i18n="sectionFund">ファンド入力</h2>

<!-- Chart section -->
<h2 data-i18n="sectionChart">現在のアセット配分</h2>

<!-- Allocation section -->
<h2 data-i18n="sectionAlloc">目標アロケーション &amp; 差分</h2>

<!-- Withdrawal section -->
<h2 data-i18n="sectionWithdraw">取り崩しシミュレーション</h2>

<!-- Data section -->
<h2 data-i18n="sectionData">データ管理</h2>
```

**Step 3: Update fund form**

```html
<input type="text" name="fundName" data-i18n-placeholder="phFundName" placeholder="ファンド名" required>
<input type="text" name="amount" data-i18n-placeholder="phAmount" placeholder="金額" required inputmode="numeric">
<button type="submit" class="btn btn-primary" data-i18n="btnAdd">追加</button>
```

The asset type select first option:
```html
<option value="" data-i18n="phAssetType">アセット種別</option>
```

**Step 4: Update fund table static headers**

```html
<thead><tr><th></th><th data-i18n="thFundName">ファンド名</th><th data-i18n="thAssetType">アセット種別</th><th class="amount" data-i18n="thAmount">金額</th><th></th></tr></thead>
```

**Step 5: Update rebalance toggle buttons**

```html
<button type="button" data-mode="tsumitate" data-i18n="modeTsumitate">積立</button>
<button type="button" class="active" data-mode="normal" data-i18n="modeNormal">通常</button>
<button type="button" data-mode="buyonly" data-i18n="modeBuyonly">買い増しリバランス</button>
```

Extra invest / tsumitate inputs (placeholder only):
```html
<input type="text" class="rebalance-extra-input" id="extraInvestInput" data-i18n-placeholder="phExtraInvest" placeholder="追加投資額" inputmode="numeric" style="display:none;">
<input type="text" class="rebalance-extra-input" id="tsumitateInput" data-i18n-placeholder="phTsumitate" placeholder="毎月の積立額" inputmode="numeric" style="display:none;">
```

allocWarn:
```html
<p class="validation-warn" id="allocWarn" data-i18n="allocWarn">目標比率の合計が100%ではありません。</p>
```

**Step 6: Update withdrawal section**

Toggle buttons:
```html
<button type="button" class="active" data-mode="amount" data-i18n="modeAmount">金額</button>
<button type="button" data-mode="percent" data-i18n="modePct">パーセント</button>
```

Calc button:
```html
<button type="button" class="btn btn-primary" id="withdrawCalcBtn" data-i18n="btnCalc">計算</button>
```

Withdrawal input placeholder — handled dynamically in JS (the placeholder changes based on mode), so do NOT add data-i18n-placeholder here.

Withdrawal result table static headers:
```html
<thead><tr>
  <th data-i18n="thAssetType">アセット種別</th>
  <th class="amount" data-i18n="thBalance">現在残高</th>
  <th class="amount" data-i18n="thSell">売却額</th>
  <th class="amount" data-i18n="thAfterBalance">取崩後残高</th>
  <th class="amount" data-i18n="thAfterRatio">取崩後比率</th>
  <th class="amount" data-i18n="thTargetRatio">目標比率</th>
</tr></thead>
```

**Step 7: Update data management buttons**

```html
<button class="btn btn-secondary" id="exportBtn" data-i18n="btnExport">JSONエクスポート</button>
<button class="btn btn-secondary" id="importBtn" data-i18n="btnImport">JSONインポート</button>
<button class="btn btn-danger" id="clearBtn" data-i18n="btnClear">全データクリア</button>
```

**Step 8: Verify**

Open page in browser. All text should still display correctly in Japanese (no changes yet — `applyLang()` hasn't been written). Check DevTools that no JS errors appear.

**Step 9: Commit**

```bash
git add index.html
git commit -m "feat: add data-i18n attributes to static HTML elements"
```

---

## Task 5: Implement `applyLang()` and Wire Up the Toggle

**File:** `index.html` — JS section, after the `centerTextPlugin` block

**Step 1: Add `applyLang()` function**

Add this function after the `centerTextPlugin` constant (around line 337):

```js
  // --- Language ---
  function applyLang() {
    document.documentElement.lang = lang;
    document.title = t('appTitle');
    document.querySelectorAll('[data-i18n]').forEach(el => {
      el.textContent = t(el.dataset.i18n);
    });
    document.querySelectorAll('[data-i18n-placeholder]').forEach(el => {
      el.placeholder = t(el.dataset.i18nPlaceholder);
    });
    // Sync lang toggle buttons
    document.querySelectorAll('#langToggle button').forEach(btn => {
      btn.classList.toggle('active', btn.dataset.lang === lang);
    });
    renderAll();
  }
```

**Step 2: Wire up the toggle event listener**

Find the `// --- Events ---` section (around line 930). Add the lang toggle handler right after `document.getElementById('settingsBtn').addEventListener('click', openSettings);`:

```js
  // Language toggle
  document.getElementById('langToggle').addEventListener('click', function(e) {
    const btn = e.target.closest('button[data-lang]');
    if (!btn || btn.dataset.lang === lang) return;
    lang = btn.dataset.lang;
    save();
    applyLang();
  });
```

**Step 3: Call `applyLang()` in the init section**

Find the `// --- Init ---` section at the bottom:
```js
  // --- Init ---
  load();
  renderAll();
```
Replace with:
```js
  // --- Init ---
  load();
  applyLang();
```
(This calls `renderAll()` internally via `applyLang()`, so `renderAll()` doesn't need to be called separately.)

**Step 4: Verify toggle works**

Open `index.html` in browser. Click `EN`:
- Header title should change to "Asset Balance Manager"
- Section headings: "Fund Entry", "Current Asset Allocation", etc.
- Buttons: "Add", "Delete", "Calculate", etc.
- Table headers change
- Click `JA` → reverts to Japanese
- Reload page → language is remembered

**Step 5: Commit**

```bash
git add index.html
git commit -m "feat: implement applyLang() and wire up language toggle"
```

---

## Task 6: Update Dynamic Render Functions

**File:** `index.html` — JS `renderFundTable()`, `renderChart()`, `renderAllocTable()`, and empty message

**Step 1: Update `renderFundTable()`**

The empty message element already exists in static HTML. Add `data-i18n` to it (covered in Task 4 Step 2 — actually we handled `emptyMsg` there? No — let me check).

The empty message `<p class="empty-msg" id="emptyMsg">` is static HTML:
```html
<p class="empty-msg" id="emptyMsg" data-i18n="emptyFunds">ファンドが登録されていません。上のフォームから追加してください。</p>
```
Add `data-i18n="emptyFunds"` to this element (if not already done in Task 4). The `renderFundTable()` JS shows/hides it via `style.display` — it doesn't set `textContent`, so `applyLang()` handles the text automatically.

In the `renderFundTable()` template literal, update the delete button:
```js
        <td><button class="btn btn-danger" data-delete="${f.id}">${t('btnDelete')}</button></td>
```

**Step 2: Update `renderChart()` — center text plugin**

The `centerTextPlugin` uses a hardcoded `'合計'`. Update it:
```js
      ctx.fillText(t('chartTotal'), cx, cy - 14);
```

The empty legend message:
```js
      legend.innerHTML = '<p class="empty-msg">' + t('noData') + '</p>';
```

**Step 3: Update `renderAllocTable()` — all three mode headers**

**Buy-only mode** (find `thead.innerHTML = '<th>アセット種別</th>...'`):
```js
      thead.innerHTML = `<th>${t('thAssetType')}</th><th class="amount">${t('thTargetPct')}</th><th class="amount">${t('thCurrentAmt')}</th><th class="amount">${t('thBuyAmt')}</th><th class="amount">${t('thAfterAmt')}</th><th class="amount">${t('thAfterPct')}</th>`;
```

Buy-only asset type cell (replace the `${t}` variable — the `t` in template literal here is the loop variable, but our function is also named `t`): The loop uses `.map((t, i) =>` where `t` is the asset type name. Rename the loop variable to avoid conflict:
```js
      tbody.innerHTML = assetTypes.map((assetType, i) => {
        const current = totals[assetType];
        const pct = targetPcts[assetType] || 0;
        // ... rest of logic using assetType instead of t
      })
```
The asset type cell in buy-only:
```js
          <td><span class="chart-legend-color" style="background:${assetColors[i]};display:inline-block;vertical-align:middle;margin-right:0.4rem;"></span>${tAsset(assetType)}</td>
```

Buy-only note messages:
```js
        note.textContent = t('noteNoBuyExtra');
```

Buy-only tfoot:
```js
      tfoot.innerHTML = `<td>${t('totalLabel')}</td><td class="amount" id="totalTargetPct">${totalPct.toFixed(1)}%</td><td class="amount" id="totalCurrent">${formatYen(grand)}</td><td class="amount ${totalBuy > 0.5 ? 'diff-positive' : ''}" id="totalDiff">${totalBuy > 0.5 ? '+' + formatYen(totalBuy) : '-'}</td><td class="amount" id="totalTarget">${formatYen(afterGrand)}</td><td class="amount" id="totalDiffPct">100%</td>`;
```

**Tsumitate mode** (find `thead.innerHTML = '<th>アセット種別</th><th class="amount">目標比率(%)</th>...'`):
```js
      thead.innerHTML = `<th>${t('thAssetType')}</th><th class="amount">${t('thTargetPct')}</th><th class="amount">${t('thMonthlyAlloc')}</th>`;
```

Tsumitate loop (rename `t` loop var to `assetType`):
```js
      tbody.innerHTML = assetTypes.map((assetType, i) => {
        const pct = targetPcts[assetType] || 0;
        const alloc = (hasInput && totalPct > 0) ? monthlyTotal * pct / totalPct : 0;
        sumAlloc += alloc;
        return `<tr>
          <td><span class="chart-legend-color" style="background:${assetColors[i]};display:inline-block;vertical-align:middle;margin-right:0.4rem;"></span>${tAsset(assetType)}</td>
          <td class="amount"><input type="number" min="0" max="100" step="0.1" value="${pct}" data-asset="${assetType}"></td>
          <td class="amount">${hasInput && pct > 0 ? formatYen(alloc) : '-'}</td>
        </tr>`;
      }).join('');
```

Tsumitate tfoot:
```js
      tfoot.innerHTML = `<td>${t('totalLabel')}</td><td class="amount" id="totalTargetPct">${totalPct.toFixed(1)}%</td><td class="amount" id="totalCurrent">${hasInput ? formatYen(sumAlloc) : '-'}</td>`;
```

Tsumitate note:
```js
        note.textContent = t('noteTsumitate');
```

**Normal mode** (find `thead.innerHTML = '<th>アセット種別</th><th class="amount">目標比率(%)</th>...'`):
```js
      thead.innerHTML = `<th>${t('thAssetType')}</th><th class="amount">${t('thTargetPct')}</th><th class="amount">${t('thCurrentAmt')}</th><th class="amount">${t('thTargetAmt')}</th><th class="amount">${t('thDiff')}</th><th class="amount">${t('thDiffPct')}</th>`;
```

Normal loop (rename `t` to `assetType`):
```js
      tbody.innerHTML = assetTypes.map((assetType, i) => {
        const current = totals[assetType];
        const pct = targetPcts[assetType] || 0;
        const target = grand * pct / 100;
        const diff = target - current;
        const diffClass = diff > 0.5 ? 'diff-positive' : diff < -0.5 ? 'diff-negative' : '';
        const diffPct = grand > 0 ? (diff / grand * 100).toFixed(1) : '0.0';
        return `<tr>
          <td><span class="chart-legend-color" style="background:${assetColors[i]};display:inline-block;vertical-align:middle;margin-right:0.4rem;"></span>${tAsset(assetType)}</td>
          <td class="amount"><input type="number" min="0" max="100" step="0.1" value="${pct}" data-asset="${assetType}"></td>
          <td class="amount">${formatYen(current)}</td>
          <td class="amount">${formatYen(target)}</td>
          <td class="amount ${diffClass}">${diff >= 0 ? '+' : ''}${formatYen(diff)}</td>
          <td class="amount ${diffClass}">${diff >= 0 ? '+' : ''}${diffPct}%</td>
        </tr>`;
      }).join('');
```

Normal tfoot:
```js
      tfoot.innerHTML = `<td>${t('totalLabel')}</td><td class="amount" id="totalTargetPct">${totalPct.toFixed(1)}%</td><td class="amount" id="totalCurrent">${formatYen(grand)}</td><td class="amount" id="totalTarget">${formatYen(totalTarget)}</td><td class="amount ${totalDiff > 0.5 ? 'diff-positive' : totalDiff < -0.5 ? 'diff-negative' : ''}" id="totalDiff">${(totalDiff >= 0 ? '+' : '') + formatYen(totalDiff)}</td><td class="amount ${totalDiff > 0.5 ? 'diff-positive' : totalDiff < -0.5 ? 'diff-negative' : ''}" id="totalDiffPct">${(totalDiff >= 0 ? '+' : '') + (grand > 0 ? (totalDiff / grand * 100).toFixed(1) : '0.0')}%</td>`;
```

**Step 4: Update `renderAssetSelect()`**

```js
    sel.innerHTML = `<option value="">${t('phAssetType')}</option>` +
      assetTypes.map(type => `<option value="${escapeHtml(type)}">${escapeHtml(tAsset(type))}</option>`).join('');
```

**Step 5: Update withdrawal body**

In the `withdrawCalcBtn` click handler, update the tbody template:
```js
      const assetType = t;   // t is the loop var — already renamed in allocTable, but withdrawBody uses assetTypes.map(t => ...)
```

Find:
```js
    tbody.innerHTML = assetTypes.map((t, i) => {
      const r = result[t];
      if (r.current <= 0 && r.targetPct <= 0) return '';
      return `<tr>
        <td><span ...></span>${t}</td>
```

Update to:
```js
    tbody.innerHTML = assetTypes.map((assetType, i) => {
      const r = result[assetType];
      if (r.current <= 0 && r.targetPct <= 0) return '';
      return `<tr>
        <td><span class="chart-legend-color" style="background:${assetColors[i]};display:inline-block;vertical-align:middle;margin-right:0.4rem;"></span>${tAsset(assetType)}</td>
        <td class="amount">${formatYen(r.current)}</td>
        <td class="amount sell-amount">${r.sell > 0.5 ? '-' + formatYen(r.sell) : '-'}</td>
        <td class="amount">${formatYen(r.after)}</td>
        <td class="amount">${r.afterPct.toFixed(1)}%</td>
        <td class="amount">${r.targetPct.toFixed(1)}%</td>
      </tr>`;
    }).join('');
```

**Step 6: Update withdrawal mode toggle (placeholder)**

In the `withdrawToggle` click handler, find:
```js
    document.getElementById('withdrawInput').placeholder = withdrawMode === 'amount' ? '取り崩し額' : '取り崩し率(%)';
```
Replace with:
```js
    document.getElementById('withdrawInput').placeholder = withdrawMode === 'amount' ? t('phWithdrawAmount') : t('phWithdrawPct');
```

**Step 7: Verify**

Switch to EN mode. Check that:
- All section headings, buttons, table headers show English
- Allocation table (all 3 modes) shows English headers and "Total" label
- Default asset type names (国内株 etc.) show in English in dropdowns, chart legend, table rows
- Custom-named types (if any) remain unchanged
- Withdrawal section shows English

**Step 8: Commit**

```bash
git add index.html
git commit -m "feat: update dynamic renders to use t() and tAsset()"
```

---

## Task 7: Update Settings Dialog and Alert/Confirm Messages

**File:** `index.html` — `openSettings()` and other event handlers

**Step 1: Update `openSettings()` dialog HTML**

Find in `openSettings()`:
```js
    dialog.innerHTML = `
      <h2>アセット種別設定</h2>
      <div class="settings-list"></div>
      <div class="settings-add-row">
        <button type="button" class="btn btn-secondary" id="settingsAddBtn">＋ アセット種別を追加</button>
      </div>
      <div class="settings-actions">
        <button type="button" class="btn btn-secondary" id="settingsDefaultBtn">デフォルトに戻す</button>
        <button type="button" class="btn btn-secondary" id="settingsCancelBtn">キャンセル</button>
        <button type="button" class="btn btn-primary" id="settingsSaveBtn">保存</button>
      </div>`;
```

Replace with:
```js
    dialog.innerHTML = `
      <h2>${t('settingsDialogTitle')}</h2>
      <div class="settings-list"></div>
      <div class="settings-add-row">
        <button type="button" class="btn btn-secondary" id="settingsAddBtn">${t('btnSettingsAdd')}</button>
      </div>
      <div class="settings-actions">
        <button type="button" class="btn btn-secondary" id="settingsDefaultBtn">${t('btnSettingsDefault')}</button>
        <button type="button" class="btn btn-secondary" id="settingsCancelBtn">${t('btnSettingsCancel')}</button>
        <button type="button" class="btn btn-primary" id="settingsSaveBtn">${t('btnSettingsSave')}</button>
      </div>`;
```

**Step 2: Update alert/confirm in `applyAssetTypeChanges()`**

```js
    if (validItems.length === 0) {
      alert(t('alertMinAssetType'));
      return;
    }
```

For the confirm with fund names:
```js
    if (deletedWithFunds.length > 0) {
      const names = deletedWithFunds.join('、');
      const msg = lang === 'en'
        ? `"${names}" has registered funds. Deleting will leave those funds without a type. Continue?`
        : `「${names}」にはファンドが登録されています。種別を削除するとこれらのファンドが種別なしになります。続行しますか？`;
      if (!confirm(msg)) return;
    }
```

**Step 3: Update fund form alert**

```js
    if (isNaN(amount)) { alert(t('alertInvalidAmount')); return; }
```

**Step 4: Update fund delete confirm**

```js
      const label = fund ? fund.name : (lang === 'en' ? 'this fund' : 'このファンド');
      const msg = lang === 'en' ? `Delete "${label}"?` : `「${label}」を削除しますか？`;
      if (!confirm(msg)) return;
```

**Step 5: Update clear all confirm**

```js
    if (!confirm(t('confirmClearAll'))) return;
```

**Step 6: Update import alerts**

```js
        alert(t('alertImportDone'));
      } catch(err) {
        alert(t('alertImportFail'));
```

**Step 7: Update withdrawal error messages**

```js
      errEl.textContent = t('errNoFunds');
      // ...
      errEl.textContent = t('errNoAlloc');
      // ...
      errEl.textContent = t('errInvalidPct');
      // ...
      errEl.textContent = t('errInvalidWithdraw');
      // ...
      errEl.textContent = t('errExceedsTotal');
```

**Step 8: Verify**

In EN mode:
- Open settings dialog (⚙) → title "Asset Type Settings", buttons in English
- Try to add a fund with invalid amount → English alert
- Try to delete a fund → English confirm
- Click "Clear All Data" → English confirm
- Import invalid JSON → English error alert
- Test withdrawal with no funds/no allocation → English error messages

**Step 9: Commit**

```bash
git add index.html
git commit -m "feat: translate settings dialog and alert/confirm messages"
```

---

## Task 8: Final Verification

**Step 1: Full JA → EN → JA round-trip test**

1. Start in JA mode (default): verify all text is Japanese
2. Add a fund (e.g., "eMAXIS Slim 全世界株式", 国内株, 100000)
3. Switch to EN: verify fund table still shows the fund; column headers in English; "Domestic Equity" shown if 国内株 is in default list
4. Set target allocation percentages
5. Switch rebalance modes (Normal / Buy-only / Accumulation): verify each mode's headers are in English
6. Run withdrawal simulation: verify English labels
7. Open settings dialog (⚙): verify English labels; add/delete/rename works
8. Reload page: verify EN is remembered, everything still displays correctly
9. Switch back to JA: verify full Japanese display
10. Reload: verify JA is remembered

**Step 2: Verify `data-asset` attribute still works**

The allocation table inputs use `data-asset="${assetType}"` (formerly `data-asset="${t}"`). In the change event handler:
```js
  document.getElementById('allocTable').addEventListener('change', function(e) {
    if (e.target.dataset.asset) {
      targetPcts[e.target.dataset.asset] = ...
```
The key here is always the Japanese asset type name (that's how it's stored). Verify that editing target percentages in EN mode correctly updates `targetPcts`.

**Step 3: Commit any fixes found during verification**

```bash
git add index.html
git commit -m "fix: [describe any fixes]"
```

---

## Summary of Changes

| Location | Change |
|---|---|
| JS top | Add `TRANSLATIONS`, `ASSET_TYPE_MAP`, `lang` variable |
| JS utilities | Add `t()`, `tAsset()`, `applyLang()` |
| `save()` / `load()` | Persist `lang` |
| `<header>` HTML | Add `[JA|EN]` toggle, `data-i18n` on `<h1>` |
| All static `<h2>`, buttons, inputs | Add `data-i18n` / `data-i18n-placeholder` |
| `renderFundTable()` | Delete button uses `t()`, `data-i18n` on emptyMsg |
| `renderChart()` | `t('chartTotal')`, `t('noData')` |
| `renderAllocTable()` | All 3 mode headers, notes, tfoot use `t()` and `tAsset()`; loop var renamed from `t` to `assetType` |
| `renderAssetSelect()` | Option labels use `tAsset()` |
| Withdrawal result | tbody uses `tAsset()`, error messages use `t()` |
| `openSettings()` | Dialog HTML uses `t()` |
| All `alert()`/`confirm()` | Use `t()` |
| Init | `applyLang()` instead of `renderAll()` |
| Events | Lang toggle listener |
