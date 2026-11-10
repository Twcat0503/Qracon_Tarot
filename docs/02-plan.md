# 02 — 網站設計與建置計劃

**版本**：v2.1（2026-11-10，Gemini 4 第二輪複核後核定）
**狀態**：**已核定**。9 項 P0 全數關閉，見 `04-gemini-review.md`。
**範圍**：只做規劃。本文件不含任何程式碼實作。
**配套**：`01-copy-deck.md`（文案）、`00-source-audit.md`（原文診斷）、`03-open-questions.md`（待決）

---

## 0. 摘要

一個手機優先的單頁互動：七張牌背朝上的塔羅牌扇形展開，使用者點一張，其餘六張碎散，被選的那張翻開，接著展開成一張「觀測報告」票券。全程 30–60 秒，最後把人帶到議程頁與報名頁。

技術上與主站同源（Astro 靜態輸出、零框架、原生 CSS），素材直接沿用主站的字標、躍貓、字型、tokens 與票券元件。新增的只有：牌組資料、扇形互動、七張結果頁、分享卡。

### 需要核定的四個決策

| # | 決策 | 建議 | 狀態 |
| --- | --- | --- | --- |
| D1 | 色彩方向 | **A 暗房（純灰階）**：墨黑場、紙白牌面、銀墨壓紋 | Gemini 4 核准此版本 |
| D2 | 牌面插畫 | 設計組交付 7 張；先用「字體牌」上線 | 核准 |
| D3 | 獎勵系統（QXP／SSS／NFC／冒險者系統） | 未確認前不寫，CTA 只導向議程與 KKTIX | 核准 |
| D4 | 部署位置 | 主網域子路徑 `/tarot`；建置整合方式待決（§11） | 核准，但需先解決雙 repo 部署 |

---

## 1. 目標與非目標

### 目標

1. 讓人在 60 秒內完成一次抽牌，並記住自己抽到哪一張。
2. 抽完之後，有一個明確的下一步：看議程，或報名。
3. 七張牌各自可分享、可被搜尋索引（不是只能用一次的黑箱）。
4. 讀起來像「學生做的量子占卜」，不像企業活動的導流頁。

### 非目標

- 不做會員、登入、集點、排行榜。
- 不做心理測驗量表，也不宣稱任何預測能力。
- 不做 3D、不做 WebGL shader（主站已經有了，這裡不需要第二個）。
- 不做多語系。
- 不做後端。抽牌結果只存在瀏覽器。

---

## 2. 與主站的關係

塔羅站是主站的**附屬頁面**，不是新品牌。三條硬規則：

1. **素材不重畫**。字標、躍貓、favicon、字型、tokens 直接沿用（§7）。
2. **元件同名同義**。`Button`、`Stamp`、`FlapText`、`Ticket`、`Words` 複製過來後保持介面一致。`Button` 的 prop 是 `variant?: 'ink' | 'line'`，塔羅站只新增一個值 `'on-ink'`，不新增 `tone` 這種別名。
3. **不連回主站導覽**。塔羅站只有自己的頁尾，加一行「QRACON 2026 官網 ↗」。反過來，主站之後要加入口（另一個 PR，不在本計劃）。

**複製或共用的決定**：建議把塔羅站做成主站 repo 的 `apps/tarot` 子目錄（§11 D4）。這樣元件不會分叉、`/tarot` 只是 Astro 的 `base`、部署是單一 Pages 專案。若維持兩個 repo，複製的檔案開頭要寫 `// 來源：qracon_web_2026@<commit>`，並在每次主站改 tokens 時手動同步。

---

## 3. 設計方向（D1）

### 選項

**A. 暗房（建議，純灰階版）**

主站是一張印好的紙，塔羅站是暗房裡顯影中的底片。同一個材質系統，反相使用。

