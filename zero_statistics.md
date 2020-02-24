## ０件統計は何故だめなのか

ORACLEは統計情報を元に実行計画を立てますが、典型的におかしな実行計画になるケースとして０件の状態で統計情報を収集していることがよくあります。本ページでは実際に0件状態の統計で遅い実行計画を選択させ、何故おかしくなるのかを解説します。


## ０件統計のやばさがわかるスクリプト
以下を実行すると**索引スキャンではなくFULLスキャンの実行計画が採択されます**。データとしてはcol1=1で10万件中1件まで絞り込めるデータ分布のため100%索引のほうが早くなりますが、0件状態で統計情報をとると現状のバージョンでは不具合ではなく仕様としてこのような動作になります。

    --表作成
    create table b(col1 number ,col2 varchar2(100),col3 char(2000));
    
    --0件の状態で統計情報を収集
    exec dbms_stats.gather_table_stats(null,'B');
    
    --適当なデータをinsert
    begin
      for i in 1..100000 loop
        insert into b values(i,i,i);
      end loop;
      commit;
    end;
    /
    
    --索引を作成＋索引の統計収集
    create index b_ix01 on b(col1,col2,col3);
    
    set autot on
    select count(*) from B where col1 = 1;
    ★索引を使わずFULLスキャンになってしまう。
    実行計画
    ----------------------------------------------------------
    Plan hash value: 749587668
    
    ---------------------------------------------------------------------------
    | Id  | Operation          | Name | Rows  | Bytes | Cost (%CPU)| Time     |
    ---------------------------------------------------------------------------
    |   0 | SELECT STATEMENT   |      |     1 |    13 |     2   (0)| 00:00:01 |
    |   1 |  SORT AGGREGATE    |      |     1 |    13 |            |          |
    |*  2 |   TABLE ACCESS FULL| B    |     1 |    13 |     2   (0)| 00:00:01 |
    ---------------------------------------------------------------------------
    
    Predicate Information (identified by operation id):
    ---------------------------------------------------
    
       2 - filter("COL1"=1)
    
    
    統計
    ----------------------------------------------------------
              0  recursive calls
              0  db block gets
          33441  consistent gets
              0  physical reads
              0  redo size
            573  bytes sent via SQL*Net to client
            399  bytes received via SQL*Net from client
              2  SQL*Net roundtrips to/from client
              0  sorts (memory)
              0  sorts (disk)
              1  rows processed


## なぜFULLスキャンになるのか

ORACLEは統計情報を元に様々な実行計画で最もCOSTの低い実行計画を選択する動きをします。上記の例では**FULLスキャンの実行計画のCOSTが2**と非常に低いことがわかり、本来適切であろう索引スキャンをヒントで強制してみると **2より大きい336となっており索引スキャンよりもFULLスキャンのほうがCOSTが低い(早い)** と判断しているためFULLスキャンになっていることがわかります。

    SQL> select /*+ index(b) */ count(*) from B where col1 = 1;
    
    実行計画
    ----------------------------------------------------------
    Plan hash value: 2794869083
    
    ----------------------------------------------------------------------------
    | Id  | Operation         | Name   | Rows  | Bytes | Cost (%CPU)| Time     |
    ----------------------------------------------------------------------------
    |   0 | SELECT STATEMENT  |        |     1 |    13 |   336   (0)| 00:00:01 |
    |   1 |  SORT AGGREGATE   |        |     1 |    13 |            |          |
    |*  2 |   INDEX RANGE SCAN| B_IX01 |     1 |    13 |   336   (0)| 00:00:01 |
    ----------------------------------------------------------------------------
    
    Predicate Information (identified by operation id):
    ---------------------------------------------------
    
       2 - access("COL1"=1)
    
    
    統計
    ----------------------------------------------------------
              0  recursive calls
              0  db block gets
              3  consistent gets
              0  physical reads
              0  redo size
            573  bytes sent via SQL*Net to client
            415  bytes received via SQL*Net from client
              2  SQL*Net roundtrips to/from client
              0  sorts (memory)
              0  sorts (disk)
              1  rows processed

## なぜFULLスキャンのコストが異常に低いのか


<!--stackedit_data:
eyJoaXN0b3J5IjpbNzM2MDQ5Mzg4LC03NTc0OTQ0NDksMjU5Mz
I1NTk3LC01MDA2MDM4OTMsMTA3MTkxMTg1MywxMzQzMjU5MzQ1
XX0=
-->