# impomysql_bugreports

This page lists all bugs found by impomysql (https://github.com/qaqcatz/impomysql_binary). Up to now, we have found 41 bugs in MySQL, MariaDB, TiDB and OceanBase, 39 of which have been confirmed by developers.

## MySQL

* #1 http://bugs.mysql.com/108936

  **Status**: Verified

  **Version**: 8.0.29, 8.0.30

  **Test case**

  ```sql
  drop table if exists t;
  create table t (c1 int);
  insert into t values (1), (2), (3);
  
  mysql> SELECT ~f1 FROM (SELECT REPEAT(1234567890, 3) AS f1 FROM t) AS t1; --sql1
  +------+
  | ~f1  |
  +------+
  |    0 |
  |    0 |
  |    0 |
  +------+
  3 rows in set, 3 warnings (0.00 sec)
  
  mysql> SELECT ~f1 FROM (SELECT DISTINCT REPEAT(1234567890, 3) AS f1 FROM t) AS t1; --sql2
  +---------------------+
  | ~f1                 |
  +---------------------+
  | 9223372036854775808 |
  +---------------------+
  1 row in set, 1 warning (0.00 sec)
  ```

* #2 http://bugs.mysql.com/108937

  **Status**: Verified

  **Version**: 5.6.17, 8.0.30

  **Test case**

  ```sql
  drop table if exists t;
  create table t (c1 double);
  insert into t values (79.1819),(12.991),(1);
  
  mysql> SELECT c1-DATE_SUB('2008-05-25', INTERVAL 1 HOUR_MINUTE) AS f1 FROM t HAVING f1 != 0; -- sql1
  +------------+
  | f1         |
  +------------+
  | -1928.8181 |
  |  -1995.009 |
  |      -2007 |
  +------------+
  3 rows in set (0.00 sec)
  
  mysql> SELECT c1-DATE_SUB('2008-05-25', INTERVAL 1 HOUR_MINUTE) AS f1 FROM t HAVING 1; -- sql2
  +---------------------+
  | f1                  |
  +---------------------+
  | -20080524235820.816 |
  | -20080524235887.008 |
  |     -20080524235899 |
  +---------------------+
  3 rows in set (0.00 sec)
  ```

* #3 http://bugs.mysql.com/108938

  **Status**: Verified

  **Version**: 5.7.6, 8.0.30

  **Test case**

  ```sql
  drop table if exists t;
  create table t (c1 double);
  insert into t values (-13064),(0),(71.051);
  
  mysql> SELECT f1 FROM (SELECT 1) AS t1 JOIN (SELECT (c1+DATE_SUB('2018-05-17', INTERVAL 1 DAY_MICROSECOND)) AS f1 FROM t) AS t2 ON f1 != 0; -- sql1
  +----------+
  | f1       |
  +----------+
  |   -11046 |
  |     2018 |
  | 2089.051 |
  +----------+
  3 rows in set (0.00 sec)
  
  mysql> SELECT f1 FROM (SELECT 1) AS t1 JOIN (SELECT (c1+DATE_SUB('2018-05-17', INTERVAL 1 DAY_MICROSECOND)) AS f1 FROM t) AS t2 ON 1; -- sql2
  +-------------------+
  | f1                |
  +-------------------+
  |  20180516222895.9 |
  |  20180516235959.9 |
  | 20180516236030.95 |
  +-------------------+
  3 rows in set (0.00 sec)
  ```

* #4 http://bugs.mysql.com/108946

  **Status**: Verified

  **Version**: 8.0.0, 8.0.30

  **Test case**

  ```sql
  drop table if exists t;
  create table t (c1 double);
  insert into t values (0.1);
  
  mysql> (SELECT (c1 DIV 1)*0.1 FROM t) UNION ALL (SELECT '1');
  +----------------+
  | (c1 DIV 1)*0.1 |
  +----------------+
  | 0              |
  | 1              |
  +----------------+
  2 rows in set (0.00 sec)
  
  mysql> (SELECT DISTINCT (c1 DIV 1)*0.1 FROM t) UNION ALL (SELECT '1');
  +----------------+
  | (c1 DIV 1)*0.1 |
  +----------------+
  | 0.0            |
  | 1              |
  +----------------+
  2 rows in set (0.00 sec)
  ```

* #5 https://bugs.mysql.com/bug.php?id=109351

  **Status**: Verified

  **Version**: 5.6.17, 8.0.31

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 int);
  INSERT INTO t VALUES (1);
  
  mysql> SELECT 1%`f1` FROM (SELECT DAYNAME('2010-07-20') AS `f1` FROM t WHERE DATE_ADD(TIMEDIFF('2000-08-04 02:09:39', '2002-08-11 22:00:35'), INTERVAL 1 YEAR) NOT IN (1)) AS `t2`;
  +--------+
  | 1%`f1` |
  +--------+
  |      0 |
  +--------+
  1 row in set, 1 warning (0.00 sec)
  
  mysql> SELECT 1%`f1` FROM (SELECT DISTINCT DAYNAME('2010-07-20') AS `f1` FROM t WHERE DATE_ADD(TIMEDIFF('2000-08-04 02:09:39', '2002-08-11 22:00:35'), INTERVAL 1 YEAR) NOT IN (1)) AS `t2`;
  +--------+
  | 1%`f1` |
  +--------+
  |   NULL |
  +--------+
  1 row in set, 3 warnings (0.00 sec)
  ```

* #6 https://bugs.mysql.com/bug.php?id=109352

  **Status**: Verified

  **Version**: 8.0.23, 8.0.31

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 FLOAT UNSIGNED);
  INSERT INTO t VALUES (0.0001);
  
  mysql> SELECT UCASE(`f1`) FROM (SELECT (-c1) AS `f1` FROM t) AS `t1`;
  +-----------------------+
  | UCASE(`f1`)           |
  +-----------------------+
  | -9.999999747378752E-5 |
  +-----------------------+
  1 row in set (0.00 sec)
  
  mysql> SELECT UCASE(`f1`) FROM (SELECT DISTINCT (-c1) AS `f1` FROM t) AS `t1`;
  +-------------------------+
  | UCASE(`f1`)             |
  +-------------------------+
  | -0.00009999999747378752 |
  +-------------------------+
  1 row in set (0.00 sec)
  ```

