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
