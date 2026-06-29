# SAVE_RULES.md — 存檔規則設計

解決「過夜重載」問題：手機瀏覽器回收記憶體後頁面重載，遊戲進度消失。

---

## 設計原則

參考 `jp_learning_mvp.html` 的存儲模式（`vocabBook_v1`, `study_log_v1` 等）：
- 單一 JSON 物件存入 localStorage
- Key 帶版本號，格式變動時直接棄舊
- try/catch 包裹，iOS Safari private mode 不 crash

---

## localStorage Key

```
loole_save_v1   // 遊戲中存檔
```

---

## 存什麼（序列化清單）

存檔只需保存**能重現遊戲狀態**的最小資料集。不需存 DOM。

```js
const SAVE_V1 = {
  v: 1,                        // schema 版本，未來 breaking change 用
  ts: Date.now(),              // 存檔時間戳（用來顯示「X 分鐘前的存檔」）

  // 基本計分
  score, money, totalEarned, scrubEarned, itemSpend,
  leftCount, servedCount, elapsed,

  // 便斗（不含 DOM 相關）
  urinals: urinals.map(u => ({
    id: u.id,
    dirty: u.dirty,
    broken: u.broken,
    repairing: u.repairing,
    repairElapsed: u.repairElapsed,
    // 清潔中狀態：reload 後不恢復（0.5s 很短，直接放棄）
    items: u.items.map(it => ({
      defId: it.def.id,        // 只存 id，reload 時從 ITEM_DEFS 還原 def
      durAccum: it.durAccum,
      cleanCount: it.cleanCount,
    })),
    // occupied 客人不存（reload 後移除正在使用的客人，視為已完成）
  })),

  // 等待區（只存基本資料，不存 waitElapsed 以免重載後大量人離開）
  queue: queue
    .filter(c => !c.isInspector)   // 馬蓋仙不恢復
    .map(c => ({
      id: c.id,
      type: c.type,
      waitMax: c.waitMax,
      waitElapsed: 0,              // 重置等待時間，給玩家喘息空間
      isDispatched: c.isDispatched,
    })),

  // Surge / 波次（全部捨棄，reload 後重新計算）
  nextPeak: elapsed + CONFIG.PEAK_INTERVAL,  // 重置為下次高峰
  peaksFired,
  delinquentSurgeCount,
  soldierWavesFired,
  inspectionReady,
  // 巡查中狀態不恢復（馬蓋仙離場）

  // 階段
  phaseStage,

  // 統計
  itemsBoughtThisRun: [...itemsBoughtThisRun],  // Set → Array
};
```

---

## 儲存時機

```js
// 每 30 秒自動存檔（在 gameTick 裡累積計時）
if (Math.floor(state.elapsed) % 30 === 0 && !_lastSaveFloor) {
  _lastSaveFloor = true;
  saveGame();
} else if (Math.floor(state.elapsed) % 30 !== 0) {
  _lastSaveFloor = false;
}

// 另外：遊戲暫停時也存一次（_pauseGame 裡）
```

---

## 儲存函數

```js
const SAVE_KEY = 'loole_save_v1';

function saveGame() {
  if (!state.running) return;
  const data = buildSaveData();  // 按上方清單建立
  try {
    localStorage.setItem(SAVE_KEY, JSON.stringify(data));
  } catch (e) {
    log('Save failed: ' + e.message, 'warn');
  }
}

function clearSave() {
  localStorage.removeItem(SAVE_KEY);
}
```

---

## 讀取 & 恢復

```js
function loadSave() {
  try {
    const raw = localStorage.getItem(SAVE_KEY);
    if (!raw) return null;
    const data = JSON.parse(raw);
    if (!data || data.v !== 1) { clearSave(); return null; }
    return data;
  } catch (e) {
    clearSave();
    return null;
  }
}

function restoreGame(data) {
  // 1. 還原 state 基本值
  // 2. 重建 urinals（用 data.urinals，items 從 ITEM_DEFS 還原 def）
  // 3. 還原 queue（重建 customer 物件，waitElapsed=0）
  // 4. cidCounter 設為 max(id) + 1
  // 5. 呼叫 buildUrinalDOM() / buildQueueDOM() / buildShop()
  // 6. applyPhaseStyle(data.phaseStage)
  // 7. playBGM(_baseBGM())
  // 8. 啟動 gameLoop
}
```

---

## Title 畫面的「繼續遊戲」按鈕

```html
<!-- 在 screen-title 的按鈕區 -->
<button id="btn-continue" class="btn-s" onclick="continueSave()" style="display:none">
  繼續遊戲 ⟳
</button>
```

```js
window.addEventListener('load', () => {
  const save = loadSave();
  const btn = document.getElementById('btn-continue');
  if (save && btn) {
    const mins = Math.round((Date.now() - save.ts) / 60000);
    btn.textContent = `繼續遊戲 (${mins} 分鐘前，${save.score} 分) ⟳`;
    btn.style.display = '';
  }
});

function continueSave() {
  const data = loadSave();
  if (!data) { startGame(); return; }
  clearSave();  // 讀取後清除，避免重複恢復
  restoreGame(data);
  showScreen('screen-game');
}
```

---

## 結束時清除存檔

```js
// endGame() 開頭加：
clearSave();
```

---

## iOS Safari 注意事項

Private Browsing 下 localStorage quota 為 0，任何寫入都會拋出 QuotaExceededError。
所有 `localStorage.setItem` 必須在 try/catch 內。存檔失敗時靜默處理，不影響遊戲流程。