* #7 https://bugs.mysql.com/bug.php?id=109353

  **Status**: Verified

  **Version**: 8.0.22, 8.0.31

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 FLOAT UNSIGNED,c2 CHAR(20));
  INSERT INTO t VALUES (0,'0'),(0,'0'),(0,'0'),(0,'0'),(0,'0'),(0,'0'),(0,'-0');
  
  mysql> SELECT 1 FROM (SELECT c2 FROM t) AS `t1` WHERE (c2 IN (SELECT c1 FROM t)) AND (c2 NOT IN ('0','0',0.1));
  Empty set (0.00 sec)
  
  mysql> SELECT 1 FROM (SELECT DISTINCT c2 FROM t) AS `t1` WHERE (c2 IN (SELECT c1 FROM t)) AND (c2 NOT IN ('0','0',0.1));
  +---+
  | 1 |
  +---+
  | 1 |
  +---+
  1 row in set (0.00 sec)
  ```

* #8 https://bugs.mysql.com/bug.php?id=109354

  **Status**: Verified

  **Version**: 8.0.21, 8.0.31

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 int);
  INSERT INTO t VALUES (1);
  
  mysql> (SELECT ('2009-05-09 01:41:51'>>1) AS `f1`) UNION (SELECT `f2` FROM (SELECT (NULL) AS `f2` FROM (SELECT 1 FROM t) AS `t1`) AS `t2`);
  +------+
  | f1   |
  +------+
  | 1004 |
  | NULL |
  +------+
  2 rows in set, 1 warning (0.00 sec)
  
  mysql> (SELECT ('2009-05-09 01:41:51'>>1) AS `f1`) UNION (SELECT `f2` FROM (SELECT DISTINCT (NULL) AS `f2` FROM (SELECT 1 FROM t) AS `t1`) AS `t2`);
  +-------------------------------------+
  | f1                                  |
  +-------------------------------------+
  | 1004.000000000000000000000000000000 |
  |                                NULL |
  +-------------------------------------+
  2 rows in set, 1 warning (0.01 sec)
  ```

* #9 https://bugs.mysql.com/bug.php?id=109356

  **Status**: Verified

  **Version**: 8.0.16, 8.0.31

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 DECIMAL(40,20) UNSIGNED);
  INSERT INTO t VALUES (1);
  
  mysql> SELECT f1 FROM (SELECT FROM_DAYS(1) AS f1 FROM t) AS `t3` JOIN (SELECT c1 FROM t) AS `t4` ON ((NOT (f1<=ANY (SELECT c1 FROM t))) OR (c1 = 0)) IS TRUE;
  Empty set (0.00 sec)
  
  mysql> SELECT f1 FROM (SELECT DISTINCT FROM_DAYS(1) AS f1 FROM t) AS `t3` JOIN (SELECT c1 FROM t) AS `t4` ON ((NOT (f1<=ANY (SELECT c1 FROM t))) OR (c1 = 0)) IS TRUE;
  +------------+
  | f1         |
  +------------+
  | 0000-00-00 |
  +------------+
  1 row in set, 2 warnings (0.01 sec)
  ```

* #10 https://bugs.mysql.com/bug.php?id=109362

  **Status**: Verified

  **Version**: 8.0.13, 8.0.31

  **Test case**

  ```sql
  drop table if exists t;
  create table t (c1 double, c2 decimal(40, 20), key (c1));
  insert into t values (-13064,-2),(71.0510,12.991),(-0,47.1515);
  
  mysql> SELECT c1 FROM (SELECT c1, c2 FROM t) AS `t1` WHERE (c1 BETWEEN DAYNAME('2003-02-12') AND c1);
  +--------+
  | c1     |
  +--------+
  | 71.051 |
  +--------+
  1 row in set (0.00 sec)
  
  mysql> SELECT c1 FROM (SELECT DISTINCT c1, c2 FROM t) AS `t1` WHERE (c1 BETWEEN DAYNAME('2003-02-12') AND c1);
  +--------+
  | c1     |
  +--------+
  | 71.051 |
  |      0 |
  +--------+
  2 rows in set (0.00 sec)
  ```

* #11 https://bugs.mysql.com/bug.php?id=109363

  **Status**: Verified

  **Version**: 8.0.4, 8.0.31

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 FLOAT UNSIGNED);
  INSERT INTO t VALUES (1.009),(0.0001),(36.0002);
  
  mysql> (SELECT ~1) UNION ALL (SELECT c1 DIV 1.0*(-LAST_DAY('2011-08-03')) FROM t);
  +----------------------+
  | ~1                   |
  +----------------------+
  | 18446744073709552000 |
  |            -20110831 |
  |                    0 |
  |           -723989916 |
  +----------------------+
  4 rows in set (0.00 sec)
  
  mysql> (SELECT ~1) UNION ALL (SELECT DISTINCT c1 DIV 1.0*(-LAST_DAY('2011-08-03')) FROM t);
  +----------------------+
  | ~1                   |
  +----------------------+
  | 18446744073709552000 |
  |            -20110831 |
  |                   -0 |
  |           -723989916 |
  +----------------------+
  4 rows in set (0.00 sec)
  ```

