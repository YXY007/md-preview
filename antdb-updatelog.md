### AntDB 更新日志：
------

##### 2018-04-15：

* AntDB 3.1
    -  功能优化
    	-  需要启动adb_reduce时，进行路径和版本检查，避免因环境变量导致找不到adb_reduce产生core；新增 print_reduce_debug_log 参数，控制是否打印 adb_reduce 相关日志 
    - BUG修复 
    	- 修正SubPlan使用hashstore后因没有end接口无法结束hashstore导致事务结束时报临时文件未删除的告警,删除SubPlanState中没有使用的nullsstore变量
    	- 修正因 MakeExecNodesByOids 给 en_relid 赋值导致后续的 GetRemoteNodeList 获取节点列表异常的问题
    	- 修正save_oid_class函数没有判断表是否为临时表,导致load_oid_class函数还原时找不到表信息的问题
    	
##### 2018-03-09：
* AntDB 3.1
	- 新增功能
		- 添加 Cluster Barrier 功能 	
    -  功能优化
    	- 限制在源码目录中执行configure命令，源码编译只能在非源码目录编译
    	- 优化adbmgr提示信息：在agent停止情况下failover 命令报错信息不准确问题
    - BUG修复 
    	- 修正range类型数据收发出现的问题，根据range内部上下界元素类型判断收发数据是否转换处理
    	- 修正 enable_adb_remote_test 的一处缺陷: 复制表与分片表关联时，无法下沉的bug
    	- 修正"psql -V"命令行显示的版本信息不正确的问题
    	- 修复issue65:创建type后使用查询报错"cache lookup failed for type 34523ß
    	- 修复issue61:集群计划下，limit失效的问题。