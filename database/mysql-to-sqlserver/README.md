# MySQL to SQL Server - Custom SSIS Data Type Mappings

## Overview

Custom XML data type mapping files that facilitate the
transfer of MySQL tables into Microsoft SQL Server using
the SQL Server Import and Export Wizard.

## Background

Two issues were encountered when attempting to import
MySQL tables into SQL Server:

- The MySQL ODBC driver returned inconsistent and
  corrupted data type metadata
- The SQL Server Import and Export Wizard lacked native
  MySQL data type mapping files

The ODBC connection was replaced with MySQL Connector/NET,
and custom XML mapping files were developed using existing
Microsoft SSIS mapping files as structural templates.

## Files

| File | Description |
| --- | --- |
| [MySqlToSSIS.xml](MySqlToSSIS.xml) | Maps MySQL source data types to intermediate SSIS data types. |
| [MySqlToMSSql.xml](MySqlToMSSql.xml) | Defines corresponding SQL Server destination data types. |

## Tested Environment

- Windows 10 (64-bit)
- MySQL Server 8.0.43
- MySQL Connector/NET 26.7
- Microsoft SQL Server 2022
- SQL Server Management Studio (SSMS) 21
- SQL Server Import and Export Wizard bundled with SSMS 21
- Microsoft OLE DB Driver for SQL Server

## Installation

1. Install [MySQL Connector/NET](https://dev.mysql.com/downloads/connector/net/)
2. Copy both XML mapping files into the SSIS `MappingFiles`
   directory

### Verified Installation Directory (SSMS 21)

`C:\Program Files\Microsoft SQL Server Management Studio 21\Release\Common7\IDE\CommonExtensions\Microsoft\SSIS\170\MappingFiles`

**Note:** The installation directory may vary depending on
the SSMS or SSIS version. If the Import and Export Wizard
is already running, restart it to load the new mapping
files.

## Usage

1. In SSMS, right-click the destination database and select
   **Tasks > Import Data**
2. Select **.NET Framework Data Provider for MySQL**
   as the source provider
3. Configure the MySQL connection
4. Select **Microsoft OLE DB Driver for SQL Server**
   as the destination provider
5. Configure the SQL Server connection
6. Specify a SQL query to retrieve the source data
7. Review the destination table name and column mappings
8. Execute the import

## Verified Data Type Mappings

| MySQL | SSIS | SQL Server |
| --- | --- | --- |
| CHAR(n) | DT_WSTR | NCHAR(n) |
| VARCHAR(n) | DT_WSTR | NVARCHAR(n) |
| DATE | DT_DBDATE | DATE |
| DECIMAL(p,s) | DT_NUMERIC | DECIMAL(p,s) |

Character lengths, decimal precision, and decimal scale
were successfully preserved in testing.

Unicode character types were selected to preserve
character data while maintaining the distinction between
fixed-length and variable-length strings.

The mapping files also accommodate native MySQL type
identifier `246` (`MYSQL_TYPE_NEWDECIMAL`), which was
encountered during decimal column conversion.

## Validation

Five tables were successfully imported from a MySQL
database into SQL Server using the custom mapping
configuration.

Validation included:

- Inspection of destination column definitions using
  the SQL Server system stored procedure `sp_help`
- Verification of character lengths and numeric precision
  and scale
- Execution of multi-table joins and window functions
  against the imported data

## Known Limitations

- Not all mapped data types have been individually tested
- Primary keys, foreign keys, indexes, and other database
  constraints require separate migration
- Differences in supported data type ranges and precision
  between MySQL and SQL Server may require additional
  conversions
- Unsigned integers, spatial data types, and other
  specialized data types require further investigation
- Compatibility with ODBC providers and other SSIS
  versions has not been established
- Software upgrades may require reinstallation or
  modification of the mapping files

## References

- [MySQL Connector/NET Documentation](https://dev.mysql.com/doc/connector-net/en/)
- [MySQL ODBC Bug #117005](https://bugs.mysql.com/bug.php?id=117005)
- [SQL Server Integration Services Documentation](https://learn.microsoft.com/en-us/sql/integration-services/)

## Version History

### Version 1.0 - November 25, 2026

- Initial release of custom MySQL-to-SSIS and
  MySQL-to-SQL Server mapping files
- Preservation of fixed-length and variable-length
  character distinctions
- Support for native MySQL decimal type identifier `246`
- Successful migration testing across five tables
