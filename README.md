# Wazuh與pfSense偵測關聯與回應實作報告

**系級：** 醫資三
**學號：** 412570182
**姓名：** 閻宇虹

---

## 一、 作業目標
主要目標是透過Wazuh Agent將pfSense防火牆的Suricata偵測日誌傳送到Wazuh Manager，進行安全事件的關聯分析。透過模擬真實的攻擊行為，驗證系統的監控與警報能力。

---

## 二、 實作過程紀錄與技術修正

在實作過程中，遇到了幾個關鍵問題並成功排除，以下是本次作業最重要的學習成果：

1. **版本不一修正**：
   由於官方4.13.1版本下載受限，導致Agent無法安裝。我將Wazuh Manager與Dashboard同步升級至v4.14.1，確保後端與 pfSense Agent版本完全相容。

2. **通訊協定校正(TCP)**：
   原本Agent預設使用UDP連線導致無法註冊，手動修改pfSense的ossec.conf，將協定改為TCP (Protocol: tcp)，成功解決了與 Manager之間的連線問題。

3. **日誌路徑定位**：
   透過find指令精確找出Suricata生成的日誌路徑為 '/var/log/suricata/suricata_em16826/eve.json`，並同步修改Agent 設定，確保Wazuh能讀取到正確的告警資料。

---

## 三、 攻擊模擬與偵測驗證

### 1. 撰寫自訂規則 (Custom Rules)
為了辨識特定攻擊，在pfSense中撰寫了一條帶有學號的自訂規則：
`alert tcp any any -> $HOME_NET any (msg:"412570182 Nmap Scan Detected"; flags:S; sid:1000001; rev:1;)`

### 2. 發動攻擊行為 (Port Scan)
使用Ubuntu Server對pfSense執行Nmap SYN掃描。

### 3. 實驗成果證明
* **pfSense 端**：Suricata Alerts成功抓到掃描行為。
* **Wazuh 端**：Dashboard成功接收到來自pfSense的警告訊息，描述為「Suricata: Alert - 412570182 Nmap Scan Detected」。這證明了資料傳送管線完全暢通。

---

## 四、 風險分析與回應建議

### 1. 事件風險分析
此告警代表攻擊者正在對我方防火牆進行「偵查階段」的埠口掃描。這在真實環境中是非常危險的信號，代表駭客正在尋找開放的服務（如SSH或網頁服務）試圖入侵。如果沒有及時發現，接下來可能就是針對性的漏洞攻擊。

### 2. 回應與阻擋建議
目前系統已完成「偵測」與「傳送」。若要達成「回應（阻擋）」，可以在pfSense的Suricata設定中啟用"Block Offenders"功能。一旦偵測到此類Nmap掃描，防火牆會自動封鎖來源IP，實現即時的主動防禦。
