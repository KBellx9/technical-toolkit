# MySQL to SQL Server — Custom SSIS Data Type Mappings

## Overview

Custom XML data type mapping files developed to facilitate
the transfer of MySQL tables into Microsoft SQL Server
using the SQL Server Import and Export Wizard.

## Background

During attempts to import MySQL database tables into SQL
Server, two issues were encountered:

- The MySQL ODBC driver returned inconsistent and
  corrupted data type metadata.
- The SQL Server Import and Export Wizard lacked native
  MySQL data type mapping files.

The ODBC connection was replaced with MySQL Connector/NET,
and custom XML mapping files were developed using existing
Microsoft SSIS mapping files as structural templates.

## Files

| File | Description |
| --- | --- |
| MySqlToSSIS.xml | Maps MySQL source data types to intermediate SSIS data types. |
| MySqlToMSSql.xml | Maps MySQL source data types directly to SQL Server destination data types. |

## Tested Environment

- Windows 10 (64-bit)
- MySQL Server 8.0.43
- Microsoft SQL Server 2022
- SQL Server Management Studio 21
- MySQL Connector/NET 26.7
- Microsoft OLE DB Driver for SQL Server

## Installation

1. Install [MySQL Connector/NET](https://dev.mysql.com/downloads/connector/net/).
2. Close the SQL Server Import and Export Wizard.
3. Copy both XML files into the applicable SSIS
   MappingFiles directory.
4. Restart the wizard.

### SSMS 21 Directory

`C:\Program Files\Microsoft SQL Server Management Studio 21\Release\Common7\IDE\CommonExtensions\Microsoft\SSIS\170\MappingFiles`

### SQL Server 2022 Directory

`C:\Program Files\Microsoft SQL Server\160\DTS\MappingFiles`

The appropriate directory depends on which version of the
Import and Export Wizard is being used.

Successful migration testing was performed using the
wizard bundled with SSMS 21.

## Usage

1. Launch the SQL Server Import and Export Wizard.
2. Select **.NET Framework Data Provider for MySQL**
   as the source provider.
3. Supply the MySQL server, database, and authentication
   credentials.
4. Select **Microsoft OLE DB Driver for SQL Server**
   as the destination provider.
5. Configure the destination connection.
6. Specify the source SQL query.
7. Review the destination table name and column mappings.
8. Execute the import.

## Verified Data Type Mappings

| MySQL | SQL Server |
| --- | --- |
| CHAR(n) | NCHAR(n) |
| VARCHAR(n) | NVARCHAR(n) |
| DATE | DATE |
| DECIMAL(p,s) | DECIMAL(p,s) |

Character lengths, decimal precision, and decimal scale
were successfully preserved in testing.

The mapping files also accommodate native MySQL type
identifier `246` (`MYSQL_TYPE_NEWDECIMAL`), which was
encountered during decimal column conversion.

Unicode character types were selected to preserve
character data while maintaining the distinction between
fixed-length and variable-length strings.

## Validation

Successfully imported five tables from a MySQL database
into SQL Server using the custom mapping configuration.

Imported tables were subsequently queried in SSMS,
including multi-table joins and window functions.

Destination column definitions were inspected using
the SQL Server system stored procedure `sp_help`.

## Known Limitations

- Not all mapped MySQL data types have been individually
  tested.
- Primary keys, foreign keys, indexes, and other database
  constraints require separate consideration.
- MySQL and SQL Server have differences in supported
  data type ranges and precision.
- Connector or SSIS upgrades may require reinstallation
  or modification of the mapping files.
- Compatibility with ODBC providers has not been
  established.
- Unsigned integers, spatial data types, and other
  specialized MySQL data types require further testing.

## References

- [MySQL Connector/NET Documentation](https://dev.mysql.com/doc/connector-net/en/)
- [MySQL ODBC Bug #117005](https://bugs.mysql.com/bug.php?id=117005)
- [SQL Server Integration Services Documentation](https://learn.microsoft.com/en-us/sql/integration-services/)

## Version History

### Version 1.0 — November 25, 2026

- Created custom MySQL-to-SSIS mappings.
- Implemented direct MySQL-to-SQL Server mappings.
- Preserved fixed-length and variable-length character
  distinctions.
- Added support for native MySQL decimal type
  identifier 246.
- Successfully completed initial database migration
  testing across five tables.
