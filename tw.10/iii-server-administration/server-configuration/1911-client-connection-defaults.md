# 19.11. 用戶端連線預設參數[^1]

### 19.11.1. Statement Behavior

`search_path`\(`string`\)

這個參數表示，當一個物件（資料表、資料型別、函數等）以未指定 schema 的簡單名稱引用時，其搜尋的路徑順序。當不同 schema 中有相同名稱的物件時，將採用搜尋路徑中第一個找到的物件。不在搜尋路徑中的任何 schema 中物件，就只能透過使用限定名稱來指定其 schema 來引用。

search\_path 的內容必須是逗號分隔的 schema 名稱列表。任何非現有 schema 的名稱，或是使用者不具有 USAGE 權限的 schema，都將被忽略。

如果其中一個項目是特殊名稱 $user，則會使用 SESSION\_USER 回傳的名稱作為 schema 名稱，確認該 schema 存在且使用者具有 USAGE 權限。 （如果沒有權限，$user 將被忽略。）

系統目錄 pg\_catalog 一定會被搜尋，無論是否列在搜尋路徑中。如果列在搜尋路徑中了，那麼它將按照指定的順序被搜尋。 如果 pg\_catalog 不在搜尋路徑中，那麼它將會優先被搜尋。

同樣地，目前連線的臨時資料表的schema，pg\_temp\_nnn，如果它存在的話，就一定會被搜尋。它可以透過使用別名 pg\_temp 明確列在搜尋路徑中。如果沒有在搜尋路徑中列出的話，則優先搜尋（在 pg\_catalog 之前）。但是，臨時 schema 只是搜索關連（資料表、view，序列等）和資料型別名稱。不會搜尋函數或運算子名稱。

建立物件時沒有指定特定的 schema，那麼它們將被放置在 search\_path 中的第一個有效 schema 中。如果搜尋路徑為空，則會產生錯誤。

這個參數的預設值是 “$user”，public。此設定用來支援共享資料庫，沒有使用者具有私有 schema、所有共享使用 public、私人自有 schema ，以及以上情境的組合。其他的需求也可以透過更改預設的搜索路徑設置來達到，無論是全域或自有搜尋路徑。

搜尋路徑的目前內容可以使用 SQL 函數 current\_schemas 來檢查（詳見 [9.25 節](/ii-the-sql-language/functions-and-operators/925-system-information-functions.md)）。這與檢查 search\_path 的內容並不完全相同，因為 current\_schemas 表示 search\_path 中出現的項目是如何解析的。

有關 schema 處理的更多訊息，請參見第 [5.8 節](/ii-the-sql-language/data-definition/58-schemas.md)。

`row_security`\(`boolean`\)

此參數控制在資料列安全原則檢查時是否進行錯誤中斷。設定為 on 時，安全原則以正常方式運作。當設定為 off 時，除非查詢失敗，否則會至少符合一個原則。 預設值為 on。變更為 off 時，將會限制資料列的可視性，而可能造成不正確的結果；例如，pg\_dump 就會變更其預設值。此參數對於可以繞過每個安全原則的角色，也就是對具有 BYPASSRLS 屬性的超級使用者和角色都不會產生影響。

有關於資料列安全原則的更多訊息，請參閱 [CREATE POLICY](/vi-reference/i-sql-commands/create-policy.md)。

`default_tablespace`\(`string`\)

此參數指的是在 CREATE 指令未明確指定資料表空間（tablespace）時用於建立的資料庫物件（資料表和索引）的預設資料表空間。

該值可以是資料表空間的名稱，也可以是使用空字串表示為目前資料庫的預設資料表空間。如果該值與不符合任何現有的資料表空間名稱時，PostgreSQL 將自動使用目前資料庫的預設資料表空間。如果指定了非預設的資料表空間，則使用者必須具有 CREATE 權限，否則建立的操作將會失敗。

這個參數不用於臨時資料表；對於臨時資料表來說，會參考 temp\_tablespaces 參數。

建立資料庫時也不會使用這個參數。預設情況下，新的資料庫將複製的樣板資料庫，並繼承其資料表空間的設定。

有關於資料表空間的更多資訊，請參閱[第 22.6 節](/iii-server-administration/226-tablespaces.md)。

`temp_tablespaces`\(`string`\)

此參數指定在 CREATE 指令未指定資料表空間時創立臨時物件（臨時資料表和臨時資料表的索引）的資料表空間。用於排序大量資料集的臨時檔案也在這些資料表空間中創立。

