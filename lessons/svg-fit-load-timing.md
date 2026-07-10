# SVG 載入瞬間 clientWidth 可能是 0，fit() 要重試

一行摘要：script 在 body 尾端同步執行時，flex 版面可能還沒排好，`svg.clientWidth` 回 0 → 縮放算出 `scale(0)` 整頁看似空白。

**為什麼**：`Math.min(vw/worldW, vh/worldH)` 在 vw=0 時得 0，transform 變 `scale(0)`，所有內容縮成一個點。console 不會有任何錯誤，很難察覺。

**解法**（已套用在 index.html 的 fit()）：

```js
if(vw < 50 || vh < 50){ requestAnimationFrame(fit); return; }
```

尺寸未就緒就下一幀重試，直到版面排好。
