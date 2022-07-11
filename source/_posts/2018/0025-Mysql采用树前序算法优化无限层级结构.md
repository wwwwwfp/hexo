---
title: Mysql 采用树前序算法优化无限层级结构
index_img: /img/cover/25.jpg
categories:
  - Mysql
tags:
  - 树前序算法
abbrlink: 4f91d70a
date: 2018-08-23 16:37:23
---
#### 前序遍历：

特点：前序遍历首先访问根结点然后遍历左子树，最后遍历右子树。在遍历左、右子树时，仍然先访问根结点，然后遍历左子树，最后遍历右子树。

这种方式的表结构，可以很方便的找到对应节点的所有子节点或父节点的信息。

+ 具体实现步骤如下,前序遍历二叉树图：
![](1.png)
+ 数据结构（主要字段）
![](2.png)
+ 主要思路
  + 根据记录左右节点获取子孙节点或父子节点。
  + 如：南方集团深圳分部的子孙节点为的条件为：l_node > 2  and  r_node < 15
  + 如：中医骨科所有的父节点条件为： l_node < 4  and  r_node > 9
+ 难点：
    ```java
    新增、修改、删除时业务逻辑相对复杂，需要对影响到列所有数据的左右节点做修改。
    如果业务中已经存在数据，只需要新增两个节点字段 l_node ,r_node 然后根据parent_id 初始化相应的值
    ```
+ 初始化已存在数据的存储过程：
    ```java
    1、该存储过程适用于原始数据按照parent_id 升序排列时 当前记录的父级记录一定要在前面
       可以在记录中记录该情况的数据，当其它记录数据处理完成后再处理一次这些记录的数据。
    2、查询集合时要过滤掉删除的记录和不存在父级的记录（可以到该存储过程中的查询语句中加相应的条件）
    ```
    ```sql
    DELIMITER $$
    DROP PROCEDURE IF EXISTS `reset_node`$$
    CREATE PROCEDURE `reset_node`()
    BEGIN
    DECLARE _id BIGINT DEFAULT 0;
    DECLARE _pid BIGINT DEFAULT 0;
    DECLARE _lnode BIGINT DEFAULT 0;
    DECLARE _rnode BIGINT DEFAULT 0;
    DECLARE _num BIGINT DEFAULT 1;
    DECLARE pidtemp BIGINT DEFAULT 0;
    DECLARE _done BIGINT DEFAULT 0;
    DECLARE _count BIGINT DEFAULT 0;
    DECLARE _isnull INT DEFAULT 0;
    DECLARE _cur CURSOR FOR SELECT id,parent_id FROM tree_sort WHERE parent_id >0  ORDER BY parent_id ASC;
    DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET _done = 1;#错误定义，标记循环结束  
    OPEN _cur;
    REPEAT
    FETCH _cur INTO _id,_pid;
    SET _isnull = 0;
    IF NOT _done THEN
    IF _pid != pidtemp THEN   -- 当前记录的pid = 初始化的pid   
    SELECT l_node,r_node INTO _lnode,_rnode FROM tree_sort WHERE id = _pid;
    IF _lnode IS NOT NULL THEN
    SET _isnull = 0;
    SET pidtemp = _pid;
    SET _num = 0; -- 记录当前父节点有没有子节点  0没有 1 有
                        ELSE 
                            SET _isnull = 1;
                        END IF;
                    END IF;
                    IF _isnull = 0 THEN
                        IF _num = 0 THEN 
                            UPDATE tree_sort SET l_node = l_node +2 WHERE l_node > _lnode;		
                            UPDATE tree_sort SET r_node = r_node +2 WHERE r_node >= _rnode ;
                            UPDATE tree_sort SET l_node = _rnode,r_node = _rnode+1 WHERE id = _id;
                            SET _rnode = _rnode+1;
                        ELSE 
                            UPDATE tree_sort SET l_node = l_node +2 WHERE l_node > _rnode;			
                            UPDATE tree_sort SET r_node = r_node +2 WHERE r_node > _rnode;	
                            UPDATE tree_sort SET l_node = _rnode+1,r_node = _rnode+2 WHERE id = _id;
                            SET _rnode = _rnode +2;
                        END IF;
                    END IF;
                END IF;
            UNTIL _done END REPEAT; #当_done=1时退出被循  
        CLOSE _cur;  
    END$$
    DELIMITER ;
    调用过程
    CALL reset_node();
    ```
