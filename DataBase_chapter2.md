# 索引
一个索引中包含指定列的所有值及其相应的存储位置

# 数据库安全性

## 安全基本要素
基本概念：

安全，低于可能的风险和威胁的能力，一般只关心由于人为因素所产生的风险和威胁。

威胁(threats):对安全的潜在破坏。这种破坏不一定等实际发生才构成威胁

攻击(attack):可能导致破坏的行为，发起人被称为攻击者

基本要素：Confidentiality/Integrity/Availability/Non-repudiation(不可抵赖性)/可审计性

##数据库安全控制机制
- (基于口令的)用户鉴别(散列函数/OTP/智能卡/生物特征）
- 存取控制（包括自主存取控制 Discretionary Access Control DAC,强制存取控制 Mandatory Access Control MAC)
   - 用户权限定义
   - 权限检查机制

### 自助存取控制方法

- 授权GRANT 

GRANT <权限>[,<权限>]...  
ON <对象类型> <对象名>   
TO <用户>[,<用户>]...   
[WITH GRANT OPTION];


所有权限（ALL PREVILIGES）所有用户（PUBLIC）

- 权限回收revoke

REVOKE <权限>[,<权限>]...  
ON <对象类型> <对象名>   
FROM <用户>[,<用户>]...;

CASCADE级联收回

### 数据库角色
被命名的一组与数据库操作相关的权限

- 角色是权限的集合
- 可以为一组具有相同权限的用户创建一个角色

通过角色来实现将一组权限授予一个用户。 步骤如下:

1. 首先创建一个角色 R1 
```CREATE ROLE R1;```
2. 然后使用GRANT语句，使角色R1拥有Student表的SELECT、 UPDATE、INSERT权限
```GRANT SELECT，UPDATE，INSERT ON TABLE Student
TO R1;```
3. 将这个角色授予王平，张明，赵玲。使他们具有角色R1所包含的 全部权限
```GRANT R1
TO 王平，张明，赵玲;```
4. 可以一次性通过R1来回收王平的这3个权限
```REVOKE R1 FROM 王平;```

### 强制存取控制方法


- 主体是系统中的活动实体
  - DBMS所管理的实际用户 
  - 代表用户的各进程
- 客体是系统中受主体操纵的被动实体:文件、基表、索引、视图 
- 敏感度标记(Label)
  -  绝密(Top Secret) 
  -  机密(Secret)
  -  可信(Confidential) 
  -  公开(Public)
- 主体的敏感度标记称为许可证级别(Clearance Level) 
- 客体的敏感度标记称为密级(Classification Level)
- 强制存取控制规则
  - 仅当主体的许可证级别大于或等于客体的密级时，该主体才能读取相应
的客体
  - 仅当主体的许可证级别等于客体的密级时，该主体才能写相应的客体 (禁止了拥有高许可证级别的主体更新低密级的数据对象)

<mark>SQL语法分析&语义检查->DAC检查->MAC检查->继续语义检查</mark>


## 视图机制
把要保密的数据对无权存取这些数据的用户 隐藏起来，对数据提供一定程度的安全保护

## 审计

## 数据加密

                            
