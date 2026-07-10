# HANDOFF — 架構與修改指南

一行摘要：`index.html` 分五區（資料／卡片佈局繪製／互動／說明欄／尺寸詳圖檢視器），改東西前先找對區。

## index.html 內部結構（由上到下）

```
<style>        版面 CSS + SVG 卡片樣式 + 尺寸詳圖檢視器樣式
<header>       搜尋框 #search、分類色塊 #chips、縮放鈕 zin/zout/zfit
<main>         左 #stage>svg#canvas>g#world（卡片牆）＋右 aside（說明欄三分頁）
#viewer3d      全螢幕 overlay：頂欄＋速查數字條＋切換鈕＋剖面詳圖／尺寸表／立體 canvas
<script>
  ① 資料區     BSP / NPT / METRIC / UNIFIED 原始表 → GROUPS 統一結構
  ② 卡片繪製   render()：每群組一橫排卡片，圓依 SCALE=1.5 px/mm 真實比例
  ③ 互動       fit()/zoomAt() + pointer 事件（拖曳、雙指縮放）＋ applyFilter() 搜尋
  ④ 說明欄     select() 點卡片 → renderPanel()；mat/id 兩個固定分頁；含開詳圖按鈕
  ⑤ 尺寸詳圖   threadGeom()＋buildProfileSVG()＋buildHero()＋buildDimTable()＋setView()
              ＋buildMesh()/renderV3D()（立體牙紋）
</script>
```

## 尺寸詳圖檢視器（本專案的核心說明工具）

點卡片右欄「📐 尺寸詳圖＋3D 立體檢視」→ `open3D()` 開全螢幕 overlay `#viewer3d`。
**設計原則：旋轉的 3D 螺桿讀不出牙角(55°/60°)與牙距——這些只能標在軸向剖面詳圖上。**
所以主角是 2D 剖面詳圖，3D 只當「立體長相」的輔助。詳見 [lessons/3d-cant-show-thread-angle.md](lessons/3d-cant-show-thread-angle.md)。

overlay 由上到下：
1. **頂欄** `#v3dbar`：規格名 + 俗稱色塊 `#v3dtag` + `#v3dinfo`(標稱) + ✕ 關閉。
2. **速查數字條** `#v3dhero`：`buildHero()` 產 5 磚（外徑／牙距＋TPI／牙角／底孔／錐直），手機 2 欄、桌機 5 欄。
3. **切換鈕** `#v3dviews`：原生 button「📐 牙型剖面詳圖」(預設)／「🧊 立體牙紋」→ `setView('profile'|'solid')`。
4. **牙型剖面詳圖** `#v3dprofile`：`buildProfileSVG()` 回傳的純 SVG 字串（viewBox 0 0 1000 740）。
5. **圖例列** `#v3dlegend` + **尺寸總表** `#v3dtable`（`buildDimTable()`，附標準依據 `STD[]`）。
6. **立體牙紋** `#v3dsolid > canvas#v3dcanvas`：`buildMesh()`＋`renderV3D()` 軟體渲染。

### 幾何全部集中在 threadGeom(item, group)（已驗證，勿改）
- 牙角 α：PT/PF=55°（Whitworth），NPT/M/UN=60°（ISO/UN）
- 理論三角高 H = P/(2·tan(α/2))
- **牙深 h = 0.6403·P（55°）／0.6134·P（60°）**（已對 M10×1.5→小徑8.16、PT½→18.63 驗證吻合）
- 小徑 d₁ = 大徑 − 2h；中徑 d₂ = 大徑 − h
- 錐牙錐度 1:16（沿直徑），單邊錐角 = atan(1/32) ≈ 1.79° ≈ 1°47′
> 改牙深係數＝同時改 buildProfileSVG（剖面）與 buildMesh（立體），兩者同源，別只改一個。

### buildProfileSVG 兩區塊
- **主視圖**（上下鏡射縱剖）：牙峰/牙底鋸齒＋45°剖面線；牙距(橘)在頂、外徑/底徑(藍)在右、
  底孔與錐度/平行註記(灰/紫)在下。徑向為「放大示意」（外徑差 3～114mm 無法與細牙同比例），但**數字是真值**。
