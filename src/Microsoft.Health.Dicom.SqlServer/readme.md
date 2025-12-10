# SQL 开发指南

## 学习资源

- [SQL server Internals	Microsoft SQL Server 2012 Internals](https://learning.oreilly.com/library/view/microsoft-sql-server/9780735670174/)
- [Managing SQL Server Performance](https://app.pluralsight.com/library/courses/managing-sql-server-database-performance/table-of-contents)

## 风格指南

https://github.com/ktaranov/sqlserver-kit/blob/master/SQL%20Server%20Name%20Convention%20and%20T-SQL%20Programming%20Style.md

## 修改规范

### 已有 SQL 文件
- 在现有 SQL 文件中直接编辑

### 新表/存储过程
- 在 `Sql` 目录的对应文件夹添加新 SQL 文件
- 在 csproj 中新增 `SqlScript` 或 `TSqlScript` 元素

### 通用
- 更新 csproj 属性：
    - `<LatestSchemaVersion>` 为最新版本
    - `EmbeddedResource` 以生成最新 C# 模型 
- 完整架构由 Sql target build 任务自动生成
- 迁移差异脚本需手动生成，并遵循[此指南](https://github.com/microsoft/healthcare-shared-components/tree/master/src/Microsoft.Health.SqlServer/SqlSchemaScriptsGuidelines.md)

## 性能检查清单

  - 谨慎使用计算列——即便持久化也可能有问题，简单存取可用。
  - 避免 CURSOR——需要它通常意味着设计有问题。
  - 少用 Trigger——很难正确实现。
  - 小心 @tables——在 TVP 可用，其他场景基数默认 1 可能严重失真。
  - 尽量少用临时表——创建/销毁开销大。
  - 使用 \#tables 时创建后不要再修改结构，否则无法缓存，索引需一次建好。
  - 禁用动态 SQL（安全、编译、临时表等风险），仅服务脚本/WIT 允许。
  - 只有一种合理执行计划时，用 hint 固定；否则优化器可能跑出坏计划。
  - 不要滥建二级索引，确认实际被用到；可用 sys.dm\_db\_index\_usage\_stats。
  - WHERE/JOIN 有索引时避免 OR，可用 UNION 达成同效果。

### 在线索引与页面压缩 

数据库升级时创建或重建索引务必使用 **ONLINE=ON**，否则 SQL Azure 会对表持有排他锁，阻塞访问并可能引起 LSI。示例：

``` sql
IF EXISTS (
    SELECT  *
    FROM    sys.indexes
    WHERE   name = 'IXC_PropertyDefinition_PropertyId'
            AND object_id = OBJECT_ID('PropertyDefinition')
            AND ignore_dup_key = 1
)
BEGIN
    CREATE UNIQUE NONCLUSTERED INDEX IXCl_PropertyDefinition_PropertyId ON PropertyDefinition (PartitionId, PropertyId ASC) INCLUDE (TypeId)
    WITH (DROP_EXISTING=ON, ONLINE=ON)
END
```
## SQL 迁移 

https://github.com/microsoft/healthcare-shared-components/tree/master/src/Microsoft.Health.SqlServer/SqlSchemaScriptsGuidelines.md

## SQL 兼容性

二进制应至少与最新及次新架构版本兼容。 

