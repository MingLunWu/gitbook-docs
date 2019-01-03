# 41. 規則系統

本章討論 PostgreSQL中的規則系統（rule system）。產品的規則系統在概念上很簡單，但實際使用它們會涉及許多細微的觀點。

其他一些資料庫系統定義了有效的資料庫規則，這些規則通常是儲存過程和觸發器。在 PostgreSQL 中，這些可以使用函數和觸發器來達成。

規則系統（更確切地說，查詢覆寫規則系統）與儲存過程和觸發器完全不同。它以規則修改查詢，然後將修改的查詢傳遞給查詢規劃器進行規劃和執行。它的功能非常強大，可用於查詢語言程序、view 和版本等多種功能。\[[ston90b](../../can-kao-shu-mu.md)\] 和 \[[ong90](../../can-kao-shu-mu.md)\] 也討論了這個規則系統的理論基礎和能力。