該內容是資料表空間名稱的列表。當列表中有多個名稱時，PostgreSQL 在每次建立臨時物件時都會隨機選擇一個列表成員；除非是在一個交易中，連續建立的臨時物件將會被放置在列表的後續資料表空間中。 如果列表的元素是空字串，PostgreSQL 將自動使用目前資料庫的預設資料表空間。

設定 temp\_tablespaces 時，指定一個不存在的資料表空間會造成錯誤，因為指定一個使用者沒有 CREATE 權限的資料表空間。但是，使用先前設定的內容時，不存在的資料表空間將被忽略，使用者缺少 CREATE 權限的資料表空間也將被忽略。特別是，在使用 postgresql.conf 中設定的內容時，此規則適用。

預設值是一個空字串，這將會使用目前資料庫的預設資料空間中建立所有臨時物件。

另請參閱本頁的 default\_tablespace。

`check_function_bodies`\(`boolean`\)

這個參數通常是啓用（on）的。如果把它關閉（off）的話，將在 [CREATE FUNCTION](/vi-reference/i-sql-commands/create-function.md) 時關閉函數內容檢驗的措施。停用檢驗可避免檢驗過程的副作用，避免由於物件引用等問題所導致的誤報。例如以其他使用者載入函數之前，將此參數設置為 off；pg\_dump 將會自動執行此操作。

`default_transaction_isolation`\(`enum`\)

每組 SQL 交易查詢都有一個隔離的等級，可以是「read uncommitted」、「read committed」、「repeatable read」或「serializable」。此參數控制每個新的交易產生時的預設隔離等級。預設是「read committed」。

請參閱[第 13 章](/ii-the-sql-language/concurrency-control.md)和 [SET TRANSACTION](/vi-reference/i-sql-commands/set-transaction.md) 以取得更多訊息。

`default_transaction_read_only`\(`boolean`\)

一個唯讀的 SQL 交易不能更新非臨時的資料表。此參數控制每個新的交易的預設為唯讀狀態。預設是關閉（off）的（可讀／可寫）。

請參閱 [SET TRANSACTION](/vi-reference/i-sql-commands/set-transaction.md) 以取得更多訊息。

`default_transaction_deferrable`\(`boolean`\)

以 serializable 的隔離等級執行時，可延遲的唯讀 SQL 交易可能會被延遲，稍後才允許繼續。但是，一旦開始執行，就不會產生確保可序列化所需的任何成本；所以序列化代碼將不會因為同步更新而強制中止，使得這個選項適合用於長時間運行的唯讀交易。

此參數控制每個新交易查詢的預設可延期狀態。它目前對讀寫交易或者低於 serializable 隔離等級的操作沒有影響。預設是關閉（off）的。

請參閱 [SET TRANSACTION](/vi-reference/i-sql-commands/set-transaction.md) 以取得更多訊息。

`session_replication_role`\(`enum`\)

控制目前連線與複寫相關觸發器與規則。設定此參數需要超級使用者權限，會導致放棄任何先前快取的查詢計劃。可能的值是 origin（預設）、replica 和 local。 有關更多訊息，請參閱 [ALTER TABLE](/vi-reference/i-sql-commands/alter-table.md)。

`statement_timeout`\(`integer`\)

任何指令執行超過指定的時間時，就會中止其執行。時間單位為 millisecond（毫秒）。以伺服器接受到的時間起算。 如果 log\_min\_error\_statement 設定為 ERROR 或更低的等級時，則超時的查詢語句將被記錄下來。設定值為零（預設值），將其關閉功能。

不建議在 postgresql.conf 中設定 statement\_timeout，因為它會影響所有的連線。

`lock_timeout`\(`integer`\)

當你企圖鎖定資料表、索引、資料列或其他資料庫物件上時，任何等待超過指定的毫秒數的語句都會被強制中止。時間限制會分別適用於每次鎖定取得的嘗試。此限制適用於明確的鎖定請求（例如 LOCK TABLE 或 SELECT FOR UPDATE without NOWAIT）以及隱含的鎖定請求。如果將 log\_min\_error\_statement 設定為 ERROR 或更低的等級時，則會記錄超時的語查詢句。設定值為零（預設值），將其關閉功能。

與 statement\_timeout 不同，這個超時設定只會在等待鎖定的時候有作用。請注意，如果 statement\_timeout 不為零，則將 lock\_timeout 設定為相同或更大的值是毫無意義的，因為查詢語句超時總是會首先觸發。