* #12 https://bugs.mysql.com/bug.php?id=109367

  **Status**: Verified

  **Version**: 5.7.22, 8.0.31

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 DECIMAL(40,20));
  INSERT INTO t VALUES (-0),(120);
  
  mysql> SELECT (1^`f5`) AS `f3` FROM (SELECT c1 AS `f4` FROM t) AS `t1` JOIN (SELECT (DATE_ADD(c1, INTERVAL 1 DAY_SECOND)) AS `f5` FROM t) AS `t3` ON (FROM_DAYS(1)) NOT IN (`f4`,`f5`);
  +----------------+
  | f3             |
  +----------------+
  | 20000120000000 |
  +----------------+
  1 row in set, 1 warning (0.00 sec)
  
  mysql> SELECT (1^`f5`) AS `f3` FROM (SELECT c1 AS `f4` FROM t) AS `t1` JOIN (SELECT DISTINCT (DATE_ADD(c1, INTERVAL 1 DAY_SECOND)) AS `f5` FROM t) AS `t3` ON (FROM_DAYS(1)) NOT IN (`f4`,`f5`);
  +------+
  | f3   |
  +------+
  | 2001 |
  +------+
  1 row in set, 2 warnings (0.00 sec)
  ```

* #13 https://bugs.mysql.com/bug.php?id=109406

  **Status**: Verified

  **Version**: 5.7.11, 8.0.31

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 BIGINT UNSIGNED, c2 DECIMAL(40,20), key(c1));
  INSERT INTO t VALUES (2,120),(0,2);
  
  mysql> SELECT f1 FROM (SELECT DATE_ADD('2015-06-23', INTERVAL 1 MINUTE_SECOND)%c2 AS `f1` FROM t WHERE LN(0.5) NOT IN (SELECT c1 FROM t)) AS t1 WHERE `f1` != 1; -- sql1
  +------+
  | f1   |
  +------+
  |   95 |
  +------+
  1 row in set (0.00 sec)
  
  mysql> SELECT f1 FROM (SELECT DATE_ADD('2015-06-23', INTERVAL 1 MINUTE_SECOND)%c2 AS `f1` FROM t WHERE LN(0.5) NOT IN (SELECT c1 FROM t)) AS t1 WHERE 1; -- sql2 
  +------+
  | f1   |
  +------+
  |   41 |
  |    1 |
  +------+
  2 rows in set (0.00 sec)
  ```

* #14 https://bugs.mysql.com/bug.php?id=109407

  **Status**: Verified

  **Version**: 5.7.5, 8.0.31

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 DECIMAL(40,20) UNSIGNED);
  INSERT INTO t VALUES (1);
  
  mysql> select version();
  +-----------+
  | version() |
  +-----------+
  | 8.0.31    |
  +-----------+
  1 row in set (0.00 sec)
  
  mysql> SELECT 1 FROM t WHERE (NOT (FROM_DAYS(1)=ALL (SELECT c1 FROM t)));
  Empty set, 1 warning (0.00 sec)
  
  mysql> SELECT 1 FROM t WHERE (NOT (FROM_DAYS(1)>=ALL (SELECT c1 FROM t)));
  +---+
  | 1 |
  +---+
  | 1 |
  +---+
  1 row in set (0.00 sec)
  ```

## MariaDB

* #1 https://jira.mariadb.org/browse/MDEV-30249

  **Status**: Verified

  **Version**: 5.5.40, 10.11.1

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 FLOAT UNSIGNED);
  INSERT INTO t VALUES (0);
   
  mysql> SELECT f1 FROM (SELECT (c1-~LN(4)) AS f1 FROM t) AS t1 where f1 != 1;
  +------+
  | f1   |
  +------+
  |    2 |
  +------+
  1 row in set (0.00 sec)
   
  mysql> SELECT f1 FROM (SELECT (c1-~LN(4)) AS f1 FROM t) AS t1 where 1;
  +------------------------+
  | f1                     |
  +------------------------+
  | -1.8446744073709552e19 |
  +------------------------+
  1 row in set (0.00 sec)
  ```