- 場：純墨 `#000`（`--color-ink`），不是深藍、不是紫。
- 牌面：紙白 `#FFF`（`--color-white`）——全頁唯一「被點亮」的表面，沿用主站的 Lit Sign Rule。
- 次要字：`#B3B3B3`（`--color-on-ink-soft`，墨上 10:1）；細線 `#4D4D4D`（`--color-on-ink-rule`，裝飾用）。
- 抬升面：`#262626`（`--color-ink-raised`），用於牌背與面板。
- **質感靠銀墨，不靠色相**：牌背的壓紋是一層 105° 的白色高光漸層（`transparent → rgb(255 255 255 / 0.9) → transparent`）掃過，加上 `--color-on-ink-rule` 的干涉線。這是印刷上的銀墨反光，零色相。
- 硬規則：**高光不承載資訊**。把高光關掉，頁面仍然完全可用、對比仍然合格。

**B. 印刷廠（保守備案）**

整個站放在紙灰 `#D9D9D9` 上，塔羅牌當成同一批紙印出來的另一件印刷品。零色相，與主站 100% 一致。

- 優點：不需要任何例外，設計與工程成本最低。
- 缺點：沒有暗場對比，翻牌那一刻的張力不見了。

**C. 賽博霓虹（原文）**

深宇宙藍／塔羅紫底 + 螢光綠／電光藍／暖橘。

- **不採用**。與 `DESIGN.md` 的 Zero Hue Rule 直接衝突；要維護第二套色票與對比表；最大風險是「看起來像另一場活動」，而兩場活動的受眾完全重疊。

### 為什麼不是全像箔

初稿曾建議用「全像箔」當唯一彩色，理由是主視覺上那隻貓有箔的質感。**這個理由是錯的**：實測 `art/QRACON2026主視覺形象圖.jpg` 全圖最高飽和度 0.000（零色相）；`qracon-wordmark.png` 同樣為 0；`schrodinger-cat-leaping.png` 只有 0.05% 的像素有飽和（虹膜上的一點反光），不足以支撐一套材質。依 `DESIGN.md` 的 Zero Hue Rule，全像箔全部移除，改成灰階銀墨。

### 建議

**選 A（純灰階暗房）。** 理由：

1. 它讓「翻牌」有物理意義：亮的是被觀測到的那一張，其餘留在暗處。這是用版面說故事，不是加裝飾。
2. 不新增任何色彩 token。整個塔羅站的顏色都是主站已有的變數，`tokens.css` 仍是唯一來源。
3. 與 B 的差別只在「場景是白天還是暗房」，不是兩套視覺語言，日後要改回 B 只需要換一個底色。

---

## 4. 設計系統

### 4.1 色彩

**不新增任何色彩 token。** 全部使用主站 `src/styles/tokens.css` 的既有變數：

| 用途 | token | 值 | 對比 |
| --- | --- | --- | --- |
| 全頁底色 | `--color-ink` | `#000` | — |
| 牌面、票券 | `--color-white` | `#FFF` | 21:1 on ink |
| 場上主要文字 | `--color-white` | `#FFF` | 21:1 |
| 場上次要文字 | `--color-on-ink-soft` | `#B3B3B3` | 10:1 |
| 場上分隔線 | `--color-on-ink-rule` | `#4D4D4D` | 2.3:1（裝飾） |
| 牌背、抬升面 | `--color-ink-raised` | `#262626` | 裝飾 |
| 牌面上的字 | `--color-ink` | `#000` | 21:1 |
| 牌面次要字 | `--color-ink-soft` | `#4D4D4D` | 8.5:1 |

**銀墨規則**：牌背與坍縮瞬間的高光，只能是白色透明漸層或明度變化。不得出現任何 `hue` 值、不得寫 `--color-accent`、不得用 `conic-gradient` 做彩虹。

### 4.2 字型

不新增字族。Noto Sans TC（400–900）+ Barlow Condensed（600/800），設定沿用主站 `astro.config.mjs` 的 Astro Fonts API。

塔羅感用**排法**做，不用字型做：

- 牌名**直排**（`writing-mode: vertical-rl`），像牌面兩側的標題帶。
- 羅馬數字用 Barlow Condensed 800，字距 `0.08em`，大小是牌寬的 18%。
- 中文標題 900 字重、`letter-spacing: 0.02em`；不用書法體、不用襯線體。

**The Latin-Only Sign Rule（沿用主站）**：`var(--font-sign)` 只給拉丁字母與數字（羅馬數字、日期數字、屬性數值）。出現中文或全形括號就切回 `var(--font-sans)`——例如「2026.12.13（日）」的「（日）」。

