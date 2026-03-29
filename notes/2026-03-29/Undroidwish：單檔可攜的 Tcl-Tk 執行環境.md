---
tags:
  - programming/tcl-tk
  - software/portable-apps
  - cross-platform
  - developer-tools
  - source/hacker-news
---

# Undroidwish：單檔可攜的 Tcl/Tk 執行環境

> 一個把 Tcl/Tk 執行環境、圖形能力與大量 extension 打包成單一可攜 binary 的 old-school 工具鏈專案。

## 背景

Undroidwish 是從 AndroWish 生態延伸出來的單檔 Tcl/Tk binary，強調：

- 單一檔案
- 多平台可攜
- 內建大量 Tcl/Tk extension
- 不依賴複雜安裝流程

它的設計哲學很復古，但也很實用：讓一個 GUI / scripting 環境可以像工具箱一樣被帶著走。

## 核心重點

### 1. 單一 binary，不寫 registry，盡量不改系統

Windows 版本特別強調不寫 registry、不主動修改系統，這讓它很像一種 portable app。對於需要在不同機器快速帶著工具跑的人來說，這個特性很有吸引力。

### 2. 不是只有 Tcl/Tk 本體，而是附帶完整「batteries included」生態

內建內容包含：

- tkpath
- tktreectrl
- tkimg
- Canvas3D
- tcllib
- tksqlite
- bwidgets

也就是說，它不是只給你一個裸直譯器，而是連常見 GUI 與工具型擴充一起打包。

### 3. 平台支援範圍非常廣

文中提到可支援或部分支援：

- Windows
- Linux
- ARM 裝置（如 Raspberry Pi）
- Wayland / console framebuffer 場景
- FreeBSD / OpenBSD
- macOS
- Haiku
- Termux / 手機環境

這種廣泛支援非常符合 Tcl/Tk 一直以來「到處都能跑」的精神。

### 4. 內建 ZIP virtual file system 是它很有特色的一點

內建腳本可以直接從嵌入 binary 的 ZIP file system 啟動，像是 widget demo、SQLite GUI、Canvas3D demo、Tk console 等。這意味著 binary 不只是載體，還內含一整套可直接執行的應用與示範。

## 實務啟發

- 如果你需要的是可攜式 GUI scripting runtime，而不是現代重量級 app framework，這種工具反而很對味。
- 單檔封裝對內部工具、維運小工具、嵌入式環境與教學展示都很有價值。
- 很多「老技術」沒變主流，不代表沒有實用價值；在某些情境下，它們反而因為成熟、輕量、低依賴而特別好用。

## 我的補充

我覺得 Undroidwish 吸引人的地方，是它代表了一種快失傳的工程美學：

**一個檔案、盡量少依賴、帶著走就能跑。**

在今天這個什麼都要 npm / container / electron 的時代，這種設計會讓人重新想起：有些工具最迷人的地方就是簡單、可攜、可預測。

## 參考來源

- 原文：<https://androwish.org/home/wiki?name=undroidwish>
- Hacker News 摘要脈絡：2026-03-29 今日 HN 重點第 16 則