* #2 https://jira.mariadb.org/browse/MDEV-30250

  **Status**: Verified

  **Version**: 5.5.54, 10.11.1

  **Test case**

  ```sql
  drop table if exists t;
  create table t (c1 double);
  insert into t values (-13064),(0),(71.051);
  
  mysql> SELECT f1 FROM (SELECT 1) AS t1 JOIN (SELECT (c1+DATE_SUB('2018-05-17', INTERVAL 1 DAY_MICROSECOND)) AS f1 FROM t) AS t2 ON f1 != 0; -- sql1
  +----------+
  | f1       |
  +----------+
  |   -11046 |
  |     2018 |
  | 2089.051 |
  +----------+
  3 rows in set (0.00 sec)
   
  mysql> SELECT f1 FROM (SELECT 1) AS t1 JOIN (SELECT (c1+DATE_SUB('2018-05-17', INTERVAL 1 DAY_MICROSECOND)) AS f1 FROM t) AS t2 ON 1; -- sql2
  +-------------------+
  | f1                |
  +-------------------+
  |  20180516222895.9 |
  |  20180516235959.9 |
  | 20180516236030.95 |
  +-------------------+
  3 rows in set (0.00 sec)
  ```

* #3 https://jira.mariadb.org/browse/MDEV-30251

  **Status**: Verified

  **Version**: 5.5.61, 10.11.1

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 CHAR(20));
  INSERT INTO t VALUES ('1');
  
  mysql> SELECT c1%'a' AS `f1` FROM (SELECT c1 FROM t) AS `t1` WHERE (CONCAT_WS(0, 0.01, c1)) OR (NULL>=ALL (SELECT 1 FROM t)) HAVING NOT ((`f1` != 1) IS FALSE) ORDER BY c1; -- sql1
  +------+
  | f1   |
  +------+
  | NULL |
  +------+
  1 row in set, 4 warnings (0.00 sec)
   
  mysql> SELECT c1%'a' AS `f1` FROM (SELECT c1 FROM t) AS `t1` WHERE (CONCAT_WS(0, 0.01, c1)) OR (NULL>=ALL (SELECT 1 FROM t)) HAVING 1 ORDER BY c1; -- sql2
  Empty set, 1 warning (0.00 sec)
  ```

* #4 https://jira.mariadb.org/browse/MDEV-30252

  **Status**: Verified

  **Version**: 5.5.62, 10.11.1

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 INT);
  INSERT INTO t VALUES (1);
  
  mysql> SELECT PI()+~1&f2 AS f1 FROM (SELECT DATE_SUB(DATE_ADD('2009-10-15', INTERVAL 1 MICROSECOND), INTERVAL 1 DAY_SECOND) AS f2 FROM t) AS t1; -- sql1
  +----------------+
  | f1             |
  +----------------+
  | 20091014235959 |
  +----------------+
  1 row in set, 1 warning (0.00 sec)
   
  mysql> SELECT PI()+~1&f2 AS f1 FROM (SELECT DISTINCT DATE_SUB(DATE_ADD('2009-10-15', INTERVAL 1 MICROSECOND), INTERVAL 1 DAY_SECOND) AS f2 FROM t) AS t1; -- sql2
  +------+
  | f1   |
  +------+
  | 2009 |
  +------+
  1 row in set, 2 warnings (0.00 sec)
  ```

* #5 https://jira.mariadb.org/browse/MDEV-30253

  **Status**: Verified

  **Version**: 10.0.15, 10.11.1

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 DECIMAL(40,20));
  INSERT INTO t VALUES (1);
  
  mysql> SELECT ~f1 FROM (SELECT (DATE_ADD(REPEAT(c1, 3), INTERVAL 1 MINUTE_MICROSECOND)) AS f1 FROM t) AS t1; -- sql1
  +----------------------+
  | ~f1                  |
  +----------------------+
  | 18446744063608551615 |
  +----------------------+
  1 row in set (0.00 sec)
   
  mysql> SELECT ~f1 FROM (SELECT DISTINCT (DATE_ADD(REPEAT(c1, 3), INTERVAL 1 MINUTE_MICROSECOND)) AS f1 FROM t) AS t1; -- sql2
  +----------------------+
  | ~f1                  |
  +----------------------+
  | 18446744073709551614 |
  +----------------------+
  1 row in set, 1 warning (0.00 sec)
  ```

* #6 https://jira.mariadb.org/browse/MDEV-30254

  **Status**: Verified

  **Version**: 10.1.10, 10.11.1

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 DECIMAL(40,20));
  INSERT INTO t VALUES (-0),(120);
  
  mysql> SELECT (1^`f5`) AS `f3` FROM (SELECT c1 AS `f4` FROM t) AS `t1` JOIN (SELECT (DATE_ADD(c1, INTERVAL 1 DAY_SECOND)) AS `f5` FROM t) AS `t3` ON (FROM_DAYS(1)) NOT IN (`f4`,`f5`);
  +----------------+
  | f3             |
  +----------------+
  | 20000120000000 |
  | 20000120000000 |
  +----------------+
  2 rows in set, 4 warnings (0.00 sec)
   
  mysql> SELECT (1^`f5`) AS `f3` FROM (SELECT c1 AS `f4` FROM t) AS `t1` JOIN (SELECT DISTINCT (DATE_ADD(c1, INTERVAL 1 DAY_SECOND)) AS `f5` FROM t) AS `t3` ON (FROM_DAYS(1)) NOT IN (`f4`,`f5`);
  +------+
  | f3   |
  +------+
  | 2001 |
  | 2001 |
  +------+
  2 rows in set, 4 warnings (0.00 sec)
  ```