+ 新增元素存储过程：删除一个节点或多个节点，只需要关注比大大的左右节点即可，然后对响应的节点做减操作即可。
    ```sql
    DROP PROCEDURE IF EXISTS drop_node;
    DELIMITER $$
    CREATE PROCEDURE drop_node(IN leftnode BIGINT,IN rightnode BIGINT,OUT result INT)
    BEGIN
        DECLARE sql_error INT DEFAULT 0;
        DECLARE CONTINUE HANDLER FOR SQLEXCEPTION  SET sql_error = 1;  
        START TRANSACTION;
        DELETE FROM tree_sort WHERE l_node >= leftnode AND r_node <= rightnode;
        UPDATE tree_sort SET l_node = l_node-(rightnode-leftnode+1) WHERE l_node > leftnode;
        UPDATE tree_sort SET r_node = r_node-(rightnode-leftnode+1) WHERE r_node > rightnode;
        IF (sql_error = 1) THEN 
            ROLLBACK;
        ELSE
            COMMIT;
        END IF;
        SET result = sql_error;
    END
    $$
    DELIMITER ;
     
    调用过程
        SET @result = 0;    
        -- 参数分别为  删除元素的左节点值和右节点值   返回值  0 正常  1 异常
        CALL drop_node(2,7,@result);
    ```
+ 编辑元素存储过程：
  编辑元素相较于前面两者会相对的复杂一些。
  主要思路（个人）：如果有感觉比较好的想法的欢迎留言，谢谢！
  1. 对需要移动的节点外的所有节点重新排列节点顺序，类似删除操作。 
  2. 更新需要移动的节点的编号    更新为  如下方式（编号为1开始）
     ![](3.png)
  3. 然后找到需要移动到的父节点位置，到该位置做类似于添加的操作即可
    ```sql
    DROP PROCEDURE IF EXISTS update_node;
    DELIMITER $$
    CREATE  PROCEDURE update_node(IN move_id BIGINT,IN target_id BIGINT,OUT result INT)
    BEGIN
    DECLARE sql_error INT DEFAULT 0;
    DECLARE leftnode BIGINT DEFAULT 0;
    DECLARE rightnode BIGINT DEFAULT 0;
    DECLARE temp BIGINT DEFAULT 0;
    DECLARE cnt INT DEFAULT 0;
    DECLARE move_ids VARCHAR(1000) DEFAULT '';
    DECLARE update_num INT DEFAULT 0;
    DECLARE CONTINUE HANDLER FOR SQLEXCEPTION  SET sql_error = 1;  
    SELECT l_node,r_node INTO leftnode,rightnode FROM tree_sort WHERE id = move_id;  
    SET update_num = rightnode-leftnode +1;   -- 要修改的 节点数   修改元素个数为修改节点数/2  
    SET temp = leftnode - 1;
    SELECT GROUP_CONCAT(id) INTO move_ids FROM tree_sort WHERE l_node >= leftnode AND r_node <= rightnode;
    START TRANSACTION;
    UPDATE tree_sort SET l_node = 0-(l_node - temp),r_node = 0-(r_node - temp) WHERE l_node >= leftnode AND r_node <= rightnode;    -- 将修改的元素的节点数重新排序  并置为负数，防止影响到其它节点
    UPDATE tree_sort SET l_node = l_node-(rightnode-leftnode+1) WHERE l_node > leftnode;
    UPDATE tree_sort SET r_node = r_node-(rightnode-leftnode+1) WHERE r_node > rightnode;
    SELECT l_node,r_node INTO leftnode,rightnode FROM tree_sort WHERE id = target_id;     
    SELECT COUNT(1) INTO cnt FROM tree_sort WHERE l_node > leftnode AND r_node < rightnode;
    IF(cnt=0) THEN
    UPDATE tree_sort SET l_node = l_node+update_num WHERE l_node > leftnode;
    UPDATE tree_sort SET r_node = r_node+update_num WHERE r_node >= rightnode;   -- 修改删除(伪) 后节点影响到的节点序号
    UPDATE tree_sort SET l_node = 0-l_node +leftnode , r_node = 0- r_node + leftnode WHERE FIND_IN_SET (id,move_ids);   -- 将原来置为负数的值回复，并做加 leftnode操作
    ELSE
    SELECT r_node INTO rightnode FROM tree_sort WHERE l_node>leftnode AND r_node<rightnode ORDER BY r_node DESC LIMIT 1;
    UPDATE tree_sort SET l_node = l_node+update_num WHERE l_node > rightnode;
    UPDATE tree_sort SET r_node = r_node+update_num WHERE r_node > rightnode;
    UPDATE tree_sort SET l_node = 0-l_node+rightnode,r_node = 0- r_node + rightnode WHERE FIND_IN_SET (id,move_ids);
    END IF;
    IF (sql_error = 1) THEN
    ROLLBACK;
    ELSE
    COMMIT;
    END IF;
    SET result = sql_error;
    END
    $$
    DELIMITER ;
    
    调用过程
    SET @result = 0;
    -- 参数分别为  要移动的节点id，移动到的节点id   返回值  0 正常  1 异常
    CALL update_node(13,3,@result);
    SELECT * FROM tree_sort; 
    ```
+ 编辑效果图：
![](4.png)![](5.png)