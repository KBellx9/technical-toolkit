# MySQL to SQL Server - Test Results

## Overview

This document records the testing and validation of the
custom MySQL-to-SSIS and MySQL-to-SQL Server XML data
type mapping files.

**Mapping Version:** 1.1

**Initial Test Date:** November 25, 2026

## Testing Environment

- Windows 10 (64-bit)
- MySQL Server 8.0.43
- MySQL Connector/NET 26.7
- Microsoft SQL Server 2022
- SQL Server Management Studio 21
- SSIS Import and Export Wizard bundled with SSMS 21
- Microsoft OLE DB Driver for SQL Server
- SQL Server Execute Package Utility 16.0.1000.6
- Windows PowerShell

## Initial Database Migration

Five tables from an existing MySQL database were
successfully imported into SQL Server.

Validation included:

- Inspection of destination column definitions using
  `sp_help`
- Preservation of fixed-length and variable-length
  character distinctions
- Preservation of decimal precision and scale
- Execution of multi-table joins and window functions

During decimal column conversion, Connector/NET exposed
the numeric type identifier `246` rather than the
conventional DECIMAL type name.

Adding this identifier to both XML mapping files
resolved the unrecognized destination type.

## Phase 1 - Standard Data Types

### Test Configuration

A dedicated database named `ssis_mapping_test` was
created in both MySQL and SQL Server.

The source table, `standard_types`, contained 22 columns
representing common numeric, character, temporal, and
binary data types.

### Initial Mapping Results

Four columns initially returned unrecognized destination
data types.

| Source Column | MySQL Type | Reported Identifier |
| --- | --- | --- |
| test_text | TEXT | 752 |
| test_mediumtext | MEDIUMTEXT | 752 |
| test_longtext | LONGTEXT | 752 |
| test_varbinary | VARBINARY | 753 |

Both XML mapping files were updated to accommodate
these identifiers.

| Identifier | SSIS Type | SQL Server Type |
| --- | --- | --- |
| 752 | DT_NTEXT | NVARCHAR(MAX) |
| 753 | DT_BYTES | VARBINARY(n) |

Following these modifications, all 22 columns were
successfully mapped without manual column type adjustments.

### Destination Schema Validation

| MySQL Type | SQL Server Type | Result |
| --- | --- | --- |
| INT (test identifier) | INT | Pass |
| TINYINT | SMALLINT | Pass |
| SMALLINT | SMALLINT | Pass |
| MEDIUMINT | INT | Pass |
| INT | INT | Pass |
| BIGINT | BIGINT | Pass |
| FLOAT | REAL | Pass |
| DOUBLE | FLOAT(53) | Pass |
| DECIMAL(15,4) | DECIMAL(15,4) | Pass |
| CHAR(10) | NCHAR(10) | Pass |
| VARCHAR(100) | NVARCHAR(100) | Pass |
| TINYTEXT | NVARCHAR(MAX) | Pass |
| TEXT | NVARCHAR(MAX) | Pass |
| MEDIUMTEXT | NVARCHAR(MAX) | Pass |
| LONGTEXT | NVARCHAR(MAX) | Pass |
| DATE | DATE | Pass |
| DATETIME(6) | DATETIME2(6) | Pass |
| TIMESTAMP(6) | DATETIME2(6) | Pass |
| YEAR | SMALLINT | Pass |
| BINARY(4) | BINARY(4) | Pass |
| VARBINARY(100) | VARBINARY(100) | Pass |
| BLOB | VARBINARY(MAX) | Pass |

### Numeric Data Validation

| MySQL Type | Source Value | Destination Value | Result |
| --- | --- | --- | --- |
| TINYINT | -128 | -128 | Pass |
| SMALLINT | -32768 | -32768 | Pass |
| MEDIUMINT | -8388608 | -8388608 | Pass |
| INT | -2147483648 | -2147483648 | Pass |
| BIGINT | -9223372036854775808 | -9223372036854775808 | Pass |
| FLOAT | 12.345 | 12.345 | Pass |
| DOUBLE | 12345.678901234 | 12345.678901234 | Pass |
| DECIMAL(15,4) | 123456789.1234 | 123456789.1234 | Pass |

Floating-point values were verified at displayed precision.
Exact binary floating-point representation was not tested.

### Binary Data Validation

| MySQL Type | Source Value | Destination Value | Result |
| --- | --- | --- | --- |
| BINARY(4) | 0x01020304 | 0x01020304 | Pass |
| VARBINARY(100) | 0x0102030405 | 0x0102030405 | Pass |
| BLOB | 0x48656C6C6F | 0x48656C6C6F | Pass |

### Initial Temporal Data Validation

