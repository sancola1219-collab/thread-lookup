# setPointerCapture 會吃掉子元素的 click；座標要用 clientX−rect，別用 offsetX

一行摘要：真機上「只能縮放、拖不動、點卡片沒反應」——因為 pointer capture 把後續事件（含 click）改派到捕捉元素，卡片的 click 監聽永遠收不到；offsetX 的基準又隨事件目標變動導致拖曳座標亂跳。

**為什麼**：
1. `svg.setPointerCapture()` 之後，該指標的所有後續事件（包含衍生的 click）都派給 svg 本體，掛在卡片 `g` 上的 click 監聽不會觸發。
2. `e.offsetX` 是「相對於事件目標」——按在卡片上時相對卡片、捕捉後相對 svg，兩者基準不同，delta 計算全錯。
3. 某些環境 `setPointerCapture` 會丟例外，讓整個 handler 中斷。

**解法**（已套用）：
- 座標一律 `e.clientX − container.getBoundingClientRect().left`。
- 點選改在 `pointerup` 判定：pointerdown 記下 `e.target.closest('g.card')`，
  放開時「沒移動超過 4px 且不是雙指」才算點選。不掛任何 click 監聽。
- `setPointerCapture` 包 try/catch。

**更大的教訓——測試方法**：先前驗證點擊用的是 `dispatchEvent(new MouseEvent('click'))`，
這條合成路徑跳過了 pointer capture 的真實行為，所以「測試全過、真機全掛」。
驗證互動要模擬完整的 pointerdown→pointermove→pointerup 序列，或實機操作。
