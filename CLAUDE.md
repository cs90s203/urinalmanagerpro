# CLAUDE.md — 尿斗管理大師 開發指引

## 專案快覽

**單一 HTML 檔案**，所有 CSS / JS 內嵌。最新穩定版：`Loole_v5_2_1.html`。
新版本命名規則：`Loole_vX_Y_Z.html`（不覆蓋舊版，保留歷史）。

部署：GitHub Pages (s90s203.github.io) + mick-lin.com。GA4：G-2C77WQG2VP。

---

## 修改前必讀

### 三段 `<script>` 的分工

| 段落 | 內容 | 注意 |
|------|------|------|
| Script 1 | 音效引擎（Web Audio API）+ BGM 函數 | 全部以 `_` 開頭，不依賴 state |
| Script 2 | Firebase SDK + `submitGlobalScore` / `loadGlobalLeaderboard` | compat 版本，不可換成 modular |
| Script 3 | CONFIG、ITEM_DEFS、TDEFS、主遊戲邏輯 | 所有遊戲邏輯在此 |

### 一定不能動的東西

- Firebase 設定區塊（line 699–708）：apiKey 等為 Realtime Database client key，plaintext 是正常的
- `HIST_KEY='urinal_history_v1'`：改掉會讓玩家失去所有歷史紀錄
- `ACH_KEY='loole_achievements'`、`CODEX_KEY='loole_seen_chars'`：同上

---

## 常見陷阱

### 1. 新增 CONFIG 值後記得同步 DFIELDS
`DFIELDS`（line 3207）是 Config Dashboard 的欄位定義，新增 CONFIG 不會自動出現在調整介面。

### 2. `_bgmNodes` 同時包含 BGM 和 SFX 節點
`_beep()` / `_noise()` 全部 push 到 `_bgmNodes`。呼叫 `_stopBGM()` 時**會強制停止所有音頻節點**，包含剛觸發的 SFX。BGM 切換時機若和 SFX 重疊，可能被截斷。

### 3. setTimeout 回呼不檢查 `state.running`
`tryAssign` 裡的 gangster scare setTimeout（~line 2099）、`applyPhaseStyle` 裡的 abyss 延遲（~line 1526）在回呼執行時不確認遊戲是否已結束。新增 setTimeout 時請加 `if(!state.running)return;`。

### 4. `mkUrinal` 的 `cleanDuration` 初始值
`mkUrinal` 用 `CONFIG.CLEAN_TIME`（未定義），`cleanDuration` 會是 `undefined`。自動清潔碼會在觸發時覆寫為 `0.5`，目前不影響功能，但修改清潔流程時需注意。

### 5. 深淵模式 CSS 覆蓋
`#screen-game.abyss-mode` 用 `!important` 覆蓋幾乎所有 UI 顏色（CSS line 54–88）。深淵模式下修改 UI 顏色需在 `.abyss-mode` 選擇器裡一起改。

---

## 新增角色的步驟

1. 在 `TDEFS` 新增定義（label、emoji、cv、wk/pk/dk/sk 指向 CONFIG 鍵）
2. 在 `CONFIG` 新增對應的 WAIT/PEE/DIRTY/SCORE 值
3. 在 `CHAR_ORDER` 新增 type 字串（影響圖鑑顯示順序）
4. 在 `rndType()` 的機率池加入條件
5. 若有特殊行為（如 gangster、inspector），在 `tryAssign` / `finishPee` 新增對應邏輯
6. 視需要加 `MC_LINES` 和 `mcSayNewChar` keyMap

## 新增道具的步驟

1. 在 `ITEM_DEFS` 新增定義（effects 是判斷邏輯的依據）
2. 在 `installItem` 新增 `if(def.effects.yourKey){}` 分支
3. 在 `MC_PREVIEW` 新增說明文字
4. `dropTarget`: `'urinal'`（預設）/ `'queue'` / `'any'`
5. 若有持久效果，在 `gameTick` 或 `finishPee` 處理

---

## localStorage 鍵一覽

| 鍵 | 類型 | 說明 |
|----|------|------|
| `urinal_history_v1` | JSON array | 本地 TOP 10 歷史紀錄 |
| `loole_achievements` | JSON object | `{id: true}` |
| `loole_seen_chars` | JSON array | 已見過的角色 type |
| `loole_ui_scale` | string (float) | UI 縮放倍率 |
| `loole_brightness` | string (float) | 亮度值 0.3~1.0 |

---

## 已知未解決問題

- **過夜重載**：手機瀏覽器回收記憶體後頁面重載，暫停機制無效。解法：`loole_save_v1` 存檔，見 `SAVE_RULES.md`
- **iOS Safari localStorage**：部分情況下 achievements / codex 無法儲存（private browsing 等）