### 4.3 版面

- 12 欄格線，`--page-max: 88rem`，`--gutter` 沿用主站。
- 斷點沿用：72rem / 48rem / 34rem。
- 首頁：牌扇置中，垂直居中偏上；提示文字在牌扇上方 `--space-5`。
- 結果頁：單欄，最大 `46rem`；票券滿寬（手機 bleed 到 gutter）。

### 4.4 牌面規格

| 項目 | 值 |
| --- | --- |
| 比例 | 2 : 3.5（70×120mm 標準塔羅） |
| 圓角 | `--radius-chip`（4px） |
| 外框 | `var(--rule-thin)` 墨線 |
| 內框 | `var(--rule-hair)` 墨線，內縮 `var(--space-2)` |
| 牌背 | `--color-ink-raised` 底 + 白色高光掃光 + `--color-on-ink-rule` 干涉線；字標「O」紋樣 45° 重複 |
| 牌面底 | `--color-white` |
| 陰影 | 無。牌靠白面本身發亮（主站 One Ticket Rule） |

牌扇在 375px 寬：牌寬 `clamp(88px, 26vw, 132px)`，角度 ±21°，每張水平偏移 6vw，垂直位移依 `sin` 排成弧。
桌機：牌寬 200px，角度 ±18°，水平偏移 72px。

### 4.5 元件

| 元件 | 來源 | 改動 |
| --- | --- | --- |
| `Button` | 主站 | `variant` 新增 `'on-ink'`（白框白字），不新增 `tone` |
| `Stamp` | 主站 | 直接用於「你剛剛抽到」 |
| `FlapText` | 主站 | 結果頁的羅馬數字用 `mode="view"` 翻一次 |
| `Ticket` | 主站 | 抽出 `TicketShell`（去掉場次依賴），結果頁用 |
| `Words` | 主站 | 牌義長句斷行 |
| `Card`（新） | — | 牌面／牌背，含焦點與 reduced-motion |
| `AttributeBar`（新） | — | `var(--rule-heavy)` 線條進度條，四軸 |

屬性**不用雷達圖**。雷達圖是企業儀表板語彙，七邊形在手機上讀不出數值。用四條線：軌道 `--color-on-ink-rule`，填充 `--color-white`，右側接數字（Barlow，tabular-nums）。

### 4.6 屬性四軸

| 軸 | 意思 | 為什麼是這四個 |
| --- | --- | --- |
| 疊加 | 同時保有幾種可能 | — |
| 糾纏 | 跟別人的狀態綁得多緊 | — |
| 相干 | 狀態多整齊、能不能干涉 | — |
| 能階 | 處在多大的能量上 | — |

初稿的第四軸是「退相干」，與「相干」在定義上反相關（退相干就是相干性的消逝），兩軸並列會出現「相干 60／退相干 95」這種自相矛盾的數字。改成「能階」後四軸互相獨立。

七張牌的數值見 `01-copy-deck.md` §4。介面不得用「高分／低分」暗示優劣：能階高不是好事也不是壞事，它只是描述。

---

## 5. 資訊架構

所有站內路徑都掛在 `import.meta.env.BASE_URL`（`/tarot/`）底下，不得寫死根路徑。

```
/tarot/                    抽牌（洗牌 → 坍縮 → 導向結果）
/tarot/deck               七張牌總覽
/tarot/deck/[slug]        單張牌＝結果頁（?draw=1 顯示「你剛剛抽到」）
/tarot/about              這是什麼、名詞對照、製作、使用限制
```

七個 slug：`fool`、`lovers`、`hermit`、`wheel`、`death`、`tower`、`sun`。

不做 `/result/[slug]`：結果頁就是牌義頁，避免兩份同樣的內容稀釋 SEO。

### 導覽

只有一條：左上字標（回首頁）、右上「牌組」「關於」。手機收成兩顆文字按鈕，不做漢堡選單。

### SEO

- 每張牌頁有獨立 `<title>`、description、OG 圖。
- 結構化資料：`WebSite` + 每頁 `Article`（不用 `Quiz`，這不是測驗）。
- `robots.txt` + `sitemap.xml`，`base` 由 Astro 處理。

---

## 6. 互動規格

