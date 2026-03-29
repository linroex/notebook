# macOS 用 kqueue 偵測檔案變更

> 一篇從實作 file watcher 出發，往下理解 macOS 底層事件通知機制 kqueue 的教學文。

## 背景

作者原本寫了一個小工具 `reload`，會在檔案變更時重新執行命令，例如：

- `reload gcc main.c -o main && ./main`
- `reload make`

一開始他直接使用跨平台套件 fsnotify，但後來想理解 macOS 底層究竟怎麼做，因此回頭研究 kqueue 與 kevent。

## 核心重點

### 1. kqueue 是一個核心事件佇列機制

在 macOS 上，可以透過：

- `kqueue()` 建立事件佇列
- `kevent()` 註冊與等待事件

若要監看檔案變動，會用到：

- `EVFILT_VNODE`：針對 vnode / file descriptor 的事件
- `NOTE_WRITE`：關心檔案被寫入
- `EV_ADD | EV_CLEAR`：註冊事件並在事件送達後清除狀態

文章把這些資料結構與欄位講得很清楚，很適合想理解系統 API 而不是只會調套件的人。

### 2. 監看單一檔案其實不難，難的是目錄與新檔案

如果只是明確知道要監看的檔案，可以：

- 用 `O_EVTONLY` 開啟檔案
- 為每個 fd 註冊 kqueue 事件
- 在 event loop 中等待 `NOTE_WRITE`

但如果是監看整個目錄，事情就比較麻煩：

- 只監看目錄本身，只能看到新增 / 刪除
- 既有檔案內容被修改時，不會自動全部涵蓋
- 因此還需要遞迴打開目錄內檔案並逐一監看
- 若有新檔案生成，還得重新掃描加入 watcher

### 3. `close-on-exec` 是很重要的細節

作者在 Go 版實作裡特別提到 `O_CLOEXEC` / `CloseOnExec`。原因是當程式重新執行命令時，如果 child process 繼承了 watcher 的 file descriptor：

- 會造成 fd 洩漏
- 可能讓被監看的檔案無法完全釋放
- child 也可能拿到一個不會處理的 kqueue fd

這是很典型的系統程式設計細節：功能能跑不難，資源生命週期才是容易出包的地方。

## 實務啟發

- 想做 macOS 專用 file watcher 時，kqueue 是很直接的底層方案。
- 但它是「每個檔案要有 fd」的模型，因此在超大目錄樹不一定划算。
- 若你的需求只是中小型目錄監看、開發時自動 reload，這套做法非常合理。
- 讀這篇最大的收穫不是 API 記法，而是理解「看似簡單的檔案監看，其實有很多邊界條件」。

## 我的補充

這篇是很典型、我很喜歡的工程文章：

- 先有實際需求
- 再懷疑抽象套件背後在做什麼
- 最後往下挖到底層機制

這種學法的好處是，你學到的不是死知識，而是整個設計脈絡。之後就算 API 換語言，你還是知道自己在做什麼。

## 參考來源

- 原文：<https://www.vegardstikbakke.com/kqueue/>
- Hacker News 摘要脈絡：2026-03-29 今日 HN 重點第 17 則
