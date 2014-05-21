## Foreign key constraint
</br>

	create table foods(
  		id integer primary key,
  		type_id integer references food_types(id)
		on delete restrict
  		deferrable initially deferred,
		name text );

The differences are shown in bold and are easy to understand if taken one at a time. The first part of the foreign key instructs SQLite that the type_id column references the id column of the food_types table. From there, you move on to the integrity action clause, which does most of the hard work. In this example, you've used the option on delete restrict. This instructs SQLite to prevent any deletion from the food_types table that would leave a food in the foods table without a parent food id. restrict is one of five possible actions you can define. The full set is as follows:

<b>set null:</b> Change any remaining child value to NULL if the parent value is removed or changed to no longer exist.

<b>set default:</b> Change any remaining child value to the column default if the parent value is removed or changed to no longer exist.

<b>cascade:</b> Where the parent key is updated, update all child keys to match. Where it is deleted, delete all child rows with the key. Pay particular attention to this option, because cascading deletes can surprise you when you least expect them.

<b>restrict:</b> Where the update or delete of the parent key would result in orphaned child keys, prevent (abort) the transaction.

<b>no action:</b> Take a laid-back approach, and watch changes fly by without intervening. Only at the end of the entire statement (or transaction if the constraint is deferred) is an error raised.

Lastly, SQLite supports the deferrable clause, which controls whether the constraint as defined will be enforced immediately or deferred until the end of the transaction.


## Copying a table to another table command
</br>

	create table <new_table> as select <C1,C2> from <old_table>;
	
## Comparing Value types

1. The NULL storage class has the lowest class value. A value with a NULL storage class is considered less than any other value (including another value with storage class NULL). Within NULL values, there is no specific sort order.

2. integer or real storage classes have higher value than NULLs and share equal class value. integer and real values are compared numerically.

3. The text storage class has higher value than integer or real. A value with an integer or a real storage class will always be less than a value with a text storage class no matter its value. When two text values are compared, the comparison is determined by the collation defined for the values.

4. The blob storage class has the highest value. Any value that is not of class blob will always be less than a value of class blob. blob values are compared using the C function memcmp().

## Views
</br>

		create view name as select-stmt;
		
<b>The contents of views are dynamically generated. Thus, every time you use details, its associated SQL will be reexecuted, producing results based on the data in the database at that moment. </b>

to drop a table: 
	
		drop view name;
		
## Indexes

	create index [unique] index_name on table_name (columns)
		
To remove an index, use the drop index command, which is defined as follows:

	drop index index_name;
	
## Triggers

	create [temp|temporary] trigger name
	[before|after] [insert|delete|update|update of columns] on table
	action
	
A trigger is defined by a name, an action, and a table. The action, or trigger body, consists of a series of SQL commands. Triggers are said to fire when such events take place. Furthermore, triggers can be made to fire before or after the event using the before or after keyword, respectively. Events include delete, insert, and update commands issued on the specified table. 

### Update Triggers
Update triggers, unlike insert and delete triggers, may be defined for specific columns in a table. 
		
	create trigger name
	[before|after] update of column on table
	action
	
an example of how the command works:

	create temp table log(x);

	create temp trigger foods_update_log update of name on foods
	begin
	insert into log values('updated foods: new name=' || new.name);
	end;

SQLite provides access to both the old (original) row and the new (updated) row in update triggers. The old row is referred to as old and the new row as new. ou could have just as easily recorded new.type_id or old.id.

### Error handling

In order to raise an error we use the following function in SQlite:

	raise(resolution, error_message);
	
The first argument is a conflict resolution policy (abort, fail, ignore, rollback, and so on). The second argument is an error message. 

## Rolling back the changes
If you want to test something on a database you could use the follwoing command to avoid changing the data in the database (It is always the best that you create a new instance of the database when you want to do tests)

	begin;
	<Do the tests>
	rollback;

## Transactions

- <b>begin:</b> starts a transaction. Every operation following a begin can be potentially undone and will be undone if a commit is not issued before the session terminates.
- <b>commit:</b> command commits the work performed by all operations since the start of the transaction.
- <b>rollback:</b> undoes all the work performed by all operations since the start of the transaction.
- <b>savepoint:</b> It is a point where SQLite can revert to in the event of a rollback. To create a savepoint use the following function:
	
		savepoint <name of savepoint>;