* #7 https://jira.mariadb.org/browse/MDEV-30255

  **Status**: Verified

  **Version**: 10.1.37, 10.11.1

  **Test case**

  ```sql
  drop table if exists t;
  create table t (c1 double);
  insert into t values (0.1);
  
  mysql> (SELECT (c1 DIV 1)*0.1 FROM t) UNION ALL (SELECT '1');
  +----------------+
  | (c1 DIV 1)*0.1 |
  +----------------+
  | 0              |
  | 1              |
  +----------------+
  2 rows in set (0.00 sec)
   
  mysql> (SELECT DISTINCT (c1 DIV 1)*0.1 FROM t) UNION ALL (SELECT '1');
  +----------------+
  | (c1 DIV 1)*0.1 |
  +----------------+
  | 0.0            |
  | 1              |
  +----------------+
  2 rows in set (0.00 sec)
  ```

* #8 https://jira.mariadb.org/browse/MDEV-30257

  **Status**: Verified

  **Version**: 10.3.12, 10.11.1

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 DECIMAL(40,20));
  INSERT INTO t VALUES (-0.0001);
  
  mysql> SELECT (LAST_DAY('2004-12-20 21:48:16') DIV c1) AS f1 FROM t; -- sql1
  +---------------+
  | f1            |
  +---------------+
  | -200412310000 |
  +---------------+
  1 row in set (0.00 sec)
   
  mysql> SELECT DISTINCT (LAST_DAY('2004-12-20 21:48:16') DIV c1) AS f1 FROM t; -- sql2
  +-------------+
  | f1          |
  +-------------+
  | -2147483648 |
  +-------------+
  1 row in set (0.00 sec)
  ```

* #9 https://jira.mariadb.org/browse/MDEV-30258

  **Status**: Open

  **Version**: 10.3.22, 10.11.1

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 INT);
  INSERT INTO t VALUES (1);
  
  mysql> SELECT (-DAYNAME('2008-07-15')-f2 DIV 1) AS f1 FROM (SELECT 0 AS f2 FROM t HAVING 1) AS t1; -- sql1
  +------+
  | f1   |
  +------+
  |   -0 |
  +------+
  1 row in set, 1 warning (0.00 sec)
   
  mysql> SELECT DISTINCT (-DAYNAME('2008-07-15')-f2 DIV 1) AS f1 FROM (SELECT 0 AS f2 FROM t HAVING 1) AS t1; -- sql2
  +------+
  | f1   |
  +------+
  |    0 |
  +------+
  1 row in set, 1 warning (0.00 sec)
  ```

* #10 https://jira.mariadb.org/browse/MDEV-30298

  **Status**: Verified

  **Version**: 10.5.1, 10.11.1

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 DOUBLE);
  INSERT INTO t VALUES (0.0001);
  
  mysql> SELECT -f2 AS f1 FROM (SELECT (BINARY c1) AS f2 FROM t) AS t1; -- sql1
  +------+
  | f1   |
  +------+
  |  -0. |
  +------+
  1 row in set (0.00 sec)
   
  mysql> SELECT -f2 AS f1 FROM (SELECT DISTINCT (BINARY c1) AS f2 FROM t) AS t1; -- sql2
  +---------+
  | f1      |
  +---------+
  | -0.0001 |
  +---------+
  1 row in set (0.00 sec)
  ```

* #11 https://jira.mariadb.org/browse/MDEV-30299

  **Status**: Verified

  **Version**: 10.5.3, 10.11.1

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 FLOAT,c2 VARCHAR(20),key(c1));
  INSERT INTO t VALUES (94.1106,'-0'),(1,'3	'),(0.0001,'-1');
  
  mysql> SELECT 1 FROM (SELECT c1 AS f1 FROM t) AS t1 WHERE ((-f1)=ANY (SELECT c2 FROM t)) OR ((~1>=ALL (SELECT c1 FROM t)) IS FALSE); -- sql1
  +---+
  | 1 |
  +---+
  | 1 |
  +---+
  1 row in set (0.01 sec)
   
  mysql> SELECT 1 FROM (SELECT c1 AS f1 FROM t) AS t1 WHERE ((-f1)>=ANY (SELECT c2 FROM t)) OR ((~1>=ALL (SELECT c1 FROM t)) IS FALSE); -- sql2
  Empty set (0.00 sec)
  ```

* #12 https://jira.mariadb.org/browse/MDEV-30300

  **Status**: Verified

  **Version**: 10.5.4, 10.11.1

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 VARCHAR(20));
  INSERT INTO t VALUES ('\n3');
  
  mysql> SELECT (~c1*ABS(0.8)|DATE_ADD('2017-11-22', INTERVAL 1 WEEK)) AS `f1` FROM t HAVING !(`f1`=1); -- sql1
  +----------------------+
  | f1                   |
  +----------------------+
  | 14757395258967642091 |
  +----------------------+
  1 row in set, 2 warnings (0.01 sec)
   
  mysql> SELECT (~c1*ABS(0.8)|DATE_ADD('2017-11-22', INTERVAL 1 WEEK)) AS `f1` FROM t HAVING 1; -- sql2
  +----------------------+
  | f1                   |
  +----------------------+
  | 14757395258987761147 |
  +----------------------+
  1 row in set (0.00 sec)
  ```

* #13 https://jira.mariadb.org/browse/MDEV-30301

  **Status**: Open

  **Version**: 10.5.9, 10.11.1

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 FLOAT UNSIGNED,c2 INT);
  INSERT INTO t VALUES (0.0001,1);
  
  mysql> (SELECT c1 AS f1 FROM t) UNION ALL (SELECT REPLACE('what', f2, f2) AS f1 FROM (SELECT !c2 AS f2 FROM t) AS t1); -- sql1
  +--------------+
  | f1           |
  +--------------+
  | 9.9999997e-5 |
  | what         |
  +--------------+
  2 rows in set (0.00 sec)
   
  mysql> (SELECT c1 AS f1 FROM t) UNION ALL (SELECT REPLACE('what', f2, f2) AS f1 FROM (SELECT DISTINCT !c2 AS f2 FROM t) AS t1); -- sql2
  +------------------------+
  | f1                     |
  +------------------------+
  | 0.00009999999747378752 |
  | what                   |
  +------------------------+
  2 rows in set (0.00 sec)
  ```

