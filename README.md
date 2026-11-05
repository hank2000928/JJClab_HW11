# JJClab_HW11
參考筆記: https://github.com/wsunccake/sle15_notes/blob/master/practice/ch3.md  

-----------------------------------------------

# 練習 6. Stat：抽樣、對照時段與歷史

## 6-1. vmstat 快速判讀

|區域|	欄位|	意義|
|---|---|---|
|procs|	r、b	|行程排程與等待情況|
|memory|	swpd、free、buff、cache|	記憶體使用情況|
|swap|	si、so	|Swap 讀入／寫出速率|
|io|	bi、bo	|區塊裝置讀寫量|
|system|	in、cs|	中斷與上下文切換|
|cpu|	us、sy、id、wa、st|	CPU 時間分布|

以虛擬機為例，先前已經確認 sle15-a 只有 2 個 vCPU，觀察 r 時，與 CPU 數量進行比較。


|規則	|你的觀察|判斷|
|---|---|---|
|r 長期明顯大於 CPU 數 → CPU 壓力	|最大為 1，VM 有 2 vCPU|未見 CPU 排隊壓力|
|b 高且 wa 高 → 可能 I/O 等待	|b=0，後續 wa=0%|未見明顯 I/O 等待|
|si/so 持續明顯 → 記憶體不足、在用 swap	|全部為 0|未見 Swap 活動|
|st 偏高（VM）→ Host 可能超賣 CPU	|全部為 0%|未見 Hypervisor CPU Steal Time|

<img width="795" height="344" alt="image" src="https://github.com/user-attachments/assets/399bed3b-d14d-4b64-b4a9-c1a695312e42" />

## 6-2. sar / mpstat / iostat / pidstat / vnstat

|小題	|指令範例	|用途	|
|---|---|---|
|6-2-1	|sar -u 1 5	CPU	|每秒採樣一次，共五次，觀察系統整體 CPU 使用率。|
|6-2-7	|mpstat -P ALL 1 5	|各核心；與 sar -P 互補	查看整體 CPU 及各個邏輯 CPU 的使用率。|

<img width="797" height="782" alt="image" src="https://github.com/user-attachments/assets/77533948-280b-423c-98e8-b8a1602885be" />

|小題	|指令範例	|用途	|
|---|---|---|
|6-2-8	|iostat 2 3		|每隔 2 秒取得一次磁碟 I/O 統計，共三組資料。|
|6-2-9	|iostat -x 2 3	|await、util 等。	可以取得各磁碟的進階 I/O 統計，包括 r/s、w/s、r_await、w_await、aqu-sz 與 %util。|


<img width="794" height="634" alt="image" src="https://github.com/user-attachments/assets/0fdc84f7-17b4-47bf-8c33-f9d70e3ed82b" />
<img width="1153" height="667" alt="image" src="https://github.com/user-attachments/assets/2f0cc99c-2b2a-451e-a99b-15085f6121dd" />

|小題	|指令範例	|用途	|
|---|---|---|
|6-2-10	|pidstat -u 1 5		|每秒採樣一次，連續五次記錄活躍行程的 CPU 使用情況。|
|6-2-11	|pidstat -p <PID> -u 1 5|	指定 PID，每秒採樣一次 CPU 負載與排程等待，共五次。|

<img width="915" height="630" alt="image" src="https://github.com/user-attachments/assets/dfc21a33-7b23-4dab-9b4f-82bb5d720136" />
<img width="925" height="234" alt="image" src="https://github.com/user-attachments/assets/3d825004-632c-4c90-8aa9-ce52122b0adf" />

|小題	|指令範例	|用途|
|---|---|---|
|6-2-2	|sar -P ALL 1 5		|每秒採樣一次，連續五次查看所有邏輯 CPU 的使用率。|
|6-2-3	|sar -r 1 5		|每秒採樣一次，共五次，觀察系統記憶體使用情況。|

