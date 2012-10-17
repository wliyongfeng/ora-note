#讲解Oracle使用UNDO表空间
   
---
 学习Oracle时，你可能会遇到Oracle使用UNDO表空间问题，这里将介绍Oracle使用UNDO表空间问题的解决方法，在这里拿出来和大家分享一下。UNDO表空间用于存放UNDO数据，当执行DML操作(INSERT，UPDATE和DELETE)时，Oracle会将这些操作的旧数据写入到UNDO段，在Oracle9i之前，管理UNDO数据时使用(Rollback Segment)完成的。从Oracle9i开始，管理UNDO数据不仅可以使用回滚段，还可以Oracle使用UNDO表空间。因为规划和管理回滚段比较复杂，所有Oracle database 10g已经完全丢弃用回滚段。并且Oracle使用UNDO表空间来管理UNDO数据。

  UNDO数据也称为回滚(ROLLBACK)数据，它用于确保数据的一致性。当执行DML操作时，事务操作前的数据被称为UNDO记录。UNDO段用于保存事务所修改数据的旧值，其中存储着被修改数据块的位置以及修改前数据。

##UNDO数据的作用

###1 回退事务

  当执行DML操作修改数据时，UNDO数据被存放到UNDO段，而新数据则被存放到数据段中，如果事务操作存在问题，旧需要回退事务，以取消事务变化。假定用户A执行了语句UPDATE emp SET sal=1000 WHERE empno=7788后发现，应该修改雇员7963的工资，而不是雇员7788的工资，那么通过执行ROLLBACK语句可以取消事务变化。当执行ROLLBACK命令时，Oracle会将UNDO段的UNDO数据800写回的数据段中

###2 读一致性

  用户检索数据库数据时，Oracle总是使用用户只能看到被提交过的数据(读取提交)或特定时间点的数据(SELECT语句时间点)。这样可以确保数据的一致性。例如，当用户A执行语句UPDATE emp SET sal=1000 WHERE empno=7788时，UNDO记录会被存放到回滚段中，而新数据则会存放到EMP段中;假定此时该数据尚未提交，并且用户B执行SELECT sal FROM emp WHERE empno=7788，此时用户B将取得UNDO数据800，而该数据正是在UNDO记录中取得的。

###3 事务恢复

  事务恢复是例程恢复的一部分，它是由Oracle server自动完成的。如果在数据库运行过程中出现例程失败(如断电，内存故障，后台进程故障等)，那么当重启Oracle server时，后台进程SMON会自动执行例程恢复，执行例程恢复时，oracl会重新做所有未应用的记录。回退未提交事务。

###4 倒叙查询(FlashBack Query)

  倒叙查询用于取得特定时间点的数据库数据，它是9i新增加的特性，假定当前时间为上午11:00，某用户在上午10:00执行UPDATE emp SET sal=3500 WHERE empno=7788语句，修改并提交了事务(雇员原工资为3000)，为了取得10:00之前的雇员工资，用户可以使用倒叙查询特征。


##使用UNDO参数


###1 UNDO_MANAGEMENT

  该初始化参数用于指定UNDO数据的管理方式。如果要使用自动管理模式，必须设置该参数为AUTO，如果使用手工管理模式，必须设置该参数为MANUAL，使用自动管理模式时，Oracle使用UNDO表空间管理undo管理，使用手工管理模式时，Oracle会使用回滚段管理undo数据，需要注意，使用自动管理模式时，如果没有配置初始化参数UNDO_TABLESPACE。Oracle会自动选择第一个可用的UNDO表空间存放UNDO数据，如果没有可用的UNDO表空间，Oracle会使用SYSTEM回滚段存放UNDO记录，并在ALTER文件中记载警告。

###2 UNDO_TABLESPACE
  该初始化参数用于指定例程所要使用的UNDO表空间，使用自动UNDO管理模式时，通过配置该参数可以指定例程所要使用UNDO表空间。在RAC(Real Application Cluster)结构中，因为一个UNDO表空间不能由多个例程同时使用，所有必须为每个例程配置一个独立的UNDO表空间。

###3 UNDO_RETENTION
  该初始化参数用于控制UNDO数据的最大保留时间，其默认值为900秒，从9i开始，通过配置该初始化参数，可以指定undo数据的保留时间，从而确定倒叙查询特征(Flashback Query)可以查看到的最早时间点。