* #14 https://jira.mariadb.org/browse/MDEV-30302

  **Status**: Verified

  **Version**: 10.5.11, 10.11.1

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 CHAR(20));
  INSERT INTO t VALUES ('0');
  
  mysql> WITH `MYWITH` AS (SELECT (BINARY f2-UNIX_TIMESTAMP('2011-12-22 14:22:02')) AS f1 FROM (SELECT (~COERCIBILITY(c1)) AS f2 FROM t) AS t1 WHERE (DATE_ADD(BIN(f2), INTERVAL 1 WEEK) NOT IN (SELECT 1 FROM t)) AND (f2 BETWEEN '1' AND f2)) SELECT * FROM `MYWITH`; -- sql1
  +----------------------+
  | f1                   |
  +----------------------+
  | 18446744072384987000 |
  +----------------------+
  1 row in set, 6 warnings (0.00 sec)
   
  mysql> WITH `MYWITH` AS (SELECT DISTINCT (BINARY f2-UNIX_TIMESTAMP('2011-12-22 14:22:02')) AS f1 FROM (SELECT (~COERCIBILITY(c1)) AS f2 FROM t) AS t1 WHERE (DATE_ADD(BIN(f2), INTERVAL 1 WEEK) NOT IN (SELECT 1 FROM t)) AND (f2 BETWEEN '1' AND f2)) SELECT * FROM `MYWITH`; -- sql2
  +--------------------+
  | f1                 |
  +--------------------+
  | 100000000000000000 |
  +--------------------+
  1 row in set, 6 warnings (0.00 sec)
  ```

## TiDB

* #1 https://github.com/pingcap/tidb/issues/38756

  **Status**: Verified

  **Version**: 5.1.0, 6.3.0

  **Test case**

  ```sql
  drop table if exists t;
  create table t (c1 int);
  insert into t values (1), (2), (3);
  
  mysql> (SELECT SQRT(1) FROM t); -- sql1
  +---------+
  | SQRT(1) |
  +---------+
  |       1 |
  |       1 |
  |       1 |
  +---------+
  3 rows in set (0.00 sec)
  
  mysql> (SELECT DISTINCT SQRT(1) FROM t); -- sql2
  +---------+
  | SQRT(1) |
  +---------+
  |  5e-324 |
  +---------+
  1 row in set (0.00 sec)
  ```

* #2 https://github.com/pingcap/tidb/issues/38747

  **Status**: Verified

  **Version**: 5.0.0, 6.3.0

  **Test case**

  ```sql
  drop table if exists t;
  create table t (c1 double);
  insert into t values (0.0001),(-1),(12.991),(2),(1.009);
  
  SELECT (DATE_SUB(BIN(f1), INTERVAL 1 HOUR_MINUTE)) FROM (SELECT 1 FROM t) AS t1 
  JOIN (SELECT (REVERSE(c1)) AS f1 FROM t HAVING NOT (f1 LIKE '%0%')) AS t2; -- sql1
  SELECT (DATE_SUB(BIN(f1), INTERVAL 1 HOUR_MINUTE)) FROM (SELECT 1 FROM t) AS t1 
  JOIN (SELECT (REVERSE(c1)) AS f1 FROM t HAVING 1) AS t2; -- sql2
  
  -- The results are too long, see the bug report url
  ```

* #3 https://github.com/pingcap/tidb/issues/38744

  **Status**: Verified

  **Version**: 6.3.0, 6.3.0

  **Test case**

  ```sql
  drop table if exists t;
  create table t (`pk` int primary key, c1 varchar(20), key (c1)) character set utf8 partition by hash(pk) partitions 2;
  insert into t values (0,'e'),(1,'-0'),(2,'e');
  
  -- unstable bug
  mysql> (SELECT NULL FROM t) UNION (SELECT (-c1) FROM t);
  +------+
  | NULL |
  +------+
  | NULL |
  |    0 |
  +------+
  2 rows in set, 4 warnings (0.01 sec)
  
  mysql> (SELECT NULL FROM t) UNION (SELECT (-c1) FROM t);
  +------+
  | NULL |
  +------+
  | NULL |
  |   -0 |
  +------+
  2 rows in set, 4 warnings (0.00 sec)
  ```

* #4 https://github.com/pingcap/tidb/issues/40011

  **Status**: Verified

  **Version**: 6.2.0, 6.4.0

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 int);
  INSERT INTO t VALUES (1);
  
  mysql> SELECT 1%`f1` FROM (SELECT DAYNAME('2010-07-20') AS `f1` FROM t WHERE DATE_ADD(TIMEDIFF('2000-08-04 02:09:39', '2002-08-11 22:00:35'), INTERVAL 1 YEAR) NOT IN (1)) AS `t2`;
  +--------+
  | 1%`f1` |
  +--------+
  |      0 |
  +--------+
  1 row in set, 1 warning (0.00 sec)
  
  mysql> SELECT 1%`f1` FROM (SELECT DISTINCT DAYNAME('2010-07-20') AS `f1` FROM t WHERE DATE_ADD(TIMEDIFF('2000-08-04 02:09:39', '2002-08-11 22:00:35'), INTERVAL 1 YEAR) NOT IN (1)) AS `t2`;
  +--------+
  | 1%`f1` |
  +--------+
  |   NULL |
  +--------+
  1 row in set, 3 warnings (0.00 sec)
  ```

