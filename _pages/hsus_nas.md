---
layout: page
permalink: /nas_setup
title: "How to set the NAS?"
description: "Show how to login to my NAS"
redirect: true
---

# How to setup the NAS

本頁說明如何透過 **Synology Drive Client** 或 **VPN** 連線到家中 NAS，並同步檔案。

---

## Method 1：使用 Synology Drive Client（推薦）

透過 Synology 官方桌面用戶端，可用 **QuickConnect** 從外網連回 NAS，無需設定 port forwarding。

### 1. 安裝 Synology Drive Client

1. **下載用戶端**
   - 前往 [Synology Download Center](https://www.synology.com/support/download) 或您的 NAS 上的 **套件中心**。
   - 選擇您的 NAS 型號，找到 **Synology Drive Server**（NAS 端）與 **Synology Drive Client**（電腦端）。
   - 下載對應您作業系統的 **Synology Drive Client**（Windows / macOS / Linux）。

2. **安裝**
   - 執行安裝程式，依指示完成安裝。
   - 安裝完成後啟動 **Synology Drive**。

### 2. 使用 QuickConnect 連線

1. **新增連線**
   - 在 Synology Drive 主畫面點選 **「登入」** 或 **「連線至 Synology NAS」**。

2. **輸入連線資訊**
   - **伺服器位址**：選擇 **QuickConnect**，並輸入 QuickConnect ID  
     **`hsu86725352`**
   - **帳號**：您的 DSM 使用者名稱。
   - **密碼**：該使用者的 DSM 密碼。
   - 若 NAS 有啟用兩步驟驗證，請依畫面輸入驗證碼。

3. **選擇同步任務類型**
   - **Team Folder**：同步共用資料夾（需管理員已分享給您）。
   - **My Drive**：同步個人雲端硬碟。
   - 勾選要同步的資料夾，並選擇本機同步位置（例如 `C:\Users\你的帳號\SynologyDrive` 或 Mac 上的 `~/SynologyDrive`）。

4. **完成設定**
   - 確認同步方向（雙向、僅上傳、僅下傳等），按 **「套用」** 開始同步。
   - 之後 Synology Drive 會在背景同步，並可在系統列／選單列查看狀態。

### 3. 注意事項

- 首次同步若檔案量大，可能需要較長時間。
- 請確保 NAS 已開機、網路正常，且 DSM 已啟用 **QuickConnect**（**控制台 > QuickConnect**）。
- 若連線失敗，可檢查防火牆是否阻擋，或改用 **Method 2** 透過 VPN 連線。

---

## Method 2（進階、較安全）：使用 Tailscale VPN

在 NAS 與電腦都安裝 **Tailscale** 後，可透過虛擬區域網路直接以 **區域網路位址** 連線，不需經過 QuickConnect，連線較穩定且可配合 Synology Drive Client 或 SMB 使用。

### 1. 安裝 Tailscale 並登入

- 在您的 **電腦** 與 **NAS** 上分別安裝 [Tailscale](https://tailscale.com/)。
- NAS 上可從 **套件中心** 安裝 Tailscale 套件，或依 Synology 教學設定。
- 使用同一個 Tailscale 帳號登入，使裝置都在同一 Tailscale 網路內。

### 2. 取得 NAS 的 Tailscale IP

- 登入 [Tailscale 管理後台](https://login.tailscale.com/admin/machines) 或在本機執行 `tailscale status`，查詢 NAS 的 Tailscale IP（例如 `100.x.x.x`）。

### 3. 使用 Synology Drive Client 連線

- 在 Synology Drive Client 中選擇 **「登入」**。
- 伺服器位址改選 **「透過 IP 或主機名稱連線」**，輸入上一步的 **Tailscale IP**（例如 `http://100.x.x.x:5000` 或 NAS 的 Tailscale 主機名）。
- 輸入 DSM 帳號、密碼，後續同步設定同 Method 1。

此方式流量走 Tailscale 加密通道，不依賴 QuickConnect，適合對連線穩定性與隱私要求較高的情境。
