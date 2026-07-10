# requestAnimationFrame 在內嵌/背景環境可能完全不觸發，動畫改用 setInterval

一行摘要：Claude 的 Browser 預覽面板（及部分背景分頁、省電模式）rAF 一次都不會呼叫，3D 動畫看起來「畫得出第一幀但不會動」，console 也不會有錯誤。

**為什麼**：瀏覽器會對不可見／節流的 renderer 暫停 rAF；這不是程式錯誤，換到前景正常分頁就會動，所以極難用 console 除錯。當時症狀：`v3d.ry` 停在第一次增量、rAF id 永遠是 1。

**解法**（已套用在 index.html 的 3D 檢視器）：

```js
v3d.timer = setInterval(stepV3D, 33);  // 開啟時啟動，關閉時 clearInterval
```

**順帶的第二個坑**：軟體渲染的光照，若用背面剔除＋固定光向，繞錯方向會整支全黑
（shade 全為 0）。穩健做法：法向量一律翻向觀察者（n.z > 0 就取負）再與光向內積，
不做背面剔除，交給 painter's algorithm 排序。
