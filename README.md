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

|規則|	觀察|情況解析|
|---|---|---|
|r 長期明顯大於 CPU 數 → CPU 壓力|r||	
||0～2|	以 2 vCPU 而言，不一定有 CPU 排隊壓力|
||4	|可能存在多個工作爭用 CPU|
||8	|若持續發生，可能存在較明顯的 CPU 壓力|
|b 高且 wa 高 → 可能 I/O 等待	|||
|si/so 持續明顯 → 記憶體不足、在用 swap|||
|st 偏高（VM）→ Host 可能超賣 CPU	|||
