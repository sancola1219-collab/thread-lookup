# Claude Browser 預覽有時 window.innerWidth/Height = 0，vh/% 版面歸零、canvas 取不到像素

一行摘要：在 Claude 的 Browser 預覽面板跑 `mcp__Claude_Browser__javascript_tool` 時，`window.innerWidth`、`innerHeight` 可能是 0；於是 `height:56vh`、`width:100%` 全算成 0，canvas `clientWidth/Height=0`，`getImageData` 丟 IndexSizeError，`preview_screenshot` 也逾時。這是預覽環境限制，不是程式錯誤——真手機/電腦視窗正常。

**為什麼**：headless/detached 的預覽 renderer 視窗尺寸為 0，所有相對視窗的長度單位跟著歸零。

**驗證怎麼繞過**：
1. **SVG 走 DOM 座標**：`getBBox()` 回傳的是 viewBox user-unit，和視窗大小無關——所以 2D 剖面詳圖的幾何、文字位置、有無重疊、有無超出 viewBox，全部量得到、驗得準（比肉眼看截圖更嚴謹）。
2. **canvas 需暫時強制固定 px 尺寸**：`cv.style.width='600px'; cv.style.height='420px'`，再觸發重繪，才拿得到像素做分析。
3. **判斷牙紋是否浮現**：不要只看灰階 std（平滑圓桶的方向光也有 std）；改沿掃描線數「明暗起伏次數(局部極值)」——螺紋每條線會有很多次起伏，光滑面幾乎沒有。
4. canvas 空白區是「透明黑(0,0,0,α=0)」不是 CSS 背景色，過濾背景要用 `alpha<20 或 lum<牙面最小灰階`，別用 CSS 顏色比對。

**教訓**：互動/幾何優先用 DOM 量測驗證（穩、與視窗無關）；像素級驗證在此環境要先自己給元素固定尺寸。