### 6.1 狀態機

```
idle ──pointerenter──▶ hover（牌抬起 4px、轉正 60%）
idle ──activate─────▶ resolving（鎖輸入，prefetch 同時開始）
                        ├ 其餘 6 張：碎裂 520ms
                        ├ 被選的牌：轉正 → 放大 1.08 → 亮邊 180ms
                        └ 翻牌 600ms（rotateY 180°）
resolving ──end─────▶ navigating（View Transition 到 /tarot/deck/[slug]?draw=1）
result ──重新抽一張──▶ 重新載入 /tarot/（重新洗牌）
```

`resolving` 期間：`pointer-events: none`、`aria-busy="true"`、`inert` 套在牌扇容器上。

**預取（R11）**：點擊的 0ms 就對目標頁插 `<link rel="prefetch">`，並預載該張牌面圖。不要在 1300ms 動畫結束後才開始請求——4G 訊號差的時候會卡在翻開那一格。

### 6.2 選牌：洗位置，不是抽亂數

七張牌背是同一張圖，牌的位置才是隨機來源。

1. 頁面載入時，用 `crypto.getRandomValues` 對七個 slug 做 Fisher–Yates 洗牌。
2. 把洗好的 slug 寫回七個 `<a>` 的 `href`（DOM 順序固定，內容隨機）。
3. 使用者點哪一張，就開那一張的 `href`。**沒有另外的亂數**——使用者的選擇就是唯一的隨機。
4. 「重新抽一張」＝重新載入頁面，重新洗一次。

**無 JS 時**：`<a href>` 維持建置時的固定順序（fool → sun），仍然七張都能開，只是位置不隨機。

不做 `sessionStorage` 鎖定：結果頁本身就是 URL，按上一頁回來看到的是同一頁；抽牌頁重洗不影響已抽到的結果。

### 6.3 碎裂

不用 WebGL。六張牌各自切成 6 片 `clip-path` 多邊形（共 36 個元素），每片給不同的 `--tx/--ty/--rot`，`transform: translate3d() rotate()` + `opacity` 520ms，`--ease-out`。另加一層 Canvas2D 塵埃（最多 120 粒，1 秒後停止並移除 canvas）。

理由：`interference.ts` 是全螢幕 WebGL2 shader，用在「六張小卡碎掉」是殺雞用牛刀，低階手機會掉幀。主站的首屏值得那個成本，這裡不值得。

### 6.4 鍵盤與螢幕閱讀器

- 七張牌是 `<a>`，包在 `<ol>` 裡，`aria-label="第 1 張牌，牌背"`。
- 點擊時 `preventDefault()` 播動畫；**沒有 JS 就正常跳頁**（漸進增強，也是 SEO 的底）。
- Tab 順序 = 牌序；焦點框 `var(--rule-heavy)` 白框外推 3px。
- 結果頁 `?draw=1` 時，`<h1>` 是角色名，並用 `aria-live="polite"` 宣告「你抽到觀測者冷酷貓」。

### 6.5 Reduced motion

| 關閉的 | 改成 |
| --- | --- |
| 牌扇進場與展開 | 直接顯示最終位置 |
| 碎裂 | 六張 120ms 淡出 |
| 翻牌 | 交叉淡化 |
| 粒子 | 不建立 canvas |
| View Transition | 關閉，直接跳頁 |
| 銀墨掃光 | 不播 |

`prefers-reduced-motion: reduce` 下整段流程 ≤ 300ms。

### 6.6 重抽

- 結果頁有「重新抽一張」，回到 `/tarot/` 重洗。
- 同一 session 內允許無限重抽（沒有集點，沒有理由限制）。
- 不做「今天已經抽過」的鎖——那會讓分享連結進來的人卡住。

### 6.7 回訪

結果頁載入時寫 `localStorage['qracon.tarot.last'] = { slug, at }`。首頁讀到就顯示：

```
你上次抽到：觀測者冷酷貓
[ 再看一次 ]  [ 重新抽一張 ]
```

---

## 7. 牌面資產（D2）

### 7.1 設計組交付規格

