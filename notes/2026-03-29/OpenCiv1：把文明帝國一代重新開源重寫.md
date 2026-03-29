---
tags:
  - gaming/civilization
  - open-source
  - software-preservation
  - reverse-engineering
  - source/hacker-news
---

# OpenCiv1：把文明帝國一代重新開源重寫

> 一個試圖在不碰原始版權素材的前提下，逐步重建 Civilization 1 的開源重寫專案。

## 背景

OpenCiv1 是針對 1991 年原版《Civilization》進行的 open-source rewrite。這類專案常常不只是「懷舊重做」，而是軟體保存、逆向理解、版權切割與現代平台移植的綜合工程。

## 核心重點

### 1. 目標不是 clone 一份版權有問題的舊遊戲，而是逐步替換到完全自由的版本

OpenCiv1 明確強調：

- repo 內不包含原版 Civilization 1 的原始檔案
- 使用者若要執行，仍需合法擁有原版 DOS 遊戲檔
- 專案透過虛擬 CPU 模擬部分 assembly 邏輯，其餘程式碼逐步以新 C# 程式重寫

也就是說，它不是單純 dump 出舊程式，而是在法律邊界內重建行為與玩法。

### 2. 保留原作精神，但不排斥現代化功能

作者表示目標是保留原版規則、整體手感與視覺風格，同時逐步加入：

- 跨平台能力
- 高解析圖像與音效
- 多語言
- Multiplayer
- Plugin capability
- Web / modern interface 等未來方向

這代表它不是像 FreeCiv 那種另起爐灶的變體，而更像是「可長期維護的 Civilization 1 保存與延伸版本」。

### 3. C# 的選擇是工程現實，不是情懷優先

原版遊戲主要與 x86 assembly 綁得很深，但 OpenCiv1 選擇用 C# 重建，原因是：

- 平台獨立
- 安全
- 管理式執行環境較容易維護
- API 與現代工具鏈較成熟

這也顯示一件事：做歷史軟體保存，不代表一定要完全複製當年的技術條件；有時更重要的是忠實重建行為與體驗。

## 實務啟發

- 遇到老軟體保存問題時，open rewrite 往往是比 binary patch 更可持續的路。
- 法律上，**重現功能** 與 **直接搬原始素材** 是兩回事；很多開源重製案能成立，靠的就是這條分界。
- 若要讓老遊戲跨平台、可維護、可被社群參與，現代語言與乾淨授權通常比原味技術棧更重要。

## 我的補充

我很喜歡這類專案，因為它們其實在回答一個大問題：
**經典軟體能不能在不依賴版權灰區與過時平台的情況下，繼續活下去？**

OpenCiv1 的答案是可以，但代價是要做大量枯燥又長期的重建工作。也因此這種專案很值得敬佩——它不是 flashy demo，而是耐心把數十年前的作品重新變成可維護系統。

## 參考來源

- 原始專案：<https://github.com/rajko-horvat/OpenCiv1>
- Hacker News 摘要脈絡：2026-03-29 今日 HN 重點第 9 則
