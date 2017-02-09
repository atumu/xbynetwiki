title: sql函数运用 

#  SQL函数运用 
参考http://www.yiibai.com/sql/sql_function_substring_index.html
##  substring_index()函数 
出现的分隔符的指定数量的前返回一个字符串的子串.中间第二个参数指定分隔符
SELECT SUBSTRING_INDEX('2015-07-28 09:07:27', ' ', 1)输出2015-07-28
SELECT SUBSTRING_INDEX('2015-07-28 09:07:27', ' ', -1)输出09:07:27
SELECT SUBSTRING_INDEX('/asa/ss/s', '/', 1)输出空
SELECT SUBSTRING_INDEX('/asa/ss/s', '/', 2)输出/asa
SELECT SUBSTRING_INDEX('/asa/ss/s', '/', -1)输出s

##  SUBSTRING()函数 
` 注意：sql中字符串字符计数从1开始 `
SUBSTRING(str,pos) 从字符串str的pos处开始位置返回一个字符串。
SUBSTRING(str,pos,len) 用len参数的格式从字符串str从位置pos返回子len个字符。
另外，也可以使用pos负值。在这种情况下，子字符串的开始是从字符串的末尾，而不是从pos字符开始。负值可用于在任何该函数的pos形式。
SELECT SUBSTRING('Quadratically',5);输出ratically
SELECT SUBSTRING('Quadratically',5,6);输出ratica

##  CONCAT()函数 
SQL CONCAT函数用于连接两个字符串，形成一个字符串
SELECT CONCAT('FIRST', 'SECOND');输出FIRSTSECOND
SELECT CONCAT('FIRST/', 'SECOND');输出FIRST/SECOND
SELECT CONCAT(id, name, work_date)

##  REPLACE()函数 
SELECT REPLACE('www.mysql.com', 'w', 'Ww');输出WwWwWw.mysql.com
##  复杂查询运用一 
```

SELECT CONCAT(
	'http://localhost:8080/resource//',
	REPLACE(SUBSTRING('D:\\sasaas\\reousece\\aasas\\aaasas\\asa\\aaa.jpg',(select INSTR(
		'D:\\sasaas\\reousece\\aasas\\aaasas\\asa\\aaa.jpg',
		'reousece\\'
	)+9)),'\\','/'));

```
java中：
```

"SELECT CONCAT("+
	"'http://localhost:8080/resource//',"+
	"REPLACE(SUBSTRING('D:\\\\sasaas\\\\reousece\\\\aasas\\\\aaasas\\\\asa\\\\aaa.jpg',(select INSTR(
		'D:\\sasaas\\reousece\\aasas\\aaasas\\asa\\aaa.jpg', 
		'reousece\\\\'
	)+9)),'\\\\','/'));"

```
返回:http://localhost:8080/resource//aasas/aaasas/asa/aaa.jpg