* #5 https://github.com/pingcap/tidb/issues/40012

  **Status**: Verified

  **Version**: 6.0.0, 6.4.0

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 INT);
  INSERT INTO t VALUES (1);
  
  mysql> SELECT 0.1 AS f1 FROM (SELECT 1 FROM t) AS `t1` HAVING (DATE_SUB(-1, INTERVAL 1 DAY_MINUTE) != f1) OR 1;
  +----------------------------------+
  | f1                               |
  +----------------------------------+
  | 0.100000000000000000000000000000 |
  +----------------------------------+
  1 row in set, 1 warning (0.00 sec)
  
  mysql> SELECT 0.1 AS f1 FROM (SELECT 1 FROM t) AS `t1` HAVING 1;
  +-----+
  | f1  |
  +-----+
  | 0.1 |
  +-----+
  1 row in set (0.00 sec)
  ```

* #6 https://github.com/pingcap/tidb/issues/40013

  **Status**: Verified

  **Version**: 5.3.4, 6.4.0

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 FLOAT UNSIGNED);
  INSERT INTO t VALUES (47),(28.1237);
  
  mysql> SELECT (DATE_ADD(f2, INTERVAL 1 YEAR_MONTH)) AS f1 FROM (SELECT 1 FROM t) AS t1 JOIN (SELECT (DATE_ADD(BIN(c1), INTERVAL 1 YEAR)) AS f2 FROM t) AS t2; -- sql1
  +------+
  | f1   |
  +------+
  | NULL |
  | NULL |
  | NULL |
  | NULL |
  +------+
  4 rows in set, 11 warnings (0.00 sec)
  
  mysql> SELECT (DATE_ADD(f2, INTERVAL 1 YEAR_MONTH)) AS f1 FROM (SELECT DISTINCT 1 FROM t) AS t1 JOIN (SELECT (DATE_ADD(BIN(c1), INTERVAL 1 YEAR)) AS f2 FROM t) AS t2; -- sql2
  +------------+
  | f1         |
  +------------+
  | 2011-12-11 |
  | NULL       |
  +------------+
  2 rows in set, 5 warnings (0.00 sec)
  ```

* #7 https://github.com/pingcap/tidb/issues/40014

  **Status**: Verified

  **Version**: 5.3.1, 6.4.0

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 BIGINT UNSIGNED);
  INSERT INTO t VALUES (38.1089);
  
  mysql> SELECT f1 FROM (SELECT (-SEC_TO_TIME(-1)%~c1) AS f1 FROM t) AS t1 WHERE ((f1>LOG(5)) IS FALSE) OR 1; -- sql1
  +------+
  | f1   |
  +------+
  |    1 |
  +------+
  1 row in set (0.00 sec)
  
  mysql> SELECT DISTINCT f1 FROM (SELECT (-SEC_TO_TIME(-1)%~c1) AS f1 FROM t) AS t1 WHERE ((f1>LOG(5)) IS FALSE) OR 1; -- sql2
  +----------------------------------+
  | f1                               |
  +----------------------------------+
  | 1.000000000000000000000000000000 |
  +----------------------------------+
  1 row in set (0.00 sec)
  ```

* #8 https://github.com/pingcap/tidb/issues/40015

  **Status**: Verified

  **Version**: 5.0.2, 6.4.0

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 FLOAT UNSIGNED);
  INSERT INTO t VALUES (47),(28.1237);
  
  mysql> SELECT (DATE_ADD(BIN(c1), INTERVAL 1 DAY_HOUR)) AS f1 FROM (SELECT 1 FROM t) AS t1 JOIN (SELECT c1 FROM t) AS t2 ON 1; -- sql1
  +---------------------+
  | f1                  |
  +---------------------+
  | 2010-11-11 01:00:00 |
  | 2010-11-11 01:00:00 |
  | NULL                |
  | NULL                |
  +---------------------+
  4 rows in set, 5 warnings (0.01 sec)
  
  mysql> SELECT (DATE_ADD(BIN(c1), INTERVAL 1 DAY_HOUR)) AS f1 FROM (SELECT 1 FROM t) AS t1 JOIN (SELECT c1 FROM t) AS t2 ON (0 AND c1 != 1) IS FALSE; -- sql2
  +------+
  | f1   |
  +------+
  | NULL |
  | NULL |
  | NULL |
  | NULL |
  +------+
  4 rows in set, 9 warnings (0.00 sec)
  ```

