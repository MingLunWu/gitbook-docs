# CREATE MATERIALIZED VIEW

CREATE MATERIALIZED VIEW — 定義一個新的具體化檢視表

### 語法

```text
CREATE MATERIALIZED VIEW [ IF NOT EXISTS ] table_name
    [ (column_name [, ...] ) ]
    [ WITH ( storage_parameter [= value] [, ... ] ) ]
    [ TABLESPACE tablespace_name ]
    AS query
    [ WITH [ NO ] DATA ]
```

### 說明

CREATE MATERIALIZED VIEW 定義查詢的具體化檢視表。執行該查詢並在命令完成後將資料儲存於檢視表中（除非使用 WITH NO DATA），並可在稍後使用 `REFRESH MATERIALIZED VIEW 更新。`

`CREATE MATERIALIZED VIEW 與 CREATE TABLE AS 類似，只是它還記得用於初始化檢視表的查詢，以便稍後可以根據需要更新它。具體化檢視表具有許多與資料表相同的屬性，但不支援臨時具體化檢視表或自動產生 O`ID。

### 參數

`IF NOT EXISTS`

如果已經存在具有相同名稱的具體化檢視表的話，請不要拋出錯誤。在這種情況下會發布通知。請注意，這不能保證現有的具體化檢視表與想要建立的檢視表相似。

_`table_name`_

要建立的具體化檢視表名稱（可選擇性加上所屬綱要）。

_`column_name`_

新的具體化檢視表中欄位的名稱。如果未提供欄位名稱，則從查詢的輸出的欄位名稱中取得它們。

`WITH ( `_`storage_parameter`_ \[= _`value`_\] \[, ... \] \)

此子句為新的具體化檢視表指定選擇性的儲存參數；請參閱[儲存參數選項](create-table.md#storage-parameters)了解更多訊息。CREATE TABLE 支援的所有參數也都支援 CREATE MATERIALIZED VIEW，只有 OIDS 除外。有關更多訊息，請參閱 [CREATE TABLE](create-table.md)。

`TABLESPACE `_`tablespace_name`_

tablespace\_name 用於要在其中建立新的具體化檢視表的資料表空間名稱。如果未指定，則會使用 [default\_tablespace](../../server-administration/runtime-config/runtime-config-client.md#19-11-1-cha-ju-de-hang) 設定。

_`query`_

A [SELECT](https://www.postgresql.org/docs/10/static/sql-select.html), [TABLE](https://www.postgresql.org/docs/10/static/sql-select.html#SQL-TABLE), or [VALUES](https://www.postgresql.org/docs/10/static/sql-values.html) command. This query will run within a security-restricted operation; in particular, calls to functions that themselves create temporary tables will fail.

`WITH [ NO ] DATA`

This clause specifies whether or not the materialized view should be populated at creation time. If not, the materialized view will be flagged as unscannable and cannot be queried until `REFRESH MATERIALIZED VIEW` is used.

### Compatibility

`CREATE MATERIALIZED VIEW` is a PostgreSQL extension.

### See Also

[ALTER MATERIALIZED VIEW](https://www.postgresql.org/docs/10/static/sql-altermaterializedview.html), [CREATE TABLE AS](https://www.postgresql.org/docs/10/static/sql-createtableas.html), [CREATE VIEW](https://www.postgresql.org/docs/10/static/sql-createview.html), [DROP MATERIALIZED VIEW](https://www.postgresql.org/docs/10/static/sql-dropmaterializedview.html), [REFRESH MATERIALIZED VIEW](https://www.postgresql.org/docs/10/static/sql-refreshmaterializedview.html)
