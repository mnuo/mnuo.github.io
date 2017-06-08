---
title: mysql视图
date: 2016-06-20 15:20:08 
tags: MySql
category: MySQL
---
> 视图是一种虚拟存在的表,对于使用视图的用户来说基本上是透明的,视图并不在数据库中实际存在,行和列的数据来自定义视图的查询中使用的表,并且是在使用视图时动态生成的
		
    优势:
        简单:使用视图的用户完全不需要关心后面对应的表的结构,关联条件和帅选条件,对用户来说已经是过滤好的复合条件的结果集
		安全:使用视图的用户只能访问他们被允许查询的结果集,对标的权限管理并不能限制到某个行某个列,但是通过视图就可以简单的实现
		数据独立: 一旦视图的结构确定了,可以屏蔽表结构变化对用户的影响,源表增加列对视图没有影响;源表修改列名,则可以通过修改试图来解决,不会造成对访问者的影响
		
> 创建视图:   

    CREATE [OR REPLACE][]ALGORITHM={UNDEFINED|MERGE|TEMPTABLE}
			VIEW view_name [(column_list)]
			AS select_statement
			[WITH [CASCADED|LOCAL] CHECK OPTION]
		
	ALTER [ALGORITHM={UNDEFINED|MERGE|TEMPTABLE}]
			VIEW view_name [(column_list)]
			AS select_statement
			[WITH [CASCADED|LOCAL] CHECK OPTION]
		
	DROP VIEW view_name;
		
		mysql> show table status like 'view_deviewshop'
			-> ;
		+-----------------+--------+---------+------------+------+----------------+-----                                                                                --------+-----------------+--------------+-----------+----------------+---------                                                                                ----+-------------+------------+-----------+----------+----------------+--------                                                                                -+
		| Name            | Engine | Version | Row_format | Rows | Avg_row_length | Data                                                                                _length | Max_data_length | Index_length | Data_free | Auto_increment | Create_t                                                                                ime | Update_time | Check_time | Collation | Checksum | Create_options | Comment                                                                                 |
		+-----------------+--------+---------+------------+------+----------------+-----                                                                                --------+-----------------+--------------+-----------+----------------+---------                                                                                ----+-------------+------------+-----------+----------+----------------+--------                                                                                -+
		| view_deviewshop | NULL   |    NULL | NULL       | NULL |           NULL |                                                                                        NULL |            NULL |         NULL |      NULL |           NULL | NULL                                                                                        | NULL        | NULL       | NULL      |     NULL | NULL           | VIEW                                                                                    |
		+-----------------+--------+---------+------------+------+----------------+-----                                                                                --------+-----------------+--------------+-----------+----------------+---------                                                                                ----+-------------+------------+-----------+----------+----------------+--------                                                                                -+
		1 row in set (0.00 sec)

		mysql> show table status like 'view_deviewshop' \G;
		*************************** 1. row ***************************
				   Name: view_deviewshop
				 Engine: NULL
				Version: NULL
			 Row_format: NULL
				   Rows: NULL
		 Avg_row_length: NULL
			Data_length: NULL
		Max_data_length: NULL
		   Index_length: NULL
			  Data_free: NULL
		 Auto_increment: NULL
			Create_time: NULL
			Update_time: NULL
			 Check_time: NULL
			  Collation: NULL
			   Checksum: NULL
		 Create_options: NULL
				Comment: VIEW
		1 row in set (0.00 sec)

		ERROR:
		No query specified

		mysql> show create view view_deviewshop \G;
		*************************** 1. row ***************************
			   View: view_deviewshop
		Create View: CREATE ALGORITHM=UNDEFINED DEFINER=`root`@`%` SQL SECURITY DEFINER VIEW `view_deviewshop` AS select `t`.`id` AS `id`,`t`.`name` AS `dbname`,`t`.`mark` AS `mark`,`t`.`add_time` AS `add_time`,`t`.`flag` AS `flag`,`t`.`delete_time` AS `delete_time`,`t`.`remark` AS `remark`,`t`.`group_id` AS `group_id`,`tp`.`task_id` AS `task_id` from (((`is_dbviewshop` `t` left join `is_preset_parentViewShops` `pp` on((`t`.`id` = `pp`.`child_viewshopId`))) left join `is_preset` `p` on((`pp`.`presetId` = `p`.`id`))) left join `is_taskrecord_preset` `tp` on((`p`.`id` = `tp`.`preset_id`))) where isnull(`p`.`presetNameId`) union all select `t`.`id` AS `id`,`t`.`name` AS `name`,`t`.`mark` AS `mark`,`t`.`add_time` AS `add_time`,`t`.`flag` AS `flag`,`t`.`delete_time` AS `delete_time`,`t`.`remark` AS `remark`,`t`.`group_id` AS `group_id`,`tp`.`task_id` AS `task_id` from (((`is_dbviewshop` `t` left join `is_presetName_parentViewShops` `pv` on((`t`.`id` = `pv`.`child_viewshopId`))) left join `is_preset` `p` on((`p`.`presetNameId` = `pv`.`presetNameId`))) left join `is_taskrecord_preset` `tp` on((`p`.`id` = `tp`.`preset_id`)))
		1 row in set (0.00 sec)

		ERROR:
		No query specified

		mysql> select * from information_schema.views where table_name='view_deviewshop'\G;
		*************************** 1. row ***************************
		  TABLE_CATALOG: NULL
		   TABLE_SCHEMA: shopwebdb
			 TABLE_NAME: view_deviewshop
		VIEW_DEFINITION: /* ALGORITHM=UNDEFINED */ select `t`.`id` AS `id`,`t`.`name` AS `dbname`,`t`.`mark` AS `mark`,`t`.`add_time` AS `add_time`,`t`.`flag` AS `flag`,`t`.`delete_time` AS `delete_time`,`t`.`remark` AS `remark`,`t`.`group_id` AS `group_id`,`tp`.`task_id` AS `task_id` from (((`shopwebdb`.`is_dbviewshop` `t` left join `shopwebdb`.`is_preset_parentViewShops` `pp` on((`t`.`id` = `pp`.`child_viewshopId`))) left join `shopwebdb`.`is_preset` `p` on((`pp`.`presetId` = `p`.`id`))) left join `shopwebdb`.`is_taskrecord_preset` `tp` on((`p`.`id` = `tp`.`preset_id`))) where isnull(`p`.`presetNameId`) union all select `t`.`id` AS `id`,`t`.`name` AS `name`,`t`.`mark` AS `mark`,`t`.`add_time` AS `add_time`,`t`.`flag` AS `flag`,`t`.`delete_time` AS `delete_time`,`t`.`remark` AS `remark`,`t`.`group_id` AS `group_id`,`tp`.`task_id` AS `task_id` from (((`shopwebdb`.`is_dbviewshop` `t` left join `shopwebdb`.`is_presetName_parentViewShops` `pv` on((`t`.`id` = `pv`.`child_viewshopId`))) left join `shopwebdb`.`is_preset` `p` on((`p`.`presetNameId` = `pv`.`presetNameId`))) left join `shopwebdb`.`is_taskrecord_preset` `tp` on((`p`.`id` = `tp`.`preset_id`)))
		   CHECK_OPTION: NONE
		   IS_UPDATABLE: NO
				DEFINER: root@%
		  SECURITY_TYPE: DEFINER
		1 row in set (0.00 sec)

		ERROR:
		No query specified
