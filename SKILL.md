---
name: database-standards
description: 约束 AI 生成数据库代码质量的规范集（SQL/表设计/索引/分页/查询反模式/数据安全）。生成 SQL、建表 DDL、索引、分页查询、查询优化代码前必须加载；写 Mapper/DAO/Repository 层、审查慢查询时加载。触发场景：写 SQL、建表、设计索引、分页查询、N+1 排查、SQL 优化、DELETE/UPDATE 审查。
---

# Database Standards

约束 AI 生成数据库代码（SQL / 表设计 / 索引 / 分页 / 查询）的规范集。面向 MySQL，ORM 无关，适用任何技术栈。

## 加载矩阵

| 任务类型 | 必读 | 建议读 |
|---|---|---|
| 写查询 SQL | `sql-standards.md` | `index-standards.md` / `query-anti-patterns.md` |
| 建表 / 表结构 DDL | `table-design-standards.md` | `index-standards.md` |
| 设计索引 | `index-standards.md` | `sql-standards.md` |
| 分页查询 | `pagination-standards.md` | `index-standards.md` |
| 写 DAO / Mapper / Repository | `query-anti-patterns.md` + `sql-standards.md` | `pagination-standards.md` |
| 写 UPDATE / DELETE / DDL 变更 | `data-safety.md`（最高优先） | `table-design-standards.md` |
| 优化慢查询 | `query-anti-patterns.md` + `index-standards.md` | `sql-standards.md` |

## 核心规则速查

### 安全（最高优先，写操作必过）
- UPDATE/DELETE 必带 WHERE；大范围写先备份 + SELECT 预验证
- 大表 DDL 用在线变更，进版本管理；时间统一 DATETIME，左闭右开边界

### 查询
- 禁止 SELECT *、禁止 N+1（循环查库）、禁止超大 IN（>1000）
- 禁止函数包裹索引列 / 隐式类型转换（索引失效）
- join ≤ 3-4 表，小表驱动大表，ON 显式关联

### 索引
- 复合索引最左前缀，等值前范围后；索引数 ≤ 5；唯一约束有唯一索引兜底
- 区分度低列不单独建索引；EXPLAIN 验证 type 不为 ALL

### 分页
- 深分页用键集分页（id < lastId）；ORDER BY 加唯一字段兜底；pageSize 有上限

## 使用要求

生成任何 SQL/数据库代码前，按「任务类型 → 加载矩阵」读取规范；生成后对照对应规范「自检清单」逐项核对。违反强制规则即返工。写操作类 SQL（UPDATE/DELETE/DDL）必须过 `data-safety.md` 自检。