| 項目 | 規格 |
| --- | --- |
| 張數 | 7 |
| 尺寸 | 1440×2520 px（2:3.5，300dpi 級） |
| 格式 | PNG 去背或滿版，另附 SVG 線稿 |
| 單張上限 | 400 KB（網站用 AVIF） |
| 安全區 | 四周內縮 8% 不放主體（對應內框） |
| 風格 | 與 `art/QRACON2026主視覺形象圖.jpg` 同一支筆：手繪黑貓、墨線、**純灰階** |
| 禁止 | 漸層背景、寫實 3D、彩色寶石、傳統偉特牌構圖直接挪用 |

原文提到「傳統塔羅牌面會放旁邊」——那是給插畫師的參考，**網站不顯示傳統牌面**（著作權與風格衝突）。

### 7.2 字體牌（插畫未到位時）

插畫是設計組的交付物，網站不能等它。先用「字體牌」上線，插畫到了只換圖，不動版面：

```
┌──────────────────┐
│  1.5px 外框       │
│  ┌────────────┐  │
│  │    IX      │  │  ← Barlow 800，牌寬 18%
│  │  ┌──────┐  │  │
│  │  │ 圖示 │  │  │  ← 40×40 SVG 線稿
│  │  └──────┘  │  │
│  │   觀      │  │  ← 直排牌名，Noto 900
│  │   測      │  │
│  │   者      │  │
│  └────────────┘  │
└──────────────────┘
```

**首頁牌扇的牌面只放三件事**：羅馬數字、圖示、直排牌名。牌寬最小 88px，塞不下一句定義——13px 的中文在這個尺寸低於主站 `--text-xs` 的可讀下限。「一句定義」只出現在結果頁的大尺寸牌面。

七個 SVG 圖示（全部 `var(--rule-thin)` 線稿，墨色）：

| 牌 | 圖示 |
| --- | --- |
| 愚者 | 兩條相位不同的正弦波重疊 |
| 戀人 | 兩個點，中間一條來回震盪的線 |
| 隱士 | 一條波被垂直線切斷，右側只剩一個點 |
| 命運之輪 | 三條水平能階 + 一支向上的箭頭 |
| 死神 | 干涉條紋的明暗對比逐漸衰減，最後變成均勻灰階 |
| 塔 | 一條線突然階梯上升，接著劇烈震盪 |
| 太陽 | 兩條同相位正弦波疊成加粗的一條 |

字體牌是**成品**，不是灰塊佔位——它本身就符合品牌（純黑白、印刷感、直排）。插畫是加分，不是前提。

---

## 8. 資料模型

```ts
export type CardId = 'fool' | 'lovers' | 'hermit' | 'wheel' | 'death' | 'tower' | 'sun';

export interface TarotCard {
  id: CardId;
  arcana: number;            // 0, 6, 9, 10, 13, 16, 19
  numeral: string;           // '0' | 'VI' | ...
  name: string;              // 愚者
  nameEn: string;            // The Fool
  persona: string;           // 疊加態混亂人
  personaEn: string;         // Superposition Chaos
  concept: {
    zh: string;              // 疊加態
    en: string;              // Superposition
    line: string;            // 一句定義（≤ 24 字）
    note: string;            // 科普註（≤ 70 字）
  };
  art: string | null;        // 插畫 import；null → 字體牌
  reading: {
    fortune: string;         // 最近的運勢
    quote: string;           // 牌義引言
    traits: [string, string]; // 兩條行為
  };
  partner: { id: CardId; why: string };
  rival: { id: CardId; why: string };
  stats: { superposition: number; entanglement: number; coherence: number; energy: number };
}
```

`src/data/cards.ts` 是唯一來源；結果頁、`/deck`、OG 產生腳本都從它推導。文案改動只改這個檔。

驗證（`astro check` 抓不到的）：

- `id` 七個齊全、不重複。
- `partner` / `rival` 的 `id` 必須存在，且不等於自己。
- `stats` 四項 0–100。
- `numeral` 與 `arcana` 對應。
- `concept.note` ≤ 70 字、`concept.line` ≤ 24 字。

寫成 `validateCards()`，在 build 時跑，錯就讓 build 失敗。

---

## 9. 分享與 OG

### 9.1 OG 圖（7 張）

1200×630，每張一個檔案：`public/og/fool.png` … `sun.png`。

