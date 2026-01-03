# Wazuh 與 pfSense 偵測關聯與回應實作報告

**系級：** 醫資三  
**學號：** 412570182  
**姓名：** 閻宇虹  

---

## 一、 作業目標
主要目標是透過 Wazuh Agent 將 pfSense 防火牆的 Suricata 偵測日誌傳送到 Wazuh Manager，進行安全事件的關聯分析。透過模擬真實的攻擊行為，驗證系統的監控與警報能力。

---

## 二、 實作過程紀錄與技術修正

在實作過程中，遇到了幾個關鍵問題並成功排除，以下是本次作業最重要的學習成果：

### 1. 版本不一修正
由於官方 4.13.1 版本下載受限，導致 Agent 無法安裝。我將 Wazuh Manager 與 Dashboard 同步升級至 **v4.14.1**，確保後端與 pfSense Agent 版本完全相容。

![Wazuh 版本資訊](pfsense連到wazuh%20dashboard.png)
*(圖：確認 Agent 版本為 v4.14.1 且狀態為 Active)*

### 2. 通訊協定校正 (TCP)
原本 Agent 預設使用 UDP 連線導致連線失敗（Closing connection）。手動修改 pfSense 的 `ossec.conf`，將協定改為 **TCP**，成功解決了連線問題。

### 3. 日誌路徑定位
透過 `find` 指令精確找出 Suricata 生成的日誌真實路徑。

---

## 三、 攻擊模擬與偵測驗證

### 1. 撰寫自訂規則 (Custom Rules)
為了辨識特定攻擊，在 pfSense 中撰寫了一條帶有學號的自訂規則。

![自訂規則設定](自訂規則.png)
*(圖：自訂規則 msg:"412570182 Nmap Scan Detected")*

### 2. 發動攻擊行為 (Port Scan)
使用 Ubuntu Server 對 pfSense 執行 **Nmap SYN 掃描**。

![Nmap 攻擊模擬](nmap.png)
*(圖：執行 sudo nmap -sS 10.0.3.1 進行掃描)*

### 3. 實驗成果證明

* **pfSense 端**：Suricata Alerts 成功抓到掃描行為。
![pfSense 警告紀錄](pfsense警告.png)
*(圖：pfSense 介面顯示 412570182 警告訊息)*

* **Wazuh 端**：Dashboard 成功接收到告警訊息，證明資料傳送管線完全暢通。
![Wazuh 警告紀錄](pfsense%20agent警告.png)
*(圖：Wazuh Dashboard 接收到含有學號之警告事件)*

---

## 四、 風險分析與回應建議

### 1. 事件風險分析
此告警代表攻擊者正在進行「偵查階段」的埠口掃描。這在真實環境中代表駭客正試圖尋找開放的服務漏洞。若不及時發現，接下來可能就是針對性的滲透攻擊。

### 2. 回應與阻擋建議
目前系統已完成「偵測」與「傳送」。若要達成「回應（阻擋）」，可以在 pfSense 的 Suricata 設定中啟用 **"Block Offenders"** 功能。一旦偵測到此類 Nmap 掃描，防火牆會自動封鎖來源 IP，實現即時的主動防禦。
### 2. 回應與阻擋建議
目前系統已完成「偵測」與「傳送」。若要達成「回應（阻擋）」，可以在pfSense的Suricata設定中啟用"Block Offenders"功能。一旦偵測到此類Nmap掃描，防火牆會自動封鎖來源IP，實現即時的主動防禦。
