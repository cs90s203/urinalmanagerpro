# CHANGELOG — 尿斗管理大師

## v5.3.1（目前穩定版）
- **Bug fix**：加馬桶等道具在「折扣後價格 < 原價」情境下（例如加馬桶首購五折）無法拖曳安裝——`startItemDrag()` 誤用未打折的 `def.price` 判斷能不能拖，跟按下當下用的 `effectivePrice(def)` 不一致，改成統一用 `effectivePrice(def)`
- **Bug fix**：手機瀏覽器拖曳道具/客人到便斗時，偶發被系統原生滾動手勢搶走，導致拖曳失敗——`.shop-card`、`.cc` 加上 `touch-action:none`，從 CSS 層面阻止瀏覽器把觸控當滾動處理
- 移除已停用的 GA4 追蹤碼（含文件內殘留說明）
- 加入 LICENSE（CC BY-NC-ND 4.0）與程式碼版權宣告註解，準備公開分享用

## v5.3.0
- 道具動態定價：分數達 7000 起 ×2.5，之後每階線性 +0.75（原本規劃 ×1.5 疊乘會在滿階衝到 ×42.7，太誇張，改成線性避免指數爆炸）
- 加馬桶首購 50% 折扣，含專屬 SALE 標示 UI
- 高峰間隔隨階段加速（`PHASE_SPEED_MULT` 套用到 `PEAK_INTERVAL`），呼應「後期沒難度」的回饋
- **Bug fix**：手動暫停後遊戲偶爾仍在背景運行——`gameTick` 補上 `_manualPaused` 守門
- **Bug fix**：高峰觸發的波次（阿兵哥/中二生）密集時會互相重疊，導致客人瞬間湧入被氣走——高峰鎖定時間改為涵蓋波次總長度，並統一巡查結束後的波次重排間隔
- README 改寫為英文，準備公開分享

## v5.2.5
- **Bug fix**：背景切換時若正在拖曳道具/客人，`itemDrag.active`/`drag.active` 會卡住鎖死操作——`_pauseGame()` 補上 `endDrag()`/`endScrub()`/`endItemDrag()`
- **Bug fix**：暫停 overlay 的 pointerdown/pointerup 監聽器每次呼叫 `_showManualPauseScreen()` 都重新綁定一次，多次暫停後疊加成 N 次觸發——改成只在 overlay 第一次建立時綁定一次

## v5.2.4
- 暫停畫面 UI 改版：拿掉「▶ 繼續」按鈕，改成點擊畫面任意位置解除暫停（拖曳/長按不算點擊，不會誤觸發）
- 「點擊任意位置繼續」文字與框線間距調整為 12px

## v5.2.3
- 背景切換的自動暫停與手動暫停（⏸ 按鈕）統一使用同一個暫停畫面
- 暫停畫面新增「囉唆馬蓋仙」道具說明卡片、「📲 加到主畫面，確保存檔」提示卡片
- **Bug fix**：`restoreGame()` 還原存檔後等待區客人卡片不會顯示（`buildQueueDOM()` 清空 DOM 後沒有重建卡片元素）

## v5.2.2
- **Bug fix**：暫停按鈕解除暫停後顏色變黑——`pauseBtn.style.color=''` 清空 inline style 後沒有正確落回 `var(--muted)`，改成明確賦值
- **Bug fix**：子彈時間中等待區倒數沒有變慢——`timeScale` 判斷條件誤用 `state.itemPreview`，漏了 `state.bulletTime`，改成 `(state.bulletTime||state.itemPreview)`
- **Bug fix**：手動暫停後解除暫停 BGM 不會恢復——`_stopBGM()` 會清空 `_currentBGM`，新增 `_pausedBGM` 在暫停當下記住原本播放的 BGM，恢復時優先用它

## v5.2.1
- 深淵模式（Stage 14，24500分）：全 UI 暗化、專屬 BGM `abyss`/`abyss_peak`/`abyss_surge`
- 深淵 BGM 新增三首：`_loopAbyss`、`_loopAbyssPeak`、`_loopAbyssSurge`
- 深淵台詞系統：進入後每 30 秒輪播 `abyss_1/2/3`
- 深淵成就 🌑，結算專屬 feedback 三選一
- 結算畫面：成就 rainbow 🌈 在上、abyss 🌑 在下的排列

## v5.1.x（馬蓋仙系統穩定化）

### v5.1.7+（三層保護修正）
- **Bug fix**：馬蓋仙有時永久卡在等待區不進便斗
- `tryAssign` 加回傳值檢查（失敗不清空 `inspectionCid`）
- 馬蓋仙每秒重試節流（`_lastAssignTry`）
- 超時 20 秒或所有格不可用 → 強制開始掃描
- 馬蓋仙免疫加時道具（`!c.isInspector` 條件）
- 馬蓋仙可繞過黑尿幫鎖定（`pendingC.isInspector` 特例）

## v5.0.x（特殊角色系統）
- 黑尿幫（🕴️）：動態鎖定相鄰格，不付錢，10% 機率贓款 £300
- 中二生（🧑‍🎤）+ 阿兵哥（💂）Surge 波次系統（`pendingWaves`）
- 派遣系統（來人啊！）：£5 觸發 Surge，客人帶 dispatch-glow 且費用×2
- 滴鞋老（👴）：相鄰額外減速 `ELDER_NEIGHBOR_SLOW`
- 子彈操作道具（⚡）：子彈時間內可操作 30 秒

## v4.x（商店系統）
- 道具商店：標靶貼🎯、蒼蠅貼🪰、芳香劑🌸、清掃助理🧹、修復工🔧
- 加馬桶🚽（上限 6 格）
- 等待時間道具：報紙📰、漫畫📖、理髮💈
- 6 格時道具縱向堆疊（flex-direction:column）
- 拖曳道具安裝系統（mouse + touch）

## v3.x（成就 & 圖鑑）
- 成就系統 7 個（🌈🌎🚨🚀🚫✌️）
- 角色圖鑑（CHAR_ORDER）
- 全球排行榜（Firebase Realtime DB）
- 本地 TOP 10 歷史（urinal_history_v1）

## v2.x（摩擦清潔 & 暫停）
- 摩擦清潔：滑動手指 -15%，越髒費用越高
- 背景暫停系統（visibilitychange + pagehide/pageshow 雙重備援）
- 暫停遮罩（⏸ 點擊繼續）

## v1.x（核心玩法）
- 拖曳客人 + 點選帶位
- 2 種基本客人（來都來了、頻尿人）
- 髒污系統、損壞系統
- 基礎 BGM（Web Audio API 合成）
- 馬蓋仙台詞系統（typewriter effect）
