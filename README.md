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

### 記錄各段 ping 是否成功、RTT 大約多少  
確認本機的預設閘道為 192.168.146.2，透過 eth0 介面連線。RTT 最小值為 0.143 ms，平均值為 0.761 ms，最大值為 2.378 ms。
因此，本次測試確認 VMware Guest 與預設閘道之間的 ICMP 通訊正常，沒有觀察到封包遺失。  
<img width="869" height="250" alt="image" src="https://github.com/user-attachments/assets/bfb7105e-c3bc-4245-b846-0a3393038fcb" />

