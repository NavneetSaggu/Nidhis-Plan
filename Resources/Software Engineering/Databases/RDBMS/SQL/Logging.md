# Logging

The log levels used in MySQL can be configured using the `log_error_verbosity` system variable. The available log levels are:

- 1 (Errors only) - logs only error messages.
- 2 (Errors and warnings) - logs error and warning messages.
- 3 (Errors, warnings, and notes) - logs error, warning, and informational messages.
- 4 (Errors, warnings, notes, and connections) - logs all types of messages, as well as information about client connections and disconnections.
- 5 (Errors, warnings, notes, connections, and queries) - logs all types of messages, as well as the text of SQL statements executed by clients.

## Binary Logging

- Records changes made to the database in a binary format
- Can be used for replication, backup, and point-in-time recovery.
- Is a form of transaction logging; it records all the changes made to database as sequence of transactions, each having unique transaction ID.
- These logs are written to a binary log file; which can be read by other MySQL servers to replicate the changes.
- This includes all DML, including INSERT, UPDATE, DELETE, and DDL statements, as well as changes to system variables and the contents of stored procedures and triggers.