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

|小題	|指令範例	|用途（請填）	|
|---|---|---|
|6-2-1	|sar -u 1 5	CPU	|每秒採樣一次，共五次，觀察系統整體 CPU 使用率。|
|6-2-2	|sar -P ALL 1 5		|
|6-2-3	|sar -r 1 5		|
|6-2-4	|sar -b 1 5		|
|6-2-5	|sar -n DEV 1 5		|
|6-2-6	|sar -u -f /var/log/sa/sa10	讀當月第 DD 日歷史	|
|6-2-7	|mpstat -P ALL 1 5	各核心；與 sar -P 互補	|查看整體 CPU 及各個邏輯 CPU 的使用率。|
|6-2-10	|pidstat -u 1 5		|
|6-2-11	|pidstat -p <PID> -u 1 5	指定 PID	|
|6-2-12	|vnstat / vnstat -h / vnstat --top 10	長期流量 vs iftop 當下	|

<img width="797" height="782" alt="image" src="https://github.com/user-attachments/assets/77533948-280b-423c-98e8-b8a1602885be" />

|小題	|指令範例	|用途（請填）	|
|---|---|---|
|6-2-8	|iostat 2 3		|每隔 2 秒取得一次磁碟 I/O 統計，共三組資料。|
|6-2-9	|iostat -x 2 3	await、util 等	|可以取得各磁碟的進階 I/O 統計，包括 r/s、w/s、r_await、w_await、aqu-sz 與 %util。|


<img width="794" height="634" alt="image" src="https://github.com/user-attachments/assets/0fdc84f7-17b4-47bf-8c33-f9d70e3ed82b" />
<img width="1153" height="667" alt="image" src="https://github.com/user-attachments/assets/2f0cc99c-2b2a-451e-a99b-15085f6121dd" />
