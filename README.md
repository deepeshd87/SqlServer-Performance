# SqlServer-Performance

# SQL Server Index Maintenance Script

A dynamic T-SQL script that automatically detects and fixes index fragmentation 
across all (or a specified) SQL Server database — supporting both reorganize and 
rebuild operations based on fragmentation thresholds.

---

##  Features

- Scans **all databases** or a **specific database**
- Automatically chooses between **REORGANIZE** or **REBUILD** based on fragmentation level
- Supports **Always On Availability Groups (HADR)** — skips secondary replicas
- **Report-only mode** — preview fragmentation without making changes
- Verbose logging mode for debugging
- Skips system databases (`master`, `model`, `msdb`, `tempdb`, `distribution`)

---

##  Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `@databaseToCheck` | VARCHAR(250) | NULL | Target database. NULL = all databases |
| `@reportOnly` | BIT | — | 1 = report only, 0 = execute fixes |
| `@NOTIFY_ON_ALERT_ONLY` | CHAR(1) | 'N' | Alert notification toggle |

---

##  Internal Configuration

| Setting | Value | Description |
|---|---|---|
| `@fragmentationThreshold` | 15% | Minimum fragmentation to act on |
| `@indexFillFactor` | 70 | Fill factor used during REBUILD |
| `@indexStatisticsScanningMode` | LIMITED | Scan mode for DMV query |
| `@sortInTempdb` | ON | Sort index rebuild in TempDB |

---

##  Fragmentation Logic

| Fragmentation Level | Action |
|---|---|
| > 30% | `ALTER INDEX ... REBUILD` |
| 5% – 30% | `ALTER INDEX ... REORGANIZE` |
| < 5% | No action |

> Only indexes with **more than 500 pages** are included to avoid 
> unnecessary maintenance on small indexes.

---

## 📋 Prerequisites

- SQL Server 2005 or later (compatibility level ≥ 90)
- **sysadmin** server role required
- Script must be run on the target SQL Server instance

---

## 💻 Usage

### Report Mode (no changes made)
```sql
DECLARE @databaseToCheck VARCHAR(250) = NULL  -- NULL = all databases
DECLARE @reportOnly BIT = 1                   -- Report only
```

### Execute Mode (applies fixes)
```sql
DECLARE @databaseToCheck VARCHAR(250) = 'AdventureWorks2022'
DECLARE @reportOnly BIT = 0   -- Execute reorganize/rebuild
```

---

##  Output (Report Mode)

When `@reportOnly = 1`, returns a result set with:

| Column | Description |
|---|---|
| `dbName` | Database name |
| `tableName` | Table name |
| `schemaName` | Schema name |
| `indexName` | Index name |
| `PageCount` | Number of pages |
| `AvgFragmentationPercentage` | Fragmentation % (sorted DESC) |
---


## ⚠️ Notes

- Requires **sysadmin** privileges — will print an error and exit otherwise
- AlwaysOn secondary replicas are automatically excluded
- REBUILD operations lock the table — consider running during off-peak hours
- Use `@verboseMode = 1` (inside the script) for detailed execution logs

---

## 📜 License

MIT License — free to use and modify.
