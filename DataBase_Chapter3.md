# 数据库事务处理

## 事物概念
`BEGIN TRANSACTION;`  
`COMMIT;`  
`ROLLBACK`

事务是程序的逻辑运行单元

- 计算环境的脆弱性—故障恢复问题
- 计算环境的分布性—并发控制问题

ACID  

- Atomicity原子性 
  - 事务的所有操作要么完全执行, 要么全不做 
  - 由DBMS恢复机制实现
- <MARK>Consistency一致性
  - 事务执行必须保证数据库的一致性 
  - 数据库事务的目标
- <mark>Isolation隔离性
  - 每个事务感觉到都是在没有其它事务的情况下执行的
  - 事务不受其它并发执行事务的影响。对任何一对事务T1，T2，在 T1看来，T2要么在T1开始之前已经结束，要么在T1完成之后再 开始执行
  - 隔离性通过DBMS并发控制机制实现
- Durability持久性
  - 一旦事务提交， 事务对数据库的影响就不会丢失, 即使再发生系 统故障
  - 系统发生故障不能改变事务的结果 
  - 通过恢复机制实现

## 事物状态

Active/Partially committed/Failed/Aborted/Committed

## 原子性与持久性的实现
数据库系统的恢复管理(recovery-management ) 部件负责实现对原子性与持久性的支持  
shadow-database方案:

- 所有更新都是对数据库的一份影子拷贝(shadow copy) 进行的, 仅当事务到达部分提交状态并且所 有更新页都已写回磁盘，db_pointer 才指向更新 后的shadow copy.
  
  
## 并发控制概述
并发操作带来的数据不一致性包括丢失修改、不可重复读和脏读。  

并发控制的主要技术：
- locking封锁（常用）
- timestamp时间戳
- 乐观控制发

调度（Schedule）：使并发事物的代码按时间顺序执行的序列

### 封锁（locking）
- X锁（exclusive locks)
- S锁（share locks）

- 一级封锁协议 X锁 可以防止‘丢失修改’，不能保证可重复读和脏读。
- 二级封锁协议 可以防止脏读
- 三级封锁协议

活锁：先来先服务  
死锁：预防/诊断与解除

### 并发调度的可串行性

#### 可串行化调度serializable
多个事务的并发执行是正确的，当且仅当其结果与某一次序串行地执行这些事务时的结果相同。  

冲突操作指不同的事务对同一个数据的读写操作和写写操作。

#### 冲突可串行化调度
可串行化调度的充分条件  
一个调度S在保证冲突操作的次序不变的情况下，通过交换两个事务不 冲突操作的次序得到另一个调度S’，如果S’是串行的，称调度S为冲 突可串行化的调度【二者冲突等价】

不能交换(Swap)的动作:
- 同一事务的两个操作
- 不同事务的冲突操作

### 两段锁协议
两段封锁协议(Two-Phase Locking，简称2PL)是最常用的一种封锁协议，理论上已证明使用两段封锁协议产生的是可串行化调度  
数据库管理系统普遍采用两段锁协议的方法实现并发调度 的可串行性，从而保证调度的正确性

两段锁协议:所有事务分两个阶段对数据项加锁和解锁

-  在对任何数据进行读、写操作之前，事务首先要获得对该数据的封锁
-  在释放一个封锁之后，事务不再申请和获得任何其他封锁

- 第一阶段是获得封锁，也称为扩展阶段
- 第二阶段是释放封锁，也称为收缩阶段
    - 事务可以释放任何数据项上的任何类型的锁，但是不能再申请任何锁。  

  
事务遵守两段锁协议是可串行化调度的充分条件，而不是必要条件

### 封锁粒度
封锁对象的大小称为封锁粒度(Granularity)  

封锁的对象: 逻辑单元，物理单元    


封锁粒度与系统的并发度和并发控制的开销密切相关
多粒度封锁(Multiple Granularity Locking)  
在一个系统中同时支持多种封锁粒度供不同的事务

- 需要处理多个关系的大量元组的用户事务:以数据库为封锁单位
- 只处理少量元组的用户事务:以元组为封锁单位

多粒度树  
显式封锁和隐式封锁