In order to revert back to a savepoint use the following command:
		
		rollback [transaction] to <name of savepoint>;
		
## Conflict Resolution

1. <b>replace: </b> When a unique constraint violation is encountered, SQLite removes the row (or rows) that caused the violation and replaces it (them) with the new row from the insert or update. The SQL operation continues without error. If a NOT NULL constraint violation occurs, the NULL value is replaced by the default value for that column. If the column has no default value, then SQLite applies the abort policy. It is important to note that when this conflict resolution strategy deletes rows in order to satisfy a constraint, it does not invoke delete triggers on those rows. This behavior, however, is subject to change in a future release.
2. <b>ignore: </b>When a constraint violation is encountered, SQLite allows the command to continue and leaves the row that triggered the violation unchanged. Other rows before and after the row in question continue to be modified by the command. Thus, all rows in the operation that trigger constraint violations are simply left unchanged, and the command proceeds without error.
3. <b>fail:</b>  When a constraint violation is encountered, SQLite terminates the command but does not restore the changes it made prior to encountering the violation. That is, all changes within the SQL command up to the violation are preserved. For example, if an update statement encountered a constraint violation on the 100th row it attempts to update, then the changes to the first 99 rows already modified remain intact, but changes to rows 100 and beyond never occur as the command is terminated.
4. <b>abort: </b>When a constraint violation is encountered, SQLite restores all changes the command made and terminates it. abort is the default resolution for all operations in SQLite. It is also the behavior defined in the SQL standard. As a side note, abort is also the most expensive conflict resolution policy—requiring extra work even if no conflicts ever occur.
5. <b>rollback: </b> When a constraint violation is encountered, SQLite performs a rollback—aborting the current command along with the entire transaction. The net result is that all changes made by the current command and all previous commands in the transaction are rolled back. This is the most drastic level of conflict resolution where a single violation results in a complete reversal of everything performed in a transaction.

		insert or resolution into table (column_list) values (value_list);
		update or resolution table set (value_list) where predicate;
		
## Database Locking
There are five locking states in sqlite database:

1. <b>unlocked: </b>In this state, no session is accessing data from the database. When you connect to a database or even initiate a transaction with BEGIN, your connection is in the unlocked state.
2. <b>shared: </b> For a session to read from the database (not write), it must first enter the shared state and must therefore acquire a shared lock. Multiple sessions can simultaneously acquire and hold shared locks at any given time. Therefore, multiple sessions can read from a common database at any given time. However, no session can write to the database during this time—while any shared locks are active.
3. <b>reserved: </b>If a session wants to write to the database, it must require this lock.Only one reserved lock may be held at one time for a given database. Shared locks can coexist with a reserved lock. A reserved lock is the first phase of writing to a database. It does not block sessions with shared locks from reading, and it does not prevent sessions from acquiring new shared locks. Once a session has a reserved lock, it can begin the process of making modifications; however, these modifications are cached and not actually written to disk. The reader's changes are stored in a memory cache.
4. <b>pending: </b> To get an exclusive lock, it must first promote its reserved lock to a pending lock. A pending lock starts a process of attrition whereby no new shared locks can be obtained.That is, other sessions with existing shared locks are allowed to continue as normal, but other sessions cannot acquire new shared locks. At this point, the session with the pending lock is waiting for the other sessions with shared locks to finish what they are doing and release them.
5. <b>exclusive: </b>When the session wants to commit the changes (or transaction) to the database, it begins the process of promoting its reserved lock to an exclusive lock. Once all shared locks are released, the session with the pending lock can promote it to an exclusive lock. It is then free to make changes to the database. All of the previously cached changes are written to the database file.

## Transcation Types
Different transaction types can be used to avoid deadlock situation.

	begin [ deferred | immediate | exclusive ] transaction;
	
1. <b>deferred: </b> It does not acquire any locks until it has to. Thus, with a deferred transaction, the begin statement itself does nothing—it starts in the unlocked state. This is the default.
2. <b>immediate: </b> It attempts to obtain a reserved lock as soon as the begin command is executed. If successful, begin immediate guarantees that no other session will be able to write to the database. Other sessions can continue to read from the database, but the reserved lock prevents any new sessions from reading. Another consequence of the reserved lock is that no other sessions will be able to successfully issue a begin immediate or begin exclusive command.
3. <b>exclusive: </b>An exclusive transaction obtains an exclusive lock on the database. This works similarly to immediate, but when you successfully issue it, exclusive guarantees that no other session is active in the database and that you can read or write with impunity.

