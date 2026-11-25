# MySQL to SQL Server - Custom SSIS Data Type Mappings

## Overview

Custom XML data type mapping files that facilitate the
transfer of MySQL tables into Microsoft SQL Server using
the SQL Server Import and Export Wizard.

## Background

Two issues were initially encountered when attempting to
import MySQL tables into SQL Server:

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
| [TEST_RESULTS.md](TEST_RESULTS.md) | Documents testing, validation, and technical findings. |

Both XML files contain 31 data type mapping entries.

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
is already running, restart it to load the new mapping files.

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

## Verified Destination Data Type Mappings

| MySQL | SQL Server |
| --- | --- |
| TINYINT (signed) | SMALLINT |
| SMALLINT (signed) | SMALLINT |
| MEDIUMINT (signed) | INT |
| INT (signed) | INT |
| BIGINT (signed) | BIGINT |
| FLOAT | REAL |
| DOUBLE | FLOAT(53) |
| DECIMAL(p,s) | DECIMAL(p,s) |
| CHAR(n) | NCHAR(n) |
| VARCHAR(n) | NVARCHAR(n) |
| TINYTEXT | NVARCHAR(MAX) |
| TEXT | NVARCHAR(MAX) |
| MEDIUMTEXT | NVARCHAR(MAX) |
| LONGTEXT | NVARCHAR(MAX) |
| DATE | DATE |
| DATETIME(6) | DATETIME2(6) |
| TIMESTAMP(6) | DATETIME2(6) |
| YEAR | SMALLINT |
| BINARY(n) | BINARY(n) |
| VARBINARY(n) | VARBINARY(n) |
| BLOB | VARBINARY(MAX) |

Unicode character types were selected to preserve character
data while maintaining fixed-length and variable-length
distinctions.

These mappings have been verified using destination column
definitions and representative test values. Full data type
ranges and boundary conditions have not yet been tested.

## Additional Provider Type Identifiers

The mapping files accommodate numeric type identifiers
observed during Connector/NET metadata processing.

| Identifier | MySQL Source Type | SQL Server Type |
| --- | --- | --- |
| 246 | DECIMAL | DECIMAL |
| 752 | TEXT, MEDIUMTEXT, LONGTEXT | NVARCHAR(MAX) |
| 753 | VARBINARY | VARBINARY(n) |

## Validation

Successfully imported five tables from an existing MySQL
database and a dedicated 22-column standard data type
test table.

Validation included inspection of destination column
definitions, signed integer boundary values, decimal
precision, binary values, and execution of SQL queries
against the imported data.

Detailed findings are available in
[TEST_RESULTS.md](TEST_RESULTS.md).

## Temporal Precision

The Import and Export Wizard was found to assign
DT_DBTIMESTAMP to temporal data flow columns despite
correctly generating DATETIME2(6) destination columns.

This resulted in fractional-second precision loss beyond
three digits.

A package-level correction using DT_DBTIMESTAMP2
successfully preserved all six fractional-second digits.

An automated PowerShell correction utility is planned.
The XML mapping files alone do not resolve this issue.

## Known Limitations

- Temporal precision beyond milliseconds requires an
  additional package-level correction
- Not all mapping entries or supported value ranges have
  been fully tested
- Unsigned integers and specialized data types require
  further validation
- MySQL and SQL Server data type ranges, encodings, and
  temporal semantics differ
- Primary keys, foreign keys, indexes, defaults, and other
  schema objects require separate migration
- Compatibility with other providers and SSIS versions
  has not been established
- Software upgrades may require reinstallation or
  modification of the mapping files

## References

- [MySQL Connector/NET Documentation](https://dev.mysql.com/doc/connector-net/en/)
- [MySQL ODBC Bug #117005](https://bugs.mysql.com/bug.php?id=117005)
- [SQL Server Integration Services Documentation](https://learn.microsoft.com/en-us/sql/integration-services/)
- [Integration Services Data Types](https://learn.microsoft.com/en-us/sql/integration-services/data-flow/integration-services-data-types)

## Version History

### Version 1.1 - November 25, 2026

- Expanded mapping coverage to 31 entries per file
- Added provider type identifiers `752` and `753`
- Successfully mapped all 22 standard test columns
- Identified and verified a package-level correction for
  temporal precision loss
- Added separate testing documentation

### Version 1.0 - November 25, 2026

- Initial release of custom XML mapping files
- Preserved character lengths and numeric precision
- Added support for provider type identifier `246`
- Successfully imported five MySQL tables