產生方式：`scripts/build-og.mjs`，用 **sharp**（已經是直接依賴）把一段 SVG 轉 PNG。SVG 裡是：墨底 + 字標 + 羅馬數字 + 角色名 + 一句定義。

- 字型：SVG 內指定 `Noto Sans TC, Microsoft JhengHei, sans-serif`。**在本機執行一次，把 PNG commit 進 repo**，不在 CI 產生（CI 沒有這些字型，輸出會不一致）。
- 重跑條件：`cards.ts` 文案或牌面插畫變更時。

### 9.2 站內分享

- 「分享」按鈕：`navigator.canShare({ files })` 為真才顯示，呼叫 `navigator.share()`。
- 「下載分享卡」：Canvas 2D 產 **1080×1920（9:16）** PNG，`<a download>`。
  - 學生主要在 IG 限時動態分享，9:16 優先；4:5 列為第二版。
  - 上下各留 15% 安全區，避開 IG 介面。
  - 字型：**不能寫死 `'Noto Sans TC'`**。Astro Fonts API 會給字族名加 hash，先用 `getComputedStyle(document.documentElement).getPropertyValue('--font-sans')` 取實際字族名，再用 `document.fonts.load()` 等它載入，否則會拍到 fallback 字。
  - 內容：墨底、牌面、羅馬數字、角色名、一句定義、QR Code、活動日期地點。

### 9.3 In-App WebView（R10）

受眾從 IG 限時動態或 LINE 點進來。IG／LINE 的內建瀏覽器常常讓 `<a download>` 對 `blob:` 沒反應，也不支援 `navigator.canShare({ files })`。

對策：偵測 UA 含 `Instagram`／`Line` 時，下載後（或直接）在畫面彈出 `<img>` 預覽層，提示「若沒有自動下載，長按圖片存到相簿」。

### 9.4 QR Code

`sharp` 只會把 SVG 點陣化，沒有 QR 矩陣（Reed–Solomon）編碼器，所以不能「用 sharp 產生 QR」。

做法：用一次性腳本（devDependency `qrcode`，只給 `scripts/` 用，不進 bundle）產生 7 張 PNG，commit 進 `public/qr/`。前端只載 PNG 疊圖。
QR 內容是該張牌的完整網址（含 `/tarot` 前綴），產生時就要用正式網域。

---

## 10. 效能與無障礙

### 預算

| 項目 | 上限 |
| --- | --- |
| 首頁 JS（gzip） | 35 KB |
| 結果頁 JS（gzip） | 20 KB |
| 單張牌面插畫（AVIF） | 90 KB |
| 首頁總重（首次載入） | 350 KB |
| LCP（4G，中階 Android） | 2.0s |
| CLS | < 0.05 |
| INP | < 200ms |

沒有 WebGL、沒有 Three.js、沒有動畫函式庫。字型走 Astro Fonts API 的 unicode-range 切片。

### 無障礙（WCAG 2.2 AA）

- 所有牌可 Tab 到、可 Enter 開啟；焦點框 `var(--rule-heavy)`、外推 3px。
- 顏色不承載狀態（主站規則沿用）。
- 動畫全部可被 `prefers-reduced-motion` 關閉。
- 牌扇的旋轉是裝飾：螢幕閱讀器讀 `<ol>` 順序，不讀視覺角度。
- 結果頁的「你抽到 X」用 `aria-live`。
- 對比：白/墨 21:1、`#B3B3B3`/墨 10:1、`#4D4D4D`/白 8.5:1。

---

## 11. 分期、部署與驗收

### 部署架構（R9）

`/tarot` 子路徑需要先決定建置怎麼合併，否則兩個 Pages 專案會互相覆蓋根目錄。

| 方案 | 做法 | 評價 |
| --- | --- | --- |
| **A（建議）** | 塔羅站成為主站 repo 的 `apps/tarot`，單一建置、單一 Pages 專案 | 元件不分叉、`base: '/tarot'` 直接生效。代價是要搬一次 repo |
| B | 維持兩個 repo，主站 CI 拉塔羅站的建置產物放進 `dist/tarot/` | 需要 cross-repo token，兩邊版本要對齊 |
| C | 獨立子網域 `tarot.entangletech.tw` | 最簡單，但 SEO 權重不共用，`base` 也不用設 |