<img width="899" height="595" alt="image" src="https://github.com/user-attachments/assets/21888834-1d18-4706-839a-fbd1da331c25" />
<img width="1026" height="204" alt="image" src="https://github.com/user-attachments/assets/07de3197-9bb1-4c52-8f91-e77bb42b8d50" />

|小題	|指令範例	|用途	|
|---|---|---|
|6-2-4	|sar -b 1 5		|每秒採樣一次，共五次，觀察整體區塊裝置的讀寫情況。
|6-2-5	|sar -n DEV 1 5		|每秒採樣一次，共五次，觀察各個網路介面的封包傳輸速率與資料吞吐量。

<img width="893" height="199" alt="image" src="https://github.com/user-attachments/assets/f103f10d-b608-4eed-8402-146e53c1b05a" />
<img width="898" height="490" alt="image" src="https://github.com/user-attachments/assets/5239ad85-3d8b-42d6-9063-33a7c7ab3aa2" />

|小題	|指令範例	|用途	|
|---|---|---|
|6-2-6	|sar -u -f /var/log/sa/sa10		|確認系統存在 sar 歷史資料目錄，但查詢 /var/log/sa/sa* 時沒有找到歷史統計檔案。目前 sysstat 已能進行即時採樣，但尚未找到可供讀取的歷史紀錄。可能原因是套件剛安裝、定期資料收集尚未啟用，或尚未執行第一次收集。|
|6-2-12	|vnstat / vnstat -h / vnstat --top 10|package vnstat is not installed|

## 6-3. 監控策略
Monitor 工具主要用於即時觀察系統狀態，例如透過 top 找出高 CPU 使用率行程，或使用 iotop 找出正在大量讀寫磁碟的行程。
Stat 工具則適合定時採樣及分析效能趨勢，例如 vmstat 可快速觀察整體系統壓力，mpstat 可以比較不同 CPU 核心的使用率，iostat -x 可分析磁碟 I/O 延遲，而 pidstat 能針對指定 PID 持續採樣。
當系統變慢時，可以先使用 vmstat 建立整體資源使用情況，再根據異常指標選擇適當工具。例如 r 持續偏高時使用 mpstat 與 pidstat；b、wa 偏高時使用 iostat -x 與 iotop；若需要確認異常是否曾在其他時段發生，則使用 sar 的歷史資料進行比較。

-----------------------------------------------
# 練習 7. 網路排查（一）：連通、介面、監聽、DNS

## 7-1. ping / traceroute

### 7-1-1. 記錄各段 ping 是否成功、RTT 大約多少  

確認本機的預設閘道為 192.168.146.2，透過 eth0 介面連線。RTT 最小值為 0.143 ms，平均值為 0.761 ms，最大值為 2.378 ms。
因此，本次測試確認 VMware Guest 與預設閘道之間的 ICMP 通訊正常，沒有觀察到封包遺失。  

<img width="869" height="250" alt="image" src="https://github.com/user-attachments/assets/bfb7105e-c3bc-4245-b846-0a3393038fcb" />

### 7-1-2. 說明「能 ping IP、不能 ping 名稱」下一步查什麼

當外部 IP 可以 ping 成功，但網域名稱無法解析時，應優先檢查 DNS。
可以使用 cat /etc/resolv.conf 確認 DNS Server 設定，並使用 getent 或 dig 測試名稱解析。若 DNS Server 設定正確，則進一步檢查路由、防火牆與 DNS 服務是否正常。
<img width="880" height="375" alt="image" src="https://github.com/user-attachments/assets/3b8a3ef2-8818-4236-9b10-c52535680b4e" />

### 7-1-3. traceroute 某跳無回應，是否一定代表該跳故障？為什麼？

