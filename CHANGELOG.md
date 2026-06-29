# CHANGELOG — 尿斗管理大師

## v5.2.1（目前穩定版）
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
