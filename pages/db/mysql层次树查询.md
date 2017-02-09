modifyAt:2016-12-21 17:15:17
title:MySQL层次树查询
location:db/mysql层次树查询
author:xbynet
createAt:2016-12-21 17:13:33

原文:https://yq.aliyun.com/articles/48885
最近遇到了一个问题，**在mysql中如何完成节点下的所有节点或节点上的所有父节点的查询？**
`在Oracle中我们知道有一个Hierarchical Queries可以通过CONNECT BY来查询，但是，在MySQL中还没有对应的函数!!!`
下面给出一个function来完成的方法
下面是sql脚本，想要运行的直接赋值粘贴进数据库即可。
创建表treenodes（可以根据需要进行更改）
```
-- ---------------------------- 
-- Table structure for `treenodes` 
-- ---------------------------- 
DROP TABLE IF EXISTS `treenodes`; 
CREATE TABLE `treenodes` ( 
  `id` int(11) NOT NULL, 
  `nodename` varchar(20) DEFAULT NULL, 
  `pid` int(11) DEFAULT NULL, 
  PRIMARY KEY (`id`) 
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```
插入几条数据
```
-- ---------------------------- 
-- Records of treenodes 
-- ---------------------------- 
INSERT INTO `treenodes` VALUES ('1', 'A', '0'); 
INSERT INTO `treenodes` VALUES ('2', 'B', '1'); 
INSERT INTO `treenodes` VALUES ('3', 'C', '1'); 
INSERT INTO `treenodes` VALUES ('4', 'D', '2'); 
INSERT INTO `treenodes` VALUES ('5', 'E', '2'); 
INSERT INTO `treenodes` VALUES ('6', 'F', '3'); 
INSERT INTO `treenodes` VALUES ('7', 'G', '6'); 
INSERT INTO `treenodes` VALUES ('8', 'H', '0'); 
INSERT INTO `treenodes` VALUES ('9', 'I', '8'); 
INSERT INTO `treenodes` VALUES ('10', 'J', '8'); 
INSERT INTO `treenodes` VALUES ('11', 'K', '8'); 
INSERT INTO `treenodes` VALUES ('12', 'L', '9'); 
INSERT INTO `treenodes` VALUES ('13', 'M', '9'); 
INSERT INTO `treenodes` VALUES ('14', 'N', '12'); 
INSERT INTO `treenodes` VALUES ('15', 'O', '12'); 
INSERT INTO `treenodes` VALUES ('16', 'P', '15'); 
INSERT INTO `treenodes` VALUES ('17', 'Q', '15');
```
把下面的语句直接粘贴进命令行执行即可（注意修改传入的参数，默认rootId，表明默认treenodes）
根据传入id查询所有父节点的id
```
delimiter // 
CREATE FUNCTION `getParLst`(rootId INT)
RETURNS varchar(1000) 

BEGIN
	DECLARE sTemp VARCHAR(1000);
	DECLARE sTempPar VARCHAR(1000); 
	SET sTemp = ''; 
	SET sTempPar =rootId; 
	
	#循环递归
	WHILE sTempPar is not null DO 
		#判断是否是第一个，不加的话第一个会为空
		IF sTemp != '' THEN
			SET sTemp = concat(sTemp,',',sTempPar);
		ELSE
			SET sTemp = sTempPar;
		END IF;

		SET sTemp = concat(sTemp,',',sTempPar); 
		SELECT group_concat(pid) INTO sTempPar FROM treenodes where pid<>id and FIND_IN_SET(id,sTempPar)>0; 
	END WHILE; 
	
RETURN sTemp; 
END
//
```
执行命令
```
select * from treenodes where FIND_IN_SET(id,getParList(15));
```

结果：

|id	|nodename	|pid|
| -------- | -------- | -------- |
|8	|H	|0|
|9	|I	|8|
|12	|L	|9|
|15	|O	|12|

根据传入id查询所有子节点的id

```
delimiter // 
CREATE FUNCTION `getParLst`(rootId INT)
RETURNS varchar(1000) 

BEGIN
	DECLARE sTemp VARCHAR(1000);
    DECLARE sTempChd VARCHAR(1000);

    SET sTemp = '$';
    SET sTempChd =cast(rootId as CHAR);

    WHILE sTempChd is not null DO
    	SET sTemp = concat(sTemp,',',sTempChd);
        SELECT group_concat(id) INTO sTempChd FROM  treeNodes where FIND_IN_SET(pid,sTempChd)>0;
   	END WHILE;
    RETURN sTemp; 
END
//
```
执行命令

```
select * from treenodes where FIND_IN_SET(id,getChildList(7));
```

结果：

|id	|nodename	|pid|
| -------- | -------- | -------- |
|7	|G	|6|