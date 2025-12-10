# Indexes

本文说明核心表的 SQL 索引组织方式。

## 背景

Study、Series、Instance 表都有 PartitionKey、*Keys、*Uids：
- `*Uids`：客户生成的唯一 ID。
- `*Keys`：系统生成的主键，仅内部 JOIN 使用，体积更小，不应外发给客户。
- `PartitionKey`：按分区进行逻辑分组，弱约束；过滤查询务必包含该列，例外需谨慎评审。

## 最佳实践

- 所有索引第一列使用 PartitionKey；所有 SQL 过滤应包含 PartitionKey。当前不支持跨分区 QIDO，后续可在 Analytics 支持。
- ExtendedQueryTag / ChangeFeed / Re-Indexing 等跨分区场景，会在 watermark 上建跨分区索引。
|- Uids 与 Keys 均建索引：Uids 供客户侧 STOW/WADO，Keys 仅内部 JOIN。

### 在事务外创建索引

索引应在事务外、事务提交后创建。放在事务内会在创建期间锁表，表越大问题越严重。即便使用 ONLINE=ON，开始会拿共享锁，结束会拿排他锁，都会阻塞写入，因此必须放在事务外。确保 `GO` 在索引之后，而不是在事务开始之后。

```sql
SET XACT_ABORT ON
BEGIN TRANSACTION
GO
/*************************************************************
    Stored procedures and other things can be within trasnaction
**************************************************************/
      
COMMIT TRANSACTION


IF NOT EXISTS 
(
    SELECT *
    FROM    sys.indexes
    WHERE   NAME = 'IXC_FileProperty_InstanceKey_Watermark_ContentLength'
        AND Object_id = OBJECT_ID('dbo.FileProperty')
)
BEGIN
    CREATE NONCLUSTERED INDEX IXC_FileProperty_InstanceKey_Watermark_ContentLength ON dbo.FileProperty
    (
    InstanceKey,
    Watermark,
    ContentLength
    ) WITH (DATA_COMPRESSION = PAGE, ONLINE = ON)
END
GO
```