不建議在 postgresql.conf 中設定 lock\_timeout，因為這會影響所有的連線。

`idle_in_transaction_session_timeout`\(`integer`\)

如果空閒時間超過指定的持續時間時（以毫秒為單位）未完成的交易將會被終止。這會釋放該連線所持有的任何鎖定，並使連線可以重新使用；也只有 tuple 才能看到這個交易被清除。有關這方面的更多細節，請參閱[第 24.1 節](/iii-server-administration/routine-database-maintenance-tasks/241-routine-vacuuming.md)。

預設值 0 表停用此功能。

`vacuum_freeze_table_age`\(`integer`\)

如果資料表的 pg\_class.relfrozenxid 欄位值已達到此設定的指定時間，VACUUM 將主動執行掃描。主動的掃描不同於一般的 VACUUM，因為它會訪問每個可能包含解開的 XID 或 MXID的頁面，而不僅僅是那些可能包含廢棄 tuple 的頁面。預設是 1.5 億筆交易。 儘管使用者可以設定的範圍為 0 到 20 億，但 VACUUM 將自動地將有效值限制為 [autovacuum\_freeze\_max\_age](/iii-server-administration/server-configuration/1910-automatic-vacuuming.md) 的 95%，以便在啟動資料表的 anti-wraparound 自動清理之前，定期的手動 VACUUM 有機會運行。欲了解更多訊息，請參閱[第 24.1.5 節](/iii-server-administration/routine-database-maintenance-tasks/241-routine-vacuuming.md)。

`vacuum_freeze_min_age`\(`integer`\)

指定 VACUUM 是否決定在掃描資料表時凍結資料列版本的截止期限（交易中）。預設是5000萬交易。 儘管使用者可以設定此值為 0 到 10 億之間的任何值，但 VACUUM 將自動地將有效值限制為 [autovacuum\_freeze\_max\_age](/iii-server-administration/server-configuration/1910-automatic-vacuuming.md) 值的一半，以便在強制自動清理之間沒有過短的不合理時間間隔。欲了解更多訊息，請參閱[第 24.1.5 節](/iii-server-administration/routine-database-maintenance-tasks/241-routine-vacuuming.md)。

`vacuum_multixact_freeze_table_age`\(`integer`\)