- **牙型放大詳圖**（真實牙角）：畫 2 顆牙，兩側面夾角＝真實 55°/60°（角度弧誠實），標牙角(綠)與牙深 h(綠)。
- 標註分群色：藍=直徑、橘=牙距、綠=牙型/牙角、紫=錐度、灰=軸線/資訊；箭頭用 `<marker orient="auto-start-reverse">`。

### 立體牙紋為什麼以前像光滑圓桶、怎麼修好的
- 舊：牙深 H×0.72、圓周 52 段、無 AO ＋ 近端面視角 → 牙紋被抹平。
- 新：牙深視覺放大 `depth = h×1.9`（示意，明標）、圓周 `NT=80`、加**假環境光遮蔽 AO**
  （`occl` 依局部半徑：牙峰亮牙底暗，與視角無關）＋拉大灰階對比。這是牙紋浮出的關鍵。
- 3D 迴圈只在切到「立體牙紋」時 `setInterval(stepV3D,33)`，切走/關閉 `clearInterval`（省電）。

## 常見修改怎麼做

### 加一筆規格（同類）
在對應原始表加一列，卡片與詳圖全自動生成：
- 管件牙：`BSP` 或 `NPT` 加 `{size:'5/8', od:22.911, tpi:14}`，並補 `PIPE_ALIAS`、`RC_DRILL`/`NPT_DRILL`。
- 公制：`METRIC` 加 `[36, 4.0, '3.0']`（外徑、粗牙距、細牙字串）。
- UNC/UNF：`UNIFIED` 加 `['1-1/8"', 28.575, 7, 12, 25.0, '9分牙']`。

### 加一整類螺牙
`GROUPS` 加物件（key/color/bg/title/sub/seal/note/use/items）；`STD` 補該類標準字串。
色塊、卡片、詳圖、尺寸表全自動跟上。

### 改詳圖顯示
- 剖面幾何/標註 → `buildProfileSVG()`；速查磚 → `buildHero()`；尺寸表列 → `buildDimTable()`。
- 立體牙紋視覺 → `buildMesh()`（depth 倍率、NT）與 `renderV3D()`（AO、灰階對比）。

## 已知限制

1. 未含牙種：BSW/BSF 惠氏機械牙、Tr 梯形牙、ACME、自攻牙、木牙 — 需求出現再加。
2. NPT 2-1/2" 以上底孔沒列（大口徑實務直接買管件不攻牙），詳圖與速查顯「—」。
3. 剖面主視圖「徑向」是放大示意（非等比），已在圖上明標「放大示意，數字為實際尺寸」。
4. 立體牙紋牙深是視覺放大（×1.9），已在圖下明標「立體示意」；精確尺寸一律以剖面詳圖／尺寸表為準。
5. Claude 的 Browser 預覽在此環境 `window.innerWidth/Height=0`，`vh/%` 版面歸零、canvas 無法取像素，
   `preview_screenshot` 也逾時。驗證改用：SVG 走 getBBox/DOM 座標（viewBox 與視窗無關）、
   canvas 測試需暫時強制固定 px 尺寸。見 [lessons/zero-viewport-headless-preview.md](lessons/zero-viewport-headless-preview.md)。

## 驗證清單（改完必跑）

1. `python -m http.server 8799` 開 `http://localhost:8799`
2. 初始畫面＝五大類全覽（不是空白、不是 scale(0)）
3. 搜尋「4分」→ PT/PF/NPT 1/2 與 4分牙；「21」→ 外徑 21mm 附近；「M10」→ 只有 M10
4. 點卡片 → 右欄規格表；點「📐 尺寸詳圖」→ overlay 預設顯示剖面詳圖
5. 剖面詳圖數值：M10×1.5 小徑=8.16、PT½ 小徑=18.63；放大詳圖畫出的牙角＝真實 55°/60°（可用 getBBox 量兩側面夾角）
6. 錐牙有「錐度 1:16」＋三角示意；平行牙標「平行牙・無錐度」；底孔 null 顯「—」
7. 切「🧊 立體牙紋」→ 牙紋明顯（掃描線多次明暗起伏，非光滑圓桶）；切回/✕ 關閉 setInterval 已 clearInterval
8. F12 console 無錯誤