維持「子路徑 `/tarot`」的決定，但方案在 P0 選定。**不接受**把塔羅站直接部署到根目錄。

### 分期

| 階段 | 內容 | 估時 | 完成定義 |
| --- | --- | --- | --- |
| P0 決策 | D1–D4 拍板；部署方案；插畫排程；獎勵制度確認 | 1 天 | `03-open-questions.md` 清空阻擋項 |
| P1 骨架 | Astro 初始化、複製 tokens/字型/元件、`cards.ts`、路由、SEO | 2 天 | `pnpm build` 出 10 頁，`/tarot/` 可開 |
| P2 抽牌 | 牌扇、洗位置、狀態機、鍵盤、無 JS 備援、reduced motion、prefetch | 3 天 | 鍵盤全程可完成抽牌 |
| P3 結果 | 票券、屬性條、契合度、CTA、字體牌 | 3 天 | 七張牌頁皆可讀 |
| P4 分享 | OG ×7、9:16 分享卡、Web Share、WebView 退路、QR | 2 天 | iOS/Android 各下載一次；IG WebView 測過 |
| P5 打磨 | 效能、a11y、真機、文案校對 | 2 天 | 預算全數達標 |
| P6 上線 | 部署、`_headers`、sitemap | 1 天 | 正式網域可開 |

### 驗收清單

- [ ] `pnpm check`、`pnpm build` 通過
- [ ] 1440 / 375 兩個寬度實際看過（牌扇、結果頁、分享卡）
- [ ] 關掉 JS 仍能點牌進結果頁
- [ ] 只用鍵盤完成一次抽牌
- [ ] `prefers-reduced-motion: reduce` 全程 ≤ 300ms 且無位移
- [ ] 首頁 JS ≤ 35 KB gzip
- [ ] Lighthouse：Accessibility ≥ 95、Performance ≥ 90（手機模擬）
- [ ] 七張 OG 圖在 Facebook / X / LINE 各預覽一次
- [ ] IG 與 LINE 內建瀏覽器各開一次，分享卡能存
- [ ] `01-copy-deck.md` 的文案 100% 上線，沒有殘留 emoji
- [ ] 物理名詞由校稿者簽過
- [ ] 全站掃過：沒有裸色碼、沒有 `hue`、沒有 `conic-gradient`

---

## 12. 風險

| # | 風險 | 影響 | 對策 |
| --- | --- | --- | --- |
| R1 | 七張插畫延遲 | 上線卡住 | 字體牌先上，插畫後補（§7.2） |
| R2 | 主視覺印的是 12/5，實際是 12/13 | 對外資訊錯誤 | 塔羅站不裁用主視覺的日期區；日期只從 `site.ts` 取 |
| R3 | 獎勵系統不存在 | 文案承諾跳票 | D3；未確認前不寫 |
| R4 | 「占卜」被讀成偽科學 | 觀感 | 免責 + 每個名詞一句正確解釋 + 不談吉凶 |
| R5 | 色彩方向與主站衝突 | 品牌分裂 | D1 已改為純灰階，衝突解除 |
| R6 | 分享卡字型在 CI 不一致 | 圖壞掉 | 本機產生、commit PNG；前端用 `--font-sans` 取值 |
| R7 | 元件從主站複製後分叉 | 兩站不一致 | 方案 A（同 repo 子目錄）優先 |
| R8 | 塔羅在台灣的宗教聯想 | 受眾排斥 | 定位「娛樂 + 科普」，不做吉凶斷言 |
| **R9** | **雙 repo 共用 `/tarot` 子路徑，部署互相覆蓋** | **主站被蓋掉** | §11 部署架構，P0 選定；不接受根目錄部署 |
| **R10** | **IG／LINE WebView 擋下載與 Web Share** | **分享失效** | §9.3，長按存圖的退路 |
| **R11** | **1300ms 動畫結束才請求目標頁，4G 下卡住** | **轉場失敗** | §6.1，點擊 0ms 就 prefetch |

---

## 13. 待決事項

見 `03-open-questions.md`。擋住 P1 的是：部署方案（R9）、獎勵制度、插畫排程。