## Attaching Databases
When you attach a database, all of its contents are accessible in the global scope of the current database file.

	attach [database] filename as database_name;
	
	detach [database] database_name;
	
## Cleaning Databases
SQLite has two commands designed for cleaning—reindex and vacuum. reindex is used to rebuild indexes. It has two forms:

	reindex collation_name;
	reindex table_name|index_name;

- The first form rebuilds all indexes that use the collation name given by collation_name. It is needed only when you change the behavior of a user-defined collating sequence (for example, multiple sort orders in Chinese).
- <i>Vacuum</i> cleans out any unused space in the database by rebuilding the database file. Vacuum will not work if there are any open transactions. An alternative to manually running VACUUM statements is autovacuum. This feature is enabled using the auto_vacuum pragma, described in the next section.

## Database Configuration
use pragma before these
### The Connection Cache Size

	 pragma cache_size; -- Return the cache size of the database
	 pragma cache_size=10000; -- Set the cache size of the database

You can permanently set the cache size for all sessions using the default_cache_size pragma.

### Getting Database Information

- <i>database_list</i>: Lists information about all attached databases.

- <i>index_info</i>: Lists information about the columns within an index. It takes an index name as an argument.
	
		pragma index_info(<index name>);

- <i>index_list</i>: Lists information about the indexes in a table. It takes a table name as an argument.

		pragma index_list(<table_name);

- <i>table_info</i>: Lists information about all columns in a table.

		pragma  table_info(<table name>);
		
### Synchronous Writes

Normally, SQLite commits all changes to disk at critical moments to ensure transaction durability. You can turn this off using the following commands

	pragma synchronous [full|normal|off];
	
- <b><i>full: </i></b> SQLite will pause at critical moments to make sure that data has actually been written to the disk surface before continuing. This ensures that if the operating system crashes or if there is a power failure, the database will be uncorrupted after rebooting. Full synchronous is very safe, but it is also slow.
- <b><i>normal: </i></b>SQLite will still pause at the most critical moments but less often than in full mode. There is a very small (though nonzero) chance that a power failure at just the wrong time could corrupt the database in normal mode. But in practice, you are more likely to suffer a catastrophic disk failure or some other unrecoverable hardware fault.
- <b><i>off: </i></b>SQLite continues operation without pausing as soon as it has handed data off to the operating system. This can speed up some operations as much as 50 or more times. If the application running SQLite crashes, the data will be safe. However, if the operating system crashes or the computer loses power, the database may be corrupted.

### Temporary Storage

Temporary storage is where SQLite keeps transient data such as temporary tables, indexes, and other objects. By default, SQLite uses a compiled-in location, which varies between platforms.

- <i><b>temp_store: </b></i>It determines whether SQLite uses memory or disk for temporary storage. There are actually three possible values: default, file, or memory.
	- <i>default: </i>uses the compiled-in default
	- <i>file: </i> uses an operating system file
	- <i>memory: </i> uses RAM
- <i><b>temp_store_directory: </b></i>It can be used to set the directory in which the temporary storage file is placed.

### Page Size, Encoding, and Autovacuum
The database page size, encoding, and autovacuuming must be set before a database is created. That is, to alter the defaults, you must first set these pragmas before creating any database objects in a new database.

SQLite supports page sizes ranging from 512 to 32,786 bytes, in powers of 2. 

Supported encodings are UTF-8, UTF-16le (little-endian UTF-16 encoding), and UTF-16be (big-endian UTF-16 encoding).

A database's size can be automatically kept to a minimum using the auto_vacuum pragma.

## The System Catalog
The sqlite_master table is a system table that contains information about all the tables, views, indexes, and triggers in the database. 

## View Query Plans
You can view the way SQLite goes about executing a query by using the 
		
		explain query plan <commands>;

The explain query plan command lists the steps SQLite carries out to access and process tables and data to satisfy your query.

You can spot when and how indices are used and the order in which tables are used in joins. This is of immense help when troubleshooting long-running queries and other issues.






