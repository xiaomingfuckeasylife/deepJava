## 记录自己所遇到的一些工作中的问题以及寻找到的答案

### 1 如何备份oracle数据库

* 导出：exp userid=yourusername/youruserpassword@Connect_Identifier File=OSPath
* 导入：imp userid/password@Connect_identifier fromuser=user_name_you_have_data_unloaded_from touser=new_user_name file=Path_to_*.dmp file

[传送门](http://stackoverflow.com/questions/12419340/how-to-backup-and-restore-oracle-database-11g-like-sql2005-database)

### 2 