如果資料表的 pg\_class.relminmxid 欄位值已達到此設定指定的時間，VACUUM 將主動執行掃描。主動的掃描不同於一般的 VACUUM，因為它會訪問每個可能包含解開的 XID 或 MXID 的頁面，而不僅僅是那些可能包含廢棄 tuple 的頁面。預設值是 1.5 億個交易。儘管使用者可以設定的範圍為 0 到 20 億，但 VACUUM 將自動地將有效值限制為 [autovacuum\_freeze\_max\_age](https://www.gitbook.com/book/pgsql-tw/documents/edit#)的 95%，以便在啟動資料表的 anti-wraparound 自動清理之前，定期的手動 VACUUM 有機會運行。欲了解更多訊息，請參閱[第 24.1.5 節](https://www.gitbook.com/book/pgsql-tw/documents/edit#)。

`vacuum_multixact_freeze_min_age`\(`integer`\)

指定 VACUUM 在掃描資料表時是使用較新的 transaction ID 或是 multixact ID，來替換多個 multixact ID 的截斷年限（以 multixact 表示）。預設是500萬個 multixact。儘管使用者可以設定此值為 0 到 10 億之間的任何值，但 VACUUM 將自動地將有效值限制為 [autovacuum\_freeze\_max\_age](https://www.gitbook.com/book/pgsql-tw/documents/edit#) 值的一半，以便在強制自動清理之間沒有過短的不合理時間間隔。欲了解更多訊息，請參閱 [第 24.1.5.1 節](https://www.gitbook.com/book/pgsql-tw/documents/edit#)。

`bytea_output`\(`enum`\)

設定預設的輸出格式型別為`bytea。合法的設定值為 hex（預設）和 escape（傳統的 PostgreSQL 格式）。請參閱`[`第 8.4 節`](/ii-the-sql-language/data-types/84-binary-data-types.md)`取得更多資訊。無論這個設定如何，bytea 型別在輸入時，兩種格式都能接受。`

`xmlbinary`\(`enum`\)

設定如何在 XML 中編碼二進位數值。例如，當 bytea 值被函數 xmlelement 或 xmlforest 轉換為XML時，就適用這個設定。可以使用的值是 base64 和 hex，都是在 XML Schema 標準中定義的。 預設值是 base64。有關 XML 相關函數的更多訊息，請參閱[第 9.14 節](/ii-the-sql-language/functions-and-operators/914-xml-functions.md)。

實際上的選擇主要是習慣問題，僅受限於客戶端應用程式中的可能限制。這兩種方法都支援所有可能的值，儘管 hex 編碼會比 base64 編碼稍大。

`xmloption`\(`enum`\)

在 XML 和字串之間轉換時，設定是否隱含 DOCUMENT 或 CONTENT。請參閱 [8.13 節](/ii-the-sql-language/data-types/813-xml-type.md)的描述。有效值是 DOCUMENT 和 CONTENT。預設值是 CONTENT。

根據 SQL 標準，設定此選項的命令是

```
SET XML OPTION { DOCUMENT | CONTENT };
```

這個語法在 PostgreSQL 中也是可以使用的。

`gin_pending_list_limit`\(`integer`\)

設定啟用 fastupdate 時使用的 GIN 排程列表的最大空間。如果列表大於這個最大空間，則透過將其中的項目整批移動到主 GIN 資料結構來清除它。預設值是 4MB。透過更改索引的儲存參數，可以為單個 GIN 索引覆寫此設定。有關更多訊息，請參閱[第 64.4.1 節](/vii-internals/gin-indexes/644-implementation.md)和[第 64.5 節](/vii-internals/gin-indexes/645-gin-tips-and-tricks.md)。

### 19.11.2. 語系格式（Locale and Formatting）

`DateStyle`\(`string`\)

設定日期和時間內容的顯示格式，以及解釋模糊日期輸入的規則。由於歷史的因素，此參數包含兩個獨立的參數：輸出格式規範（ISO、Postgres、SQL 或 German）以及年/月/日次序（DMY、MDY 或 YMD）的輸入/輸出規範。它們可以單獨或一起設定。 關鍵字 Euro 和 European 是 DMY 的同義詞；關鍵字 US、NonEuro 和 NonEuropean 是 MDY 的同義詞。有關更多訊息，請參閱[第 8.5 節](/ii-the-sql-language/data-types/85-datetime-types.md)。 內建的預設值是 ISO、MDY，但是 initdb 會以使用所選的 lc\_time 語言環境相對應的設定來初始化設定內容。

`IntervalStyle`\(`enum`\)

設定間隔時間內容的顯示格式。設定為 sql\_standard 時，將產生合於 SQL 標準的間隔時間的輸出。當 DateStyle 參數設定為 ISO 時，設定為 postgres（預設值）將會產生與 8.4 之前的 PostgreSQL 版本相容輸出。當 DateStyle 參數設定為 non-ISO 時，設定為 postgres\_verbose 將生成與 8.4之前的 PostgreSQL 版本相容輸出。 設定為 iso\_8601 時，將產生 ISO 8601 中 4.4.3.2 節裡所定義的時間間隔「格式與標誌符」相容的輸出。

Interval Style 參數也會影響模糊區間輸入的解釋。有關更多訊息，請參閱[第 8.5.4 節](/ii-the-sql-language/data-types/85-datetime-types.md)。

`TimeZone`\(`string`\)

設定顯示和解釋時間戳記的時區。內建的預設值是 GMT，但通常會在 postgresql.conf 中被覆寫；initdb 將在安裝時取得其系統環境相對應的設定。 有關更多訊息，請參閱[第 8.5.3 節](/ii-the-sql-language/data-types/85-datetime-types.md)。

`timezone_abbreviations`\(`string`\)

設定日期時間輸入能被伺服器接受的時區縮寫集合。預設是「Default」，這是一個在世界大部分地區都可以使用的集合；還有「Australia」和「India」，並且可以為特定定義安裝其他集合。 更多訊息詳見 [B.3 節](/viii-appendixes/datetime-support/b3-datetime-configuration-files.md)。

`extra_float_digits`\(`integer`\)

This parameter adjusts the number of digits displayed for floating-point values, including`float4`,`float8`, and geometric data types. The parameter value is added to the standard number of digits \(`FLT_DIG`or`DBL_DIG`as appropriate\). The value can be set as high as 3, to include partially-significant digits; this is especially useful for dumping float data that needs to be restored exactly. Or it can be set negative to suppress unwanted digits. See also[Section 8.1.3](https://www.postgresql.org/docs/10/static/datatype-numeric.html#DATATYPE-FLOAT).

`client_encoding`\(`string`\)

Sets the client-side encoding \(character set\). The default is to use the database encoding. The character sets supported by thePostgreSQLserver are described in[Section 23.3.1](https://www.postgresql.org/docs/10/static/multibyte.html#MULTIBYTE-CHARSET-SUPPORTED).

`lc_messages`\(`string`\)

Sets the language in which messages are displayed. Acceptable values are system-dependent; see[Section 23.1](https://www.postgresql.org/docs/10/static/locale.html)for more information. If this variable is set to the empty string \(which is the default\) then the value is inherited from the execution environment of the server in a system-dependent way.

On some systems, this locale category does not exist. Setting this variable will still work, but there will be no effect. Also, there is a chance that no translated messages for the desired language exist. In that case you will continue to see the English messages.

Only superusers can change this setting, because it affects the messages sent to the server log as well as to the client, and an improper value might obscure the readability of the server logs.

`lc_monetary`\(`string`\)

Sets the locale to use for formatting monetary amounts, for example with the`to_char`family of functions. Acceptable values are system-dependent; see[Section 23.1](https://www.postgresql.org/docs/10/static/locale.html)for more information. If this variable is set to the empty string \(which is the default\) then the value is inherited from the execution environment of the server in a system-dependent way.

`lc_numeric`\(`string`\)

Sets the locale to use for formatting numbers, for example with the`to_char`family of functions. Acceptable values are system-dependent; see[Section 23.1](https://www.postgresql.org/docs/10/static/locale.html)for more information. If this variable is set to the empty string \(which is the default\) then the value is inherited from the execution environment of the server in a system-dependent way.

`lc_time`\(`string`\)

Sets the locale to use for formatting dates and times, for example with the`to_char`family of functions. Acceptable values are system-dependent; see[Section 23.1](https://www.postgresql.org/docs/10/static/locale.html)for more information. If this variable is set to the empty string \(which is the default\) then the value is inherited from the execution environment of the server in a system-dependent way.

`default_text_search_config`\(`string`\)

Selects the text search configuration that is used by those variants of the text search functions that do not have an explicit argument specifying the configuration. See[Chapter 12](https://www.postgresql.org/docs/10/static/textsearch.html)for further information. The built-in default is`pg_catalog.simple`, butinitdbwill initialize the configuration file with a setting that corresponds to the chosen`lc_ctype`locale, if a configuration matching that locale can be identified.

### 19.11.3. Shared Library Preloading

Several settings are available for preloading shared libraries into the server, in order to load additional functionality or achieve performance benefits. For example, a setting of`'$libdir/mylib'`would cause`mylib.so`\(or on some platforms,`mylib.sl`\) to be preloaded from the installation's standard library directory. The differences between the settings are when they take effect and what privileges are required to change them.

PostgreSQLprocedural language libraries can be preloaded in this way, typically by using the syntax`'$libdir/plXXX'`where`XXX`is`pgsql`,`perl`,`tcl`, or`python`.

Only shared libraries specifically intended to be used with PostgreSQL can be loaded this way. Every PostgreSQL-supported library has a“magic block”that is checked to guarantee compatibility. For this reason, non-PostgreSQL libraries cannot be loaded in this way. You might be able to use operating-system facilities such as`LD_PRELOAD`for that.

In general, refer to the documentation of a specific module for the recommended way to load that module.

`local_preload_libraries`\(`string`\)

This variable specifies one or more shared libraries that are to be preloaded at connection start. It contains a comma-separated list of library names, where each name is interpreted as for the[LOAD](https://www.postgresql.org/docs/10/static/sql-load.html)command. Whitespace between entries is ignored; surround a library name with double quotes if you need to include whitespace or commas in the name. The parameter value only takes effect at the start of the connection. Subsequent changes have no effect. If a specified library is not found, the connection attempt will fail.

This option can be set by any user. Because of that, the libraries that can be loaded are restricted to those appearing in the`plugins`subdirectory of the installation's standard library directory. \(It is the database administrator's responsibility to ensure that only“safe”libraries are installed there.\) Entries in`local_preload_libraries`can specify this directory explicitly, for example`$libdir/plugins/mylib`, or just specify the library name —`mylib`would have the same effect as`$libdir/plugins/mylib`.

The intent of this feature is to allow unprivileged users to load debugging or performance-measurement libraries into specific sessions without requiring an explicit`LOAD`command. To that end, it would be typical to set this parameter using the`PGOPTIONS`environment variable on the client or by using`ALTER ROLE SET`.

However, unless a module is specifically designed to be used in this way by non-superusers, this is usually not the right setting to use. Look at[session\_preload\_libraries](https://www.postgresql.org/docs/10/static/runtime-config-client.html#GUC-SESSION-PRELOAD-LIBRARIES)instead.

`session_preload_libraries`\(`string`\)

This variable specifies one or more shared libraries that are to be preloaded at connection start. It contains a comma-separated list of library names, where each name is interpreted as for the[LOAD](https://www.postgresql.org/docs/10/static/sql-load.html)command. Whitespace between entries is ignored; surround a library name with double quotes if you need to include whitespace or commas in the name. The parameter value only takes effect at the start of the connection. Subsequent changes have no effect. If a specified library is not found, the connection attempt will fail. Only superusers can change this setting.

The intent of this feature is to allow debugging or performance-measurement libraries to be loaded into specific sessions without an explicit`LOAD`command being given. For example,[auto\_explain](https://www.postgresql.org/docs/10/static/auto-explain.html)could be enabled for all sessions under a given user name by setting this parameter with`ALTER ROLE SET`. Also, this parameter can be changed without restarting the server \(but changes only take effect when a new session is started\), so it is easier to add new modules this way, even if they should apply to all sessions.

Unlike[shared\_preload\_libraries](https://www.postgresql.org/docs/10/static/runtime-config-client.html#GUC-SHARED-PRELOAD-LIBRARIES), there is no large performance advantage to loading a library at session start rather than when it is first used. There is some advantage, however, when connection pooling is used.

`shared_preload_libraries`\(`string`\)

This variable specifies one or more shared libraries to be preloaded at server start. It contains a comma-separated list of library names, where each name is interpreted as for the[LOAD](https://www.postgresql.org/docs/10/static/sql-load.html)command. Whitespace between entries is ignored; surround a library name with double quotes if you need to include whitespace or commas in the name. This parameter can only be set at server start. If a specified library is not found, the server will fail to start.

Some libraries need to perform certain operations that can only take place at postmaster start, such as allocating shared memory, reserving light-weight locks, or starting background workers. Those libraries must be loaded at server start through this parameter. See the documentation of each library for details.

Other libraries can also be preloaded. By preloading a shared library, the library startup time is avoided when the library is first used. However, the time to start each new server process might increase slightly, even if that process never uses the library. So this parameter is recommended only for libraries that will be used in most sessions. Also, changing this parameter requires a server restart, so this is not the right setting to use for short-term debugging tasks, say. Use[session\_preload\_libraries](https://www.postgresql.org/docs/10/static/runtime-config-client.html#GUC-SESSION-PRELOAD-LIBRARIES)for that instead.

### Note

On Windows hosts, preloading a library at server start will not reduce the time required to start each new server process; each server process will re-load all preload libraries. However,`shared_preload_libraries`is still useful on Windows hosts for libraries that need to perform operations at postmaster start time.

### 19.11.4. Other Defaults

`dynamic_library_path`\(`string`\)

If a dynamically loadable module needs to be opened and the file name specified in the`CREATE FUNCTION`or`LOAD`command does not have a directory component \(i.e., the name does not contain a slash\), the system will search this path for the required file.

The value for`dynamic_library_path`must be a list of absolute directory paths separated by colons \(or semi-colons on Windows\). If a list element starts with the special string`$libdir`, the compiled-inPostgreSQLpackage library directory is substituted for`$libdir`; this is where the modules provided by the standardPostgreSQLdistribution are installed. \(Use`pg_config --pkglibdir`to find out the name of this directory.\) For example:

```
dynamic_library_path = '/usr/local/lib/postgresql:/home/my_project/lib:$libdir'
```

or, in a Windows environment:

```
dynamic_library_path = 'C:\tools\postgresql;H:\my_project\lib;$libdir'
```

The default value for this parameter is`'$libdir'`. If the value is set to an empty string, the automatic path search is turned off.

This parameter can be changed at run time by superusers, but a setting done that way will only persist until the end of the client connection, so this method should be reserved for development purposes. The recommended way to set this parameter is in the`postgresql.conf`configuration file.

`gin_fuzzy_search_limit`\(`integer`\)

Soft upper limit of the size of the set returned by GIN index scans. For more information see[Section 64.5](https://www.postgresql.org/docs/10/static/gin-tips.html).

---

[^1]:  [PostgreSQL: Documentation: 10: 19.11. Client Connection Defaults](https://www.postgresql.org/docs/10/static/runtime-config-client.html)