* #9 https://github.com/pingcap/tidb/issues/40016

  **Status**: Verified

  **Version**: 4.0.11, 6.4.0

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 BIGINT,c2 VARCHAR(20));
  INSERT INTO t VALUES (-0,'0'),(45.0855,'\n3');
  
  mysql> SELECT 1 FROM (SELECT c1 FROM t) AS `t1` WHERE (LTRIM(c1) BETWEEN 'a' AND 1) AND (c1=ANY (SELECT c2 FROM t)); -- sql1
  +---+
  | 1 |
  +---+
  | 1 |
  +---+
  1 row in set, 1 warning (0.00 sec)
  
  mysql> SELECT 1 FROM (SELECT c1 FROM t) AS `t1` WHERE (LTRIM(c1) BETWEEN 'a' AND 1) AND (c1>=ANY (SELECT c2 FROM t)); -- sql2
  Empty set, 1 warning (0.00 sec)
  ```

* #10 https://github.com/pingcap/tidb/issues/40017

  **Status**: Verified

  **Version**: 4.0.9, 6.4.0

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 CHAR(20));
  INSERT INTO t VALUES ('well'),('3	');
  
  mysql> SELECT 1 FROM (SELECT 1 AS `f5` FROM t) AS `t1` WHERE ((DATE_SUB(TO_DAYS('2003-03-25'), INTERVAL 1 DAY)) NOT IN (SELECT c1 FROM t)) OR (NOT ((DAYOFWEEK('2010-12-07')>>1+~`f5`)>=ANY (SELECT c1 FROM t))); -- sql1
  +---+
  | 1 |
  +---+
  | 1 |
  | 1 |
  +---+
  2 rows in set, 4 warnings (0.00 sec)
  
  mysql> SELECT 1 FROM (SELECT 1 AS `f5` FROM t) AS `t1` WHERE ((DATE_SUB(TO_DAYS('2003-03-25'), INTERVAL 1 DAY)) NOT IN (SELECT c1 FROM t)) OR (NOT ((DAYOFWEEK('2010-12-07')>>1+~`f5`)=ANY (SELECT c1 FROM t))); -- sql2
  Empty set, 6 warnings (0.01 sec)
  ```

* #11 https://github.com/pingcap/tidb/issues/40018

  **Status**: Verified

  **Version**: 3.0.12, 6.4.0

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 CHAR(20));
  INSERT INTO t VALUES ('1'),('w');
  
  mysql> SELECT 1 FROM (SELECT 1 AS f1 FROM t) AS `t1` WHERE (!f1)=ANY (SELECT c1 FROM t); -- sql1
  +---+
  | 1 |
  +---+
  | 1 |
  | 1 |
  +---+
  2 rows in set, 2 warnings (0.01 sec)
  
  mysql> SELECT 1 FROM (SELECT 1 AS f1 FROM t) AS `t1` WHERE (!f1)>=ANY (SELECT c1 FROM t); -- sql2
  Empty set (0.00 sec)
  ```

## OceanBase

* #1 https://github.com/oceanbase/oceanbase/issues/1100

  **Status**: Fixed (https://github.com/oceanbase/oceanbase/commit/8851d2c3ba208f4eb9a28b7feff61541dd1509ce)

  **Version**: 3.1.2, 3.1.4

  **Test case**

  ```sql
  drop table if exists t;
  create table t ( c1 float );
  insert into t values (2),(-0.0001),(-9.183);
  
  mysql> (SELECT 1 FROM t WHERE COT(0.2)=0) UNION ALL (SELECT (BINARY c1 | 0) FROM t); -- original sql
  +------+
  | 1    |
  +------+
  |    2 |
  |    0 |
  |    0 |
  +------+
  3 rows in set (0.00 sec)
  
  mysql> (SELECT 1 FROM t WHERE 1 ) UNION ALL (SELECT (BINARY c1 | 0) FROM t); -- mutated sql1
  +----------------------+
  | 1                    |
  +----------------------+
  |                    1 |
  |                    1 |
  |                    1 |
  |                    2 |
  |                    0 |
  | 18446744073709551607 |
  +----------------------+
  6 rows in set, 2 warnings (0.01 sec)
  ```

* #2 https://github.com/oceanbase/oceanbase/issues/1248

  **Status**: Verified

  **Version**: 3.1.4, 4.0.0

  **Test case**

  ```sql
  drop table if exists t;
  CREATE TABLE t (c1 INT);
  INSERT INTO t VALUES (1);
  
  mysql> SELECT TO_BASE64(f1) AS `f1`, 1 FROM (SELECT (1&ASIN(4242208586805532840)) AS f1 FROM t) AS t1 JOIN (SELECT 1 FROM t) AS t2; -- sql1
  +------+---+
  | f1   | 1 |
  +------+---+
  | NULL | 1 |
  +------+---+
  1 row in set (0.00 sec)
  
  mysql> SELECT TO_BASE64(f1) AS `f1`, 1 FROM (SELECT DISTINCT (1&ASIN(4242208586805532840)) AS f1 FROM t) AS t1 JOIN (SELECT 1 FROM t) AS t2; -- sql2
  +------+---+
  | f1   | 1 |
  +------+---+
  |      | 1 |
  +------+---+
  1 row in set (0.00 sec)
  ```