| MySQL Type | Source Value | Destination Value | Result |
| --- | --- | --- | --- |
| DATETIME(6) | 12:30:45.123456 | 12:30:45.123000 | Fail |
| TIMESTAMP(6) | 12:30:45.123456 | 12:30:45.123000 | Fail |

Although the destination columns were correctly defined
as DATETIME2(6), fractional-second precision was lost
during the transfer.

## Temporal Precision Investigation

### Component Isolation

Independent Connector/NET testing was performed through
Windows PowerShell using the MySQL data reader.

The `GetDateTime()` method successfully returned all six
fractional-second digits.

A second import was performed using the .NET Framework
Data Provider for SQL Server instead of the Microsoft
OLE DB Driver for SQL Server.

The same precision loss occurred.

| Test | Fractional Seconds | Result |
| --- | --- | --- |
| MySQL source database | .123456 | Pass |
| Direct Connector/NET retrieval | .123456 | Pass |
| SSIS with OLE DB destination | .123000 | Fail |
| SSIS with .NET destination | .123000 | Fail |

These results isolated the issue to the SSIS transfer
process in the tested configuration.

### Root Cause

Inspection of the generated SSIS package revealed that
the wizard assigned DT_DBTIMESTAMP to the internal
temporal data flow columns.

| SSIS Type | Maximum Fractional-Second Digits |
| --- | --- |
| DT_DBTIMESTAMP | 3 |
| DT_DBTIMESTAMP2 | 7 |

The lower-precision internal representation resulted in
the loss of microsecond precision before the values were
stored in the destination columns.

### Package-Level Correction

A copy of the generated SSIS package was modified to
replace the affected temporal metadata.

Original XML attribute:

`dataType="dbTimeStamp"`

Corrected XML attributes:

`dataType="dbTimeStamp2" scale="6"`

The corresponding source, data flow, and destination
metadata references were updated consistently.

The modified package was executed through Windows
PowerShell using `DTExec.exe`.

Execution returned:

`DTSER_SUCCESS (0)`

### Corrected Temporal Data Validation

| MySQL Type | Source Value | Destination Value | Result |
| --- | --- | --- | --- |
| DATETIME(6) | 12:30:45.123456 | 12:30:45.123456 | Pass |
| TIMESTAMP(6) | 12:30:45.123456 | 12:30:45.123456 | Pass |

The package-level correction successfully preserved
all six fractional-second digits.

### Conclusion

The custom XML mapping files correctly define the
destination temporal data types but do not independently
prevent the wizard from assigning a lower-precision
internal data flow type.

A separate package-level correction is required in the
tested configuration.

Automation of this correction through a reusable
PowerShell script is planned.

## Testing Status

| Test Category | Status |
| --- | --- |
| Initial five-table migration | Pass |
| Standard destination data type mappings | 22/22 Pass |
| Minimum signed integer values | Pass |
| Decimal sample precision and scale | Pass |
| Floating-point sample values | Pass |
| Fixed-length and variable-length character definitions | Pass |
| Binary sample values | Pass |
| Temporal destination column definitions | Pass |
| Unmodified temporal data transfer | Fail |
| Corrected temporal data transfer | Pass |
| Comprehensive text and Unicode validation | Pending |
| NULL value testing | Pending |
| Unsigned integer testing | Pending |
| Specialized data type testing | Pending |
| Maximum-length value testing | Pending |
| Automated temporal precision correction | Pending |
| Schema object migration | Pending |
| Additional SSIS version compatibility | Pending |

## Planned Testing

### Phase 2 - Unsigned Integers

- TINYINT UNSIGNED
- SMALLINT UNSIGNED
- MEDIUMINT UNSIGNED
- INT UNSIGNED
- BIGINT UNSIGNED
- Maximum values and overflow prevention

### Phase 3 - Specialized Data Types

- BIT and BOOLEAN
- ENUM and SET
- JSON
- TIME and extended duration values
- Spatial data types

### Phase 4 - Boundary Conditions

- NULL values
- Minimum and maximum numeric values
- High-precision decimal values
- Unicode and supplementary characters
- Maximum-length character and binary values
- Temporal range and time zone behavior

### Phase 5 - Schema Migration

- Primary keys
- Foreign keys
- Indexes
- Default values
- Auto-increment and identity properties
- Check constraints

### Phase 6 - Compatibility Testing

- Additional Import and Export Wizard versions
- Additional Connector/NET versions
- Automated SSIS package correction

## References

- [Integration Services Data Types](https://learn.microsoft.com/en-us/sql/integration-services/data-flow/integration-services-data-types)
- [DTExec Utility](https://learn.microsoft.com/en-us/sql/integration-services/packages/dtexec-utility)
- [MySQL Connector/NET Documentation](https://dev.mysql.com/doc/connector-net/en/)
