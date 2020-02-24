## ０件統計は何故だめなのか

ORACLEは統計情報を元に実行計画を立てますが、０件の状態で収集した統計情報は非常に性能劣化リスクの高い実行計画を選定する可能性が高くなります。本ページでは実際に


##０件統計のやばさがわかる即興スクリプト

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


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE3MzU0MjIxOTldfQ==
-->