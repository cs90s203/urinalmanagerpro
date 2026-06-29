# TECH.md — 尿斗管理大師 技術參考 v5.2.1

## 遊戲主循環

```
startGame()
  → gameLoop = setInterval(gameTick, 80)   ← 80ms tick = ~12.5fps

gameTick(dt)
  1. 計算 dt（含子彈時間縮放）
  2. 高峰期觸發檢查
  3. pendingWaves 派發
  4. spawnAccum 累積 → spawnCustomer()
  5. 等待區計時 → customerLeave()
  6. 馬蓋仙超時重試
  7. 便斗循環（修復/清潔/尿尿進度 → finishPee）
  8. 清掃助理自動清潔
  9. BGM 自動回 normal 判斷
  10. 巡查進行中 → tickInspection()
  11. 分數階段計算 → applyPhaseStyle()
  12. 同步 DOM（syncQueueDOM / syncUrinalDOM / updateHUD）
```

## 關鍵函數索引

| 函數 | 行數 | 說明 |
|------|------|------|
| `startGame()` | ~1544 | 重置 state，啟動 gameLoop |
| `gameTick()` | ~1601 | 主循環，每 80ms 執行 |
| `spawnCustomer(type?)` | ~2036 | 生成客人 → `_addToQueue` |
| `_addToQueue(type)` | ~2041 | 加入等待區，建立 DOM card |
| `tryAssign(cid, uid)` | ~2062 | 把客人安排到便斗 |
| `finishPee(u)` | ~2430 | 客人離開便斗，計分、計費、加髒污 |
| `customerLeave(id)` | ~2414 | 客人氣走，扣分，檢查失敗條件 |
| `installItem(uid, itemId)` | ~1225 | 安裝道具到便斗/等待區 |
| `triggerSurge(type)` | ~1856 | 觸發特殊波次（中二生/阿兵哥）|
| `triggerInspection()` | ~1923 | 馬蓋仙進場 |
| `startInspection(uid)` | ~1944 | 開始巡查掃描 |
| `tickInspection(dt)` | ~1953 | 逐格掃描計時 |
| `endInspection()` | ~1992 | 巡查結束，發獎懲 |
| `endGame()` | ~2817 | 遊戲結束，計算結算 |
| `applyPhaseStyle(stage)` | ~1500 | 階段顏色 + 深淵模式切換 |
| `calcSpd(u)` | ~1826 | 計算便斗速度（鄰近/髒污減速）|
| `calcEntryFee()` | ~1207 | 計算入廁費 |
| `playBGM(type, returnAfterMs?)` | ~542 | 切換 BGM |
| `playSFX(name)` | ~480 | 播放音效 |
| `mcSay(key)` | ~3076 | 一次性台詞 |
| `mcSayForced(key)` | ~3115 | 可重複台詞 |
| `checkAchievements()` | ~1430 | 檢查並解鎖成就 |
| `buildUrinalDOM()` | ~2495 | 重建便斗 DOM（加馬桶時呼叫）|
| `syncUrinalDOM()` | ~2701 | 每幀同步便斗狀態到 DOM |
| `syncQueueDOM()` | ~2676 | 每幀同步等待區到 DOM |

---

## State 物件完整結構

```js
state = {
  // 基本計分
  score,            // 當前分數
  money,            // 當前金錢 £
  totalEarned,      // 累計收入（含清潔費）
  scrubEarned,      // 清潔費收入
  itemSpend,        // 道具支出

  // 遊戲進度
  leftCount,        // 氣走人數（>=3 結束）
  servedCount,      // 已服務人數
  elapsed,          // 遊戲時間（秒，受 timeScale 影響）

  // 核心資料
  urinals: [],      // 便斗陣列（見 mkUrinal）
  queue: [],        // 等待區客人陣列

  // 選取/拖曳狀態
  selectedId,       // 選中的客人 cid，null=無

  // 高峰/生成
  nextPeak,         // 下次高峰觸發時間（elapsed）
  spawnAccum,       // 生成時間累積器
  peaksFired,       // 已觸發高峰次數
  peakBGMEnd,       // 高峰 BGM 結束時間

  // 遊戲流程
  running,          // 遊戲是否進行中
  failed,           // 是否失敗（氣走 ≥3）

  // 子彈時間
  bulletTime,       // 是否開啟子彈時間
  bulletOpRemain,   // 子彈操作剩餘秒數（用 rawDt 扣）

  // 等待加時
  waitBoostEnd,     // 加時效果結束時間
  waitBoostEmoji,   // 目前加時道具 emoji
  activeWaitBoostAmt, // 加時秒數

  // Surge 系統
  surgeType,        // 目前 surge 類型（顯示用）
  surgeEnd,         // surge 結束時間（舊版，已廢棄）
  pendingWaves: [], // [{type, at, group, isDispatched?}]
  delinquentSurgeCount, // 中二生 surge 累計次數
  gangsterSurgeEnd,     // 黑尿幫 surge 結束時間
  soldierWavesFired,    // 阿兵哥波次累計

  // 馬蓋仙巡查
  specialEventCount,    // 特殊事件總計數
  inspectionReady,      // 是否已解鎖巡查
  inspectionActive,     // 是否正在巡查
  inspectionScanIdx,    // 目前掃描格 index
  inspectionScanElapsed,// 目前格已掃描時間
  inspectionPassed,     // 本次巡查是否通過
  inspectionCid,        // 等待中的馬蓋仙 cid

  // 階段
  phaseStage,       // 0–14

  // 統計
  itemsBoughtThisRun: new Set(), // 本局買過的道具 id

  // 派遣
  dispatchedWaveTypes: [], // 待標記為派遣的 surge 類型

  // 預覽（道具長按）
  itemPreview,      // 道具預覽中（時間縮放到 10%）
}
```

