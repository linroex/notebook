---
tags:
  - systems/linux
  - systems/boot
  - programming/concepts
  - reverse-engineering
  - source/hacker-news
---

# Linux 是一種直譯器

> 一篇用 boot、initrd、cpio 與 kexec 把 Linux 啟動流程重新解釋成「直譯器模型」的硬派文章。

## 背景

這篇文章延續作者先前一系列關於 Linux 開機、initramfs 與 kexec 的實驗。它從一個看起來很危險的 `curl | sh` 啟動腳本開始，把裡面包住的內容一層層拆開，最後導向一個非常 hacker 風格的核心主張：**Linux kernel 可以被視為執行 initrd 的直譯器。**

## 核心重點

### 1. 一個可疑的 shell script，實際上包著完整的開機映像

文章先把 gz 壓縮檔解開，發現它表面上是 shell script，但內部其實塞了大量 base64 資料。這些資料解碼後是一個 cpio archive，裡面又包含：

- 一個 kernel 映像 `k`
- 一個 `init` 腳本
- 其他 rootfs 內容

接著透過 `kexec --load` / `kexec --exec`，用新的 kernel 與 ramdisk 直接替換掉目前正在跑的系統。

### 2. initrd 不只是資料包，也可以視為「程式」

作者把傳統對 initramfs / initrd 的理解往前推一步：

- 我們平常把它當作 boot 時的臨時 root filesystem
- 但其實 kernel 會讀取它、掛載它、執行其中的 `/init`
- 從這個角度看，kernel 就像是在「解譯」initrd 這個程式包

這是整篇最有趣的觀念轉換。

### 3. 用 kexec 做遞迴，形成一種 Linux 自我執行結構

文章展示的 `init` 腳本會：

- 掛載 `/proc`
- 把當前系統重新打包成新的 cpio
- 再次 `kexec` 到新 kernel + 新 initrd

因此整個系統會反覆把自己重新包裝、重新啟動。作者把它比喻成一種 tail-call optimized recursion：

- 不是傳統函式堆疊遞迴
- 而是每次都用新的 kernel 取代舊 kernel
- 舊的執行狀態留在記憶體某處，新的一份接手繼續跑

### 4. 文章把 quine、shebang、ELF interpreter 串成同一條概念線

後半段更精彩。作者把這個 Linux 啟動遞迴和 quine（輸出自身的程式）連在一起思考，甚至延伸到：

- shell script 用 shebang 指定 interpreter
- ELF binary 其實也有自己的 interpreter（動態 linker）
- 因此「被另一個程式讀入並執行」這件事，並不是腳本語言獨有

最後形成一個很強的統一視角：
**程式執行常常不是直接被 CPU 跑，而是被另一層執行系統詮釋。Linux kernel 也可以放進這個脈絡。**

## 實務啟發

- 如果你在做 boot、recovery、embedded Linux 或特殊部署，這篇很值得讀，因為它會幫你把 initrd 看成更主動的執行單位。
- `kexec` 不只是快速重開機工具，也可以拿來做系統切換、鏈式啟動與奇特的實驗性控制流。
- 理解「誰在解譯誰」這個問題，對 shell、ELF、boot chain、container/runtime 都很有幫助。

## 我的補充

這篇文章最迷人的地方不是某個技巧本身，而是它在做「觀念重命名」。

平常我們會把 Linux boot 過程拆成很多孤立名詞：kernel、initrd、shebang、dynamic linker、exec。但作者把它們全部收斂到一個直覺很強的框架：**interpreter all the way down**。

這種文章的價值不只是教你做事，而是改變你之後看系統的方式。

## 參考來源

- 原文：<https://astrid.tech/2026/03/28/0/linux-is-an-interpreter/>
- Hacker News 摘要脈絡：2026-03-29 今日 HN 重點第 8 則