使用 traceroute -n -m 15 -w 1 -q 1 example.com 追蹤網路路徑。
本次第一跳成功收到 192.168.146.2 的回覆，RTT 為 0.199 ms，與本機預設閘道相符。
第 2～15 跳均顯示 *，表示探測封包沒有在指定時間內收到預期回覆。
可能原因包含路由器限制 ICMP、VMware NAT 或防火牆過濾，以及 UDP traceroute 探測受到限制。
由於先前 ping 8.8.8.8 和 ping example.com 都成功，封包遺失率為 0%，因此不能直接將 traceroute 的星號判定為外部網路故障。可以進一步使用 traceroute -I 進行交叉驗證。

<img width="930" height="396" alt="image" src="https://github.com/user-attachments/assets/713252cb-16ec-4243-a6db-9fbd85a47ff5" />

## 7-2. ip：介面、路由、鄰居

|檢查項目	|你會看什麼|本機結果摘要|
|---|---|---|
|介面是否| ip a 旗標|UP	eth0 有 UP、LOWER_UP	正常|
|IP／遮罩	|CIDR 是否正確|192.168.146.129/24	設定正常|
|預設閘道|	ip r 是否有 default|192.168.146.2	路由存在|
|ARP／鄰居	|ip n MAC 對應|閘道 MAC 00:50:56:fc:cd:2d	已成功建立對應|

使用 ip a 確認本機 VMware 虛擬網卡 eth0 的狀態為 UP、LOWER_UP，IPv4 位址為 192.168.146.129/24，MAC Address 為 00:0c:29:a4:24:be。
使用 ip r 確認預設閘道為 192.168.146.2，透過 eth0 傳送外部網路封包。
使用 ip n 查詢鄰居表，確認閘道 192.168.146.2 對應的 MAC Address 為 00:50:56:fc:cd:2d，狀態為 STALE。這代表曾成功建立鄰居對應，但近期尚未重新確認可達性，並非網路故障。
結合先前成功的 ping 測試，判斷本次觀察沒有發現明顯的網卡、IP、路由或 ARP 異常。

<img width="929" height="401" alt="image" src="https://github.com/user-attachments/assets/77b21244-e221-49e6-8b95-9cd65e887af7" />

## 7-3. ss（取代舊 netstat）

|選項|	意義|
|---|---|
|-t	|顯示 TCP Socket|
|-u|	顯示 UDP Socket|
|-n	|使用數字顯示 IP 和 Port，不進行名稱解析|
|-l	|只看監聽中的 Socket|
|-p	|顯示使用 Socket 的行程資訊|
|-a	|顯示所有 Socket，包括監聽與非監聽|

|小題|	任務|結果|
|---|---|---|
|7-3-1	|找出本機 SSH 的監聽位址、埠與行程（PID／程式名）|本機 SSH 服務由 sshd 提供，PID 為 2152，監聽 TCP Port 22。IPv4 監聽位址為 0.0.0.0:22，IPv6 監聽位址為 [::]:22，狀態皆為 LISTEN。|
|7-3-2	|列出目前 LISTEN 的服務|查詢到 SSH 服務 sshd 監聽 TCP Port 22，master 監聽本機 Loopback 的 TCP Port 25，以及 wicked-dhcp4 使用 UDP Port 68|
|7-3-3	|說明如何用 ss 診斷 Address already in use，並與 lsof -i 交叉驗證|SSH Port 22 的占用者為 sshd，PID 2152。確認占用者後，應判斷它是否為正常服務，再決定修改服務 Port、調整設定或處理舊行程，不應直接終止未知行程。|
|7-3-4	|「本機 ss 顯示 LISTEN」是否等於「外網可連」？中間還差哪些層？|顯示 LISTEN，只能證明本機服務已建立監聽 Socket，不能保證其他主機一定能成功連線。若外部無法連線，還需要確認網路路由、VMware NAT 或 Bridge 設定、防火牆是否允許對應 Port，以及來源端是否有網路存取限制。因此應搭配從客戶端進行的 TCP 連通測試，才能確認實際的端到端連線能力。|


<img width="931" height="346" alt="image" src="https://github.com/user-attachments/assets/4cfbd956-596e-473c-9462-9ae599a1434c" />