---

## 便斗物件結構

```js
{
  id,               // 0-based index
  dirty,            // 0–120%（超過 100% 立即損壞）
  occupied,         // 客人物件或 null
  peeElapsed,       // 目前客人已尿秒數
  cleaning,         // 是否清潔中
  cleanElapsed,     // 已清潔秒數
  cleanDuration,    // 清潔總秒數（auto-clean 設為 0.5）
                    // ⚠️ mkUrinal 用 CONFIG.CLEAN_TIME（未定義），初始為 undefined
  repairing,        // 是否修復中
  repairElapsed,    // 已修復秒數
  broken,           // 是否損壞
  items: [],        // [{def, durAccum, cleanCount}]
}
```

---

## 客人物件結構

```js
{
  id,               // cidCounter 遞增
  type,             // 'small'|'elder'|'med'|'large'|'drunk'|'delinquent'|'soldier'|'gangster'|'inspector'
  label,            // 顯示名稱
  emoji,            // 顯示 emoji
  cv,               // 顏色 CSS variable
  waitMax,          // 等待時間上限（可被加時道具修改）
  waitElapsed,      // 已等待秒數
  peeBase,          // 如廁基礎時間
  dirtyAmt,         // 完成後加的髒污量
  score,            // 得分
  isInspector?,     // 馬蓋仙標記
  isDispatched?,    // 派遣客人（費用×2、得分×2）
  _aromaApplied?,   // 芳香劑加成已套用
  _lastAssignTry?,  // 馬蓋仙重試節流時間戳
}
```

---

## 道具 effects 鍵說明

| effect key | 類型 | 說明 |
|------------|------|------|
| `dirtyMult` | float | 髒污乘數（0.85 = 減 15%）|
| `entryFeeBonus` | float | 入廁費乘數 |
| `waitBonus` | float | 等待時間乘數（芳香劑）|
| `ignoreDirtySlow` | bool | 忽略髒污減速 |
| `autoClean` | bool | 清掃助理標記 |
| `repairBroken` | bool | 修復工標記 |
| `addUrinal` | bool | 加馬桶標記 |
| `bulletOp` | bool | 子彈操作標記 |
| `waitBoost` | int | 等待加時秒數 |
| `boostDuration` | int | 加時效果持續秒數 |

---

## BGM 切換邏輯

```
_baseBGM()  = stage >= 14 ? 'abyss' : 'normal'
_surgeBGM() = stage >= 14 ? (peak→'abyss_peak', else→'abyss_surge') : type

觸發高峰         → playBGM(_surgeBGM('peak'), 12000)
觸發 surge       → playBGM(_surgeBGM(type), 10000)
觸發巡查         → playBGM('boss')
巡查/surge 結束  → playBGM(_baseBGM())
進入深淵         → playSFX('abyss_enter') → setTimeout 2500ms → playBGM('abyss')
```

---

## Bug 清單（已知 + Code Review 發現）

### 1. `CONFIG.CLEAN_TIME` 未定義（latent bug）
- **位置**：`mkUrinal`，line ~1593
- **現象**：`cleanDuration` 初始為 `undefined`，但自動清潔碼在觸發前會覆寫為 `0.5`，目前不影響功能
- **建議**：在 CONFIG 加 `CLEAN_TIME:0.5` 或在 mkUrinal 直接寫 `0.5`

### 2. SFX 節點可能被 BGM 切換截斷
- **位置**：`_beep()` / `_noise()` push 到 `_bgmNodes`；`_stopBGM()` 停止全部
- **現象**：BGM 切換瞬間觸發的音效可能被截斷
- **建議**：SFX 用獨立陣列 `_sfxNodes`，不在 `_stopBGM` 時清除

### 3. setTimeout 回呼缺 `state.running` 檢查
- **位置**：`tryAssign` gangster scare（line ~2099）、`applyPhaseStyle` abyss 延遲（line ~1526）
- **現象**：遊戲結束後 1–2.5 秒內，上述回呼仍會修改 state 或切換 BGM
- **建議**：回呼開頭加 `if(!state.running)return;`

### 4. `checkAchievements()` 被呼叫兩次
- **位置**：`installItem`，line 1291 和 1295
- **現象**：無功能影響，輕微效能浪費

### 5. 本地排行榜（#hist-wrap）未渲染
- **位置**：`renderHistory()` 在 line 3421 被呼叫，但 HTML 中沒有 `id="hist-wrap"` 元素
- **現象**：Title 畫面沒有顯示本地 TOP 5（只有全球排行榜）
- **建議**：在 title screen HTML 新增 `<div id="hist-wrap"></div>` 並呼叫 `renderHistory()`

### 6. `dispatchCustomer` 費用硬編碼
- **位置**：line ~3268，`const DISPATCH_COST=5`
- **現象**：按鈕上顯示的 `£5` 和邏輯分離，不受 CONFIG 控制
- **建議**：移至 `CONFIG.DISPATCH_COST`

---

## 過夜重載問題

手機瀏覽器（尤其 iOS Safari）在背景停留過久後會直接卸載頁面，`pagehide` 暫停機制無效。根本解法是實作 mid-game save，見 `SAVE_RULES.md`。
