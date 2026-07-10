# HANDOFF — 架構與修改指南

一行摘要：`index.html` 分四區（資料／佈局繪製／互動／說明欄），改東西前先找對區。

## index.html 內部結構（由上到下）

```
<style>        版面 CSS + SVG 卡片樣式（.card / .dimmed / .selected）
<header>       搜尋框 #search、分類色塊 #chips、縮放鈕 zin/zout/zfit
<main>         左 #stage>svg#canvas>g#world（畫布）＋右 aside（說明欄三分頁）
<script>
  ① 資料區     BSP / NPT / METRIC / UNIFIED 原始表 → GROUPS 統一結構
  ② 佈局繪製   render()：每群組一橫排卡片，圓依 SCALE=1.5 px/mm 真實比例
  ③ 互動       fit()/zoomAt() + pointer 事件（拖曳、雙指縮放）＋ applyFilter() 搜尋
  ④ 說明欄     select() 點卡片 → renderPanel()；mat/id 兩個固定分頁
</script>
```

## 常見修改怎麼做

### 加一筆規格（同類）
在對應原始表加一列即可，render 全自動：
- 管件牙：`BSP` 或 `NPT` 陣列加 `{size:'5/8', od:22.911, tpi:14}`，並在 `PIPE_ALIAS`、`RC_DRILL`/`NPT_DRILL` 補俗稱與底孔。
- 公制：`METRIC` 加 `[36, 4.0, '3.0']`（外徑、粗牙距、細牙字串）。
- UNC/UNF：`UNIFIED` 加 `['1-1/8"', 28.575, 7, 12, 25.0, '9分牙']`。

### 加一整類螺牙（如 BA、Tr 梯形牙）
在 `GROUPS` 加一個物件：`key/color/bg/title/sub/seal/note/use/items`。
items 每筆必填 `name, alias, od, odIn, tpi, pitch, drill, taper, desig`（`extra` 選填）。
記得色塊 chips 是照 GROUPS 自動生成的，不用另外改。

### 改卡片上顯示的欄位
`render()` 內的六行 `txt(g, 12, y, ...)`，y 間距 18～22；加行要同步加大 `TEXT_H`（現為 128）。

### 改說明欄內容
- 點選詳細：`renderPanel()` 的 spec 段
- 材質總表：`MATERIALS` / `SEALS` 陣列
- 辨識小抄：id 段的 HTML

## 設計取捨（別「順手優化」掉）

- **圓是真實比例**（SCALE px/mm）— 這是「遠看比大小」的核心，別改成統一大小。
- **文字跟著 SVG 縮放** — 遠看糊是刻意的（那是縮圖視角），近看才要清楚。
- **搜尋的數字比對只在純數字時啟用**（`/^[\d.]+$/`）— 見 lessons。
- 底孔（drill）：PF 與 M 用公式（外徑−牙距），PT/NPT 用業界慣用表 — 兩者來源不同是刻意的。

## 已知限制

1. 未含的牙種：BSW/BSF 惠氏機械牙、Tr 梯形牙、ACME、自攻牙、木牙 — 需求出現再加。
2. NPT 2-1/2" 以上底孔沒列（大口徑實務直接買管件不攻牙）。
3. 手機版說明欄固定佔高 42%，尚未做收合。
4. 截圖驗證時 Claude 的 preview_screenshot 工具會逾時（工具問題、非頁面問題）— 用 preview_eval 查 DOM 驗證即可。

## 驗證清單（改完必跑）

1. `python -m http.server 8799` 開 `http://localhost:8799`
2. 初始畫面＝五大類全覽（不是空白、不是 scale(0)）
3. 搜尋「4分」→ PT/PF/NPT 1/2 與 4分牙；「21」→ 外徑 21mm 附近；「M10」→ 只有 M10
4. 點任一卡片 → 右欄出現規格表；三個分頁都有內容
5. F12 console 無錯誤
